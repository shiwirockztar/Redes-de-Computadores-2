# Parcial 4 - Dimensionamiento de capacidad y no perdida de paquetes

## 1) Objetivo
Continuando con la topologia jerarquica del punto anterior, garantizar que:

- Cada switch de acceso entregue en promedio 250 Mbps hacia distribucion sin perdida por saturacion.
- Cada switch de distribucion entregue en promedio 275 Mbps hacia Core sin perdida por saturacion.
- Solo hay clientes en la capa de acceso.

## 2) Analisis rapido de capacidad

### 2.1 Trafico esperado

- Acceso -> Distribucion:
  - SW4: 250 Mbps
  - SW5: 250 Mbps
  - SW6: 250 Mbps
  - Total hacia distribucion: 750 Mbps

- Distribucion -> Core:
  - SW2: 275 Mbps
  - SW3: 275 Mbps
  - Total hacia Core: 550 Mbps

### 2.2 Implicacion tecnica

Un enlace FastEthernet individual (100 Mbps) no soporta 250 Mbps ni 275 Mbps. Por tanto, se requiere aumentar capacidad de uplinks.

## 3) Solucion propuesta

Usar EtherChannel (Port-Channel) con enlaces FastEthernet paralelos:

- Cada uplink Acceso -> Distribucion: 3 x FastEthernet = 300 Mbps utiles.
- Cada uplink Distribucion -> Core: 3 x FastEthernet = 300 Mbps utiles.

Con esto se cumple:

- 300 > 250 Mbps en acceso/distribucion.
- 300 > 275 Mbps en distribucion/core.

Margen por enlace:

- Acceso -> Distribucion: 300 - 250 = 50 Mbps.
- Distribucion -> Core: 300 - 275 = 25 Mbps.

## 4) Reasignacion de puertos (sugerida)

Manteniendo la estructura del parcial, se crean port-channels dedicados:

- SW4 <-> SW2: Fa0/1-3 (Po14)
- SW5 <-> SW3: Fa0/1-3 (Po15)
- SW6 <-> SW3: Fa0/1-3 (Po16)
- SW2 <-> SW1: Fa0/4-6 (Po12)
- SW3 <-> SW1: Fa0/5-7 (Po13)

Nota: ajusta numeracion si en tu modelo esos puertos no existen. La regla es usar 3 enlaces por cada uplink critico.

## 5) Configuracion base de EtherChannel (LACP)

Recomendacion: usar LACP en modo active en ambos extremos.

### 5.1 Ejemplo SW4 <-> SW2 (Po14)

En SW4:

```bash
enable
configure terminal
interface range fa0/1 - 3
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 channel-group 14 mode active
 no shutdown
exit
interface port-channel 14
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
copy running-config startup-config
```

En SW2:

```bash
enable
configure terminal
interface range fa0/1 - 3
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 channel-group 14 mode active
 no shutdown
exit
interface port-channel 14
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
copy running-config startup-config
```

### 5.2 Replicar para los demas uplinks

Usa el mismo patron para:

- Po15: SW5 <-> SW3
- Po16: SW6 <-> SW3
- Po12: SW2 <-> SW1
- Po13: SW3 <-> SW1

## 6) STP sobre Port-Channel

Para evitar inconsistencias y mantener convergencia limpia:

- Configura STP sobre interfaces Port-Channel (no solo sobre miembros fisicos).
- Conserva las prioridades STP del punto 3 del parcial anterior.
- Enlaces agregados se ven como un enlace logico, reduciendo bloqueos innecesarios por puerto fisico.

Comandos utiles:

```bash
show spanning-tree vlan 10
show spanning-tree vlan 20
show etherchannel summary
show interfaces port-channel 12
show interfaces port-channel 13
show interfaces port-channel 14
show interfaces port-channel 15
show interfaces port-channel 16
```

## 7) Verificacion de no perdida a trafico promedio

## 7.1 Validacion de capacidad por enlace

Debes confirmar que todos los uplinks criticos esten en Port-Channel con 3 miembros activos (SU o RU segun plataforma) en `show etherchannel summary`.

Resultado esperado:

- Cada Po de uplink con ancho de banda agregado de 300 Mbps.
- Sin miembro suspendido o standalone.

### 7.2 Validacion operativa

1. Genera trafico entre clientes de acceso en distintas VLAN (segun escenario del parcial).
2. Verifica que no haya caidas intermitentes en ping durante carga promedio.
3. Revisa contadores de descarte:

```bash
show interfaces counters errors
show interfaces fa0/1
show interfaces fa0/2
show interfaces fa0/3
show interfaces port-channel 14
```

Repite para los otros Port-Channel.

Condicion de cumplimiento:

- Sin incremento significativo de drops por congestion en los uplinks agregados bajo carga promedio de 250/275 Mbps.

## 8) Checklist final

- Uplinks Acceso -> Distribucion en EtherChannel de 3xFE.
- Uplinks Distribucion -> Core en EtherChannel de 3xFE.
- Trunks con VLAN 10,20,30,40 y native VLAN 30 consistentes.
- STP estable sobre interfaces Port-Channel.
- Sin sintomas de saturacion en trafico promedio exigido.

## 9) Plan alterno recomendado

Si tu switch virtual soporta GigabitEthernet entre capas, una alternativa mas simple es:

- 1 enlace Gigabit por uplink critico.

Como 1000 Mbps > 250 y > 275, tambien cumple sin perdida por capacidad promedio y simplifica la configuracion frente a EtherChannel.
