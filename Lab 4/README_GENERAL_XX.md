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
