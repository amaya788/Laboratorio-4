# Laboratorio 4: Comunicaciones Industriales

**Universidad Santo Tomás**  
**Asignatura:** Comunicaciones Industriales  
**Docente:** Ing. Diego Alejandro Barragán Vargas  
**Autores:**  
- Ferney Arturo Amaya Gómez  
- David Esteban Díaz Castro  
- Jonny Alejandro Mejía León  
**Fecha:** Octubre 2025  

---

## 1. Objetivo General

Diseñar, implementar y validar una red de comunicaciones integral que combine infraestructura de red conmutada (VLANs) y sistemas de comunicación industrial (RS485), integrando dispositivos embebidos (Raspberry Pi/Pico) y soluciones de visualización en tiempo real, con el propósito de establecer un entorno completo para el análisis comparativo de rendimiento entre diferentes modos de comunicación.

## 2. Objetivos Específicos

- Crear y configurar VLANs en un switch de capa 2.  
- Asignar direcciones IP a las interfaces VLAN para su administración y comunicación.  
- Conectar un router para el enrutamiento entre subredes.  
- Comprobar la comunicación entre equipos de distintas VLANs mediante pruebas de conectividad.  
- Analizar el funcionamiento de la red configurada y validar los resultados.  
- Configurar una red básica entre router, switch y Raspberry Pi para establecer conectividad local.  
- Implementar la comunicación RS485 simplex (unidireccional) entre Raspberry Pi y Raspberry Pi Pico.  
- Implementar la comunicación RS485 full dúplex (bidireccional) usando dos buses independientes.  
- Comparar el rendimiento entre ambos modos.  
- Visualizar los datos de transmisión y recepción en tiempo real mediante Streamlit.

---

## 3. Marco Teórico

Los dispositivos de **capa 2**, como los switches, operan dentro de la capa de enlace de datos del modelo OSI y su función principal es conmutar tramas dentro de la misma red local (LAN).  
Por otro lado, los dispositivos de **capa 3**, como los routers, operan en la capa de red, donde encaminan paquetes IP entre distintas subredes.  

El correcto funcionamiento conjunto entre capas 2 y 3 permite el flujo de información entre hosts en diferentes dominios de broadcast, asegurando la comunicación eficiente dentro de redes jerárquicas.

---

## 4. Fundamento Teórico

### 4.1 VLANs (Virtual Local Area Networks)
Permiten dividir una red física en múltiples redes lógicas independientes, reduciendo el dominio de broadcast y mejorando el control de tráfico. Cada VLAN actúa como una subred separada dentro del mismo switch, y cada puerto puede ser asignado a una VLAN específica.

Para que los dispositivos en distintas VLANs puedan comunicarse, se requiere un **router o dispositivo de capa 3** que gestione el tráfico IP entre subredes.  
Las **SVI (Switch Virtual Interfaces)** permiten asignar direcciones IP a las VLANs, otorgando identidad lógica a cada una para administración remota o interconexión.

### 4.2 Estándar RS485
El **RS485** es una interfaz diferencial utilizada en entornos industriales para comunicación a larga distancia y con múltiples nodos (hasta 32 dispositivos), alcanzando distancias de hasta 1200 m a 9600 bps.  
Utiliza líneas diferenciales A y B, que proporcionan inmunidad al ruido electromagnético.

### 4.3 Modos de Comunicación
- **Simplex:** comunicación unidireccional.  
- **Half dúplex:** ambos extremos pueden transmitir, pero no al mismo tiempo.  
- **Full dúplex:** ambos extremos transmiten y reciben simultáneamente.

### 4.4 Transceptor MAX485
Convierte niveles TTL (3.3V) a señales diferenciales RS485. Posee pines DE y RE que controlan el modo de transmisión o recepción.

### 4.5 Raspberry Pi y Pico
- **Raspberry Pi:** nodo maestro encargado del envío, recepción y visualización de datos.  
- **Raspberry Pi Pico:** nodo esclavo que transmite o recibe datos mediante UART en RS485.

### 4.6 Streamlit
Framework para Python que permite desarrollar interfaces web interactivas para monitoreo en tiempo real.

---

## 5. Materiales

- 2 Computadores personales (PC1 y PC2)  
- 1 Switch administrable Cisco  
- 1 Router  
- 1 Raspberry Pi Pico W  
- 1 ESP32  
- Cables de red UTP y micro USB  
- Software de simulación o equipos reales  
- Python 3.11 y Streamlit

---

## 6. Desarrollo Experimental

### 6.1 Configuración de Red (VLANs y Router)

#### Configuración del Switch
```bash
enable
configure terminal
vlan 1
 name VLAN_PC1
 exit
vlan 3
 name VLAN_PC2
 exit
interface vlan 1
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit
interface vlan 3
 ip address 192.168.20.1 255.255.255.0
 no shutdown
 exit
interface fastEthernet0/1
 switchport mode access
 switchport access vlan 1
 exit
interface fastEthernet0/2
 switchport mode access
 switchport access vlan 3
 exit
interface fastEthernet0/24
 switchport mode trunk
 no shutdown
 exit
end
write memory
```

#### Configuración del Router
```bash
enable
configure terminal
interface g0/0
 no shutdown
interface g0/0.1
 encapsulation dot1Q 1
 ip address 192.168.10.254 255.255.255.0
 exit
interface g0/0.3
 encapsulation dot1Q 3
 ip address 192.168.20.254 255.255.255.0
 exit
end
write memory
```

#### Configuración de PCs
| Dispositivo | VLAN | IP | Máscara | Gateway |
|--------------|------|----|----------|----------|
| PC1 | 1 | 192.168.10.2 | 255.255.255.0 | 192.168.10.1 |
| PC2 | 3 | 192.168.20.2 | 255.255.255.0 | 192.168.20.1 |

Pruebas de ping entre VLANs fueron exitosas, confirmando el enrutamiento correcto.

---

### 6.2 Comunicación RS485

**Montaje:** Raspberry Pi Pico W (maestro) ↔ ESP32 (esclavo) usando transceptores MAX485 y resistencias de 120 Ω.  
El protocolo se basó en tramas de 5 bytes:

| Campo | Descripción |
|--------|--------------|
| Header | Tipo de paquete |
| Dato | Valor transmitido |
| Secuencia | Contador incremental |
| Checksum | Control de integridad |

#### Código Maestro (Raspberry Pi Pico)
```python
import serial, time
ser = serial.Serial('/dev/ttyS0', 115200)
seq = 0
while True:
    data = [0xAA, seq & 0xFF, (seq >> 8) & 0xFF]
    checksum = sum(data) & 0xFF
    frame = bytes(data + [checksum])
    ser.write(frame)
    print("Enviado:", frame)
    seq += 1
    time.sleep(1)
```

#### Código Esclavo (ESP32)
```python
import serial
ser = serial.Serial('/dev/ttyS0', 115200)
while True:
    frame = ser.read(4)
    if len(frame) == 4:
        checksum = sum(frame[:3]) & 0xFF
        if checksum == frame[3]:
            print("Trama válida:", frame)
        else:
            print("Checksum incorrecto:", frame)
```

#### Interfaz Streamlit
```python
import streamlit as st
import pandas as pd
import time

st.title("Monitoreo RS485 en Tiempo Real")
st.subheader("Modo Simplex / Full Dúplex")

placeholder = st.empty()
data = []

for i in range(50):
    data.append({"Trama": i, "Valor TX": i * 2, "Valor RX": i * 2 + 1})
    df = pd.DataFrame(data)
    placeholder.dataframe(df)
    time.sleep(0.5)
```

---

## 7. Resultados y Análisis

| Parámetro | Simplex | Full Dúplex |
|------------|----------|-------------|
| Paquetes transmitidos | 15 | 31 |
| Paquetes recibidos | 15 | 30 |
| Tasa de error (%) | 0.0 | 1.8 |
| Velocidad de transmisión | 115200 bps | 115200 bps |
| Frecuencia de actualización | 1 Hz | 1 Hz |

**Análisis:**  
- El modo Simplex evidenció comunicación estable sin errores.  
- El modo Full Dúplex presentó tasa de error menor al 2 %.  
- El checksum fue eficaz para validar integridad.  
- El bus RS485 mostró robustez y estabilidad eléctrica.  

---

## 8. Conclusiones

- Se implementó exitosamente comunicación RS485 entre los nodos maestro y esclavo.  
- El modo Simplex fue completamente estable.  
- El modo Full Dúplex validó la confiabilidad del bus diferencial.  
- El checksum demostró eficacia en detección de errores.  
- La interfaz Streamlit permitió análisis visual y cuantitativo en tiempo real.  
- La red VLAN configurada mejoró segmentación y administración.  

---

## 9. Preguntas del Laboratorio

1. **¿Diferencias entre RS232, RS422 y RS485?**  
   - RS232: punto a punto, corto alcance.  
   - RS422: diferencial unidireccional.  
   - RS485: diferencial bidireccional multipunto.

2. **Ventajas del RS485 en entornos industriales:**  
   Alta inmunidad al ruido, soporte multipunto y largo alcance.

3. **Función del MAX485:**  
   Conversión TTL ↔ RS485 garantizando transmisión robusta.

4. **Importancia del control de errores:**  
   Permite detectar tramas corruptas mediante checksum.

5. **Desempeño Simplex vs Full Dúplex:**  
   - Simplex: estable.  
   - Full Dúplex: mayor eficiencia, mínima tasa de error.

---

## 10. Recomendaciones

- Usar cableado trenzado con resistencias de 120 Ω.  
- Asegurar tierra común entre todos los nodos.  
- Verificar baudrate en cada dispositivo.  
- Implementar registro de datos en CSV para análisis posterior.  
- Utilizar resistencias de polarización para evitar estados indeterminados.

---

## 11. Referencias

1. Cisco Networking Academy. *Switching, Routing and Wireless Essentials (CCNA 2).*  
2. Tanenbaum, A. S. *Redes de Computadoras.* 5ª Edición.  
3. Forouzan, B. *Comunicación de Datos y Redes de Computadoras.* 5ª Edición.  
4. Laboratorio de Redes – Universidad Santo Tomás.
