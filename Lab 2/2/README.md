# 📘 Práctica STP + EtherChannel (Packet Tracer)

---

## 🔷 (a) Direccionamiento y conectividad

### 🌐 Direccionamiento

Una sola VLAN → misma red:

* PC1 (en SW3) → 192.168.1.1 /24
* PC2 (en SW1) → 192.168.1.2 /24
* PC3 (en SW2) → 192.168.1.3 /24

👉 Sin gateway

---

### 🔌 Configuración previa

Primero configuras las PCs, luego conectas switches.

---

### ❓ ¿Se puede transmitir inmediatamente?

❌ No

---

### 🔎 ¿Por qué hay que esperar?

Porque STP entra en proceso de convergencia:

* Elección del Root Bridge
* Cálculo de caminos
* Bloqueo de enlaces redundantes

Los puertos pasan por:

* Listening
* Learning
* Forwarding

⏱️ Esto tarda ~30 segundos

👉 Antes de eso, no hay tráfico estable

---

## 🌳 (b) Root Bridge

Según tu salida:

```
This bridge is the root
```

👉 SW2 es el Root Bridge

---

### 📌 Criterio de elección

STP elige el menor:

```
Bridge ID = Prioridad + MAC
```

* Prioridad: igual en todos (32768)
* Se decide por MAC más baja

👉 SW2 ganó por tener la MAC más baja

---

## 🔁 (c) Bucles y enlaces a deshabilitar

### 🔥 Tu topología

* 3 switches en triángulo
* 2 enlaces entre cada par

👉 Hay múltiples bucles

---

### 📊 Regla clave

Para N switches:

```
Enlaces activos = N - 1
```

👉 Con 3 switches:

* Necesitas 2 enlaces activos
* El resto → deben bloquearse

---

### 🔎 ¿Qué pasa en tu red?

* SW2 (root):

  * Todos sus puertos → Designated Forwarding ✅

* SW1 y SW3:

  * Tendrán:

    * 1 Root Port (FWD) hacia SW2
    * Puertos redundantes → Blocking 🚫

---

### ✔ ¿Cuántos enlaces se bloquean?

Como hay enlaces duplicados:

👉 STP bloqueará varios puertos redundantes
(aprox. 3 o más dependiendo del costo)

---

### 🎯 Justificación de roles

* Root Port → menor costo al root
* Designated Port → mejor camino en ese segmento
* Blocking → evita bucles

---

## 🌳 (d) Spanning Tree resultante

Como SW2 es root:

```
        SW2 (ROOT)
       /        \
     SW1        SW3
```

👉 El enlace SW1–SW3:

* parcialmente bloqueado

👉 Enlaces duplicados:

* uno activo, otro bloqueado

---

## 🔗 (e) EtherChannel

Agrupas los enlaces entre switches

Ejemplo SW1–SW2:

```
enable
configure terminal
interface range fa0/1 - 2
channel-group 1 mode active
exit
end
copy running-config startup-config
```

Repetir:

* SW1–SW3 → grupo 2
* SW2–SW3 → grupo 3

---

### 🔎 Verificación

```
show etherchannel summary
```

Ejemplo esperado:

```
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

Number of channel-groups in use: 1
Number of aggregators:           1

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------

1      Po1(SU)           LACP   Fa0/1(P) Fa0/2(P)
```

Debe aparecer:

👉 **SU → activo**

---

## 🌳 (f) STP con EtherChannel

Ahora STP ve:

👉 Cada grupo = un solo enlace lógico

---

### 📊 ¿Qué puertos forman el árbol?

* Port-channel hacia SW2 → activos
* Un camino alterno → puede bloquearse

---

### ✅ Ventajas de EtherChannel

✔ No bloquea enlaces individuales
✔ Usa todo el ancho de banda
✔ Redundancia real
✔ Menos puertos en blocking
✔ STP más simple

---

## 🔌 (g) Apagar un enlace físico

Si haces:

```
interface fa0/x
shutdown
```

---

### 🔎 ¿Qué pasa?

#### ✅ Con EtherChannel:

* El canal sigue activo
* Solo baja el ancho de banda
* STP NO cambia

---

#### ❌ Sin EtherChannel:

* STP detecta cambio
* Recalcula topología
* Puede desbloquear puertos

---

### 📌 ¿Por qué ocurre?

STP reacciona a:

* Cambios de topología
* Caída de enlaces

👉 Busca siempre mantener un árbol sin bucles

---

## ✅ Conclusión final

* SW2 = Root Bridge
* STP bloquea enlaces redundantes
* No hay transmisión inmediata
* EtherChannel mejora eficiencia
* STP se adapta a fallos automáticamente

---
