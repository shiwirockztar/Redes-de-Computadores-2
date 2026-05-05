# README General - Lab 4 con red variable 192.168.XX.0/24

## 1. Objetivo

Este documento sirve como plantilla para cualquier usuario que elija su propia red base:

- Red local del laboratorio: `192.168.XX.0/24`
- `XX` son 2 digitos definidos por cada usuario (00-99).

Despues de montar la topologia de Lab 4 y conectar Cloud a R4, puedes interconectar tu topologia con otra igual (otro usuario) repitiendo los mismos pasos y cambiando `XX` por su valor.

## 2. Convencion de variables

- `XX`: tu red local (ejemplo: 78 -> `192.168.78.0/24`)
- `YY`: red de otro laboratorio (ejemplo: 45 -> `192.168.45.0/24`)

## 3. Topologia base para cualquier XX

Mantiene la misma estructura del Lab 4, solo cambia el tercer octeto:

- LAN_A: `192.168.XX.0/26`
- LAN_B: `192.168.XX.64/26`
- Switch1: `192.168.XX.128/26`
- Enlace R3-R4: `192.168.XX.192/29`
- LAN_C: `192.168.XX.200/29`

---

## 3.1 Tabla de Direccionamiento para tu Red 192.168.XX.0/24

### Red base y subdivisión

**Red asignada:** `192.168.XX.0/24` (256 direcciones)

**Cinco redes requeridas:**

| # | RED | CIDR | Rango | Hosts útiles | Dispositivos |
| --- | --- | --- | --- | --- | --- |
| 1 | LAN_A | /26 | 192.168.XX.0 - 192.168.XX.63 | 62 | LAN_A ↔ R1 |
| 2 | LAN_B | /26 | 192.168.XX.64 - 192.168.XX.127 | 62 | LAN_B ↔ R2 |
| 3 | Switch1 | /26 | 192.168.XX.128 - 192.168.XX.191 | 62 | R1, R2, R3 ↔ Switch1 |
| 4 | R3-R4 | /29 | 192.168.XX.192 - 192.168.XX.199 | 6 | R3 ↔ R4 (Serial) |
| 5 | LAN_C | /29 | 192.168.XX.200 - 192.168.XX.207 | 6 | R4 ↔ LAN_C |

### Tabla de direcciones resumen

| Dispositivo | Interfaz | Dirección IP | Máscara | Red |
| --- | --- | --- | --- | --- |
| R1 | Fa0/0 | 192.168.XX.1 | 255.255.255.192 | LAN_A |
| R1 | Fa0/1 | 192.168.XX.129 | 255.255.255.192 | Switch1 |
| R2 | Fa0/0 | 192.168.XX.65 | 255.255.255.192 | LAN_B |
| R2 | Fa0/1 | 192.168.XX.130 | 255.255.255.192 | Switch1 |
| R3 | Fa0/0 | 192.168.XX.131 | 255.255.255.192 | Switch1 |
| R3 | s0/0 | 192.168.XX.193 | 255.255.255.248 | R3-R4 |
| R4 | Fa0/0 | 192.168.XX.201 | 255.255.255.248 | LAN_C |
| R4 | s0/0 | 192.168.XX.194 | 255.255.255.248 | R3-R4 |
| Switch1 | VLAN 1 | 192.168.XX.132 | 255.255.255.192 | Switch1 |
| LAN_A | Ethernet | 192.168.XX.10 | 255.255.255.192 | LAN_A |
| LAN_B | Ethernet | 192.168.XX.74 | 255.255.255.192 | LAN_B |
| LAN_C | Ethernet | 192.168.XX.202 | 255.255.255.248 | LAN_C |

### Puertos de conexión por Telnet (GNS3)

| Dispositivo | Puerto | Comando |
| --- | --- | --- |
| LAN_A | 5004 | `telnet localhost 5004` |
| LAN_B | 5006 | `telnet localhost 5006` |
| LAN_C | 5008 | `telnet localhost 5008` |
| R1 | 5000 | `telnet localhost 5000` |
| R2 | 5001 | `telnet localhost 5001` |
| R3 | 5002 | `telnet localhost 5002` |
| R4 | 5003 | `telnet localhost 5003` |
| Switch1 | none | N/A |

Nota: Los números de puerto pueden variar según la configuración de GNS3. Verifica en la consola de GNS3 el puerto específico de cada dispositivo si estos valores no corresponden.

---

## 4. Procedimiento paso a paso (tu topologia local)

### Paso 1. Configurar interfaces principales (R1-R4)

R1:
```bash
conf t
int fa0/0
 ip address 192.168.XX.1 255.255.255.192
 no shut
int fa0/1
 ip address 192.168.XX.129 255.255.255.192
 no shut
end
wr
```

### Paso 1.1 Configurar PCs de las LAN (VPCS)

LAN_A:
```bash
ip 192.168.XX.10/26 192.168.XX.1
save
ping 192.168.XX.1
```

LAN_B:
```bash
ip 192.168.XX.74/26 192.168.XX.65
save
ping 192.168.XX.65
```

LAN_C:
```bash
ip 192.168.XX.202/29 192.168.XX.201
save
ping 192.168.XX.201
```

Si usas Linux en vez de VPCS, el equivalente es:

LAN_A (Linux):
```bash
ip addr add 192.168.XX.10/26 dev eth0
ip route add default via 192.168.XX.1
ping 192.168.XX.1
```

LAN_B (Linux):
```bash
ip addr add 192.168.XX.74/26 dev eth0
ip route add default via 192.168.XX.65
ping 192.168.XX.65
```

LAN_C (Linux):
```bash
ip addr add 192.168.XX.202/29 dev eth0
ip route add default via 192.168.XX.201
ping 192.168.XX.201
```

R2:
```bash
conf t
int fa0/0
 ip address 192.168.XX.65 255.255.255.192
 no shut
int fa0/1
 ip address 192.168.XX.130 255.255.255.192
 no shut
end
wr
```

R3:
```bash
conf t
int fa0/0
 ip address 192.168.XX.131 255.255.255.192
 no shut
int s0/0
 ip address 192.168.XX.193 255.255.255.248
 no shut
end
wr
```

R4 (lado interno):
```bash
conf t
int fa0/0
 ip address 192.168.XX.201 255.255.255.248
 no shut
int s0/0
 ip address 192.168.XX.194 255.255.255.248
 no shut
end
wr
```

## 4.1 Análisis de conectividad y tablas de enrutamiento (Paso b)

### (b) Configuración IP de la topología, conectividad inicial y análisis de tablas

Después de configurar todas las interfaces según el Paso 1, verifica la conectividad inicial dentro de cada red.

#### Conectividad esperada **antes** de agregar rutas estáticas adicionales:

- Ping exitoso dentro de cada red directamente conectada (PC a su gateway).
- Entre routers, ping exitoso solo entre vecinos directos (por ejemplo R3 <-> R4 por serial).
- Ping fallido entre LAN_A y LAN_B, LAN_A y LAN_C, LAN_B y LAN_C.

#### Ejemplos de verificación:

Desde LAN_A (VPCS), ping al gateway local:
```bash
ping 192.168.XX.1
```

Salida esperada (éxito):
```text
84 bytes from 192.168.XX.1 icmp_seq=1 ttl=255 time=1.2 ms
84 bytes from 192.168.XX.1 icmp_seq=2 ttl=255 time=0.9 ms
84 bytes from 192.168.XX.1 icmp_seq=3 ttl=255 time=1.1 ms
84 bytes from 192.168.XX.1 icmp_seq=4 ttl=255 time=1.0 ms
84 bytes from 192.168.XX.1 icmp_seq=5 ttl=255 time=1.1 ms
```

Desde LAN_A (VPCS), hacia LAN_B (aún sin rutas):
```bash
ping 192.168.XX.74
```

Salida esperada (fallo):
```text
Request timeout for icmp_seq 1
Request timeout for icmp_seq 2
Request timeout for icmp_seq 3
Request timeout for icmp_seq 4
Request timeout for icmp_seq 5
```

O alternativamente, rechazo explícito desde el gateway:
```text
*192.168.XX.1 icmp_seq=1 ttl=255 time=15.338 ms (ICMP type:3, code:1, Destination host unreachable)
```

Desde R3 (router), verificar enlace serial con vecino directo R4:
```bash
ping 192.168.XX.194
```

Salida esperada (éxito entre vecinos directos, serial):
```text
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.XX.194, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5)
```

#### Análisis de tablas de enrutamiento (directamente conectadas, sin rutas estáticas aún):

En cualquier router, ejecuta:
```bash
show ip route
```

Expected output (solo rutas conectadas y locales):
```bash
C   192.168.XX.0/26 is directly connected, FastEthernet0/0
L   192.168.XX.1/32 is directly connected, FastEthernet0/0
C   192.168.XX.128/26 is directly connected, FastEthernet0/1
L   192.168.XX.129/32 is directly connected, FastEthernet0/1
```

**Interpretación:**
- `C` = Connected (red conectada directamente)
- `L` = Local (IP local del router en esa red)
- Aún no hay rutas `S` (Static) hacia redes remotas
- Por eso, si el destino no está conectado directamente, el paquete se descarta

---

### Paso 4.2 Rutas estáticas entre R1 y R2 (Paso c)

## (c) Rutas estáticas en R1 y R2 para alcanzar redes conectadas del otro

Como R1 y R2 comparten el segmento `192.168.XX.128/26` con R3, la recomendación es configurar rutas estáticas con **siguiente salto** (next-hop) en lugar de solo interfaz de salida.

#### ¿Por qué siguiente salto es mejor en redes Ethernet multiacceso?

- Con solo interfaz de salida, el router intenta resolver ARP del destino final remoto en ese enlace, generando tráfico innecesario y ambigüedad.
- Con siguiente salto, la resolución ARP es contra el vecino inmediato correcto, lo que garantiza entrega más eficiente.

#### Configuración sugerida

En R1:
```bash
conf t
ip route 192.168.XX.64 255.255.255.192 192.168.XX.130
end
wr
```

En R2:
```bash
conf t
ip route 192.168.XX.0 255.255.255.192 192.168.XX.129
end
wr
```

#### Recorrido de un paquete (ejemplo: LAN_B → LAN_A)

1. Equipo en LAN_B envía paquete hacia LAN_A (ej: `192.168.XX.10`) a su gateway R2 (`192.168.XX.65`).
2. R2 consulta su tabla de enrutamiento y encuentra la ruta estática a `192.168.XX.0/26` vía siguiente salto `192.168.XX.129` (R1).
3. R2 reencapsula el paquete con MAC de R1 (resuelta por ARP a través de la red compartida `192.168.XX.128/26`) y lo reenvia por `Fa0/1`.
4. R1 recibe el paquete, consulta su tabla y detecta que `192.168.XX.0/26` está conectada directamente en `Fa0/0`, así que lo entrega al host destino en LAN_A.

#### Verificación después de (c)

Desde LAN_A, prueba hacia LAN_B:
```bash
ping 192.168.XX.74
```

Salida esperada (ahora debe ser exitoso):
```text
84 bytes from 192.168.XX.74 icmp_seq=1 ttl=126 time=2.8 ms
84 bytes from 192.168.XX.74 icmp_seq=2 ttl=126 time=2.6 ms
84 bytes from 192.168.XX.74 icmp_seq=3 ttl=126 time=2.7 ms
84 bytes from 192.168.XX.74 icmp_seq=4 ttl=126 time=2.5 ms
84 bytes from 192.168.XX.74 icmp_seq=5 ttl=126 time=2.6 ms
```

**Nota:** Aún no hay conectividad entre LAN_A/LAN_B y LAN_C, porque faltan rutas por defecto y retorno completo por R3/R4.

---

### Paso 4.3 Rutas por defecto en R1 y R2, propagación en R3 y R4 (Paso d)

## (d) Ruta por defecto en R1 y R2 + comandos en R3 y R4 para completar conectividad

Objetivo: que R1 y R2 alcancen redes del resto de la topología (R3, R4, LAN_C) a través de R3. Simultáneamente, R3 y R4 deben tener rutas para regresar tráfico hacia LAN_A y LAN_B.

#### Configuración de rutas por defecto

En R1:
```bash
conf t
ip route 0.0.0.0 0.0.0.0 192.168.XX.131
end
wr
```

En R2:
```bash
conf t
ip route 0.0.0.0 0.0.0.0 192.168.XX.131
end
wr
```

Esta ruta por defecto `0.0.0.0/0` indica que cualquier destino no conocido debe enviarse a R3 (`192.168.XX.131`).

#### Configuración de rutas de retorno en R3 y R4

En R3, agregar rutas específicas para alcanzar LAN_A y LAN_B:
```bash
conf t
ip route 192.168.XX.0 255.255.255.192 192.168.XX.129
ip route 192.168.78.200 255.255.255.248 192.168.78.194
ip route 192.168.XX.64 255.255.255.192 192.168.XX.130
end
wr
```
Si desde LAN_A ves este patrón (si por ejemplo no anexas 
ip route 192.168.78.200 255.255.255.248 192.168.78.194):

- `ping 192.168.78.202` con `ICMP type:3, code:1` desde `192.168.78.131`.
- `trace 192.168.78.202` llega a `192.168.78.131` y luego marca unreachable.

Diagnóstico directo: el tráfico llega hasta R3, pero R3 no está alcanzando LAN_C (`192.168.78.200/29`) o el enlace serial R3-R4 no está operativo.


En R4, agregar rutas que apunten hacia R3 para cualquier destino en redes de R1 y R2:
```bash
conf t
ip route 192.168.XX.0 255.255.255.192 192.168.XX.193
ip route 192.168.XX.64 255.255.255.192 192.168.XX.193
end
wr
```

#### ¿Se pueden usar rutas por defecto en R3 y R4 para llegar a redes de R1?

**Respuesta técnica:** Sí, es posible, pero **no siempre es conveniente**, especialmente en escenarios de crecimiento.

**¿Por qué no es óptimo a futuro?**

- Si a la derecha de la topología se agregan nuevas redes, una ruta por defecto mal orientada puede generar caminos subóptimos o incluso loops de enrutamiento.
- En escenarios donde la topología evoluciona, conviene mantener rutas específicas (o migrar a protocolos de enrutamiento dinámico como OSPF o RIPv2).
- R3 y R4 con rutas específicas ofrecen mayor control y predictibilidad.

**Mejor práctica:** Usa rutas específicas en R3 y R4 (como se muestra arriba) al menos mientras la topología sea pequeña y controlable.

#### Verificación después de (d)

Desde LAN_A, ping extremo a extremo hacia LAN_C:
```bash
ping 192.168.XX.202
```

Salida esperada (ahora debe ser exitoso):
```text
84 bytes from 192.168.XX.202 icmp_seq=1 ttl=124 time=8.1 ms
84 bytes from 192.168.XX.202 icmp_seq=2 ttl=124 time=7.9 ms
84 bytes from 192.168.XX.202 icmp_seq=3 ttl=124 time=8.0 ms
84 bytes from 192.168.XX.202 icmp_seq=4 ttl=124 time=8.2 ms
84 bytes from 192.168.XX.202 icmp_seq=5 ttl=124 time=7.8 ms
```

Desde LAN_A, trazar la ruta completa:
```bash
trace 192.168.XX.202
```

Salida esperada (muestra los saltos):
```text
trace to 192.168.XX.202, 8 hops max
 1   192.168.XX.1      (R1 local)
 2   192.168.XX.131    (R3)
 3   192.168.XX.194    (R4)
 4   192.168.XX.202    (host en LAN_C)
```

#### Checklist preventivo de troubleshooting para paso (d)

Si el ping desde LAN_A a LAN_C falla:

1. Verifica que todas las interfaces estén en estado `up/up`:
   ```bash
   show ip interface brief
   ```

2. En todos los routers, verifica que existan las rutas esperadas:
   ```bash
   show ip route static
   ```

3. En R3, verifica específicamente que tenga ruta hacia LAN_C:
   ```bash
   show ip route 192.168.XX.200
   ```
   Debe mostrar la ruta estática a través de `192.168.XX.194`.

4. En R4, verifica que tenga rutas de retorno hacia R1 y R2:
   ```bash
   show ip route 192.168.XX.0
   show ip route 192.168.XX.64
   ```

5. Verifica manualmente que el enlace serial R3-R4 está operativo:
   ```bash
   # En R3:
   ping 192.168.XX.194
   
   # En R4:
   ping 192.168.XX.193
   ```

Si la serial está `down/down`, actívala:
```bash
conf t
interface s0/0
 clock rate 64000
 no shutdown
end
wr
```

---

## (e) Conectividad con topologías de compañeros usando una ruta estática por vecino

Objetivo: conectar tu topología (`192.168.XX.0/24`) con las de tus compañeros (`192.168.YY.0/24`, `192.168.ZZ.0/24`, etc.) usando el Cloud como segmento de tránsito y una sola ruta estática por R4 hacia cada compañero.

### Prerequisito: Cloud conectado a R4

1. Arrastra un nodo Cloud en GNS3.
2. Conecta Cloud a la interfaz `Fa0/1` de tu R4.
3. Asocia el Cloud a una interfaz real/virtual de tu host.

### Configuración de R4 (lado Cloud)

R4:
```bash
conf t
interface fa0/1
 ip address dhcp
 no shutdown
exit
end
wr
```

Verifica que obtuvo IP por DHCP (ejemplo: `192.168.0.106`):
```bash
show ip interface brief
```

Salida esperada:
```
FastEthernet0/1            192.168.0.106   YES DHCP   up                    up
```

### Agregar una ruta por cada compañero

En tu R4, agrega una ruta estática hacia cada red de compañero usando su IP Cloud como next-hop. Si son 7 estudiantes, agregarás 6 rutas (una por cada compañero):

```bash
conf t
ip route 192.168.YY.0 255.255.255.0 <IP_CLOUD_COMPANERO_1>
ip route 192.168.ZZ.0 255.255.255.0 <IP_CLOUD_COMPANERO_2>
! Continuar una ruta por cada compañero adicional...
end
wr
```

Ejemplo concreto (si tienes 3 compañeros):
```bash
conf t
ip route 192.168.45.0 255.255.255.0 192.168.0.120
ip route 192.168.12.0 255.255.255.0 192.168.0.135
ip route 192.168.89.0 255.255.255.0 192.168.0.150
end
wr
```

### Propagación en routers internos (R3, R1, R2)

Cada router interno debe poder llegar a las redes de compañeros. Las rutas deben apuntar hacia R4 como next-hop:

En R3:
```bash
conf t
ip route 192.168.YY.0 255.255.255.0 192.168.XX.194
ip route 192.168.ZZ.0 255.255.255.0 192.168.XX.194
! Una ruta por cada red de compañero...
end
wr
```

En R1 (ruta por defecto existe desde paso d, o agregar específicas):
```bash
conf t
ip route 192.168.YY.0 255.255.255.0 192.168.XX.131
ip route 192.168.ZZ.0 255.255.255.0 192.168.XX.131
end
wr
```

En R2 (idem):
```bash
conf t
ip route 192.168.YY.0 255.255.255.0 192.168.XX.131
ip route 192.168.ZZ.0 255.255.255.0 192.168.XX.131
end
wr
```

### Coordinación con compañeros

Cada compañero debe:

1. Obtener tu IP Cloud en su interfaz `Fa0/1` ejecutando `show ip interface brief`.
2. Crear ruta de retorno hacia tu red (`192.168.XX.0/24`) usando tu IP Cloud:
   ```bash
   conf t
   ip route 192.168.XX.0 255.255.255.0 <TU_IP_CLOUD>
   end
   wr
   ```

### Verificación

En R4, verifica conexión al Cloud:
```bash
show ip interface brief
show ip route static
ping <IP_CLOUD_COMPANERO_1>
ping <IP_CLOUD_COMPANERO_2>
```

Desde tu LAN_A, prueba alcanzar un host en LAN de compañero:
```bash
ping 192.168.YY.10
trace 192.168.YY.10
```

Salida esperada del `trace`:
```
trace to 192.168.YY.10, 8 hops max
 1   192.168.XX.1      (R1)
 2   192.168.XX.131    (R3)
 3   192.168.XX.194    (R4)
 4   <IP_CLOUD_R4_COMPANERO> (R4 compañero al otro lado del Cloud)
 5   <IP_Interna_Compañero> (compañero R3 o similar)
 ... (saltos internos del compañero hasta host destino)
```

---

## (f) ¿Qué pasaría si todos los estudiantes usaran solo rutas por defecto?

### Análisis

Si todos los routers en el laboratorio configuraran **únicamente rutas por defecto** (`0.0.0.0/0`):

**Problemas esperados:**

1. **Loops de enrutamiento:** Si R4 tiene default apuntando hacia el Cloud y el Cloud reenvía el tráfico de forma indeterminada, pueden crearse ciclos.
2. **Black holes:** Tráfico destinado a redes internas que debería rechazarse en lugar de ser reenviado indefinidamente.
3. **Falta de control:** No hay camino único garantizado; el destino del tráfico depende de cómo esté configurado cada enlace.
4. **Difícil depuración:** Sin rutas específicas, los comandos como `show ip route <destino>` no indicarán claramente hacia dónde irá un paquete.

### ¿Sería posible conseguir conectividad completa con solo defaults?

**Respuesta corta:** No de manera robusta y predecible.

**Explicación:**

- En redes jerárquicas estrictas y pequeñas, podrías obtener conectividad **por casualidad**, pero no es garantizado.
- Cada router necesitaría que su default esté apuntando al "siguiente nivel" correcto, lo cual requiere acuerdo global.
- Si dos estudiantes apuntan sus defaults al mismo sitio, el otro lado no sabría devolver tráfico.
- No hay mecanismo de "convergencia" como en protocolos dinámicos (OSPF, RIP).

**Conclusión:** Rutas específicas (o enrutamiento dinámico) son necesarias para conectividad completa, confiable y escalable.

---

## (g) Proceso del router con ruta estática configurada por **interfaz de salida**

Cuando un router recibe un paquete y encuentra coincidencia con una ruta estática del tipo `ip route <red> <mascara> <interfaz>`:

### Secuencia de pasos

1. **Recepción y validación:** Router recibe trama, valida FCS y extrae el paquete IP.

2. **Decremento de TTL:** Decrementa el campo TTL del paquete IP en 1.

3. **Búsqueda en tabla:** Aplica búsqueda por prefijo más largo (Longest Prefix Match) contra la tabla de enrutamiento.

4. **Coincidencia encontrada:** Localiza ruta estática `S` especificando solo interfaz de salida:
   ```
   S   192.168.XX.64/26 [1/0] via <interfaz_salida>
   ```

5. **Determinación de interfaz:** Identifica interfaz de salida (ejemplo: `Fa0/0`).

6. **Resolución ARP (problema potencial):** En redes Ethernet multiacceso, necesita resolver **MAC destino**:
   - El router intenta resolver ARP del **destino final remoto**.
   - Ejemplo: si el destino es `192.168.XX.74` y la interfaz es Ethernet compartida, envía ARP request para `192.168.XX.74`.
   - Esto genera tráfico ARP innecesario y puede fallar si el destino no está directamente conectado.

7. **Reencapsulación:** Reencapsula el paquete en nueva trama L2 con:
   - MAC origen: dirección MAC de la interfaz de salida.
   - MAC destino: MAC resuelta por ARP (o fallida).

8. **Reenvío:** Envía la trama por la interfaz de salida.

### Riesgos en intancias Ethernet monoaccesu

- **Generación de ARP broadcast innecesarios** hacia destinos finales remotos.
- **Falta de claridad** sobre el siguiente vecino inmediato.
- **Potencial fallo** si el destino final no responde ARP en el mismo segmento L2.

---

## (h) Proceso del router con ruta estática configurada por **siguiente salto** (next-hop)

Cuando un router recibe un paquete y encuentra coincidencia con una ruta estática del tipo `ip route <red> <mascara> <next_hop>`:

### Secuencia de pasos

1. **Recepción y validación:** Router recibe trama, valida FCS y extrae el paquete IP.

2. **Decremento de TTL:** Decrementa TTL en 1.

3. **Búsqueda en tabla:** Aplica prefijo más largo en tabla de enrutamiento.

4. **Coincidencia encontrada:** Localiza ruta estática `S` especificando siguiente salto:
   ```
   S   192.168.XX.0/26 [1/0] via 192.168.XX.129 (next-hop)
   ```

5. **Búsqueda recursiva del next-hop:** Realiza nueva búsqueda recursiva para determinar **cómo alcanzar el next-hop**:
   - Si next-hop está directamente conectado (en una red propia del router), identifica la interfaz.
   - Si no, busca otra ruta hacia el next-hop.

6. **Determinación de interfaz e interfaz de salida:**
   - Una vez resuelta la interfaz para alcanzar el next-hop, la usa como interfaz de salida real.
   - Ejemplo: Para alcanzar next-hop `192.168.XX.129` (R1), es directamente conectado en `Fa0/1`.

7. **Resolución ARP:** En Ethernet, resuelve ARP del **vecino next-hop** (no del destino final):
   - Envía ARP request para dirección MAC de `192.168.XX.129` (R1).
   - Recibe respuesta con MAC de R1.
   - Esto es más eficiente y determinístico.

8. **Reencapsulación:** Reencapsula paquete en nueva trama L2 con:
   - MAC origen: dirección MAC de interfaz de salida.
   - MAC destino: MAC del next-hop.

9. **Reenvío:** Envía trama hacia el siguiente router (next-hop), quien procesa y repite el ciclo.

### Ventajas sobre interfaz de salida

- **ARP targeting correcto:** Resuelve MAC solo del vecino inmediato.
- **Menor ambigüedad:** Ruta explícitamente "saltada" a un dispositivo conocido.
- **Mejor en multiacceso:** En enlaces compartidos, garantiza entrega al vecino correcto.
- **Mejor práctica:** Recomendado en productions networks.

---

## (i) Proceso cuando hay múltiples coincidencias en la tabla (Longest Prefix Match)

Cuando un router recibe un paquete cuya dirección destino coincide con **más de una ruta**:

### Algoritmo de selección

1. **Recolección de coincidencias:** Identifica todas las rutas cuyo prefijo es compatible con el destino.

   Ejemplo (destino `192.168.XX.70`):
   - Ruta 1: `192.168.XX.0/26` ✓ (coincide)
   - Ruta 2: `192.168.0.0/8` ✓ (coincide)
   - Ruta 3: `0.0.0.0/0` (default) ✓ (coincide)

2. **Aplicación de Longest Prefix Match (LPM):** Elige la ruta con **máscara más específica** (prefijo más largo = más bits de coincidencia).

   Orden de precedencia:
   - `/26` (26 bits específicos) → ganador
   - `/8` (8 bits específicos)
   - `/0` (default, 0 bits específicos)

   **Resultado:** Se elige `192.168.XX.0/26`.

3. **Empate de prefijo (desempate):** Si dos rutas tienen el mismo prefijo/máscara:
   - Compara **distancia administrativa** (AD):
     - Connected: AD=0
     - Static: AD=1
     - RIP: AD=120
   - Gana la con menor AD.
   - Si mismo AD, compara **métrica de costo**.

4. **Falta total de coincidencia:** Si no hay ninguna ruta que coincida:
   - Genera **ICMP Destination Unreachable** (tipo 3, código 1).
   - Devuelve mensaje al origen informando destino inalcanzable.

### Ejemplo completo

Destino: `192.168.XX.70`

Tabla de enrutamiento:
```
C   192.168.XX.0/26 [0/0] via 192.168.XX.1, Fa0/0
S   192.168.XX.64/26 [1/0] via 192.168.XX.130
S   0.0.0.0/0 [1/0] via 192.168.XX.131
```

Proceso:
1. `192.168.XX.70` AND `/26` → `192.168.XX.64` (no coincide con `/26` de Connected).
2. `192.168.XX.70` AND `/26` → `192.168.XX.64` (coincide con Static `/26`).
3. `192.168.XX.70` AND `/0` → cualquier IP (coincide con default).
4. **Longest Prefix Match:** `/26` es más específica que `/0`.
5. **Resultado:** Se usa ruta Static `192.168.XX.64/26` vía `192.168.XX.130`.

---

### Paso 2. Configurar rutas estaticas internas del Lab 4

R1:
```bash
conf t
ip route 192.168.XX.64 255.255.255.192 192.168.XX.130
ip route 192.168.XX.192 255.255.255.248 192.168.XX.131
ip route 192.168.XX.200 255.255.255.248 192.168.XX.131
end
wr
```

R2:
```bash
conf t
ip route 192.168.XX.0 255.255.255.192 192.168.XX.129
ip route 192.168.XX.192 255.255.255.248 192.168.XX.131
ip route 192.168.XX.200 255.255.255.248 192.168.XX.131
end
wr
```

R3:
```bash
conf t
ip route 192.168.XX.0 255.255.255.192 192.168.XX.129
ip route 192.168.XX.64 255.255.255.192 192.168.XX.130
ip route 192.168.XX.200 255.255.255.248 192.168.XX.194
end
wr
```

R4:
```bash
conf t
ip route 192.168.XX.0 255.255.255.192 192.168.XX.193
ip route 192.168.XX.64 255.255.255.192 192.168.XX.193
ip route 192.168.XX.128 255.255.255.192 192.168.XX.193
end
wr
```

### Paso 3. Conectar Cloud a R4 (salida externa)

1. Arrastra un nodo Cloud en GNS3.
2. Asocialo a una interfaz real/virtual del host.
3. Conecta Cloud a `fa0/1` de R4 (o una interfaz libre equivalente).

R4 (lado externo al Cloud):
```bash
conf t
int fa0/1
 ip address 10.10.10.1 255.255.255.252
 no shut
end
wr
```

## 5. Conectar con otra topologia similar (XX <-> YY)

Supuesto: la otra topologia tambien tiene Lab 4 completo, pero con red `192.168.YY.0/24`.

### Paso 1. En la topologia remota (YY), conectar Cloud a R4 remoto

R4 remoto (hacia el mismo segmento de transito Cloud):
```bash
conf t
int fa0/1
 ip address 10.10.10.2 255.255.255.252
 no shut
end
wr
```

### Paso 2. Crear rutas entre las dos redes base

En R4 local (XX), ruta hacia red remota:
```bash
conf t
ip route 192.168.YY.0 255.255.255.0 10.10.10.2
end
wr
```

En R4 remoto (YY), ruta hacia red local:
```bash
conf t
ip route 192.168.XX.0 255.255.255.0 10.10.10.1
end
wr
```

### Paso 3. Propagar rutas en routers internos de cada topologia

Topologia local (XX):

R3 local:
```bash
conf t
ip route 192.168.YY.0 255.255.255.0 192.168.XX.194
end
wr
```

R1 y R2 local:
```bash
conf t
ip route 192.168.YY.0 255.255.255.0 192.168.XX.131
end
wr
```

Topologia remota (YY):

R3 remoto:
```bash
conf t
ip route 192.168.XX.0 255.255.255.0 192.168.YY.194
end
wr
```

R1 y R2 remoto:
```bash
conf t
ip route 192.168.XX.0 255.255.255.0 192.168.YY.131
end
wr
```

## 6. Como saber si la conexion funciono

### Prueba A. Estado de interfaces

En ambos lados:
```bash
show ip interface brief
```

Debe verse `up/up` en:

- enlace interno R3-R4
- interfaz externa de R4 conectada al Cloud (`fa0/1`)

### Prueba B. Ver tabla de rutas

En R4, R3, R1 y R2:
```bash
show ip route
```

Debes ver rutas a:

- `192.168.XX.0/24`
- `192.168.YY.0/24`

### Prueba C. Ping extremo a extremo

Desde un host de la red XX hacia un host de la red YY:
```bash
ping 192.168.YY.10
```

Desde un host de la red YY hacia un host de la red XX:
```bash
ping 192.168.XX.10
```

### Prueba D. Trazado de ruta

Desde red XX:
```bash
traceroute 192.168.YY.10
```

Debe pasar por R4 local, enlace Cloud/transito y luego R4 remoto.

## 7. Regla para conectar mas redes

Para conectar con otra red, repite exactamente el mismo procedimiento:

1. Sustituye `YY` por el nuevo valor (ejemplo 33, 52, 91).
2. Configura la interfaz de transito del R4 remoto.
3. Agrega rutas de ida y vuelta en R4, R3, R1 y R2.
4. Valida con `show ip route`, `ping` y `traceroute`.

## 8. Errores comunes

- Usar el mismo `XX` en dos topologias (conflicto de direccionamiento).
- Falta de ruta de retorno (ping en un solo sentido).
- Interfaz `fa0/1` de R4 en `shutdown`.
- Cloud asociado a la interfaz equivocada del host.
- Firewall del host o VM bloqueando ICMP.

---

## 9. Habilitar Telnet en Windows

### 9.1 Pasos de instalación:

1. Abre **Características de Windows** (Windows Features).
   - En Windows 10/11: Presiona `Windows + R`, escribe `windows features` y presiona Enter.
   - O accede a Panel de Control → Programas → Características de Windows.

2. Busca **Telnet Client** en la lista de características.

3. Marca la casilla para activar Telnet Client.

4. Aplica los cambios y reinicia el sistema si es necesario.

5. Abre una terminal (cmd o PowerShell) y verifica la instalación:
   ```bash
   telnet --version
   ```

La conexión por Telnet a equipos y puertos de GNS3 está documentada en la sección **3.1** (tabla de puertos).

### 9.2 Cómo conectarse por Telnet y configurar IP

#### Ejemplo práctico (LAN_A)

1. Abre una terminal (cmd o PowerShell).

2. Conéctate a LAN_A por Telnet:
   ```bash
   telnet localhost 5004
   ```

3. Se abrirá la consola de LAN_A. Ejecuta el comando de configuración:
   ```bash
   ip 192.168.XX.10/26 192.168.XX.1
   ```

4. Guarda la configuración:
   ```bash
   save
   ```

5. Verifica conectividad (ping al gateway):
   ```bash
   ping 192.168.XX.1
   ```

### 9.3 Salir de Telnet

#### Procedimiento para desconectar:

Cuando estés conectado por Telnet a un dispositivo:

1. Presiona **`Ctrl + ]`** (Control y corchete derecho cerrado).
   - Esto abre el prompt de Telnet: `telnet>`

2. Escribe:
   ```
   quit
   ```

3. Presiona **Enter** para cerrar la conexión Telnet.

La sesión Telnet se cerrará y regresarás a tu terminal (cmd/PowerShell).

### 9.4 Notas importantes sobre Telnet

- **No está encriptado:** Telnet transmite credenciales y datos en texto plano. Usar **solo en redes internas de prueba** donde la seguridad no es crítica.

- **SSH es la alternativa segura:** Para producción y acceso remoto en internet, utiliza SSH (Secure Shell) en su lugar.

- **En laboratorios educativos:** Telnet es aceptable porque la red de GNS3 es local y aislada.

- **Puertos por defecto:** Los puertos 5000-5007 en localhost suelen usarse en GNS3 para acceso local a equipos (sin exponer a internet).

- **Troubleshooting:** Si Telnet no conecta, verifica:
  1. GNS3 está en ejecución.
  2. El dispositivo está encendido en la topología.
  3. El puerto Telnet coincide con el del dispositivo.
  4. Firewall del sistema no bloquea localhost.
