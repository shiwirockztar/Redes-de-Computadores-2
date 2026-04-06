# Parcial - Diseño jerárquico con VLAN, trunk, Telnet y SSH

## 1. Objetivo
Implementar en Packet Tracer una red de 6 switches con arquitectura jerárquica:
- Núcleo (Core): SW1
- Distribución: SW2, SW3
- Acceso: SW4, SW5, SW6

Requisitos funcionales:
- VLAN de usuarios: 10, 20, 30
- VLAN de administración: 40
- VLAN 30 debe ser nativa en todos los enlaces troncales (viaja sin etiqueta 802.1Q)
- Enable secret en todos los switches: redes
- VTY password: udea
- Usuario SSH: admin / martes
- Habilitar Telnet y SSH en todos los switches

## 2. Topología y puertos sugeridos

Todas las conexiones entre switches son FastEthernet.

| Switch | Nivel | Enlace | Puerto local | Puerto remoto |
|---|---|---|---|---|
| SW1 | Core | SW1-SW2 | Fa0/1 | SW2 Fa0/1 |
| SW1 | Core | SW1-SW3 | Fa0/2 | SW3 Fa0/1 |
| SW2 | Distribución | SW2-SW3 | Fa0/2 | SW3 Fa0/2 |
| SW2 | Distribución | SW2-SW4 | Fa0/3 | SW4 Fa0/1 |
| SW3 | Distribución | SW3-SW5 | Fa0/3 | SW5 Fa0/1 |
| SW3 | Distribución | SW3-SW6 | Fa0/4 | SW6 Fa0/1 |
| SW4 | Acceso | SW4-SW5 | Fa0/2 | SW5 Fa0/2 |
| SW5 | Acceso | SW5-SW6 | Fa0/3 | SW6 Fa0/2 |

Puertos de usuarios sugeridos (acceso):
- SW4: Fa0/4 (VLAN 10), Fa0/5 (VLAN 20), Fa0/6 (VLAN 30)
- SW5: Fa0/4 (VLAN 10), Fa0/5 (VLAN 20), Fa0/6 (VLAN 30)
- SW6: Fa0/3 (VLAN 10), Fa0/4 (VLAN 20), Fa0/5 (VLAN 30)

## 3. Plan de direccionamiento de gestión (VLAN 40)

Red de administración: 192.168.40.0/24

| Equipo | IP gestión SVI VLAN 40 |
|---|---|
| SW1 | 192.168.40.1/24 |
| SW2 | 192.168.40.2/24 |
| SW3 | 192.168.40.3/24 |
| SW4 | 192.168.40.4/24 |
| SW5 | 192.168.40.5/24 |
| SW6 | 192.168.40.6/24 |

Gateway de administración (si existe router/L3): 192.168.40.254

## 4. Configuración base común (ejecutar en TODOS los switches)

Configura nombre del equipo según corresponda (SW1, SW2, ..., SW6):

```cisco
enable
configure terminal
hostname SWX

enable secret redes

vlan 10
 name USUARIOS10
vlan 20
 name USUARIOS20
vlan 30
 name USUARIOS30_NATIVE
vlan 40
 name ADMIN

service password-encryption
ip domain-name empresa.local
username admin secret martes
crypto key generate rsa modulus 1024
ip ssh version 2

line vty 0 4
 login local
 transport input ssh
line vty 5 15
 password udea
 login
 transport input telnet
 exec-timeout 10 0
end
write memory
```

Nota: esta configuración separa VTY para cumplir explícitamente ambos requisitos:
- SSH autenticando con usuario local admin/martes (vty 0-4).
- Telnet autenticando con password de línea udea (vty 5-15).

## 5. Troncales por switch (VLAN 30 nativa)

## SW1

```cisco
enable
configure terminal
interface range fa0/1 - 2
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
write memory
```

## SW2

```cisco
enable
configure terminal
interface range fa0/1 - 3
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
write memory
```

## SW3

```cisco
enable
configure terminal
interface range fa0/1 - 4
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
write memory
```

## SW4

```cisco
enable
configure terminal
interface range fa0/1 - 2
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
write memory
```

## SW5

```cisco
enable
configure terminal
interface range fa0/1 - 3
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
write memory
```

## SW6

```cisco
enable
configure terminal
interface range fa0/1 - 2
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
write memory
```

## 6. Puertos de acceso para usuarios (SW4, SW5, SW6)

## SW4

```cisco
enable
configure terminal
interface fa0/4
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
interface fa0/5
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
interface fa0/6
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
end
write memory
```

## SW5

```cisco
enable
configure terminal
interface fa0/4
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
interface fa0/5
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
interface fa0/6
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
end
write memory
```

## SW6

```cisco
enable
configure terminal
interface fa0/3
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
interface fa0/4
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
interface fa0/5
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
end
write memory
```

## 7. SVI de administración (VLAN 40)

Configurar en cada switch la IP correspondiente de la tabla de la sección 3.

Ejemplo SW1:

```cisco
enable
configure terminal
interface vlan 40
 ip address 192.168.40.1 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.40.254
end
write memory
```

Repetir en SW2-SW6 cambiando únicamente la IP.

## 8. Recomendación STP para estabilidad (opcional pero sugerida)

Para evitar que el root bridge quede aleatorio:

En SW1 (core):

```cisco
enable
configure terminal
spanning-tree vlan 10,20,30,40 root primary
end
write memory
```

En SW2 (respaldo):

```cisco
enable
configure terminal
spanning-tree vlan 10,20,30,40 root secondary
end
write memory
```

## 9. Verificación de cumplimiento (checklist)

Ejecutar en todos los switches:

```cisco
show vlan brief
show interfaces trunk
show spanning-tree vlan 10
show spanning-tree vlan 20
show spanning-tree vlan 30
show spanning-tree vlan 40
show ip interface brief
show running-config | include enable secret
show running-config | section line vty
show running-config | include username|ip ssh|transport input|password udea|login local
```

Qué debes confirmar:
- Existen VLAN 10, 20, 30 y 40 en todos los switches.
- Todos los enlaces entre switches están en trunk.
- La VLAN nativa en todos los trunks es la 30.
- Allowed VLAN en trunks incluye 10,20,30,40.
- Cada switch tiene SVI VLAN 40 con IP única y estado up/up (cuando haya puertos activos en VLAN 40).
- Existe enable secret redes.
- En líneas VTY existe password udea (vty 5-15).
- SSH está habilitado con login local (vty 0-4).
- Telnet está habilitado con login por password de línea (vty 5-15).
- Usuario local admin con contraseña martes existe para acceso por SSH.

## 10. Pruebas finales sugeridas

Desde un PC en VLAN 10:
- Ping a otro PC en VLAN 10 conectado a otro switch de acceso.
- Ping a 192.168.40.1, 192.168.40.2, ..., 192.168.40.6.

Desde un PC de administración (VLAN 40):

```bash
telnet 192.168.40.1
ssh -l admin 192.168.40.1
```

Repetir para los demás switches.

## 11. Validación de la condición jerárquica

Se cumple la condición pedida:
- Distribución:
	- SW2 conectado al nivel superior (SW1) y a su mismo nivel (SW3).
	- SW3 conectado al nivel superior (SW1) y a su mismo nivel (SW2).
- Acceso:
	- SW4 conectado al nivel superior (SW2) y a su mismo nivel (SW5).
	- SW5 conectado al nivel superior (SW3) y a su mismo nivel (SW4 y SW6).
	- SW6 conectado al nivel superior (SW3) y a su mismo nivel (SW5).

Con esto tu topología y configuración cumplen el procedimiento solicitado.