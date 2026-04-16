# Laboratorio 2: Spanning Tree Protocol (STP)

## a) Determinar el Root Bridge

### Objetivo
Identificar cuál switch actúa como Root Bridge y entender los criterios de selección del protocolo STP.

### Criterios de selección

El Root Bridge se elige en función del **Bridge ID**, que está compuesto por:
- **Prioridad** (valor por defecto: 32768)
- **Dirección MAC** (identificador único de cada switch)

El switch con el **Bridge ID más bajo** es elegido como Root Bridge. Si la prioridad es igual, gana el que tenga la **MAC más baja**.

### Comandos para verificar

Ejecuta los siguientes comandos en cada switch:

```
enable
show spanning-tree
```

### Forzar un switch como Root (Opcional)

Para garantizar que un switch específico sea el Root Bridge:

```
enable
configure terminal
spanning-tree vlan 1 priority 4096
end
```

Verifica nuevamente con `show spanning-tree`.

### Resultado

| Parámetro | Valor |
|-----------|-------|
| Root ID Priority | 32769 |
| Root ID Address (MAC) | 0001.6461.3A61 |
| Cost | 19 |
| Root Port | Fa0/3 |

**Conclusión:** El Root Bridge es el switch con MAC `0001.6461.3A61` y prioridad `32769`, seleccionado por poseer el Bridge ID más bajo.

---

## b) Encontrar bucles y roles de puertos

### Objetivo
Comprender cómo STP previene bucles de red bloqueando puertos específicos.

### Tabla de puertos

| Puerto | Rol | Estado | Costo | Descripción |
|--------|-----|--------|-------|-------------|
| Fa0/3 | Root Port | FWD (Forwarding) | 19 | Puerto activo hacia el Root Bridge |
| Fa0/1 | Alternate Port | BLK (Blocking) | 19 | Puerto de respaldo en espera |
| Fa0/2 | Alternate Port | BLK (Blocking) | 19 | Puerto de respaldo en espera |

### Interpretación

- **Fa0/3 (Root Port - FWD):** Es el puerto con el camino más corto hacia el Root Bridge. El tráfico fluye activamente por este puerto.

- **Fa0/1 y Fa0/2 (Alternate Ports - BLK):** Están bloqueados para evitar bucles de red. Se mantienen en espera de redundancia, listos para activarse si el Root Port falla.

### Prevención de bucles

Tenemos **3 enlaces diferentes** entre los switches:
- **1 enlace activo** (Fa0/3) → el tráfico fluye normalmente
- **2 enlaces bloqueados** (Fa0/1 y Fa0/2) → evitan tormentas de broadcast

**Resultado:** STP mantiene la redundancia en la red sin crear bucles que causarían tormentas de tráfico.

---

## c) Dibujar el Spanning Tree resultante

### Topología

```
           Root Bridge (SW1)
               |
            Fa0/3
        [Forwarding]
               |
            ↓ SW2 ↓
           /      \
      Fa0/1      Fa0/2
    [Blocked]    [Blocked]
```

### Análisis de la topología

- **Root Bridge:** Switch principal (MAC: 0001.6461.3A61, prioridad: 32769)
- **Fa0/3:** Root Port en estado Forwarding → único camino activo hacia el Root Bridge
- **Fa0/1 y Fa0/2:** Alternate Ports en estado Blocking → proporcionan redundancia sin crear bucles

### Conclusión

La topología de STP asegura que:
1. **Solo hay un camino activo** desde cada switch hacia el Root Bridge
2. **La redundancia se mantiene** con puertos en Blocking listos para activarse
3. **No hay bucles de red** que causen tormentas de broadcast
