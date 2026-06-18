# FASE 5: Configuración de QoS

## 🧠 Objetivo de la implementación

La implementación de QoS (Quality of Service) tiene como objetivo **gestionar y priorizar el tráfico de red** para evitar que la congestión afecte a los servicios más sensibles como gaming y streaming.

---

## 1️⃣ Clasificación de tráfico (ACLs)

Se crean ACLs para identificar el tipo de tráfico:

- ACL-GAME → UDP puerto 5001  
- ACL-VIDEO → UDP/TCP puerto 5002  
- ACL-BULK → TCP puerto 5003  

Esto permite que el router **reconozca el tipo de tráfico desde su origen**.

---

## 2️⃣ Class-map (agrupación de tráfico)

Se asocian las ACLs a clases QoS:

- CM-GAME  
- CM-VIDEO  
- CM-BULK  

Esto convierte el tráfico identificado en **clases tratables por QoS**.

---

## 3️⃣ Policy-map de marcado (DSCP)

Se asignan marcas de prioridad:

- Gaming → EF  
- Streaming → AF41  
- Descargas → CS1  

El objetivo es que el paquete **lleve su prioridad dentro del header IP**.

---

## 4️⃣ Aplicación del marcado en entrada (Ingress QoS)

Se aplica la política en la interfaz de entrada:

```bash
service-policy input PM-MARKING

Esto permite que el tráfico sea clasificado y marcado desde que entra al router.

5️⃣ LLQ + CBWFQ (colas de salida)

Se definen colas de transmisión:

Gaming → PRIORITY (LLQ)
Streaming → CBWFQ 30%
Descargas → CBWFQ 10%
Default → Best effort

Esto controla cómo se transmite el tráfico cuando hay congestión.

6️⃣ Shaping (control de tasa global)
shape average 8000000

Limita la salida total a 8 Mbps.

Esto evita que la interfaz se sature y hace el tráfico más predecible y estable.

7️⃣ Policing (control de descargas)
police 2000000

Si el tráfico de descargas supera el límite, los paquetes se descartan.

Esto protege el resto del tráfico bajo alta carga.

📊 RESUMEN GENERAL DE LA FASE 5
🔴 SIN QoS (Fase 4)
FIFO sin control
TCP domina el ancho de banda
UDP pierde gran cantidad de paquetes
Alta latencia y jitter
Red congestionada y no determinística
🟢 CON QoS (Fase 5)
Tipo de tráfico	Mejora
Gaming	Baja latencia + prioridad alta
Streaming	Flujo estable y garantizado
Descargas	Limitadas para no afectar la red
Red general	Controlada y predecible
🎯 IDEA CLAVE

QoS transforma una red congestionada y no determinística en un sistema de priorización controlada donde el tráfico crítico (gaming) recibe tratamiento preferencial mediante LLQ, mientras que el tráfico no crítico (descargas) es limitado mediante policing y CBWFQ, estabilizando la latencia y reduciendo la pérdida de paquetes.
