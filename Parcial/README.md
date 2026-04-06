# Parcial - Guia sencilla paso a paso

## 1) Objetivo
Implementar una red de 6 switches en topologia jerarquica:
- Core: SW1
- Distribucion: SW2, SW3
- Acceso: SW4, SW5, SW6

Debe cumplir:
- VLAN 10, 20, 30 y 40.
- Usuarios en VLAN 10, 20, 30 y 40 en switches de acceso.
- VLAN 40 para administracion (Telnet y SSH en todos los switches).
- Enable secret: redes.
- VTY password: udea.
- Usuario SSH: admin, password: martes.
- VLAN 30 nativa (sin etiqueta 802.1Q) en todos los trunks.

## 2) Topologia y puertos

Todas las conexiones entre switches son FastEthernet.

| Switch | Nivel | Enlace | Puerto local | Puerto remoto |
|---|---|---|---|---|
| SW1 | Core | SW1-SW2 | Fa0/1 | SW2 Fa0/1 |
| SW1 | Core | SW1-SW3 | Fa0/2 | SW3 Fa0/1 |
| SW2 | Distribucion | SW2-SW3 | Fa0/2 | SW3 Fa0/2 |
| SW2 | Distribucion | SW2-SW4 | Fa0/3 | SW4 Fa0/1 |
| SW3 | Distribucion | SW3-SW5 | Fa0/3 | SW5 Fa0/1 |
| SW3 | Distribucion | SW3-SW6 | Fa0/4 | SW6 Fa0/1 |
| SW4 | Acceso | SW4-SW5 | Fa0/2 | SW5 Fa0/2 |
| SW5 | Acceso | SW5-SW6 | Fa0/3 | SW6 Fa0/2 |

Puertos de usuario sugeridos:
- SW4: Fa0/4 VLAN 10, Fa0/5 VLAN 20, Fa0/6 VLAN 30, Fa0/7 VLAN 40
- SW5: Fa0/4 VLAN 10, Fa0/5 VLAN 20, Fa0/6 VLAN 30, Fa0/7 VLAN 40
- SW6: Fa0/3 VLAN 10, Fa0/4 VLAN 20, Fa0/5 VLAN 30, Fa0/6 VLAN 40

## 3) Crear VLANs en todos los switches

```bash
enable
configure terminal
vlan 10
 name USUARIOS10
vlan 20
 name USUARIOS20
vlan 30
 name USUARIOS30
vlan 40
 name ADMIN
copy running-config startup-config
```

## 4) Puertos de acceso (SW4, SW5, SW6)

SW4:

```bash
enable
configure terminal
interface fa0/4
 switchport mode access
 switchport access vlan 10
interface fa0/5
 switchport mode access
 switchport access vlan 20
interface fa0/6
 switchport mode access
 switchport access vlan 30
interface fa0/7
 switchport mode access
 switchport access vlan 40
copy running-config startup-config
```

SW5:

```bash
enable
configure terminal
interface fa0/4
 switchport mode access
 switchport access vlan 10
interface fa0/5
 switchport mode access
 switchport access vlan 20
interface fa0/6
 switchport mode access
 switchport access vlan 30
interface fa0/7
 switchport mode access
 switchport access vlan 40
copy running-config startup-config
```

SW6:

```bash
enable
configure terminal
interface fa0/3
 switchport mode access
 switchport access vlan 10
interface fa0/4
 switchport mode access
 switchport access vlan 20
interface fa0/5
 switchport mode access
 switchport access vlan 30
interface fa0/6
 switchport mode access
 switchport access vlan 40
copy running-config startup-config
```

## 5) Enlaces troncales (VLAN 30 nativa)

SW1:

```bash
enable
configure terminal
interface range fa0/1 - 2
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
copy running-config startup-config
```

SW2:

```bash
enable
configure terminal
interface range fa0/1 - 3
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
copy running-config startup-config
```

SW3:

```bash
enable
configure terminal
interface range fa0/1 - 4
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
copy running-config startup-config
```

SW4:

```bash
enable
configure terminal
interface range fa0/1 - 2
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
copy running-config startup-config
```

SW5:

```bash
enable
configure terminal
interface range fa0/1 - 3
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
copy running-config startup-config
```

SW6:

```bash
enable
configure terminal
interface range fa0/1 - 2
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
copy running-config startup-config
```

## 6) VLAN 40 de administracion (SVI)

Plan de direccionamiento:
- SW1: 192.168.40.1/24
- SW2: 192.168.40.2/24
- SW3: 192.168.40.3/24
- SW4: 192.168.40.4/24
- SW5: 192.168.40.5/24
- SW6: 192.168.40.6/24

Ejemplo (cambiar IP segun switch):

```bash
enable
configure terminal
interface vlan 40
 ip address 192.168.40.1 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.40.254
copy running-config startup-config
```

## 7) Telnet y SSH en todos los switches

```bash
enable
configure terminal
enable secret redes
ip domain-name empresa.local
username admin password martes
crypto key generate rsa modulus 1024
ip ssh version 2
line vty 0 4
 password udea
 login local
 transport input ssh
line vty 5 15
 password udea
 login
 transport input telnet
copy running-config startup-config
```

Cumplimiento remoto:
- Telnet: password de linea vty = udea.
- SSH: usuario = admin, password = martes.

## 8) Verificacion rapida

```bash
show vlan brief
show interfaces trunk
show ip interface brief
show running-config | include enable secret
show running-config | section line vty
show running-config | include username|ip ssh|transport input|password udea
```

Debe verse:
- VLAN 10,20,30,40 creadas.
- En SW4, SW5 y SW6 hay puertos de usuario en VLAN 10,20,30,40.
- Trunks activos con native vlan 30.
- Allowed VLAN en trunks: 10,20,30,40.
- Enable secret redes.
- VTY password udea.
- SSH habilitado con admin/martes.

## 9) Pruebas finales

Desde PC de administracion (VLAN 40):

```bash
telnet 192.168.40.1
ssh -l admin 192.168.40.1
```

Repetir contra SW2-SW6.

Desde usuarios:
- Probar ping entre hosts de VLAN 10.
- Probar ping entre hosts de VLAN 20.
- Probar ping entre hosts de VLAN 30.
- Probar ping entre hosts de VLAN 40.

## 10) Validacion de la condicion jerarquica

Se cumple la conectividad minima exigida:
- SW2 y SW3 (distribucion) conectan a su nivel superior (SW1) y entre si.
- SW4, SW5 y SW6 (acceso) conectan a distribucion y a su mismo nivel.
