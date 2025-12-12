# 🌐 Topología OSPF + RIP — Cisco Packet Tracer

## 1. Descripción General

La topología está compuesta por **dos dominios de enrutamiento**:

### 🔵 Dominio OSPF (Área 0)
- Routers: **R0, R1, R2**
- Topología: Triángulo (FastEthernet)
- Hosts:
  - **PC0** conectado a R0
  - **PC1** conectado a R1

### 🟣 Dominio RIP
- Routers: **R2, R3, R4**
- Topología: Lineal (Serial)
- Host:
  - **PC2** conectado a R4

### 🔑 Router Frontera
- **R2** ejecuta **OSPF y RIP**
- Realiza **redistribución entre protocolos**

---

## 2. Esquema de Direccionamiento IP

### 2.1 LANs (PCs)

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway |
|------------|----------|--------------|---------|---------|
| PC0 | Fa0 | 192.168.100.10 | 255.255.255.0 | 192.168.100.1 |
| PC1 | Fa0 | 192.168.101.10 | 255.255.255.0 | 192.168.101.1 |
| PC2 | Fa0 | 192.168.200.10 | 255.255.255.0 | 192.168.200.1 |

---

### 2.2 Enlaces OSPF (FastEthernet)

| Enlace | Red | Router | IP |
|------|-----|--------|----|
| R0–R1 | 192.168.10.0/24 | R0 | 192.168.10.1 |
|  |  | R1 | 192.168.10.2 |
| R0–R2 | 192.168.20.0/24 | R0 | 192.168.20.1 |
|  |  | R2 | 192.168.20.2 |
| R1–R2 | 192.168.30.0/24 | R1 | 192.168.30.1 |
|  |  | R2 | 192.168.30.2 |

---

### 2.3 Enlaces RIP (Serial)

| Enlace | Red | Router | Interfaz | IP |
|------|------|--------|----------|----|
| R2–R3 | 192.168.40.0/30 | R2 | Se0/1/0 | 192.168.40.1 |
|  |  | R3 | Se0/1/0 | 192.168.40.2 |
| R3–R4 | 192.168.50.0/30 | R3 | Se0/1/1 | 192.168.50.1 |
|  |  | R4 | Se0/1/1 | 192.168.50.2 |

---

### 2.4 Loopbacks (Router-ID)

| Router | Loopback0 |
|-------|-----------|
| R0 | 10.0.0.1/32 |
| R1 | 10.0.0.2/32 |
| R2 | 10.0.0.3/32 |
| R3 | 10.0.0.4/32 |
| R4 | 10.0.0.5/32 |

---

## 3. Configuración de Routers

---

### 3.1 R0 — OSPF

```bash
enable
configure terminal
hostname R0

interface Loopback0
ip address 10.0.0.1 255.255.255.255
exit

interface FastEthernet0/0
ip address 192.168.20.1 255.255.255.0
no shutdown
exit

interface FastEthernet0/1
ip address 192.168.10.1 255.255.255.0
no shutdown
exit

interface Vlan10
ip address 192.168.100.1 255.255.255.0
no shutdown
exit

interface FastEthernet0/1/0
switchport mode access
switchport access vlan 10
no shutdown
exit

router ospf 1
router-id 10.0.0.1
network 10.0.0.1 0.0.0.0 area 0
network 192.168.10.0 0.0.0.255 area 0
network 192.168.20.0 0.0.0.255 area 0
network 192.168.100.0 0.0.0.255 area 0
end
write memory
```

### 3.2 R1 — OSPF

```bash
enable
configure terminal
hostname R1

interface Loopback0
ip address 10.0.0.2 255.255.255.255
exit

interface FastEthernet0/0
ip address 192.168.30.1 255.255.255.0
no shutdown
exit

interface FastEthernet0/1
ip address 192.168.10.2 255.255.255.0
no shutdown
exit

interface Vlan11
ip address 192.168.101.1 255.255.255.0
no shutdown
exit

interface FastEthernet0/1/0
switchport mode access
switchport access vlan 11
no shutdown
exit

router ospf 1
router-id 10.0.0.2
network 10.0.0.2 0.0.0.0 area 0
network 192.168.10.0 0.0.0.255 area 0
network 192.168.30.0 0.0.0.255 area 0
network 192.168.101.0 0.0.0.255 area 0
end
write memory

```

### 3.3 R2 — OSPF + RIP

```bash
enable
configure terminal
hostname R2

interface Loopback0
ip address 10.0.0.3 255.255.255.255
exit

interface FastEthernet0/0
ip address 192.168.20.2 255.255.255.0
no shutdown
exit

interface FastEthernet0/1
ip address 192.168.30.2 255.255.255.0
no shutdown
exit

interface Serial0/1/0
ip address 192.168.40.1 255.255.255.252
clock rate 64000
no shutdown
exit

router ospf 1
router-id 10.0.0.3
network 10.0.0.3 0.0.0.0 area 0
network 192.168.20.0 0.0.0.255 area 0
network 192.168.30.0 0.0.0.255 area 0
redistribute rip subnets
exit

router rip
version 2
no auto-summary
network 192.168.40.0
redistribute ospf 1 metric 5
end
write memory

```

### 3.4 R3 — RIP

```bash
enable
configure terminal
hostname R3

interface Loopback0
ip address 10.0.0.4 255.255.255.255
exit

interface Serial0/1/0
ip address 192.168.40.2 255.255.255.252
no shutdown
exit

interface Serial0/1/1
ip address 192.168.50.1 255.255.255.252
clock rate 64000
no shutdown
exit

router rip
version 2
no auto-summary
network 10.0.0.0
network 192.168.40.0
network 192.168.50.0
end
write memory

```

### 3.5 R4 — RIP

```bash
enable
configure terminal
hostname R4

interface Loopback0
ip address 10.0.0.5 255.255.255.255
exit

interface Serial0/1/1
ip address 192.168.50.2 255.255.255.252
no shutdown
exit

interface FastEthernet0/0
ip address 192.168.200.1 255.255.255.0
no shutdown
exit

router rip
version 2
no auto-summary
network 10.0.0.0
network 192.168.50.0
network 192.168.200.0
end
write memory

```

### Verificación

```bash
show ip route
show ip ospf neighbor
show ip rip database
ping 192.168.200.10

```