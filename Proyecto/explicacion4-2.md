# 📊 Interpretación de resultados iperf3 (SIN QoS)

Tus resultados muestran claramente el comportamiento de una red **congestión total sin QoS**, donde el tráfico compite en una cola FIFO sin priorización.

---

# 🟢 1. TRÁFICO GAMING (UDP - puerto 5015)

## 📌 Resultado observado
- Bitrate enviado: **500 Kbits/sec**
- Pérdida sender: **0%**
- Pérdida receiver: **92%**
- Jitter: ~**10 ms**

---

## ⚠️ Interpretación

- El **emisor cree que todo se envió correctamente**
- El **receptor pierde la mayoría de paquetes (92%)**
- Esto indica **descartes en routers intermedios (congestión)**

---

## 🧠 Explicación técnica

- UDP no retransmite
- Paquetes pequeños (80 bytes)
- En colas FIFO saturadas:
  - los paquetes se descartan sin control
  - no hay prioridad para tráfico sensible

---

## 🎮 Impacto en gaming

- Latencia variable (lag)
- pérdida de paquetes crítica
- experiencia de juego inutilizable

---

# 🟡 2. TRÁFICO STREAMING (UDP - 3 Mbps)

## 📌 Resultado observado
- Bitrate enviado: **3 Mbps**
- Bitrate recibido: **~567 Kbps**
- Pérdida receiver: **80%**
- Jitter: ~**7 ms**

---

## ⚠️ Interpretación

- El tráfico intenta enviar caudal constante
- Pero la red solo entrega una fracción del flujo

---

## 🧠 Explicación técnica

- UDP no se adapta a congestión
- Sigue transmitiendo aunque la red esté saturada
- La mayor parte del tráfico es descartado

---

## 📺 Impacto en streaming

- cortes constantes de video
- buffering extremo
- degradación severa de calidad

---

# 🔴 3. TRÁFICO DE DESCARGAS (TCP - 4 streams)

## 📌 Resultado observado
- Throughput total: **~454 Kbits/sec**
- Retransmisiones: **hasta 10 por flujo**
- Algunos flujos con **0 Bytes recibidos**

---

## ⚠️ Interpretación

- TCP detecta congestión y reduce ventana (cwnd)
- Se producen retransmisiones constantes
- Algunos flujos casi colapsan

---

## 🧠 Explicación técnica

- TCP se adapta a congestión:
  - reduce velocidad
  - entra en slow start repetidamente
- No elimina congestión, solo la reacciona

---

## 📥 Impacto en descargas

- velocidad muy baja
- inestabilidad en throughput
- flujos TCP desiguales

---

# 📊 RESUMEN GENERAL (SIN QoS)

| Tipo de tráfico | Estado | Problema principal |
|----------------|--------|-------------------|
| Gaming UDP | ❌ Crítico | 92% pérdida |
| Streaming UDP | ❌ Grave | 80% pérdida |
| TCP Downloads | ⚠️ Degradado | baja velocidad + retransmisiones |

---

# 🚨 CONCLUSIÓN TÉCNICA

Este escenario muestra una red:

- saturada
- sin priorización de tráfico
- con colas FIFO simples
- sin control de congestión a nivel de QoS

👉 Resultado: todos los servicios degradados simultáneamente

---

# 🎯 IMPORTANCIA PARA EL PROYECTO

Estos resultados sirven como **baseline (referencia SIN QoS)** y serán comparados contra:

- LLQ (gaming)
- CBWFQ (streaming)
- Best-effort limitado (descargas)

---

# 🚀 SIGUIENTE PASO

Cuando implementes QoS deberías observar:

- ↓ reducción de jitter en gaming
- ↓ pérdida de paquetes en UDP
- ↑ estabilidad general
- ↓ impacto de TCP sobre tráfico crítico
