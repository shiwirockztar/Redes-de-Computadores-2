# Laboratorio 3 - Punto 1

## 1. Contexto

- PC1 en VLAN 10.
- PC2 en VLAN 20.
- VLAN 99 para administracion de switches (SVI).
- Los switches son de capa 2.

Sin un equipo de capa 3, no hay comunicacion entre VLANs.

---

## 2. Apartado (a): Router usando interfaces access (sin trunk al router)

### 2.1 Diseno

En este escenario se usa una interfaz fisica del router por cada VLAN.

| VLAN | Red | Gateway (Router) |
| --- | --- | --- |
| 10 | 192.168.10.0/24 | 192.168.10.1 |
| 20 | 192.168.20.0/24 | 192.168.20.1 |
| 99 | 192.168.99.0/24 | 192.168.99.1 |

### 2.2 Interfaces del router Cisco 2911

Normalmente:

- GigabitEthernet0/0
- GigabitEthernet0/1
- GigabitEthernet0/2

Se puede conectar una por VLAN (porque no se permite trunk).

### 2.3 Conexion sugerida (router conectado a S3)

| Router 2911 | Switch S3 | VLAN |
| --- | --- | --- |
| G0/0 | Fa0/10 | VLAN 10 |
| G0/1 | Fa0/11 | VLAN 20 |
| G0/2 | Fa0/12 | VLAN 99 |

### 2.4 Configuracion base en switches

```bash
enable
configure terminal
vlan 10
 name VLAN10
vlan 20
 name VLAN20
vlan 99
 name ADMIN
end
copy running-config startup-config
```

### 2.5 Puertos de acceso para hosts

S1 (PC1 en VLAN 10):

```bash
enable
configure terminal
interface fa0/1
 switchport mode access
 switchport access vlan 10
end
copy running-config startup-config
```

S2 (PC2 en VLAN 20):

```bash
enable
configure terminal
interface fa0/1
 switchport mode access
 switchport access vlan 20
end
copy running-config startup-config
```

### 2.6 Enlaces entre switches (trunk)

```bash
enable
configure terminal
interface fa0/2
 switchport mode trunk
interface fa0/3
 switchport mode trunk
end
copy running-config startup-config
```

### 2.7 SVI de administracion (VLAN 99)

S1:

```bash
enable
configure terminal
interface vlan 99
 ip address 192.168.99.11 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.99.1
end
copy running-config startup-config
```

S2:

```bash
enable
configure terminal
interface vlan 99
 ip address 192.168.99.12 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.99.1
end
copy running-config startup-config
```

S3:

```bash
enable
configure terminal
interface vlan 99
 ip address 192.168.99.13 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.99.1
end
copy running-config startup-config
```

### 2.8 Puertos de S3 hacia el router (access)

```bash
enable
configure terminal
interface fa0/10
 switchport mode access
 switchport access vlan 10

interface fa0/11
 switchport mode access
 switchport access vlan 20

interface fa0/12
 switchport mode access
 switchport access vlan 99
end
copy running-config startup-config
```

### 2.9 Router (una interfaz por VLAN)

```bash
enable
configure terminal
interface g0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface g0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown

interface g0/2
 ip address 192.168.99.1 255.255.255.0
 no shutdown
end
copy running-config startup-config
```

### 2.10 Hosts

- PC1 (VLAN 10): IP 192.168.10.10 /24, GW 192.168.10.1
- PC2 (VLAN 20): IP 192.168.20.10 /24, GW 192.168.20.1

### 2.11 Telnet en switches

```bash
enable
configure terminal
line vty 0 4
 password cisco
 login
end
copy running-config startup-config
```

### 2.12 Pruebas sugeridas

```bash
ping 192.168.20.10
ping 192.168.99.11
telnet 192.168.99.11
telnet 192.168.99.12
telnet 192.168.99.13
```

### 2.13 Verificacion final

En el router:

```bash
show ip interface brief
```

Esperado:

```text
G0/0  192.168.10.1  up up
G0/1  192.168.20.1  up up
G0/2  192.168.99.1  up up
```

---

## 3. Apartado (b): Gateway por defecto y nodo gateway

### 3.1 Escenario sin router

- No existe dispositivo de capa 3.
- Ningun nodo puede actuar como gateway.
- No hay enrutamiento entre VLANs.

### 3.2 Escenario con router (apartado a)

- PCs: si requieren gateway por defecto.
- Switches (gestion): si requieren `ip default-gateway` para administracion remota.
- Router: no requiere gateway por defecto para redes directamente conectadas.

Gateways por VLAN:

- VLAN 10 -> `192.168.10.1`
- VLAN 20 -> `192.168.20.1`
- VLAN 99 -> `192.168.99.1`

Texto breve para informe:

> Si es necesario configurar gateway por defecto en computadores y switches de gestion, pero no en el router. El router actua como gateway para todas las VLANs: 10, 20 y 99.

---

## 4. Apartado (c): Una unica interfaz entre switch y router

Se usa router-on-a-stick:

- Un unico enlace fisico switch-router.
- Puerto del switch en trunk 802.1Q.
- Subinterfaces en el router (una por VLAN).

Switch (puerto hacia router):

```bash
interface fa0/24
 switchport mode trunk
```

Router:

```bash
interface fa0/0
 no shutdown

interface fa0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface fa0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface fa0/0.99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0
```

---

## 5. Apartado (d): Camino del paquete (PC -> SVI) con router-on-a-stick

Supuestos:

- PC en VLAN 10.
- SVI del switch en VLAN 99.
- Router-on-a-stick configurado.

Proceso:

1. El PC genera un ICMP Echo Request hacia la IP de la SVI.
2. El PC detecta que el destino esta en otra red y envia al gateway.
3. Si no conoce la MAC del gateway, hace ARP en VLAN 10.
4. El router responde ARP y el PC envia la trama al switch por puerto access.
5. El switch etiqueta la trama con 802.1Q VLAN 10 al salir por trunk.
6. El router recibe en la subinterfaz `fa0/0.10` y enruta hacia VLAN 99.
7. Si es necesario, el router resuelve ARP en VLAN 99.
8. El router reenvia la trama etiquetada como VLAN 99.
9. El switch retira la etiqueta y entrega el paquete a su interfaz VLAN.

---

## 6. Errores comunes

- No configurar `ip default-gateway` en switches.
- Dejar interfaces del router en `shutdown`.
- Asignar puertos a VLAN incorrecta.
- Omitir `no shutdown` en la SVI.
- Configurar mal el tipo de enlace (`access` vs `trunk`).

## 7. Resultado esperado

- PC1 y PC2 tienen conectividad entre VLANs (cuando hay router).
- Hay acceso de administracion (por ejemplo Telnet) a switches en VLAN 99.
- Existe comunicacion estable entre hosts, switches y router.
