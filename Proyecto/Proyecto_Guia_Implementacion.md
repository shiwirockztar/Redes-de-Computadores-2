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
| PC Gamer | VM Linux Cliente | Genera tráfico UDP sensible a latencia (puerto 5001) |
| PC Streaming | VM Linux Cliente | Genera tráfico UDP/TCP continuo (puerto 5002) |
| PC Descargas | VM Linux Cliente | Genera tráfico TCP pesado (puerto 5003) |
| R1 | Router de borde (cliente) | Entrada de la red, marca tráfico, aplica QoS |
| R2 | Router intermedio (ruta rápida) | Part of ruta A (costo 50 por enlace) |
| R3 | Router intermedio (ruta lenta) | Parte de ruta B (costo 200/150) |
| R4 | Router de borde (servidor) | Salida de la red |
| Servidor | VM Linux | Responde con iperf3 en puerto 5001, 5002, 5003 |

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
| R1 | G0/0 | 192.168.10.1 | 255.255.255.0 |
| R1 | S0/0/0 | 10.0.12.1 | 255.255.255.252 |
| R1 | S0/0/1 | 10.0.13.1 | 255.255.255.252 |
| R2 | S0/0/0 | 10.0.12.2 | 255.255.255.252 |
| R2 | S0/0/1 | 10.0.24.1 | 255.255.255.252 |
| R3 | S0/0/0 | 10.0.13.2 | 255.255.255.252 |
| R3 | S0/0/1 | 10.0.34.1 | 255.255.255.252 |
| R4 | S0/0/0 | 10.0.24.2 | 255.255.255.252 |
| R4 | S0/0/1 | 10.0.34.2 | 255.255.255.252 |
| R4 | G0/0 | 192.168.40.1 | 255.255.255.0 |

**Hosts clientes (VMs Linux):**

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
3. Ubuntu/Debian VMs preparadas (al menos 2GB RAM cada una)
4. iperf3 instalado en VMs Linux

### Instalación de iperf3 en VMs Linux

```bash
# En cada VM Ubuntu/Debian
sudo apt-get update
sudo apt-get install -y iperf3

# Verificar instalación
iperf3 --version
```

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

2. **VMs Linux**: Crea 4 VMs con Ubuntu/Debian
   - `PC_Gamer` (192.168.10.10)
   - `PC_Streaming` (192.168.10.20)
   - `PC_Descargas` (192.168.10.30)
   - `Servidor` (192.168.40.10)

3. **Switches**: Agregar 2 switches ethernet (simular LANs)
   - `SW1` (conecta cliente con R1)
   - `SW2` (conecta servidor con R4)

### Paso 3: Conectar componentes

**Conexiones físicas:**

| De | Interface | A | Interface | Tipo |
|----|-----------|---|-----------|------|
| R1 | G0/0 | SW1 | Port 1 | Ethernet |
| SW1 | Port 2 | PC_Gamer | eth0 | Ethernet |
| SW1 | Port 3 | PC_Streaming | eth0 | Ethernet |
| SW1 | Port 4 | PC_Descargas | eth0 | Ethernet |
| R1 | S0/0/0 | R2 | S0/0/0 | Serial (DTE/DCE) |
| R1 | S0/0/1 | R3 | S0/0/0 | Serial (DTE/DCE) |
| R2 | S0/0/1 | R4 | S0/0/0 | Serial (DTE/DCE) |
| R3 | S0/0/1 | R4 | S0/0/1 | Serial (DTE/DCE) |
| R4 | G0/0 | SW2 | Port 1 | Ethernet |
| SW2 | Port 2 | Servidor | eth0 | Ethernet |

## 2.3 Configuración de Direcciones IP

### Configurar VMs Linux

**En PC_Gamer:**
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses: [192.168.10.10/24]
      gateway4: 192.168.10.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

Repetir para PC_Streaming (192.168.10.20) y PC_Descargas (192.168.10.30) y Servidor (192.168.40.10)

Aplicar cambios:
```bash
sudo netplan apply
ip addr show eth0  # Verificar
```

### Configurar R1

```ios
enable
configure terminal

hostname R1
no ip domain-lookup

interface g0/0
 description LAN CLIENTE
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface s0/0/0
 description ENLACE A R2
 ip address 10.0.12.1 255.255.255.252
 clock rate 64000
 no shutdown

interface s0/0/1
 description ENLACE A R3
 ip address 10.0.13.1 255.255.255.252
 clock rate 64000
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

interface s0/0/0
 description ENLACE A R1
 ip address 10.0.12.2 255.255.255.252
 no shutdown

interface s0/0/1
 description ENLACE A R4
 ip address 10.0.24.1 255.255.255.252
 clock rate 64000
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

interface s0/0/0
 description ENLACE A R1
 ip address 10.0.13.2 255.255.255.252
 no shutdown

interface s0/0/1
 description ENLACE A R4
 ip address 10.0.34.1 255.255.255.252
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

interface s0/0/0
 description ENLACE A R2
 ip address 10.0.24.2 255.255.255.252
 no shutdown

interface s0/0/1
 description ENLACE A R3
 ip address 10.0.34.2 255.255.255.252
 no shutdown

interface g0/0
 description LAN SERVIDOR
 ip address 192.168.40.1 255.255.255.0
 no shutdown

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

- [ ] Topología montada en GNS3 con 4 routers y 4 VMs
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
interface s0/0/0
 delay 5000       ! 5ms en ruta A
 ip ospf cost 50  ! Costo bajo para ruta rápida

interface s0/0/1
 delay 20000      ! 20ms en ruta B
 ip ospf cost 200 ! Costo alto para ruta lenta

! Activar OSPF
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.13.0 0.0.0.3 area 0
 passive-interface g0/0  ! No envía OSPF por LAN cliente

end
write memory
```

### Configurar R2 con OSPF

```ios
configure terminal

interface s0/0/0
 delay 5000      ! Ruta A
 ip ospf cost 50

interface s0/0/1
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

interface s0/0/0
 delay 20000     ! Ruta B
 ip ospf cost 200

interface s0/0/1
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

interface s0/0/0
 delay 5000      ! Ruta A
 ip ospf cost 50

interface s0/0/1
 delay 15000     ! Ruta B
 ip ospf cost 150

router ospf 1
 network 10.0.24.0 0.0.0.3 area 0
 network 10.0.34.0 0.0.0.3 area 0
 network 192.168.40.0 0.0.0.255 area 0
 passive-interface g0/0  ! No envía OSPF por LAN servidor

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
10.0.12.2       0     FULL/  -        00:00:37    10.0.12.2       Serial0/0/0
10.0.13.2       0     FULL/  -        00:00:37    10.0.13.2       Serial0/0/1
```

### Verificar tabla de rutas OSPF

**En R1:**
```ios
show ip route ospf
```

**Esperado** (debe mostrar ruta a 192.168.40.0 a través de R2, costo total 100):
```
O    192.168.40.0/24 [110/150] via 10.0.12.2, 00:05:32, Serial0/0/0
```

*Nota: El costo calculado por OSPF depende del ancho de banda de la interfaz*

### Verificar cost en interfaces

```ios
show ip ospf interface s0/0/0
```

**Esperado**:
```
Serial0/0/0 is up, line protocol is up
  Internet Address 10.0.12.1/30, Area 0, Attached via Network Statement
  Process ID 1, Router ID 10.0.12.1, Network Type POINT_TO_POINT, Cost: 50
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
- [ ] Tabla de rutas muestra ruta A como preferida (costo 100)
- [ ] Ping exitoso desde PC_Gamer a Servidor
- [ ] Traceroute muestra la ruta correcta (R1 → R2 → R4)

**Entregable**: Outputs de `show` commands + capturas de pruebas de conectividad

---

# FASE 4: Simulación de Tráfico sin QoS

**Duración**: Semanas 5-6  
**Objetivo**: Generar tráfico real con iperf3 y medir métricas baseline (sin QoS).

## 4.1 Configuración del Servidor iperf3

### En la VM Servidor (192.168.40.10)

```bash
# Iniciar iperf3 en modo servidor (daemon)
iperf3 -s -D

# Verificar que está escuchando
netstat -tuln | grep iperf
ps aux | grep iperf3
```

## 4.2 Generación de Tráfico de Prueba

### Prueba 1: Línea Base - Tráfico Individual

**Gaming (UDP) - desde PC_Gamer:**
```bash
iperf3 -u -c 192.168.40.10 -p 5001 -b 1M -l 120 -t 30
```

**Captura:**
- Copiar salida completa (throughput, jitter, pérdida)
- Registrar RTT con ping simultáneo

**Streaming (UDP) - desde PC_Streaming:**
```bash
iperf3 -u -c 192.168.40.10 -p 5002 -b 8M -l 1400 -t 30
```

**Descargas (TCP) - desde PC_Descargas:**
```bash
iperf3 -c 192.168.40.10 -p 5003 -P 4 -t 30
```

### Prueba 2: Congestión Simultánea (SIN QoS)

Ejecutar los tres tipos de tráfico en paralelo para congestionar la red:

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

**Esperar a que terminen:**
```bash
wait
```

## 4.3 Medición de Métricas Baseline

### Métrica 1: Latencia (RTT)

**Ejecutar durante el tráfico de congestión:**

```bash
# En PC_Gamer (mientras corre iperf3)
ping -c 100 192.168.40.10 > ping_sin_qos.txt

# Analizar resultados
grep min ping_sin_qos.txt
```

**Captura esperada sin QoS:**
```
round-trip min/avg/max/stddev = 5.123/85.456/234.567/45.234 ms
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
- [ ] Tráfico generado exitosamente en los tres clientes
- [ ] Mediciones de latencia, jitter y pérdida capturadas
- [ ] Tabla de baseline documentada
- [ ] Evidencia visual (capturas de pantalla) del tráfico

**Entregable**: Archivo "Mediciones_Sin_QoS.md" con:
- Outputs completos de iperf3
- Resultados de ping
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

interface g0/0
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

interface s0/0/0
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
show policy-map interface s0/0/0
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
- [ ] Policy-map de marcado aplicada en interfaz G0/0 (entrada)
- [ ] Policy-map de colas aplicada en interfaz S0/0/0 (salida)
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
