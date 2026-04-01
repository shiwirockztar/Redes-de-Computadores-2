# Laboratorio 3 - Punto 2

## 1. Objetivo

Adicionar un segundo router a la topologia del punto 1 y configurar redundancia de gateway con HSRP para VLAN 10, VLAN 20 y VLAN 99, garantizando conectividad completa entre todas las VLAN.

## 2. Topologia propuesta

- Se mantienen S1, S2 y S3 del punto 1.
- Se agregan 2 routers: R1 y R2.
- S3 queda como switch de distribucion para los routers.
- Enlaces:
   - S3 <-> R1: trunk 802.1Q por Fa0/10
   - S3 <-> R2: trunk 802.1Q por Fa0/11
  - S1 <-> S3 y S2 <-> S3: trunk (igual que en punto 1)

### 2.1 Conexion fisica final (ajustada a la topologia del punto 1)

| Dispositivo A | Puerto A | Dispositivo B | Puerto B | Uso |
| --- | --- | --- | --- | --- |
| R1 | G0/0 | S3 | Fa0/10 | Trunk 802.1Q VLAN 10,20,99 |
| R2 | G0/0 | S3 | Fa0/11 | Trunk 802.1Q VLAN 10,20,99 |
| S3 | Fa0/12 | Libre | - | Reserva / no usado en este punto |

Nota: en el punto 1, Fa0/10, Fa0/11 y Fa0/12 estaban en modo access hacia un solo router. En este punto 2 se reutilizan Fa0/10 y Fa0/11 como trunks hacia R1 y R2 para implementar HSRP.

## 3. Plan de direccionamiento

Para conservar compatibilidad con el punto 1, las IP virtuales HSRP quedan como gateway .1 en cada VLAN.

| VLAN | Red | Gateway virtual HSRP | R1 | R2 |
| --- | --- | --- | --- | --- |
| 10 | 192.168.10.0/24 | 192.168.10.1 | 192.168.10.2 | 192.168.10.3 |
| 20 | 192.168.20.0/24 | 192.168.20.1 | 192.168.20.2 | 192.168.20.3 |
| 99 | 192.168.99.0/24 | 192.168.99.1 | 192.168.99.2 | 192.168.99.3 |

## 4. Configuracion en S3 (trunks hacia routers)

Configuracion final: R1 en Fa0/10 y R2 en Fa0/11.

```bash
enable
configure terminal
interface fa0/10
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99

interface fa0/11
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99

interface fa0/12
 shutdown
end
copy running-config startup-config
```

## 5. Configuracion de R1 (router-on-a-stick + HSRP)

Se usa la interfaz fisica `g0/0` hacia S3.

```bash
enable
configure terminal
interface g0/0
 no shutdown

interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.2 255.255.255.0
standby version 2
 standby 10 ip 192.168.10.1
 standby 10 priority 110
 standby 10 preempt

interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.2 255.255.255.0
standby version 2
 standby 10 ip 192.168.20.1
 standby 10 priority 110
 standby 10 preempt

interface g0/0.99
 encapsulation dot1Q 99
 ip address 192.168.99.2 255.255.255.0
standby version 2
 standby 10 ip 192.168.99.1
 standby 10 priority 110
 standby 10 preempt
end
copy running-config startup-config
```

## 6. Configuracion de R2 (router-on-a-stick + HSRP)

Tambien sobre `g0/0` hacia S3.

```bash
enable
configure terminal
interface g0/0
 no shutdown

interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.3 255.255.255.0
standby version 2
 standby 10 ip 192.168.10.1
 standby 10 priority 100
 standby 10 preempt

interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.3 255.255.255.0
standby version 2
 standby 10 ip 192.168.20.1
 standby 10 priority 100
 standby 10 preempt

interface g0/0.99
 encapsulation dot1Q 99
 ip address 192.168.99.3 255.255.255.0
standby version 2
 standby 10 ip 192.168.99.1
 standby 10 priority 100
 standby 10 preempt
end
copy running-config startup-config
```

## 7. Gateways por defecto de hosts y switches

- PC en VLAN 10: gateway `192.168.10.1`
- PC en VLAN 20: gateway `192.168.20.1`
- Switches S1/S2/S3 (gestion VLAN 99): `ip default-gateway 192.168.99.1`

Con esto, todos apuntan al gateway virtual y no dependen de una IP fisica de router.

## 8. Verificacion de redundancia y conectividad

En R1 y R2:

```bash
show standby brief
show ip interface brief
show ip route
```

Pruebas recomendadas desde PCs y switches:

```bash
ping 192.168.10.10
ping 192.168.20.10
ping 192.168.99.11
ping 192.168.99.12
ping 192.168.99.13
```

Prueba de falla:

1. Apagar `g0/0` en R1 (router activo esperado).
2. Validar en R2 que pase a estado Active en `show standby brief`.
3. Repetir pings entre VLANs y confirmar continuidad.

## 9. Numeral (a): camino completo de un paquete (PC -> SVI del switch)

Escenario de ejemplo:

- PC1 en VLAN 10 (192.168.10.10).
- Destino: SVI de S3 en VLAN 99 (192.168.99.13).
- Gateway por defecto del PC: 192.168.10.1 (virtual HSRP grupo 10).
- R1 activo, R2 standby (si no hay falla).

Proceso detallado:

1. El PC crea un ICMP Echo Request (capa 3) con:
   - IP origen: 192.168.10.10
   - IP destino: 192.168.99.13

2. El PC revisa su tabla de enrutamiento local.
   - Como 192.168.99.13 no esta en su red /24, decide enviar al gateway por defecto 192.168.10.1.

3. ARP en VLAN 10 (capa 2).
   - Si no conoce la MAC del gateway virtual, el PC envia ARP Request broadcast: quien tiene 192.168.10.1?
   - El router activo HSRP responde con la MAC virtual del grupo.
   - El PC guarda entrada ARP: 192.168.10.1 -> MAC virtual HSRP.

4. El PC encapsula y envia la trama al switch de acceso.
   - Trama Ethernet:
     - MAC origen: MAC del PC
     - MAC destino: MAC virtual HSRP
   - Paquete IP interno:
     - origen 192.168.10.10, destino 192.168.99.13

5. El trafico cruza la red de switches hasta S3.
   - En puertos access viaja sin etiqueta.
   - En enlaces trunk, cada switch agrega/retira etiqueta 802.1Q VLAN 10 segun corresponda.

6. Tramo S3 -> routers (zona critica de redundancia).
   - S3 envia la trama por el trunk a los routers en VLAN 10 (802.1Q).
   - Ambos routers escuchan hello HSRP, pero solo el activo posee la MAC virtual en forwarding para ese grupo.
   - El standby no enruta trafico de la MAC virtual.

7. Enrutamiento en el router activo (capa 3).
   - El router recibe en subinterfaz .10, quita cabecera Ethernet y procesa IP.
   - Consulta tabla de enrutamiento: 192.168.99.0/24 esta conectada directamente por subinterfaz .99.
   - Decrementa TTL y recalcula checksum IP.

8. Resolucion ARP de salida en VLAN 99.
   - Si no conoce MAC de 192.168.99.13, envia ARP Request en VLAN 99.
   - S3 responde con la MAC de su interfaz VLAN 99.

9. Reencapsulacion hacia VLAN 99.
   - Nueva trama Ethernet:
     - MAC origen: MAC de R1/R2 en subinterfaz VLAN 99
     - MAC destino: MAC de S3 VLAN 99
   - El frame sale etiquetado 802.1Q VLAN 99 por el trunk.

10. S3 recibe, quita etiqueta 802.1Q y entrega al plano de control.
    - Como IP destino es su SVI (192.168.99.13), procesa el ICMP Echo Request y genera Echo Reply.

11. Camino de regreso.
    - S3 envia respuesta al gateway virtual 192.168.99.1.
    - ARP para MAC virtual de HSRP grupo 99.
    - Router activo enruta de vuelta hacia VLAN 10 y el PC recibe respuesta.

12. Comportamiento ante falla.
    - Si falla el router activo, el standby toma estado Active y asume IP/MAC virtual.
    - Hosts y switches siguen usando el mismo gateway virtual, sin reconfiguracion.
    - Se mantiene conectividad inter-VLAN tras la convergencia HSRP.

## 10. Conclusiones

- HSRP permite gateway redundante por VLAN usando una IP virtual estable.
- 802.1Q permite transportar multiples VLAN por un mismo enlace fisico hacia los routers.
- ARP resuelve siempre hacia MAC virtual HSRP para el gateway.
- El enrutamiento entre VLAN lo ejecuta el router activo consultando su tabla de rutas conectadas.
- Con la prueba de falla se valida continuidad del servicio y conectividad completa entre VLAN 10, 20 y 99.
