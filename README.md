Laboratorio 4: Comunicaciones Industriales
Universidad Santo Tomás
Facultad de Ingeniería Electrónica

📋 Descripción General
Este repositorio documenta el diseño, implementación y validación de una red de comunicaciones que integra:

VLANs en un switch de capa 2.

Comunicación RS485 entre dispositivos embebidos (Raspberry Pi Pico W y ESP32).

Visualización en tiempo real mediante Streamlit.

El objetivo principal es comparar el rendimiento entre los modos de comunicación Simplex y Full Dúplex en un entorno industrial.

🎯 Objetivos
Objetivo General
Diseñar, implementar y validar una red de comunicaciones integral combinando infraestructura de red computarizada (VLANs) y sistemas de comunicación industrial (RS485).

Objetivos Específicos
Crear y configurar VLANs en un switch de capa 2.

Asignar direcciones IP a las interfaces VLAN.

Configurar un router para enrutamiento entre subredes.

Implementar comunicación RS485 en modo Simplex y Full Dúplex.

Comparar rendimiento entre modos (tasa de transmisión, errores, estabilidad).

Visualizar datos en tiempo real usando Streamlit.

🛠 Materiales Utilizados
2 Computadores (PC1 y PC2).

1 Switch administrable Cisco.

1 Router.

1 Raspberry Pi Pico W.

1 ESP32.

Transceptores MAX485.

Cables de red UTP, cables micro USB, resistencias de 120 Ω.

Software: Cisco Packet Tracer, Python 3.11, Streamlit.

⚙️ Configuración de la Red VLAN
Topología
PC1 → VLAN 1 → IP: 192.168.10.2/24

PC2 → VLAN 3 → IP: 192.168.20.2/24

Switch: Interfaces VLAN con IPs 192.168.10.1 y 192.168.20.1.

Router: Configurado para enrutamiento inter-VLAN.

Configuración del Switch
bash
Switch> enable
Switch# configure terminal

! Creación de VLANs
Switch(config)# vlan 1
Switch(config-vlan)# name VLAN_PC1
Switch(config-vlan)# exit

Switch(config)# vlan 3
Switch(config-vlan)# name VLAN_PC2
Switch(config-vlan)# exit

! Asignación de IPs a interfaces VLAN
Switch(config)# interface vlan 1
Switch(config-if)# ip address 192.168.10.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit

Switch(config)# interface vlan 3
Switch(config-if)# ip address 192.168.20.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit

! Asignación de puertos a VLANs
Switch(config)# interface fastEthernet0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 1
Switch(config-if)# exit

Switch(config)# interface fastEthernet0/2
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 3
Switch(config-if)# exit

! Puerto trunk hacia el router
Switch(config)# interface fastEthernet0/24
Switch(config-if)# switchport mode trunk
Switch(config-if)# no shutdown
Switch(config-if)# exit

Switch# write memory
Configuración del Router
bash
Router> enable
Router# configure terminal
Router(config)# interface g0/0
Router(config-if)# no shutdown
Router(config)# interface g0/0.1
Router(config-subif)# encapsulation dot1Q 1
Router(config-subif)# ip address 192.168.10.254 255.255.255.0
Router(config-subif)# exit
Router(config)# interface g0/0.3
Router(config-subif)# encapsulation dot1Q 3
Router(config-subif)# ip address 192.168.20.254 255.255.255.0
Router(config-subif)# exit
Router# write memory
Verificación
Comando: show ip interface brief en switch y router.

Prueba de conectividad: ping 192.168.20.2 desde PC1.

🔌 Comunicación RS485
Esquema de Conexión
Función	Pico W (Maestro)	MAX485 (Maestro)	MAX485 (Esclavo)	ESP32 (Esclavo)
TXD	→ DI			
RXD	← RO			
Línea A		A → A		
Línea B		B → B		
GND	GND	GND	GND	GND
Alimentación	3.3V → VCC	3.3V → VCC	3.3V → VCC	3.3V
Protocolo de Comunicación
Estructura de trama (5 bytes):

Header (1 byte): Tipo de paquete.

Dato (2 bytes): Valor simulado.

Secuencia (1 byte): Número de trama.

Checksum (1 byte): Suma de verificación.

Implementación
Maestro (Pico W): Envía tramas periódicas, calcula checksum, supervisa respuestas.

Esclavo (ESP32): Recibe tramas, verifica checksum, responde con ACK o error.

Pruebas Realizadas
Modo Simplex: Comunicación unidireccional estable, sin errores.

Modo Full Dúplex: Comunicación bidireccional con tasa de error < 2%.

Errores forzados: Verificación de detección de checksum incorrecto.

📊 Visualización con Streamlit
Interfaz de Monitoreo
Gráficos en tiempo real de datos TX/RX.

Métricas de rendimiento: tramas transmitidas, recibidas, tasa de error.

Tabla de últimos paquetes intercambiados.

Ejecución
bash
streamlit run monitor_rs485.py
📈 Resultados y Análisis
Parámetro	Modo Simplex	Modo Full Dúplex
Paquetes transmitidos	15	31
Paquetes recibidos	15	30
Tasa de error (%)	0.0	1.8
Velocidad	115200 bps	115200 bps
Conclusiones
✅ Comunicación RS485 implementada exitosamente.

✅ Modo Simplex: Comunicación estable sin errores.

✅ Modo Full Dúplex: Tasa de error mínima (< 2%).

✅ Checksum efectivo para detección de errores.

✅ Streamlit: Herramienta útil para monitoreo en tiempo real.

❓ Preguntas del Laboratorio
¿Diferencia entre RS232, RS422 y RS485?

RS232: Punto a punto, corta distancia.

RS422: Diferencial unidireccional.

RS485: Diferencial bidireccional multipunto.

¿Ventajas de RS485 en entornos industriales?

Inmunidad al ruido, multidrop, larga distancia.

¿Función de los transceptores MAX485?

Conversión de señales TTL a RS485.

¿Por qué usar control de errores?

Detección de tramas corruptas.

¿Comparación entre Simplex y Full Dúplex?

Simplex: Mayor estabilidad.

Full Dúplex: Mayor eficiencia con posible ligera tasa de error.

📁 Estructura del Repositorio
text
Lab4-Comunicaciones-Industriales/
│
├── README.md
├── configs/
│   ├── switch_config.txt
│   ├── router_config.txt
│   └── pico_esp32_code/
│       ├ maestro_pico.py
│       └ esclavo_esp32.py
├── streamlit/
│   └── monitor_rs485.py
├── imagenes/
│   ├── topologia.png
│   ├── esquema_rs485.png
│   └── streamlit_ui.png
└── referencias/
    └── bibliografia.md
👥 Autores
Ferney Arturo Amaya Gómez

David Esteban Díaz Castro

Jonny Alejandro Mejia Leon

Docente: Ing. Diego Alejandro Barragán Vargas
Fecha: Octubre 2025

📚 Referencias
Cisco Networking Academy. Switching, Routing and Wireless Essentials.

Tanenbaum, A. S. Redes de Computadoras.

Forouzan, B. Comunicación de Datos y Redes de Computadoras.

Laboratorio de Redes – Universidad Santo Tomás.
