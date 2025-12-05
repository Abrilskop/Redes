# 📄 DOCUMENTACIÓN FINAL: Laboratorio RIP - Ejemplo 1

**Tema:** Enrutamiento Dinámico utilizando el protocolo RIP v1.  
**Escenario:** Red triangular con 3 Routers (Sede1, Sede2, Sede3) conectados vía Serial y 3 redes LAN conectadas vía FastEthernet.

---

## 1. Tabla de Direccionamiento (Configuración Final)

Esta tabla refleja la configuración corregida utilizando el puerto **FastEthernet0/1** para las LANs.

| Dispositivo | Hostname | Interfaz LAN (Gateway) | Interfaz Serial (DCE/Maestro) | Interfaz Serial (DTE/Esclavo) |
| :--- | :--- | :--- | :--- | :--- |
| **Router0** | `Sede1` | **Fa0/1**: 192.168.0.1 | **Se0/1/0**: 192.168.10.1 (Clock) | **Se0/1/1**: 192.168.30.2 |
| **Router1** | `Sede2` | **Fa0/1**: 192.168.1.1 | **Se0/1/0**: 192.168.20.1 (Clock) | **Se0/1/1**: 192.168.10.2 |
| **Router2** | `Sede3` | **Fa0/1**: 192.168.2.1 | **Se0/1/0**: 192.168.30.1 (Clock) | **Se0/1/1**: 192.168.20.2 |

> **Nota:** Los puertos DCE (Maestros) llevan el comando `clock rate 64000`.

---

## 2. Scripts de Configuración de Routers

Copia y pega estos bloques en la CLI de cada router (modo privilegiado `#`).

### 🖧 Router 0: Sede1
```cisco
enable
configure terminal
hostname Sede1

! Configuración LAN (Puerto Fa0/1)
interface FastEthernet0/1
 ip address 192.168.0.1 255.255.255.0
 no shutdown
 exit

! Enlace Serial hacia Sede2 (Maestro - Lleva Reloj)
interface Serial0/1/0
 ip address 192.168.10.1 255.255.255.0
 clock rate 64000
 bandwidth 64000
 no shutdown
 exit

! Enlace Serial hacia Sede3 (Esclavo)
interface Serial0/1/1
 ip address 192.168.30.2 255.255.255.0
 no shutdown
 exit

! Configuración RIP
router rip
 network 192.168.0.0
 network 192.168.10.0
 network 192.168.30.0
 end
write
```
### 🖧 Router 1: Sede2
```cisco
enable
configure terminal
hostname Sede2

! Configuración LAN (Puerto Fa0/1)
interface FastEthernet0/1
 ip address 192.168.1.1 255.255.255.0
 no shutdown
 exit

! Enlace Serial hacia Sede3 (Maestro - Lleva Reloj)
interface Serial0/1/0
 ip address 192.168.20.1 255.255.255.0
 clock rate 64000
 bandwidth 64000
 no shutdown
 exit

! Enlace Serial hacia Sede1 (Esclavo)
interface Serial0/1/1
 ip address 192.168.10.2 255.255.255.0
 no shutdown
 exit

! Configuración RIP
router rip
 network 192.168.1.0
 network 192.168.10.0
 network 192.168.20.0
 end
write
```

### 🖧 Router 2: Sede3
```cisco
enable
configure terminal
hostname Sede3

! Configuración LAN (Puerto Fa0/1)
interface FastEthernet0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
 exit

! Enlace Serial hacia Sede1 (Maestro - Lleva Reloj)
interface Serial0/1/0
 ip address 192.168.30.1 255.255.255.0
 clock rate 64000
 bandwidth 64000
 no shutdown
 exit

! Enlace Serial hacia Sede2 (Esclavo)
interface Serial0/1/1
 ip address 192.168.20.2 255.255.255.0
 no shutdown
 exit

! Configuración RIP
router rip
 network 192.168.2.0
 network 192.168.20.0
 network 192.168.30.0
 end
write
```