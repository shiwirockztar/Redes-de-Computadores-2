# Solucionario - Lab 4 (Numerales a a i)

Este documento contiene la resolución completa del enunciado, desde el numeral (a) hasta el (i).

## (a) Direccionamiento a utilizar

Red base asignada: `192.168.78.0/24` (donde `X = 78`, según el caso trabajado).

Subredes definidas para la topología:

- LAN_A: `192.168.78.0/26`
- LAN_B: `192.168.78.64/26`
- Red compartida R1-R2-R3 (Switch1): `192.168.78.128/26`
- Enlace serial R3-R4: `192.168.78.192/29`
- LAN_C: `192.168.78.200/29`

Asignación por interfaces de routers y computadores:

| Dispositivo | Interfaz | Dirección IP | Máscara | Observación |
| --- | --- | --- | --- | --- |
| R1 | Fa0/0 | 192.168.78.1 | 255.255.255.192 | Gateway LAN_A |
| R1 | Fa0/1 | 192.168.78.129 | 255.255.255.192 | Hacia red compartida |
| R2 | Fa0/0 | 192.168.78.65 | 255.255.255.192 | Gateway LAN_B |
| R2 | Fa0/1 | 192.168.78.130 | 255.255.255.192 | Hacia red compartida |
| R3 | Fa0/0 | 192.168.78.131 | 255.255.255.192 | Hacia red compartida |
| R3 | S0/0 | 192.168.78.193 | 255.255.255.248 | Hacia R4 |
| R4 | S0/0 | 192.168.78.194 | 255.255.255.248 | Hacia R3 |
| R4 | Fa0/0 | 192.168.78.201 | 255.255.255.248 | Gateway LAN_C |
| LAN_A | Ethernet | 192.168.78.10 | 255.255.255.192 | GW: 192.168.78.1 |
| LAN_B | Ethernet | 192.168.78.74 | 255.255.255.192 | GW: 192.168.78.65 |
| LAN_C | Ethernet | 192.168.78.202 | 255.255.255.248 | GW: 192.168.78.201 |

Interfaz del router conectada a la red del laboratorio por DHCP:

```bash
conf t
interface <interfaz_hacia_red_laboratorio>
 ip address dhcp
 no shutdown
end
wr
```

Nota: en la topología usada en este laboratorio, el direccionamiento interno principal es estático y solo la interfaz conectada al laboratorio debe ir por DHCP (según el enunciado institucional).

## (b) Configuración IP de la topología, conectividad inicial y análisis de tablas

Con el plan de direccionamiento ya definido, se configuran las interfaces así:

- R1: `Fa0/0 = 192.168.78.1/26`, `Fa0/1 = 192.168.78.129/26`
- R2: `Fa0/0 = 192.168.78.65/26`, `Fa0/1 = 192.168.78.130/26`
- R3: `Fa0/0 = 192.168.78.131/26`, `S0/0 = 192.168.78.193/29`
- R4: `S0/0 = 192.168.78.194/29`, `Fa0/0 = 192.168.78.201/29`
- LAN_A: `192.168.78.10/26 GW 192.168.78.1`
- LAN_B: `192.168.78.74/26 GW 192.168.78.65`
- LAN_C: `192.168.78.202/29 GW 192.168.78.201`

Si el enunciado exige que la interfaz conectada a la red del laboratorio use DHCP, en ese puerto del router se debe usar:

```bash
conf t
interface <interfaz_hacia_red_laboratorio>
 ip address dhcp
 no shutdown
end
wr
```

Conectividad esperada **antes** de agregar rutas estáticas:

- Ping exitoso dentro de cada red directamente conectada (PC a su gateway).
- Entre routers, ping exitoso solo entre vecinos directos (por ejemplo R3 <-> R4 por serial).
- Ping fallido entre LAN_A y LAN_B, LAN_A y LAN_C, LAN_B y LAN_C.

### Ejemplos de verificación (estado inicial, sin rutas estáticas)

Desde LAN_A (VPCS):

```bash
ping 192.168.78.1
```

Salida esperada (éxito hacia gateway local):

```text
84 bytes from 192.168.78.1 icmp_seq=1 ttl=255 time=1.2 ms
84 bytes from 192.168.78.1 icmp_seq=2 ttl=255 time=0.9 ms
84 bytes from 192.168.78.1 icmp_seq=3 ttl=255 time=1.1 ms
84 bytes from 192.168.78.1 icmp_seq=4 ttl=255 time=1.0 ms
84 bytes from 192.168.78.1 icmp_seq=5 ttl=255 time=1.1 ms
```

Desde LAN_A (VPCS), hacia LAN_B:

```bash
ping 192.168.78.74
```

Salida esperada (fallo, aun no hay ruta remota):

```text
Request timeout for icmp_seq 1
Request timeout for icmp_seq 2
Request timeout for icmp_seq 3
Request timeout for icmp_seq 4
Request timeout for icmp_seq 5
```

Tambien es normal ver rechazo explicito desde el gateway (R1):

```text
*192.168.78.1 icmp_seq=1 ttl=255 time=15.338 ms (ICMP type:3, code:1, Destination host unreachable)
*192.168.78.1 icmp_seq=2 ttl=255 time=14.910 ms (ICMP type:3, code:1, Destination host unreachable)
*192.168.78.1 icmp_seq=3 ttl=255 time=15.624 ms (ICMP type:3, code:1, Destination host unreachable)
*192.168.78.1 icmp_seq=4 ttl=255 time=15.622 ms (ICMP type:3, code:1, Destination host unreachable)
*192.168.78.1 icmp_seq=5 ttl=255 time=15.401 ms (ICMP type:3, code:1, Destination host unreachable)
```

Interpretacion de esa salida:

1. LAN_A si llega a su gateway (192.168.78.1).
2. El gateway R1 no sabe como llegar a 192.168.78.74.
3. Por eso R1 devuelve ICMP tipo 3 codigo 1 (Destination host unreachable).

Desde R3 (router), vecino directo por serial:

```bash
ping 192.168.78.194
```

Salida esperada (éxito entre vecinos directos):

```text
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.78.194, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5)
```

Explicación con tablas de enrutamiento (`show ip route`):

- Cada router solo conoce rutas `C` (Connected) y `L` (Local) de sus interfaces activas.
- Aún no existen rutas `S` (Static) hacia redes remotas.
- Por eso, si el destino no coincide con una red conectada y no hay ruta por defecto, el paquete se descarta.

## (c) Rutas estáticas en R1 y R2 para alcanzar redes conectadas del otro

Como R1 y R2 comparten el segmento `192.168.78.128/26` con R3, lo recomendado es configurar rutas estáticas con **siguiente salto**.

### Configuración sugerida

En R1:

```bash
conf t
ip route 192.168.78.64 255.255.255.192 192.168.78.130
end
wr
```

En R2:

```bash
conf t
ip route 192.168.78.0 255.255.255.192 192.168.78.129
end
wr
```

¿Interfaz de salida o siguiente salto?

- En enlaces Ethernet multiacceso, es mejor **siguiente salto**.
- Con solo interfaz de salida, el router puede intentar resolver ARP del destino final remoto en ese enlace, lo que introduce tráfico innecesario y ambigüedad.
- Con siguiente salto, la resolución de Capa 2 es contra el vecino inmediato correcto.

Recorrido de un paquete de red de R2 a red de R1 (ejemplo LAN_B -> LAN_A):

1. LAN_B envía paquete a su gateway R2 (`192.168.78.65`).
2. R2 consulta tabla y encuentra ruta estática a `192.168.78.0/26` vía `192.168.78.129`.
3. R2 reencapsula con MAC de R1 (resuelta por ARP) y reenvía por `Fa0/1`.
4. R1 recibe, detecta que `192.168.78.0/26` está conectada por `Fa0/0` y entrega al host destino.

### Verificación después de (c)

Después de configurar solo las rutas estáticas entre R1 y R2, debe cumplirse:

- LAN_A <-> LAN_B: éxito.
- LAN_A/LAN_B hacia LAN_C: aún puede fallar, porque faltan rutas por defecto y retorno completo por R3/R4.

Prueba sugerida desde LAN_A:

```bash
ping 192.168.78.74
```

Salida esperada:

```text
84 bytes from 192.168.78.74 icmp_seq=1 ttl=126 time=2.8 ms
84 bytes from 192.168.78.74 icmp_seq=2 ttl=126 time=2.6 ms
84 bytes from 192.168.78.74 icmp_seq=3 ttl=126 time=2.7 ms
84 bytes from 192.168.78.74 icmp_seq=4 ttl=126 time=2.5 ms
84 bytes from 192.168.78.74 icmp_seq=5 ttl=126 time=2.6 ms
```

Prueba sugerida desde LAN_A hacia LAN_C (todavía incompleto en este punto):

```bash
ping 192.168.78.202
```

Salida esperada tipica en esta etapa:

```text
Request timeout for icmp_seq 1
Request timeout for icmp_seq 2
Request timeout for icmp_seq 3
Request timeout for icmp_seq 4
Request timeout for icmp_seq 5
```

Alternativamente, puede verse ICMP type:3 code:1 (Destination host unreachable) generado por el gateway intermedio.

## (d) Ruta por defecto en R1 y R2 + comandos en R3 y R4

Objetivo: que R1 y R2 alcancen redes del resto de la topología a través de R3.

En R1:

```bash
conf t
ip route 0.0.0.0 0.0.0.0 192.168.78.131
end
wr
```

En R2:

```bash
conf t
ip route 0.0.0.0 0.0.0.0 192.168.78.131
end
wr
```

Para que R3 y R4 regresen tráfico hacia redes de R1 y R2:

En R3:

```bash
conf t
ip route 192.168.78.0 255.255.255.192 192.168.78.129
ip route 192.168.78.64 255.255.255.192 192.168.78.130
end
wr
```

En R4:

```bash
conf t
ip route 192.168.78.0 255.255.255.192 192.168.78.193
ip route 192.168.78.64 255.255.255.192 192.168.78.193
end
wr
```

¿Se pueden usar rutas por defecto en R3 y R4 para llegar a redes de R1?

- Técnicamente sí, pero no siempre es conveniente.
- Si a la derecha de la topología se agregan nuevas redes, una default mal orientada puede generar caminos subóptimos o loops.
- En escenarios de crecimiento, conviene mantener rutas específicas (o migrar a enrutamiento dinámico).

### Verificación después de (d)

Con rutas por defecto en R1 y R2, y rutas de retorno en R3/R4, ya debe existir conectividad extremo a extremo.

Desde LAN_A:

```bash
ping 192.168.78.202
trace 192.168.78.202
```

Salida esperada del `ping`:

```text
84 bytes from 192.168.78.202 icmp_seq=1 ttl=124 time=8.1 ms
84 bytes from 192.168.78.202 icmp_seq=2 ttl=124 time=7.9 ms
84 bytes from 192.168.78.202 icmp_seq=3 ttl=124 time=8.0 ms
84 bytes from 192.168.78.202 icmp_seq=4 ttl=124 time=8.2 ms
84 bytes from 192.168.78.202 icmp_seq=5 ttl=124 time=7.8 ms
```

Salida esperada del `trace` (saltos de referencia):

```text
trace to 192.168.78.202, 8 hops max
 1   192.168.78.1
 2   192.168.78.131
 3   192.168.78.194
 4   192.168.78.202
```

Nota: los tiempos y TTL pueden variar según simulación/equipo, pero el criterio de evaluación es que pase de fallo a éxito cuando las rutas correctas están aplicadas.

### Notas de corrección rápida (cuando falla el literal d)

Si desde LAN_A ves este patrón:

- `ping 192.168.78.202` con `ICMP type:3, code:1` desde `192.168.78.131`.
- `trace 192.168.78.202` llega a `192.168.78.131` y luego marca unreachable.

Diagnóstico directo: el tráfico llega hasta R3, pero R3 no está alcanzando LAN_C (`192.168.78.200/29`) o el enlace serial R3-R4 no está operativo.

Corrección mínima en R3:

```bash
conf t
ip route 192.168.78.200 255.255.255.248 192.168.78.194
end
wr
```

Comando que faltó (crítico en este caso real):

```bash
R3(config)# ip route 192.168.78.200 255.255.255.248 192.168.78.194
```

Si este comando no está, LAN_A puede llegar hasta R3 pero no a LAN_C, y aparece `Destination host unreachable` desde `192.168.78.131`.

Verificación recomendada por equipo:

En R3:

```bash
show ip route 192.168.78.200
show ip interface brief
ping 192.168.78.194
```

En R4:

```bash
show ip interface brief
show ip route 192.168.78.0
show ip route 192.168.78.64
```

En LAN_A (VPCS):

```bash
ping 192.168.78.202
trace 192.168.78.202
```

Criterio de éxito final:

1. `ping` LAN_A -> LAN_C responde.
2. `trace` muestra salto por `192.168.78.1 -> 192.168.78.131 -> 192.168.78.194 -> 192.168.78.202`.
3. Interfaces clave (`R3 S0/0` y `R4 S0/0`) en estado `up/up`.

Si la serial está down:

```bash
conf t
interface s0/0
no shutdown
```

Y en el lado DCE del serial, configurar reloj (ejemplo):

```bash
clock rate 64000
```

### Procedimiento preventivo y de troubleshooting (usar siempre antes de sustentar)

Aplicar este flujo en orden reduce casi por completo los errores repetitivos del literal (d).

#### 1) Checklist preventivo previo (antes de probar ping extremo a extremo)

En todos los routers (R1, R2, R3, R4):

```bash
show ip interface brief
show ip route static
```

Validar obligatoriamente:

1. Interfaces de trabajo en `up/up`.
2. R1 y R2 con ruta por defecto a `192.168.78.131`.
3. R3 con rutas de retorno a `192.168.78.0/26` y `192.168.78.64/26`.
4. R3 con ruta a LAN_C: `192.168.78.200/29` via `192.168.78.194`.
5. R4 con rutas de retorno a LAN_A y LAN_B via `192.168.78.193`.

En hosts VPCS:

```bash
show ip
```

Validar IP, máscara y gateway de cada LAN, especialmente LAN_C:

- IP: `192.168.78.202`
- Mask: `255.255.255.248`
- Gateway: `192.168.78.201`

#### 2) Flujo de diagnóstico por capas (cuando un ping falla)

Orden recomendado de pruebas:

1. Host -> gateway local.
2. Router intermedio -> vecino directo.
3. Router intermedio -> host final.
4. Host origen -> host final.
5. `trace` desde host origen para ubicar el salto donde muere el tráfico.

Comandos rápidos:

```bash
# En VPCS
ping <gateway_local>
ping <destino_final>
trace <destino_final>

# En routers
show ip route <red_destino>
ping <next-hop>
ping <host_destino>
```

#### 3) Matriz rápida: síntoma -> causa probable -> corrección

| Síntoma observado | Causa probable | Verificación | Corrección típica |
| --- | --- | --- | --- |
| `ICMP type:3 code:1` desde `192.168.78.1` | R1 sin ruta para red remota | `show ip route <red_remota>` en R1 | Ajustar default o ruta específica en R1. |
| `ICMP type:3 code:1` desde `192.168.78.131` | R3 sin ruta a LAN_C | `show ip route 192.168.78.200` en R3 | `ip route 192.168.78.200 255.255.255.248 192.168.78.194` |
| R3 llega a `192.168.78.194` pero no a `192.168.78.202` | Problema en Fa0/0 de R4 o en LAN_C | `show ip int brief` en R4, `show ip` en LAN_C | Corregir IP/máscara/GW de LAN_C o levantar Fa0/0. |
| `trace` se queda en R3 y no pasa a R4 | Falta ruta o retorno incompleto | Revisar rutas estáticas en R3 y R4 | Completar rutas ida y vuelta. |
| Serial `down/down` o `administratively down` | Interfaz apagada o sin clock DCE | `show ip int brief`, `show controllers serial` | `no shutdown` y `clock rate` en lado DCE. |

#### 4) Secuencia de cierre (evidencia final para entregar)

1. En R3: `show ip route 192.168.78.200`
2. En R4: `show ip interface brief`
3. En LAN_A: `ping 192.168.78.202`
4. En LAN_A: `trace 192.168.78.202`

Resultado esperado de cierre:

- Ping exitoso extremo a extremo.
- Trazado por `192.168.78.1 -> 192.168.78.131 -> 192.168.78.194 -> 192.168.78.202`.

Con este flujo, cualquier falla queda acotada en pocos comandos y se corrige sin rehacer toda la configuración.

## (e) Conectividad entre topologías de compañeros con una ruta por vecino

Se pide una ruta estática por router para cada topología vecina conectada a la red del laboratorio.

Estrategia:

1. Identificar para cada vecino su prefijo agregado (resumen de su topología).
2. En R4 (router de borde hacia laboratorio), crear una ruta por cada topología vecina:

```bash
conf t
ip route <RED_VECINO_1> <MASCARA> <NEXT_HOP_LAB_1>
ip route <RED_VECINO_2> <MASCARA> <NEXT_HOP_LAB_2>
...
end
wr
```

3. En routers internos (R1, R2, R3), mantener salida hacia R4/R3 según diseño (default o rutas específicas).

Si hay 7 estudiantes, en R4 se agregan 6 rutas (una por cada topología vecina).

## (f) ¿Qué pasa si todos configuran solo ruta por defecto?

Si todos usan únicamente default routes:

- Puede haber conectividad parcial si la topología es estrictamente jerárquica y sin caminos ambiguos.
- En red compartida entre varios estudiantes, es probable crear reenvíos circulares (loops), black holes o rutas no determinísticas.
- También aumenta la dependencia de un único camino y dificulta depuración.

Conclusión: no es buena práctica para lograr conectividad completa y estable en un escenario multi-topología. Se requieren rutas específicas (o protocolo dinámico).

## (g) Proceso cuando la ruta estática coincide y fue configurada con interfaz de salida

Proceso general en el router:

1. Recibe trama, valida FCS y extrae paquete IP.
2. Decrementa TTL del paquete.
3. Busca coincidencia por prefijo más largo en tabla de enrutamiento.
4. Encuentra ruta estática tipo `S` con interfaz de salida (ejemplo `ip route 10.0.0.0 255.255.255.0 fa0/1`).
5. Determina que debe enviar por esa interfaz.
6. Si el medio es Ethernet, necesita resolver dirección MAC de siguiente dispositivo (ARP).
7. Reencapsula en nueva trama L2 con MAC origen de salida y MAC destino resuelta.
8. Reenvía el paquete.

Riesgo en redes multiacceso: al no definir siguiente salto explícito, el router puede intentar resolver ARP del destino final remoto.

## (h) Proceso cuando la ruta estática coincide y fue configurada con siguiente salto

Proceso general:

1. Router recibe paquete y decrementa TTL.
2. Aplica búsqueda de prefijo más largo.
3. Encuentra ruta estática tipo `S` con next-hop (ejemplo `ip route 10.0.0.0 255.255.255.0 192.168.1.2`).
4. Realiza búsqueda recursiva para llegar al next-hop (si no está directamente conectado).
5. Cuando identifica interfaz de salida, resuelve MAC del next-hop (ARP en Ethernet).
6. Reencapsula trama L2 hacia el vecino next-hop.
7. Reenvía paquete.

Ventaja: comportamiento más preciso en enlaces multiacceso y menor ambigüedad de reenvío.

## (i) Proceso cuando hay más de una coincidencia (incluida posible default)

El router aplica esta lógica:

1. Evalúa todas las rutas que coinciden con IP destino.
2. Elige la de **prefijo más largo** (Longest Prefix Match).
3. Si hubiera empate exacto de prefijo entre distintas fuentes, aplica distancia administrativa y métrica según protocolo.
4. Si no hay coincidencia específica, usa la ruta por defecto (`0.0.0.0/0`) si existe.
5. Si no hay default, descarta paquete y normalmente genera ICMP Destination Unreachable.

Ejemplo:

- Destino `192.168.78.70`
- Coincidencias posibles: `192.168.78.64/26` y `0.0.0.0/0`
- Se selecciona `192.168.78.64/26` por ser más específica.

## Comandos de verificación recomendados

En routers:

```bash
show ip interface brief
show ip route
show ip route static
show arp
traceroute <ip_destino>
```

En VPCS:

```bash
show ip
ping <ip_destino>
trace <ip_destino>
```

Con estas verificaciones puedes justificar cada resultado de conectividad y el camino real de los paquetes.

## Checklist rapido para sustentacion

Usa esta tabla como guion corto: ejecuta comando, muestra salida y concluye si cumple.

| Etapa | Prueba | Equipo origen | Comando | Resultado esperado |
| --- | --- | --- | --- | --- |
| Inicial (sin rutas estaticas) | Gateway local LAN_A | LAN_A (VPCS) | `ping 192.168.78.1` | Exitoso (responde). |
| Inicial (sin rutas estaticas) | LAN_A a LAN_B | LAN_A (VPCS) | `ping 192.168.78.74` | Fallido (timeout o Destination host unreachable desde 192.168.78.1). |
| Inicial (sin rutas estaticas) | Vecinos directos serial | R3 | `ping 192.168.78.194` | Exitoso (5/5). |
| Despues de (c) | LAN_A a LAN_B | LAN_A (VPCS) | `ping 192.168.78.74` | Exitoso. |
| Despues de (c) | LAN_A a LAN_C | LAN_A (VPCS) | `ping 192.168.78.202` | Aun puede fallar (si no esta completo el retorno). |
| Despues de (d) | LAN_A a LAN_C | LAN_A (VPCS) | `ping 192.168.78.202` | Exitoso extremo a extremo. |
| Despues de (d) | Camino LAN_A a LAN_C | LAN_A (VPCS) | `trace 192.168.78.202` | Saltos esperados por R1 -> R3 -> R4 -> LAN_C. |
| Validacion de rutas | Ver rutas estaticas | R1/R2/R3/R4 | `show ip route static` | Deben aparecer rutas `S` configuradas. |
| Validacion de interfaz | Estado interfaces | R1/R2/R3/R4 | `show ip interface brief` | Interfaces usadas en `up/up`. |

Tip rapido para exponer:

1. Muestra primero un ping que falla (estado inicial).
2. Aplica rutas y repite el mismo ping para evidenciar cambio a exito.
3. Cierra con `trace` para justificar el recorrido.