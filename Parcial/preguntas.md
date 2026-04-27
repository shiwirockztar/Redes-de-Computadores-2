# Preguntas y respuestas - Parcial

Este documento organiza las dudas frecuentes del parcial y su explicacion tecnica.

## 1) ¿Por que al apagar G0/1 en R1 falla el ping entre VLAN de administracion y VLAN 10?

### Situacion observada

- Si se elimina o apaga G0/1 en R1, el ping entre un PC de administracion y un PC en VLAN 10 falla.
- Si se elimina o apaga G0/0 en R1, ese mismo ping puede seguir funcionando.

### Explicacion tecnica

Segun el diseno consolidado del parcial (puntos 1-2 hasta 6), el rol de interfaces en los routers es este:

- G0/0: Router-on-a-Stick para VLAN 10, 20 y 30 (subinterfaces dot1Q).
- G0/1: interfaz fisica dedicada a VLAN 40 (administracion), sin subinterfaces.

Por eso, si se apaga G0/1 en R1, R1 pierde la salida fisica que usa para la VLAN 40. Aunque exista R2 como respaldo, ese respaldo solo funciona si R2 esta configurado para tomar el rol de gateway de esa VLAN y la topologia ya convergio. Si eso no pasa todavia, el ping falla.

En cambio, apagar G0/0 en R1 no siempre tumba inmediatamente ese ping en todos los casos, porque el trafico de VLAN 10 puede seguir entrando por la ruta de respaldo que tiene HSRP con R2. Es decir, el respaldo existe para el gateway virtual, no para cualquier interfaz apagada.

Resumen simple: R2 no es un respaldo "magico". Solo toma el control si la VLAN afectada tiene su gateway virtual, su configuracion HSRP y su camino de capa 2 y 3 listos para hacerlo.

### ¿Como hago que R2 asuma correctamente el cargo de respaldo?

Para que R2 sirva como respaldo real, debe cumplir estos **requisitos previos obligatorios**:

#### 1) R2 debe tener configuradas LAS MISMAS subinterfaces y VLAN que R1

**En R2 debes replicar:**

```
interface g0/0
 no ip address
 no shutdown

interface g0/0.10
 encapsulation dot1Q 10
 ip address 10.10.8.1 255.255.255.0
 no shutdown

interface g0/0.20
 encapsulation dot1Q 20
 ip address 10.10.8.65 255.255.255.0
 no shutdown

interface g0/0.30
 encapsulation dot1Q 30
 ip address 10.10.8.129 255.255.255.0
 no shutdown

interface g0/1
 ip address 10.10.8.193 255.255.255.0
 no shutdown
```

**Nota**: R1 y R2 pueden tener **addresses HSRP diferentes**, pero **las subinterfaces deben existir en ambos** para que el trafico inter-VLAN funcione.

#### 2) R2 debe tener HSRP correctamente configurado

HSRP (Hot Standby Routing Protocol) es lo que determina cuál router es activo. Segun el parcial:

- **R1 activo en VLAN 10 y 30** (prioridad 150 o mas)
- **R2 activo en VLAN 20 y 40** (prioridad 150 o mas)

**Configuracion HSRP en R2 para VLAN 40 (administracion):**

```
interface g0/1
 standby 40 ip 10.10.8.194
 standby 40 priority 150
 standby 40 preempt
```

**Configuracion HSRP en R2 para VLAN 20:**

```
interface g0/0.20
 standby 20 ip 10.10.8.66
 standby 20 priority 150
 standby 20 preempt
```

**Configuracion HSRP en R2 para VLAN 10 y 30 (standby/respaldo):**

```
interface g0/0.10
 standby 10 ip 10.10.8.2
 standby 10 priority 100
 standby 10 preempt

interface g0/0.30
 standby 30 ip 10.10.8.130
 standby 30 priority 100
 standby 30 preempt
```

#### 3) Verificar que HSRP esta activo

```
show standby brief
```

Debes ver algo como:

```
Interface   Grp  Pri  P State    Active            Standby
g0/0.10     10   100  P Standby 10.10.8.1         10.10.8.2
g0/0.20     20   150  P Active  10.10.8.65        10.10.8.66
g0/0.30     30   100  P Standby 10.10.8.129       10.10.8.130
g0/1        40   150  P Active  10.10.8.193       10.10.8.194
```

- **Active**: R2 es el activo en esa VLAN.
- **Standby**: R2 esta en espera, R1 es activo.

#### 4) La capa 2 (switch) TAMBIÉN debe estar lista

El switch debe permitir que ambos routers reciban trafico:

```
Puerto hacia R1 (g0/0):
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30

Puerto hacia R1 (g0/1):
 switchport mode access
 switchport access vlan 40

Puerto hacia R2 (g0/0):
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30

Puerto hacia R2 (g0/1):
 switchport mode access
 switchport access vlan 40
```

Ambos routers deben estar accesibles por capa 2 desde el switch. Si el puerto de R2 esta shutdown o en vlan incorrecta, R2 nunca podra tomar el rol activo.

#### 5) Red convergia = recien ahi R2 toma el control

El flujo que sucede al apagar G0/1 en R1:

1. G0/1 de R1 se apaga.
2. R1 detiene sus anuncios HSRP (heartbeats).
3. R2 espera el **dead time** (default 10 segundos).
4. R2 se declara a si mismo ACTIVO porque R1 no responde.
5. Desde ese momento, **R2 recibe todo el trafico de VLAN 40**.

**Esto solo ocurre si:**
- R2 tiene subinterfaces de VLAN 40 configuradas.
- R2 tiene HSRP activo con prioridad >= 150.
- El switch permite trafico a G0/1 de R2.
- El dead time paso (10 segundos default).

#### 6) Resumen del checklist para que R2 sea respaldo real

| Elemento | ¿Está? | ¿Cómo verificar? |
|----------|--------|-----------------|
| R2 tiene G0/0 en estado up/up | ✓ | `show ip int brief` |
| R2 tiene subinterfaces G0/0.10,20,30 | ✓ | `show running-config int g0/0.10` |
| R2 tiene G0/1 en estado up/up | ✓ | `show ip int brief` |
| HSRP configurado en R2 | ✓ | `show standby brief` |
| HSRP con `preempt` activado | ✓ | `show running-config int g0/0.20` |
| Switch permite trafico a R2 G0/0 | ✓ | `show interfaces trunk` |
| Switch permite trafico a R2 G0/1 | ✓ | `show vlan brief` |
| HSRP estados son Active o Standby | ✓ | `show standby brief` |

Si alguno falla, R2 no asumira el rol.

#### 7) Prueba final

Despues de configurar todo, verifica el failover:

```
R2# show standby brief   (anota qué es activo en cada VLAN)

Luego en R1:
R1# interface g0/1
R1(config-if)# shutdown

Ahora vuelve a R2 y corre:
R2# show standby brief   (deberia ver que g0/1 ahora es Active)

Y hace ping desde un PC de VLAN 40 a un PC de VLAN 10.
Deberia funcionar porque R2 ahora es el gateway activo de VLAN 40.
```

Si el ping funciona despues de que R1 cae, entonces R2 esta listo como respaldo real.

### Ejemplo facil de envio de tramas

Supongamos que un PC de administracion quiere hacer ping a un PC en VLAN 10:

1. El PC de administracion crea una trama Ethernet con:
	- MAC origen: la MAC del PC.
	- MAC destino: la MAC del gateway virtual de VLAN 40 o del router activo.
	- IP origen: la IP del PC de administracion.
	- IP destino: la IP del PC en VLAN 10.
2. El switch envia esa trama hacia el router que tenga activo el gateway.
3. Si G0/1 de R1 esta activo, R1 recibe la trama, la enruta hacia VLAN 10 y la vuelve a encapsular con una nueva trama para esa VLAN.
4. El PC de VLAN 10 responde con otra trama de vuelta hacia su gateway.
5. Si G0/1 de R1 esta apagado, pero R2 no ha tomado correctamente el rol de gateway de VLAN 40, la primera trama ya no tiene por donde salir y el ping falla.
6. Si R2 si tomo el rol de respaldo, entonces la trama entra por R2, cruza la ruta activa y el ping sigue funcionando.

## 2) Verificacion rapida recomendada

En R1:

1. `show ip interface brief`
2. `show running-config interface g0/1`
3. `show running-config interface g0/0.10`
4. `show running-config interface g0/0.20`
5. `show running-config interface g0/0.30`
6. `show standby brief`
7. `show ip arp`
8. `show ip route`

En el switch conectado a R1:

1. `show interfaces trunk`
2. `show vlan brief`
3. `show etherchannel summary`
4. `show spanning-tree vlan 10`
5. `show spanning-tree vlan 40`

## 3) ¿Que debe estar correcto para que funcione?

- Subinterfaces o interfaz de gateway en estado up/up.
- `encapsulation dot1Q` correcta por VLAN (si aplica).
- IP de gateway correcta en cada VLAN.
- Puerto hacia G0/0 en modo trunk con VLAN 10,20,30 permitidas y nativa coherente.
- Puerto hacia G0/1 en modo access VLAN 40 (sin subinterfaces para VLAN 40).
- Puerta de enlace correcta configurada en los PCs.
- HSRP operativo y con prioridades segun el parcial (R1 principal en 10/30 y R2 principal en 20/40).
- Consistencia con STP/EtherChannel para evitar bloqueos o caminos inesperados en la capa 2.

## 4) Coherencia con la secuencia del parcial (README 1-2 a 6)

- Punto 1-2: base de VLAN, troncales, gestion y acceso remoto.
- Punto 3: control del arbol STP por VLAN.
- Punto 4: capacidad con EtherChannel para evitar saturacion.
- Punto 5: direccionamiento exclusivo 10.10.8.0/24.
- Punto 6: inter-VLAN redundante con dos routers y HSRP, con VLAN 40 por G0/1 fisica.

La interpretacion de esta pregunta debe leerse con esa secuencia completa, no como un caso aislado.

## 5) Conclusión

El comportamiento reportado es coherente con el diseno: al apagar la interfaz que presta servicio de gateway a las VLAN involucradas, el trafico inter-VLAN deja de funcionar. Si se apaga una interfaz no usada en ese trayecto, el ping se mantiene.

## 6) Respuesta corta para sustentacion oral

"En este parcial, G0/0 en los routers transporta VLAN 10, 20 y 30 por subinterfaces dot1Q, mientras que VLAN 40 va por G0/1 fisica, sin subinterfaces. Si bajo G0/1 en R1, elimino la salida directa de R1 para administracion. R2 solo reemplaza ese camino si HSRP ya le dio el rol activo y la red convergio. Si bajo G0/0 en R1, no siempre se cae todo porque el gateway virtual puede seguir por R2. En resumen, no basta con tener un respaldo: ese respaldo debe estar activo, configurado y alcanzable por capa 2 y 3." 

## 7) Como se podria corregir si falla el ping

Si al apagar o recuperar interfaces el ping entre VLAN 10 y VLAN 40 no responde, aplicar esta secuencia:

1. Verificar estado de interfaces en ambos routers:
	- `show ip interface brief`
	- Corregir interfaces caidas con `no shutdown` en `g0/0`, subinterfaces `g0/0.x` y `g0/1` segun el caso.

2. Validar configuracion de VLAN por interfaz en router:
	- En G0/0 deben existir subinterfaces de VLAN 10,20,30 con `encapsulation dot1Q` correcta.
	- En G0/1 debe existir IP de VLAN 40 fisica, sin subinterfaces.

3. Confirmar HSRP y roles esperado del parcial:
	- `show standby brief`
	- Esperado: R1 activo en VLAN 10/30 y R2 activo en VLAN 20/40 (con `preempt`).

4. Revisar puertos de switch hacia routers:
	- Puerto hacia G0/0: `switchport mode trunk`, VLAN 10,20,30 permitidas, nativa coherente.
	- Puerto hacia G0/1: `switchport mode access`, `switchport access vlan 40`.
	- Comandos utiles: `show interfaces trunk`, `show vlan brief`.

5. Verificar capa 2 del parcial (STP/EtherChannel):
	- `show spanning-tree vlan 10`
	- `show spanning-tree vlan 40`
	- `show etherchannel summary`
	- Corregir puertos en estado inconsistente, bloqueo inesperado o channel-group mal armado.

6. Confirmar direccionamiento de hosts y gateways:
	- Host VLAN 10 con gateway `10.10.8.1`.
	- Host VLAN 40 con gateway `10.10.8.193`.
	- Todo dentro de `10.10.8.0/24`.

7. Limpiar aprendizaje temporal y reintentar prueba:
	- `clear arp` (router)
	- repetir ping entre hosts de VLAN 10 y 40.

Regla practica: primero recupera capa 1/2 (interfaces, trunk, access, STP, EtherChannel) y luego capa 3 (subinterfaces, IPs y HSRP). Asi se resuelve mas rapido y con menor riesgo de cambiar algo que ya estaba bien.

## 8) ¿Por qué al hacer ping hay inundaciones de ICMP al principio?

### Situacion observada

- Al hacer un primer ping entre dos VLAN, se ven multiples copias del ICMP request/reply en la captura de trafico.
- Es normal ver inundaciones ARP, pero ¿por qué también aparecen inundaciones ICMP?
- Despues de los primeros pings, la inundacion desaparece.

### Explicacion tecnica

La "inundacion ICMP" que observas **no es un problema de ICMP**, sino una consecuencia del **flooding de capa 2** (switching) cuando el switch desconoce las MAC destino.

**Flujo del comportamiento:**

1. El PC origen envia un **ICMP request unicast** hacia la MAC del gateway o del destino.
2. El switch **no tiene esa MAC en su tabla CAM** (o la entrada expiro).
3. El switch hace **flooding** del frame por todos los puertos de la VLAN (excepto el de entrada).
4. Cada puerto que recibe el frame lo entrega a su VLAN, causando multiples copias del mismo ICMP.
5. Los destinos responden con multiples **ICMP replies**.
6. Despues de esto, el switch **aprende las MAC** en su tabla CAM y el flooding se detiene.

### ¿Por qué se ve con ICMP y no solo con ARP?

- **ARP flood es intencional**: ARP usa destinatario broadcast (FF-FF-FF-FF-FF-FF), es decir, esta diseñado para llegar a todos.
- **ICMP flood es consecuencia de MAC desconocida**: ICMP va unicast con una MAC especifica, pero si el switch no conoce esa MAC, hace flooding.

### Esto es completamente normal cuando:

- El switch acaba de arrancar o cargó la topologia.
- La tabla MAC del switch no esta llena o expiro la entrada.
- Hay convergencia de STP en curso.
- Interfaces de router o switch acaban de levantarse.
- Es el primer ping entre dos hosts en diferentes VLAN.

### Para verificar que es esto en la captura:

- **ICMP request unico** enviado por el origen.
- **Multiples copias idénticas del ICMP request** llegando al destino (flooding del switch).
- **Multiples ICMP replies** volviendo hacia el origen.
- Despues de algunos pings, todas las MAC estan aprendidas y el flooding cesa.

### Respuesta corta para sustentacion oral

"El flooding ICMP que se ve al principio no es un fallo de ICMP ni del router. Es el switch haciendo flooding en capa 2 porque no conoce la MAC destino. ARP flood es intencional (broadcast), pero ICMP va unicast, entonces cuando el switch no lo conoce, lo inunda por todos los puertos de la VLAN. Una vez que el switch aprende las MAC en su tabla CAM (despues de los primeros pings), el flooding desaparece. Es normal y esperado durante la fase de convergencia de la red."

### Como evitarlo o minimizarlo:

1. Usar `portfast` en puertos de acceso de PCs (no en puertos troncales).
2. Asegurar que STP converja correctamente: `show spanning-tree vlan X`.
3. Mantener las MAC aging time adecuadas (default 300s es suficiente).
4. En topologias muy grandes, usar `bpdu guard` y `root guard` para estabilidad.

La practica muestra que esto **desaparece naturalmente** despues de los primeros intercambios, confirmando que es solo un fallo temporal de aprendizaje de MAC, no un problema estructural.