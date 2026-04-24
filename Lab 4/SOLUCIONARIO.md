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