# Revisión técnica y correcciones pendientes

Revisión de los dos Markdown y las 23 páginas del PDF, incluida su topología. No se inspeccionó la configuración interna del `.pkt`. No se modificaron dispositivos ni se ejecutaron pruebas de red. Los hallazgos son documentales salvo los cálculos matemáticos, verificados con `ipaddress` de Python.

## A. Solapamiento en tránsito MAN

`10.2.8.32/27` cubre `.32–.63`; contiene completamente `.36/30` y `.40/30`. En la guía pertenecen a dominios L2 diferentes. Un WRT de P2 podría intentar resolver por ARP direcciones de P3 como si estuvieran en su LAN WAN. Las rutas más específicas pueden ocultar algunos síntomas; un ping exitoso no elimina el error.

Revisar MS3/MS4 con `show running-config`, `show ip interface brief`, `show ip route`, y WAN de los cuatro WRT. Si el PKT ya utiliza redes distintas sin solapamiento, corregir solo los documentos con las direcciones reales.

### Propuesta conservando los /30 de Piso 3

Si `10.2.8.64/27` está libre, sustituir el tránsito P2:

| Parámetro | Documentado | Propuesto |
|---|---|---|
| SVI VLAN 100 MS3 | 10.2.8.33/27 | 10.2.8.65/27 |
| WAN P2_R1 | 10.2.8.34/27 | 10.2.8.66/27 |
| WAN P2_R2 | 10.2.8.35/27 | 10.2.8.67/27 |
| Gateway WAN ambos WRT | 10.2.8.33 | 10.2.8.65 |

Guardar copia de configuración antes de intervenir. En MS3, únicamente si coincide con el estado actual:

```cisco
enable
configure terminal
interface vlan 100
 no ip address
 ip address 10.2.8.65 255.255.255.224
 no shutdown
exit
no ip route 192.198.28.0 255.255.255.128 10.2.8.34
no ip route 192.198.28.128 255.255.255.128 10.2.8.35
ip route 192.198.28.0 255.255.255.128 10.2.8.66
ip route 192.198.28.128 255.255.255.128 10.2.8.67
router eigrp 8
 no network 10.2.8.32 0.0.0.3
 network 10.2.8.64 0.0.0.31
 redistribute static
 no auto-summary
end
```

Retirar otras líneas antiguas del tránsito solo si están presentes y no sirven a otra interfaz; mantener el anuncio de Po2. Actualizar WAN, máscara y gateway en **ambos WRT** mediante el método admitido por el auxiliar. Las LAN, SSID y pools P2 no cambian. Hasta terminar ambos extremos habrá interrupción.

Validar desde MS3 `ping 10.2.8.66`, `ping 10.2.8.67`, rutas estáticas y `D EX` en MS2. Repetir conectividad bilateral P2/P1/P3 y DNS/HTTP. Si falla, restaurar las configuraciones guardadas de MS3 y ambos WRT conjuntamente. Actualizar README con el estado realmente aplicado; no dejar tablas antiguas como finales.

## B. COCINA no admite 60 terminales con el /26 actual

62 usables − VIP − IP R3 − IP R4 = 59 terminales. ADMIN dispone de 14 − 3 = 11, suficiente para 10. Aumentar el máximo DHCP de COCINA a 60 sin cambiar la subred entregaría una dirección fuera del espacio permitido; no hacerlo.

Si los 60 hosts son terminales, propuesta dentro del /24 original:

| Segmento | Red / máscara | VIP | R3 | R4 | Pool | Capacidad clientes | Broadcast |
|---|---|---|---|---|---|---:|---|
| COCINA | 192.198.18.0/25 / 255.255.255.128 | .1 | .2 | .3 | .4–.126 | 123 | .127 |
| ADMIN | 192.198.18.128/28 / 255.255.255.240 | .129 | .130 | .131 | .132–.142 | 11 | .143 |

**No basta con cambiar /26 por /25**: ADMIN actual `.64/28` quedaría dentro de COCINA. Plan de intervención en copia del PKT:

1. Exportar configuración de R3/R4 y parámetros DHCP. Desactivar temporalmente los dos pools cableados durante el cambio.
2. En ambos routers, trasladar primero ADMIN a `.128/28`; conservar VLAN/grupo 18, prioridades y preempt. Esto libera la parte que ocupará COCINA.
3. Ampliar COCINA a /25 en ambos routers; conservar VLAN/grupo 28, VIP, IP físicas y prioridades.
4. Actualizar anuncios EIGRP y pools; mantener helper en ambas VLAN de ambos routers.
5. Activar pools, renovar DHCP en todos los clientes cableados y repetir T06, T09, T10 y HSRP.

R3 (el bloque supone exactamente el direccionamiento antiguo de la guía):

```cisco
enable
configure terminal
interface gigabitEthernet0/1.18
 no standby 18 ip 192.198.18.65
 no ip address
 ip address 192.198.18.130 255.255.255.240
 standby 18 ip 192.198.18.129
 ip helper-address 192.198.100.130
exit
end
```

R4: ejecutar el mismo bloque usando IP física `192.198.18.131`. **Completar ADMIN en ambos antes de ampliar COCINA**. Luego, R3:

```cisco
configure terminal
interface gigabitEthernet0/1.28
 ip address 192.198.18.2 255.255.255.128
 ip helper-address 192.198.100.130
exit
router eigrp 8
 no network 192.198.18.0 0.0.0.63
 no network 192.198.18.64 0.0.0.15
 network 192.198.18.0 0.0.0.127
 network 192.198.18.128 0.0.0.15
end
```

R4: repetir el último bloque usando `192.198.18.3` en COCINA. Conservar su anuncio MAN. Si IOS rechaza reemplazar una máscara, retirar la IP de esa subinterfaz y volver a asignarla, durante la intervención controlada.

Pools nuevos: ADMIN inicio `192.198.18.132`, máximo 11, máscara `.240`, gateway `.129`; COCINA inicio `192.198.18.4`, máximo 123 (o un límite de al menos 60 según el diseño final), máscara `.128`, gateway `.1`. DNS en ambos `192.198.100.2`. Eliminar o actualizar los pools anteriores para evitar asignaciones antiguas. Las direcciones `.130/.131` de ADMIN pertenecen a `192.198.18.x`, distintas del servidor DHCP en `192.198.100.130`.

Actualizar todas las pruebas que usan el gateway ADMIN `.65` para usar `.129`; registrar las IP DHCP reales. Reversión: reducir primero COCINA a /26 en ambos routers, después devolver ADMIN a `.64/28`, anuncios y pools originales, y renovar clientes. No restaurar ADMIN dentro de un /25 todavía activo.

Si el auxiliar define que los 60 hosts incluyen infraestructura, registrar esa aclaración y su criterio de capacidad. No afirmar que el /26 admite 60 clientes más HSRP.

## C. Omisiones y coherencia documental

- Relay: verificar R3/R4; el README incluye el bloque de corrección si falta.
- Broadcast SSID: Piso 2 oculto, Piso 3 visible. Conexión manual al SSID oculto es parte de la prueba.
- Contraseña administrativa: distinta de la clave WiFi. Falta evidencia del cumplimiento.
- X WiFi: confirmar 8 frente a 17; dominio siempre `Grupo17`.
- Restricción consola: pendiente aclaración específica para Server-PT y WRT300N. La observación no impide inspeccionar ni probar equipos ya configurados.
- HSRP: no figura tracking de uplink. La prueba principal debe aislar la LAN o simular caída completa; una prueba adicional WAN requiere evaluar tracking y no puede asumirse cubierta.
- Capacidad de infraestructura: HSRP protege gateway; no duplica los servidores, switches de acceso ni el core. No describir la topología como inmune a cualquier falla.
- Nombre del archivo/carpeta y contenido HTML: completar antes de entrega.
- Ponderaciones: resumen y detalle del PDF no coinciden en varias filas; seguir todos los criterios y consultar al auxiliar para la ponderación definitiva.

## D. Resultado de esta revisión

Se verificó matemáticamente el solapamiento y la capacidad de hosts. Se identificaron requisitos y se prepararon manual, comandos, pruebas y bitácora. **No se certifica que la práctica cumpla íntegramente** hasta contrastar configuración real, aplicar lo necesario y obtener evidencias de las pruebas.
