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

- SW4 <-> SW2: Fa0/1-3 (Po1)
- SW5 <-> SW3: Fa0/1-3 (Po2)
- SW6 <-> SW3: SW6 Fa0/1-3 <-> SW3 Fa0/4-6 (Po3)
- SW2 <-> SW1: SW2 Fa0/4-6 <-> SW1 Fa0/1-3 (Po4)
- SW3 <-> SW1: SW3 Fa0/7-9 <-> SW1 Fa0/4-6 (Po5)

Enlaces laterales adicionales (recomendados en Gigabit para contingencia):

- SW2 <-> SW3: SW2 Gi0/1 <-> SW3 Gi0/1
- SW4 <-> SW5: SW4 Gi0/1 <-> SW5 Gi0/1
- SW5 <-> SW6: SW5 Gi0/2 <-> SW6 Gi0/1

Nota: ajusta numeracion si en tu modelo esos puertos no existen. La regla es usar 3 enlaces por cada uplink critico.

### 4.1 Mapa de puertos por switch (para reconexion)

SW1 (Core):
- Fa0/1-3 -> hacia SW2 (Po4)
- Fa0/4-6 -> hacia SW3 (Po5)

SW2 (Distribucion):
- Fa0/1-3 -> hacia SW4 (Po1)
- Fa0/4-6 -> hacia SW1 (Po4)
- Gi0/1 -> hacia SW3 (enlace lateral)

SW3 (Distribucion):
- Fa0/1-3 -> hacia SW5 (Po2)
- Fa0/4-6 -> hacia SW6 (Po3)
- Fa0/7-9 -> hacia SW1 (Po5)
- Gi0/1 -> hacia SW2 (enlace lateral)

SW4 (Acceso):
- Fa0/1-3 -> hacia SW2 (Po1)
- Gi0/1 -> hacia SW5 (enlace lateral)

SW5 (Acceso):
- Fa0/1-3 -> hacia SW3 (Po2)
- Gi0/1 -> hacia SW4 (enlace lateral)
- Gi0/2 -> hacia SW6 (enlace lateral)

SW6 (Acceso):
- Fa0/1-3 -> hacia SW3 (Po3)
- Gi0/1 -> hacia SW5 (enlace lateral)

Importante:
- Verifica que un mismo puerto fisico no quede asignado a dos Port-Channel distintos.
- Configura los enlaces laterales como trunk y evita agregarlos a un channel-group si se desean mantener como enlaces independientes.
- Mantener Fa0/4-Fa0/7 de SW4, SW5 y SW6 para puertos de acceso de usuarios (VLAN 10/20/30/40).
- Objetivo de los enlaces laterales en Gigabit: evitar congestion y perdida de paquetes en escenarios de falla/convergencia.
- Si tu modelo no tiene suficientes FastEthernet para dos uplinks de 3 enlaces en un mismo switch, redisena numeracion o usa puertos Gigabit.

## 5) Configuracion base de EtherChannel (LACP)

Recomendacion: usar LACP en modo active en ambos extremos.

### 5.1 Ejemplo SW4 <-> SW2 (Po1)

En SW4:

```bash
enable
configure terminal
interface range fa0/1 - 3
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 channel-group 1 mode active
 no shutdown
exit
interface port-channel 1
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
 channel-group 1 mode active
 no shutdown
exit
interface port-channel 1
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
copy running-config startup-config
```

### 5.2 Replicar para los demas uplinks

Usa el mismo patron para:

- Po2: SW5 <-> SW3
- Po3: SW6 <-> SW3
- Po4: SW2 <-> SW1
- Po5: SW3 <-> SW1

```bash
show etherchannel summary
show interfaces trunk
```

#### Po2: SW5 <-> SW3

En SW5:

```bash
enable
configure terminal
interface range fa0/1 - 3
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 channel-group 2 mode active
 no shutdown
exit
interface port-channel 2
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
copy running-config startup-config
```

En SW3:

```bash
enable
configure terminal
interface range fa0/1 - 3
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 channel-group 2 mode active
 no shutdown
exit
interface port-channel 2
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
copy running-config startup-config
```

#### Po3: SW6 <-> SW3

En SW6:

```bash
enable
configure terminal
interface range fa0/1 - 3
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 channel-group 3 mode active
 no shutdown
exit
interface port-channel 3
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
copy running-config startup-config
```

En SW3:

```bash
enable
configure terminal
interface range fa0/4 - 6
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 channel-group 3 mode active
 no shutdown
exit
interface port-channel 3
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
copy running-config startup-config
```

#### Po4: SW2 <-> SW1

En SW2:

```bash
enable
configure terminal
interface range fa0/4 - 6
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 channel-group 4 mode active
 no shutdown
exit
interface port-channel 4
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
copy running-config startup-config
```

En SW1:

```bash
enable
configure terminal
interface range fa0/1 - 3
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 channel-group 4 mode active
 no shutdown
exit
interface port-channel 4
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
copy running-config startup-config
```

#### Po5: SW3 <-> SW1

En SW3:

```bash
enable
configure terminal
interface range fa0/7 - 9
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 channel-group 5 mode active
 no shutdown
exit
interface port-channel 5
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
copy running-config startup-config
```

En SW1:

```bash
enable
configure terminal
interface range fa0/4 - 6
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 channel-group 5 mode active
 no shutdown
exit
interface port-channel 5
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
copy running-config startup-config
```

### 5.3 Configuracion de enlaces laterales Gigabit (trunk)

Estos enlaces se mantienen fuera de EtherChannel y se configuran como troncales independientes.

SW2 (hacia SW3 por Gi0/1):

```bash
enable
configure terminal
interface gi0/1
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
copy running-config startup-config
```

SW3 (hacia SW2 por Gi0/1):

```bash
enable
configure terminal
interface gi0/1
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
copy running-config startup-config
```

SW4 (hacia SW5 por Gi0/1):

```bash
enable
configure terminal
interface gi0/1
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
copy running-config startup-config
```

SW5 (hacia SW4 por Gi0/1 y hacia SW6 por Gi0/2):

```bash
enable
configure terminal
interface range gi0/1 - 2
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
copy running-config startup-config
```

SW6 (hacia SW5 por Gi0/1):

```bash
enable
configure terminal
interface gi0/1
 switchport mode trunk
 switchport trunk native vlan 30
 switchport trunk allowed vlan 10,20,30,40
 no shutdown
end
copy running-config startup-config
```

Verificacion recomendada:

```bash
show interfaces trunk
show interfaces gi0/1 switchport
show interfaces gi0/2 switchport
```

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
show interfaces port-channel 1
show interfaces port-channel 2
show interfaces port-channel 3
show interfaces port-channel 4
show interfaces port-channel 5
```

## 7) Verificacion de no perdida a trafico promedio

## 7.1 Validacion de capacidad por enlace

Debes confirmar que todos los uplinks criticos esten en Port-Channel con 3 miembros activos en `show etherchannel summary`.

Para 2960-24TT, estado esperado:

- Port-Channel en `SU`.
- Miembros con `P` (bundled).
- Protocolo `LACP` (si usas `mode active`).

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
show interfaces port-channel 1
show etherchannel summary
show lacp neighbor
```

Repite para los otros Port-Channel.

Condicion de cumplimiento:

- Sin incremento significativo de drops por congestion en los uplinks agregados bajo carga promedio de 250/275 Mbps.

## 8) Checklist final

- Uplinks Acceso -> Distribucion en EtherChannel de 3xFE.
- Uplinks Distribucion -> Core en EtherChannel de 3xFE.
- Enlaces laterales (SW2-SW3, SW4-SW5, SW5-SW6) en Gigabit y en trunk para contingencia.
- Trunks con VLAN 10,20,30,40 y native VLAN 30 consistentes.
- STP estable sobre interfaces Port-Channel.
- Sin sintomas de saturacion en trafico promedio exigido.

## 9) Plan alterno recomendado

Si tu switch virtual soporta GigabitEthernet entre capas, una alternativa mas simple es:

- 1 enlace Gigabit por uplink critico.

Como 1000 Mbps > 250 y > 275, tambien cumple sin perdida por capacidad promedio y simplifica la configuracion frente a EtherChannel.
