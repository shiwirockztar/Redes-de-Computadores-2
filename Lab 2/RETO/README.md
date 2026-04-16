# Reto Lab 2: VLANs + STP (PVST)

Este documento resume la configuracion del reto en Packet Tracer para una red con segmentacion por VLANs y redundancia de enlaces, controlada con Spanning Tree Protocol (PVST).

## 1. Objetivo

Implementar una topologia con:

- VLAN 10 para administracion.
- VLAN 20 para usuarios.
- Enlaces troncales entre switches.
- Seleccion controlada de Root Bridge para optimizar STP.

## 2. VLANs y direccionamiento IP

### VLAN 10 (Administracion)

| Equipo | IP | Mascara |
|---|---|---|
| PC1 | 192.168.10.1 | 255.255.255.0 |
| PC4 | 192.168.10.2 | 255.255.255.0 |
| SW1 (SVI VLAN 10) | 192.168.10.11 | 255.255.255.0 |
| SW2 (SVI VLAN 10) | 192.168.10.12 | 255.255.255.0 |
| SW3 (SVI VLAN 10) | 192.168.10.13 | 255.255.255.0 |
| SW4 (SVI VLAN 10) | 192.168.10.14 | 255.255.255.0 |
| SW5 (SVI VLAN 10) | 192.168.10.15 | 255.255.255.0 |

### VLAN 20 (Usuarios)

| Equipo | IP | Mascara |
|---|---|---|
| PC2 | 192.168.20.1 | 255.255.255.0 |
| PC3 | 192.168.20.2 | 255.255.255.0 |

Nota: todos los switches deben tener creadas ambas VLANs (10 y 20) para mantener consistencia y permitir PVST por VLAN.

## 3. Configuracion base por switch

### 3.1 Crear VLANs

```bash
enable
configure terminal
vlan 10
 name ADMIN
vlan 20
 name USERS
end
copy running-config startup-config
```

### 3.2 Puertos de acceso para PCs

- En switches donde se conecten PCs de administracion (VLAN 10), configurar Fa0/10 como access VLAN 10.
- En switches donde se conecten PCs de usuarios (VLAN 20), configurar Fa0/10 como access VLAN 20.

Ejemplo (VLAN 10):

```bash
enable
configure terminal
interface fa0/10
 switchport mode access
 switchport access vlan 10
end
copy running-config startup-config
```

Ejemplo (VLAN 20):

```bash
enable
configure terminal
interface fa0/10
 switchport mode access
 switchport access vlan 20
end
copy running-config startup-config
```

### 3.3 SVI de administracion (VLAN 10)

Configurar la IP de gestion correspondiente en cada switch:

```bash
enable
configure terminal
interface vlan 10
 ip address 192.168.10.X 255.255.255.0
 no shutdown
end
copy running-config startup-config
```

## 4. Root Bridge (STP)

Se define **SW3** como Root Bridge para VLAN 10 y VLAN 20 por su posicion central en la topologia.

```bash
enable
configure terminal
spanning-tree vlan 10 priority 4096
spanning-tree vlan 20 priority 4096
end
copy running-config startup-config
```

Importante: la prioridad cambia de inmediato en el switch local, pero el arbol STP global se consolida cuando los switches intercambian BPDUs por enlaces troncales activos.

## 5. Enlaces troncales entre switches

Todos los enlaces entre switches se configuran como trunk y permitiendo VLAN 10 y 20.

```bash
enable
configure terminal
interface range fa0/1 - 8
 switchport mode trunk
 switchport trunk allowed vlan 10,20
end
copy running-config startup-config
```

No aplicar modo trunk a puertos conectados a PCs (por ejemplo Fa0/10).

## 6. Verificacion

Ejecutar en los switches:

```bash
show spanning-tree vlan 10
show spanning-tree vlan 20
show vlan brief
show interfaces trunk
```

Resultado esperado:

- 2 arboles STP (uno por VLAN) al usar PVST.
- SW3 apareciendo como Root Bridge en ambas VLANs.
- Enlaces redundantes en estado blocking segun el costo STP.

## 7. Analisis solicitado

### 7.1 Forma del arbol

- SW3 es el Root Bridge.
- Los caminos de menor costo convergen hacia SW3.
- Los enlaces alternos redundantes quedan bloqueados para evitar bucles.

### 7.2 Recorrido de trama (PC1 hacia SW2 en VLAN 10)

Recorrido esperado:

```text
PC1 -> SW4 -> SW3 (Root) -> SW2
```

### 7.3 Si se elimina el Root Bridge actual

Accion posible:

```bash
spanning-tree vlan 10 priority 61440
```

o apagar SW3.

Efecto: STP recalcula y el nuevo Root Bridge sera el switch con menor Bridge ID disponible (prioridad y, en empate, MAC).

### 7.4 Agregar un nuevo switch como Root

Antes de conectarlo a la red:

```bash
spanning-tree vlan 10 priority 0
spanning-tree vlan 20 priority 0
```

Efecto: ese switch pasa a ser candidato principal a Root Bridge para ambas VLANs.

## 8. Conclusiones

- La red queda segmentada correctamente por VLAN.
- PVST permite un arbol independiente por cada VLAN.
- Definir un Root central mejora la eficiencia de rutas STP.
- Los trunks son indispensables para intercambiar BPDUs y converger.
- STP responde automaticamente ante cambios de topologia.
