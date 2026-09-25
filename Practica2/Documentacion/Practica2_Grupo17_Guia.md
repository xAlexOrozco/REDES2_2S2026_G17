# Práctica 2 - Restaurante | Grupo 17

> **Revisión documental del 25/09/2026:** consultar el [README de entrega](../README.md), las [pruebas paso a paso](../Documentacion/Pruebas_Integrante3.md) y los [hallazgos/correcciones](../Documentacion/Revision_y_Correcciones.md). Se detectó solapamiento entre el /27 de tránsito P2 y los /30 de P3, y capacidad de solo 59 clientes en COCINA al reservar HSRP. Las marcas de completado y resultados de este archivo son reportes históricos del grupo; no constituyen una validación nueva del PKT.

Guía técnica consolidada para documentar y defender la implementación realizada en Cisco Packet Tracer.

> **Importante:** este README de guía refleja la configuración que quedó implementada durante el desarrollo. El README inicial tenía algunos nombres/mapas de interfaces que quedaron desactualizados; para documentar se debe tomar como referencia la configuración actual del `.pkt` y esta guía consolidada.

---

## 1. Datos generales

- **Curso:** Ingeniería en Ciencias y Sistemas
- **Práctica:** 2 - Restaurante
- **Grupo:** 17
- **X:** `1 + 7 = 8`
- **Protocolo de routing:** EIGRP 8, porque el grupo 17 es impar.
- **Archivo esperado:** `Practica2_17.pkt`
- **Dominio web:** `www.practica2_Grupo17.com`

La práctica exige VLANs, VLSM/FLSM, WiFi segura, DHCP, DNS, HTTP, HSRP y LACP; además se valida conectividad cableada/inalámbrica, DHCP y servicios.

---

# 2. VLANs

| VLAN | Nombre | Uso |
|---:|---|---|
| 18 | ADMIN | Administración |
| 28 | COCINA | Cocina |
| 38 | WEB_SERVERS | Servidor Web / DNS |
| 48 | DHCP_SERVERS | Servidor DHCP |

## Crear VLANs

```cisco
enable
configure terminal
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

## SW1 - Servicios Centrales

```cisco
configure terminal
interface fa0/1
 switchport mode access
 switchport access vlan 38
 exit

interface fa0/2
 switchport mode access
 switchport access vlan 48
 exit

interface range gigabitEthernet0/1-2
 switchport mode trunk
 switchport trunk allowed vlan all
 exit
end
```

## SW2 - Piso 1 cableado

```cisco
configure terminal
interface fa0/1
 switchport mode access
 switchport access vlan 18
 exit

interface fa0/2
 switchport mode access
 switchport access vlan 28
 exit

interface range gigabitEthernet0/1-2
 switchport mode trunk
 switchport trunk allowed vlan all
 exit
end
```

> El acceso final de Piso 1 quedó: `SW2 Fa0/1 = ADMIN` y `SW2 Fa0/2 = COCINA`. Los trunks van hacia los routers R3/R4.

---

# 3. Direccionamiento y subnetting

## 3.1 Piso 1 - VLSM

Red general: `192.198.18.0/24`

### COCINA - VLAN 28

- Red: `192.198.18.0/26`
- Máscara: `255.255.255.192`
- Gateway HSRP: `192.198.18.1`
- Usables: `192.198.18.1 - 192.198.18.62`
- Broadcast: `192.198.18.63`

### ADMIN - VLAN 18

- Red: `192.198.18.64/28`
- Máscara: `255.255.255.240`
- Gateway HSRP: `192.198.18.65`
- Usables: `192.198.18.65 - 192.198.18.78`
- Broadcast: `192.198.18.79`

---

## 3.2 Piso 2 - FLSM /25

Red general: `192.198.28.0/24`

### WLAN1

- Red: `192.198.28.0/25`
- Gateway: `192.198.28.1`
- DHCP: `192.198.28.2 - 192.198.28.126`
- Broadcast: `192.198.28.127`

### WLAN2

- Red: `192.198.28.128/25`
- Gateway: `192.198.28.129`
- DHCP: `192.198.28.130 - 192.198.28.254`
- Broadcast: `192.198.28.255`

---

## 3.3 Piso 3 - FLSM /25

Red general: `192.198.38.0/24`

### WLAN1

- Red: `192.198.38.0/25`
- Gateway: `192.198.38.1`
- DHCP: `192.198.38.2 - 192.198.38.126`
- Broadcast: `192.198.38.127`

### WLAN2

- Red: `192.198.38.128/25`
- Gateway: `192.198.38.129`
- DHCP: `192.198.38.130 - 192.198.38.254`
- Broadcast: `192.198.38.255`

---

## 3.4 Servicios Centrales - FLSM /25

### WEB_SERVERS - VLAN 38

- Red: `192.198.100.0/25`
- Gateway HSRP: `192.198.100.1`
- ServerWeb: `192.198.100.2`
- Broadcast: `192.198.100.127`

### DHCP_SERVERS - VLAN 48

- Red: `192.198.100.128/25`
- Gateway HSRP: `192.198.100.129`
- ServerDHCP: `192.198.100.130`
- Broadcast: `192.198.100.255`

---

# 4. Red de enrutamiento 10.2.8.0/24

## Enlaces MAN principales

| Enlace | Dispositivo A | IP A | Dispositivo B | IP B | Red |
|---|---|---:|---|---:|---|
| Po1 | MS4 | 10.2.8.1 | MS2 | 10.2.8.2 | 10.2.8.0/30 |
| Po2 | MS2 | 10.2.8.5 | MS3 | 10.2.8.6 | 10.2.8.4/30 |
| Po3 | MS2 | 10.2.8.9 | MS1 | 10.2.8.10 | 10.2.8.8/30 |
| MAN-R3 | MS2 Fa0/9 | 10.2.8.13 | R3 Gi0/0 | 10.2.8.14 | 10.2.8.12/30 |
| MAN-R4 | MS2 Fa0/10 | 10.2.8.17 | R4 Gi0/0 | 10.2.8.18 | 10.2.8.16/30 |
| MAN-R1 | MS1 Gi0/1 | 10.2.8.21 | R1 Gi0/0 | 10.2.8.22 | 10.2.8.20/30 |
| MAN-R2 | MS1 Gi0/2 | 10.2.8.25 | R2 Gi0/0 | 10.2.8.26 | 10.2.8.24/30 |

> En el `.pkt` actual, los enlaces de Piso 3 hacia MS4 usan `10.2.8.36/30` y `10.2.8.40/30`, y los WRT funcionan como routers con una interfaz WAN estática.

---

# 5. Topología actual resumida

```text
                              ┌──────────────┐
                              │     MS4      │
                              │ Po1 10.2.8.1 │
                              └──────┬───────┘
                               LACP 4x
                                    │
                                    │
                              10.2.8.2
                              ┌──────┴───────┐
                              │     MS2      │
                              │     CORE     │
                              └──┬────┬───┬──┘
                       Po2 4x │    │    │ Po3 4x
                              │    │    │
                            MS3   R3   R4   MS1
                             │
                       Piso 2 WRTs

MS4 Fa0/5 10.2.8.37/30 ── WAN P3_R1 10.2.8.38
MS4 Fa0/6 10.2.8.41/30 ── WAN P3_R3 10.2.8.42

MS3:
  VLAN 100 / 10.2.8.33/27
  ├── P2_R1 WAN 10.2.8.34
  └── P2_R2 WAN 10.2.8.35

R3/R4:
  Piso 1 + HSRP VLAN 18 y 28

R1/R2:
  Servicios Centrales + HSRP VLAN 38 y 48
```

> **Nota de nomenclatura:** algunos nombres del README inicial corresponden a una versión anterior del mapa. Para documentar el estado final, usar los nombres e interfaces mostrados en esta sección y validar con `show cdp neighbors` / `show ip interface brief`.

---

# 6. LACP / EtherChannel

La práctica exige 4 interfaces por enlace agregado.

## MS4 - MS2

### MS4

```cisco
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
```

### MS2

```cisco
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
end
```

## MS2 - MS3

### MS2

```cisco
configure terminal
interface port-channel 2
 no switchport
 ip address 10.2.8.5 255.255.255.252
 exit

interface range fa0/5-8
 no switchport
 channel-group 2 mode active
 exit
end
```

### MS3

```cisco
configure terminal
interface port-channel 2
 no switchport
 ip address 10.2.8.6 255.255.255.252
 exit

interface range fa0/1-4
 no switchport
 channel-group 2 mode active
 exit
end
```

## MS2 - MS1

### MS2

```cisco
configure terminal
interface port-channel 3
 no switchport
 ip address 10.2.8.9 255.255.255.252
 exit

interface range fa0/11-14
 no switchport
 channel-group 3 mode active
 exit
end
```

### MS1

```cisco
configure terminal
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

## Verificación LACP

```cisco
show etherchannel summary
```

Estado esperado:

- `RU` en los Port-Channel.
- `LACP` como protocolo.
- Miembros con bandera `P`.

---

# 7. EIGRP 8

El grupo 17 es impar, por lo que se utilizó EIGRP 8.

## MS4

```cisco
enable
configure terminal
router eigrp 8
 network 10.2.8.0 0.0.0.3
 network 10.2.8.36 0.0.0.3
 network 10.2.8.40 0.0.0.3
 no auto-summary
exit
end
```

Después de crear las rutas estáticas de Piso 3:

```cisco
configure terminal
ip route 192.198.38.0 255.255.255.128 10.2.8.38
ip route 192.198.38.128 255.255.255.128 10.2.8.42

router eigrp 8
 redistribute static
 exit
end
```

## MS2

```cisco
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
```

## MS3

```cisco
enable
configure terminal
router eigrp 8
 network 10.2.8.4 0.0.0.3
 network 10.2.8.32 0.0.0.3
 no auto-summary
exit
end
```

Para las WLAN de Piso 2 se utilizaron rutas estáticas y redistribución:

```cisco
configure terminal
ip route 192.198.28.0 255.255.255.128 10.2.8.34
ip route 192.198.28.128 255.255.255.128 10.2.8.35

router eigrp 8
 redistribute static
 exit
end
```

> Durante el desarrollo apareció una línea antigua `network 10.2.8.28 0.0.0.3` en MS3. No forma parte del diseño final de Piso 2 y debe retirarse si todavía está presente.

## MS1

```cisco
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

## Routers R1/R2/R3/R4

Los cuatro routers participan en EIGRP 8 anunciando sus enlaces de routing y las redes directamente conectadas a sus subinterfaces.

Ejemplo de verificación:

```cisco
show ip eigrp neighbors
show ip route eigrp
show ip protocols
```

---

# 8. Piso 1 - HSRP y Router-on-a-Stick

## R3 - Piso 1

```cisco
enable
configure terminal

interface gigabitEthernet0/0
 ip address 10.2.8.14 255.255.255.252
 no shutdown
 exit

interface gigabitEthernet0/1
 no shutdown
 exit

interface gigabitEthernet0/1.18
 encapsulation dot1Q 18
 ip address 192.198.18.66 255.255.255.240
 standby 18 ip 192.198.18.65
 standby 18 preempt
 exit

interface gigabitEthernet0/1.28
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
end
```

## R4 - Piso 1

```cisco
enable
configure terminal

interface gigabitEthernet0/0
 ip address 10.2.8.18 255.255.255.252
 no shutdown
 exit

interface gigabitEthernet0/1
 no shutdown
 exit

interface gigabitEthernet0/1.18
 encapsulation dot1Q 18
 ip address 192.198.18.67 255.255.255.240
 standby 18 ip 192.198.18.65
 standby 18 priority 110
 standby 18 preempt
 exit

interface gigabitEthernet0/1.28
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
end
```

### HSRP final esperado - Piso 1

| VLAN | VIP | R3 | R4 |
|---:|---:|---|---|
| 18 | 192.198.18.65 | Standby | Active |
| 28 | 192.198.18.1 | Active | Standby |

---

# 9. Servicios Centrales - HSRP

## R1

```cisco
enable
configure terminal

interface gigabitEthernet0/0
 ip address 10.2.8.22 255.255.255.252
 no shutdown
 exit

interface gigabitEthernet0/1
 no shutdown
 exit

interface gigabitEthernet0/1.38
 encapsulation dot1Q 38
 ip address 192.198.100.3 255.255.255.128
 standby 38 ip 192.198.100.1
 standby 38 priority 110
 standby 38 preempt
 exit

interface gigabitEthernet0/1.48
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
end
```

## R2

```cisco
enable
configure terminal

interface gigabitEthernet0/0
 ip address 10.2.8.26 255.255.255.252
 no shutdown
 exit

interface gigabitEthernet0/1
 no shutdown
 exit

interface gigabitEthernet0/1.38
 encapsulation dot1Q 38
 ip address 192.198.100.4 255.255.255.128
 standby 38 ip 192.198.100.1
 standby 38 preempt
 exit

interface gigabitEthernet0/1.48
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
end
```

### HSRP final esperado - Servicios Centrales

| VLAN | VIP | R1 | R2 |
|---:|---:|---|---|
| 38 | 192.198.100.1 | Active | Standby |
| 48 | 192.198.100.129 | Standby | Active |

---

# 10. Piso 2 - Router inalámbrico R1

## WAN

```text
IP:       10.2.8.34
Mask:     255.255.255.224
Gateway:  10.2.8.33
```

## LAN / DHCP

```text
LAN:      192.198.28.1/25
DHCP:     192.198.28.2 - 192.198.28.126
DNS:      192.198.100.2
Mode:     Router
```

## WiFi

```text
SSID:      PISO2_G8_R1
Security:  WPA2-Personal
Encryption AES
Password:  G8_PISO2
```

---

# 11. Piso 2 - Router inalámbrico R2

## WAN

```text
IP:       10.2.8.35
Mask:     255.255.255.224
Gateway:  10.2.8.33
```

## LAN / DHCP

```text
LAN:      192.198.28.129/25
DHCP:     192.198.28.130 - 192.198.28.254
DNS:      192.198.100.2
Mode:     Router
```

## WiFi

```text
SSID:      PISO2_G8_R2
Security:  WPA2-Personal
Encryption AES
Password:  G8_PISO2
```

### MS3 para Piso 2

```cisco
configure terminal
vlan 100
 name P2_WAN
exit

interface vlan 100
 ip address 10.2.8.33 255.255.255.224
 no shutdown
exit

interface fa0/5
 switchport mode access
 switchport access vlan 100
 no shutdown
exit

interface fa0/6
 switchport mode access
 switchport access vlan 100
 no shutdown
exit
end
```

> Los WRT reciben `10.2.8.34` y `10.2.8.35` como WAN. Se usó una red /27 de tránsito porque el WRT300N de Packet Tracer fue más estable con ese esquema.

---

# 12. Piso 3 - Router inalámbrico R1

## WAN

```text
IP:       10.2.8.38
Mask:     255.255.255.252
Gateway:  10.2.8.37
DNS:      192.198.100.2
```

## LAN / DHCP

```text
LAN:      192.198.38.1/25
DHCP:     192.198.38.2 - 192.198.38.126
DNS:      192.198.100.2
Mode:     Router
```

## WiFi

```text
SSID:      PISO3_G8_R1
Security:  WPA2-Personal
Encryption AES
Password:  G8_PISO3
```

---

# 13. Piso 3 - Router inalámbrico R3

## WAN

```text
IP:       10.2.8.42
Mask:     255.255.255.252
Gateway:  10.2.8.41
DNS:      192.198.100.2
```

## LAN / DHCP

```text
LAN:      192.198.38.129/25
DHCP:     192.198.38.130 - 192.198.38.254
DNS:      192.198.100.2
Mode:     Router
```

## WiFi

```text
SSID:      PISO3_G8_R3
Security:  WPA2-Personal
Encryption AES
Password:  G8_PISO3
```

### MS4 para Piso 3

```cisco
configure terminal
ip route 192.198.38.0 255.255.255.128 10.2.8.38
ip route 192.198.38.128 255.255.255.128 10.2.8.42

router eigrp 8
 redistribute static
exit
end
```

---

# 14. Servidor Web / DNS

## Configuración de red

```text
IP:       192.198.100.2
Mask:     255.255.255.128
Gateway:  192.198.100.1
DNS:      192.198.100.2
```

## DNS

Servicio DNS: **ON**

Registro:

```text
Name:    www.practica2_Grupo17.com
Address: 192.198.100.2
```

## HTTP

Servicio HTTP: **ON**

Página estática: `index.html`

La página final incluye:

- Nombre de la práctica.
- Grupo 17.
- Dominio.
- Integrantes.
- Tecnologías utilizadas: VLAN, VLSM, DHCP, DNS, HTTP, EIGRP, LACP y HSRP.

---

# 15. Servidor DHCP

## Configuración de red

```text
IP:       192.198.100.130
Mask:     255.255.255.128
Gateway:  192.198.100.129
DNS:      192.198.100.2
```

## Pool ADMIN - implementación actual

```text
Pool:             POOL_ADMIN
Network:          192.198.18.64
Mask:             255.255.255.240
Default Gateway:  192.198.18.65
Start IP:         192.198.18.68
Maximum Users:    11
Rango:            192.198.18.68 - 192.198.18.78
DNS:              192.198.100.2
```

Se reservaron `.66` y `.67` para las IP físicas de R3/R4.

## Pool COCINA - implementación actual

```text
Pool:             POOL_COCINA
Network:          192.198.18.0
Mask:             255.255.255.192
Default Gateway:  192.198.18.1
Start IP:         192.198.18.4
Maximum Users:    59
Rango:            192.198.18.4 - 192.198.18.62
DNS:              192.198.100.2
```

Se reservaron `.2` y `.3` para las IP físicas de R3/R4.

> En Packet Tracer el servicio DHCP del servidor se configuró por GUI; por eso no existe un bloque IOS equivalente al de un router.

---

# 16. Pruebas de conectividad realizadas

## Piso 2

Se comprobó DHCP y conectividad de clientes de ambas WLAN.

Ejemplos:

```text
PC5 → 192.198.28.129       OK
PC5 → 192.198.18.68        OK
PC5 → 192.198.18.5         OK
PC5 → 192.198.100.1       OK
PC5 → 192.198.100.129     OK
```

También se conectó un Smartphone nuevo al `PISO2_G8_R2` y obtuvo DHCP, por ejemplo:

```text
IPv4:       192.198.28.133
Mask:       255.255.255.128
Gateway:    192.198.28.129
DNS:        192.198.100.2
```

## Piso 3

Se comprobó en ambos routers:

```text
Cliente P3_R1 → gateway 192.198.38.1       OK
Cliente P3_R1 → 10.2.8.37                 OK
Cliente P3_R1 → 10.2.8.38                 OK
Cliente P3_R1 → 10.2.8.1                  OK
Cliente P3_R1 → 10.2.8.2                  OK
Cliente P3_R1 → Piso 1                     OK
Cliente P3_R1 → Servicios Centrales        OK

Cliente P3_R3 → gateway 192.198.38.129     OK
Cliente P3_R3 → 10.2.8.41                 OK
Cliente P3_R3 → 10.2.8.42                 OK
Cliente P3_R3 → 10.2.8.1                  OK
Cliente P3_R3 → 10.2.8.2                  OK
Cliente P3_R3 → Piso 1                     OK
Cliente P3_R3 → Servicios Centrales        OK
```

## DNS / HTTP

Los clientes de Piso 1, Piso 2 y Piso 3 ya pueden abrir:

```text
http://www.practica2_Grupo17.com
```

La página se muestra correctamente desde las PCs probadas.

---

# 17. Comandos de verificación importantes

## Interfaces

```cisco
show ip interface brief
```

## VLANs

```cisco
show vlan brief
```

## Trunks

```cisco
show interfaces trunk
```

## EtherChannel / LACP

```cisco
show etherchannel summary
```

## EIGRP

```cisco
show ip eigrp neighbors
show ip protocols
show ip route eigrp
```

## Tabla de routing

```cisco
show ip route
```

## Rutas específicas

```cisco
show ip route 192.198.28.0
show ip route 192.198.28.128
show ip route 192.198.38.0
show ip route 192.198.38.128
```

## ARP

```cisco
show arp
```

## HSRP

```cisco
show standby brief
show standby
```

## Configuración completa

```cisco
show running-config
```

## Guardar configuración

```cisco
copy running-config startup-config
```

O:

```cisco
write memory
```

---

# 18. Pruebas que todavía deben ejecutarse antes de entregar

## HSRP failover - prioridad máxima

Ejecutar primero:

```cisco
show standby brief
```

Registrar cuál router está `Active` y cuál `Standby`.

Luego provocar falla controlada del router activo (por ejemplo, apagando una interfaz o el equipo según la estrategia de la defensa), y volver a ejecutar:

```cisco
show standby brief
```

Comprobar que el standby asume la VIP y que un cliente conserva conectividad.

Realizar la prueba en:

1. Piso 1.
2. Servicios Centrales.

## Nuevo dispositivo cableado

Conectar un PC nuevo a VLAN 18 o VLAN 28 y comprobar DHCP.

## Nuevo dispositivo WiFi Piso 2

Conectar un Smartphone nuevo a `PISO2_G8_R1` o `PISO2_G8_R2` y comprobar DHCP.

## Nuevo dispositivo WiFi Piso 3

Conectar un Smartphone nuevo a `PISO3_G8_R1` o `PISO3_G8_R3` y comprobar DHCP.

---

# 19. Estado técnico actual

| Área | Estado |
|---|---|
| VLANs | ✅ |
| VLSM/FLSM | ✅ |
| Routing 10.2.8.x | ✅ |
| EIGRP 8 | ✅ |
| LACP | ✅ |
| Piso 1 cableado | ✅ |
| DHCP cableado | ✅ |
| Piso 2 WLAN1 | ✅ |
| Piso 2 WLAN2 | ✅ |
| Piso 3 WLAN1 | ✅ |
| Piso 3 WLAN2 | ✅ |
| DNS | ✅ |
| HTTP | ✅ |
| Página web | ✅ |
| HSRP configurado | ✅ |
| HSRP failover validado | ⏳ |
| Prueba final de nuevo host cableado | ⏳ |
| Prueba final de nuevo WiFi Piso 2 | ⏳ |
| Prueba final de nuevo WiFi Piso 3 | ⏳ |
| README final | ⏳ |

---

# 20. Evidencias recomendadas para el README

Guardar capturas de:

1. Topología completa.
2. `show vlan brief`.
3. `show interfaces trunk`.
4. `show etherchannel summary`.
5. `show ip eigrp neighbors`.
6. `show ip route`.
7. `show standby brief` antes y después del failover.
8. DHCP de ADMIN.
9. DHCP de COCINA.
10. DHCP de P2_R1.
11. DHCP de P2_R2.
12. DHCP de P3_R1.
13. DHCP de P3_R3.
14. Configuración DNS.
15. Configuración HTTP.
16. Página `www.practica2_Grupo17.com` desde Piso 1.
17. Página desde Piso 2.
18. Página desde Piso 3.
19. Prueba de nuevo dispositivo WiFi Piso 2.
20. Prueba de nuevo dispositivo WiFi Piso 3.

---

# 21. Preguntas que cada integrante debe poder responder

- ¿Por qué el grupo 17 utiliza `X=8`?
- ¿Por qué ADMIN es VLAN 18 y COCINA VLAN 28?
- ¿Por qué Piso 1 usa VLSM y Piso 2/Piso 3 usan /25?
- ¿Por qué el grupo 17 usa EIGRP y no OSPF?
- ¿Qué función cumple LACP?
- ¿Por qué se utilizan cuatro interfaces por Port-Channel?
- ¿Cuál es el gateway de cada WLAN?
- ¿Qué diferencia hay entre IP física y VIP de HSRP?
- ¿Cuál router es Active y cuál Standby en cada VLAN?
- ¿Qué ocurre si falla el router Active?
- ¿Quién entrega DHCP a Piso 2 y Piso 3?
- ¿Quién entrega DHCP a ADMIN y COCINA?
- ¿Qué IP tiene el servidor Web?
- ¿Qué IP tiene el servidor DHCP?
- ¿Cómo funciona la resolución de `www.practica2_Grupo17.com`?
- ¿Qué rutas de Piso 2 y Piso 3 se redistribuyen en EIGRP?
- ¿Qué diferencia hay entre una ruta `D` y `D EX` en `show ip route`?

---

# 22. Entrega

Antes de entregar:

- Guardar el `.pkt`.
- Usar el nombre `Practica2_17.pkt`.
- Mantener la carpeta `Práctica 2` en el repositorio.
- Actualizar el README con topología, subnetting, DHCP, DNS/HTTP, comandos y pruebas.
- Verificar que todos los integrantes sepan explicar la implementación.

---

## Fuente de requisitos

Documento oficial: **Práctica 2 - Restaurante, Segundo Semestre 2026, USAC / Ingeniería en Ciencias y Sistemas**.

Esta guía consolida la configuración realmente implementada en el proyecto y debe contrastarse con el `.pkt` final antes de la entrega.
