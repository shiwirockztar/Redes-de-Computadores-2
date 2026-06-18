🧠 FASE 5: CONFIGURACIÓN DE QoS (explicación por pasos)
1️⃣ Clasificación de tráfico (ACLs)
📌 Qué haces

Creas ACLs:

ACL-GAME → UDP puerto 5001
ACL-VIDEO → UDP/TCP puerto 5002
ACL-BULK → TCP puerto 5003
🧠 Para qué sirve

Esto es el “etiquetado inicial” lógico del tráfico.

👉 El router ahora puede decir:

esto es gaming
esto es streaming
esto es descarga
⚠️ Problema que soluciona

Antes de QoS:

Todo el tráfico era “igual”
Router trataba UDP y TCP igual
🚀 Mejora

Permite:

separar tráfico crítico (gaming)
identificar tráfico pesado (downloads)
aplicar reglas diferentes por tipo
2️⃣ Class-map (clasificación real en el router)
📌 Qué haces

Asocias ACLs a clases:

CM-GAME
CM-VIDEO
CM-BULK
🧠 Para qué sirve

Convierte “listas de tráfico” en clases QoS reales.

👉 El router ahora crea “grupos de tratamiento”.

🚀 Mejora

Ya no se trata tráfico por IP o puerto manualmente en cada decisión:

todo queda organizado por clases
base para aplicar políticas
3️⃣ Policy-map de marcado (DSCP)
📌 Qué haces

Asignas etiquetas DSCP:

Tráfico	DSCP
Gaming	EF
Streaming	AF41
Descargas	CS1
🧠 Para qué sirve

Esto es el punto clave:

👉 estás marcando la prioridad dentro del paquete

⚠️ Problema que soluciona

Antes:

el router tenía que “adivinar” prioridad

Ahora:

el paquete ya lleva prioridad escrita
🚀 Mejora real

Esto permite que:

todos los routers en el camino entiendan prioridad
el tráfico crítico no dependa de su origen
4️⃣ Aplicación del marcado en entrada (ingress QoS)
📌 Qué haces

Aplicas:

service-policy input PM-MARKING

en la interfaz LAN del R1

🧠 Para qué sirve

Es el punto donde:

👉 el tráfico entra a la red y se clasifica

⚠️ Problema que soluciona

Antes:

el tráfico entraba sin control

Ahora:

entra ya clasificado y etiquetado
🚀 Mejora
evita que tráfico malicioso o pesado “se mezcle”
primer punto de control de congestión
5️⃣ LLQ + CBWFQ (colas de salida)
📌 Qué haces

Defines colas:

Clase	Tipo
Gaming	PRIORITY (LLQ)
Streaming	CBWFQ 30%
Downloads	CBWFQ 10%
Default	Best effort
🧠 Esto es lo MÁS importante del proyecto

Aquí es donde realmente cambia el rendimiento.

⚠️ Problema sin QoS

En Fase 4:

TCP (descargas) saturaba la cola
UDP (gaming/streaming) sufría pérdida masiva
🚀 Mejora con QoS
🎮 Gaming (LLQ)
siempre primero
cola de baja latencia
casi cero jitter

👉 soluciona:

lag
pérdida UDP crítica
📺 Streaming (CBWFQ 30%)
tiene ancho garantizado

👉 soluciona:

buffering extremo
pérdida masiva UDP
📥 Descargas (10%)
limitado intencionalmente

👉 evita:

saturación de red
que TCP destruya el resto
6️⃣ Shaping (control de tasa global)
📌 Qué haces
shape average 8000000
🧠 Para qué sirve

👉 limita toda la salida a 8 Mbps

⚠️ Problema que soluciona

Sin shaping:

tráfico puede saturar interfaz física
colas se vuelven incontrolables
🚀 Mejora
evita congestión “brusca”
convierte la red en flujo controlado
hace QoS predecible
7️⃣ Policing (control agresivo de descargas)
📌 Qué haces
police 2000000
🧠 Para qué sirve

👉 si descargas exceden límite → se descartan

🚀 Mejora
evita que TCP consuma toda la red
protege tráfico crítico incluso bajo carga extrema
📊 RESUMEN GENERAL DE LA FASE 5
🔴 SIN QoS (tu Fase 4)
FIFO simple
TCP domina todo
UDP pierde hasta 90%
jitter alto
red caótica
🟢 CON QoS (Fase 5)
Elemento	Mejora
Gaming	baja latencia + baja pérdida
Streaming	estable + garantizado
TCP downloads	limitado intencionalmente
Red	controlada y predecible
🎯 IDEA CLAVE PARA TU INFORME

Puedes resumirlo así:

QoS transforma una red congestionada y no determinística en un sistema de priorización controlada donde el tráfico crítico (gaming) recibe tratamiento preferencial mediante LLQ, mientras que el tráfico no crítico (descargas) es limitado mediante policing y CBWFQ, estabilizando la latencia y reduciendo la pérdida de paquetes.
