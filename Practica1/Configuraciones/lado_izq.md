## Configuración VTP (lado izq)

- Switch12 (Modo Server)
```bash
enable
configure terminal
vtp version 2
vtp mode server
vtp domain G17
vtp password redes2grupo17
end
```

- Switch1-11 (Modo Client)
```bash
enable
configure terminal
vtp version 2
vtp mode client
vtp domain G17
vtp password redes2grupo17
end
```

## Configuración VLAN en Switch12

```bash
configure terminal
vlan 18
name Primaria
vlan 28
name Basicos
vlan 38
name Bachillerato
end
```

## Establecer enlaces TRUNK para propagar VLANS

- Switch12
```bash
interface range fa0/1-10
switchport mode trunk
switchport trunk allowed vlan all
```

- Switch1
```bash
interface range fa0/1-4
switchport mode trunk
switchport trunk allowed vlan all
```

- Switch2
```bash
interface range fa0/1-3
switchport mode trunk
switchport trunk allowed vlan all
```

- Switch3
```bash
interface range fa0/1-2
switchport mode trunk
switchport trunk allowed vlan all
```

- Switch4
```bash
interface range fa0/1-3
switchport mode trunk
switchport trunk allowed vlan all
```

- Switch5
```bash
interface range fa0/1-4
switchport mode trunk
switchport trunk allowed vlan all
```

- Switch6
```bash
interface range fa0/1-4
switchport mode trunk
switchport trunk allowed vlan all
```

- Switch7
```bash
interface range fa0/1-2
switchport mode trunk
switchport trunk allowed vlan all
```

- Switch8
```bash
interface range fa0/1-2
switchport mode trunk
switchport trunk allowed vlan all
```

- Switch9
```bash
interface fa0/1
switchport mode trunk
switchport trunk allowed vlan all
```

- Switch10
```bash
interface range fa0/1-2
switchport mode trunk
switchport trunk allowed vlan all
```

- Switch11
```bash
interface range fa0/1-2
switchport mode trunk
switchport trunk allowed vlan all
```

## Configuración de interfaces a dispositivos

- Switch7
```bash
interface fa0/3
switchport mode access
switchport access vlan 28
```

- Switch8
```bash
interface fa0/3
switchport mode access
switchport access vlan 38
```

- Switch9
```bash
interface fa0/2
switchport mode access
switchport access vlan 38
```

- Switch10
```bash
interface fa0/3
switchport mode access
switchport access vlan 18
```

- Switch11
```bash
interface fa0/3
switchport mode access
switchport access vlan 18
```


## Configuración STP (lado izq - PVST)

- En todos los Switches
```bash
enable 
conf terminal
spanning-tree mode pvst
```

- Root Bridge Principal = Switch12
```bash
enable 
conf terminal
spanning-tree vlan 18,28,38 root primary
```

- Root Bridge Secundario = Switch 1
```bash
enable 
conf terminal
spanning-tree vlan 18,28,38 root secondary
```

- Optimizar los puertos de acceso
```bash
# Switch7-8-10-11
interface fa0/3
spanning-tree portfast 
spanning-tree bpduguard enable

# Switch9
interface fa0/2
spanning-tree portfast 
spanning-tree bpduguard enable
```

## Configuración Router Izquierdo (R-IZQ)

```bash
enable
configure terminal
interface g0/1
no shutdown 
exit 
interface g0/1.18 
encapsulation dot1Q 18
ip address 192.178.18.1 255.255.255.0
exit
interface g0/1.28
encapsulation dot1Q 28
ip address 192.178.28.1 255.255.255.0
exit
interface g0/1.38
encapsulation dot1Q 38
ip address 192.178.38.1 255.255.255.0
exit
```

## Configuración ruteo dinámico

- R_IZQ
```bash
interface GigabitEthernet 0/0
ip address 10.10.18.1 255.255.255.0
no shutdown
exit

router ospf 1
router-id 1.1.1.1
network 10.10.18.0 0.0.0.255 area 0
network 192.178.18.0 0.0.0.255 area 0
network 192.178.28.0 0.0.0.255 area 0
network 192.178.38.0 0.0.0.255 area 0
```

- R1
```bash
# LADO IZQUIERDO
enable
configure terminal
interface GigabitEthernet0/1
ip address 10.10.18.2 255.255.255.0
no shutdown
exit

router ospf 1
router-id 2.2.2.2
network 10.10.18.0 0.0.0.255 area 0 
redistribute rip subnets 
exit

# LADO DERECHO
interface GigabitEthernet0/0
ip address 10.10.19.1 255.255.255.0
no shutdown
exit

router rip
version 2
no auto-summary
network 10.10.19.0
redistribute ospf 1 metric 5
exit
```

- R2
```bash
# LADO IZQUIERDO
enable
configure terminal
interface GigabitEthernet0/1
ip address 10.10.19.2 255.255.255.0
no shutdown
exit

router rip
version 2
no auto-summary
network 10.10.19.0
redistribute eigrp 100 metric 5 
exit

# LADO DERECHO
interface GigabitEthernet0/0
ip address 10.10.20.1 255.255.255.0
no shutdown
exit

router eigrp 100
network 10.10.20.0 0.0.0.255
no auto-summary
redistribute rip metric 10000 100 255 1 1500
exit
```

- R_DER
```bash
enable
configure terminal
interface GigabitEthernet0/1
ip address 10.10.20.2 255.255.255.0
no shutdown
exit

router eigrp 100
network 10.10.20.0 0.0.0.255
network 192.178.68.0 0.0.0.255
network 192.178.78.0 0.0.0.255
network 192.178.88.0 0.0.0.255
no auto-summary
exit
```

## Direcciones IP (Lado izq)

| Equipo | VLAN | Dirección IP | Gateway |
| --- | --- | --- | --- |
| Basicos1 | 28 | 192.178.28.10 | 192.178.28.1 |
| Bachillerato1 | 38 | 192.178.38.10 | 192.178.38.1 |
| Bachillerato2 | 38 | 192.178.38.11 | 192.178.38.1 |
| Primaria1 | 18 | 192.178.18.10 | 192.178.18.1 |
| Primaria2 | 18 | 192.178.18.11 | 192.178.18.1 |

## Direcciones IP (Lado der)

| Equipo | VLAN | Dirección IP | Gateway |
| --- | --- | --- | --- |
| Basicos2 | 78 | 192.178.78.10 | 192.178.78.1 |
| Basicos3 | 78 | 192.178.78.10 | 192.178.78.1 |
| Bachillerato3 | 68 | 192.178.68.10 | 192.178.68.1 |
| Primaria3 | 88 | 192.178.88.10 | 192.178.88.1 |
| Primaria4 | 88 | 192.178.88.11 | 192.178.88.1 |


