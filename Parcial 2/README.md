# Parcial 2 - Laboratorio GNS3 (AS 65078)

Guia organizada en 6 numerales para montar, configurar y verificar la topologia del Parcial 2 en GNS3.

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

```ios
show ip protocols
show ip route
ping 172.16.0.1
ping 10.10.10.10
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

## 6. Numeral 5 y 6: verificacion final

### 6.1 IGP y BGP

En routers IS-IS:

```ios
show isis neighbors
show isis database
```

En routers OSPF:

```ios
show ip ospf neighbor
show ip ospf database
```

En todos los routers:

```ios
show ip route
show ip interface brief
```

En BGP:

```ios
show ip bgp summary
show ip bgp
```

### 6.2 Pruebas end-to-end

Desde LAN A:

```bash
ping 172.16.0.2
ping 10.10.10.10
```

Desde LAN B:

```bash
ping 10.10.10.1
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

### 6.3 Comportamiento esperado

- R1 debe mostrar `% No routing protocol is configured.`
- R7 debe alcanzar `10.10.10.10`.
- LAN A debe llegar a LAN B y viceversa.
- R5 debe actuar como Route Reflector iBGP.
- Las sesiones iBGP deben usar `update-source loopback0`.

## 7. Orden recomendado de implementacion

1. Crear el proyecto en GNS3.
2. Agregar 7 routers 3745, 2 VPCS y 1 Cloud o router ISP.
3. Instalar NM-2FE2W y 2 WIC-2T en cada router.
4. Cablear segun la tabla de la seccion 2.
5. Configurar interfaces e IPs.
6. Levantar IS-IS en R7 y R6.
7. Levantar OSPF en R4, R5, R2 y R3.
8. Crear loopbacks y sesiones iBGP.
9. Configurar R1 con rutas estaticas.
10. Configurar LAN A y LAN B.
11. Validar vecinos, rutas y pings.

## 8. Resumen rapido por router

- R7: LAN A + IS-IS.
- R6: IS-IS + redistribucion de ruta estatica hacia LAN B.
- R1: solo rutas estaticas.
- R4: OSPF + redistribucion de ruta estatica hacia LAN A + BGP.
- R5: OSPF + Route Reflector iBGP.
- R2: OSPF + cliente iBGP.
- R3: OSPF + cliente iBGP + eBGP con ISP.

## 9. Nota importante

La direccion del vecino eBGP y el ASN remoto del profesor pueden cambiar en el examen. La estructura interna del README queda lista para GNS3 y solo habria que ajustar ese dato puntual si el docente lo define de otra forma.
