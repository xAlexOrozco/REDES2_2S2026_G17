# Integrante 1 – Red cableada + Routing

- Calcular el direccionamiento y subnetting VLSM/FLSM.
- Definir subredes, gateways y rangos de IP.
- Configurar las VLAN:
    - ADMIN
    - COCINA
    - WEB_SERVERS
    - DHCP_SERVERS
- Configurar la infraestructura cableada del Piso 1.
- Configurar el enrutamiento entre los diferentes segmentos.
- Configurar OSPF o EIGRP según corresponda por nuestro número de grupo.
- Configurar el enlace LACP entre Piso 1 <-> Piso 2.
- Hacer pruebas de conectividad relacionadas con esta parte.

# Integrante 2 – WiFi + DHCP + parte de integración

- Configurar las redes inalámbricas del Piso 2.
- Configurar las redes inalámbricas del Piso 3.
- Configurar SSID, WPA2 y contraseñas.
- Configurar DHCP de los routers inalámbricos del Piso 2.
- Configurar DHCP de los routers inalámbricos del Piso 3.
- Configurar el DHCP Server para las VLAN cableadas.
- Configurar los pools DHCP necesarios.
- Probar que dispositivos nuevos puedan conectarse y obtener IP tanto en Piso 2 como en Piso 3.
- Ayudar con la integración y pruebas generales de la red.

# Integrante 3 – Servicios + HSRP + Documentación

- Configurar el servidor Web.
- Configurar DNS con el dominio solicitado.
- Crear la página web estática con los datos de los integrantes y número de grupo.
- Configurar HSRP en Piso 1 y Servicios Centrales según la topología.
- Probar el failover de HSRP cuando falle el router activo.
- Configurar los enlaces LACP restantes:
    - Piso 3 <-> Piso 1
    - Datacenter <-> Piso 1
- Encargarse de toda la documentación de la práctica y del README, incluyendo:
    - Topología.
    - Subnetting VLSM/FLSM.
    - Direccionamiento.
    - VLANs.
    - DHCP.
    - WiFi.
    - OSPF/EIGRP.
    - LACP.
    - HSRP.
    - DNS y HTTP.
    - Comandos utilizados.
    - Pruebas realizadas.

- Mantener actualizado el README conforme vayamos avanzando.


---

# Practica 2



## Configuraciones de VLAN

```
Numero de Grupo = 17
X = 1 + 7 = 8
```

| VLAN         | Numero  |
| ---          | ---     |
| ADMIN        | 10+8=18 |
| COCINA       | 20+8=28 |
| WEB_SERVERS  | 30+8=38 |
| DHCP_SERVERS | 40+8=48 |


- Crear Vlans en los Switches
```bash
enable

conf terminal
vlan 18
name ADMIN
vlan 28
name COCINA
vlan 38
name WEB_SERVERS
vlan 48
name DHCP_SERVERS

end
```

- Asignar enlaces a las Vlans
```bash
# En SW1
conf terminal

interface Fa 0/1
switchport mode access
switchport access vlan 38

interface Fa 0/2
switchport mode access
switchport access vlan 48

interface range Gig 0/1-2
switchport mode trunk
switchport trunk allowed vlan all

end

# En SW2
conf terminal

interface Fa 0/1
switchport mode access
switchport access vlan 18

interface Fa 0/2
switchport mode access
switchport access vlan 28

interface range Gig 0/1-2
switchport mode trunk
switchport trunk allowed vlan all

end
```

---
## Direccionamiento y subredes, ***usando VLSM: $2^n - 2$***


### Piso 1 `192.198.18.0/24`

1. Subred Cocina:

    - 60 hos requeridos: $n = 6$ ($2^6 - 2 = 62$ hosts útiles).
    - Máscara de subred: $/26$ (255.255.255.192).
    - Dirección de Red: `192.198.18.0/26`
    - Gateway (Primera IP usable): `192.198.18.1`
    - Rango de IP usables: `192.198.18.1 – 192.198.18.62`
    - Broadcast: `192.198.18.63`

2. Subred Administración

    - 10 hosts requeridos: $n = 4$ ($2^4 - 2 = 14$ hosts útiles).
    - Máscara de subred: $/28$ (255.255.255.240).
    - Dirección de Red: `192.198.18.64/28`
    - Gateway (Primera IP usable): `192.198.18.65`
    - Rango de IP usables: `192.198.18.65 – 192.198.18.78`
    - Broadcast: `192.198.18.79`

---
### Piso 2 `192.198.28.0/24`

- 80 hosts por WLAN: $n = 7$ bits de host ($2^7 - 2 = 126$ hosts útiles por subred)

Dividimos la red en 2 partes iguales:

1. Subred WLAN1

    - Máscara de subred: $/25$ (255.255.255.128).
    - Dirección de Red: `192.198.28.0/25`
    - Gateway (Primera IP usable): `192.198.28.1`
    - Rango de IP usables: `192.198.28.1 – 192.198.28.126`
    - Broadcast: `192.198.28.127`

2. Subred WLAN2

    - Máscara de subred: $/25$ (255.255.255.128).
    - Dirección de Red: `192.198.28.128/25`
    - Gateway (Primera IP usable): `192.198.28.129`
    - Rango de IP usables: `192.198.28.129 – 192.198.28.254`
    - Broadcast: `192.198.28.255`

---
### Piso 3 `192.198.38.0/24`

- 80 hosts por WLAN: $n = 7$ bits de host ($2^7 - 2 = 126$ hosts útiles por subred)

Dividimos la red en 2 partes iguales:

1. Subred WLAN1

    - Máscara de subred: $/25$ (255.255.255.128).
    - Dirección de Red: `192.198.38.0/25`
    - Gateway (Primera IP usable): `192.198.38.1`
    - Rango de IP usables: `192.198.38.1 – 192.198.38.126`
    - Broadcast: `192.198.38.127`

2. Subred WLAN2

    - Máscara de subred: $/25$ (255.255.255.128).
    - Dirección de Red: `192.198.38.128/25`
    - Gateway (Primera IP usable): `192.198.38.129`
    - Rango de IP usables: `192.198.38.129 – 192.198.38.254`
    - Broadcast: `192.198.38.255`

---
### Servicios Centrales `192.198.100.0/24`

1. VLAN WEB_SERVERS

    - Máscara de subred: $/25$ (255.255.255.128).
    - Dirección de Red: `192.198.100.0/25`
    - Gateway (Primera IP usable): `192.198.100.1`
    - IP Estática ServerWeb: `192.198.100.2`
    - Rango de IP usables: `192.198.100.1 – 192.198.100.126`
    - Broadcast: `192.198.100.127`

2. VLAN DHCP_SERVERS

    - Máscara de subred: $/25$ (255.255.255.128).
    - Dirección de Red: `192.198.100.128/25`
    - Gateway (Primera IP usable): `192.198.100.129`
    - IP Estática ServerDHCP: `192.198.100.130`
    - Rango de IP usables: `192.198.100.129 – 192.198.100.254`
    - Broadcast: `192.198.100.255`

---
## Enrutamiento — Subredes Punto a Punto


| Enlace Punto a Punto | Descripción del Enlace | Subred /30 | IPs Usables |
| --- | --- | --- | --- |
| Enlace 1 | Multilayer Switch0 - Multilayer Switch1 (Port-Channel 1) | 10.2.8.0/30 | 10.2.8.1 – 10.2.8.2 | 
| Enlace 2 | Multilayer Switch1 - Multilayer Switch2 (Port-Channel 2) | 10.2.8.4/30 | 10.2.8.5 – 10.2.8.6 | 
| Enlace 3 | Multilayer Switch1 - Multilayer Switch3 (Port-Channel 3) | 10.2.8.8/30 | 10.2.8.9 – 10.2.8.10 |
| Enlace 4 | Multilayer Switch1 - R3 | 10.2.8.12/30 | 10.2.8.13 – 10.2.8.14 |
| Enlace 5 | Multilayer Switch1 - R4 | 10.2.8.16/30 | 10.2.8.17 – 10.2.8.18 |
| Enlace 6 | Multilayer Switch3 - R1 | 10.2.8.8/30 | 10.2.8.9 – 10.2.8.10 |
| Enlace 7 | Multilayer Switch1 - R1 | 10.2.8.20/30 | 10.2.8.21 – 10.2.8.22 |
| Enlace 8 | Multilayer Switch1 - R2 | 10.2.8.24/30 | 10.2.8.25 – 10.2.8.26 |

## Pools para el Servidor DHCP

| Nombre del Pool DHCP | Red / Máscara | Default Gateway | Rango DHCP Asignable |
| ---                  | ---           | ---             | ---                  |
| POOL_COCINA   | 192.198.18.0 / 255.255.255.192   | 192.198.18.1   | 192.198.18.2 – 192.198.18.62    |
| POOL_ADMIN    | 192.198.18.64 / 255.255.255.240  | 192.198.18.65  | 192.198.18.66 – 192.198.18.78   |
| POOL_P2_WLAN1 | 192.198.28.0 / 255.255.255.128   | 192.198.28.1   | 192.198.28.2 – 192.198.28.126   |
| POOL_P2_WLAN2 | 192.198.28.128 / 255.255.255.128 | 192.198.28.129	| 192.198.28.130 – 192.198.28.254 |
| POOL_P3_WLAN1 | 192.198.38.0 / 255.255.255.128   | 192.198.38.1   | 192.198.38.2 – 192.198.38.126   |
| POOL_P3_WLAN2 | 192.198.38.128 / 255.255.255.128 | 192.198.38.129	| 192.198.38.130 – 192.198.38.254 |


---
## Configurar enlaces LACP + IP Routing

```bash
# En Multilayer Switch0
enable
configure terminal
ip routing

interface port-channel 1
no switchport
ip address 10.2.8.1 255.255.255.252
exit

interface range fa0/1-4
no switchport
channel-group 1 mode active
exit

end

# En Multilayer Switch1
enable
configure terminal
ip routing

interface port-channel 1
no switchport
ip address 10.2.8.2 255.255.255.252
exit

interface range fa0/1-4
no switchport
channel-group 1 mode active
exit

interface port-channel 2
no switchport
ip address 10.2.8.5 255.255.255.252
exit

interface range fa0/5-8
no switchport
channel-group 2 mode active
exit

interface port-channel 3
no switchport
ip address 10.2.8.9 255.255.255.252
exit

interface range fa0/11-14
no switchport
channel-group 3 mode active
exit

end

# En Multilayer Switch2
enable
configure terminal
ip routing

interface port-channel 2
no switchport
ip address 10.2.8.6 255.255.255.252
exit

interface range fa0/1-4
no switchport
channel-group 2 mode active
exit

end

# En Multilayer Switch3
enable
configure terminal
ip routing

interface port-channel 3
no switchport
ip address 10.2.8.10 255.255.255.252
exit

interface range fa0/1-4
no switchport
channel-group 3 mode active
exit

end
```

---
## Configurar enlaces MAN a red cableada

- En Multiplayer Switch1
```bash
enable
configure terminal

interface Fa0/9
no switchport
ip address 10.2.8.13 255.255.255.252
no shutdown
exit


interface Fa0/10
no switchport
ip address 10.2.8.17 255.255.255.252
no shutdown
exit

end
```

- En Multiplayer Switch3
```bash
enable
configure terminal

interface GigabitEthernet0/1
no switchport
ip address 10.2.8.21 255.255.255.252
no shutdown
exit

interface GigabitEthernet0/2
no switchport
ip address 10.2.8.25 255.255.255.252
no shutdown
exit

end
```

---
## Configuración EIGRP en red MAN

```bash
# En Multilayer Switch0
enable
configure terminal
router eigrp 8
network 10.2.8.0 0.0.0.3
no auto-summary
exit
end

# En Multilayer Switch1
enable
configure terminal
router eigrp 8
network 10.2.8.0 0.0.0.3
network 10.2.8.4 0.0.0.3
network 10.2.8.8 0.0.0.3
network 10.2.8.12 0.0.0.3
network 10.2.8.16 0.0.0.3
no auto-summary
exit
end

# En Multilayer Switch2
enable
configure terminal
router eigrp 8
network 10.2.8.4 0.0.0.3
no auto-summary
exit
end

# En Multilayer Switch3
enable
configure terminal
router eigrp 8
network 10.2.8.8 0.0.0.3
network 10.2.8.20 0.0.0.3
network 10.2.8.24 0.0.0.3
no auto-summary
exit
end
```

## Configuracion Punto a Punto, Router-on-a-Stick con HSRP en Routers

- En R1
```bash
enable
configure terminal

interface Gig0/0
ip address 10.2.8.22 255.255.255.252
no shutdown
exit

interface Gig0/1
no shutdown
exit

interface Gig0/1.38
encapsulation dot1Q 38
ip address 192.198.100.3 255.255.255.128
standby 38 ip 192.198.100.1
standby 38 priority 110
standby 38 preempt
exit

interface Gig0/1.48
encapsulation dot1Q 48
ip address 192.198.100.131 255.255.255.128
standby 48 ip 192.198.100.129
standby 48 preempt
exit

router eigrp 8
network 10.2.8.20 0.0.0.3
network 192.198.100.0 0.0.0.127
network 192.198.100.128 0.0.0.127
no auto-summary
exit
```

- En R2
```bash
enable
configure terminal

interface Gig0/0
ip address 10.2.8.26 255.255.255.252
no shutdown
exit

interface Gig0/1
no shutdown
exit

interface Gig0/1.38
encapsulation dot1Q 38
ip address 192.198.100.4 255.255.255.128
standby 38 ip 192.198.100.1
standby 38 preempt
exit

interface Gig0/1.48
encapsulation dot1Q 48
ip address 192.198.100.132 255.255.255.128
standby 48 ip 192.198.100.129
standby 48 priority 110
standby 48 preempt
exit

router eigrp 8
network 10.2.8.24 0.0.0.3
network 192.198.100.0 0.0.0.127
network 192.198.100.128 0.0.0.127
no auto-summary
exit
```

- En R3
```bash
enable
configure terminal

interface Gig0/0
ip address 10.2.8.14 255.255.255.252
no shutdown
exit

interface Gig0/1
no shutdown
exit

interface Gig0/1.18
encapsulation dot1Q 18
ip address 192.198.18.66 255.255.255.240
standby 18 ip 192.198.18.65
standby 18 preempt
exit

interface Gig0/1.28
encapsulation dot1Q 28
ip address 192.198.18.2 255.255.255.192
standby 28 ip 192.198.18.1
standby 28 priority 110
standby 28 preempt
exit

router eigrp 8
network 10.2.8.12 0.0.0.3
network 192.198.18.0 0.0.0.63
network 192.198.18.64 0.0.0.15
no auto-summary
exit
```

- En R4
```bash
enable
configure terminal

interface Gig0/0
ip address 10.2.8.18 255.255.255.252
no shutdown
exit

interface Gig0/1
no shutdown
exit

interface Gig0/1.18
encapsulation dot1Q 18
ip address 192.198.18.67 255.255.255.240
standby 18 ip 192.198.18.65
standby 18 priority 110
standby 18 preempt
exit

interface Gig0/1.28
encapsulation dot1Q 28
ip address 192.198.18.3 255.255.255.192
standby 28 ip 192.198.18.1
standby 28 preempt
exit

router eigrp 8
network 10.2.8.16 0.0.0.3
network 192.198.18.0 0.0.0.63
network 192.198.18.64 0.0.0.15
no auto-summary
exit
```

## 