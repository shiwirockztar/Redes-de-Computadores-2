# Parcial 2 - Laboratorio GNS3 (AS 65078)

Guia organizada para montar, configurar y verificar la topologia del Parcial 2 en GNS3.

## 1. Dos IGP en la topologia

| Requisito | Detalle |
|---|---|
| Red interna | 192.168.78.0/24 |
| Subnetting | Fijo, /29 |
| Hosts utiles por subred | 6 |
| LAN A | 172.16.0.0/24 |
| LAN B | 10.10.10.0/24 |
| AS | 65078 |
| IGP 1 | IS-IS |
| IGP 2 | OSPF area 0 |
| iBGP | Sin full mesh, con Route Reflector |
| eBGP | Con el router/Cloud del profesor |

En la topologia dibujada, el circulo de **protocolo 1** representa el dominio izquierdo de la red, donde R1 es un router de transito sin enrutamiento dinamico y R4 participa como punto de borde. El circulo de **protocolo 2** representa el dominio derecho, correspondiente a R2 y R3 con OSPF area 0.

Los routers R6 y R7 tambien forman parte de la solucion porque sostienen el acceso a LAN A. R7 conecta la LAN A y anuncia esa red al resto de la topologia, mientras que R6 sirve como transito hacia R1.

Con esa lectura, el documento cumple los numerales asi:
- Numeral 1: existen 2 IGP, IS-IS y OSPF.
- Numeral 2: R1 no ejecuta enrutamiento dinamico.
- Numeral 3: iBGP sin malla completa mediante Route Reflector.
- Numeral 4: solo se anuncia una red externa por eBGP, distinta de LAN A y LAN B.
- Numeral 5: existe la sesion eBGP con la topologia del profesor.
- Numeral 6: las actualizaciones iBGP salen por loopback0, no por interfaz fisica.

Topologia de referencia:

```text
LAN A: 172.16.0.0/24
   |
R7 - R6 - R1 - R4 - R5 - R2 - R3 - LAN B: 10.10.10.0/24
                                          |
                                      Cloud / ISP
```

Enlaces seriales:
- R7-R6
- R1-R4
- R2-R3

Enlaces Ethernet:
- R6-R1
- R4-R5
- R5-R2
- R7-LAN A
- R3-LAN B
- R3-Cloud/ISP

## 2. Subnetting y topologia en GNS3

### 2.1 Dispositivos y puertos

Se usa Cisco 3745 con NM-2FE2W y 2 WIC-2T por router.

| Router | Interfaz | IP | Rol |
|---|---|---:|---|
| R7 | Fa0/0 | 172.16.0.2/24 | LAN A |
| R7 | Se0/0 | 192.168.78.1/29 | Serial a R6 |
| R6 | Se0/0 | 192.168.78.2/29 | Serial a R7 |
| R6 | Fa0/1 | 192.168.78.9/29 | Ethernet a R1 |
| R1 | Fa0/0 | 192.168.78.10/29 | Ethernet a R6 |
| R1 | Se0/0 | 192.168.78.17/29 | Serial a R4 |
| R4 | Se0/0 | 192.168.78.18/29 | Serial a R1 |
| R4 | Fa0/1 | 192.168.78.25/29 | Ethernet a R5 |
| R5 | Fa0/0 | 192.168.78.26/29 | Ethernet a R4 |
| R5 | Fa0/1 | 192.168.78.33/29 | Ethernet a R2 |
| R2 | Fa0/0 | 192.168.78.34/29 | Ethernet a R5 |
| R2 | Se0/0 | 192.168.78.41/29 | Serial a R3 |
| R3 | Se0/0 | 192.168.78.42/29 | Serial a R2 |
| R3 | Fa0/1 | 10.10.10.1/24 | LAN B |
| R3 | Fa1/0 | DHCP | Cloud / ISP |

### 2.2 DCE en seriales

Configurar clock rate 64000 solo en el lado DCE:

| Enlace | Lado DCE |
|---|---|
| R7-R6 | R7 |
| R1-R4 | R1 |
| R2-R3 | R2 |

### 2.3 Subnetting de 192.168.78.0/24

| Enlace | Subred | IP A | IP B | Medio |
|---|---|---:|---:|---|
| R7-R6 | 192.168.78.0/29 | 192.168.78.1 | 192.168.78.2 | Serial |
| R6-R1 | 192.168.78.8/29 | 192.168.78.9 | 192.168.78.10 | Ethernet |
| R1-R4 | 192.168.78.16/29 | 192.168.78.17 | 192.168.78.18 | Serial |
| R4-R5 | 192.168.78.24/29 | 192.168.78.25 | 192.168.78.26 | Ethernet |
| R5-R2 | 192.168.78.32/29 | 192.168.78.33 | 192.168.78.34 | Ethernet |
| R2-R3 | 192.168.78.40/29 | 192.168.78.41 | 192.168.78.42 | Serial |

### 2.4 VPCS de usuario

LAN A:

```bash
ip 172.16.0.1 172.16.0.2 24
save
```

LAN B:

```bash
ip 10.10.10.10 10.10.10.1 24
save
```

## 3. Numeral 2: R1 sin protocolo de enrutamiento

R1 no ejecuta IS-IS ni OSPF. Solo pasa trafico con rutas estaticas.

### 3.1 Configuracion de R1

```ios
conf t

interface fa0/0
 ip address 192.168.78.10 255.255.255.248
 no shutdown

interface se0/0
 ip address 192.168.78.17 255.255.255.248
 clock rate 64000
 no shutdown

ip route 172.16.0.0 255.255.255.0 192.168.78.9
ip route 10.10.10.0 255.255.255.0 192.168.78.18
ip route 203.0.113.0 255.255.255.0 192.168.78.18
ip route 0.0.0.0 0.0.0.0 192.168.78.18

end
wr
```

### 3.2 Verificacion de R1

> Si el ping a `10.10.10.10` falla, revisa primero que R6 anuncie también la red `192.168.78.8/29` en IS-IS. Sin esa red no existe retorno completo hacia R1.

```ios
show ip protocols
show ip route
ping 172.16.0.1
ping 10.10.10.10
```

### 3.3 Configuracion de R6

R6 conecta R7 con R1 y participa en IS-IS hacia el lado izquierdo de la topologia.

```ios
conf t

interface se0/0
 ip address 192.168.78.2 255.255.255.248
 ip router isis CORE
 no shutdown

interface fa0/1
 ip address 192.168.78.9 255.255.255.248
 ip router isis CORE
 no shutdown

ip route 0.0.0.0 0.0.0.0 192.168.78.10

router isis CORE
 net 49.0001.0000.0000.0006.00
 is-type level-2-only
 redistribute connected

end
wr
```

### 3.4 Configuracion de R7

R7 es el borde de LAN A y por eso anuncia la red de usuarios al dominio interno.

```ios
conf t

interface fa0/0
 ip address 172.16.0.2 255.255.255.0
 ip router isis CORE
 no shutdown

interface se0/0
 ip address 192.168.78.1 255.255.255.248
 ip router isis CORE
 clock rate 64000
 no shutdown

router isis CORE
 net 49.0001.0000.0000.0007.00
 is-type level-2-only
 redistribute connected

end
wr
```

### 3.5 Verificacion de R6 y R7

```ios
show ip interface brief
show isis neighbors
show ip route
```

En R7 tambien debe responder la LAN A:

```bash
ping 172.16.0.1
```

## 4. Numeral 3: iBGP en el interior

Se usa R5 como Route Reflector para evitar full mesh. Las sesiones iBGP se levantan por loopback0.

### 4.1 Loopbacks BGP

| Router | Loopback0 |
|---|---:|
| R4 | 4.4.4.4/32 |
| R5 | 5.5.5.5/32 |
| R2 | 2.2.2.2/32 |
| R3 | 3.3.3.3/32 |

### 4.2 OSPF area 0

#### R4

```ios
conf t

interface se0/0
 ip address 192.168.78.18 255.255.255.248
 no shutdown

interface fa0/1
 ip address 192.168.78.25 255.255.255.248
 no shutdown

interface loopback0
 ip address 4.4.4.4 255.255.255.255

ip route 172.16.0.0 255.255.255.0 192.168.78.17

router ospf 1
 router-id 4.4.4.4
 passive-interface loopback0
 network 192.168.78.24 0.0.0.7 area 0
 network 4.4.4.4 0.0.0.0 area 0
 redistribute static subnets metric 20 metric-type 2

end
wr
```

#### R5

```ios
conf t

interface fa0/0
 ip address 192.168.78.26 255.255.255.248
 no shutdown

interface fa0/1
 ip address 192.168.78.33 255.255.255.248
 no shutdown

interface loopback0
 ip address 5.5.5.5 255.255.255.255

router ospf 1
 router-id 5.5.5.5
 passive-interface loopback0
 network 192.168.78.24 0.0.0.7 area 0
 network 192.168.78.32 0.0.0.7 area 0
 network 5.5.5.5 0.0.0.0 area 0

end
wr
```

#### R2

```ios
conf t

interface fa0/0
 ip address 192.168.78.34 255.255.255.248
 no shutdown

interface se0/0
 ip address 192.168.78.41 255.255.255.248
 no shutdown

interface loopback0
 ip address 2.2.2.2 255.255.255.255

router ospf 1
 router-id 2.2.2.2
 passive-interface loopback0
 network 192.168.78.32 0.0.0.7 area 0
 network 192.168.78.40 0.0.0.7 area 0
 network 2.2.2.2 0.0.0.0 area 0

end
wr
```

#### R3

```ios
conf t

interface se0/0
 ip address 192.168.78.42 255.255.255.248
 no shutdown

interface fa0/1
 ip address 10.10.10.1 255.255.255.0
 no shutdown

interface fa1/0
 ip address dhcp
 no shutdown

interface loopback0
 ip address 3.3.3.3 255.255.255.255

router ospf 1
 router-id 3.3.3.3
 passive-interface loopback0
 network 192.168.78.40 0.0.0.7 area 0
 network 10.10.10.0 0.0.0.255 area 0
 network 3.3.3.3 0.0.0.0 area 0

end
wr
```

### 4.3 iBGP sin malla completa

#### R5 como Route Reflector

```ios
conf t

router bgp 65078
 bgp log-neighbor-changes
 neighbor 4.4.4.4 remote-as 65078
 neighbor 4.4.4.4 update-source loopback0
 neighbor 4.4.4.4 route-reflector-client
 neighbor 2.2.2.2 remote-as 65078
 neighbor 2.2.2.2 update-source loopback0
 neighbor 2.2.2.2 route-reflector-client
 neighbor 3.3.3.3 remote-as 65078
 neighbor 3.3.3.3 update-source loopback0
 neighbor 3.3.3.3 route-reflector-client

end
wr
```

#### R4

```ios
conf t

router bgp 65078
 bgp log-neighbor-changes
 neighbor 5.5.5.5 remote-as 65078
 neighbor 5.5.5.5 update-source loopback0
 network 172.16.0.0 mask 255.255.255.0

end
wr
```

#### R2

```ios
conf t

router bgp 65078
 bgp log-neighbor-changes
 neighbor 5.5.5.5 remote-as 65078
 neighbor 5.5.5.5 update-source loopback0

end
wr
```

#### R3

```ios
conf t

router bgp 65078
 bgp log-neighbor-changes
 neighbor 5.5.5.5 remote-as 65078
 neighbor 5.5.5.5 update-source loopback0
 neighbor 200.78.0.2 remote-as 65079
 neighbor 200.78.0.2 description ISP_del_profesor
 neighbor 5.5.5.5 next-hop-self
 network 10.10.10.0 mask 255.255.255.0
 network 203.0.113.0 mask 255.255.255.0

end
wr
```

## 5. Numeral 4: eBGP con el profesor

La sesion eBGP se levanta en R3 hacia la direccion del profesor. En este README se deja como ejemplo `200.78.0.2` con AS `65079`.

La unica red anunciada por eBGP en este ejemplo es `203.0.113.0/24`, que es distinta de LAN A y LAN B.

## 6. Verificacion por numeral

### Numeral 1: dos IGP en la topologia

Que se hizo: se configuro IS-IS en la parte de LAN A y OSPF en la parte de LAN B.

Como comprobarlo: verifica vecinos y tablas de enrutamiento en ambos dominios.

```ios
show isis neighbors
show ip ospf neighbor
show ip route
```

### Numeral 2: R1 sin protocolo de enrutamiento

Que se hizo: R1 solo usa rutas estaticas y no ejecuta ningun protocolo dinamico.

Como comprobarlo: en R1 debe aparecer que no hay protocolo de enrutamiento configurado y aun asi debe responder hacia LAN A.

```ios
show ip protocols
ping 172.16.0.1
```

### Numeral 3: iBGP sin malla completa

Que se hizo: se uso R5 como Route Reflector para evitar una malla completa iBGP.

Como comprobarlo: revisa el resumen BGP y confirma que los vecinos iBGP usan loopbacks.

```ios
show ip bgp summary
show ip bgp neighbors
```

### Numeral 4: una sola red anunciada por eBGP

Que se hizo: solo se anuncio `203.0.113.0/24` por eBGP.

Como comprobarlo: en la tabla BGP debe verse ese prefijo y no deben anunciarse LAN A ni LAN B por eBGP.

```ios
show ip bgp
```

### Numeral 5: eBGP con la topologia del profesor

Que se hizo: R3 quedo listo para levantar la sesion eBGP con el vecino que defina el profesor.

Como comprobarlo: cuando se cargue la IP correcta, `show ip bgp summary` debe mostrar la sesion en estado estable.

```ios
show ip bgp summary
```

### Numeral 6: iBGP sin usar IP fisica como origen

Que se hizo: las sesiones iBGP usan `update-source loopback0`.

Como comprobarlo: revisa la configuracion BGP y confirma que el origen sea la loopback, no una interfaz fisica.

```ios
show ip bgp neighbors
```

### Pruebas finales recomendadas

Desde LAN A:

```bash
ping 10.10.10.10
```

Desde LAN B:

```bash
ping 172.16.0.1
```

Desde R7:

```ios
ping 10.10.10.10
traceroute 10.10.10.10
```

Desde R3:

```ios
ping 172.16.0.1
traceroute 172.16.0.1
```

## 7. Orden recomendado de implementacion

1. Crear el proyecto en GNS3.
2. Agregar 7 routers 3745, 2 VPCS y 1 Cloud o router ISP.
3. Instalar NM-2FE2W y 2 WIC-2T en cada router.
4. Cablear segun la tabla de la seccion 2.
5. Configurar interfaces e IPs.
6. Levantar IS-IS en R6 y R7.
7. Levantar OSPF en R4, R5, R2 y R3.
8. Crear loopbacks y sesiones iBGP.
9. Configurar R1 con rutas estaticas.
10. Configurar LAN A y LAN B.
11. Validar vecinos, rutas y pings.

## 8. Resumen rapido por router

- R7: LAN A + IS-IS.
- R6: IS-IS + ruta estatica hacia R1.
- R1: solo rutas estaticas.
- R4: OSPF + BGP interior.
- R5: OSPF + Route Reflector iBGP.
- R2: OSPF + cliente iBGP.
- R3: OSPF + cliente iBGP + eBGP con ISP.

## 9. Nota importante

La direccion del vecino eBGP y el ASN remoto del profesor pueden cambiar en el examen. La estructura interna del README queda lista para GNS3 y solo habria que ajustar ese dato puntual si el docente lo define de otra forma.

faltantes
Te preparo una versión más corta del README para exponerla oralmente.
Te saco un resumen por router, listo para memorizar en la sustentación.
Te reviso el archivo README backup.md para dejarlo alineado con la versión final