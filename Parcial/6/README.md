# Parcial 6 - Inter-VLAN con 2 routers principal/backup

## 1) Objetivo

Implementar enrutamiento inter-VLAN con 2 enrutadores en esquema principal/backup para mantener servicio si uno falla.

Condiciones del enunciado:

- Ambos routers se conectan a switches de capas distintas.
- Debe existir redundancia para inter-VLAN.
- VLAN 40 debe enrutarse sin subinterfaces.
- Balanceo de primario por VLAN:
  - Un router principal para VLAN 10 y 30.
  - El otro router principal para VLAN 20 y 40.

## 2) Base de direccionamiento (continuidad con punto 5)

Se mantiene uso exclusivo de 10.10.8.0/24 dividido en /26:

- VLAN 10: 10.10.8.0/26
- VLAN 20: 10.10.8.64/26
- VLAN 30: 10.10.8.128/26
- VLAN 40: 10.10.8.192/26

Gateways virtuales (FHRP):

- VLAN 10 -> 10.10.8.1
- VLAN 20 -> 10.10.8.65
- VLAN 30 -> 10.10.8.129
- VLAN 40 -> 10.10.8.193

## 3) Diseno logico propuesto

- R1 conectado a switch de Core (ejemplo SW1).
- R2 conectado a switch de Distribucion (ejemplo SW2 o SW3).
- FHRP recomendado: HSRP por VLAN para gateway virtual redundante.
- VLAN 10, 20, 30 por Router-on-a-Stick (subinterfaces en G0/0).
- VLAN 40 sin subinterfaces: interfaz fisica dedicada G0/1 en cada router, como puerto de acceso VLAN 40 en el switch.

## 4) Plan de IP por router

### R1

- G0/0.10: 10.10.8.2/26
- G0/0.20: 10.10.8.66/26
- G0/0.30: 10.10.8.130/26
- G0/1 (VLAN 40 fisica): 10.10.8.194/26

### R2

- G0/0.10: 10.10.8.3/26
- G0/0.20: 10.10.8.67/26
- G0/0.30: 10.10.8.131/26
- G0/1 (VLAN 40 fisica): 10.10.8.195/26

## 5) Prioridades HSRP (principal/backup)

Distribucion de primario exigida:

- R1 principal en VLAN 10 y 30.
- R2 principal en VLAN 20 y 40.

Tabla sugerida:

| VLAN | Grupo HSRP | VIP | Primario | Prioridad primario | Backup |
|---|---|---|---|---|---|
| 10 | 10 | 10.10.8.1 | R1 | 120 | R2 |
| 20 | 20 | 10.10.8.65 | R2 | 120 | R1 |
| 30 | 30 | 10.10.8.129 | R1 | 120 | R2 |
| 40 | 40 | 10.10.8.193 | R2 | 120 | R1 |

Usar preempt para que el router primario recupere liderazgo al volver.

## 6) Configuracion de routers

## 6.1 R1

```bash
enable
configure terminal

interface g0/0
 no shutdown

interface g0/0.10
 encapsulation dot1Q 10
 ip address 10.10.8.2 255.255.255.192
 standby 10 ip 10.10.8.1
 standby 10 priority 120
 standby 10 preempt

interface g0/0.20
 encapsulation dot1Q 20
 ip address 10.10.8.66 255.255.255.192
 standby 20 ip 10.10.8.65
 standby 20 priority 100
 standby 20 preempt

interface g0/0.30
 encapsulation dot1Q 30
 ip address 10.10.8.130 255.255.255.192
 standby 30 ip 10.10.8.129
 standby 30 priority 120
 standby 30 preempt

interface g0/1
 ip address 10.10.8.194 255.255.255.192
 standby 40 ip 10.10.8.193
 standby 40 priority 100
 standby 40 preempt
 no shutdown

end
copy running-config startup-config
```

## 6.2 R2

```bash
enable
configure terminal

interface g0/0
 no shutdown

interface g0/0.10
 encapsulation dot1Q 10
 ip address 10.10.8.3 255.255.255.192
 standby 10 ip 10.10.8.1
 standby 10 priority 100
 standby 10 preempt

interface g0/0.20
 encapsulation dot1Q 20
 ip address 10.10.8.67 255.255.255.192
 standby 20 ip 10.10.8.65
 standby 20 priority 120
 standby 20 preempt

interface g0/0.30
 encapsulation dot1Q 30
 ip address 10.10.8.131 255.255.255.192
 standby 30 ip 10.10.8.129
 standby 30 priority 100
 standby 30 preempt

interface g0/1
 ip address 10.10.8.195 255.255.255.192
 standby 40 ip 10.10.8.193
 standby 40 priority 120
 standby 40 preempt
 no shutdown

end
copy running-config startup-config
```

## 7) Configuracion minima de switches para los enlaces a routers

### Enlace de trunk al router (G0/0)

En el puerto de switch que conecta a G0/0 de cada router:

```bash
interface fa0/x
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 switchport trunk native vlan 30
```

### Enlace fisico de VLAN 40 al router (G0/1, sin subinterfaces)

En el puerto de switch que conecta a G0/1 de cada router:

```bash
interface fa0/y
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
```

Con esto VLAN 40 se enruta por interfaz fisica, cumpliendo el requisito de no usar subinterfaces para esa VLAN.

## 8) Configuracion de hosts

Cada host debe usar como gateway la IP virtual HSRP de su VLAN:

- VLAN 10 -> 10.10.8.1
- VLAN 20 -> 10.10.8.65
- VLAN 30 -> 10.10.8.129
- VLAN 40 -> 10.10.8.193

## 9) Verificacion

En ambos routers:

```bash
show standby brief
show ip interface brief
show running-config | section interface g0/0.10
show running-config | section interface g0/0.20
show running-config | section interface g0/0.30
show running-config | section interface g0/1
```

Debes observar:

- R1 activo en grupos 10 y 30.
- R2 activo en grupos 20 y 40.
- Interfaces up/up.

## 10) Prueba de failover

1. Desde un host en cada VLAN, hacer ping a un host de otra VLAN.
2. Apagar R1 y repetir pruebas.
3. Encender R1 y verificar retorno de liderazgo en VLAN 10 y 30.
4. Apagar R2 y repetir pruebas.
5. Confirmar continuidad de inter-VLAN en ambos escenarios.

## 11) Checklist final

- 2 routers conectados a switches de capas distintas.
- Inter-VLAN con esquema principal/backup operativo.
- VLAN 40 enrutada sin subinterfaces (interfaz fisica dedicada).
- R1 primario en VLAN 10 y 30.
- R2 primario en VLAN 20 y 40.
- Todo el direccionamiento dentro de 10.10.8.0/24.
