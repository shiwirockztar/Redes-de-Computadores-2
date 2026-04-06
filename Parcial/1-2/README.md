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
exit
end
copy running-config startup-config
```

## 4) Puertos de acceso (SW4, SW5, SW6)


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
exit
end
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
exit
end
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
exit
end
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
exit
end
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
exit
end
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
exit
end
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
exit
end
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
end
copy running-config startup-config
```

## 7) Telnet y SSH en todos los switches

Primero configura Telnet:

```bash
ena
conf t
line vty 0 15
password udea
enable password redes
login
end
copy running-config startup-config
```

```bash
enable
configure terminal
enable secret redes
line vty 0 15
 password udea
 login
 transport input telnet
end
copy running-config startup-config
```

Luego configura SSH:

```bash
enable
configure terminal
ip domain-name empresa.local
username admin password martes
crypto key generate rsa modulus 1024
ip ssh version 2
line vty 0 15
 login local
 transport input ssh telnet
end
copy running-config startup-config
```

Codigo completo :

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

## 9) Pruebas paso a paso (puertos, conexiones y PCs)

### 9.1 Revisar puertos y enlaces fisicos

1. En Packet Tracer, confirma que cada enlace switch-switch este en los puertos correctos:
	- SW1 Fa0/1 <-> SW2 Fa0/1
	- SW1 Fa0/2 <-> SW3 Fa0/1
	- SW2 Fa0/2 <-> SW3 Fa0/2
	- SW2 Fa0/3 <-> SW4 Fa0/1
	- SW3 Fa0/3 <-> SW5 Fa0/1
	- SW3 Fa0/4 <-> SW6 Fa0/1
	- SW4 Fa0/2 <-> SW5 Fa0/2
	- SW5 Fa0/3 <-> SW6 Fa0/2
2. Verifica que los LEDs esten en verde (enlace activo).
3. Ejecuta en todos los switches:

```bash
show interfaces status
show interfaces trunk
```

Que debes validar:
- Todos los puertos de enlace entre switches deben salir como trunk.
- Native VLAN debe ser la 30 en todos los trunks.
- Allowed VLANs debe incluir 10,20,30,40.

### 9.2 Configurar PCs para pruebas por VLAN

Conecta una PC por VLAN (puedes usar 4 PCs en total):
- PC-VLAN10 al puerto de acceso VLAN 10 (ejemplo SW4 Fa0/4)
- PC-VLAN20 al puerto de acceso VLAN 20 (ejemplo SW4 Fa0/5)
- PC-VLAN30 al puerto de acceso VLAN 30 (ejemplo SW4 Fa0/6)
- PC-ADMIN (VLAN40) al puerto de acceso VLAN 40 (ejemplo SW4 Fa0/7)

Configura IP estatica en Desktop > IP Configuration:
- PC-VLAN10: IP 192.168.10.10 /24, Gateway 192.168.10.254
- PC-VLAN20: IP 192.168.20.10 /24, Gateway 192.168.20.254
- PC-VLAN30: IP 192.168.30.10 /24, Gateway 192.168.30.254
- PC-ADMIN: IP 192.168.40.10 /24, Gateway 192.168.40.254

Nota:
- Si no tienes router o switch capa 3 para inter-VLAN, no habra comunicacion entre VLANs distintas. En ese caso, las pruebas principales son L2 (misma VLAN) y gestion en VLAN 40.

### 9.3 Validar asignacion de VLAN en puertos de acceso

En SW4, SW5 y SW6:

```bash
show vlan brief
```

Debes ver cada puerto de usuario en la VLAN correcta:
- VLAN 10: puertos de usuarios 10
- VLAN 20: puertos de usuarios 20
- VLAN 30: puertos de usuarios 30
- VLAN 40: puertos de administracion/gestion

### 9.4 Prueba de conectividad por capa 2 (misma VLAN)

1. Coloca otra PC en VLAN 10 en otro switch de acceso (por ejemplo SW5 Fa0/4) con IP 192.168.10.11/24.
2. Desde PC-VLAN10 (192.168.10.10), ejecuta:

```bash
ping 192.168.10.11
```

3. Repite lo mismo para VLAN 20 y VLAN 30 con dos PCs por VLAN.

Resultado esperado:
- Ping exitoso entre equipos de la misma VLAN aunque esten en switches diferentes.

### 9.5 Prueba de administracion en VLAN 40

Desde PC-ADMIN (192.168.40.10):

1. Verifica alcance IP a las SVI de switches:

```bash
ping 192.168.40.1
ping 192.168.40.2
ping 192.168.40.3
ping 192.168.40.4
ping 192.168.40.5
ping 192.168.40.6
```

2. Prueba Telnet (si lo dejaste habilitado):

```bash
telnet 192.168.40.1
```

Credencial esperada:
- Password VTY: udea

3. Prueba SSH:

```bash
ssh -l admin 192.168.40.1
```

Credenciales esperadas:
- Usuario: admin
- Password: martes

4. Repite Telnet/SSH contra 192.168.40.2 a 192.168.40.6.

### 9.6 Checklist final de validacion

- Trunks arriba entre todos los switches esperados.
- Native VLAN 30 consistente en todos los trunks.
- VLAN 10,20,30,40 creadas en todos los switches.
- Puertos de usuarios en la VLAN correcta.
- SVI VLAN 40 activas con IP correcta por switch.
- Telnet responde con password udea.
- SSH responde con usuario admin y password martes.
- Configuracion guardada:

```bash
copy running-config startup-config
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
