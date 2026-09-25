# Pruebas paso a paso — Integrante 3

Esta guía indica **dónde trabajar, qué ejecutar, qué esperar y qué capturar**. Ninguna prueba de esta lista se ejecutó desde el asistente. Los valores corresponden al diseño original de la guía; si corriges el direccionamiento, actualiza primero el README y sustituye los valores afectados. En particular, ADMIN pasaría de gateway `.65` a `.129` con la propuesta de ampliación de COCINA.

## Preparación

1. Abre `Practica2_G17.pkt` en Cisco Packet Tracer y guarda una copia de trabajo con File → Save As. Conserva el original.
2. Usa modo **Realtime** para verificar conectividad. Simulation se utiliza solo cuando se indique. Espera la convergencia antes de tomar la línea base.
3. Los comandos IOS se ejecutan dentro del router/switch → **CLI**, Enter y `enable`. No son comandos de PowerShell. Para usar consola física: cable Console del RS-232 de una PC al puerto Console del equipo y PC → Desktop → Terminal → parámetros predeterminados. No configures routers/switches por la pestaña Config.
4. Los comandos `ipconfig`, `ping` y `nslookup` se ejecutan en **PC/Laptop → Desktop → Command Prompt**. El navegador está en Desktop → Web Browser. Smartphones no necesariamente ofrecen esos mismos comandos: utiliza una laptop asociada a la misma WLAN para las pruebas de terminal.
5. Guarda salidas en `Documentacion/Evidencias/` y completa su bitácora. Captura nombre del dispositivo, comando, origen/destino, salida y fase de prueba. No usar capturas de ejemplo como resultados reales.
6. El PDF exige consola para configurar (p. 9), aunque pide equipos de servicios/WiFi gestionados por pantallas. Las rutas GUI de esta guía permiten **inspeccionar servicios y operar clientes**; antes de reconfigurar WRT/Server-PT, aclarar el criterio con el auxiliar. No hay comandos IOS equivalentes inventados para ellos.

Orden sugerido: T01–T04 para detectar errores de base; T06–T10 para confirmar servicios; T11–T15 para HSRP; T05 para LACP y T16 para cerrar. No iniciar failover si el ping de referencia ya falla.

## T01 — Inventario y topología

En cada MS1–MS4 y R1–R4:

```cisco
enable
show cdp neighbors
show ip interface brief
show running-config
```

1. Identifica quién es el core (MS2), Piso 2 (MS3), Piso 3 (MS4) y Datacenter (MS1). Los nombres del archivo pueden diferir; confirma mediante vecinos y direcciones.
2. Compara con la imagen oficial en README. Deben existir las tres agregaciones de cuatro enlaces, las dos parejas HSRP, los dos servidores y los cuatro WRT.
3. Guarda captura general con etiquetas de pisos, subredes, VLAN y enlaces. Guarda running-config por equipo como texto, avanzando con espacio si aparece `--More--`.

**Aprobación:** mapa real coincide con la topología exigida y con las tablas finales. Evidencia: `T01-topologia.png`, `T01-R1-running.txt`, etc. La topología del PDF no sustituye esta captura.

## T02 — VLAN y trunks

En **SW1 y SW2**:

```cisco
enable
show vlan brief
show interfaces trunk
show interfaces status
```

1. SW1 Fa0/1 debe pertenecer a VLAN 38 y Fa0/2 a VLAN 48.
2. SW2 Fa0/1 debe pertenecer a VLAN 18 y Fa0/2 a VLAN 28.
3. Gi0/1 y Gi0/2 de cada switch deben transportar sus VLAN hacia ambos routers; revisar las VLAN permitidas y activas en trunk.
4. Si un comando no está soportado por el modelo, usa `show running-config` y `show interfaces` para obtener los mismos datos.

**Aprobación:** VLAN correctas, acceso correcto y ambas rutas trunk funcionales. No esperar VLAN de usuario en los Po MAN: son capa 3.

## T03 — Direccionamiento y capacidad

1. En MS3, revisa IP/máscara de `interface Vlan100`; en MS4, Fa0/5 y Fa0/6; en ambos WRT de Piso 2, WAN.
2. Compara redes completas, no solo IP diferentes. Si P2 sigue en `.32/27` y P3 en `.36/30` y `.40/30`, registra **FALLÓ: solapamiento** y usa el plan de correcciones.
3. Revisa R3/R4 y los pools: COCINA `/26` solo admite 59 terminales con tres direcciones HSRP reservadas. Documenta corrección o aclaración del criterio de hosts.
4. Comprueba que ningún pool incluya VIP, IP física de router, dirección de red o broadcast.
5. Contrasta los cuatro pools inalámbricos con /25 y capacidad para al menos 80 clientes cada uno.

**Aprobación:** subredes no superpuestas entre segmentos distintos, reservas correctas y capacidad suficiente. No es necesario crear 80 equipos por WLAN para comprobar aritmética de capacidad; sí verificar la configuración del pool y la obtención de una concesión.

## T04 — EIGRP y rutas de retorno

En **MS1, MS2, MS3, MS4 y R1–R4**:

```cisco
enable
show ip protocols
show ip eigrp neighbors
show ip route
show ip route eigrp
```

1. Confirmar AS 8 en los participantes y `ip routing` en multilayer.
2. MS2 debe tener vecinos a través de Po1, Po2, Po3 y hacia R3/R4 según el mapa. Puede haber vecinos adicionales por LAN compartidas; no exigir un conteo fijo sin revisar interfaces.
3. En MS3 deben existir rutas hacia `192.198.28.0/25` y `.128/25` por los WRT correctos; en MS4, las equivalentes `192.198.38...`.
4. En MS2 y routers remotos verificar alcance a esas cuatro WLAN y a `192.198.100.0/25`, `.128/25` y redes de Piso 1. Usar `show ip route` completo si la consulta por prefijo no es aceptada.
5. Verificar rutas en ambos sentidos: una ruta de ida no demuestra retorno. En el origen de una ruta estática aparecerá `S`; redistribuida en otro equipo, normalmente `D EX`.

**Aprobación:** vecinos estables y rutas útiles de ida/vuelta. Si falta una ruta, revisar IP/máscara, enlace, AS, `network`, siguiente salto y redistribución antes de probar HSRP.

## T05 — Tres LACP con cuatro miembros y tolerancia a una falla

Primero, en **MS1–MS4**:

```cisco
enable
show etherchannel summary
show ip interface brief
```

Debe indicar LACP, Po de capa 3 en uso (`RU`) y **cuatro miembros `P`** en cada extremo del enlace probado.

| Caso | Extremos | Po | Puerto que apagar en MS2 | Tráfico que debe atravesarlo |
|---|---|---:|---|---|
| Piso 3–core | MS4/MS2 | 1 | Fa0/1 | Cliente P3 → cliente P1 |
| Piso 2–core | MS3/MS2 | 2 | Fa0/5 | Cliente P2 → cliente P1 |
| DC–core | MS1/MS2 | 3 | Fa0/11 | Cliente P1 → 192.198.100.2 |

1. Confirma puertos reales con T01. Ejecuta ping de referencia desde el cliente indicado.
2. En **MS2**, para el primer caso:

```cisco
configure terminal
interface fastEthernet0/1
 shutdown
end
show etherchannel summary
```

3. Repite el ping. Espera el canal operativo con tres miembros agregados. Puede existir pérdida transitoria; registra la observada. Guarda salida en ambos extremos.
4. Restaura **antes de probar el siguiente caso**:

```cisco
configure terminal
interface fastEthernet0/1
 no shutdown
end
show etherchannel summary
```

5. Repite pasos 1–4 sustituyendo Fa0/1 por Fa0/5 y luego Fa0/11. Al finalizar cada caso deben volver los cuatro `P`.

**Aprobación:** LACP correcto en tres enlaces, cuatro miembros iniciales/finales y continuidad con un miembro fuera. No apagar todos los miembros ni combinar esta prueba con HSRP. No atribuir cuatro veces la velocidad a un único flujo por el solo hecho de agregar cuatro interfaces.

## T06 — DHCP y nuevo host cableado

Repetir en ADMIN **y** COCINA para cubrir los dos pools.

1. Inspecciona ServerDHCP → Services → DHCP: inicio, máscara, gateway, DNS, máximo y servicio activo. Compara con README.
2. En R3 y R4, `show running-config`: busca `ip helper-address 192.198.100.130` bajo Gi0/1.18 y Gi0/1.28. Si falta, el bloque correctivo está en README.
3. Agrega una **PC-PT nueva**, conecta FastEthernet0 mediante cobre directo a un puerto libre de SW2. Los siguientes comandos suponen Fa0/3 libre; compruébalo antes para no alterar otro enlace.
4. En SW2 → CLI, asigna el puerto ADMIN:

```cisco
enable
configure terminal
interface fastEthernet0/3
 switchport mode access
 switchport access vlan 18
 no shutdown
end
```

5. En la PC nueva → Desktop → IP Configuration → DHCP. En Command Prompt:

```text
ipconfig /all
ping 192.198.18.65
ping 192.198.100.130
ping 192.198.100.2
```

6. Espera IP ADMIN `.68–.78`, máscara `/28`, gateway `.65`, DNS `.100.2` en el diseño original. Si se aplicó la ampliación, ADMIN será `.132–.142` con gateway `.129`. `0.0.0.0` o `169.254.x.x` no es una concesión válida del pool.
7. Para COCINA usa otra PC nueva y otro puerto libre (por ejemplo Fa0/4), con `switchport access vlan 28`. Espera `.4–.62` con /26 y gateway `.1` en diseño original; si se corrigió, `.4–.126` con /25 y el límite de clientes configurado.
8. En ambas PCs abre el dominio web. Guarda la configuración del pool, concesión y ping/navegador. Anota IP real de ADMIN como **IP_ADMIN** y de COCINA como **IP_COCINA** para otras pruebas.

**Aprobación:** los dos clientes nuevos obtienen automáticamente parámetros correctos y alcanzan servicios. No fijar IP manual para hacer pasar DHCP. Si falla, revisar puerto/VLAN, trunk, relay, rutas, servicio, gateway del servidor y agotamiento/colisión de pools.

## T07 — Nuevo WiFi Piso 2, ambas WLAN

1. Inspecciona cada WRT: SSID exacto, **SSID Broadcast Disabled**, WPA2-Personal/AES, clave WiFi, contraseña administrativa, DHCP/DNS y modo Router. La guía usa `PISO2_G8_R1`, `PISO2_G8_R2`, clave `G8_PISO2`; aplicar la interpretación de X acordada por el grupo/auxiliar.
2. Agrega un **Laptop-PT nuevo** por WLAN. Si no tiene adaptador inalámbrico: Physical → apagar laptop → retirar módulo Ethernet y colocar WPC300N compatible → encender. También puede usarse un Smartphone nuevo con Wireless0.
3. En laptop → Desktop → PC Wireless → Profiles → New, crea un perfil manual/Advanced con el SSID exacto. Como está oculto, **no debe depender de que aparezca en el escaneo**. Selecciona WPA2-Personal y escribe la clave. Los nombres de botones pueden variar por versión.
4. Si el cliente usa Config → Wireless0 para asociación, introduce allí SSID, WPA2 y clave; luego solicita DHCP en Desktop → IP Configuration. Esto es operación del cliente, no configuración IOS del router.
5. En Command Prompt de la laptop:

```text
ipconfig /all
ping 192.198.28.1
ping 192.198.100.2
```

Para WLAN2 reemplaza el primer destino por `192.198.28.129`. Captura IP/máscara/gateway/DNS; rangos esperados en README. Repite ping hacia **IP_ADMIN** e **IP_COCINA** reales.
6. En Desktop → Web Browser abre `http://www.practica2_Grupo17.com`.
7. Repite con el otro WRT. Guarda perfil manual, estado asociado, DHCP y página web.

**Aprobación:** ambas WLAN funcionan y el dispositivo nuevo entra al SSID oculto, recibe DHCP local y alcanza Piso 1/servicios. Una clave administrativa correcta no sirve como clave WiFi. No habilitar broadcast para evitar la prueba del SSID oculto.

## T08 — Nuevo WiFi Piso 3, ambas WLAN

1. Inspecciona `PISO3_G8_R1` y `PISO3_G8_R3`, **SSID Broadcast Enabled**, WPA2-Personal/AES, clave `G8_PISO3` y administración según README.
2. Agrega una laptop/smartphone nuevo. En PC Wireless → Connect escanea, selecciona SSID y coloca la clave. Debe anunciarse el SSID.
3. Solicita DHCP; en laptop ejecuta `ipconfig /all`. WLAN1: `.38.2–.126`, /25, gateway `192.198.38.1`; WLAN2: `.38.130–.254`, /25, gateway `192.198.38.129`; DNS `192.198.100.2`.
4. Haz ping al gateway correcto, a IP_ADMIN, IP_COCINA y `192.198.100.2`. Abre el dominio web.
5. Repite para la segunda WLAN y captura resultados.

**Aprobación:** ambos WRT entregan DHCP y conectividad; los nuevos clientes se asocian a redes visibles y seguras.

## T09 — Conectividad entre pisos y Datacenter

Anota primero las direcciones **reales** con `ipconfig /all`; los nombres IP_ADMIN/IP_COCINA/IP_P2_R1/etc. son variables de esta guía: sustituirlos, no escribirlos literalmente en Packet Tracer.

| Origen | Destinos | Qué cubre |
|---|---|---|
| Cliente P2 WLAN1 y WLAN2 | IP_ADMIN, IP_COCINA | Piso 2–Piso 1 |
| Cliente P3 WLAN1 y WLAN2 | IP_ADMIN, IP_COCINA | Piso 3–Piso 1 |
| ADMIN y COCINA | IP de cada uno de los cuatro clientes WiFi | Retorno y acceso desde cableado |
| ADMIN y COCINA | 192.198.100.2 y 192.198.100.130 | Piso 1–Datacenter |
| ServerWeb y ServerDHCP, Desktop → Command Prompt | IP_ADMIN, IP_COCINA | Datacenter–Piso 1 |

Ejecuta `ping DIRECCION_REAL` por cada celda. Si el primer ping pierde un paquete por ARP/convergencia, repítelo y documenta ambas observaciones. No atribuyas pérdidas persistentes a ARP. Tras estabilizar, esperar respuestas sostenidas.

Si falla solo desde cableado hacia WiFi, revisar modo Router, rutas estáticas de retorno y firewall de WRT. No desactivar seguridad indiscriminadamente; identificar primero qué tráfico se bloquea.

**Aprobación:** todos los trayectos exigidos funcionan en ambos sentidos. Captura origen/destino y resultado, no solo un sobre verde sin contexto.

## T10 — DNS y HTTP desde los tres pisos

1. En ServerWeb confirma IP `.100.2/25`, gateway `.100.1`, DNS `.100.2`; Services → DNS activado y registro A correcto; Services → HTTP activado y `index.html` con nombres, carnés y Grupo 17.
2. En un cliente de cada VLAN cableada y de cada WLAN, ejecuta:

```text
ipconfig /all
ping 192.198.100.2
nslookup www.practica2_Grupo17.com
```

3. Si esa versión no admite `nslookup`, usa `ping www.practica2_Grupo17.com` y verifica la dirección resuelta; para evidencia DNS específica usa Simulation, filtros DNS/UDP, un cliente nuevo y abre el dominio para observar consulta/respuesta. Una página en caché no demuestra una consulta nueva.
4. En Desktop → Web Browser abre primero `http://192.198.100.2` y después `http://www.practica2_Grupo17.com`.
5. Captura como mínimo una página por piso y resultados para ambas WLAN de cada piso. La URL por dominio debe verse en la captura junto al contenido del grupo.

**Aprobación:** resolución a `192.198.100.2` y HTML correcto desde los tres pisos. Si funciona por IP pero no por nombre, revisar DNS recibido y registro. Si responde ping pero no HTTP, revisar servicio/archivo; ping no prueba HTTP.

## T11 — Estado inicial HSRP

En **R1, R2, R3 y R4**, CLI:

```cisco
enable
show ip interface brief
show standby brief
show standby
```

| Grupo | Activo preferido | Respaldo | VIP |
|---:|---|---|---|
| 18 ADMIN | R4 | R3 | 192.198.18.65 |
| 28 COCINA | R3 | R4 | 192.198.18.1 |
| 38 WEB | R1 | R2 | 192.198.100.1 |
| 48 DHCP | R2 | R1 | 192.198.100.129 |

Cada grupo debe tener **un Active y un Standby**, misma VIP y grupo, interfaces operativas y preempt. Registrar roles observados, prioridades y timers, no asumirlos. Dos activos en un mismo grupo sugieren que no intercambian mensajes; revisar VLAN/trunks antes de continuar.

Desde ADMIN y COCINA prueba gateway, `192.198.100.2` y dominio. Desde ServerWeb y ServerDHCP prueba IP_ADMIN. Si no hay conectividad inicial, detener la inyección de fallas y resolver el problema.

## T12–T15 — Failover HSRP: procedimiento principal

La p. 19 evalúa que al fallar el activo el standby tome el control y se recupere la conectividad. **No especifica un comando de apagado ni obliga a cero paquetes perdidos.** Aquí se aísla por consola al router mediante sus interfaces LAN y WAN; simula su salida de la red sin cortar el acceso local a la CLI. No equivale a un apagado eléctrico. Si el evaluador pide apagar físicamente el router, usa su interruptor y repite las mismas verificaciones, guardando antes su configuración.

**Por qué cuatro casos:** R3 es activo de COCINA, R4 de ADMIN, R1 de WEB y R2 de DHCP. Una sola caída por zona no demuestra failover del activo de cada VLAN.

| Prueba | Router que aislar | Router donde verificar durante falla | Grupo principal | Tráfico de prueba |
|---|---|---|---:|---|
| T12 | R3 | R4 | 28 | COCINA → 192.198.100.2 y dominio |
| T13 | R4 | R3 | 18 | ADMIN → 192.198.100.2 y dominio |
| T14 | R1 | R2 | 38 | ServerWeb → IP_ADMIN, y ADMIN/WiFi → web |
| T15 | R2 | R1 | 48 | ServerDHCP → IP_ADMIN, y cliente cableado nuevo/renovado → DHCP |

En T14/T15 el ping originado en el servidor obliga a usar su gateway local; probar solo desde una PC hacia una VIP no basta para demostrar continuidad del servicio.

### Paso A — Antes de cada caso

1. Confirmar que **todos los routers están restaurados**. No apagar dos routers de una misma pareja al mismo tiempo.
2. Capturar `show standby brief` en los dos routers y `show standby` para timers. Si los roles no coinciden con la tabla, resolver prioridades/preempt o adaptar el caso al activo real y documentarlo.
3. Obtener `ipconfig /all` en el cliente. Anotar IP, máscara, VIP/gateway y DNS; no modificarlos durante la prueba.
4. Ejecutar ping al destino de la tabla y abrir el dominio para tener línea base.
5. Para observar continuidad, consultar `ping /?` en la PC. Si admite `-t`, ejecutar por ejemplo `ping -t 192.198.100.2` y detener con Ctrl+C. Si no lo admite, repetir `ping 192.198.100.2` inmediatamente antes, durante y después. No dar por soportadas opciones de Windows en Packet Tracer.

### Paso B — Provocar la falla

En el **router de la columna “aislar”**, verificar primero que Gi0/1 es la LAN y Gi0/0 la MAN. Según la guía, ejecutar:

```cisco
enable
configure terminal
interface gigabitEthernet0/1
 shutdown
exit
interface gigabitEthernet0/0
 shutdown
end
show ip interface brief
```

Gi0/1 contiene las subinterfaces VLAN; apagar su interfaz padre retira ambas. Gi0/0 retira el enlace MAN. **No guardar startup-config mientras el equipo esté aislado.** No usar `reload`, borrar configuración ni desconectar enlaces de otros equipos.

### Paso C — Verificar durante la falla

1. En el **router sobreviviente**:

```cisco
enable
show standby brief
show standby
show ip route
```

2. Debe quedar Active para **ambos grupos de su pareja**. El grupo principal de la tabla debe haber cambiado de router activo. El peer puede mostrarse desconocido al no haber respaldo disponible: es coherente mientras el otro esté aislado.
3. Registrar hora/tiempo simulado de la falla y de la recuperación. Esperar según los timers observados y la convergencia de rutas; no inventar una duración fija ni afirmar que fueron cero pérdidas si no se midió.
4. Repetir tráfico de la tabla. Pueden perderse respuestas durante convergencia; después deben recuperarse de forma sostenida sin cambiar gateway/IP del cliente. Registrar enviados, recibidos, perdidos y método de medición. Si son pings manuales, indicar que la medición es aproximada.
5. En T12/T13, con el router original aún fuera, renovar DHCP en un cliente de prueba o agregar otro host al puerto/VLAN correspondiente. Si admite comandos:

```text
ipconfig /release
ipconfig /renew
ipconfig /all
```

Si no, solicitar DHCP desde IP Configuration en un cliente nuevo. Esto detecta relay configurado solo en el router caído. No liberar la IP del único cliente con el que estés midiendo continuidad; usa otro para DHCP.
6. En T14 abrir la web desde Piso 1 y al menos un WiFi mientras R1 sigue fuera. En T15 solicitar una concesión nueva y verificar DNS/gateway mientras R2 sigue fuera; una concesión antigua no demuestra que DHCP continúa sirviendo.
7. Capturar estado del sobreviviente y conectividad **antes de restaurar**. La evidencia posterior no demuestra failover durante la falla.

### Paso D — Restaurar siempre, incluso si falló la prueba

En el mismo router aislado, restaurar primero MAN y luego LAN:

```cisco
enable
configure terminal
interface gigabitEthernet0/0
 no shutdown
exit
interface gigabitEthernet0/1
 no shutdown
end
show ip interface brief
show ip eigrp neighbors
show standby brief
```

Esperar convergencia. En ambos routers comprobar roles originales (preempt), pings y dominio. Solo al quedar estable, guardar:

```cisco
copy running-config startup-config
```

Repetir A–D para el siguiente caso. Guardar el PKT al completar y restaurar los cuatro casos.

### Criterios y evidencias por caso

**Aprobado** únicamente si existía conectividad inicial, el standby pasó a Active en el grupo probado, los servicios se recuperaron con el router aislado y la restauración devolvió un estado estable. No marcar aprobado por ver “Active” sin tráfico exitoso.

Guardar `T12-antes.txt/png`, `T12-falla.txt/png`, `T12-durante-ping.png`, `T12-despues.txt/png` y equivalentes T13/T14/T15; añadir DHCP y HTTP donde corresponda. Registrar tiempo y pérdidas en la bitácora.

### Si falla

- Standby no cambia: revisar grupo/VIP/VLAN, trunk al sobreviviente, estado inicial y timers; confirmar que se aisló el activo real.
- Cambia a Active pero no hay tráfico: revisar rutas EIGRP, uplink del sobreviviente, gateway de cliente y servidores, DNS y rutas de retorno.
- Ping funciona pero DHCP nuevo no: revisar helper en **ambos** routers de Piso 1 y retorno desde ServerDHCP.
- Al restaurar no vuelve el activo preferido: comprobar prioridad 110 frente a 100 y `preempt`; registrar lo observado.
- Caída solo de WAN sin cambio HSRP: no es la prueba anterior. HSRP puede seguir viendo al peer por LAN si no hay tracking. No tomar ese resultado como demostración de caída completa ni declarar protección de uplink sin probarla.

## T16 — Cierre documental y ensayo

1. Ejecuta `show ip interface brief`, `show etherchannel summary`, `show ip eigrp neighbors` y `show standby brief` donde corresponda. Todos los enlaces que apagaste deben estar restaurados.
2. Comprueba que los dispositivos nuevos usan DHCP y que el dominio abre en los tres pisos.
3. Guarda configuraciones, PKT final y evidencias. Actualiza README con direccionamiento realmente usado, nombres/carnés, capturas y resultados; no dejar propuestas como si fueran implementaciones.
4. Revisa cada fila de la matriz de cumplimiento del README y de la bitácora. Si queda un fallo, dejarlo visible hasta resolverlo.
5. Guarda versión final como `Practica2_17.pkt` en `Práctica 2` del repositorio original, con README y evidencias. Reabre esa copia para verificar que los cambios quedaron guardados. Comprueba archivos en GitHub y entrega por UEDI.
6. Ensaya explicar: por qué X=8; por qué EIGRP en grupo 17; diferencia entre IP física y VIP; cuál router es activo por VLAN; por qué dos pools centrales y cuatro locales; DNS frente a HTTP; LACP y sus cuatro miembros; cálculo VLSM con reservas; ruta `D` frente a `D EX`; límites de HSRP.

**Entrega no equivale a validación:** solo cerrar como completado cuando los resultados observados y sus archivos respalden los requisitos.
