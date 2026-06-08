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
| Equipo | Cisco **3745** + 2 puertos Fast Ethernet adicionales + 2 tarjetas **WIC-2T** |
| Enlaces punteados | Tipo **Serial** |
| Enlaces sólidos | Tipo **Ethernet** |
| IGP 1 | **IS-IS** (círculo protocolo 1: R1 y R4) |
| IGP 2 | Libre elección → **OSPF** (círculo protocolo 2: R2 y R3) |
| Restricción crítica | **R1 NO ejecuta protocolo de enrutamiento**, pero debe garantizar conectividad hacia LAN A |

---

## 2. Topología en GNS3

### 2.1 Dispositivos

| Dispositivo | Rol | Imagen sugerida |
|-------------|-----|-----------------|
| R1 – R7 | Enrutadores | Cisco 3745 |
| LAN A | Host / gateway `.1` | VPCS o Linux |
| LAN B | Host / gateway `.1` | VPCS o Linux |
| Cloud1_internet | Internet simulada | Cloud (NAT/DHCP) |

**Módulos en cada 3745 (según enunciado):**

- `NM-2FE2W` → interfaces Fast Ethernet adicionales (`Fa1/0`, etc.)
- `WIC-2T` → interfaces seriales (`Se0/0`, `Se0/1`) si el diagrama usa enlaces punteados

### 2.2 Conexiones físicas

| Origen | Interfaz | Destino | Interfaz | Tipo |
|--------|----------|---------|----------|------|
| LAN A (`172.16.0.1/24`) | eth0 | R7 | Fa0/0 | Ethernet |
| R7 | Fa0/1 | R6 | Fa0/0 | Ethernet |
| R6 | Fa0/1 | R1 | Fa0/0 | Ethernet |
| R1 | Fa0/1 | R4 | Fa0/0 | Ethernet |
| R4 | Fa0/1 | R5 | Fa0/0 | Ethernet |
| R5 | Fa0/1 | R2 | Fa0/0 | Ethernet |
| R2 | Fa0/1 | R3 | Fa0/0 | Ethernet |
| R3 | Fa0/1 | LAN B (`10.10.10.1/24`) | eth0 | Ethernet |
| R3 | Fa1/0 | Cloud1_internet | eth2 | Ethernet |

> **Nota Serial:** Si tu diagrama muestra líneas punteadas entre routers, sustituye el par Fast Ethernet por `Serial0/0` ↔ `Serial0/0`, asigna `clock rate 64000` en el lado DCE y conserva las mismas IPs de la tabla de enlaces.

### 2.3 Esquema lógico

```
LAN A ── R7 ── R6 ── R1 ── R4 ── R5 ── R2 ── R3 ── LAN B
                                              │
                                         Cloud (Internet)
```

**Dominios de protocolo:**

- **Círculo protocolo 1 (IS-IS):** R4, R5, R6, R7 — R1 queda fuera (solo rutas estáticas).
- **Círculo protocolo 2 (OSPF):** R2, R3.
- **Frontera IGP:** enlace R5 ↔ R2 (redistribución entre IS-IS y OSPF).

---

## 3. Plan de direccionamiento

### 3.1 Subnetting — `192.168.78.0/24`

Máscara fija elegida: **`255.255.255.248` (`/29`)** → 6 hosts útiles por subred (cumple mínimo de 4).

| Enlace | Subred | Router A | IP A | Router B | IP B |
|--------|--------|----------|------|----------|------|
| R7 ↔ R6 | `192.168.78.0/29` | R7 | `.1` | R6 | `.2` |
| R6 ↔ R1 | `192.168.78.8/29` | R6 | `.9` | R1 | `.10` |
| R1 ↔ R4 | `192.168.78.16/29` | R1 | `.17` | R4 | `.18` |
| R4 ↔ R5 | `192.168.78.24/29` | R4 | `.25` | R5 | `.26` |
| R5 ↔ R2 | `192.168.78.32/29` | R5 | `.33` | R2 | `.34` |
| R2 ↔ R3 | `192.168.78.40/29` | R2 | `.41` | R3 | `.42` |

### 3.2 Redes de usuario (fuera del /24 interno)

| Red | Gateway | Conectada en |
|-----|---------|--------------|
| `172.16.0.0/24` | `172.16.0.1` (LAN A) | R7 Fa0/0 → `172.16.0.2` |
| `10.10.10.0/24` | `10.10.10.1` (LAN B) | R3 Fa0/1 |
| Internet | DHCP (Cloud) | R3 Fa1/0 |

---

## 4. Diseño de enrutamiento

| Router | Protocolo | Función |
|--------|-----------|---------|
| R1 | **Ninguno** | Solo rutas estáticas hacia el backbone |
| R4, R6, R7 | **IS-IS** (`CORE`, L2) | Círculo protocolo 1 + anuncio de LAN A desde R7 |
| R5 | **IS-IS + OSPF** | Backbone IS-IS + frontera con OSPF (redistribución) |
| R2, R3 | **OSPF 1** (área 0) | Círculo protocolo 2 + LAN B |

### 4.1 R1 — caso especial

R1 no participa en IGP. Usa:

- Ruta por defecto hacia R6 para alcanzar el resto de la red.
- Ruta específica hacia LAN A (`172.16.0.0/24`) vía R6.

### 4.2 Conectividad extremo a extremo

Para que LAN A alcance LAN B e Internet:

1. R7 anuncia `172.16.0.0/24` en IS-IS.
2. R5 redistribuye rutas IS-IS hacia OSPF.
3. R2 redistribuye rutas OSPF hacia IS-IS (o solo en R5, según prefieras un punto único de frontera).
4. R3 anuncia `10.10.10.0/24` en OSPF.

---

## 5. Configuración IOS por router

Copiar y pegar en la consola de cada dispositivo en GNS3.

### R1 — sin protocolo de enrutamiento

```ios
conf t
interface fa0/0
 ip address 192.168.78.10 255.255.255.248
 no shutdown
interface fa0/1
 ip address 192.168.78.17 255.255.255.248
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
interface fa0/1
 ip address 192.168.78.9 255.255.255.248
 no shutdown
router isis CORE
 net 49.0001.0000.0000.0006.00
 is-type level-2-only
interface fa0/0
 ip router isis CORE
interface fa0/1
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
interface fa0/0
 ip address 192.168.78.18 255.255.255.248
 no shutdown
interface fa0/1
 ip address 192.168.78.25 255.255.255.248
 no shutdown
router isis CORE
 net 49.0001.0000.0000.0004.00
 is-type level-2-only
interface fa0/0
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
interface fa0/1
 ip address 192.168.78.41 255.255.255.248
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
interface fa0/0
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
# LAN A
ip 172.16.0.10 172.16.0.1 24

# LAN B
ip 10.10.10.10 10.10.10.1 24
```

---

## 6. Pasos de implementación en GNS3

1. Crear proyecto nuevo en GNS3.
2. Arrastrar **7 routers 3745**, **2 VPCS**, **1 Cloud**.
3. En cada 3745: agregar slot con `NM-2FE2W` y `WIC-2T` (si usarás seriales).
4. Conectar cables según la tabla de la sección 2.2.
5. Iniciar todos los nodos.
6. Aplicar configuraciones de la sección 5 en orden: **R7 → R6 → R1 → R4 → R5 → R2 → R3**.
7. Configurar IPs en VPCS (LAN A y LAN B).
8. En Cloud: habilitar NAT o asignar red para que R3 obtenga IP por DHCP en Fa1/0.

---

## 7. Verificación

### 7.1 Comandos por router

```ios
show ip interface brief
show ip route
show isis neighbors
show ip ospf neighbor
```

### 7.2 Pruebas de conectividad

| Desde | Hacia | Comando esperado |
|-------|-------|------------------|
| R1 | R6 | `ping 192.168.78.9` |
| R1 | LAN A | `ping 172.16.0.1` |
| R7 | LAN A | `ping 172.16.0.1` |
| R3 | LAN B | `ping 10.10.10.10` |
| LAN A (VPCS) | LAN B | `ping 10.10.10.10` |
| LAN B (VPCS) | LAN A | `ping 172.16.0.10` |
| R3 | Internet | `ping <IP Cloud>` |

### 7.3 Checklist de cumplimiento

- [ ] Subnetting `/29` aplicado en todos los enlaces `192.168.78.x`
- [ ] IS-IS activo en R4, R5, R6, R7
- [ ] OSPF activo en R2, R3 (y frontera en R5)
- [ ] R1 **sin** `router isis` ni `router ospf`
- [ ] R1 tiene ruta hacia `172.16.0.0/24`
- [ ] LAN A (`172.16.0.0/24`) alcanzable desde LAN B
- [ ] Redistribución IS-IS ↔ OSPF en R5
- [ ] AS 65078 documentado (identificador del laboratorio)

---

## 8. Notas y correcciones respecto al borrador anterior

| Problema detectado | Corrección aplicada |
|--------------------|---------------------|
| R1 sin ruta explícita a LAN A | Se agregó `ip route 172.16.0.0` vía R6 |
| Ruta estática incorrecta en R6 hacia LAN A vía R1 | Eliminada: LAN A cuelga de **R7**, no de R1 |
| R7 no anunciaba LAN A en IS-IS | Se habilitó `ip router isis CORE` en Fa0/0 |
| Sin conectividad entre dominios IS-IS y OSPF | Redistribución mutua en **R5** |
| Documento informal y sin pasos GNS3 | Estructura README con checklist y orden de despliegue |
| Falta de verificación LAN A ↔ LAN B | Tabla de pruebas end-to-end |

---

## 9. Referencia rápida — NET IS-IS

| Router | NET |
|--------|-----|
| R4 | `49.0001.0000.0000.0004.00` |
| R5 | `49.0001.0000.0000.0005.00` |
| R6 | `49.0001.0000.0000.0006.00` |
| R7 | `49.0001.0000.0000.0007.00` |
