# 🧠 FASE 5: CONFIGURACIÓN DE QoS (Explicación Paso a Paso)

## 1️⃣ Clasificación de tráfico (ACLs)

### 📌 Qué haces

Creas las siguientes ACLs:

- **ACL-GAME** → UDP puerto 5001
- **ACL-VIDEO** → UDP/TCP puerto 5002
- **ACL-BULK** → TCP puerto 5003

### 🧠 Para qué sirve

Este es el **etiquetado inicial** del tráfico.

El router ahora puede identificar:

- 🎮 Gaming
- 📺 Streaming
- 📥 Descargas

### ⚠️ Problema que soluciona

Antes de QoS:

- Todo el tráfico era tratado por igual.
- El router procesaba UDP y TCP sin diferenciación.

### 🚀 Mejora

Permite:

- Separar tráfico crítico (gaming).
- Identificar tráfico pesado (downloads).
- Aplicar reglas diferentes según el tipo de tráfico.

---

## 2️⃣ Class-Map (Clasificación real en el router)

### 📌 Qué haces

Asocias las ACLs a clases QoS:

- **CM-GAME**
- **CM-VIDEO**
- **CM-BULK**

### 🧠 Para qué sirve

Convierte listas de tráfico en clases QoS reales.

El router ahora crea grupos de tratamiento específicos para cada tipo de tráfico.

### 🚀 Mejora

- El tráfico ya no se trata manualmente por IP o puerto.
- Todo queda organizado por clases.
- Sirve como base para aplicar políticas QoS.

---

## 3️⃣ Policy-Map de marcado (DSCP)

### 📌 Qué haces

Asignas etiquetas DSCP:

| Tráfico | DSCP |
|----------|------|
| Gaming | EF |
| Streaming | AF41 |
| Descargas | CS1 |

### 🧠 Para qué sirve

Marca la prioridad directamente dentro de cada paquete.

### ⚠️ Problema que soluciona

Antes:

- El router debía inferir o adivinar la prioridad del tráfico.

Ahora:

- Cada paquete transporta explícitamente su nivel de prioridad.

### 🚀 Mejora real

Permite que:

- Todos los routers del camino entiendan la prioridad.
- El tráfico crítico reciba tratamiento preferencial.
- La prioridad sea independiente del origen del tráfico.

---

## 4️⃣ Aplicación del marcado en entrada (Ingress QoS)

### 📌 Qué haces

Aplicas la política:

```bash
service-policy input PM-MARKING
```

en la interfaz LAN de **R1**.

### 🧠 Para qué sirve

Es el punto donde el tráfico entra a la red y se clasifica.

### ⚠️ Problema que soluciona

Antes:

- El tráfico ingresaba sin control ni clasificación.

Ahora:

- Entra ya clasificado y etiquetado.

### 🚀 Mejora

- Evita que tráfico pesado o malicioso se mezcle con tráfico crítico.
- Establece el primer punto de control de congestión.

---

## 5️⃣ LLQ + CBWFQ (Colas de salida)

### 📌 Qué haces

Defines las siguientes colas:

| Clase | Tipo |
|---------|---------|
| Gaming | PRIORITY (LLQ) |
| Streaming | CBWFQ 30% |
| Downloads | CBWFQ 10% |
| Default | Best Effort |

### 🧠 Punto más importante del proyecto

Aquí es donde realmente cambia el rendimiento de la red.

### ⚠️ Problema sin QoS

En la Fase 4:

- TCP (descargas) saturaba las colas.
- UDP (gaming y streaming) sufría pérdidas masivas.

### 🚀 Mejora con QoS

#### 🎮 Gaming (LLQ)

- Siempre tiene prioridad.
- Cola de baja latencia.
- Jitter mínimo.

**Soluciona:**

- Lag.
- Pérdida crítica de paquetes UDP.

#### 📺 Streaming (CBWFQ 30%)

- Dispone de ancho de banda garantizado.

**Soluciona:**

- Buffering excesivo.
- Pérdidas masivas de paquetes.

#### 📥 Descargas (CBWFQ 10%)

- Se limitan intencionalmente.

**Evita:**

- Saturación de red.
- Que TCP afecte al resto de aplicaciones.

---

## 6️⃣ Shaping (Control de tasa global)

### 📌 Qué haces

```bash
shape average 8000000
```

### 🧠 Para qué sirve

Limita toda la salida a **8 Mbps**.

### ⚠️ Problema que soluciona

Sin shaping:

- El tráfico puede saturar la interfaz física.
- Las colas se vuelven incontrolables.

### 🚀 Mejora

- Evita congestión repentina.
- Convierte la red en un flujo controlado.
- Hace que QoS sea predecible.

---

## 7️⃣ Policing (Control agresivo de descargas)

### 📌 Qué haces

```bash
police 2000000
```

### 🧠 Para qué sirve

Si las descargas exceden el límite configurado, los paquetes excedentes se descartan.

### 🚀 Mejora

- Evita que TCP consuma toda la red.
- Protege el tráfico crítico incluso bajo carga extrema.

---

# 📊 RESUMEN GENERAL DE LA FASE 5

## 🔴 Sin QoS (Fase 4)

- FIFO simple.
- TCP domina el enlace.
- UDP puede perder hasta el 90% de los paquetes.
- Jitter elevado.
- Red impredecible y caótica.

## 🟢 Con QoS (Fase 5)

| Elemento | Mejora |
|-----------|---------|
| Gaming | Baja latencia y baja pérdida |
| Streaming | Tráfico estable y garantizado |
| TCP Downloads | Limitado intencionalmente |
| Red | Controlada y predecible |

---

# 🎯 Idea clave para el informe

> QoS transforma una red congestionada y no determinística en un sistema de priorización controlada donde el tráfico crítico (gaming) recibe tratamiento preferencial mediante LLQ, mientras que el tráfico no crítico (descargas) es limitado mediante policing y CBWFQ, estabilizando la latencia y reduciendo la pérdida de paquetes.
