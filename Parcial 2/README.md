# Parcial 2 — Laboratorio GNS3 (AS 65078)

Guía para montar, configurar y verificar la topología en GNS3.

---

## 1. Requisitos del ejercicio

| Requisito | Detalle |
|-----------|---------|
| Red interna | `192.168.78.0/24` con subnetting FLSM |
| Hosts por subred | Mínimo 4 hosts útiles |
| LAN A | `172.16.0.0/24` (gateway `172.16.0.1`) |
| LAN B | `10.10.10.0/24` (gateway `10.10.10.1`) |
| Sistema autónomo | **AS 65078** |
| Equipo | Cisco **3745** + 2 puertos Fast Ethernet adicionales + **2 tarjetas WIC-2T** |
| Enlaces punteados (Serial) | **R6–R1**, **R1–R4**, **R2–R3** |
| Enlaces sólidos (Ethernet) | R7–R6, R4–R5, R5–R2, LAN A, LAN B, Cloud |
| IGP 1 | **IS-IS** (círculo protocolo 1: R1 y R4) |
| IGP 2 | Libre elección → **OSPF** (círculo protocolo 2: R2 y R3) |
| Restricción crítica | **R1 NO ejecuta protocolo de enrutamiento**, pero debe garantizar conectividad hacia LAN A |

---

## 2. Topología en GNS3

### 2.1 Dispositivos y módulos

| Dispositivo | Rol | Imagen sugerida |
|-------------|-----|-----------------|
| R1 – R7 | Enrutadores | Cisco 3745 |
| LAN A | Host / gateway `.1` | VPCS o Linux |
| LAN B | Host / gateway `.1` | VPCS o Linux |
| Cloud1_internet | Internet simulada | Cloud (NAT/DHCP) |

**Módulos en cada 3745 (según enunciado):**

| Módulo | Función |
|--------|---------|
| `NM-2FE2W` | Puertos Fast Ethernet adicionales (`Fa1/0`, etc.) |
| `WIC-2T` × 2 | Puertos seriales (`Se0/0`, `Se0/1`, `Se1/0`, `Se1/1`) |

**Puertos seriales utilizados en este laboratorio:**

| Router | Serial activo | Conecta a |
|--------|---------------|-----------|
| R1 | `Se0/0`, `Se0/1` | R6, R4 |
| R2 | `Se0/0` | R3 |
| R3 | `Se0/0` | R2 |
| R4 | `Se0/0` | R1 |
| R6 | `Se0/0` | R1 |

### 2.2 Tabla maestra de conexiones físicas

> **Fuente de verdad:** todas las demás tablas e configuraciones IOS de este README derivan de esta tabla.

| # | Origen | If. origen | Destino | If. destino | Tipo | Subred | IP origen | IP destino |
|---|--------|------------|---------|-------------|------|--------|-----------|------------|
| 1 | LAN A | eth0 | R7 | Fa0/0 | Ethernet | `172.16.0.0/24` | `172.16.0.1` | `172.16.0.2` |
| 2 | R7 | Fa0/1 | R6 | Fa0/0 | Ethernet | `192.168.78.0/29` | `192.168.78.1` | `192.168.78.2` |
| 3 | R6 | Se0/0 | R1 | Se0/0 | **Serial** | `192.168.78.8/29` | `192.168.78.9` | `192.168.78.10` |
| 4 | R1 | Se0/1 | R4 | Se0/0 | **Serial** | `192.168.78.16/29` | `192.168.78.17` | `192.168.78.18` |
| 5 | R4 | Fa0/1 | R5 | Fa0/0 | Ethernet | `192.168.78.24/29` | `192.168.78.25` | `192.168.78.26` |
| 6 | R5 | Fa0/1 | R2 | Fa0/0 | Ethernet | `192.168.78.32/29` | `192.168.78.33` | `192.168.78.34` |
| 7 | R2 | Se0/0 | R3 | Se0/0 | **Serial** | `192.168.78.40/29` | `192.168.78.41` | `192.168.78.42` |
| 8 | R3 | Fa0/1 | LAN B | eth0 | Ethernet | `10.10.10.0/24` | `10.10.10.1` | `10.10.10.10` |
| 9 | R3 | Fa1/0 | Cloud1_internet | eth2 | Ethernet | DHCP (Cloud) | — | — |

**Enlaces Serial — lado DCE (`clock rate 64000`):**

| Enlace (#) | Lado DCE | Interfaz DCE |
|------------|----------|--------------|
| R6 ↔ R1 (3) | R6 | Se0/0 |
| R1 ↔ R4 (4) | R1 | Se0/1 |
| R2 ↔ R3 (7) | R2 | Se0/0 |

> En GNS3 el cable Serial indica cuál extremo es DCE. Configura `clock rate 64000` **solo** en ese extremo. Encapsulación por defecto: **HDLC**.

### 2.3 Esquema lógico

```
LAN A ── R7 ══ R6 - - R1 - - R4 ══ R5 ══ R2 - - R3 ── LAN B
                                              │
                                         Cloud (Internet)

══  Ethernet (línea sólida)
- -  Serial (línea punteada)
```

### 2.4 Interfaces por router

| Router | Interfaz | Tipo | Conecta a (#) | IP / rol |
|--------|----------|------|---------------|----------|
| R7 | Fa0/0 | Ethernet | LAN A (1) | `172.16.0.2/24` |
| R7 | Fa0/1 | Ethernet | R6 (2) | `192.168.78.1/29` |
| R6 | Fa0/0 | Ethernet | R7 (2) | `192.168.78.2/29` |
| R6 | Se0/0 | Serial | R1 (3) | `192.168.78.9/29` — DCE |
| R1 | Se0/0 | Serial | R6 (3) | `192.168.78.10/29` |
| R1 | Se0/1 | Serial | R4 (4) | `192.168.78.17/29` — DCE |
| R4 | Se0/0 | Serial | R1 (4) | `192.168.78.18/29` |
| R4 | Fa0/1 | Ethernet | R5 (5) | `192.168.78.25/29` |
| R5 | Fa0/0 | Ethernet | R4 (5) | `192.168.78.26/29` |
| R5 | Fa0/1 | Ethernet | R2 (6) | `192.168.78.33/29` |
| R2 | Fa0/0 | Ethernet | R5 (6) | `192.168.78.34/29` |
| R2 | Se0/0 | Serial | R3 (7) | `192.168.78.41/29` — DCE |
| R3 | Se0/0 | Serial | R2 (7) | `192.168.78.42/29` |
| R3 | Fa0/1 | Ethernet | LAN B (8) | `10.10.10.1/24` |
| R3 | Fa1/0 | Ethernet | Cloud (9) | DHCP |

### 2.5 Dominios de protocolo

| Dominio | Routers | Protocolo |
|---------|---------|-----------|
| Círculo protocolo 1 | R4, R5, R6, R7 | **IS-IS** (`CORE`, L2) |
| Círculo protocolo 1 (excluido) | R1 | Sin protocolo — solo rutas estáticas |
| Círculo protocolo 2 | R2, R3 | **OSPF 1** (área 0) |
| Frontera IGP | R5 ↔ R2 (enlace 6) | Redistribución IS-IS ↔ OSPF en **R5** |

---

## 3. Plan de direccionamiento

### 3.1 Subnetting — `192.168.78.0/24`

Máscara fija: **`255.255.255.248` (`/29`)** → 6 hosts útiles por subred (cumple mínimo de 4).

| Enlace (#) | Subred | Router A | If. A | IP A | Router B | If. B | IP B | Medio |
|------------|--------|----------|-------|------|----------|-------|------|-------|
| 2 | `192.168.78.0/29` | R7 | Fa0/1 | `.1` | R6 | Fa0/0 | `.2` | Ethernet |
| 3 | `192.168.78.8/29` | R6 | Se0/0 | `.9` | R1 | Se0/0 | `.10` | **Serial** |
| 4 | `192.168.78.16/29` | R1 | Se0/1 | `.17` | R4 | Se0/0 | `.18` | **Serial** |
| 5 | `192.168.78.24/29` | R4 | Fa0/1 | `.25` | R5 | Fa0/0 | `.26` | Ethernet |
| 6 | `192.168.78.32/29` | R5 | Fa0/1 | `.33` | R2 | Fa0/0 | `.34` | Ethernet |
| 7 | `192.168.78.40/29` | R2 | Se0/0 | `.41` | R3 | Se0/0 | `.42` | **Serial** |

### 3.2 Redes de usuario

| Red | Gateway | Router | Interfaz | IP router |
|-----|---------|--------|----------|-----------|
| `172.16.0.0/24` | `172.16.0.1` | R7 | Fa0/0 | `172.16.0.2` |
| `10.10.10.0/24` | `10.10.10.1` | R3 | Fa0/1 | `10.10.10.1` |
| Internet | DHCP | R3 | Fa1/0 | asignada por Cloud |

---

## 4. Diseño de enrutamiento

| Router | Protocolo | Interfaces participantes |
|--------|-----------|--------------------------|
| R1 | **Ninguno** | Se0/0, Se0/1 — rutas estáticas |
| R4, R6, R7 | **IS-IS** | Fa y Se según tabla 2.4 |
| R5 | **IS-IS + OSPF** | Fa0/0 (IS-IS), Fa0/1 (OSPF) + redistribución |
| R2, R3 | **OSPF 1** área 0 | Fa0/0 + Se0/0 en R2; Se0/0 + Fa0/1 en R3 |

### 4.1 R1 — caso especial

R1 no ejecuta IGP. Configuración requerida:

| Ruta | Next-hop | Motivo |
|------|----------|--------|
| `0.0.0.0/0` | `192.168.78.9` (R6 Se0/0) | Salida hacia el backbone |
| `172.16.0.0/24` | `192.168.78.9` (R6) | Conectividad hacia LAN A vía IS-IS en R6→R7 |

### 4.2 Conectividad extremo a extremo

1. R7 anuncia `172.16.0.0/24` en IS-IS (Fa0/0).
2. R3 anuncia `10.10.10.0/24` en OSPF (Fa0/1).
3. **R5** redistribuye IS-IS → OSPF y OSPF → IS-IS (único punto de frontera).
4. R3 origina ruta por defecto hacia Internet (`default-information originate`).

---

## 5. Configuración IOS por router

> Las interfaces e IPs coinciden con la **tabla maestra (sección 2.2)**.

### R1 — sin protocolo de enrutamiento

```ios
conf t
interface se0/0
 ip address 192.168.78.10 255.255.255.248
 no shutdown
interface se0/1
 ip address 192.168.78.17 255.255.255.248
 clock rate 64000
 no shutdown
ip route 0.0.0.0 0.0.0.0 192.168.78.9
ip route 172.16.0.0 255.255.255.0 192.168.78.9
end
wr
```

### R6 — IS-IS

```ios
conf t
interface fa0/0
 ip address 192.168.78.2 255.255.255.248
 no shutdown
interface se0/0
 ip address 192.168.78.9 255.255.255.248
 clock rate 64000
 no shutdown
router isis CORE
 net 49.0001.0000.0000.0006.00
 is-type level-2-only
interface fa0/0
 ip router isis CORE
interface se0/0
 ip router isis CORE
end
wr
```

### R7 — IS-IS + LAN A

```ios
conf t
interface fa0/0
 ip address 172.16.0.2 255.255.255.0
 no shutdown
interface fa0/1
 ip address 192.168.78.1 255.255.255.248
 no shutdown
router isis CORE
 net 49.0001.0000.0000.0007.00
 is-type level-2-only
interface fa0/0
 ip router isis CORE
interface fa0/1
 ip router isis CORE
end
wr
```

### R4 — IS-IS

```ios
conf t
interface se0/0
 ip address 192.168.78.18 255.255.255.248
 no shutdown
interface fa0/1
 ip address 192.168.78.25 255.255.255.248
 no shutdown
router isis CORE
 net 49.0001.0000.0000.0004.00
 is-type level-2-only
interface se0/0
 ip router isis CORE
interface fa0/1
 ip router isis CORE
end
wr
```

### R5 — IS-IS + OSPF (frontera)

```ios
conf t
interface fa0/0
 ip address 192.168.78.26 255.255.255.248
 no shutdown
interface fa0/1
 ip address 192.168.78.33 255.255.255.248
 no shutdown
router isis CORE
 net 49.0001.0000.0000.0005.00
 is-type level-2-only
 redistribute ospf 1 metric 20
interface fa0/0
 ip router isis CORE
router ospf 1
 router-id 5.5.5.5
 redistribute isis CORE subnets
 network 192.168.78.32 0.0.0.7 area 0
end
wr
```

### R2 — OSPF

```ios
conf t
interface fa0/0
 ip address 192.168.78.34 255.255.255.248
 no shutdown
interface se0/0
 ip address 192.168.78.41 255.255.255.248
 clock rate 64000
 no shutdown
router ospf 1
 router-id 2.2.2.2
 network 192.168.78.32 0.0.0.7 area 0
 network 192.168.78.40 0.0.0.7 area 0
end
wr
```

### R3 — OSPF + LAN B + Internet

```ios
conf t
interface se0/0
 ip address 192.168.78.42 255.255.255.248
 no shutdown
interface fa0/1
 ip address 10.10.10.1 255.255.255.0
 no shutdown
interface fa1/0
 ip address dhcp
 no shutdown
router ospf 1
 router-id 3.3.3.3
 network 192.168.78.40 0.0.0.7 area 0
 network 10.10.10.0 0.0.0.255 area 0
 default-information originate
end
wr
```

### Hosts LAN (VPCS)

```bash
# LAN A — enlace #1
ip 172.16.0.10 172.16.0.1 24

# LAN B — enlace #8
ip 10.10.10.10 10.10.10.1 24
```

---

## 6. Pasos de implementación en GNS3

1. Crear proyecto nuevo en GNS3.
2. Arrastrar **7 routers 3745**, **2 VPCS**, **1 Cloud**.
3. En cada 3745: instalar `NM-2FE2W` + **2× WIC-2T** (según enunciado).
4. Cablear según **tabla maestra 2.2**:
   - Enlaces **1, 2, 5, 6, 8, 9** → cable **Ethernet**.
   - Enlaces **3, 4, 7** → cable **Serial**.
5. Iniciar todos los nodos.
6. Aplicar configs IOS (sección 5) en orden: **R7 → R6 → R1 → R4 → R5 → R2 → R3**.
7. Verificar seriales `up/up`: `show ip interface brief`.
8. Configurar VPCS (LAN A y LAN B).
9. Configurar Cloud/NAT para DHCP en R3 Fa1/0 (enlace #9).

---

## 7. Verificación

### 7.1 Comandos por router

```ios
show ip interface brief
show interfaces serial 0/0
show interfaces serial 0/1    ! solo en R1
show controllers serial 0/0
show ip route
show isis neighbors
show ip ospf neighbor
```

### 7.2 Pruebas de conectividad

| Desde | Hacia | Enlace # | Comando |
|-------|-------|----------|---------|
| R1 | R6 | 3 | `ping 192.168.78.9` |
| R1 | R4 | 4 | `ping 192.168.78.18` |
| R1 | LAN A | 1 vía R6/R7 | `ping 172.16.0.1` |
| R2 | R3 | 7 | `ping 192.168.78.42` |
| R7 | LAN A | 1 | `ping 172.16.0.1` |
| R3 | LAN B | 8 | `ping 10.10.10.10` |
| LAN A (VPCS) | LAN B | 1→8 | `ping 10.10.10.10` |
| LAN B (VPCS) | LAN A | 8→1 | `ping 172.16.0.10` |

### 7.3 Checklist de cumplimiento

- [ ] Cableado coincide con tabla maestra 2.2 (9 enlaces)
- [ ] Serial en enlaces **3, 4 y 7** (R6–R1, R1–R4, R2–R3)
- [ ] Ethernet en enlaces **1, 2, 5, 6, 8, 9**
- [ ] `clock rate 64000` en DCE: R6 Se0/0, R1 Se0/1, R2 Se0/0
- [ ] Subnetting `/29` en enlaces 2–7
- [ ] IS-IS en R4, R5, R6, R7
- [ ] OSPF en R2, R3 y frontera en R5
- [ ] R1 **sin** `router isis` ni `router ospf`
- [ ] R1 con rutas estáticas a `0.0.0.0/0` y `172.16.0.0/24`
- [ ] Redistribución IS-IS ↔ OSPF **solo en R5**
- [ ] LAN A alcanzable desde LAN B
- [ ] AS **65078** identificado en el laboratorio

---

## 8. Validación de consistencia interna

Auditoría cruzada entre todas las secciones del README:

| Verificación | Estado |
|--------------|--------|
| Tabla 2.2 ↔ Tabla 2.4 (interfaces e IPs) | ✅ Coinciden en 15 interfaces |
| Tabla 2.2 ↔ Tabla 3.1 (subredes enlaces 2–7) | ✅ Mismas subredes e IPs |
| Tabla 2.2 ↔ Config IOS (sección 5) | ✅ Cada `interface` e IP verificada |
| Enlaces Serial 2.2 ↔ DCE 2.2 ↔ `clock rate` en IOS | ✅ R6 Se0/0, R1 Se0/1, R2 Se0/0 |
| Enlaces Serial ↔ Medio en 3.1 | ✅ Filas 3, 4 y 7 |
| Círculo IS-IS ↔ routers con `router isis` | ✅ R4, R5, R6, R7 — R1 excluido |
| Círculo OSPF ↔ routers con `router ospf` | ✅ R2, R3, R5 |
| LAN A en R7 Fa0/0 | ✅ `172.16.0.2` — enlace #1 |
| LAN B en R3 Fa0/1 | ✅ `10.10.10.1` — enlace #8 |
| Cloud en R3 Fa1/0 | ✅ Requiere `NM-2FE2W` — enlace #9 |
| R1 next-hop estático `192.168.78.9` | ✅ R6 Se0/0 enlace #3 |
| Redistribución | ✅ Solo R5 (corregido; R2 no redistribuye) |

---

## 9. Referencia rápida — NET IS-IS

| Router | NET |
|--------|-----|
| R4 | `49.0001.0000.0000.0004.00` |
| R5 | `49.0001.0000.0000.0005.00` |
| R6 | `49.0001.0000.0000.0006.00` |
| R7 | `49.0001.0000.0000.0007.00` |
