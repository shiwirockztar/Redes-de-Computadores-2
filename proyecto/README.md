# Proyecto Redes 2: Optimizacion de latencia para gaming con QoS

## 1. Objetivo

Diseñar y documentar un laboratorio academico donde una red tenga congestion real de prueba y luego se aplique QoS para mejorar el trafico sensible a latencia, como el de un juego en linea tipo League of Legends.

La idea no es reproducir WTFast de forma literal, sino simular el principio tecnico que usa este tipo de servicio:

1. Seleccion de ruta con menor retardo.
2. Priorizacion del trafico interactivo frente a trafico pesado.
3. Reduccion de jitter y perdida bajo congestion.

## 2. Alcance y herramientas

- GNS3: recomendado para pruebas reales con iperf3, Ubuntu o containers.
- Cisco Packet Tracer: util para la maqueta logica y validacion basica de routing y conectividad.
- IOS en routers Cisco: para EIGRP, QoS, LLQ, CBWFQ, shaping y policing.

Nota practica: Packet Tracer tiene limitaciones con algunas funciones avanzadas de QoS. Si una politica no se comporta como en IOS real, valida la parte de QoS en GNS3 con IOSv, IOU o routers reales.

## 3. Topologia propuesta

Se propone una red con dos caminos posibles entre el cliente y el servidor, para comparar latencia y comportamiento con y sin QoS.

```text
                   Ruta A: menor retardo, mayor preferencia
PC Gamer --- SW1 --- R1 ---- R2 ---- R4 --- SW2 --- Servidor
                    \\                    /
                     \\                  /
                      ---- R3 ----------

Ruta B: mayor retardo, menor preferencia
PC Gamer --- SW1 --- R1 ---- R3 ---- R4 --- SW2 --- Servidor
```

### Roles de los equipos

- PC Gamer: genera trafico UDP pequeno y sensible a latencia.
- PC Streaming: genera trafico continuo de media prioridad.
- PC Descargas: genera trafico TCP pesado para congestionar el enlace.
- R1: router de borde del cliente.
- R2 y R3: routers intermedios.
- R4: router de borde del servidor.
- Servidor: responde al trafico de prueba y aloja el proceso iperf3.

## 4. Plan de direccionamiento IP

### LAN de cliente

- Red: 192.168.10.0/24
- Gateway: 192.168.10.1
- PC Gamer: 192.168.10.10
- PC Streaming: 192.168.10.20
- PC Descargas: 192.168.10.30

### LAN de servidor

- Red: 192.168.40.0/24
- Gateway: 192.168.40.1
- Servidor: 192.168.40.10

### Enlaces WAN

- R1-R2: 10.0.12.0/30
- R2-R4: 10.0.24.0/30
- R1-R3: 10.0.13.0/30
- R3-R4: 10.0.34.0/30

### Asignacion de interfaces

| Equipo | Interfaz | IP |
| --- | --- | --- |
| R1 | G0/0 | 192.168.10.1/24 |
| R1 | S0/0/0 | 10.0.12.1/30 |
| R1 | S0/0/1 | 10.0.13.1/30 |
| R2 | S0/0/0 | 10.0.12.2/30 |
| R2 | S0/0/1 | 10.0.24.1/30 |
| R3 | S0/0/0 | 10.0.13.2/30 |
| R3 | S0/0/1 | 10.0.34.1/30 |
| R4 | S0/0/0 | 10.0.24.2/30 |
| R4 | S0/0/1 | 10.0.34.2/30 |
| R4 | G0/0 | 192.168.40.1/24 |

## 5. Diseno de latencia y ancho de banda

La latencia no solo depende de la distancia. En este laboratorio se simula tambien el efecto del colapso de colas y la congestion del enlace.

### Ruta A: preferida por metricas

- Enlace R1-R2: bandwidth 10000 kbps, delay 5000 us
- Enlace R2-R4: bandwidth 10000 kbps, delay 5000 us

### Ruta B: alternativa menos atractiva

- Enlace R1-R3: bandwidth 5000 kbps, delay 20000 us
- Enlace R3-R4: bandwidth 5000 kbps, delay 15000 us

Con EIGRP, la ruta A debe ganar por tener menor delay y mayor bandwidth. Si luego se congestiona, QoS ayudara a que el trafico de gaming no quede atrapado en cola.

## 6. Escenario de trafico a simular

### Trafico 1: gaming UDP

- Flujo pequeno, constante y sensible a jitter.
- Puerto sugerido: UDP 5001.
- Objetivo: simular paquetes cortos, frecuentes y criticos.

### Trafico 2: streaming

- Flujo sostenido, tolera algo mas de retardo que el juego.
- Puerto sugerido: TCP 5002 o UDP 5002.

### Trafico 3: descargas

- Trafico TCP pesado.
- Puerto sugerido: TCP 5003.

### Como generar trafico

- En GNS3 usa una VM Ubuntu o un container con iperf3.
- En Packet Tracer usa ping, traceroute y pruebas basicas de conectividad; para iperf3 hace falta un host real o virtual con Linux.

Ejemplo en el servidor:

```bash
iperf3 -s
```

Ejemplo de trafico gaming desde el cliente:

```bash
iperf3 -u -c 192.168.40.10 -p 5001 -b 1M -l 120 -t 30
```

Ejemplo de streaming:

```bash
iperf3 -u -c 192.168.40.10 -p 5002 -b 8M -l 120 -t 60
```

Ejemplo de descarga pesada:

```bash
iperf3 -c 192.168.40.10 -p 5003 -P 4 -t 60
```

## 7. Escenario comparativo

### Caso A: sin QoS

1. Todos los flujos comparten el mismo enlace de salida.
2. El trafico de descargas satura la cola.
3. El trafico de gaming sufre aumento de RTT, jitter y perdida.

### Caso B: con QoS

1. El trafico de gaming se marca como EF.
2. Streaming recibe una clase intermedia.
3. Descargas quedan como trafico best effort o con policer.
4. La cola de baja latencia protege al trafico sensible.

## 8. Configuracion base de routing con EIGRP

Se usa EIGRP porque su metrica considera bandwidth y delay, lo cual encaja bien con el objetivo del proyecto.

### R1

```ios
hostname R1
no ip domain-lookup

interface g0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface s0/0/0
 ip address 10.0.12.1 255.255.255.252
 bandwidth 10000
 delay 5000
 clock rate 64000
 no shutdown

interface s0/0/1
 ip address 10.0.13.1 255.255.255.252
 bandwidth 5000
 delay 20000
 clock rate 64000
 no shutdown

router eigrp 100
 no auto-summary
 network 192.168.10.0 0.0.0.255
 network 10.0.12.0 0.0.0.3
 network 10.0.13.0 0.0.0.3
 passive-interface g0/0
```

### R2

```ios
hostname R2
no ip domain-lookup

interface s0/0/0
 ip address 10.0.12.2 255.255.255.252
 bandwidth 10000
 delay 5000
 no shutdown

interface s0/0/1
 ip address 10.0.24.1 255.255.255.252
 bandwidth 10000
 delay 5000
 clock rate 64000
 no shutdown

router eigrp 100
 no auto-summary
 network 10.0.12.0 0.0.0.3
 network 10.0.24.0 0.0.0.3
```

### R3

```ios
hostname R3
no ip domain-lookup

interface s0/0/0
 ip address 10.0.13.2 255.255.255.252
 bandwidth 5000
 delay 20000
 no shutdown

interface s0/0/1
 ip address 10.0.34.1 255.255.255.252
 bandwidth 5000
 delay 15000
 clock rate 64000
 no shutdown

router eigrp 100
 no auto-summary
 network 10.0.13.0 0.0.0.3
 network 10.0.34.0 0.0.0.3
```

### R4

```ios
hostname R4
no ip domain-lookup

interface s0/0/0
 ip address 10.0.24.2 255.255.255.252
 bandwidth 10000
 delay 5000
 no shutdown

interface s0/0/1
 ip address 10.0.34.2 255.255.255.252
 bandwidth 5000
 delay 15000
 no shutdown

interface g0/0
 ip address 192.168.40.1 255.255.255.0
 no shutdown

router eigrp 100
 no auto-summary
 network 10.0.24.0 0.0.0.3
 network 10.0.34.0 0.0.0.3
 network 192.168.40.0 0.0.0.255
 passive-interface g0/0
```

## 9. Clasificacion y marcado de trafico

La clasificacion debe hacerse en el borde de entrada, cerca del cliente, para marcar antes de que el trafico entre al core.

### ACL de trafico

```ios
ip access-list extended ACL-GAME
 permit udp any any eq 5001

ip access-list extended ACL-VIDEO
 permit udp any any eq 5002
 permit tcp any any eq 5002

ip access-list extended ACL-BULK
 permit tcp any any eq 5003
```

### Class-map

```ios
class-map match-any CM-GAME
 match access-group name ACL-GAME

class-map match-any CM-VIDEO
 match access-group name ACL-VIDEO

class-map match-any CM-BULK
 match access-group name ACL-BULK
```

### Policy de marcado

```ios
policy-map PM-MARKING
 class CM-GAME
  set dscp ef
 class CM-VIDEO
  set dscp af41
 class CM-BULK
  set dscp cs1
```

Aplicacion sugerida en R1, sobre la interfaz que recibe trafico desde la LAN:

```ios
interface g0/0
 service-policy input PM-MARKING
```

## 10. CBWFQ y LLQ

Una vez marcado el trafico, se crea una politica de salida para la WAN.

### Politica de cola

```ios
policy-map PM-WAN-CHILD
 class CM-GAME
  priority percent 20
 class CM-VIDEO
  bandwidth percent 30
 class CM-BULK
  bandwidth percent 10
 class class-default
  fair-queue
```

Interpretacion:

- `priority percent 20` crea la LLQ para el trafico de juego.
- `bandwidth percent 30` reserva ancho para streaming.
- `bandwidth percent 10` limita la atencion minima del bulk.
- `fair-queue` deja el resto al comportamiento general.

## 11. Traffic shaping

Si el enlace WAN real es mas rapido que el cuello de botella logico, conviene moldear el trafico para evitar drops mas adelante.

Ejemplo de politica jerarquica en R1 hacia la WAN:

```ios
policy-map PM-PARENT-SHAPE
 class class-default
  shape average 8000000
  service-policy PM-WAN-CHILD
```

Aplicacion en la interfaz de salida de R1 hacia la ruta preferida:

```ios
interface s0/0/0
 service-policy output PM-PARENT-SHAPE
```

La idea es que todo el trafico pase por una cola controlada antes de llegar al enlace saturable.

## 12. Policing

El policing sirve para cortar trafico excesivo, sobre todo descargas que no deben dominar el enlace.

Ejemplo simple para bulk:

```ios
policy-map PM-POLICE-BULK
 class CM-BULK
  police 2000000 conform-action transmit exceed-action drop
```

Aplicacion posible en el borde de entrada o en la salida hacia el servidor si deseas restringir abuso de ancho de banda.

## 13. Flujo recomendado de implementacion

1. Monta la topologia fisica y asigna IPs.
2. Verifica conectividad basica con ping.
3. Configura EIGRP y comprueba la tabla de rutas.
4. Ajusta bandwidth y delay de interfaces para crear la ruta preferida.
5. Genera trafico pesado sin QoS y toma medidas.
6. Aplica marcado, LLQ, CBWFQ, shaping y policing.
7. Repite las pruebas y compara resultados.

## 14. Comandos de validacion

### Routing

```ios
show ip route
show ip eigrp topology
show ip eigrp neighbors
```

### Interfaces y metricas

```ios
show interfaces s0/0/0
show interfaces s0/0/1
show running-config interface s0/0/0
```

### QoS

```ios
show policy-map interface s0/0/0
show policy-map interface g0/0
show access-lists
```

### Pruebas desde el host

```bash
ping 192.168.40.10
traceroute 192.168.40.10
```

## 15. Metricas a reportar

### Latencia

Usa `ping` para reportar RTT promedio, minimo y maximo.

### Jitter

Usa `iperf3 -u`. El receptor mostrara jitter y datagramas perdidos.

### Perdida de paquetes

Reporta la cantidad y porcentaje de paquetes perdidos en pruebas UDP.

### Interpretacion esperada

- Sin QoS: mayor RTT bajo carga, jitter alto, posibles perdidas.
- Con QoS: el RTT del trafico de juego se vuelve mas estable, el jitter cae y las perdidas disminuyen en ese flujo.

Importante: QoS no elimina la latencia fisica de la red. Lo que mejora es la latencia de cola, es decir, el tiempo que el trafico espera dentro del router cuando hay congestion.

## 16. Como presentar el proyecto

Estructura recomendada para la sustentacion:

1. Problema: la congestion afecta juegos en tiempo real.
2. Modelo: topologia con dos rutas y enlaces de distinta calidad.
3. Metodologia: baseline sin QoS, luego QoS con marcado y colas.
4. Evidencia: capturas de ping, traceroute, iperf3 y `show policy-map interface`.
5. Resultado: comparacion antes y despues.
6. Conclusion: la optimizacion no reduce la distancia geografica, pero si la espera en colas y la variabilidad.

### Justificacion tecnica corta

Un juego en linea necesita consistencia temporal. Cuando una cola se satura por trafico masivo, los paquetes del juego compiten con descargas y streaming. LLQ da trato preferente al trafico critico, CBWFQ reparte el resto de forma controlada, shaping evita rafagas destructivas y policing protege contra flujos abusivos.

## 17. Resultado esperado para el informe

Incluye una tabla antes y despues con valores medidos, por ejemplo:

| Escenario | RTT promedio | Jitter | Perdida |
| --- | ---: | ---: | ---: |
| Sin QoS | Alto | Alto | Presente |
| Con QoS | Menor o mas estable | Menor | Muy baja o nula |

El punto clave del analisis es que el trafico de gaming deja de pelear contra descargas pesadas en la misma cola.

## 18. Sugerencia final de implementacion

Si quieres que el laboratorio quede mas cercano a una solucion profesional, hazlo en dos fases:

1. Fase 1: Packet Tracer para documentar la topologia y el routing.
2. Fase 2: GNS3 para ejecutar iperf3, medir jitter real y capturar evidencia de QoS.

Si luego quieres, se puede generar una version agresiva con las configuraciones completas por router, lista para copiar y pegar en IOS.

## 19. Configuraciones completas listas para copiar

Este anexo junta la configuracion final por dispositivo. El orden recomendado es:

1. Configurar primero IPs e interfaces.
2. Activar EIGRP.
3. Verificar rutas.
4. Aplicar marcado QoS en R1.
5. Aplicar la politica de salida en la WAN de R1.

### R1 completo

```ios
enable
configure terminal
hostname R1
no ip domain-lookup

interface g0/0
 description LAN CLIENTE
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface s0/0/0
 description ENLACE A R2
 ip address 10.0.12.1 255.255.255.252
 bandwidth 10000
 delay 5000
 clock rate 64000
 no shutdown

interface s0/0/1
 description ENLACE A R3
 ip address 10.0.13.1 255.255.255.252
 bandwidth 5000
 delay 20000
 clock rate 64000
 no shutdown

ip access-list extended ACL-GAME
 permit udp any any eq 5001

ip access-list extended ACL-VIDEO
 permit udp any any eq 5002
 permit tcp any any eq 5002

ip access-list extended ACL-BULK
 permit tcp any any eq 5003

class-map match-any CM-GAME
 match access-group name ACL-GAME

class-map match-any CM-VIDEO
 match access-group name ACL-VIDEO

class-map match-any CM-BULK
 match access-group name ACL-BULK

policy-map PM-MARKING
 class CM-GAME
  set dscp ef
 class CM-VIDEO
  set dscp af41
 class CM-BULK
  set dscp cs1

policy-map PM-WAN-CHILD
 class CM-GAME
  priority percent 20
 class CM-VIDEO
  bandwidth percent 30
 class CM-BULK
  bandwidth percent 10
 class class-default
  fair-queue

policy-map PM-PARENT-SHAPE
 class class-default
  shape average 8000000
  service-policy PM-WAN-CHILD

router eigrp 100
 no auto-summary
 network 192.168.10.0 0.0.0.255
 network 10.0.12.0 0.0.0.3
 network 10.0.13.0 0.0.0.3
 passive-interface g0/0

interface g0/0
 service-policy input PM-MARKING

interface s0/0/0
 service-policy output PM-PARENT-SHAPE

end
write memory
```

### R2 completo

```ios
enable
configure terminal
hostname R2
no ip domain-lookup

interface s0/0/0
 description ENLACE A R1
 ip address 10.0.12.2 255.255.255.252
 bandwidth 10000
 delay 5000
 no shutdown

interface s0/0/1
 description ENLACE A R4
 ip address 10.0.24.1 255.255.255.252
 bandwidth 10000
 delay 5000
 clock rate 64000
 no shutdown

router eigrp 100
 no auto-summary
 network 10.0.12.0 0.0.0.3
 network 10.0.24.0 0.0.0.3

end
write memory
```

### R3 completo

```ios
enable
configure terminal
hostname R3
no ip domain-lookup

interface s0/0/0
 description ENLACE A R1
 ip address 10.0.13.2 255.255.255.252
 bandwidth 5000
 delay 20000
 no shutdown

interface s0/0/1
 description ENLACE A R4
 ip address 10.0.34.1 255.255.255.252
 bandwidth 5000
 delay 15000
 clock rate 64000
 no shutdown

router eigrp 100
 no auto-summary
 network 10.0.13.0 0.0.0.3
 network 10.0.34.0 0.0.0.3

end
write memory
```

### R4 completo

```ios
enable
configure terminal
hostname R4
no ip domain-lookup

interface s0/0/0
 description ENLACE A R2
 ip address 10.0.24.2 255.255.255.252
 bandwidth 10000
 delay 5000
 no shutdown

interface s0/0/1
 description ENLACE A R3
 ip address 10.0.34.2 255.255.255.252
 bandwidth 5000
 delay 15000
 no shutdown

interface g0/0
 description LAN SERVIDOR
 ip address 192.168.40.1 255.255.255.0
 no shutdown

router eigrp 100
 no auto-summary
 network 10.0.24.0 0.0.0.3
 network 10.0.34.0 0.0.0.3
 network 192.168.40.0 0.0.0.255
 passive-interface g0/0

end
write memory
```

### Configuracion de hosts

PC Gamer:

```text
IP: 192.168.10.10
Mask: 255.255.255.0
Gateway: 192.168.10.1
```

PC Streaming:

```text
IP: 192.168.10.20
Mask: 255.255.255.0
Gateway: 192.168.10.1
```

PC Descargas:

```text
IP: 192.168.10.30
Mask: 255.255.255.0
Gateway: 192.168.10.1
```

Servidor:

```text
IP: 192.168.40.10
Mask: 255.255.255.0
Gateway: 192.168.40.1
```

## 20. Secuencia de pruebas sugerida

1. Desde R1 ejecuta `show ip eigrp neighbors` para confirmar adyacencias.
2. Desde R1 ejecuta `show ip route` y verifica que la ruta preferida sea la de R2.
3. Desde el PC Gamer ejecuta `ping 192.168.40.10` durante una descarga TCP pesada.
4. Levanta `iperf3` en el servidor.
5. Corre el flujo de gaming UDP y registra jitter.
6. Repite la prueba con QoS habilitado y compara.

## 21. Que deberias observar

- Sin QoS, el flujo UDP sensible tendra variacion mayor en RTT y perdida cuando el enlace este cargado.
- Con QoS, el trafico EF debe conservar prioridad sobre los flujos bulk.
- La salida de `show policy-map interface s0/0/0` debe mostrar contadores de clases y el uso de la LLQ.

## 22. Nota de laboratorio

Si Packet Tracer no acepta alguna politica jerarquica, conserva la logica del proyecto con EIGRP, marcado DSCP y colas simples. En GNS3 podras aplicar la version completa sin las limitaciones del simulador.

## 23. Guia de verificacion paso a paso

Esta seccion resume que deberias ver en cada fase y como confirmar que todo funciona.

| Paso | Que deberias ver | Como verificarlo |
| --- | --- | --- |
| 1. Topologia fisica | Los 4 routers, 2 switches, 3 clientes y 1 servidor conectados segun el diagrama | En Packet Tracer o GNS3 revisa cada enlace y confirma que no haya puertos apagados |
| 2. Direccionamiento IP | Cada interfaz del router y cada host con la IP correcta | En routers usa `show ip interface brief`; en PCs revisa configuracion IP manual |
| 3. Enlaces WAN | R1-R2 y R2-R4 con mejores metricas que R1-R3 y R3-R4 | En routers usa `show running-config interface s0/0/x` y confirma `bandwidth` y `delay` |
| 4. Routing EIGRP | Debe aparecer la red remota del servidor y la ruta preferida por R2 | Usa `show ip route` y `show ip eigrp topology`; la ruta via R2 debe verse como la principal |
| 5. Conectividad base | El cliente debe poder llegar al servidor sin QoS | Haz `ping 192.168.40.10` y `traceroute 192.168.40.10` desde el cliente |
| 6. Trafico de carga | El servidor debe recibir sesiones iperf3 simultaneas de gaming, streaming y bulk | Levanta `iperf3 -s` en el servidor y ejecuta los tres clientes; verifica que haya trafico activo |
| 7. Sin QoS | El ping sube de forma visible cuando la descarga satura el enlace | Ejecuta ping continuo durante una descarga y compara RTT, variacion y perdida |
| 8. Marcado DSCP | El trafico de gaming debe salir marcado como EF, streaming como AF41 y bulk como CS1 | En R1 usa `show policy-map interface g0/0` y revisa que aumenten los contadores de cada clase |
| 9. LLQ y CBWFQ | El trafico de juego debe tener prioridad y el streaming una cola con ancho reservado | En R1 usa `show policy-map interface s0/0/0` y confirma que la clase `CM-GAME` tenga actividad en la cola prioritaria |
| 10. Shaping | La salida WAN debe verse controlada, sin picos violentos | Revisa `show policy-map interface s0/0/0`; el trafico debe salir dentro del limite configurado |
| 11. Policing | El trafico bulk no debe dominar el enlace | En la politica correspondiente, confirma contadores de drops o excedidos para `CM-BULK` |
| 12. Con QoS | El ping del trafico de gaming debe estabilizarse aunque sigan las descargas | Repite la prueba y compara con el caso sin QoS; el RTT deberia ser mas consistente |

### Lectura esperada de resultados

- Si `show ip route` no muestra la red del servidor, el problema esta en el routing EIGRP o en alguna interfaz apagada.
- Si `ping` falla, revisa primero IP, gateway y estado de interfaces antes de revisar QoS.
- Si EIGRP funciona pero el trafico sigue yendo por la ruta menos preferida, revisa los valores de `bandwidth` y `delay`.
- Si `show policy-map interface` no aumenta contadores, revisa que la ACL coincida con el puerto correcto y que la policy este aplicada en la interfaz correcta.
- Si no ves mejora en RTT, probablemente no hay congestion suficiente; aumenta el trafico bulk o reduce temporalmente el ancho de banda del enlace.

### Criterio minimo de exito

El laboratorio se considera correcto si puedes demostrar estas tres cosas:

1. Antes de QoS, el trafico de gaming empeora bajo carga.
2. Con QoS, la clase de gaming recibe prioridad y su comportamiento mejora.
3. Los comandos de verificacion muestran contadores coherentes con el trafico generado.