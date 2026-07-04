# VPN Site-to-Site FortiGate

**Estudiante:** Edwin De Paula  
**Matricula:** 2024-2415  
**Institución:** Instituto Tecnológico de las Américas (ITLA)  
**Asignatura:** Seguridad en Redes

---

## Video

| Recurso | URL |
|---|---|
| Video YouTube | _pendiente_ |

---

## Objetivo

Implementar una VPN Site-to-Site entre dos firewalls FortiGate a través de un router ISP, estableciendo un túnel IPSec cifrado que permita la comunicación segura entre las redes LAN de ambos sitios. La configuración se realiza y verifica mediante la interfaz gráfica (GUI) de FortiGate, y la conectividad se prueba mediante traceroute entre las redes LAN.

---

## Topología

![Topología del laboratorio](docs/screenshots/topology.png)

| Dispositivo | Interfaz | Dirección IP | Descripción |
|---|---|---|---|
| FortiGate-A | port1 | 192.168.24.1/24 | LAN Site A |
| FortiGate-A | port2 | 10.24.15.1/30 | WAN hacia ISP |
| FortiGate-A | port3 | 192.168.10.200/24 | Management (vmnet8) |
| ISP | Ethernet0/0 | 10.24.15.2/30 | WAN hacia FortiGate-A |
| ISP | Ethernet0/1 | 10.24.15.5/30 | WAN hacia FortiGate-B |
| FortiGate-B | port2 | 10.24.15.6/30 | WAN hacia ISP |
| FortiGate-B | port1 | 192.168.15.1/24 | LAN Site B |
| PC-LAN-A | eth0 | 192.168.24.10/24 | Gateway: 192.168.24.1 |
| PC-LAN-B | eth0 | 192.168.15.10/24 | Gateway: 192.168.15.1 |
| Kali Linux | eth0 | 192.168.10.250/24 | Acceso a GUI de FortiGate-A |

---

## Parámetros de Configuración

### Fase 1 - IPSec

| Parámetro | Valor |
|---|---|
| Nombre | VPN-2415 |
| Interfaz WAN | port2 |
| Propuesta | des-sha256 |
| Grupo Diffie-Hellman | 14 (2048 bits) |
| Remote Gateway A | 10.24.15.6 |
| Remote Gateway B | 10.24.15.1 |
| Pre-shared Key | Edwin2024 |

### Fase 2 - IPSec

| Parámetro | Valor |
|---|---|
| Nombre | VPN-2415-P2 |
| Propuesta | des-sha256 |
| Grupo DH | 14 |
| Subred origen A | 192.168.24.0/24 |
| Subred destino A | 192.168.15.0/24 |
| Subred origen B | 192.168.15.0/24 |
| Subred destino B | 192.168.24.0/24 |

### Rutas Estáticas

| Dispositivo | Destino | Via |
|---|---|---|
| FortiGate-A | 0.0.0.0/0 | 10.24.15.2 por port2 |
| FortiGate-A | 192.168.15.0/24 | VPN-2415 |
| FortiGate-B | 0.0.0.0/0 | 10.24.15.5 por port2 |
| FortiGate-B | 192.168.24.0/24 | VPN-2415 |

### Políticas de Firewall

| Política | Origen | Destino | Acción |
|---|---|---|---|
| LAN-to-VPN | port1 | VPN-2415 | Accept |
| VPN-to-LAN | VPN-2415 | port1 | Accept |

---

## Explicación de la Configuración

### VPN Site-to-Site en FortiGate

FortiGate implementa VPN IPSec usando interfaces virtuales de túnel. La Fase 1 define los parámetros de negociación IKE entre los dos peers, y la Fase 2 define qué tráfico se cifra y con qué algoritmos. FortiGate crea automáticamente una interfaz virtual para el túnel que se usa en políticas de firewall y rutas estáticas.

### Acceso a la GUI de FortiGate

Para la administración gráfica, se configuró `port3` en FortiGate-A con la IP `192.168.10.200` en la red vmnet8, accesible desde Kali Linux mediante `http://192.168.10.200`. Esto permite demostrar la configuración y estado del túnel a través de la interfaz gráfica oficial de FortiGate.

### Políticas de Firewall

Las políticas de firewall en FortiGate son obligatorias para permitir el tráfico. Se configuraron dos políticas simétricas — una para tráfico saliente de la LAN hacia el túnel, y otra para tráfico entrante del túnel hacia la LAN.

### Flujo de Negociación

1. FortiGate-A inicia negociación IKE Fase 1 hacia 10.24.15.6
2. Se negocia el canal ISAKMP con des-sha256 y grupo 14
3. Se establece la SA de Fase 2 con los selectores de subred
4. El tráfico de 192.168.24.0/24 hacia 192.168.15.0/24 fluye cifrado
5. Las políticas de firewall permiten el tráfico en ambas direcciones

---

## Verificación mediante GUI

### Dashboard de FortiGate-A

![Dashboard de FortiGate-A](docs/screenshots/gui-dashboard.png)

Vista general del dashboard de administración de FortiGate-A accesible desde el navegador mediante `http://192.168.10.200`.

### Túnel VPN Activo

![VPN → IPsec Tunnels](docs/screenshots/gui-tunnel.png)

La sección VPN → IPsec Tunnels muestra el túnel VPN-2415 activo con el peer remoto 10.24.15.6.

### Políticas de Firewall

![Policy & Objects → Firewall Policy](docs/screenshots/gui-policy.png)

Las políticas LAN-to-VPN y VPN-to-LAN configuradas y activas en FortiGate-A.

### Interfaces de Red

![Network → Interfaces](docs/screenshots/gui-interfaces.png)

Las interfaces port1, port2 y port3 con sus direcciones IP configuradas.

---

## Verificación mediante CLI

### Ping entre LANs

```
execute ping-options source 192.168.24.1
execute ping 192.168.15.1
```

![Ping entre LANs](docs/screenshots/ping-test.png)

100% de success rate confirma la comunicación cifrada entre las redes LAN de ambos sitios.

### Traceroute entre LANs

```
execute traceroute-options source 192.168.24.1
execute traceroute 192.168.15.1
```

![Traceroute entre LANs](docs/screenshots/traceroute.png)

Un único salto directo hacia FortiGate-B confirma que el tráfico viaja a través del túnel VPN.

---

## Archivos del Repositorio

```
fortigate-site-to-site/
├── configs/
│   ├── FortiGate-A.txt
│   ├── FortiGate-B.txt
│   └── ISP.txt
├── docs/
│   └── screenshots/
│       ├── topology.png
│       ├── gui-dashboard.png
│       ├── gui-tunnel.png
│       ├── gui-policy.png
│       ├── gui-interfaces.png
│       ├── ping-test.png
│       └── traceroute.png
└── README.md
```

---

## Herramientas Utilizadas

- PNetLab — Plataforma de emulación de red
- FortiGate VM64 KVM v6.46 build1879 — Firewall Fortinet
- Cisco IOSv 15.4(2)T4 — Router ISP
- Kali Linux — Acceso a GUI de FortiGate
- VMware Workstation — Virtualización del servidor PNetLab
