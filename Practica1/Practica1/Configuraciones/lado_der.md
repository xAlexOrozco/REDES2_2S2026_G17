# Práctica 1 - Redes de Computadoras 2
# Manual Técnico - Edificio Derecho

## Universidad de San Carlos de Guatemala
Facultad de Ingeniería

Curso: Redes de Computadoras 2

Grupo: 17

Número de grupo utilizado: **8**

---

# Configuración General

## VLANs utilizadas

| VLAN | Nombre | Red |
|------|---------|----------------|
|68|Primaria|192.178.68.0/24|
|78|Básicos|192.178.78.0/24|
|88|Bachillerato|192.178.88.0/24|

---

# Dominio VTP

```
Dominio: G17
```

---

# Switch VTP Server

Hostname

```bash
hostname SW1_G17
```

Contraseña

```bash
enable secret redes2grupo17
```

Configuración VTP

```bash
vtp domain G17
vtp mode server
```

Creación de VLANs

```bash
vlan 68
 name Primaria

vlan 78
 name Basicos

vlan 88
 name Bachillerato
```

---

# Switches Clientes

En todos los switches clientes

Hostname

```bash
hostname SW1_G17
```

(Modificar el número según corresponda)

Configuración

```bash
vtp domain G17
vtp mode client
```

---

# Configuración de Troncales

Todos los enlaces entre switches fueron configurados como Trunk.

Ejemplo

```bash
interface FastEthernet0/1
 switchport mode trunk
```

La misma configuración se aplicó a todos los enlaces Switch-Switch.

---

# Configuración de Puertos Access

## VLAN Primaria

```bash
interface FastEthernet0/3

switchport mode access

switchport access vlan 68
```

---

## VLAN Básicos

```bash
interface FastEthernet0/3

switchport mode access

switchport access vlan 78
```

---

## VLAN Bachillerato

```bash
interface FastEthernet0/3

switchport mode access

switchport access vlan 88
```

---

# Seguridad (Port Security)

La seguridad únicamente fue aplicada a los puertos pertenecientes a la VLAN Básicos.

Configuración

```bash
interface FastEthernet0/3

switchport mode access

switchport access vlan 78

switchport port-security

switchport port-security maximum 1

switchport port-security mac-address sticky

switchport port-security violation shutdown
```

Características

- Máximo una dirección MAC.
- Aprendizaje automático mediante Sticky MAC.
- Si aparece otra MAC el puerto entra en estado Shutdown.

---

# Rapid PVST

Todos los switches fueron configurados con Rapid PVST.

```bash
spanning-tree mode rapid-pvst
```

---

# Root Bridge

En el switch principal

```bash
spanning-tree vlan 68 root primary

spanning-tree vlan 78 root primary

spanning-tree vlan 88 root primary
```

---

# Router del Edificio Derecho

## Habilitación de interfaz física

```bash
interface GigabitEthernet0/0

no shutdown
```

---

## Subinterfaz VLAN 68

```bash
interface GigabitEthernet0/0.68

encapsulation dot1Q 68

ip address 192.178.68.1 255.255.255.0
```

---

## Subinterfaz VLAN 78

```bash
interface GigabitEthernet0/0.78

encapsulation dot1Q 78

ip address 192.178.78.1 255.255.255.0
```

---

## Subinterfaz VLAN 88

```bash
interface GigabitEthernet0/0.88

encapsulation dot1Q 88

ip address 192.178.88.1 255.255.255.0
```

---

# Configuración OSPF

Proceso

```bash
router ospf 1
```

Redes anunciadas

```bash
network 10.10.9.0 0.0.0.255 area 0

network 192.178.68.0 0.0.0.255 area 0

network 192.178.78.0 0.0.0.255 area 0

network 192.178.88.0 0.0.0.255 area 0
```

---

# Direccionamiento IP

## VLAN Primaria

Gateway

```
192.178.68.1
```

Ejemplo

|Equipo|Dirección|
|------|---------|
|PC6|192.178.68.10|

---

## VLAN Básicos

Gateway

```
192.178.78.1
```

Ejemplo

|Equipo|Dirección|
|------|---------|
|PC7|192.178.78.10|
|Laptop2|192.178.78.20|

---

## VLAN Bachillerato

Gateway

```
192.178.88.1
```

Ejemplo

|Equipo|Dirección|
|------|---------|
|PC3|192.178.88.10|
|PC8|192.178.88.20|

---

# Configuración de las PCs

## PC6

```
IP: 192.178.68.10
Máscara: 255.255.255.0
Gateway: 192.178.68.1
```

---

## PC7

```
IP: 192.178.78.10
Máscara: 255.255.255.0
Gateway: 192.178.78.1
```

---

## Laptop2

```
IP: 192.178.78.20
Máscara: 255.255.255.0
Gateway: 192.178.78.1
```

---

## PC3

```
IP: 192.178.88.10
Máscara: 255.255.255.0
Gateway: 192.178.88.1
```

---

## PC8

```
IP: 192.178.88.20
Máscara: 255.255.255.0
Gateway: 192.178.88.1
```

---

# Comandos de Verificación

## Switches

```bash
show vlan brief

show interfaces trunk

show spanning-tree

show running-config

show vtp status
```

---

## Router

```bash
show ip interface brief

show ip route

show ip ospf neighbor

show ip protocols

show running-config
```

---

## PCs

```text
ipconfig

ping <IP destino>

tracert <IP destino>
```

---

# Resultado esperado

- Comunicación entre equipos de la misma VLAN.
- Comunicación entre VLANs.
- Comunicación entre ambos edificios.
- VLANs propagadas mediante VTP.
- Rapid PVST funcionando.
- OSPF anunciando correctamente las redes.
- Port Security activo únicamente en la VLAN Básicos.