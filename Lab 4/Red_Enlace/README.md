# Laboratorio 4 - Telnet en Windows (actualizado)

## 1. Objetivo

Configurar y utilizar Telnet para conectarse a dispositivos de red desde Windows y documentar el procedimiento para conectar la red externa `192.168.3.0/24` con la topología del laboratorio (`192.168.78.0/24`).

## 2. Resumen de redes (corrección)

- Red externa real (VM/host): `192.168.3.0/24` (corrección de `192.168.78.3/24`).
- Red del laboratorio: `192.168.78.0/24` (se subdivide internamente según la topología del laboratorio).

La documentación del laboratorio mantiene las subredes dentro de `192.168.78.0/24` (LAN_A, LAN_B, Switch1, enlace R3-R4 y LAN_C). La red externa para integrar VMs/hosts es `192.168.3.0/24`.

## 3. Objetivo de la conexión entre redes

Conectar la red externa `192.168.3.0/24` (por ejemplo una VM en VirtualBox) con la topología del laboratorio `192.168.78.0/24` mediante el router R4 y el objeto Cloud de GNS3, de forma que:

- Las máquinas en `192.168.3.0/24` puedan alcanzar hosts en `192.168.78.0/24`.
- Los routers del laboratorio tengan rutas estáticas hacia `192.168.3.0/24` a través de R4.

## 4. Procedimiento recomendado (paso a paso)

1) Conectar R4 al Cloud en GNS3

 - En GNS3, arrastra un objeto Cloud y conéctalo a una interfaz física/virtual del host.
 - Conecta otra interfaz del Cloud a la interfaz del router R4 destinada al acceso externo (por ejemplo `Fa0/1` en R4).

2) Asignar IP en R4 para la red externa

 - En R4 configure la interfaz conectada al Cloud con una IP del rango `192.168.3.0/24`. Ejemplo (Cisco IOS):

```bash
enable
configure terminal
interface fa0/1
 ip address 192.168.3.1 255.255.255.0
 no shutdown
end
write memory
```

3) Configurar la VM/host (VirtualBox)

 - En VirtualBox, ponga el adaptador de red de la VM en **Adaptador puente** (Bridge) y seleccione la interfaz física del host que esté conectada a la misma red física.
 - Asigne a la VM una IP del rango `192.168.3.0/24`, por ejemplo `192.168.3.10/24`, y ponga como gateway `192.168.3.1` (la IP de R4 en el paso anterior).

Ejemplo (Linux VM):

```bash
ip addr add 192.168.3.10/24 dev eth0
ip route add default via 192.168.3.1
```

4) Añadir rutas estáticas en los routers del laboratorio

 - Para que R1, R2 y R3 conozcan la red `192.168.3.0/24`, añade rutas estáticas que apunten hacia R3/R4 según la topología.

Ejemplo de configuración mínima (Cisco IOS):

En R3 (nodo intermedio hacia R4):
```bash
conf t
ip route 192.168.3.0 255.255.255.0 192.168.78.194
end
wr
```

En R1 y R2 (apuntar a R3 como siguiente salto hacia R4):
```bash
conf t
ip route 192.168.3.0 255.255.255.0 192.168.78.131
end
wr
```

En R4 (si no está ya):
```bash
conf t
ip route 192.168.78.0 255.255.255.0 192.168.78.193
end
wr
```

Notas:

- Ajusta los siguientes saltos (`192.168.78.131`, `192.168.78.193`, `192.168.78.194`) si tu topología usa otras interfaces; las direcciones de ejemplo corresponden a R3 y R4 según la documentación del laboratorio.
- Alternativa: Propagar rutas con un protocolo de enrutamiento dinámico (OSPF/ RIP) si prefieres no usar rutas estáticas.

5) Verificaciones

 - Desde la VM (`192.168.3.10`) prueba:

```bash
ping 192.168.3.1      # R4
ping 192.168.78.201   # ejemplo: R4 Fa0/0 (LAN_C)
```

 - En R4: `show ip interface brief` y `show ip route`.
 - En R1/R2/R3: `show ip route` y `ping 192.168.3.10` para verificar conectividad inversa.

6) Firewall y NAT

 - Asegúrate de que firewalls en la VM/host permitan ICMP/puertos necesarios.
 - Si no puedes usar modo puente, considera usar NAT en el host o en el router R4 según necesidades (añade reglas NAT en R4 si necesitas salida a Internet desde la VM hacia la red del laboratorio y viceversa).

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
