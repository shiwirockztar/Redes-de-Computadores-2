# Parcial 6 - Paso a paso operativo (basado en tu .pkt)

## 1) Conexion fisica a usar

Conexiones Router-Switch recomendadas (consistentes con parciales anteriores):

- R1 G0/0 <-> SW1 Fa0/7 (trunk para VLAN 10,20,30)
- R1 G0/1 <-> SW1 Fa0/8 (access VLAN 40)
- R2 G0/0 <-> SW2 Fa0/7 (trunk para VLAN 10,20,30)
- R2 G0/1 <-> SW2 Fa0/8 (access VLAN 40)

Si en tu .pkt esos puertos estan ocupados, cambia solo los numeros de puerto y conserva la logica:
- 1 puerto trunk por router (G0/0)
- 1 puerto access VLAN 40 por router (G0/1)

## 2) Direccionamiento (10.10.8.0/24 dividido en /26)

- VLAN 10: 10.10.8.0/26, gateway virtual 10.10.8.1
- VLAN 20: 10.10.8.64/26, gateway virtual 10.10.8.65
- VLAN 30: 10.10.8.128/26, gateway virtual 10.10.8.129
- VLAN 40: 10.10.8.192/26, gateway virtual 10.10.8.193

IP de routers:
- R1: G0/0.10 10.10.8.2, G0/0.20 10.10.8.66, G0/0.30 10.10.8.130, G0/1 10.10.8.194
- R2: G0/0.10 10.10.8.3, G0/0.20 10.10.8.67, G0/0.30 10.10.8.131, G0/1 10.10.8.195

## 3) Configurar SW1 (puertos hacia R1)

```bash
enable
configure terminal

vlan 10
vlan 20
vlan 30
vlan 40

interface fa0/7
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 switchport trunk native vlan 30
 no shutdown

interface fa0/8
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
 no shutdown

end
copy running-config startup-config
```

## 4) Configurar SW2 (puertos hacia R2)

```bash
enable
configure terminal

vlan 10
vlan 20
vlan 30
vlan 40

interface fa0/7
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 switchport trunk native vlan 30
 no shutdown

interface fa0/8
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
 no shutdown

end
copy running-config startup-config
```

## 5) Configurar R1 (primario en VLAN 10 y 30)

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

## 6) Configurar R2 (primario en VLAN 20 y 40)

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

## 7) Configurar PCs

Configura el gateway segun VLAN:

- VLAN 10 -> 10.10.8.1
- VLAN 20 -> 10.10.8.65
- VLAN 30 -> 10.10.8.129
- VLAN 40 -> 10.10.8.193

Ejemplo IPs de prueba:
- PC-VLAN10: 10.10.8.10/26
- PC-VLAN20: 10.10.8.70/26
- PC-VLAN30: 10.10.8.130/26
- PC-VLAN40: 10.10.8.200/26

## 8) Verificacion final

En routers:

```bash
show ip interface brief
show standby brief
show running-config | section interface g0/0.10
show running-config | section interface g0/0.20
show running-config | section interface g0/0.30
show running-config | section interface g0/1
```

En switches (SW1 y SW2):

```bash
show interfaces trunk
show vlan brief
show interfaces status
```

Resultado esperado:
- R1 activo en HSRP grupos 10 y 30.
- R2 activo en HSRP grupos 20 y 40.
- Inter-VLAN funcionando aun cuando se apaga uno de los routers.

## 9) Prueba de failover (obligatoria)

1. Haz ping entre VLANs (por ejemplo VLAN10 -> VLAN20).
2. Apaga R1 y repite ping: debe continuar por R2.
3. Enciende R1 y valida que recupere liderazgo en VLAN 10 y 30.
4. Apaga R2 y repite ping: debe continuar por R1.
5. Valida nuevamente con `show standby brief` en ambos routers.

## 10) Ajuste rapido si tus puertos no coinciden

Solo cambia estos bloques, no toda la configuracion:

- En SW1/SW2: interfaces fa0/7 y fa0/8 por los puertos reales.
- En routers: si no tienes G0/0 y G0/1, reemplaza por la interfaz disponible equivalente (ejemplo Fa0/0 y Fa0/1) manteniendo exactamente el mismo esquema de subinterfaces y HSRP.
