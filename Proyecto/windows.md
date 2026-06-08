# Guía de Implementación del Proyecto: Optimización de Latencia para Gaming con QoS

**Proyecto**: Diseñar un laboratorio académico con optimización de latencia mediante selección de rutas y QoS.

**Herramienta**: GNS3 con máquinas virtuales Linux reales  
**Protocolo de enrutamiento**: OSPF (no EIGRP)  
**Duración estimada**: 12 semanas

---

## 📋 Índice de Fases

1. [Fase 1: Planificación y Diseño](#fase-1-planificación-y-diseño)
2. [Fase 2: Implementación en GNS3](#fase-2-implementación-en-gns3)
3. [Fase 3: Simulación de Tráfico](#fase-3-simulación-de-tráfico)
4. [Fase 4: Configuración de QoS](#fase-4-configuración-de-qos)
5. [Fase 5: Validación y Análisis](#fase-5-validación-y-análisis)
6. [Fase 6: Documentación Final](#fase-6-documentación-final)

---

# FASE 1: Planificación y Diseño

**Duración**: Semanas 1-2  
**Objetivo**: Definir la topología, direccionamiento IP y parámetros de latencia.

## 1.1 Topología de Red

### Diagrama conceptual

```text
                   Ruta A: menor retardo (costo OSPF 100)
PC Gamer --- SW1 --- R1 ---- R2 ---- R4 --- SW2 --- Servidor
                    \\                    /
                     \\                  /
                      ---- R3 ----------

Ruta B: mayor retardo (costo OSPF 350)
```

### Descripción de equipos

| Equipo | Rol | Función |
|--------|-----|---------|
| PC Gamer | VPCS Cliente | Genera tráfico UDP sensible a latencia (puerto 5001) |
| PC Streaming | VPCS Cliente | Genera tráfico UDP/TCP continuo (puerto 5002) |
| PC Descargas | VPCS Cliente | Genera tráfico TCP pesado (puerto 5003) |
| R1 | Router de borde (cliente) | Entrada de la red, marca tráfico, aplica QoS |
| R2 | Router intermedio (ruta rápida) | Part of ruta A (costo 50 por enlace) |
| R3 | Router intermedio (ruta lenta) | Parte de ruta B (costo 200/150) |
| R4 | Router de borde (servidor) | Salida de la red |
| Servidor | VPCS (puerto de gestión) | Responde a pings; para pruebas con iperf3 usa tu PC Linux local (ver nota) |

## 1.2 Plan de Direccionamiento IP

### Asignación de subredes

| Subred | Propósito | Rango |
|--------|----------|-------|
| 192.168.10.0/24 | LAN Cliente | 192.168.10.1 - 192.168.10.254 |
| 192.168.40.0/24 | LAN Servidor | 192.168.40.1 - 192.168.40.254 |
| 10.0.12.0/30 | Enlace R1-R2 | 10.0.12.0 - 10.0.12.3 |
| 10.0.13.0/30 | Enlace R1-R3 | 10.0.13.0 - 10.0.13.3 |
| 10.0.24.0/30 | Enlace R2-R4 | 10.0.24.0 - 10.0.24.3 |
| 10.0.34.0/30 | Enlace R3-R4 | 10.0.34.0 - 10.0.34.3 |

### Direcciones específicas

**Routers:**

| Router | Interfaz | IP | Máscara |
|--------|----------|----|----|
| R1 | Fa0/0 | 192.168.10.1 | 255.255.255.0 |
| R1 | Serial0/0 | 10.0.12.1 | 255.255.255.252 |
| R1 | Serial0/1 | 10.0.13.1 | 255.255.255.252 |
| R2 | Serial0/0 | 10.0.12.2 | 255.255.255.252 |
| R2 | Serial0/1 | 10.0.24.1 | 255.255.255.252 |
| R3 | Serial0/0 | 10.0.13.2 | 255.255.255.252 |
| R3 | Serial0/1 | 10.0.34.1 | 255.255.255.252 |
| R4 | Serial0/0 | 10.0.24.2 | 255.255.255.252 |
| R4 | Serial0/1 | 10.0.34.2 | 255.255.255.252 |
| R4 | Fa0/0 | 192.168.40.1 | 255.255.255.0 |

**Hosts clientes (VPCS):**

| Host | IP | Gateway | Propósito |
|------|----|----|---------|
| PC Gamer | 192.168.10.10 | 192.168.10.1 | Tráfico UDP juego (puerto 5001) |
| PC Streaming | 192.168.10.20 | 192.168.10.1 | Tráfico UDP streaming (puerto 5002) |
| PC Descargas | 192.168.10.30 | 192.168.10.1 | Tráfico TCP descargas (puerto 5003) |
| Servidor | 192.168.40.10 | 192.168.40.1 | Responde tráfico en puerto 5001,5002,5003 |

## 1.3 Diseño de Latencia y Ancho de Banda

### Parámetros de enlace

**Ruta A (Preferida):**
- Enlace R1-R2: delay 5000 μs, OSPF cost 50
- Enlace R2-R4: delay 5000 μs, OSPF cost 50
- **Costo total: 100** ✓ Esta ruta será preferida

**Ruta B (Alternativa):**
- Enlace R1-R3: delay 20000 μs, OSPF cost 200
- Enlace R3-R4: delay 15000 μs, OSPF cost 150
- **Costo total: 350** (solo si Ruta A falla)

### Justificación

La ruta A simula un camino rápido (bajo retardo, alto ancho de banda) mientras que la ruta B simula un camino lento (alto retardo, bajo ancho de banda). OSPF elegirá automáticamente la ruta A porque su costo (100) es menor que el de la ruta B (350).

## 1.4 Tipos de Tráfico a Simular

| Tráfico | Protocolo | Puerto | Características | Herramienta |
|---------|-----------|--------|-----------------|------------|
| Gaming | UDP | 5001 | Pequeño, constante, sensible a jitter | iperf3 |
| Streaming | UDP/TCP | 5002 | Volumen medio, tolera mayor retardo | iperf3 |
| Descargas | TCP | 5003 | Alto volumen, congestiona la red | iperf3 |

---

## ✅ Verificación de Fase 1

**Completado cuando:**

- [ ] Diagrama de topología validado por profesor/grupo
- [ ] Plan de direccionamiento IP documentado y revisado
- [ ] Parámetros de latencia definidos (delays en μs y costos OSPF)
- [ ] Tipos de tráfico y puertos especificados
- [ ] Documento de diseño entregado con todas las tablas anteriores

**Entregable**: Documento "Diseño_Fase1.md" con:
- Diagrama de topología
- Tabla de direccionamiento IP
- Tabla de parámetros de latencia
- Justificación de selección de rutas

---

# FASE 2: Implementación en GNS3

**Duración**: Semanas 3-4  
**Objetivo**: Montar la topología física, configurar IPs y validar conectividad básica.

## 2.1 Preparación del Entorno GNS3

### Requisitos previos

1. GNS3 instalado (v2.2+)
2. IOSv o IOU descargados (imágenes Cisco)
3. VPCS configuradas para los hosts (una por equipo)
4. iperf3 instalado en tu PC Windows/Linux (obligatorio para Fase 4; ver nota)

### Instalación de iperf3 (PC Windows o Linux — no en VPCS)

**VPCS no ejecuta `iperf3`.** Los comandos `iperf3` se ejecutan en tu PC host conectado al lab con nodos **Cloud** (ver sección 4.0). En Linux:

```bash
# En tu PC Linux local que actuará como servidor iperf3
sudo apt-get update
sudo apt-get install -y iperf3

# Verificar instalación
iperf3 --version
```

En Windows: `winget install iperf3`. Conecta el PC host al lab con nodos Cloud (Fase 4, sección 4.0).

## 2.2 Montaje de la Topología

### Paso 1: Crear el proyecto GNS3

1. Abre GNS3
2. Crea un nuevo proyecto: **File → New Project**
3. Nombre: `Proyecto_Gaming_QoS`
4. Selecciona servidor local

### Paso 2: Agregar equipos

1. **Routers**: Arrastra 4 routers IOSv/IOU desde el panel izquierdo
   - Renombra como R1, R2, R3, R4
   - Inicia todos (memoria: 512MB cada uno)

2. **VPCS**: Crea 4 VPCS en GNS3 (una por cada host)
  - `PC_Gamer` (192.168.10.10)
  - `PC_Streaming` (192.168.10.20)
  - `PC_Descargas` (192.168.10.30)
  - `Servidor` (192.168.40.10)  (nota: VPCS no ejecuta iperf3)

  **Agregar servidor VPCS (opción A):**
  - Arrastra un nuevo VPCS al proyecto y renómbralo como `Servidor`.
  - Conéctalo al switch `SW2`, que enlaza con `R4`.
  - Abre su consola por telnet usando el puerto asignado por GNS3.
  - Configura IP, gateway y guarda la configuración con `save`.
  - Verifica conectividad con `ping` hacia `192.168.40.1`.

3. **Switches**: Agregar 2 switches ethernet (simular LANs)
   - `SW1` (conecta cliente con R1)
   - `SW2` (conecta servidor con R4)

### Paso 3: Conectar componentes

**Conexiones físicas:**

| De | Interface | A | Interface | Tipo |
|----|-----------|---|-----------|------|
| R1 | Fa0/0 | SW1 | Port 1 | Ethernet |
| SW1 | Port 2 | PC_Gamer | eth0 | Ethernet |
| SW1 | Port 3 | PC_Streaming | eth0 | Ethernet |
| SW1 | Port 4 | PC_Descargas | eth0 | Ethernet |
| R1 | Serial0/0 | R2 | Serial0/0 | Serial (DTE/DCE) |
| R1 | Serial0/1 | R3 | Serial0/0 | Serial (DTE/DCE) |
| R2 | Serial0/1 | R4 | Serial0/0 | Serial (DTE/DCE) |
| R3 | Serial0/1 | R4 | Serial0/1 | Serial (DTE/DCE) |
| R4 | Fa0/0 | SW2 | Port 1 | Ethernet |
| SW2 | Port 2 | Servidor | eth0 | Ethernet |

## 2.3 Configuración de Direcciones IP

### Configurar VPCS (PCs y Servidor)

En este proyecto usamos exclusivamente VPCS. Como GNS3 corre en tu PC Linux, cada VPCS se configura mediante telnet a `localhost` usando el puerto local asignado por GNS3. Ejemplo de pasos (puertos de ejemplo):

1. PC_Gamer (puerto telnet: `5000`):

```bash
telnet localhost 5000
# en el prompt de VPCS
ip 192.168.10.10/24 192.168.10.1
save
ping 192.168.10.1
exit
```

2. PC_Streaming (puerto telnet: `5002`):

```bash
telnet localhost 5002
ip 192.168.10.20/24 192.168.10.1
save
ping 192.168.10.1
exit
```

3. PC_Descargas (puerto telnet: `5004`):

```bash
telnet localhost 5004
ip 192.168.10.30/24 192.168.10.1
save
ping 192.168.10.1
exit
```

4. Servidor (VPCS de gestión) (puerto telnet: `5010`):

```bash
telnet localhost 5010
ip 192.168.40.10/24 192.168.40.1
save
ping 192.168.40.1
exit
```

5. Verificacion de asignacion IP:

```bash
PC-Gamer> show ip

NAME        : PC-Gamer[1]
IP/MASK     : 192.168.10.10/24
GATEWAY     : 192.168.10.1
DNS         : 
MAC         : 00:50:79:66:68:00
LPORT       : 10012
RHOST:PORT  : 127.0.0.1:10013
MTU         : 1500

```

Puertos de consola/telnet para routers y VPCS en tu PC Linux (ejemplos asignados en tu proyecto):

| Dispositivo | Puerto telnet |
|-------------|---------------|
| PC-Gamer | 5000 |
| PC-Streaming | 5002 |
| PC-Descargas | 5004 |
| R1 (consola) | 5006 |
| R2 (consola) | 5007 |
| R3 (consola) | 5008 |
| Servidor | 5010 |
| R4 (consola) | 5009 |

Notas:
- Los puertos `5001..5004` son ejemplos; usa los puertos que GNS3 asignó a cada VPCS en tu proyecto.
- Sintaxis `ip` en VPCS: `ip <dirección>/<prefijo> <gateway>`.
- `save` persiste la configuración del VPCS entre reinicios del proyecto.
- VPCS es limitado: no puede ejecutar servicios complejos como `iperf3`. Para pruebas de throughput puedes:
  - usar tu PC Linux local con `iperf3` y apuntar los VPCS hacia ella, o
  - ejecutar contenedores locales si necesitas `iperf3` dentro del laboratorio.

### Configurar R1

```ios
enable
configure terminal

hostname R1
no ip domain-lookup

interface FastEthernet0/0
 description LAN CLIENTE
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit

interface Serial0/0
 description ENLACE A R2
 ip address 10.0.12.1 255.255.255.252
 clock rate 64000
 no shutdown
 exit

interface Serial0/1
 description ENLACE A R3
 ip address 10.0.13.1 255.255.255.252
 clock rate 64000
 no shutdown
 exit

end
write memory
```

### Configurar R2

```ios
enable
configure terminal

hostname R2
no ip domain-lookup

interface Serial0/0
 description ENLACE A R1
 ip address 10.0.12.2 255.255.255.252
 no shutdown
 exit

interface Serial0/1
 description ENLACE A R4
 ip address 10.0.24.1 255.255.255.252
 clock rate 64000
 no shutdown
 exit

end
write memory
```

### Configurar R3

```ios
enable
configure terminal

hostname R3
no ip domain-lookup

interface Serial0/0
 description ENLACE A R1
 ip address 10.0.13.2 255.255.255.252
 no shutdown
 exit

interface Serial0/1
 description ENLACE A R4
 ip address 10.0.34.1 255.255.255.252
 clock rate 64000
 no shutdown
 exit

end
write memory
```

### Configurar R4

```ios
enable
configure terminal

hostname R4
no ip domain-lookup

interface Serial0/0
 description ENLACE A R2
 ip address 10.0.24.2 255.255.255.252
 no shutdown
 exit

interface Serial0/1
 description ENLACE A R3
 ip address 10.0.34.2 255.255.255.252
 no shutdown
 exit

interface FastEthernet0/0
 description LAN SERVIDOR
 ip address 192.168.40.1 255.255.255.0
 no shutdown
 exit

end
write memory
```

## 2.4 Validación de Conectividad Básica

### Prueba 1: Conectividad en LANs

**Desde PC_Gamer:**
```bash
ping -c 4 192.168.10.1   # Gateway
ping -c 4 192.168.10.20  # PC_Streaming
ping -c 4 192.168.10.30  # PC_Descargas
```

**Esperado**: 4 paquetes transmitidos, 4 recibidos (0% pérdida)

### Prueba 2: Conectividad entre LANs sin OSPF

**Desde PC_Gamer:**
```bash
ping -c 4 192.168.40.10  # Servidor (sin OSPF aún)
```

**Esperado**: Fallar (timeout) porque no hay rutas aún

---

## ✅ Verificación de Fase 2

**Completado cuando:**

- [ ] Topología montada en GNS3 con 4 routers y 4 VPCS
- [ ] Todas las interfaces configuradas con IPs correctas
- [ ] Interfaces de router en estado "up"
- [ ] PCs pueden hacer ping dentro de sus LANs
- [ ] Switches conectan correctamente todos los equipos

**Validación con comandos:**

```ios
R1# show ip interface brief
R1# show running-config
R1# ping 10.0.12.2  (debe responder)
```

**Entregable**: Archivo GNS3 con topología completa + archivo de configuración de IPs

---

# FASE 3: Implementación de OSPF y Enrutamiento

**Duración**: Semanas 3-4 (paralelamente a Fase 2)  
**Objetivo**: Configurar OSPF con costos manual para preferir la ruta A.

## 3.1 Configuración OSPF en Routers

### Configurar R1 con OSPF

```ios
configure terminal

! Definir delays para simular latencia
interface Serial0/0
 delay 5000       ! 5ms en ruta A
 ip ospf cost 50  ! Costo bajo para ruta rápida

interface Serial0/1
 delay 20000      ! 20ms en ruta B
 ip ospf cost 200 ! Costo alto para ruta lenta

! Activar OSPF
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.13.0 0.0.0.3 area 0
 passive-interface FastEthernet0/0  ! No envía OSPF por LAN cliente

end
write memory
```

### Configurar R2 con OSPF

```ios
configure terminal

interface Serial0/0
 delay 5000      ! Ruta A
 ip ospf cost 50

interface Serial0/1
 delay 5000      ! Ruta A
 ip ospf cost 50

router ospf 1
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.24.0 0.0.0.3 area 0

end
write memory
```

### Configurar R3 con OSPF

```ios
configure terminal

interface Serial0/0
 delay 20000     ! Ruta B
 ip ospf cost 200

interface Serial0/1
 delay 15000     ! Ruta B
 ip ospf cost 150

router ospf 1
 network 10.0.13.0 0.0.0.3 area 0
 network 10.0.34.0 0.0.0.3 area 0

end
write memory
```

### Configurar R4 con OSPF

```ios
configure terminal

interface Serial0/0
 delay 5000      ! Ruta A
 ip ospf cost 50

interface Serial0/1
 delay 15000     ! Ruta B
 ip ospf cost 150

router ospf 1
 network 10.0.24.0 0.0.0.3 area 0
 network 10.0.34.0 0.0.0.3 area 0
 network 192.168.40.0 0.0.0.255 area 0
 passive-interface FastEthernet0/0  ! No envía OSPF por LAN servidor

end
write memory
```

## 3.2 Verificación de OSPF y Selección de Rutas

### Verificar adyacencias OSPF

**En R1:**
```ios
show ip ospf neighbor
```

**Esperado:**
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.34.1         0   FULL/  -        00:00:34    10.0.13.2       Serial0/1
10.0.24.1         0   FULL/  -        00:00:34    10.0.12.2       Serial0/0
```

### Verificar tabla de rutas OSPF

**En R1:**
```ios
show ip route ospf
```

**Esperado** (debe mostrar ruta a 192.168.40.0 a través de R2 — Ruta A):
```
O    192.168.40.0/24 [110/210] via 10.0.12.2, 00:02:30, Serial0/0
     10.0.0.0/30 is subnetted, 4 subnets
O       10.0.24.0 [110/200] via 10.0.12.2, 00:02:30, Serial0/0
O       10.0.34.0 [110/350] via 10.0.13.2, 00:02:30, Serial0/1
                  [110/350] via 10.0.12.2, 00:02:30, Serial0/0
```

*Nota: La ruta a 192.168.40.0 usa Serial0/0 (Ruta A, costo 210). La red 10.0.34.0 muestra dos caminos con el mismo costo (350) por ECMP.*

### Verificar cost en interfaces

```ios
show ip ospf interface Serial0/0
```

**Esperado**:
```
Serial0/0 is up, line protocol is up
  Internet Address 10.0.12.1/30, Area 0
  Process ID 1, Router ID 192.168.10.1, Network Type POINT_TO_POINT, Cost: 50
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:08
  Supports Link-local Signaling (LLS)
  Index 1/1, flood queue length 0
  Next 0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 4 msec, maximum is 4 msec
  Neighbor Count is 1, Adjacent neighbor count is 1
    Adjacent with neighbor 10.0.24.1
  Suppress hello for 0 neighbor(s)
```

## 3.3 Pruebas de Conectividad con OSPF

### Desde PC_Gamer hacia Servidor

```bash
ping -c 4 192.168.40.10
```

**Esperado**: 4 paquetes transmitidos, 4 recibidos ✓

### Verificar ruta con traceroute

```bash
traceroute 192.168.40.10
```

**Esperado**:
```
traceroute to 192.168.40.10 (192.168.40.10), 30 hops max
  1   192.168.10.1      5.234 ms     5.187 ms     5.245 ms  (R1)
  2   10.0.12.2         15.123 ms    15.456 ms    15.234 ms (R2)
  3   10.0.24.2         25.567 ms    25.234 ms    25.456 ms (R4)
  4   192.168.40.10     35.678 ms    35.234 ms    35.456 ms (Servidor)
```

*Nota: Los tiempos dependerán de la simulación de GNS3*

---

## ✅ Verificación de Fase 3

**Completado cuando:**

- [ ] OSPF habilitado en todos los routers (área 0)
- [ ] Todos los routers tienen adyacencias OSPF activas (FULL state)
- [ ] Tabla de rutas muestra ruta A como preferida (via 10.0.12.2, costo 210)
- [ ] Ping exitoso desde PC_Gamer a Servidor
- [ ] Traceroute muestra la ruta correcta (R1 → R2 → R4)

**Entregable**: Outputs de `show` commands + capturas de pruebas de conectividad

---

# FASE 4: Simulación de Tráfico sin QoS

**Duración**: Semanas 5-6  
**Objetivo**: Generar tráfico real con iperf3 y medir métricas baseline (sin QoS).

> **Linux y Windows:** Cada sección incluye comandos para ambos sistemas. Si trabajas desde un PC Windows, usa los bloques marcados como *Windows*.

## ⚠️ Importante: VPCS no ejecuta iperf3

Los nodos **PC-Gamer**, **PC-Streaming**, **PC-Descargas** y **Servidor** en este proyecto son **VPCS** (Virtual PC Simulator). VPCS solo admite comandos básicos:

```text
PC-Gamer> ?
```

Comandos disponibles en VPCS: `ping`, `trace`, `ip`, `save`, `show`, `set`, `echo`, `clear`, `help`, etc.

**No existe `iperf3` en VPCS.** Si escribes:

```text
PC-Gamer> iperf3 -u -c 192.168.40.10 -p 5001 -b 1M -l 120 -t 30
Bad command: "iperf3 ...". Use ? for help.
```

Eso es el comportamiento esperado. Para Fase 4 debes ejecutar `iperf3` en tu **PC Windows** (o Linux) conectado al laboratorio, no en la consola de VPCS.

| Tarea | Dónde ejecutarla |
|-------|------------------|
| `ping`, `trace`, verificar IP | Consola VPCS (`PC-Gamer>`) ✓ |
| `iperf3` cliente o servidor | PC Windows/Linux con Cloud en GNS3 ✓ |
| `iperf3` en consola VPCS | ✗ No soportado |

## 4.0 Conectar tu PC Windows al laboratorio GNS3

Para que `iperf3` atraviese routers, switches y OSPF, enlaza tu PC al lab con nodos **Cloud**.

### Paso 1: Cloud en lado servidor (iperf3 servidor)

1. En GNS3, arrastra un nodo **Cloud** al proyecto.
2. Renómbralo `Cloud_Servidor`.
3. Conéctalo: `Cloud_Servidor eth0` → `SW2` (mismo puerto donde está el VPCS Servidor).
4. Clic derecho en `Cloud_Servidor` → **Configure**.
5. En **Ethernet interfaces**, elige un adaptador de tu PC, por ejemplo:
   - `Npcap Loopback Adapter`, o
   - `Loopback Pseudo-Interface`, o
   - una tarjeta Ethernet física dedicada al lab.
6. Aplica y arranca el proyecto.

En **Windows** (Panel de control → Redes → Propiedades del adaptador elegido), asigna IP estática:

| Campo | Valor |
|-------|-------|
| IP | `192.168.40.10` |
| Máscara | `255.255.255.0` |
| Puerta de enlace | `192.168.40.1` |

Verifica desde CMD (no desde VPCS):

```cmd
ping 192.168.40.1
ping 192.168.10.10
```

*Si el VPCS Servidor también usa 192.168.40.10, apágalo o cámbiale la IP (ej. `192.168.40.11`) para evitar conflicto.*

### Paso 2: Cloud en lado cliente (iperf3 cliente)

1. Arrastra otro **Cloud**, renómbralo `Cloud_Cliente`.
2. Conéctalo: `Cloud_Cliente eth0` → `SW1` (LAN cliente).
3. Configura el mismo tipo de adaptador (puede ser otro adaptador loopback distinto).
4. Asigna en Windows:

| Campo | Valor |
|-------|-------|
| IP | `192.168.10.10` |
| Máscara | `255.255.255.0` |
| Puerta de enlace | `192.168.10.1` |

Verifica:

```cmd
ping 192.168.10.1
ping 192.168.40.10
```

*Si PC-Gamer VPCS ya usa 192.168.10.10, cámbiale la IP en VPCS (ej. `192.168.10.11`) o desconéctalo temporalmente.*

### Paso 3: Flujo de trabajo Fase 4

```text
[PC Windows - Cloud_Cliente 192.168.10.10]  →  R1 → … →  R4  →  [PC Windows - Cloud_Servidor 192.168.40.10]
         iperf3 -c … (cliente)                                              iperf3 -s … (servidor)

[VPCS PC-Gamer]  →  solo ping / trace para comprobar conectividad
```

## 4.1 Configuración del Servidor iperf3

### En tu PC Linux local con iperf3

```bash
# Iniciar iperf3 en modo servidor (daemon)
iperf3 -s -D

# Verificar que está escuchando
netstat -tuln | grep iperf
ps aux | grep iperf3
```

Para los tres flujos del proyecto, abre instancias adicionales en otros puertos:

```bash
iperf3 -s -p 5001 -D
iperf3 -s -p 5002 -D
iperf3 -s -p 5003 -D
```

### En tu PC Windows local con iperf3

**Instalación** (si aún no lo tienes):

```cmd
winget install iperf3
```

O descarga el binario desde [iperf.fr](https://iperf.fr/iperf-download.php) y agrégalo al PATH.

**Prueba local** (verificar que iperf3 funciona):

Ventana 1 (CMD o PowerShell) — servidor:

```cmd
iperf3 -s
```

Ventana 2 — cliente de prueba:

```cmd
iperf3 -c 127.0.0.1
```

Debes ver throughput en ambas ventanas. Cierra con `Ctrl+C` en la ventana del servidor.

**Servidor para el laboratorio** (ejecutar en el PC conectado por `Cloud_Servidor`, IP `192.168.40.10`; un proceso por puerto — abre 3 ventanas):

Ventana 1 — gaming (UDP 5001):

```cmd
iperf3 -s -p 5001
```

Ventana 2 — streaming (UDP 5002):

```cmd
iperf3 -s -p 5002
```

Ventana 3 — descargas (TCP 5003):

```cmd
iperf3 -s -p 5003
```

**Verificar que está escuchando:**

```cmd
netstat -an | findstr "5001 5002 5003"
```

Debes ver líneas con `LISTENING` en los puertos 5001, 5002 y 5003.

*Nota: En Windows no existe el modo daemon (`-D`). Deja las tres ventanas abiertas mientras duren las pruebas.*

## 4.2 Generación de Tráfico de Prueba

### Prueba 1: Línea Base - Tráfico Individual

> Ejecuta estos comandos en **CMD/PowerShell de Windows** (adaptador `Cloud_Cliente`), **no** en la consola VPCS `PC-Gamer>`.

**Gaming (UDP) — simula PC_Gamer (puerto 5001):**

Linux (PC host conectado al lab):

```bash
iperf3 -u -c 192.168.40.10 -p 5001 -b 1M -l 120 -t 30
```

Windows (CMD o PowerShell en `Cloud_Cliente`):

```cmd
iperf3 -u -c 192.168.40.10 -p 5001 -b 1M -l 120 -t 30
```

**Captura:**
- Copiar salida completa (throughput, jitter, pérdida)
- Registrar RTT con ping simultáneo (desde VPCS o desde Windows):

```text
PC-Gamer> ping 192.168.40.10 -c 20
```

```cmd
ping -n 20 192.168.40.10
```

**Streaming (UDP) — simula PC_Streaming (puerto 5002):**

Linux:

```bash
iperf3 -u -c 192.168.40.10 -p 5002 -b 8M -l 1400 -t 30
```

Windows:

```cmd
iperf3 -u -c 192.168.40.10 -p 5002 -b 8M -l 1400 -t 30
```

**Descargas (TCP) — simula PC_Descargas (puerto 5003):**

Linux:

```bash
iperf3 -c 192.168.40.10 -p 5003 -P 4 -t 30
```

Windows:

```cmd
iperf3 -c 192.168.40.10 -p 5003 -P 4 -t 30
```

### Prueba 2: Congestión Simultánea (SIN QoS)

Ejecutar los tres tipos de tráfico en paralelo para congestionar la red.

**Linux (PC host conectado al lab):**

```bash
iperf3 -u -c 192.168.40.10 -p 5001 -b 1M -l 120 -t 60 &
iperf3 -u -c 192.168.40.10 -p 5002 -b 8M -l 1400 -t 60 &
iperf3 -c 192.168.40.10 -p 5003 -P 4 -t 60 &
wait
```

**Windows (CMD en `Cloud_Cliente`):**

Abre tres ventanas de CMD y ejecuta una en cada una:

Ventana 1 — tráfico gaming:

```cmd
iperf3 -u -c 192.168.40.10 -p 5001 -b 1M -l 120 -t 60
```

Ventana 2 — tráfico streaming:

```cmd
iperf3 -u -c 192.168.40.10 -p 5002 -b 8M -l 1400 -t 60
```

Ventana 3 — tráfico descargas:

```cmd
iperf3 -c 192.168.40.10 -p 5003 -P 4 -t 60
```

**Windows (PowerShell — lanzar en segundo plano desde una sola ventana):**

```powershell
Start-Process iperf3 -ArgumentList "-u","-c","192.168.40.10","-p","5001","-b","1M","-l","120","-t","60"
Start-Process iperf3 -ArgumentList "-u","-c","192.168.40.10","-p","5002","-b","8M","-l","1400","-t","60"
Start-Process iperf3 -ArgumentList "-c","192.168.40.10","-p","5003","-P","4","-t","60"
```

Espera ~60 segundos a que terminen los tres procesos.

## 4.3 Medición de Métricas Baseline

### Métrica 1: Latencia (RTT)

**Ejecutar durante el tráfico de congestión:**

Linux (PC host):

```bash
# Mientras corre iperf3
ping -c 100 192.168.40.10 > ping_sin_qos.txt
grep min ping_sin_qos.txt
```

Desde VPCS (opcional, mientras corre iperf3):

```text
PC-Gamer> ping 192.168.40.10 -c 100
```

Windows (CMD en `Cloud_Cliente`):

```cmd
REM Mientras corre iperf3
ping -n 100 192.168.40.10 > ping_sin_qos.txt

REM Analizar resultados
findstr /i "Minimum Average Maximum" ping_sin_qos.txt
```

Windows (PowerShell):

```powershell
ping -n 100 192.168.40.10 | Tee-Object -FilePath ping_sin_qos.txt
Select-String -Path ping_sin_qos.txt -Pattern "Minimum|Average|Maximum"
```

**Captura esperada sin QoS:**

Linux:

```
round-trip min/avg/max/stddev = 5.123/85.456/234.567/45.234 ms
```

Windows:

```
Paquetes: enviados = 100, recibidos = 100, perdidos = 0 (0% perdidos),
Tiempos aproximados de ida y vuelta en milisegundos:
    Mínimo = 5ms, Máximo = 234ms, Media = 85ms
```

*Nota: El RTT será alto debido a congestión*

### Métrica 2: Jitter

**Del output de iperf3 UDP:**
```
[ ID] Interval           Transfer     Bandwidth       Jitter    Lost/Total Datagrams
[  5]   0.00-60.00  sec  60.3 MBytes  8.04 Mbits/sec  125.456 ms  12345/90000 (13.7%)
```

**Registrar:**
- Jitter: 125.456 ms (ALTO sin QoS)
- Pérdida: 13.7% (SIGNIFICATIVA)

### Métrica 3: Pérdida de Paquetes

**Del output de iperf3:**
```
Lost/Total Datagrams: 12345/90000 (13.7%)
```

## 4.4 Tabla de Mediciones Baseline (SIN QoS)

**Escenario: Tres flujos simultáneos sin QoS**

| Flujo | RTT promedio | Jitter | Pérdida | Throughput |
|-------|--------------|--------|--------|-----------|
| Gaming (UDP 5001) | 85 ms | 125 ms | 13.7% | ~1 Mbps |
| Streaming (UDP 5002) | 120 ms | 145 ms | 18.2% | ~7.5 Mbps |
| Descargas (TCP 5003) | 95 ms | 89 ms | 0.5% | ~6.2 Mbps |

*Estos son valores de ejemplo. Registrar los reales en tu laboratorio.*

---

## ✅ Verificación de Fase 4

**Completado cuando:**

- [ ] iperf3 servidor escuchando en puertos 5001, 5002, 5003
  - Linux: `netstat -tuln | grep 500`
  - Windows: `netstat -an | findstr "5001 5002 5003"`
- [ ] Prueba local exitosa (`iperf3 -c 127.0.0.1` en Windows o cliente contra servidor en Linux)
- [ ] Tráfico generado exitosamente en los tres clientes
- [ ] Mediciones de latencia, jitter y pérdida capturadas
- [ ] Tabla de baseline documentada
- [ ] Evidencia visual (capturas de pantalla) del tráfico

**Entregable**: Archivo "Mediciones_Sin_QoS.md" con:
- Outputs completos de iperf3
- Resultados de ping (`ping_sin_qos.txt`)
- Tabla de baseline
- Análisis inicial

---

# FASE 5: Configuración de QoS

**Duración**: Semanas 7-8  
**Objetivo**: Implementar marcado de tráfico, LLQ, CBWFQ, shaping y policing.

## 5.1 Clasificación y Marcado de Tráfico

### Definir ACLs en R1

```ios
configure terminal

! ACL para gaming
ip access-list extended ACL-GAME
 permit udp any any eq 5001

! ACL para streaming
ip access-list extended ACL-VIDEO
 permit udp any any eq 5002
 permit tcp any any eq 5002

! ACL para descargas
ip access-list extended ACL-BULK
 permit tcp any any eq 5003
```

### Definir Class-maps en R1

```ios
! Clase para gaming
class-map match-any CM-GAME
 match access-group name ACL-GAME

! Clase para streaming
class-map match-any CM-VIDEO
 match access-group name ACL-VIDEO

! Clase para descargas
class-map match-any CM-BULK
 match access-group name ACL-BULK
```

### Definir Policy-map de Marcado en R1

```ios
policy-map PM-MARKING
 class CM-GAME
  set dscp ef         ! EF = Expedited Forwarding (máxima prioridad)
 class CM-VIDEO
  set dscp af41       ! AF41 = Assured Forwarding Class 4, Drop Precedence 1
 class CM-BULK
  set dscp cs1        ! CS1 = Class Selector 1 (baja prioridad)

end
write memory
```

**Aplicar política de marcado en R1 (entrada):**

```ios
configure terminal

interface FastEthernet0/0
 description LAN CLIENTE
 service-policy input PM-MARKING

end
write memory
```

## 5.2 Configuración de Colas (LLQ + CBWFQ)

### Policy-map de Salida en R1

```ios
configure terminal

policy-map PM-WAN-CHILD
 class CM-GAME
  priority percent 20              ! LLQ: 20% garantizado + prioridad
 class CM-VIDEO
  bandwidth percent 30             ! CBWFQ: 30% garantizado
 class CM-BULK
  bandwidth percent 10             ! CBWFQ: 10% garantizado
 class class-default
  fair-queue                       ! El resto comparte equitativamente

end
write memory
```

### Policy-map con Shaping (Padre)

```ios
configure terminal

policy-map PM-PARENT-SHAPE
 class class-default
  shape average 8000000           ! Limitar a 8Mbps total
  service-policy PM-WAN-CHILD     ! Aplicar child policy

end
write memory
```

**Aplicar política de salida en R1:**

```ios
configure terminal

interface Serial0/0
 description ENLACE A R2
 service-policy output PM-PARENT-SHAPE

end
write memory
```

## 5.3 Policing para Limitar Descargas

```ios
configure terminal

policy-map PM-POLICE-BULK
 class CM-BULK
  police 2000000 conform-action transmit exceed-action drop

end
write memory
```

*Aplicación opcional en entrada de R1 si deseas un control más estricto*

## 5.4 Verificación de Configuración QoS

### Ver policies configuradas

```ios
show policy-map
```

**Esperado:**
```
Policy Map PM-MARKING
  Class CM-GAME
    set dscp ef
  Class CM-VIDEO
    set dscp af41
  Class CM-BULK
    set dscp cs1
...
```

### Ver aplicación en interfaces

```ios
show policy-map interface Serial0/0
```

**Esperado:**
```
Service-policy output: PM-PARENT-SHAPE

  Class-map: class-default (match-any)
    Queue stats
    Output queue (bytes/packets/drops)
    Shape (average) 8000000 bps
      Applied 60000 bytes in 0.156 sec(s)
      
    Service-policy: PM-WAN-CHILD
      Class-map: CM-GAME (match-any)
        Priority (%)  20 (bps) 1600000
```

---

## ✅ Verificación de Fase 5

**Completado cuando:**

- [ ] ACLs creadas para identificar los tres tipos de tráfico
- [ ] Class-maps asociadas a las ACLs
- [ ] Policy-map de marcado aplicada en interfaz FastEthernet0/0 (entrada)
- [ ] Policy-map de colas aplicada en interfaz Serial0/0 (salida)
- [ ] Shaping configurado a 8Mbps
- [ ] `show policy-map interface` muestra las políticas activas

**Entregable**: Archivo "Config_QoS.md" con:
- Todas las configuraciones de QoS
- Capturas de `show policy-map` commands
- Diagrama de flujo de tráfico con QoS

---

# FASE 6: Validación y Análisis Comparativo

**Duración**: Semanas 9-10  
**Objetivo**: Medir el impacto de QoS y comparar resultados.

## 6.1 Repetir Pruebas CON QoS

### Prueba 1: Línea Base - Tráfico Individual (CON QoS)

**Gaming (UDP) - desde PC_Gamer:**
```bash
iperf3 -u -c 192.168.40.10 -p 5001 -b 1M -l 120 -t 30
```

**Captura:** Los mismos puntos que fase anterior

### Prueba 2: Congestión Simultánea (CON QoS)

**Terminal en PC_Gamer:**
```bash
iperf3 -u -c 192.168.40.10 -p 5001 -b 1M -l 120 -t 60 &
```

**Terminal en PC_Streaming:**
```bash
iperf3 -u -c 192.168.40.10 -p 5002 -b 8M -l 1400 -t 60 &
```

**Terminal en PC_Descargas:**
```bash
iperf3 -c 192.168.40.10 -p 5003 -P 4 -t 60 &
```

### Medición de Métricas CON QoS

```bash
# En PC_Gamer (mientras corre iperf3 CON QoS)
ping -c 100 192.168.40.10 > ping_con_qos.txt

grep min ping_con_qos.txt
```

## 6.2 Tabla Comparativa: Antes vs Después de QoS

### Comparación Global

| Métrica | Sin QoS | Con QoS | Mejora |
|---------|---------|---------|--------|
| RTT Gaming (promedio) | 85 ms | 25 ms | ↓ 70.6% |
| Jitter Gaming | 125 ms | 15 ms | ↓ 88% |
| Pérdida Gaming | 13.7% | 0.1% | ↓ 99.3% |
| RTT Streaming | 120 ms | 45 ms | ↓ 62.5% |
| Pérdida Streaming | 18.2% | 0.3% | ↓ 98.4% |
| RTT Descargas | 95 ms | 90 ms | ↓ 5.3% |

*Estos son valores esperados. Registrar los reales.*

### Análisis por Flujo

**Gaming (Tráfico Crítico):**

| Métrica | Sin QoS | Con QoS | Estado |
|---------|---------|---------|--------|
| RTT promedio | 85 ms | 25 ms | ✅ Mejora significativa |
| Jitter | 125 ms | 15 ms | ✅ Mucho más estable |
| Pérdida | 13.7% | 0.1% | ✅ Casi cero pérdida |
| Throughput | ~1 Mbps | ~1 Mbps | ✅ Mantiene garantía |

**Conclusión**: El tráfico de gaming es protegido por LLQ (priority) y experimenta latencia de cola mínima.

---

**Streaming (Tráfico Intermedio):**

| Métrica | Sin QoS | Con QoS | Estado |
|---------|---------|---------|--------|
| RTT promedio | 120 ms | 45 ms | ✅ Mejora |
| Jitter | 145 ms | 35 ms | ✅ Más predecible |
| Pérdida | 18.2% | 0.3% | ✅ Casi eliminada |
| Throughput | ~7.5 Mbps | ~7.3 Mbps | ✅ Mantiene 30% reservado |

**Conclusión**: CBWFQ garantiza 30% de ancho aunque compita con otros flujos.

---

**Descargas (Tráfico Best Effort):**

| Métrica | Sin QoS | Con QoS | Estado |
|---------|---------|---------|--------|
| RTT promedio | 95 ms | 90 ms | ~ Sin cambio |
| Jitter | 89 ms | 88 ms | ~ Sin cambio |
| Pérdida | 0.5% | 0.5% | ~ Sin cambio |
| Throughput | ~6.2 Mbps | ~1.5 Mbps | ↓ Limitado a 10% |

**Conclusión**: Las descargas son relegadas a best effort (10%), cediendo ancho a tráfico prioritario.

## 6.3 Análisis Cualitativo

### Observaciones sin QoS

1. Todos los flujos compiten por el mismo espacio en la cola FIFO
2. Las descargas TCP, al ser más agresivas, llenan la cola
3. Los paquetes UDP del juego sufren esperas aleatorias
4. El jitter es alto y variable

### Observaciones con QoS

1. El tráfico de gaming es procesado en una cola de baja latencia (priority queue)
2. Streaming y descargas comparten el resto, pero con garantías mínimas
3. El jitter se vuelve más predecible
4. Las pérdidas para gaming se reducen drásticamente

---

## ✅ Verificación de Fase 6

**Completado cuando:**

- [ ] Pruebas iperf3 ejecutadas CON QoS exitosamente
- [ ] Mediciones de latencia, jitter y pérdida capturadas
- [ ] Tabla comparativa "Antes vs Después" completada
- [ ] Análisis por flujo documentado
- [ ] Gráficos visuales de mejora

**Entregable**: Archivo "Mediciones_Con_QoS.md" y "Analisis_Comparativo.md" con:
- Outputs completos de iperf3 CON QoS
- Tabla comparativa
- Análisis cualitativo
- Gráficos de RTT, jitter y pérdida antes/después

---

# FASE 7: Documentación Final

**Duración**: Semanas 11-12  
**Objetivo**: Consolidar todo el trabajo en un reporte final.

## 7.1 Contenido del Reporte Final

### Capítulo 1: Introducción

- Problema: Latencia en aplicaciones interactivas bajo congestión
- Solución: Selección de rutas + QoS
- Objetivos del proyecto
- Alcance

### Capítulo 2: Fundamentos Teóricos

- OSPF y selección de rutas
- QoS: conceptos de clase, prioridad, garantías
- LLQ (Low Latency Queuing)
- CBWFQ (Class-Based Weighted Fair Queuing)
- Shaping y Policing
- DSCP (Differentiated Services Code Point)

### Capítulo 3: Diseño de la Solución

- Topología de red
- Plan de direccionamiento IP
- Parámetros de latencia (delays y costos OSPF)
- Tipos de tráfico a simular

### Capítulo 4: Implementación

- Configuración de OSPF
- Configuración de QoS
- Paso a paso de montaje en GNS3

### Capítulo 5: Resultados

- Tabla comparativa sin/con QoS
- Análisis por flujo
- Interpretación de mejoras

### Capítulo 6: Conclusiones

- Validación de objetivos
- Lecciones aprendidas
- Recomendaciones futuras

### Anexo A: Configuraciones Completas

- Texto de configuración de cada router
- Comandos de verificación

### Anexo B: Evidencia Técnica

- Capturas de `show` commands
- Outputs de iperf3
- Gráficos de latencia

---

## 7.2 Formato de Presentación

**Documento principal**: "Proyecto_QoS_Gaming_Reporte_Final.md" o PDF

**Estructura:**

```
├── Resumen Ejecutivo
├── 1. Introducción
├── 2. Fundamentos Teóricos
├── 3. Diseño
├── 4. Implementación
├── 5. Resultados
├── 6. Conclusiones
├── 7. Referencias
├── Anexo A: Configuraciones
└── Anexo B: Evidencia
```

---

## ✅ Verificación de Fase 7

**Completado cuando:**

- [ ] Reporte final de 40+ páginas
- [ ] Todos los capítulos completados
- [ ] Tablas y gráficos incluidos
- [ ] Configuraciones documentadas en anexos
- [ ] Evidencia técnica (capturas) incluida
- [ ] Revisado y sin errores ortográficos

**Entregable**: Archivo "Proyecto_QoS_Gaming_Reporte_Final.pdf"

---

# 📊 Tabla de Control General del Proyecto

| Fase | Duración | Entregable | Estado |
|------|----------|-----------|--------|
| 1. Planificación y Diseño | Sem 1-2 | Diseño_Fase1.md | ⬜ |
| 2. Implementación GNS3 | Sem 3-4 | Proyecto.gns3 + Config_IPs.md | ⬜ |
| 3. OSPF y Enrutamiento | Sem 3-4 | Config_OSPF.md + Verificaciones | ⬜ |
| 4. Simulación Sin QoS | Sem 5-6 | Mediciones_Sin_QoS.md | ⬜ |
| 5. Configuración QoS | Sem 7-8 | Config_QoS.md | ⬜ |
| 6. Validación Con QoS | Sem 9-10 | Mediciones_Con_QoS.md + Analisis_Comparativo.md | ⬜ |
| 7. Documentación Final | Sem 11-12 | Proyecto_QoS_Gaming_Reporte_Final.pdf | ⬜ |

---

# 🎯 Checklist Final

Antes de entregar el proyecto, verifica:

- [ ] Topología en GNS3 funcional y guardada
- [ ] Todos los routers con OSPF correctamente configurado
- [ ] Ruta A preferida sobre ruta B (verificable con `show ip route`)
- [ ] iperf3 ejecutándose sin errores en clientes y servidor
- [ ] Mediciones sin QoS capturadas completamente
- [ ] Mediciones con QoS capturadas completamente
- [ ] Tabla comparativa muestra mejoras cuantificables
- [ ] Reporte final coherente y bien estructurado
- [ ] Configuraciones en anexo claras y reproducibles
- [ ] Todas las figuras y gráficos incluidos
- [ ] Sin errores ortográficos

---

**Proyecto completado**: 🎉 Listo para sustentación

**Duración total**: 12 semanas  
**Esfuerzo estimado**: 60-80 horas
