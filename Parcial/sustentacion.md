# Sustentacion - Resumen corto (Parcial)

## a) Esquema de direccionamiento

Se trabajo unicamente con la red **10.10.8.0/24**, dividida en 4 subredes /26:

- VLAN 10: 10.10.8.0/26 - Gateway virtual 10.10.8.1
- VLAN 20: 10.10.8.64/26 - Gateway virtual 10.10.8.65
- VLAN 30: 10.10.8.128/26 - Gateway virtual 10.10.8.129
- VLAN 40 (administracion): 10.10.8.192/26 - Gateway virtual 10.10.8.193

Las interfaces de routers, gateways y SVI de administracion se asignaron dentro de estas subredes sin salir del bloque permitido.

## b) Conectividad entre VLAN y acceso remoto a switches

La conectividad inter-VLAN se implemento con **dos routers** en esquema principal/backup usando **HSRP**:

- R1 principal en VLAN 10 y 30
- R2 principal en VLAN 20 y 40

Para transporte:

- VLAN 10/20/30 por subinterfaces (Router-on-a-Stick)
- VLAN 40 por interfaz fisica dedicada (sin subinterfaces)

Acceso remoto a switches:

- Gestion por VLAN 40
- Telnet y SSH habilitados
- Credenciales usadas: enable secret `redes`, VTY `udea`, SSH admin/martes

## c) Flujo de un ping entre VLANs

Ejemplo: PC en VLAN 10 -> PC en VLAN 20.

1. El host origen detecta que el destino esta en otra red y envia al gateway virtual de su VLAN.
2. Si no conoce MAC del gateway, hace ARP.
3. El trafico cruza la red de switches por trunks y enlaces activos de STP.
4. El router activo enruta de VLAN 10 hacia VLAN 20 (capa 3) y reencapsula (capa 2).
5. Si falta MAC del destino, el router hace ARP en VLAN 20.
6. El host destino responde (Echo Reply) por su gateway virtual y vuelve al origen.
7. Si falla el router activo, HSRP conmuta al backup sin cambiar el gateway configurado en hosts.

## d) Estrategia STP

Se uso **PVST/Rapid-PVST** con prioridades manuales por VLAN:

- VLAN 10 y 20 con raiz en Core (SW1)
- VLAN 30 y 40 con raiz en distribucion (SW2) y secundario SW3

Resultado:

- Sin bucles de capa 2
- Camino activo controlado por VLAN
- Enlaces redundantes listos para contingencia
- Convergencia y estabilidad junto con EtherChannel + HSRP

## Cierre

La topologia cumple: direccionamiento restringido a 10.10.8.0/24, inter-VLAN con redundancia real, gestion remota por VLAN 40 y STP correctamente planeado por VLAN.