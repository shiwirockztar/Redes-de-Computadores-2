# comando utiles en packettracer para manejar mac
//en switchs
enable 
show mac-address-table 
//en pc
arp -a 
arp -d 
arp -d *

switchport mode access

reload 
show runnig-confi

//configuracion de inicio
configure terminal

```bash
Switch>enable
Switch#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#interface range fastEthernet 0/1-2
Switch(config-if-range)#switchport mode access
Switch(config-if-range)#switchport access vlan 10
Switch(config-if-range)#interface range fastEthernet 0/3-4
Switch(config-if-range)#switchport mode access
Switch(config-if-range)#switchport access vlan 20
Switch#
%SYS-5-CONFIG_I: Configured from console by console
copy running-config startup-config 
```

Switch>enable
Switch#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#interface range fastEthernet 0/24
Switch(config-if-range)#switchport mode trunk
Switch#
%SYS-5-CONFIG_I: Configured from console by console
copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]

Vamos a continuar con los numerales (d) y (e) considerando que estás trabajando con un switch capa 2 con VLANs configuradas.

(d) Tabla MAC vs Puerto

La tabla MAC (CAM table) del switch guarda la correspondencia:

MAC address  →  Puerto

Cuando no había VLANs, la tabla era algo como:

MAC	Puerto
MAC_PC1	Fa0/1
MAC_PC2	Fa0/2
MAC_PC3	Fa0/3
MAC_PC4	Fa0/4

El switch solo necesitaba saber qué MAC estaba en qué puerto.

Con VLANs configuradas

Ahora la tabla incluye también la VLAN. El switch mantiene una tabla lógica por VLAN.

Ejemplo:

VLAN	MAC	Puerto
10	MAC_PC1	Fa0/1
10	MAC_PC2	Fa0/2
20	MAC_PC3	Fa0/3
20	MAC_PC4	Fa0/4
Diferencias principales

1️⃣ La tabla ahora está segmentada por VLAN
El switch aprende direcciones MAC dentro de cada VLAN.

2️⃣ Puede existir la misma MAC en VLAN distintas
Porque las tablas son independientes.

3️⃣ El tráfico se limita a su VLAN
Una MAC en VLAN 10 no se comunica con una MAC en VLAN 20.

4️⃣ Los broadcasts se separan
Cada VLAN tiene su propio dominio de broadcast.

(e) Administración del switch por Telnet

Para administrar el switch por Telnet, se necesita:

Crear una VLAN de administración (diferente a VLAN 1, 10 y 20).

Crear una SVI (Switch Virtual Interface).

Asignar un puerto a esa VLAN.

Configurar Telnet.

Supongamos:

VLAN	Uso
10	PCs 1–2
20	PCs 3–4
30	Administración
Paso 1: Crear VLAN de administración
enable
configure terminal

vlan 30
name ADMIN
exit
Paso 2: Asignar puerto para el PC administrador

Ejemplo: puerto Fa0/5

interface fastEthernet 0/5
switchport mode access
switchport access vlan 30
exit
Paso 3: Crear la interfaz virtual (SVI)
interface vlan 30
ip address 192.168.30.1 255.255.255.0
no shutdown
exit

Esta IP será la dirección para administrar el switch.

Paso 4: Configurar Telnet
line vty 0 4
password cisco
login
transport input telnet
exit

Opcionalmente:

enable password cisco
Paso 5: Configurar el PC de administración

Ejemplo:

Dispositivo	IP	VLAN
PC Admin	192.168.30.2	VLAN 30
Paso 6: Conexión por Telnet

Desde el PC administrador:

telnet 192.168.30.1

Esto abrirá la sesión de administración del switch.

Explicación del funcionamiento

La SVI VLAN 30 actúa como interfaz de administración del switch.

Solo los equipos en VLAN 30 pueden acceder a ella.

Las VLAN 10 y 20 permanecen aisladas del plano de administración.

El switch sigue siendo capa 2, pero puede tener una IP para gestión.

✅ Ventaja de usar una VLAN de administración separada

Mayor seguridad.

El tráfico de gestión no se mezcla con el tráfico de usuarios.

Permite controlar qué equipos pueden administrar el switch.
