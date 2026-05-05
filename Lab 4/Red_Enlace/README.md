# Laboratorio 4 - Telnet en Windows (actualizado)

## 1. Objetivo

Configurar y utilizar Telnet para conectarse a dispositivos de red desde Windows y documentar el procedimiento para conectar la red externa `192.168.3.0/24` con la topología del laboratorio (`192.168.78.0/24`).

## 2. Resumen de redes (corrección)

- Red externa real (VM/host): `192.168.3.0/24` .
- Red del laboratorio: `192.168.78.0/24` (se subdivide internamente según la topología del laboratorio).

La documentación del laboratorio mantiene las subredes dentro de `192.168.78.0/24` (LAN_A, LAN_B, Switch1, enlace R3-R4 y LAN_C). La red externa para integrar VMs/hosts es `192.168.3.0/24`.

## 3. Objetivo de la conexión entre redes

Conectar la red externa `192.168.3.0/24` (por ejemplo una VM en VirtualBox) con la topología del laboratorio `192.168.78.0/24` mediante el router R4 y el objeto Cloud de GNS3, de forma que:

- Las máquinas en `192.168.3.0/24` puedan alcanzar hosts en `192.168.78.0/24`.
- Los routers del laboratorio tengan rutas estáticas hacia `192.168.3.0/24` a través de R4.

## 4. Procedimiento recomendado (paso a paso)

### 4.1 Lado externo (VirtualBox + Cloud + interfaz externa de R4)

1) Conectar el Cloud en GNS3

 - En GNS3, arrastra un objeto Cloud y asígnalo a la interfaz del host/VM que está en la red `192.168.3.0/24`.
 - Conecta el Cloud a la interfaz externa de R4 (por ejemplo `Fa0/1`).

2) Configurar la interfaz externa de R4

```bash
enable
configure terminal
interface fa0/1
 ip address 192.168.3.1 255.255.255.0
 no shutdown
end
write memory
```

3) Configurar la VM en VirtualBox

 - En VirtualBox, usa **Adaptador puente** sobre la interfaz correcta del host.
 - Configura la VM en `192.168.3.0/24`, por ejemplo `192.168.3.10/24`, con gateway `192.168.3.1`.

Ejemplo (Linux VM):

```bash
ip addr add 192.168.3.10/24 dev eth0
ip route add default via 192.168.3.1
```

### 4.2 Lado laboratorio (red 192.168.78.0/24)

4) Verificar y levantar interfaces internas del laboratorio

 - Asegura que en R1, R2, R3 y R4 las interfaces hacia la topología `192.168.78.0/24` estén con IP correcta y en estado `up/up`.
 - En particular, el enlace R3-R4 debe estar operativo (por ejemplo `192.168.78.193/29` y `192.168.78.194/29`).

Comando de revisión en cada router:

```bash
show ip interface brief
```

5) Añadir rutas para alcanzar la red externa `192.168.3.0/24`

En R3 (hacia R4):
```bash
conf t
ip route 192.168.3.0 255.255.255.0 192.168.78.194
end
wr
```

En R1 y R2 (hacia R3):
```bash
conf t
ip route 192.168.3.0 255.255.255.0 192.168.78.131
end
wr
```

6) Añadir ruta de retorno en R4 hacia la red del laboratorio

```bash
conf t
ip route 192.168.78.0 255.255.255.0 192.168.78.193
end
wr
```

Notas:

- Ajusta los siguientes saltos (`192.168.78.131`, `192.168.78.193`, `192.168.78.194`) si en tu topología cambian los IP de interfaz.
- Alternativa: usar OSPF o RIP en vez de rutas estáticas.

### 4.3 Como se conectan ambas redes (flujo)

El camino lógico queda asi:

`VM 192.168.3.10 -> gateway 192.168.3.1 (R4) -> enlace R4-R3 (192.168.78.194/193) -> red interna 192.168.78.0/24`

Y el retorno:

`host del lab 192.168.78.x -> R3 -> R4 -> 192.168.3.1 -> VM 192.168.3.10`

Si falta alguna ruta de ida o de vuelta, el ping fallara en un solo sentido o en ambos.

### 4.4 Pruebas para validar que si hubo conexion

1) Prueba local de la red externa (VM a R4 externo)

```bash
ping 192.168.3.1
```

2) Prueba de cruce hacia el laboratorio (VM a red 192.168.78.0/24)

```bash
ping 192.168.78.201
traceroute 192.168.78.201
```

3) Prueba inversa (laboratorio hacia VM)

Desde R3 o R1:

```bash
ping 192.168.3.10
```

4) Validar tablas de enrutamiento

En R1/R2/R3/R4:

```bash
show ip route
```

Debes ver:

- En R1/R2/R3: ruta a `192.168.3.0/24`.
- En R4: ruta a `192.168.78.0/24`.

5) Validacion final (criterio de exito)

- Hay ping de VM -> red del laboratorio.
- Hay ping de laboratorio -> VM.
- El `traceroute` desde la VM muestra salto por R4 y luego por el lado interno (R3).

### 4.5 Firewall y NAT

 - Asegúrate de que firewalls en VM/host permitan ICMP y Telnet si lo vas a usar.
 - Si no puedes usar puente, puedes usar NAT, pero debes agregar reglas de traducción/rutas adicionales en R4.

## 5. Puertos Telnet y comandos de acceso

Los puertos de Telnet en GNS3 pueden variar: si mantienes los puertos documentados en la topología, usa `telnet localhost 500x` según el dispositivo.

## 6. Verificación rápida y comandos útiles

En cada router:
```bash
show ip interface brief
show ip route
```

Comprobaciones desde VM y routers:
```bash
ping 192.168.3.1       # R4 externo
ping 192.168.78.1      # gateway LAN_A
traceroute 192.168.78.1
```

## 7. Notas importantes

- Se corrigió la referencia errónea `192.168.78.3/24` a la red externa real `192.168.3.0/24`.
- Mantén consistencia en las máscaras y gateways al asignar direcciones.
- Telnet es no encriptado: usar sólo en entornos de laboratorio; para producción usar SSH.

Si quieres, puedo:

- ajustar automáticamente las rutas estáticas en los archivos de configuración de los routers del proyecto (si me indicas la ubicación),
- o generar un script con los comandos para copiar/pegar en cada router.

## 8. Scripts: Topología alternativa usando 192.168.3.0/24

La topología es la misma que la documentada para `192.168.78.0/24` pero con el prefijo `192.168.3.0/24`. A continuación tienes los comandos listos para copiar/pegar en cada dispositivo (Cisco IOS para routers/switch, comandos `ip` para PCs/VPCS). Sigue el orden: configurar routers, switch, PCs y por último verificar.

Nota: las subredes mantienen los mismos offsets (/26, /29) pero usando el prefijo `192.168.3.x`.

- LAN_A: 192.168.3.0/26 (hosts .1 - .62)
- LAN_B: 192.168.3.64/26
- Switch1: 192.168.3.128/26 (mgmt .132)
- Enlace R3-R4: 192.168.3.192/29 (hosts .193, .194)
- LAN_C: 192.168.3.200/29 (hosts .201, .202)

---

R1 (Cisco IOS)
```bash
enable
conf t
int fa0/0
 ip address 192.168.3.1 255.255.255.192
 no shutdown
int fa0/1
 ip address 192.168.3.129 255.255.255.192
 no shutdown
end
wr
```

R2 (Cisco IOS)
```bash
enable
conf t
int fa0/0
 ip address 192.168.3.65 255.255.255.192
 no shutdown
int fa0/1
 ip address 192.168.3.130 255.255.255.192
 no shutdown
end
wr
```

R3 (Cisco IOS)
```bash
enable
conf t
int fa0/0
 ip address 192.168.3.131 255.255.255.192
 no shutdown
int s0/0
 ip address 192.168.3.193 255.255.255.248
 no shutdown
end
wr
```

R4 (Cisco IOS)
```bash
enable
conf t
int fa0/0
 ip address 192.168.3.201 255.255.255.248
 no shutdown
int s0/0
 ip address 192.168.3.194 255.255.255.248
 no shutdown
end
wr
```

Switch1 (gestión)
```bash
enable
conf t
int vlan 1
 ip address 192.168.3.132 255.255.255.192
 no shutdown
ip default-gateway 192.168.3.129
end
wr
```

PCs / VPCS (ejemplos)

LAN_A (PC)
```bash
# en VPCS: 
ip 192.168.3.10/26 192.168.3.1
# o en Linux:
ip addr add 192.168.3.10/26 dev eth0
ip route add default via 192.168.3.1
```

LAN_B (PC)
```bash
ip 192.168.3.74/26 192.168.3.65
```

LAN_C (PC)
```bash
ip 192.168.3.202/29 192.168.3.201
```

---

Rutas estáticas mínimas (ejemplo)

R1:
```bash
conf t
ip route 192.168.3.64 255.255.255.192 192.168.3.130
ip route 192.168.3.192 255.255.255.248 192.168.3.131
ip route 192.168.3.200 255.255.255.248 192.168.3.131
end
wr
```

R2:
```bash
conf t
ip route 192.168.3.0 255.255.255.192 192.168.3.129
ip route 192.168.3.192 255.255.255.248 192.168.3.131
ip route 192.168.3.200 255.255.255.248 192.168.3.131
end
wr
```

R3:
```bash
conf t
ip route 192.168.3.0 255.255.255.192 192.168.3.129
ip route 192.168.3.64 255.255.255.192 192.168.3.130
ip route 192.168.3.200 255.255.255.248 192.168.3.194
end
wr
```

R4 (añadir ruta hacia la red de laboratorio alternativa si hace falta):
```bash
conf t
ip route 192.168.3.0 255.255.255.192 192.168.3.193
end
wr
```

Verificaciones

 - En cada router: `show ip interface brief` y `show ip route`.
 - Desde PCs: `ping` al gateway y a otros hosts (por ejemplo `ping 192.168.3.1`, `ping 192.168.3.65`, `ping 192.168.3.202`).

Consejos rápidos

 - Copia y pega cada bloque en la consola del dispositivo correspondiente.
 - Ajusta nombres de interfaces según tu imagen en GNS3 (`f0/0` vs `fa0/0`, `s0/0` vs `s0/1`).
 - Si prefieres, genero archivos `.txt` separados por dispositivo para importarlos.
