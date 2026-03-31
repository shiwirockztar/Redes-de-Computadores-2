# Laboratorio 3 - Punto 1

## Contexto del ejercicio

- PC1 en VLAN 10.
- PC2 en VLAN 20.
- VLAN 99 para administracion de switches (SVI).
- Los switches son de capa 2.

Sin un equipo de capa 3, no hay comunicacion entre VLANs.

## a) Inclusion del router usando interfaces access

### Diseno

Como en este escenario no se usa trunk hacia el router, se conecta una interfaz fisica por VLAN.

| VLAN | Red | Interfaz del router |
| --- | --- | --- |
| 10 | 192.168.10.0/24 | Fa0/0 |
| 20 | 192.168.20.0/24 | Fa0/1 |
| 99 | 192.168.99.0/24 | Fa1/0 |

### Configuracion base en switches

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

### Puertos de acceso para hosts (ejemplo)

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

### Enlaces entre switches (trunk)

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

### SVI de administracion (una por switch)

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

### Puertos switch-router en modo access

Asignar en cada switch el puerto que conecta al router en la VLAN correspondiente (10, 20 o 99):

```bash
interface fa0/x
 switchport mode access
 switchport access vlan <10|20|99>
```

### Router (una interfaz por VLAN)

```bash
enable
configure terminal
interface fa0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface fa0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown
interface fa1/0
 ip address 192.168.99.1 255.255.255.0
 no shutdown
end
copy running-config startup-config
```

### Telnet en switches

```bash
enable
configure terminal
line vty 0 4
 password cisco
 login
end
copy running-config startup-config
```

### Pruebas sugeridas

```bash
ping 192.168.20.10
ping 192.168.99.11
telnet 192.168.99.11
telnet 192.168.99.12
telnet 192.168.99.13
```
🧠 ✅ 1. INTERFACES DEL ROUTER 2911

El Cisco 2911 normalmente trae:

GigabitEthernet0/0
GigabitEthernet0/1
GigabitEthernet0/2

👉 Usaremos una por cada VLAN (porque NO se permite trunk)

🔌 ✅ 2. CONEXIONES FINALES

Conecta el router al Switch 3 (S3) así:

Router 2911	Switch S3	VLAN
G0/0	Fa0/10	VLAN 10
G0/1	Fa0/11	VLAN 20
G0/2	Fa0/12	VLAN 99
⚙️ ✅ 3. CONFIGURACIÓN EN S3 (IMPORTANTE)
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
🌐 ✅ 4. CONFIGURACIÓN DEL ROUTER (2911)
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
💻 ✅ 5. CONFIGURACIÓN DE HOSTS
PC1 (VLAN 10)
IP: 192.168.10.10
Mask: 255.255.255.0
Gateway: 192.168.10.1
PC2 (VLAN 20)
IP: 192.168.20.10
Mask: 255.255.255.0
Gateway: 192.168.20.1
🧩 ✅ 6. SWITCHES (GATEWAY DE ADMINISTRACIÓN)

En S1, S2 y S3:

ip default-gateway 192.168.99.1
🧠 ✅ 7. RESPUESTA EXACTA PARA EL INFORME

Puedes copiar algo así:

✔ ¿Es necesario configurar gateway por defecto?

Sí, es necesario configurar gateway por defecto en los computadores y en los switches, pero no en el router.

Computadores: requieren gateway para comunicarse con otras VLANs.
Switches: requieren gateway para permitir la administración remota (Telnet) desde otras redes.
Router: no requiere gateway porque todas las redes están directamente conectadas a sus interfaces.
✔ Nodo que cumple la función de gateway

El router (Cisco 2911) actúa como gateway para todas las VLANs:

VLAN 10 → 192.168.10.1
VLAN 20 → 192.168.20.1
VLAN 99 → 192.168.99.1
🧪 ✅ 8. VERIFICACIÓN FINAL

En el router:

show ip interface brief

Debe quedar:

G0/0  192.168.10.1  up up
G0/1  192.168.20.1  up up
G0/2  192.168.99.1  up up
🚀 🎯 CON ESTO YA CUMPLES EL NUMERAL (a)

Si algo no te hace ping:

👉 casi siempre es:

VLAN mal asignada
Gateway mal puesto
Puerto apagado (shutdown)


## b) Gateway por defecto y nodo que cumple esa funcion

### Escenario sin router

- No existe dispositivo de capa 3.
- Por lo tanto, ningun nodo puede actuar como gateway.
- Resultado: no hay enrutamiento entre VLANs.

### Escenario con router (apartado a)

- PCs: si requieren gateway por defecto.
- Switches (gestion): si requieren `ip default-gateway` para administracion remota.
- El gateway es el router:
  - VLAN 10 -> `192.168.10.1`
  - VLAN 20 -> `192.168.20.1`
  - VLAN 99 -> `192.168.99.1`

## c) Uso de una unica interfaz entre conmutador y enrutador

Para este caso se usa router-on-a-stick:

- Un unico enlace fisico switch-router.
- El puerto del switch va en trunk 802.1Q.
- El router usa subinterfaces (una por VLAN).

Switch (puerto al router):

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

## d) Camino del paquete (PC -> SVI del switch) con router-on-a-stick

Supuestos:

- PC en VLAN 10.
- SVI del switch en VLAN 99.
- Router-on-a-stick configurado.

Proceso:

1. El PC genera un ICMP Echo Request con destino a la IP de la SVI.
2. El PC detecta que el destino esta en otra red y envia al gateway.
3. Si no conoce la MAC del gateway, hace ARP en VLAN 10.
4. El router responde y el PC envia la trama al switch (puerto access).
5. El switch etiqueta con 802.1Q VLAN 10 al salir por trunk.
6. El router recibe, procesa en `fa0/0.10` y enruta hacia VLAN 99.
7. Si hace falta, el router resuelve por ARP la MAC de la SVI en VLAN 99.
8. El router reenvia la trama etiquetada como VLAN 99.
9. El switch retira etiqueta y entrega el paquete a su interfaz VLAN.

## Errores comunes

- No configurar `ip default-gateway` en switches.
- Dejar interfaces del router en shutdown.
- Asignar puertos a VLAN incorrecta.
- Omitir `no shutdown` en la SVI.
- Configurar mal el tipo de enlace (access vs trunk) segun el escenario.

## Resultado esperado

- PC1 y PC2 tienen conectividad entre VLANs (cuando hay router).
- Acceso de administracion (por ejemplo Telnet) a switches en VLAN 99.
- Comunicacion estable entre hosts, switches y router.
