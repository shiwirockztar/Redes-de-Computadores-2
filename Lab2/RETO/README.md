# Configuración de Red con VLANs y Spanning Tree

Este documento describe la configuración de red para una infraestructura segmentada por VLANs, con direccionamiento IP adecuado y la implementación de Spanning Tree Protocol (STP) para asegurar la redundancia y el balanceo de carga entre los switches.

## ✅ (a) VLANs y Direccionamiento

### 🔹 Asignación de VLANs

- **VLAN 10 (Administración)**
  - Dispositivos: PC1, PC4
  - Interfaces de administración en todos los switches
- **VLAN 20 (Usuarios)**
  - Dispositivos: PC2, PC3

### 🔹 Direccionamiento IP Propuesto

#### **VLAN 10 (Administración)**

- **PC1**: 192.168.10.1 /24
- **PC4**: 192.168.10.2 /24
- **Switches**:
  - **SW1**: 192.168.10.11
  - **SW2**: 192.168.10.12
  - **SW3**: 192.168.10.13
  - **SW4**: 192.168.10.14
  - **SW5**: 192.168.10.15

#### **VLAN 20 (Usuarios)**

- **PC2**: 192.168.20.1 /24
- **PC3**: 192.168.20.2 /24

_Nota:_ Todos los switches tienen ambas VLANs creadas (VLAN 10 y VLAN 20) para garantizar consistencia y permitir STP por VLAN (PVST).

🖥️ CONFIGURACIÓN POR SWITCH
🔵 SW1 SW4
enable
conf t
vlan 10
name ADMIN
interface fa0/10
switchport mode access
switchport access vlan 10
interface vlan 10
ip address 192.168.10.11 255.255.255.0 #cambia dependiendo del switch 4 -> ip address 192.168.10.14 255.255.255.0
no shutdown
exit
end
copy running-config startup-config

🔵 SW2 SW3
enable
conf t
vlan 20
name USERS
interface fa0/10
switchport mode access
switchport access vlan 20
interface vlan 20
ip address 192.168.10.11 255.255.255.0
no shutdown
exit
end
copy running-config startup-config

🔵 SW5
enable
conf t
interface fa0/10
switchport mode access
switchport access vlan 10
interface vlan 10
ip address 192.168.10.15 255.255.255.0
no shutdown

🧑‍💻 CONFIGURACIÓN DE PCs
🔹 PC1 (en SW4)
IP: 192.168.10.1
Máscara: 255.255.255.0
🔹 PC4 (en SW5)
IP: 192.168.10.2
Máscara: 255.255.255.0
🔹 PC2 (en SW1)
IP: 192.168.20.1
Máscara: 255.255.255.0
🔹 PC3 (en SW2)
IP: 192.168.20.2
Máscara: 255.255.255.0

## ✅ (b) Elección del Root Bridge

### 🔹 Switch Elegido

- **SW3** como Root Bridge

### 🔹 Criterio Técnico

- SW3 es central en la topología.
- Tiene más enlaces, lo que minimiza el costo total de caminos STP.
- Reduce puertos bloqueados y optimiza el flujo de tráfico.

### 🔹 Estrategia de Configuración

```bash
enable
configure terminal
spanning-tree vlan 10 priority 4096
spanning-tree vlan 20 priority 4096
exit
copy running-config startup-config
```

🔹 Efecto del Cambio
La prioridad se aplica inmediatamente en SW3.
Sin embargo, no se reflejará en la topología de la red hasta que se configuren los trunks, ya que STP necesita intercambio de BPDUs para elegir el Root Bridge global.

⚠️ Importante: El cambio tiene efecto inmediato a nivel local, pero en la red completa depende de que los switches puedan comunicarse a través de los trunks.

✅ (c) Configuración de Puertos entre Switches

Todos los enlaces entre switches se configuraron como trunks con la siguiente configuración:

enable
configure terminal
interface range fa0/1 - 8
switchport mode trunk
switchport trunk allowed vlan 10,20
exit
end
copy running-config startup-config

No se aplican trunks a los puertos conectados a PCs (fa0/10), que deben permanecer en modo access.

confirmar
show spanning-tree vlan 10
show spanning-tree vlan 20

✅ (d) Spanning Tree Resultante
🔹 Cantidad de Spanning Tree
2 STP: uno por cada VLAN (10 y 20).
Esto se debe a PVST+, que crea un árbol independiente por VLAN.
🔹 Ventajas
Balanceo de carga entre enlaces redundantes.
Mejor aprovechamiento de la topología.
Posibilidad de tener diferentes Root Bridges por VLAN.
🔹 Forma del Árbol
SW3 es el Root Bridge.
Todos los caminos convergen hacia SW3.
Los enlaces redundantes quedan en estado blocking según STP.
✅ (e) Recorrido de Trama: PC1 → SW2 (Administración)
PC1 (VLAN 10) está conectado a SW4.
El flujo de la trama será:
PC1 → SW4 → SW3 (Root Bridge) → SW2 → Interfaz de administración.

Este recorrido sigue el árbol STP para VLAN 10.

✅ (f) Eliminación del Root Bridge
Acción
Cambiar la prioridad de SW3 a un valor mayor o apagarlo:
spanning-tree vlan 10 priority 61440
Resultado
El nuevo Root Bridge será el switch con menor prioridad disponible. En caso de empate, el Root será determinado por la menor MAC.
STP recalcula la topología, ajusta los puertos bloqueados y forma un nuevo árbol.
✅ (g) Adición de un Nuevo Switch como Root

Antes de conectar el nuevo switch, se configura su prioridad a un valor mínimo:

spanning-tree vlan 10 priority 0
spanning-tree vlan 20 priority 0
Resultado
Este switch tendrá la menor prioridad y se convertirá en el Root Bridge para ambas VLANs.
Con Rapid-PVST, se convierte en Root Bridge casi inmediatamente; con STP clásico, puede tardar unos segundos hasta que la topología se ajuste.
🧠 Conclusión General
La red está segmentada por VLANs y correctamente direccionada.
Se implementó STP por VLAN (PVST).
La selección del Root Bridge en una ubicación central optimiza la topología.
Los trunks son fundamentales para que STP funcione correctamente.
STP se adapta dinámicamente ante cambios en el Root Bridge o la incorporación de nuevos switches.
