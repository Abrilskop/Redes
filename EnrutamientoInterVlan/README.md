# Laboratorio de Enrutamiento Inter-VLAN: Router on a Stick

Este repositorio contiene la solución y configuración del ejercicio de laboratorio sobre enrutamiento Inter-VLAN, basado en la guía de la **Universidad Andina del Cusco** para la materia de Infraestructura de Redes.

## 🎯 Objetivo de la Práctica

El objetivo principal de este laboratorio es configurar y verificar la comunicación entre dos Redes de Área Local Virtuales (VLANs) distintas utilizando la técnica "Router on a Stick". Se busca permitir que los equipos de la VLAN de Ventas (VLAN 10) puedan comunicarse con los equipos de la VLAN de Gerencia (VLAN 20) a través de un único enlace físico entre un switch y un router.

## Topology

La topología de red implementada en Cisco Packet Tracer es la siguiente:

![Topología de Red](img/topology.png)

*   **Router:** 1 x Cisco 2811 (R01)
*   **Switch:** 1 x Cisco 2950T-24 (sw01)
*   **VLAN 10 (Ventas):** 2 x PC (Red 10.0.0.0/8)
*   **VLAN 20 (Gerencia):** 2 x Laptop (Red 172.60.0.0/16)

## 🛠️ Tecnologías y Herramientas Utilizadas

*   **Simulador:** Cisco Packet Tracer
*   **Software:** Cisco IOS
*   **Protocolos:**
    *   VLANs
    *   IEEE 802.1Q (Encapsulación de Trunking)
    *   Enrutamiento Inter-VLAN
    *   ICMP (para pruebas de conectividad con `ping`)

## ⚙️ Configuración

A continuación se detallan los comandos utilizados para configurar cada dispositivo de la red.

### 1. Configuración del Switch (sw01)

Se crean las VLANs, se asignan los puertos de acceso correspondientes y se configura el puerto `Fa0/1` como un enlace troncal para permitir el tráfico de múltiples VLANs hacia el router.

```cisco
enable
configure terminal

! Crear VLANs
vlan 10
 name Ventas
vlan 20
 name Gerencia
exit

! Asignar puertos a la VLAN 10 (Ventas)
interface range FastEthernet0/2 - 3
 switchport mode access
 switchport access vlan 10
exit

! Asignar puertos a la VLAN 20 (Gerencia)
interface range FastEthernet0/4 - 5
 switchport mode access
 switchport access vlan 20
exit

! Configurar el puerto hacia el router como Trunk
interface FastEthernet0/1
 switchport mode trunk
exit

end
write memory```

### 2. Configuración del Router (R01)

Se crean subinterfaces virtuales, una para cada VLAN. A cada subinterfaz se le asigna la encapsulación 802.1Q correspondiente y una dirección IP que funcionará como la puerta de enlace predeterminada para los dispositivos de esa VLAN.

```cisco
enable
configure terminal

! Encender la interfaz física principal (Paso crucial)
interface FastEthernet0/1
 no shutdown
exit

! Configurar subinterfaz para VLAN 10 (Ventas)
interface FastEthernet0/1.10
 encapsulation dot1Q 10
 ip address 10.0.0.1 255.0.0.0
exit

! Configurar subinterfaz para VLAN 20 (Gerencia)
interface FastEthernet0/1.20
 encapsulation dot1Q 20
 ip address 172.60.0.1 255.255.0.0
exit

end
write memory
```

### 3. Configuración de Equipos Terminales

Cada dispositivo final debe ser configurado con una dirección IP estática, su máscara de subred y, lo más importante, su puerta de enlace predeterminada (Default Gateway), que corresponde a la dirección IP de la subinterfaz del router para su VLAN.

| Dispositivo | VLAN | Dirección IP | Máscara de Subred | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **PC0** | 10 (Ventas) | `10.0.0.2` | `255.0.0.0` | **`10.0.0.1`** |
| **PC1** | 10 (Ventas) | `10.0.0.3` | `255.0.0.0` | **`10.0.0.1`** |
| **Laptop0** | 20 (Gerencia) | `172.60.0.2` | `255.255.0.0` | **`172.60.0.1`** |
| **Laptop1** | 20 (Gerencia) | `172.60.0.3` | `255.255.0.0` | **`172.60.0.1`** |

## ✅ Verificación

Para confirmar que la configuración es correcta y que existe comunicación entre las VLANs, se realiza una prueba de `ping` desde un equipo en la VLAN 10 a un equipo en la VLAN 20.

**Desde el Command Prompt de PC0:**
```
C:\> ping 172.60.0.2

Pinging 172.60.0.2 with 32 bytes of data:
Reply from 172.60.0.2: bytes=32 time=1ms TTL=127
Reply from 172.60.0.2: bytes=32 time=1ms TTL=127
Reply from 172.60.0.2: bytes=32 time=1ms TTL=127
Reply from 172.60.0.2: bytes=32 time=1ms TTL=127

Ping statistics for 172.60.0.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 1ms, Maximum = 1ms, Average = 1ms
```
> **Nota:** El primer paquete de ping puede fallar (`Request timed out`) debido al proceso de resolución de ARP. Los paquetes subsecuentes deben ser exitosos.

## 💡 Puntos Clave y Troubleshooting

1.  **Enlace Router-Switch en Rojo:** Las interfaces de los routers Cisco vienen administrativamente desactivadas (`shutdown`) por defecto. Es indispensable ejecutar el comando `no shutdown` en la interfaz física (`FastEthernet0/1` en este caso) para activar el enlace.
2.  **Ping Falla a Pesar del Enlace Verde:** Si la conectividad falla, el error más común es una configuración incorrecta de la **Default Gateway** en los equipos terminales. Sin esta dirección, un dispositivo no puede enviar paquetes fuera de su propia red local (VLAN).

---