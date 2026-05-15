# Proyecto Redes 2: Optimizacion de latencia para gaming con QoS

## 1. Objetivo

Diseñar y documentar un laboratorio academico donde una red tenga congestion real de prueba y luego se aplique QoS para mejorar el trafico sensible a latencia, como el de un juego en linea tipo League of Legends.

La idea no es reproducir WTFast de forma literal, sino simular el principio tecnico que usa este tipo de servicio:

1. Seleccion de ruta con menor retardo.
2. Priorizacion del trafico interactivo frente a trafico pesado.
3. Reduccion de jitter y perdida bajo congestion.

## 2. Alcance y herramientas

**Herramienta principal: GNS3** con máquinas virtuales reales para simulación de tráfico.

### Componentes necesarios

- **GNS3**: emulador de red para simular topología con routers Cisco
- **IOSv o IOU**: imágenes Cisco IOS virtualizadas en GNS3
- **Ubuntu/Debian containers o VMs**: para generar tráfico real con iperf3
- **IOS en routers Cisco**: para OSPF, QoS, LLQ, CBWFQ, shaping y policing

### ¿Por qué GNS3 y no Packet Tracer?

- Packet Tracer tiene limitaciones severas con QoS avanzado y simulación de tráfico real
- GNS3 permite usar iperf3 en máquinas virtuales Linux para pruebas realistas
- Los resultados de QoS en GNS3 son reproducibles en equipos Cisco reales
- Mejor simulación de latencia, jitter y congestión

### ¿Por qué OSPF y no EIGRP?

- OSPF es estándar abierto (RFC 2328) vs EIGRP propietario de Cisco
- Facilita interoperabilidad en redes reales
- El objetivo del proyecto es optimización con QoS, no maximizar eficiencia de routing
- OSPF permite control fino de métricas mediante `cost` manual en interfaces

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

La latencia no solo depende de la distancia. En este laboratorio se simula mediante:
- Delay configurado en interfaces
- Congestión real de tráfico desde máquinas virtuales Linux
- Colas de tráfico con y sin QoS

### Ruta A: preferida (menor costo OSPF)

- Enlace R1-R2: delay 5000 us → cost OSPF: 50
- Enlace R2-R4: delay 5000 us → cost OSPF: 50
- **Costo total: 100**

### Ruta B: alternativa (mayor costo OSPF)

- Enlace R1-R3: delay 20000 us → cost OSPF: 200
- Enlace R3-R4: delay 15000 us → cost OSPF: 150
- **Costo total: 350**

**Selección de ruta con OSPF**: El router elegirá automáticamente la ruta A (costo 100) sobre la ruta B (costo 350). El `cost` se configura manualmente en cada interfaz serial usando:

```ios
interface Serial0/0/0
 ip ospf cost 50
```

Esto simula que la ruta A tiene mejor calidad (menor latencia, mayor ancho de banda).

**Generación de congestión real**: Cuando se ejecute iperf3 desde las VMs Linux con múltiples flujos, la red se congestionará de verdad, no será simulación. Sin QoS, el tráfico de gaming sufrirá degradación. Con QoS, será protegido por colas prioritarias.

## 6. Escenario de trafico a simular

En GNS3, usarás VMs Ubuntu/Debian con iperf3 para generar tráfico real. Esto es mucho más realista que Packet Tracer.

### Tipos de tráfico

**Trafico 1: gaming UDP**
- Flujo pequeño, constante y sensible a jitter
- Puerto: UDP 5001
- Simula: paquetes cortos, frecuentes y críticos (League of Legends, CS:GO, etc.)

**Trafico 2: streaming**
- Flujo sostenido, tolera más retardo que el juego
- Puerto: UDP/TCP 5002
- Simula: YouTube, Twitch, Netflix en la red

**Trafico 3: descargas pesadas**
- Tráfico TCP masivo
- Puerto: TCP 5003
- Simula: descargas de Steam, torrents, backups

### Cómo generar tráfico en GNS3

En la VM servidor (en la LAN 192.168.40.0/24):

```bash
# Terminal 1: iniciar servidor iperf3
iperf3 -s

# O en modo daemon
iperf3 -s -D
```

En las VMs clientes (en la LAN 192.168.10.0/24):

**Tráfico de gaming (UDP, sensible a latencia):**
```bash
iperf3 -u -c 192.168.40.10 -p 5001 -b 1M -l 120 -t 30
```

**Tráfico de streaming (UDP o TCP, mayor volumen):**
```bash
iperf3 -u -c 192.168.40.10 -p 5002 -b 8M -l 120 -t 60
```

**Tráfico de descarga (TCP, múltiples conexiones):**
```bash
iperf3 -c 192.168.40.10 -p 5003 -P 4 -t 60
```

### Ejecutar pruebas en paralelo

En GNS3, puedes usar múltiples terminales de Linux para ejecutar los tres tipos de tráfico simultáneamente, simulando una red real congestionada:

```bash
# En cliente1: gaming
iperf3 -u -c 192.168.40.10 -p 5001 -b 1M -l 120 -t 60 &

# En cliente2: streaming  
iperf3 -u -c 192.168.40.10 -p 5002 -b 8M -l 1400 -t 60 &

# En cliente3: descargas
iperf3 -c 192.168.40.10 -p 5003 -P 4 -t 60 &

# Esperar a que terminen
wait
```

Esto genera congestión real en los enlaces, permitiendo medir el impacto de QoS.

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

## 8. Configuracion base de routing con OSPF

Se usa OSPF (protocolo estándar abierto) con configuración manual de `cost` en interfaces para simular que la ruta A tiene mejor calidad que la ruta B.

### Conceptos clave de OSPF para este proyecto

- **Cost (métrica)**: se calcula como 100000 / bandwidth por defecto
- **Interface cost**: se puede configurar manualmente con `ip ospf cost <valor>`
- **Área OSPF**: usaremos área 0 (backbone) para simplificar
- **Passive interface**: interfaces que no forman adyacencias OSPF (hacia clientes/servidores)

### R1

```ios
hostname R1
no ip domain-lookup

interface g0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface s0/0/0
 ip address 10.0.12.1 255.255.255.252
 ip ospf cost 50
 delay 5000
 clock rate 64000
 no shutdown

interface s0/0/1
 ip address 10.0.13.1 255.255.255.252
 ip ospf cost 200
 delay 20000
 clock rate 64000
 no shutdown

router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.13.0 0.0.0.3 area 0
 passive-interface g0/0
```

### R2

```ios
hostname R2
no ip domain-lookup

interface s0/0/0
 ip address 10.0.12.2 255.255.255.252
 ip ospf cost 50
 delay 5000
 no shutdown

interface s0/0/1
 ip address 10.0.24.1 255.255.255.252
 ip ospf cost 50
 delay 5000
 clock rate 64000
 no shutdown

router ospf 1
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.24.0 0.0.0.3 area 0
```

### R3

```ios
hostname R3
no ip domain-lookup

interface s0/0/0
 ip address 10.0.13.2 255.255.255.252
 ip ospf cost 200
 delay 20000
 no shutdown

interface s0/0/1
 ip address 10.0.34.1 255.255.255.252
 ip ospf cost 150
 delay 15000
 clock rate 64000
 no shutdown

router ospf 1
 network 10.0.13.0 0.0.0.3 area 0
 network 10.0.34.0 0.0.0.3 area 0
```

### R4

```ios
hostname R4
no ip domain-lookup

interface s0/0/0
 ip address 10.0.24.2 255.255.255.252
 ip ospf cost 50
 delay 5000
 no shutdown

interface s0/0/1
 ip address 10.0.34.2 255.255.255.252
 ip ospf cost 150
 delay 15000
 no shutdown

interface g0/0
 ip address 192.168.40.1 255.255.255.0
 no shutdown

router ospf 1
 network 10.0.24.0 0.0.0.3 area 0
 network 10.0.34.0 0.0.0.3 area 0
 network 192.168.40.0 0.0.0.255 area 0
 passive-interface g0/0
```

### Verificación de selección de ruta

Desde R1, el tráfico hacia 192.168.40.0/24 debe tomar la ruta A (R1 → R2 → R4) con costo total 100, en lugar de la ruta B (R1 → R3 → R4) con costo total 350.

Verificar con:

```ios
show ip ospf neighbor
show ip route ospf
show ip ospf interface brief
```

El resultado esperado es que la tabla de routing de R1 muestre la ruta a 192.168.40.0/24 a través de R2.

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

1. Monta la topologia fisica en GNS3 (routers, switches, VMs Linux).
2. Asigna IPs a todos los equipos según el plan de direccionamiento.
3. Verifica conectividad basica con ping entre LANs.
4. Configura OSPF en todos los routers con `ip ospf cost` manual en las interfaces seriales.
5. Comprueba que la ruta A (costo 100) es preferida sobre la ruta B (costo 350) en la tabla de rutas.
6. Ajusta delay con el parámetro `delay` en interfaces (simulación visual en traceroute).
7. Lanza tráfico pesado sin QoS y toma medidas de latencia y jitter con iperf3.
8. Aplica marcado de tráfico (DSCP), LLQ, CBWFQ, shaping y policing.
9. Repite las pruebas de iperf3 con QoS activo y compara resultados.

## 14. Comandos de validacion

### Routing OSPF

```ios
show ip route ospf
show ip ospf neighbors
show ip ospf interface brief
show ip ospf interface s0/0/0
```

### Verificar cost en interfaces

```ios
show ip ospf interface s0/0/0
show running-config interface s0/0/0
```

### Interfaces y métricas

```ios
show interfaces s0/0/0
show interfaces s0/0/1
show running-config
```

### QoS

```ios
show policy-map interface s0/0/0
show policy-map interface g0/0
show access-lists
```

### Pruebas desde hosts en GNS3

```bash
# En una VM Linux (terminal)
ping -c 4 192.168.40.10
traceroute 192.168.40.10

# Ver latencia con iperf3 (modo UDP)
iperf3 -u -c 192.168.40.10 -p 5001 -b 1M -l 120 -t 10
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

Implementa el laboratorio completo en GNS3 de una sola vez:

1. **Preparación**: Descarga IOSv, crea VMs Ubuntu para clientes y servidor.
2. **Topología**: Monta la red en GNS3 según el diagrama, con 4 routers y 3 VMs Linux.
3. **Routing**: Configura OSPF con cost manual en interfaces seriales.
4. **Pruebas baseline**: Ejecuta iperf3 sin QoS y registra latencia, jitter, pérdida.
5. **QoS**: Aplica todas las políticas (marcado, LLQ, CBWFQ, shaping, policing).
6. **Validación**: Repite iperf3 con QoS y compara resultados.
7. **Documentación**: Captura outputs de `show` commands, gráficos de iperf3, análisis comparativo.

GNS3 es superior a Packet Tracer porque permite tráfico real, mediciones precisas de jitter, y validación auténtica de QoS.

## 19. Configuraciones completas listas para copiar

Este anexo junta la configuracion final por dispositivo. El orden recomendado es:

1. Configurar primero IPs e interfaces.
2. Activar OSPF con cost manual en interfaces seriales.
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
 ip ospf cost 50
 delay 5000
 clock rate 64000
 no shutdown

interface s0/0/1
 description ENLACE A R3
 ip address 10.0.13.1 255.255.255.252
 ip ospf cost 200
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

router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.13.0 0.0.0.3 area 0
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
 ip ospf cost 50
 delay 5000
 no shutdown

interface s0/0/1
 description ENLACE A R4
 ip address 10.0.24.1 255.255.255.252
 ip ospf cost 50
 delay 5000
 clock rate 64000
 no shutdown

router ospf 1
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.24.0 0.0.0.3 area 0

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
 ip ospf cost 200
 delay 20000
 no shutdown

interface s0/0/1
 description ENLACE A R4
 ip address 10.0.34.1 255.255.255.252
 ip ospf cost 150
 delay 15000
 clock rate 64000
 no shutdown

router ospf 1
 network 10.0.13.0 0.0.0.3 area 0
 network 10.0.34.0 0.0.0.3 area 0

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
 ip ospf cost 50
 delay 5000
 no shutdown

interface s0/0/1
 description ENLACE A R3
 ip address 10.0.34.2 255.255.255.252
 ip ospf cost 150
 delay 15000
 no shutdown

interface g0/0
 description LAN SERVIDOR
 ip address 192.168.40.1 255.255.255.0
 no shutdown

router ospf 1
 network 10.0.24.0 0.0.0.3 area 0
 network 10.0.34.0 0.0.0.3 area 0
 network 192.168.40.0 0.0.0.255 area 0
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

1. Desde R1 ejecuta `show ip ospf neighbors` para confirmar adyacencias.
2. Desde R1 ejecuta `show ip route ospf` y verifica que la ruta preferida sea la de R2 (costo 100).
3. Desde el PC Gamer ejecuta `ping 192.168.40.10` durante una descarga TCP pesada.
4. Levanta `iperf3` en el servidor.
5. Corre el flujo de gaming UDP y registra jitter.
6. Repite la prueba con QoS habilitado y compara.

## 21. Que deberias observar

- Sin QoS, el flujo UDP sensible tendra variacion mayor en RTT y perdida cuando el enlace este cargado.
- Con QoS, el trafico EF debe conservar prioridad sobre los flujos bulk.
- La salida de `show policy-map interface s0/0/0` debe mostrar contadores de clases y el uso de la LLQ.

## 22. Nota de laboratorio

Este proyecto debe implementarse completamente en GNS3 con IOSv. GNS3 soporta todas las políticas jerárquicas de QoS, iperf3 real, y mediciones precisas. Packet Tracer tiene limitaciones severas con OSPF jerarquico y QoS avanzado.

## 23. Guia de verificacion paso a paso

Esta seccion resume que deberias ver en cada fase y como confirmar que todo funciona.

| Paso | Que deberias ver | Como verificarlo |
| --- | --- | --- |
| 1. Topologia fisica | Los 4 routers, 2 switches, 3 clientes y 1 servidor conectados segun el diagrama | En Packet Tracer o GNS3 revisa cada enlace y confirma que no haya puertos apagados |
| 2. Direccionamiento IP | Cada interfaz del router y cada host con la IP correcta | En routers usa `show ip interface brief`; en PCs revisa configuracion IP manual |
| 3. Enlaces WAN | R1-R2 y R2-R4 con mejores metricas que R1-R3 y R3-R4 | En routers usa `show running-config interface s0/0/x` y confirma `bandwidth` y `delay` |
| 4. Routing OSPF | Debe aparecer la red remota del servidor y la ruta preferida por R2 con costo 100 | Usa `show ip route ospf` y `show ip ospf interface brief`; la ruta via R2 debe verse como la principal |
| 5. Conectividad base | El cliente debe poder llegar al servidor sin QoS | Haz `ping 192.168.40.10` y `traceroute 192.168.40.10` desde el cliente |
| 6. Trafico de carga | El servidor debe recibir sesiones iperf3 simultaneas de gaming, streaming y bulk | Levanta `iperf3 -s` en el servidor y ejecuta los tres clientes; verifica que haya trafico activo |
| 7. Sin QoS | El ping sube de forma visible cuando la descarga satura el enlace | Ejecuta ping continuo durante una descarga y compara RTT, variacion y perdida |
| 8. Marcado DSCP | El trafico de gaming debe salir marcado como EF, streaming como AF41 y bulk como CS1 | En R1 usa `show policy-map interface g0/0` y revisa que aumenten los contadores de cada clase |
| 9. LLQ y CBWFQ | El trafico de juego debe tener prioridad y el streaming una cola con ancho reservado | En R1 usa `show policy-map interface s0/0/0` y confirma que la clase `CM-GAME` tenga actividad en la cola prioritaria |
| 10. Shaping | La salida WAN debe verse controlada, sin picos violentos | Revisa `show policy-map interface s0/0/0`; el trafico debe salir dentro del limite configurado |
| 11. Policing | El trafico bulk no debe dominar el enlace | En la politica correspondiente, confirma contadores de drops o excedidos para `CM-BULK` |
| 12. Con QoS | El ping del trafico de gaming debe estabilizarse aunque sigan las descargas | Repite la prueba y compara con el caso sin QoS; el RTT deberia ser mas consistente |

### Lectura esperada de resultados

- Si `show ip route ospf` no muestra la red del servidor, el problema esta en el routing OSPF o en alguna interfaz apagada.
- Si `ping` falla, revisa primero IP, gateway y estado de interfaces antes de revisar QoS.
- Si OSPF funciona pero el trafico sigue yendo por la ruta menos preferida, revisa los valores de `ip ospf cost`.
- Si `show policy-map interface` no aumenta contadores, revisa que la ACL coincida con el puerto correcto y que la policy este aplicada en la interfaz correcta.
- Si no ves mejora en RTT, probablemente no hay congestion suficiente; aumenta el trafico bulk o reduce temporalmente el ancho de banda del enlace.

### Criterio minimo de exito

El laboratorio se considera correcto si puedes demostrar estas tres cosas:

1. Antes de QoS, el trafico de gaming empeora bajo carga.
2. Con QoS, la clase de gaming recibe prioridad y su comportamiento mejora.
3. Los comandos de verificacion muestran contadores coherentes con el trafico generado.