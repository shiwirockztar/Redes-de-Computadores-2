# 📘 Extensión de Topología STP (SW4 y SW5)

---

## 🔷 Descripción general

Se amplía la topología original agregando:

* 2 nuevos switches: **SW4 y SW5**
* 1 nuevo equipo: **PC4**

### 📡 Nueva estructura:

* SW4 conectado a SW3 (doble enlace)
* SW5 conectado a SW3 (doble enlace)
* SW5 conectado a SW2 (enlace simple)

---

## 🔌 1. Conexiones entre switches

### 🔹 SW1 ↔ SW2

* SW1 Fa0/1 ↔ SW2 Fa0/1
* SW1 Fa0/2 ↔ SW2 Fa0/2

---

### 🔹 SW1 ↔ SW3

* SW1 Fa0/3 ↔ SW3 Fa0/1
* SW1 Fa0/4 ↔ SW3 Fa0/2

---

### 🔹 SW2 ↔ SW3

* SW2 Fa0/3 ↔ SW3 Fa0/3
* SW2 Fa0/4 ↔ SW3 Fa0/4

---

### 🔹 SW3 ↔ SW4 (doble enlace)

* SW3 Fa0/5 ↔ SW4 Fa0/1
* SW3 Fa0/6 ↔ SW4 Fa0/2

---

### 🔹 SW3 ↔ SW5 (doble enlace)

* SW3 Fa0/7 ↔ SW5 Fa0/1
* SW3 Fa0/8 ↔ SW5 Fa0/2

---


### 🔹 SW4 ↔ SW5 (doble enlace)

* SW4 Fa0/3 ↔ SW5 Fa0/3
* SW4 Fa0/4 ↔ SW5 Fa0/4

---

### 🔹 SW5 ↔ SW2 (enlace simple)

* SW5 Fa0/5 ↔ SW2 Fa0/5

---

## 🖥️ 2. Conexión de PCs

### 🔹 PC1 → SW4

* PC1 FastEthernet0 ↔ SW4 Fa0/10

---

### 🔹 PC2 → SW1

* PC2 FastEthernet0 ↔ SW1 Fa0/10

---

### 🔹 PC3 → SW2

* PC3 FastEthernet0 ↔ SW2 Fa0/10

---

### 🔹 PC4 → SW5

* PC4 FastEthernet0 ↔ SW5 Fa0/10

---

## 🌐 3. Direccionamiento IP

Todos los equipos en la misma VLAN:

* PC1 → 192.168.1.1 /24
* PC2 → 192.168.1.2 /24
* PC3 → 192.168.1.3 /24
* PC4 → 192.168.1.4 /24

Máscara: 255.255.255.0
👉 Sin gateway

---

## 🌳 4. Análisis de STP

### 🔥 Características de la topología

* Múltiples enlaces redundantes
* Nuevos bucles generados

Ejemplos:

* SW3 ↔ SW5 ↔ SW2 ↔ SW3
* SW3 ↔ SW4 (doble enlace)
* SW3 ↔ SW5 (doble enlace)

---

### 📊 Comportamiento esperado

Si no se modifica la prioridad:

👉 **SW2 seguirá siendo el Root Bridge**

---

### 🔎 Roles esperados

* **SW2 (Root):**

  * Todos los puertos → Designated Forwarding

* **SW3:**

  * Root Port hacia SW2

* **SW4:**

  * Root Port hacia SW3

* **SW5:**

  * Puede elegir Root Port hacia:

    * SW2 o SW3 (según costo)

---

## 🔗 5. EtherChannel (recomendado)

Agrupar enlaces redundantes:

* SW1–SW2 → grupo 1
* SW1–SW3 → grupo 2
* SW2–SW3 → grupo 3
* SW3–SW4 → grupo 4
* SW3–SW5 → grupo 5

❌ No agrupar SW5–SW2 (solo 1 enlace)

---

## 🎯 6. Observaciones importantes

* Aumenta la cantidad de puertos en estado **blocking**
* STP deberá recalcular más caminos
* Se incrementa la complejidad de la red
* Ideal para analizar:

  * Reconvergencia
  * Redundancia
  * Selección de rutas

---

## ✅ Conclusión

* Se amplía la red manteniendo redundancia
* Se agregan nuevos caminos alternativos
* STP sigue evitando bucles automáticamente
* EtherChannel optimiza el uso de enlaces
* La topología es más robusta y tolerante a fallos

---
