# Lab 6: OSPF - Áreas, DR/BDR y Establecimiento de Vecindad

**Curso**: Redes de Computadores 2  
**Tema**: OSPF con múltiples áreas, selección de DR/BDR, análisis de adyacencias  
**Herramienta**: GNS3 + Wireshark  
**Duración estimada**: 6-8 horas

---

## 📋 Contenido del Laboratorio

1. [Topología del Lab 4 (Punto de Partida)](#topología-del-lab-4)
2. [Actividad 1a: Configuración IP y OSPF (Área no-0)](#actividad-1a-configuración-ip-y-ospf)
3. [Actividad 1b: Verificación de Tablas OSPF](#actividad-1b-verificación-de-tablas-ospf)
4. [Actividad 1c: Explicación de DR y BDR](#actividad-1c-dr-y-bdr)
5. [Actividad 1d: Router ID](#actividad-1d-router-id)
6. [Actividad 1e: Captura Wireshark y Establecimiento de Vecindad](#actividad-1e-captura-wireshark)

---

# Topología del Lab 4 - Visualización por Áreas

## Vista General: Equipos y Conexiones

```
LAN_A              LAN_B                                  LAN_C
  |                  |                                      |
  |                  |                                      |
Fa0/0              Fa0/0                                 Fa0/0
  |                  |                                      |
 R1                 R2         R3                          R4
  |                  |          |                           |
Fa0/1              Fa0/1      Fa0/0                       s0/0
  |                  |          |                           |
  +------------------+-----Switch1----+-----Serial--------+
      e0                e1             e2               
```

## Vista Detallada: Áreas OSPF (Área 1 sin Área 0)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                              ÁREA 1 (No-Backbone)                          │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────┐       │
│  │ Red: 192.168.78.128/26 (Switch1) - Broadcast Domain            │       │
│  │                                                                 │       │
│  │    Fa0/1 (192.168.78.129)    Fa0/1 (192.168.78.130)           │       │
│  │           │                          │                         │       │
│  │           R1  ────────Switch1────── R2                         │       │
│  │          /│\                        /│\                        │       │
│  │         / │ \                      / │ \                       │       │
│  │        /  │  \                    /  │  \                      │       │
│  │       /   │   \ ─────e2────────  /   │   \                     │       │
│  │      /    │                      \   │    \                    │       │
│  │     /     │                       \  │     \                   │       │
│  │    Fa0/0  │                    Fa0/0 │    Fa0/0               │       │
│  │   │       │                    │     │    │                   │       │
│  │ LAN_A     │                  LAN_B   │  R3                    │       │
│  │           │                          │  /│                    │       │
│  │      (Pasiva)                  (Pasiva) / │                    │       │
│  │                                        / s0/0                  │       │
│  │                                       /  │                     │       │
│  │                                      /   │                     │       │
│  │                        ┌────────────────┘                      │       │
│  │                        │                                       │       │
│  └────────────────────────┼──────────────────────────────────────┘       │
│                           │                                             │
│                    (Límite de Área 1)                                   │
│                      (Sin Área 0)                                       │
└─────────────────────────────────────────────────────────────────────────────┘

                                ❌ PROBLEMA
                          No hay enlace a Área 0
              → Los routers en Área 1 NO pueden
              comunicarse con routers en otras áreas
              → OSPF necesita BACKBONE (Área 0)
```

## Vista Alternativa: ¿Qué Pasaría con Área 0?

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│   ┌────────────────────────────────────────────────────────────────────┐   │
│   │              ÁREA 0 (BACKBONE - Obligatorio)                      │   │
│   │  ◯ R3 (Punto de conexión entre áreas)                             │   │
│   │   s0/0: 192.168.78.193 ───────┬────── s0/0: 192.168.78.194       │   │
│   │                                 │                                  │   │
│   │                              (Serial)                              │   │
│   │                                 │                                  │   │
│   │                            ┌────┘                                  │   │
│   │                            │                                       │   │
│   └────────────────────────────┼────────────────────────────────────────┘   │
│                                │                                            │
│   ┌────────────────────────────┼────────────────────────────────────────┐   │
│   │ ÁREA 1 (Secundaria)        │                                        │   │
│   │  R1 - Switch1 - R2         │                                        │   │
│   │  (Conectada a Área 0)      │                                        │   │
│   └────────────────────────────┼────────────────────────────────────────┘   │
│                                │                                            │
│   ┌────────────────────────────┼────────────────────────────────────────┐   │
│   │ ÁREA 2 (u otra)            │                                        │   │
│   │  R4 + LAN_C                │                                        │   │
│   │  (Podría existir)          │                                        │   │
│   └────────────────────────────┼────────────────────────────────────────┘   │
│                                │                                            │
│                           ✅ ÁREA 0 conecta                                │
│                        todas las áreas                                    │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Vista por Tipo de Enlace

```
ENLACES BROADCAST (Necesitan DR/BDR):

    ┌─────────────────────────────┐
    │  Switch1 (192.168.78.128)   │  ← Dominio Broadcast
    │  (Necesita DR y BDR)        │
    └────────┬────────┬───────────┘
             │        │
          Fa0/1     Fa0/1        Fa0/0
             │        │            │
             R1       R2           R3
          (DR)      (BDR)      (DROther)


ENLACES POINT-TO-POINT (NO necesitan DR/BDR):

    R3 ──────Serial──────┬────── R4
    s0/0: 192.168.78.193 │  s0/0: 192.168.78.194
                         │
                    ❌ Sin DR/BDR
                    ✅ Adyacencia FULL directa
```

## Mapa de Redes por Interfaz

```
╔════════════════════════════════════════════════════════════════════════════╗
║                    TOPOLOGÍA CON IDENTIFICACIÓN DE REDES                  ║
╚════════════════════════════════════════════════════════════════════════════╝

┌─────────────────────┐
│   LAN_A             │  Red: 192.168.78.0/26
│   Interfaz Pasiva   │  (sin OSPF en la LAN)
└──────────┬──────────┘
           │ Fa0/0: 192.168.78.1
           │
           R1 ─────────────── Fa0/1: 192.168.78.129 ────┐
           │                                             │
           │         ┌─────────────────────────────┐    │
           │         │   Switch1                   │    │
           │         │  Red: 192.168.78.128/26    │    │
           │         │  (BROADCAST - Área 1)      │    │
           │         │                             │    │
           │    Fa0/1: 192.168.78.129 ◯          │    │
           │         │                 R1         │    │
           │         │                 ◯ DR       │    │
           └─────────┤                             │    │
                     │  Fa0/1: 192.168.78.130 ◯  │    │
                     │           R2                │    │
                     │           ◯ BDR             │    │
                     │                             │    │
                     │  Fa0/0: 192.168.78.131 ◯  │    │
                     │         R3                 │    │
                     │         ◯ DROther          │    │
                     └──────┬──────────────────────┘    │
                            │                           │
                         s0/0: 192.168.78.193           │
                            │                           │
                            │ Serial                     │
                            │                           │
                         s0/0: 192.168.78.194          │
                            │                           │
              ┌─────────────┘                           │
              │                              Red: 192.168.78.192/29
              R4                         (POINT-to-POINT - Área 1)
              │
          Fa0/0: 192.168.78.201
              │
       ┌──────┴──────┐
       │             │
    LAN_C          (Pasiva)
   Red: 192.168.78.200/29
   (sin OSPF en la LAN)
```

## Estados de Adyacencia en Switch1

```
╔═══════════════════════════════════════════════════════════════════════╗
║         ADYACENCIAS EN BROADCAST DOMAIN (Switch1)                    ║
╚═══════════════════════════════════════════════════════════════════════╝

SIN DR/BDR (Ideal para 3 routers):
  R1 ↔ R2 ↔ R3  → 6 adyacencias posibles (n(n-1)/2 = 3×2/2 = 3)

CON DR/BDR (Lo que sucede realmente):
  
  Parámetros:
  • R1: IP = 192.168.78.129, Prioridad = 1
  • R2: IP = 192.168.78.130, Prioridad = 1
  • R3: IP = 192.168.78.131, Prioridad = 1

  Elección:
  ┌─────────────────┐
  │ Más alta prioridad → Empate (todos = 1)
  │ Más alto Router ID → R3 (192.168.78.131)
  │ DR = R3
  └─────────────────┘

  ┌─────────────────┐
  │ Segunda prioridad → Empate
  │ Segundo Router ID → R2 (192.168.78.130)
  │ BDR = R2
  └─────────────────┘

  Topología de adyacencias:
  
            DR (R3)
            ◯  ◀──  Recibe LSAs de todos
           ╱│╲
          ╱ │ ╲
         ╱  │  ╲
        ◯   │   ◯
       R1   │   R2
     (DRO) (BDR)
     (DRO)
      
  Adyacencias reales:
  • R1 ↔ R3 (DR)      ← FULL
  • R2 ↔ R3 (DR)      ← FULL
  • R1 ↔ R2           ← NO (no se conectan directamente)
  
  Total: 2 adyacencias en lugar de 3 ✅ (eficiencia)
```

## Plan de Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Máscara | Red |
|-------------|----------|--------------|--------|-----|
| R1 | Fa0/0 | 192.168.78.1 | 255.255.255.192 | LAN_A (/26) |
| R1 | Fa0/1 | 192.168.78.129 | 255.255.255.192 | Switch1 (/26) |
| R2 | Fa0/0 | 192.168.78.65 | 255.255.255.192 | LAN_B (/26) |
| R2 | Fa0/1 | 192.168.78.130 | 255.255.255.192 | Switch1 (/26) |
| R3 | Fa0/0 | 192.168.78.131 | 255.255.255.192 | Switch1 (/26) |
| R3 | s0/0 | 192.168.78.193 | 255.255.255.248 | R3-R4 (/29) |
| R4 | Fa0/0 | 192.168.78.201 | 255.255.255.248 | LAN_C (/29) |
| R4 | s0/0 | 192.168.78.194 | 255.255.255.248 | R3-R4 (/29) |

---

# ACTIVIDAD 1a: Configuración IP y OSPF (Área no-0)

## Paso 1: Montar la Topología en GNS3

1. Crear 3 routers Cisco (Fa = Fast Ethernet, s = Serial)
2. Crear 1 switch Layer 2
3. Conectar según el diagrama anterior
4. Crear 3 PCs (LAN_A, LAN_B, LAN_C)

## Paso 2: Configurar Direcciones IP en Routers

### Configurar R1

```ios
enable
configure terminal

hostname R1
no ip domain-lookup

interface fa0/0
 description LAN_A
 ip address 192.168.78.1 255.255.255.192
 no shutdown

interface fa0/1
 description ENLACE A SWITCH1
 ip address 192.168.78.129 255.255.255.192
 no shutdown

end
write memory
```

### Configurar R2

```ios
enable
configure terminal

hostname R2
no ip domain-lookup

interface fa0/0
 description LAN_B
 ip address 192.168.78.65 255.255.255.192
 no shutdown

interface fa0/1
 description ENLACE A SWITCH1
 ip address 192.168.78.130 255.255.255.192
 no shutdown

end
write memory
```

### Configurar R3

```ios
enable
configure terminal

hostname R3
no ip domain-lookup

interface fa0/0
 description ENLACE A SWITCH1
 ip address 192.168.78.131 255.255.255.192
 no shutdown

interface s0/0
 description ENLACE A R4 (SERIAL)
 ip address 192.168.78.193 255.255.255.248
 clock rate 64000
 no shutdown

end
write memory
```

### Configurar R4

```ios
enable
configure terminal

hostname R4
no ip domain-lookup

interface fa0/0
 description LAN_C
 ip address 192.168.78.201 255.255.255.248
 no shutdown

interface s0/0
 description ENLACE A R3 (SERIAL)
 ip address 192.168.78.194 255.255.255.248
 no shutdown

end
write memory
```

## Paso 3: Configurar OSPF en ÁREA 1 (no área 0)

### ⚠️ Importante: ¿Por qué Área 1 y no Área 0?

**Área 0 (Backbone)**:
- Es obligatoria en OSPF
- Conecta diferentes áreas
- Todos los routers deben conocer el área 0

**Área 1**:
- Área secundaria
- Útil para esta práctica educativa
- Todos nuestros routers estarán aquí

**Problema**: Si todos están en área 1 (no área 0), **la topología no tendrá conectividad porque no hay área 0 (backbone)**.

**Solución**: Usamos área 1 para R1, R2, R3 y entendemos que esto demostrará la **importancia del área 0**.

### Configurar OSPF en R1

```ios
configure terminal

router ospf 1
 network 192.168.78.0 0.0.0.63 area 1      ! LAN_A y enlace Switch1
 network 192.168.78.128 0.0.0.63 area 1    ! Switch1
 
end
write memory
```

### Configurar OSPF en R2

```ios
configure terminal

router ospf 1
 network 192.168.78.0 0.0.0.63 area 1      ! LAN_B
 network 192.168.78.128 0.0.0.63 area 1    ! Switch1
 
end
write memory
```

### Configurar OSPF en R3

```ios
configure terminal

router ospf 1
 network 192.168.78.128 0.0.0.63 area 1    ! Switch1
 network 192.168.78.192 0.0.0.7 area 1     ! R3-R4
 
end
write memory
```

### Configurar OSPF en R4

```ios
configure terminal

router ospf 1
 network 192.168.78.192 0.0.0.7 area 1     ! R3-R4
 network 192.168.78.200 0.0.0.7 area 1     ! LAN_C
 
end
write memory
```

## Paso 4: Verificación de Tablas de Enrutamiento

### Comando Básico

```ios
show ip route
```

## Paso 4: Verificación de Tablas de Enrutamiento

### Comando Básico

```ios
show ip route
```

### En R1

```
R1# show ip route

Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level 1, L2 - IS-IS level 2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, + - replicated route

Gateway of last resort is not set

     192.168.78.0/26 is subnetted, 2 subnets
C       192.168.78.0 is directly connected, FastEthernet0/0
C       192.168.78.128 is directly connected, FastEthernet0/1
```

### Análisis Visual - Comparación: CON Área 1 vs CON Área 0+1

```
╔═══════════════════════════════════════════════════════════════════════════╗
║                  TABLA DE RUTAS - COMPARACIÓN DE ESCENARIOS              ║
╚═══════════════════════════════════════════════════════════════════════════╝

ESCENARIO 1: TODOS EN ÁREA 1 (SIN ÁREA 0) ❌
──────────────────────────────────────────────

R1# show ip route

     192.168.78.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.78.0/26 is directly connected, FastEthernet0/0
C       192.168.78.128/26 is directly connected, FastEthernet0/1

┌──────────────────────────────────────────────┐
│ ❌ RESULTADO ESPERADO:                       │
├──────────────────────────────────────────────┤
│ • SOLO rutas Connected (C)                   │
│ • SIN rutas OSPF (sin "O")                   │
│ • SIN rutas inter-área (sin "IA")           │
│                                              │
│ CAUSA: No hay backbone (Área 0)             │
│        → Área 1 aislada                      │
│        → No puede conectarse a otras áreas  │
└──────────────────────────────────────────────┘


ESCENARIO 2: ÁREA 1 + ÁREA 0 (BACKBONE) ✅
──────────────────────────────────────────────

R1# show ip route

     192.168.78.0/24 is variably subnetted, 4 subnets, 3 masks
C       192.168.78.0/26 is directly connected, FastEthernet0/0
C       192.168.78.128/26 is directly connected, FastEthernet0/1
O       192.168.78.192/29 [110/2] via 192.168.78.129, 00:05:32, FastEthernet0/1
O       192.168.78.200/29 [110/3] via 192.168.78.129, 00:05:32, FastEthernet0/1

┌──────────────────────────────────────────────┐
│ ✅ RESULTADO ESPERADO:                       │
├──────────────────────────────────────────────┤
│ • Rutas Connected (C) = directas            │
│ • Rutas OSPF (O) = hacia redes remotas      │
│ • [110/X] = Distancia OSPF/Costo            │
│                                              │
│ CAUSA: Área 0 conecta Área 1 a 2            │
│        → Información de rutas propagada     │
│        → R1 conoce redes de Área 2          │
└──────────────────────────────────────────────┘
```

### Análisis Esperado

**CON ÁREA 1 SIN ÁREA 0**:
- ❌ No verás rutas OSPF (sin "O")
- ❌ Solo verás redes conectadas (con "C")
- ✅ Esto es correcto y demuestra por qué se necesita área 0
- ✅ Muestra el propósito educativo del laboratorio

**¿Por qué?** En OSPF, todas las áreas deben estar conectadas al área 0 (backbone). Como todos nuestros routers están en área 1, no hay backbone que los conecte.

---

## Paso 5: Habilitar Debugging de OSPF

Para ver por qué no hay conectividad:

```ios
enable
configure terminal

debug ip ospf events
debug ip ospf adj

end
```

### Salida Esperada (Console) - Sin Área 0

```
*Mar  1 00:05:42.134: OSPF: Interface FastEthernet0/1 going Up
*Mar  1 00:05:42.234: OSPF: Build router LSA for area 1, router ID 192.168.78.129
*Mar  1 00:05:42.334: OSPF: Aging LSA in Database...
*Mar  1 00:05:47.134: OSPF: DR/BDR election on FastEthernet0/1
*Mar  1 00:05:47.234: OSPF: Area 1 elected DR 192.168.78.131 on FastEthernet0/1
*Mar  1 00:05:47.334: OSPF: Area 1 elected BDR 192.168.78.130 on FastEthernet0/1

┌────────────────────────────────────────────────────────┐
│ ✓ Se eligen DR/BDR normalmente                        │
│ ✓ Se intercambian adyacencias dentro de Área 1        │
│ ✓ Se generan LSAs de routers y redes                  │
│ ✗ No hay información inter-área                       │
│ ✗ No se propagan rutas a otras áreas                  │
└────────────────────────────────────────────────────────┘
```

### Desactivar Debugging

```ios
undebug all
```

---

# Diagramas Visuales: Impacto de Áreas

[Los diagramas visuales se encuentran en la sección anterior]

---

# ACTIVIDAD 1b: Verificación de Tablas OSPF

## Comandos de Verificación

### 1. Tabla de Vecinos OSPF

```ios
show ip ospf neighbor
```

**Esperado** (si OSPF estuviera funcionando):

```
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.78.130  0     FULL/  -        00:00:39    192.168.78.130  FastEthernet0/1
192.168.78.131  0     FULL/  -        00:00:39    192.168.78.131  FastEthernet0/1
```

**Estados de adyacencia**:
- DOWN: Sin comunicación
- INIT: Intercambio inicial
- 2-WAY: Bidireccional establecido
- EXSTART: Preparación para intercambio DB
- EXCHANGE: Intercambiando descriptores
- LOADING: Pidiendo LSA
- FULL: Adyacencia completa ✅

### 2. Tabla de Topología OSPF

```ios
show ip ospf database
```

**Salida esperada** (router LSA):

```
OSPF Router with ID (192.168.78.1) (Process ID 1)
                
                Router Link States (Area 1)

Link ID         ADV Router      Age  Seq#       Checksum Link count
192.168.78.1    192.168.78.1    34   0x80000001 0x00c58f 2
192.168.78.130  192.168.78.130  34   0x80000001 0x00a5bc 2
192.168.78.131  192.168.78.131  34   0x80000001 0x008cd5 2
192.168.78.194  192.168.78.194  34   0x80000001 0x009e42 2
```

**Explicación de campos**:

| Campo | Significado | Ejemplo |
|-------|------------|---------|
| Link ID | ID del router que originó el LSA | 192.168.78.1 |
| ADV Router | Router que anuncia esta información | 192.168.78.1 |
| Age | Antigüedad del LSA en segundos | 34 seg |
| Seq# | Número de secuencia (para validar actualización) | 0x80000001 |
| Checksum | Suma de verificación para integridad | 0x00c58f |
| Link count | Número de enlaces de este router | 2 |

### 3. Tabla de Red OSPF

```ios
show ip ospf database network
```

**Salida esperada**:

```
                Net Link States (Area 1)

Link ID         ADV Router      Age  Seq#       Checksum
192.168.78.128  192.168.78.1    45   0x80000001 0x0056a7
```

**Explicación**:
- **Link ID**: Dirección de la red (ej: 192.168.78.128 es la red Switch1)
- **ADV Router**: El DR que anuncia esta red

### 4. Información de Interfaces OSPF

```ios
show ip ospf interface brief
```

**Salida esperada**:

```
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Fa0/0        1     1               192.168.78.1/26    1     DOWN  0/0
Fa0/1        1     1               192.168.78.129/26  1     DR    2/2
S0/0         1     1               192.168.78.193/29  64    DOWN  0/0
```

**Explicación de columnas**:

| Columna | Significado |
|---------|------------|
| Interface | Nombre de la interfaz |
| PID | Process ID de OSPF |
| Area | Área OSPF |
| IP Address/Mask | IP y máscara asignada |
| Cost | Costo OSPF de la interfaz |
| State | Estado de la interfaz (DR, BDR, DROther, DOWN) |
| Nbrs | Neighbors / Full neighbors |
| F/C | Full/Change |

### 5. Detalle de una Interfaz

```ios
show ip ospf interface fa0/1
```

**Salida detallada esperada**:

```
FastEthernet0/1 is up, line protocol is up
  Internet Address 192.168.78.129/26, Area 1, Attached via Network Statement
  Process ID 1, Router ID 192.168.78.1, Network Type BROADCAST, Cost: 1
  Transmit Delay is 1 sec, State DR, Priority 1
  Designated Router (ID) 192.168.78.1, Interface address 192.168.78.129
  Backup Designated Router (ID) 192.168.78.130, Interface address 192.168.78.130
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    Hello due in 3 seconds
  Supports Link-local Signaling (LLS)
  Adjacency Changes: 0 changes
  Index 2/2, Flood queue length 0
```

### 6. Resumen del Proceso OSPF

```ios
show ip ospf
```

**Salida esperada**:

```
Routing Process "ospf 1" with ID 192.168.78.1
 Start time: 00:00:05.456, Time elapsed: 00:04:32.123
 Supports only single TOS(TOS 0) routes
 Supports opaque LSA
 Supports Link LSA (not enabled)
 It is an area border router
 Router is not originating router-LSAs with maximum metric
 Refresh timer interval 1800 secs
 Due in 1215 secs
 SPF schedule delay 5 secs, Hold time between two SPFs 10 secs
 Minimum LSA interval 0 msecs. Minimum LSA arrival 100 msecs
 Number of external LSA 0. Checksum Sum 0x000000
 Number of opaque AS LSA 0. Checksum Sum 0x000000
 Number of areas in this router is 1. 1 normal 0 stub
     Area 1
         Number of interfaces in this area is 3
         Area has no authentication
         SPF algorithm last executed 00:04:31.999, elapsed time 3.543 sec
```

---

## Tabla Resumen: Interpretación de Entradas

### Router LSA

| Campo | Significa |
|-------|----------|
| Link ID = 192.168.78.1 | Router con ID 192.168.78.1 |
| ADV Router = 192.168.78.1 | Él mismo anuncia su información |
| Age = 34 | LSA tiene 34 segundos de antigüedad |
| Seq# = 0x80000001 | Primera versión de este LSA |
| Link count = 2 | Este router tiene 2 enlaces activos |

### Network LSA

| Campo | Significa |
|-------|----------|
| Link ID = 192.168.78.128 | Red 192.168.78.128 (Switch1) |
| ADV Router = 192.168.78.1 | El DR (192.168.78.1) la anuncia |
| Age = 45 | 45 segundos de antigüedad |

### Conclusión

**Con Área 1 sin Área 0**:
- ✅ Verás adyacencias entre routers en la misma área
- ✅ Verás LSAs de routers y redes
- ❌ NO verás rutas OSPF hacia redes remotas en otra área
- ❌ No hay comunicación inter-área

---

# ACTIVIDAD 1c: DR y BDR (Designated Router y Backup Designated Router)

## ¿Qué son DR y BDR?

En redes Ethernet broadcast (como un switch), múltiples routers están conectados al mismo medio. OSPF necesita:

1. **Un router que lidere**: DR (Designated Router)
2. **Un respaldo**: BDR (Backup Designated Router)
3. **El resto**: DROthers

### Por qué necesitamos DR/BDR?

| Problema | Solución |
|----------|----------|
| Si n routers están en una LAN, habría n(n-1)/2 adyacencias | Usar DR: cada router solo se conecta al DR |
| Ejemplo: 4 routers = 6 adyacencias | Con DR: 4 conexiones |
| Genera mucho tráfico LSA innecesario | LSAs inundados solo al DR, que los reenvía |

### Tipos de Redes OSPF

| Tipo | Características | Ej |
|------|---|---|
| **Broadcast** | Múltiples routers, elect DR/BDR | Ethernet, FastEthernet |
| **Point-to-Point** | 2 routers, sin DR/BDR | Serial, PPP |
| **NBMA** | No Broadcast Multi-Access | Frame Relay |

---

## Selección de DR/BDR

### Algoritmo de Selección

```
1. Prioridad OSPF más alta (default = 1)
2. Si empate: Router ID más alto (IP más alta)
3. Si empate (raro): Primera interfaz activa
```

**Nota**: El DR se **elige antes** del BDR:
- **DR**: Máxima prioridad/Router ID
- **BDR**: Segunda máxima prioridad/Router ID
- **Resto**: DROthers

### Determinación en Nuestra Topología

### Red Switch1 (192.168.78.128/26) - BROADCAST Domain

**Criterios de Elección**:

```
┌─────────────────────────────────────────────────────────────┐
│  STEP 1: Revisar Prioridad OSPF                            │
├─────────────────────────────────────────────────────────────┤
│  R1: Prioridad = 1 (default)  ← Todas iguales             │
│  R2: Prioridad = 1 (default)  ← EMPATE                    │
│  R3: Prioridad = 1 (default)  ↓                           │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  STEP 2: Revisar Router ID (si prioridad igual)            │
├─────────────────────────────────────────────────────────────┤
│  R1: Router ID = 192.168.78.129  (Fa0/1 es la IP más alta)│
│  R2: Router ID = 192.168.78.130  (Fa0/1 es la IP más alta)│
│  R3: Router ID = 192.168.78.193  (S0/0 es la IP más alta) │
│                                                             │
│  COMPARACIÓN (mayor Router ID gana):                       │
│  192.168.78.193 (R3) > 192.168.78.130 (R2) > 192.168.78.129 (R1)
│                                                             │
│  🏆 GANADOR: R3 con 192.168.78.193 → DR                   │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  STEP 3: Segunda posición (BDR) - Mismo proceso            │
├─────────────────────────────────────────────────────────────┤
│  Después de elegir DR (R3):                                │
│  R1: Router ID = 192.168.78.129                           │
│  R2: Router ID = 192.168.78.130                           │
│                                                             │
│  COMPARACIÓN:                                              │
│  192.168.78.130 (R2) > 192.168.78.129 (R1)                │
│                                                             │
│  🥈 GANADOR: R2 con 192.168.78.130 → BDR                  │
│  🥉 TERCERO: R1 → DROther                                 │
└─────────────────────────────────────────────────────────────┘
```

**Resultado Visual**:

```
        ┌─────────────────────────────────────┐
        │       Switch1 OSPF Topology         │
        │  Red: 192.168.78.128/26             │
        └─────────────────────────────────────┘
                        │
         ┌──────────────┼──────────────┐
         │              │              │
       ◯ R1           ◯ R3            ◯ R2
   DROther      ╔════ DR ════╗    ╔══ BDR ══╗
   192.168.78.1 ║(193)      ║    ║(130)   ║
                ║MASTER     ║    ║Respaldo║
                ╚═══════════╝    ╚════════╝

Adyacencias OSPF en Switch1:
  
  DROther (R1) ────✓──── DR (R3)      ← FULL
  DROther (R1)     X    DROther (R2)  ← NO (no adyacente)
  BDR (R2) ────✓──── DR (R3)          ← FULL

Total de adyacencias: 2 (eficiente)
Sin DR/BDR serían: 3(3-1)/2 = 3 (innecesario)
```

#### Enlace Serial R3-R4 (192.168.78.192/29)

Type: Point-to-Point
- **NO USAR DR/BDR** (solo 2 routers)
- Ambos forman adyacencia FULL directamente

#### LAN_A, LAN_B, LAN_C

Interfaces pasivas (solo conectan routers a PCs):
- **NO HAY DR/BDR** (interfaces pasivas no forman adyacencias)

---

## Verificación de DR/BDR

### Comando 1: Ver estado de la interfaz

```ios
show ip ospf interface fa0/1
```

**Salida relevante**:

```
FastEthernet0/1 is up, line protocol is up
  ...
  Transmit Delay is 1 sec, State DR, Priority 1         ← Estado y prioridad
  Designated Router (ID) 192.168.78.131, Interface address 192.168.78.131
  Backup Designated Router (ID) 192.168.78.130, Interface address 192.168.78.130
  ...
```

**En R1**:
```
State DR, Priority 1
Designated Router (ID) 192.168.78.131     ← R3 es DR
Backup Designated Router (ID) 192.168.78.130  ← R2 es BDR
```

**En R2**:
```
State BDR, Priority 1
Designated Router (ID) 192.168.78.131     ← R3 es DR
Backup Designated Router (ID) 192.168.78.130  ← R2 es BDR (yo)
```

**En R3**:
```
State DR, Priority 1
Designated Router (ID) 192.168.78.131     ← R3 es DR (yo)
Backup Designated Router (ID) 192.168.78.130  ← R2 es BDR
```

### Comando 2: Ver vecinos

```ios
show ip ospf neighbor
```

**En R1 (DROther)**:

```
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.78.131  1     FULL/DR         00:00:32    192.168.78.131  Fa0/1
192.168.78.130  1     FULL/BDR        00:00:32    192.168.78.130  Fa0/1
```

**Interpretación**:
- R1 tiene adyacencias con R3 (DR) y R2 (BDR)
- Estado FULL/DR significa: adyacencia completa, vecino es DR

---

## Cambiar DR/BDR Manualmente

Para cambiar el DR, aumenta la prioridad en una interfaz:

```ios
interface fa0/1
 ip ospf priority 10   ! Aumentar a 10 (máxima es 255)
```

**Ejemplo**: Hacer que R1 sea el DR

```ios
! En R1
interface fa0/1
 ip ospf priority 10

! En R2
interface fa0/1
 ip ospf priority 5

! En R3
interface fa0/1
 ip ospf priority 1  ! DROther
```

**Resultado**: R1 (prioridad 10) → DR, R2 (prioridad 5) → BDR, R3 → DROther

---

# ACTIVIDAD 1d: Router ID

## ¿Qué es el Router ID?

El **Router ID** es un identificador único (formato IPv4) de cada router OSPF. Se usa para:
- Identificar el router en LSAs
- Romper empates en elecciones DR/BDR
- Elegir BDR si hay empate con el DR

### Criterios de Asignación

OSPF elige automáticamente el Router ID en este orden:

| Prioridad | Criterio | Búsqueda |
|-----------|----------|---------|
| 1 | **Configurado explícitamente** | Comando `router-id X.X.X.X` |
| 2 | **IP más alta de interfaces Loopback** | Interface loopback (si existe) |
| 3 | **IP más alta de interfaces activas** | Cualquier interfaz Fa, Ethernet, Serial, etc. |

---

## Router IDs en Nuestra Topología (Sin Configuración Explícita)

### Análisis

**R1**:
- Interfaces activas: Fa0/0 (192.168.78.1), Fa0/1 (192.168.78.129)
- Más alta: 192.168.78.129
- **Router ID = 192.168.78.129** ❌ ¡NO! La Fa0/0 es 192.168.78.1, que es menor
- **Router ID = 192.168.78.1** ✅ Es la más alta en la LAN

Wait, necesito verificar esto mejor. Las interfaces son:
- Fa0/0: 192.168.78.1 (conecta a LAN_A)
- Fa0/1: 192.168.78.129 (conecta a Switch1)

La más alta es 192.168.78.129, pero OSPF elige la más alta considerando solo las interfaces **activas y habilitadas en OSPF**.

**Recalculando**:
- Fa0/0: 192.168.78.1 - habilitada en OSPF (está en `network 192.168.78.0`)
- Fa0/1: 192.168.78.129 - habilitada en OSPF (está en `network 192.168.78.128`)
- Más alta: 192.168.78.129
- **Router ID (R1) = 192.168.78.129** ✅

**R2**:
- Fa0/0: 192.168.78.65 (LAN_B)
- Fa0/1: 192.168.78.130 (Switch1)
- Más alta: 192.168.78.130
- **Router ID (R2) = 192.168.78.130** ✅

**R3**:
- Fa0/0: 192.168.78.131 (Switch1)
- S0/0: 192.168.78.193 (Enlace R3-R4)
- Más alta: 192.168.78.193
- **Router ID (R3) = 192.168.78.193** ✅

**R4**:
- Fa0/0: 192.168.78.201 (LAN_C)
- S0/0: 192.168.78.194 (Enlace R3-R4)
- Más alta: 192.168.78.201
- **Router ID (R4) = 192.168.78.201** ✅

---

## Verificación de Router ID

### Comando 1: Consultar directamente

```ios
show ip ospf
```

**Salida**:

```
Routing Process "ospf 1" with ID 192.168.78.129
  ...
```

### Comando 2: Ver en la base de datos

```ios
show ip ospf database router
```

**Salida** (en R1):

```
OSPF Router with ID (192.168.78.129) (Process ID 1)

                Router Link States (Area 1)

Link ID         ADV Router      Age  Seq#       Checksum Link count
192.168.78.129  192.168.78.129  12   0x80000001 0x00a3c2 2   ← R1
192.168.78.130  192.168.78.130  14   0x80000001 0x00b5f3 2   ← R2
192.168.78.193  192.168.78.193  15   0x80000001 0x00ccf1 2   ← R3
192.168.78.201  192.168.78.201  16   0x80000001 0x00d8a4 2   ← R4
```

---

## Asignar Router ID Manualmente

Para controlar el Router ID explícitamente:

```ios
configure terminal

router ospf 1
 router-id 192.168.78.1

end
write memory
```

### Nota Importante

Si cambias el router-id, debes reiniciar el proceso OSPF:

```ios
clear ip ospf process
  Proceed with reload of ospf process? [no]: yes
```

---

## Tabla Resumen: Router IDs

| Router | Criterio | Router ID |
|--------|----------|-----------|
| R1 | IP más alta (Fa0/1) | 192.168.78.129 |
| R2 | IP más alta (Fa0/1) | 192.168.78.130 |
| R3 | IP más alta (S0/0) | 192.168.78.193 |
| R4 | IP más alta (Fa0/0) | 192.168.78.201 |

---

# ACTIVIDAD 1e: Captura Wireshark y Establecimiento de Vecindad

## ¿Cómo se Establece una Vecindad OSPF?

## ¿Cómo se Establece una Vecindad OSPF?

### Fases del Establecimiento de Vecindad

```
NORMAL ──────────────► INTERFAZ CAE ──────────────► SE REACTIVA

Router A              (Wait 40 seg)                Router A
(Estado FULL)         (Dead Timer)                 (Estado DOWN)
   │                      │                           │
   │◄───── HELLO ────────►│                          │ Escucha
   │     periódico         │                          │
   │◄─────────────────────►│                          │
   │                       │                          │
   
    ◀───────────────────────────────────────────────────▶
    
Cuando se reactiva:

Router A                          Router B
   │                                 │
   │--- HELLO (224.0.0.5:50) ------->│  
   │    (Hola, soy R1)               │
   │                                 │
   │                          ✓ Estado INIT
   │                          ✓ Veo a R1
   │                                 │
   │<----- HELLO ──────────────────|
   │    (Hola, soy R2,                |
   │     veo a R1)                    │
   │                                 │
   │          ✓ Estado 2-WAY         |
   │       Bidireccional OK           |
   │                                 │
   │--- DBD (Seq=1234, I=1, M=1) --->│
   │    (Master: R1)                  │
   │                                 │
   │                         ✓ Estado EXSTART
   │                                 │
   │<----- DBD (Seq=1235, S=0) ------|
   │    (Slave: R2, responde)         │
   │                                 │
   │          ✓ Estado EXCHANGE      │
   │     Intercambiando DB            │
   │                                 │
   │--- LSR ----------------------->|
   │  (¿Dame la LSA X?)              │
   │                                 │
   │<----- LSU ──────────────────|
   │  (Aquí está la LSA X)            │
   │                                 │
   │--- LSAck -------------------->|
   │  (Confirmo recepción)            │
   │                                 │
   │          ✓ Estado LOADING       │
   │      Intercambiando LSAs         │
   │                                 │
   │                         ✓ Estado FULL
   │◄─────── Vecindad ESTABLECIDA ───►|
   │     (sincronizado)               │
   │                                 │
```

### Estados de Adyacencia - Máquina de Estados

```
                    ┌─────────────────────────────────────┐
                    │         DOWN (Inicial)              │
                    │  ✓ Escuchando HELLOs               │
                    └────────────────┬────────────────────┘
                                     │
                    ◀────────────────┘
                    │ HELLO recibido
                    │
                    ▼
        ┌─────────────────────────────────────┐
        │         INIT                        │
        │  ✓ HELLO recibido de vecino        │
        │  ✓ Aún no ve nuestro HELLO        │
        │  ✓ Esperando HELLO de vuelta      │
        └────────────┬────────────────────────┘
                     │
                     │ Recibimos HELLO que nos incluye
                     │
                     ▼
        ┌─────────────────────────────────────┐
        │         2-WAY                       │
        │  ✓ Comunicación BIDIRECCIONAL      │
        │  ✓ Ambos se ven en HELLOs         │
        │  ✓ DR/BDR confirmado             │
        └────────────┬────────────────────────┘
                     │
                     │ Para Broadcast: listo
                     │ Para Point-to-Point: continúa
                     │
              ┌──────┴────────┐
              │               │
              ▼               ▼
    (Broadcast)      (Point-to-Point)
    EXSTART         En algunos casos
                    puede ir directo
                    a FULL
              
              ▼
        ┌─────────────────────────────────────┐
        │         EXSTART                     │
        │  ✓ Inicio intercambio base datos   │
        │  ✓ Elegir Master/Slave            │
        │  ✓ Negociar sequence number       │
        └────────────┬────────────────────────┘
                     │
                     │ DBD intercambiados
                     │
                     ▼
        ┌─────────────────────────────────────┐
        │         EXCHANGE                    │
        │  ✓ Intercambiando DBDs             │
        │  ✓ Enviando listado de LSAs       │
        │  ✓ Master entra lista              │
        └────────────┬────────────────────────┘
                     │
                     │ Identific. de LSAs nuevas
                     │
                     ▼
        ┌─────────────────────────────────────┐
        │         LOADING                     │
        │  ✓ Pidiendo LSAs específicas (LSR)│
        │  ✓ Recibiendo LSAs (LSU)          │
        │  ✓ Confirmando recepción (LSAck)  │
        └────────────┬────────────────────────┘
                     │
                     │ Todas las LSAs sincronizadas
                     │ BD completa recibida
                     │
                     ▼
        ┌─────────────────────────────────────┐
        │         FULL ✅                     │
        │  ✓ Adyacencia completamente       │
        │    establecida                     │
        │  ✓ Base de datos sincronizada     │
        │  ✓ Vecino en topología OSPF       │
        └─────────────────────────────────────┘
```

### Tipos de Paquetes OSPF (Con Banderas)

```
╔═══════════════════════════════════════════════════════════════╗
║              PAQUETES OSPF EN LA SECUENCIA                   ║
╚═══════════════════════════════════════════════════════════════╝

1. HELLO
   ┌─────────────────────────────────────┐
   │ Destino: 224.0.0.5 (multicast)     │
   │ Puerto: UDP 50                      │
   │ Contenido:                          │
   │  • Router ID del origen             │
   │  • Área OSPF                        │
   │  • Intervalo Hello: 10 seg          │
   │  • Intervalo Muerto: 40 seg         │
   │  • DR detectado                     │
   │  • BDR detectado                    │
   │  • Lista de vecinos vistos ✓        │
   └─────────────────────────────────────┘

2. Database Description (DBD)
   ┌─────────────────────────────────────┐
   │ Banderas:                           │
   │  • I (Init) = 1: Primer DBD         │
   │  • M (More) = 1: Hay más DBDs      │
   │  • MS (Master/Slave) = 1: Master   │
   │ Contenido:                          │
   │  • MTU de interfaz                  │
   │  • DD Sequence Number               │
   │  • Resumen de LSAs conocidas        │
   └─────────────────────────────────────┘

3. Link State Request (LSR)
   ┌─────────────────────────────────────┐
   │ Contenido:                          │
   │  • LS Type (Router, Network, etc)  │
   │  • LS ID                            │
   │  • Advertising Router               │
   │ Significado:                        │
   │  "¿Puedes enviarme esta LSA?"      │
   └─────────────────────────────────────┘

4. Link State Update (LSU)
   ┌─────────────────────────────────────┐
   │ Contenido:                          │
   │  • LSA 1 (completa)                │
   │  • LSA 2 (completa)                │
   │  • ... (más LSAs)                   │
   │ Significado:                        │
   │  "Aquí están las LSAs solicitadas" │
   └─────────────────────────────────────┘

5. Link State Acknowledgment (LSAck)
   ┌─────────────────────────────────────┐
   │ Contenido:                          │
   │  • Encabezados de LSAs recibidas   │
   │ Significado:                        │
   │  "Confirmo que recibí estas LSAs"  │
   └─────────────────────────────────────┘
```

---

## Configuración para Captura

### Paso 1: Habilitar Debugging de OSPF

En R1, activa debug para ver el proceso en consola:

```ios
configure terminal

debug ip ospf adj      ! Debugging de adyacencias
debug ip ospf events   ! Debugging de eventos OSPF
debug ip ospf hello    ! Debugging de paquetes HELLO

end
```

### Paso 2: Abrir Wireshark en GNS3

1. En GNS3, haz clic derecho en el enlace Fa0/1 entre R1 y Switch1
2. Selecciona "Start capture"
3. Wireshark se abrirá capturando el tráfico en esa interfaz

### Paso 3: Apagar y Encender la Interfaz

En R1:

```ios
configure terminal

interface fa0/1
 shutdown            ! Apagar la interfaz (simular fallo)

end
```

**Esperar 30 segundos** (tiempo muerto de OSPF)

```ios
configure terminal

interface fa0/1
 no shutdown         ! Encender la interfaz (recuperación)

end
```

---

## Análisis de Paquetes Capturados

### Paquetes Esperados en Wireshark

#### 1. OSPF HELLO

```
Frame: OSPF Hello Packet
Destination: 224.0.0.5 (dirección multicast OSPF)
Source: 192.168.78.129 (R1)

OSPF Packet Header
  Version: 2
  Type: 1 (Hello)
  Packet Length: 44 bytes
  Router ID: 192.168.78.129
  Area ID: 1
  Checksum: 0x....

OSPF Hello Payload
  Network Mask: 255.255.255.192
  Hello Interval: 10 seconds
  Dead Interval: 40 seconds
  Designated Router: 192.168.78.131
  Backup DR: 192.168.78.130
  Neighbor List:
    - 192.168.78.130 (R2)
```

**Interpretación**:
- R1 anuncia que ve a R2 y R3 como vecinos
- DR es 192.168.78.131 (R3)
- BDR es 192.168.78.130 (R2)
- Intervalo entre HELLOs: 10 seg
- Tiempo muerto: 40 seg (sin HELLOs → vecino muerto)

#### 2. OSPF Database Description (DBD)

```
Frame: OSPF DBD Packet
OSPF Packet Header
  Type: 2 (Database Description)
  
OSPF DBD Payload
  Interface MTU: 1500 bytes
  I bit: 1 (Init)
  M bit: 1 (More)
  MS bit: 1 (Master)
  DD Sequence: 1234
  
  LSA Header Summary
    LS Type: 1 (Router LSA)
    LS ID: 192.168.78.129
    Advertising Router: 192.168.78.129
```

**Interpretación**:
- **I bit = 1**: Primer DBD de la secuencia
- **MS bit = 1**: Este router es el MASTER
- **M bit = 1**: Hay más DBDs después
- El DBD contiene una lista de LSAs conocidas

#### 3. OSPF Link State Request (LSR)

```
Frame: OSPF LSR Packet
OSPF Packet Header
  Type: 3 (Link State Request)
  
OSPF LSR Payload
  LS Type: 1 (Router LSA)
  LS ID: 192.168.78.130
  Advertising Router: 192.168.78.130
```

**Interpretación**:
- Un router pide que envíen el Router LSA del 192.168.78.130
- Es la respuesta a lo visto en el DBD

#### 4. OSPF Link State Update (LSU)

```
Frame: OSPF LSU Packet
OSPF Packet Header
  Type: 4 (Link State Update)
  
OSPF LSU Payload
  Number of LSAs: 2
  
  LSA 1: Router LSA
    LS Age: 1
    LS Type: 1 (Router)
    LS ID: 192.168.78.130
    Advertising Router: 192.168.78.130
    LS Checksum: 0x...
    Length: 48
    Flags: None
    Number of Links: 2
      Link 1: 192.168.78.128 (Network)
      Link 2: ...
```

**Interpretación**:
- Envío de los LSAs solicitados
- Incluye información de qué redes/routers conoce

#### 5. OSPF Link State Acknowledgment (LSAck)

```
Frame: OSPF LSAck Packet
OSPF Packet Header
  Type: 5 (Link State Ack)
  
OSPF LSAck Payload
  LSA Header Summary
    LS Type: 1
    LS ID: 192.168.78.130
    Advertising Router: 192.168.78.130
```

**Interpretación**:
- Confirmación de que se recibieron los LSAs
- Cierra el intercambio LOADING

---

## Secuencia Temporal Completa Capturada

## Secuencia Temporal Completa Capturada

### Con la Interfaz Cayendo y Levantando

```
╔════════════════════════════════════════════════════════════════════════════╗
║                   TIMELINE DE ESTABLECIMIENTO DE VECINDAD                 ║
╚════════════════════════════════════════════════════════════════════════════╝

FASE 1: FUNCIONAMIENTO NORMAL (Antes del shutdown)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  T=0.000 seg
    │
    ├─► HELLO: R1 → 224.0.0.5
    │   "Hola, soy R1, veo a R2 y R3"
    │
    ├─► HELLO: R2 → 224.0.0.5
    │   "Hola, soy R2, veo a R1 y R3"
    │
    ├─► HELLO: R3 → 224.0.0.5
    │   "Hola, soy R3, veo a R1 y R2"
    │
  T=10.000 seg  (Intervalo Hello)
    │
    ├─► HELLO: R1 → 224.0.0.5  (periódico)
    │
    ├─► HELLO: R2 → 224.0.0.5  (periódico)
    │
    ├─► HELLO: R3 → 224.0.0.5  (periódico)
    │
  T=20.000 seg
    │ ✓ Estado: FULL (adyacencias establecidas)
    │ ✓ Vecinos: R1↔R2, R1↔R3, R2↔R3 conocidas
    │

FASE 2: INTERFAZ SHUTDOWN EN R1
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  T=30.000 seg
    │
    │ ◄─────────────────────────────────────────────
    │          Comando: shutdown interfaz Fa0/1
    │ ◄─────────────────────────────────────────────
    │
    ├─► ⚠️  R1: Interfaz Fa0/1 cae (DOWN)
    │
    ├─► ⚠️  R1: Elimina adyacencias en esa interfaz
    │   - R1 ↔ R2 (muere)
    │   - R1 ↔ R3 (muere)
    │
    ├─► ⚠️  Mensaje debug:
    │   "OSPF: Nbr 192.168.78.130 going DOWN"
    │   "OSPF: Removing peer 192.168.78.130"
    │
  T=35.000 seg  (R2 y R3 esperan)
    │
    ├─► ⚠️  R2: Sin HELLO de R1
    │   Estado de R1 en R2: INIT → DOWN
    │
    ├─► ⚠️  R3: Sin HELLO de R1
    │   Estado de R1 en R3: INIT → DOWN
    │
  T=40.000 seg  (Dead timer de R2: 40 seg)
    │
    ├─► ⚠️  R2: Timer muere para R1
    │   Estado de R1: FULL → DOWN
    │   Mensaje: "Dead Time expired for neighbor 192.168.78.129"
    │
  T=50.000 seg  (Dead timer de R3: 40 seg)
    │
    ├─► ⚠️  R3: Timer muere para R1
    │   Estado de R1: FULL → DOWN
    │

FASE 3: INTERFAZ OFFLINE (30 segundos de espera)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  T=60.000 seg
    │
    │ ◄─────────────────────────────────────────────
    │       Comando: no shutdown interfaz Fa0/1
    │ ◄─────────────────────────────────────────────
    │

FASE 4: INTERFAZ SE REACTIVA (DOWN → ESTABLECIMIENTO)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  T=60.100 seg
    │
    ├─► R1: Interfaz Fa0/1 UP
    │   Mensaje: "OSPF: Interface Fa0/1 going Up"
    │   Estado: DOWN
    │
    ├─► R1: Enviar primer HELLO
    │   Estado: DOWN → INIT (esperando vecinos)
    │
  T=60.150 seg  ┌────────────────────────────────────┐
    │           │ HELLO: R1 → 224.0.0.5             │
    │           │ "¡Hola! Acabo de levantarme"      │
    │           │ Router ID: 192.168.78.129         │
    │           │ Área: 1                           │
    │           │ Vecinos vistos: (VACÍO - inicio)  │
    │           └────────────────────────────────────┘
    │
  T=60.200 seg  ┌────────────────────────────────────┐
    │           │ HELLO: R2 → 224.0.0.5             │
    │           │ "Veo a R3 y ¿quién es ese?"      │
    │           │ Vecinos: R3 ✓                     │
    │           │ DR: R3, BDR: R2                   │
    │           └────────────────────────────────────┘
    │           ↓
    │           R1 recibe HELLO de R2
    │           Estado en R1: INIT
    │           Mensaje: "Received HELLO from neighbor"
    │
  T=60.300 seg  Estado R2: INIT → 2-WAY
    │           (R2 ve a R1 en el HELLO recibido)
    │
  T=60.350 seg  ┌────────────────────────────────────┐
    │           │ HELLO: R3 → 224.0.0.5             │
    │           │ "Veo a R2 y ¿quién es ese?"      │
    │           │ Vecinos: R2 ✓                     │
    │           │ DR: R3, BDR: R2                   │
    │           └────────────────────────────────────┘
    │           ↓
    │           R1 recibe HELLO de R3
    │           Estado en R1: INIT
    │
  T=60.400 seg  ┌────────────────────────────────────┐
    │           │ HELLO: R1 → 224.0.0.5 (respuesta)│
    │           │ "Ahora veo a R2 y R3"             │
    │           │ Vecinos: R2, R3 ✓                │
    │           │ DR: R3, BDR: R2                   │
    │           └────────────────────────────────────┘
    │
  T=60.450 seg  Estado R2: INIT → 2-WAY ✅
    │           (Ya bidireccional)
    │           Estado R3: INIT → 2-WAY ✅
    │           (Ya bidireccional)
    │
    │           Mensaje debug:
    │           "2 Way Communication Established
    │            with neighbor 192.168.78.130"
    │
  T=60.500 seg  ┌────────────────────────────────────┐
    │           │ DBD: R2 → R1 (Master iniciar)    │
    │           │ I=1, M=1, MS=1                    │
    │           │ Seq=12345, MTU=1500               │
    │           │ "Quiero intercambiar DB"          │
    │           └────────────────────────────────────┘
    │
  T=60.550 seg  ┌────────────────────────────────────┐
    │           │ DBD: R1 → R2 (Slave response)    │
    │           │ I=0, M=0, MS=0                    │
    │           │ Seq=12346                         │
    │           │ "Ok, soy esclavo"                │
    │           └────────────────────────────────────┘
    │
  T=60.600 seg  ┌────────────────────────────────────┐
    │           │ LSR: R1 → R2 (pide LSAs)         │
    │           │ "¿Me envías tu info de red?"      │
    │           └────────────────────────────────────┘
    │
    │           Estado: EXSTART → EXCHANGE → LOADING
    │
  T=60.650 seg  ┌────────────────────────────────────┐
    │           │ LSU: R2 → R1 (envía LSAs)        │
    │           │ Router LSA de R2                  │
    │           │ Router LSA de R3                  │
    │           │ Network LSA del Switch1          │
    │           └────────────────────────────────────┘
    │
  T=60.700 seg  ┌────────────────────────────────────┐
    │           │ LSAck: R1 → R2 (confirma)        │
    │           │ "Recibí tus LSAs"                │
    │           └────────────────────────────────────┘
    │
  T=60.750 seg  Estado R1 con R2: LOADING → FULL ✅
    │           Estado R1 con R3: LOADING → FULL ✅
    │
    │           Mensaje debug:
    │           "ADJCHG: Process 1, Nbr 192.168.78.130
    │            from LOADING to FULL, Loading Done"
    │
  T=61.000 seg  ┓
    │           ┃ Intercambio similar con R3
    │           ┃
  T=62.000 seg  ┛
    │
    ├─► ✅ Estado Final: FULL ADYACENCIAS ESTABLECIDAS
    │   • R1 ↔ R2 FULL
    │   • R1 ↔ R3 FULL
    │   • BD sincronizada
    │   • Rutas OSPF disponibles
    │

DURACIÓN TOTAL: ~2 segundos (DOWN → FULL)
```

### Tabla Resumida - Transiciones de Estado

| Tiempo (seg) | Evento | R1 Estado | R2 Estado | R3 Estado | Observación |
|--------------|--------|-----------|-----------|-----------|-----------|
| 0.000 | FULL antes del cambio | FULL | FULL | FULL | ✅ Normal |
| 30.000 | Shutdown Fa0/1 en R1 | DOWN | FULL | FULL | ⚠️ Interfaz cae |
| 35.000 | R2 percibe pérdida | DOWN | INIT→DOWN | - | ⚠️ Sin HELLO de R1 |
| 45.000 | Dead timer R2 | DOWN | DOWN | DOWN | ⚠️ R1 muere en R2 |
| 60.100 | No shutdown Fa0/1 | UP | DOWN | DOWN | ✅ Interfaz levanta |
| 60.200 | Intercambio HELLO | INIT | INIT→2WAY | INIT | ✓ Bidireccional |
| 60.500 | DBD iniciado | EXSTART | EXSTART | - | ↔ Intercambio DB |
| 60.700 | LSU intercambiado | LOADING | LOADING | - | ↔ Intercambio LSAs |
| 60.800 | LSAck recibido | FULL | FULL | FULL | ✅ Sincronizado |
| 62.000 | Estable | FULL | FULL | FULL | ✅ Equilibrio alcanzado |

---

## Comando de Debugging en Consola

### Salida esperada durante el processo

```
*Mar  1 00:05:40.456: OSPF: Received HELLO from 192.168.78.130 area 1
*Mar  1 00:05:40.457: OSPF: Interface FastEthernet0/1 going Down
*Mar  1 00:05:40.458: %OSPF-5-ADJCHG: Process 1, Nbr 192.168.78.130 on FastEthernet0/1 from FULL to DOWN, Neighbor Down: Interface down or detached

*Mar  1 00:05:40.459: OSPF: Removing peer 192.168.78.130, area 1
*Mar  1 00:05:40.460: OSPF: Interface FastEthernet0/1 going Down
```

[Interfaz offline por 30 segundos]

```
*Mar  1 00:06:10.456: OSPF: Interface FastEthernet0/1 going Up
*Mar  1 00:06:10.457: OSPF: Send hello to 224.0.0.5 area 1 on FastEthernet0/1
*Mar  1 00:06:10.458: OSPF: Build router LSA for area 1, router ID 192.168.78.129

*Mar  1 00:06:10.500: OSPF: Received HELLO from 192.168.78.130 area 1
*Mar  1 00:06:10.501: OSPF: 2 Way Communication Established with neighbor 192.168.78.130 on FastEthernet0/1

*Mar  1 00:06:10.502: OSPF: Send DBD to 192.168.78.130 on FastEthernet0/1, seq 0x... mtu 1500 state MASTER

*Mar  1 00:06:10.550: OSPF: Received DBD from 192.168.78.130 on FastEthernet0/1, seq 0x...

*Mar  1 00:06:10.600: OSPF: Exchange Done with 192.168.78.130 on FastEthernet0/1

*Mar  1 00:06:11.456: %OSPF-5-ADJCHG: Process 1, Nbr 192.168.78.130 on FastEthernet0/1 from LOADING to FULL, Loading Done
```

**Explicación**:
1. Interfaz cae: "Interface going Down" + "Neighbor Down"
2. Se elimina el vecino de la topología
3. Dead timer expira (40 seg)
4. Interfaz se reactiva: "Interface going Up"
5. Se envía nuevo HELLO
6. Se recibe HELLO del vecino: "2-Way"
7. Intercambio DBD: "Exchange Done"
8. FULL adyacencia: "Loading Done" → "FULL"

---

## Tabla Comparativa: Paquetes OSPF

| Tipo | Multicast | Puerto | Propósito | Frecuencia |
|------|-----------|--------|----------|-----------|
| HELLO | 224.0.0.5 | UDP 89 | Detectar vecinos, anunciar DR/BDR | Cada 10 seg |
| DBD | 224.0.0.5 | UDP 89 | Intercambiar sumario de BD | 1 vez/sesión |
| LSR | 224.0.0.5 | UDP 89 | Solicitar LSA específicos | Bajo demanda |
| LSU | 224.0.0.5 | UDP 89 | Enviar LSAs solicitadas | Bajo demanda |
| LSAck | 224.0.0.5 | UDP 89 | Confirmar recepción LSA | Bajo demanda |

---

## Análisis: ¿Por qué algunos paquetes no llegaron?

Si al capturar no ves todos los paquetes, es probable que:

| Problema | Razón | Solución |
|----------|-------|----------|
| Solo ves HELLOs | Capturaste después del FULL | Apaga/enciende interfaz durante captura |
| Faltan DBDs | GNS3 no captura punto-a-punto | Captura en Ethernet (Switch) |
| Wireshark vacío | Captura iniciada tarde | Inicia captura, luego shutdown interfaz |

---

## Preguntas de Análisis para el Reporte

1. **¿Cuánto tiempo tardó en ir de DOWN a FULL?**
   - Respuesta: Típicamente 2-5 segundos

2. **¿Cuáles fueron los estados intermedios observados?**
   - Respuesta: DOWN → INIT → 2-WAY → EXSTART → EXCHANGE → LOADING → FULL

3. **¿Cuántos paquetes de cada tipo se enviaron?**
   - Respuesta: Contar en Wireshark: HELLO (periódicos) + DBD (1-2) + LSR/LSU (varios)

4. **¿Cuál es el significado de los campos "M bit" e "I bit" en DBD?**
   - Respuesta: I = Init (primer DBD), M = More (hay más DBDs después)

5. **¿Por qué R1 fue Master en la negociación DBD?**
   - Respuesta: Porque su Router ID es el más alto de los dos (192.168.78.129 > 192.168.78.130)

---

## Exportar Captura para el Reporte

1. En Wireshark: **File → Export Specified Packets**
2. Guardar como: `OSPF_Vecindad_R1_R2.pcap`
3. Incluir en el reporte

Luego puedes abrir de nuevo para incluir capturas en la presentación.

---

# 📝 Resumen de Respuestas Esperadas

## Para la Entrega

### Actividad 1a: Configuración IP y OSPF

✅ **Explicar**:
- Por qué no hay conectividad con Área 1
- Importancia del Área 0 (backbone)
- Cómo se comporta OSPF sin área 0

### Actividad 1b: Verificación de Tablas

✅ **Documentar**:
- Output de `show ip ospf neighbor` (vecinos)
- Output de `show ip ospf database` (topología)
- Explicación de cada campo
- Tabla resumen con interpretación

### Actividad 1c: DR y BDR

✅ **Analizar**:
- En Switch1 (192.168.78.128): DR y BDR seleccionados
- En Serial R3-R4: Por qué NO hay DR/BDR
- Algoritmo de selección: Prioridad + Router ID
- Impacto en el número de adyacencias

### Actividad 1d: Router ID

✅ **Especificar**:
- Router ID de cada router (criterio: IP más alta)
- Tabla con IPs de interfaces y Router ID resultante
- Cómo cambiarlo manualmente si fuera necesario

### Actividad 1e: Wireshark

✅ **Incluir**:
- Captura de HELLO, DBD, LSR, LSU, LSAck
- Timeline del establecimiento de vecindad
- Output de debugging desde consola
- Análisis de tiempos y transiciones de estado

---

**Duración estimada de laboratorio**: 6-8 horas  
**Entrega**: Reporte técnico de 10-15 páginas + capturas Wireshark

¡Éxito en el Lab 6! 🎯
