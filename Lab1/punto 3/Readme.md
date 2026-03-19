🔹 Topología y VLANs

Según lo que te piden:

VLAN 10 → PCs 1, 2 y 4

VLAN 20 → PCs 3 y 5

VLAN 30 → PC Admin (gestión)

🅰️ (a) Configuración inicial (puertos en acceso)
🔧 Switch 1

```bash
enable
conf t

vlan 10
vlan 20
vlan 30

interface fa0/1
switchport mode access
switchport access vlan 10

interface fa0/2
switchport mode access
switchport access vlan 10

interface fa0/3
switchport mode access
switchport access vlan 20

interface fa0/10
switchport mode access
switchport access vlan 30

interface fa0/24
switchport mode access
switchport access vlan 1   ← (por defecto)
```

🔧 Switch 2

```bash
enable
conf t

vlan 10
vlan 20

interface fa0/1
switchport mode access
switchport access vlan 10

interface fa0/2
switchport mode access
switchport access vlan 20

interface fa0/24
switchport mode access
switchport access vlan 1
```

📡 Conectividad en este punto

✔ PC1 ↔ PC2 → ✅ Sí (misma VLAN 10 y mismo switch)
✔ PC3 solo → ❌ No tiene otro en VLAN 20 en su switch
✔ PC4 → ❌ No se comunica con PC1/2 (porque el enlace entre switches está en VLAN 1)
✔ PC5 ↔ PC3 → ❌ Tampoco (por misma razón)

---

❓ ¿A qué VLAN pertenece el enlace entre switches?

➡️ VLAN 1 (por defecto)
Porque el puerto fa0/24 está en modo access y no trunk.

---

🅱️ (b) Configurar enlace TRONCAL

```bash
interface fa0/24
switchport mode trunk
```

---

📘 Explicación

En un enlace trunk se usa el protocolo:

👉 IEEE 802.1Q (dot1q)

✔ Cada trama lleva una etiqueta VLAN (tag)
✔ Esta etiqueta incluye el ID de VLAN (VID)

Ejemplo:

VLAN 10 → tag = 10

VLAN 20 → tag = 20

---

🅲 (c) Ping PC1 → PC4 (explicación completa)
🧠 Paso 1: ARP

PC1 quiere comunicarse con 192.18.1.4

1. PC1 revisa su ARP → no conoce la MAC

2. Envía ARP Request (broadcast):

- VLAN 10

- “¿Quién tiene 192.18.1.4?”

---

🔄 Paso 2: Paso por Switch 1

- Entra sin etiqueta

- Switch le agrega tag VLAN 10 al salir por trunk

---

🔗 Paso 3: Enlace troncal

- La trama viaja con tag VLAN 10

---

🔄 Paso 4: Switch 2

- Recibe trama con tag VLAN 10

- Quita la etiqueta

- Reenvía solo a puertos VLAN 10

---

🖥️ Paso 5: PC4 responde

- Envía ARP Reply con su MAC

- El proceso inverso ocurre

---

📡 Paso 6: Ping (ICMP)

Ahora sí:

1. PC1 envía ICMP Echo Request

2. Switch agrega tag VLAN 10 en trunk

3. Switch2 lo entrega a PC4

4. PC4 responde con Echo Reply

---

| Segmento       | Etiqueta        |
| -------------- | --------------- |
| PC → Switch    | ❌ Sin etiqueta |
| Switch → Trunk | ✅ VLAN tag     |
| Trunk → Switch | ✅ VLAN tag     |
| Switch → PC    | ❌ Sin etiqueta |

---

🅳 (d) Tabla MAC (CAM)

Después de hacer ping:

---

| MAC | Puerto         |
| --- | -------------- |
| PC1 | fa0/1          |
| PC2 | fa0/2          |
| PC4 | fa0/24 (trunk) |

---

| MAC | Puerto         |
| --- | -------------- |
| PC4 | fa0/1          |
| PC1 | fa0/24 (trunk) |
| PC2 | fa0/24 (trunk) |

❓ ¿Qué MAC aparecen en el puerto trunk?

➡️ Todas las MAC de las VLAN remotas

Ejemplo:

En Switch 1 → MAC de PC4 aparece en fa0/24

En Switch 2 → MAC de PC1 y PC2 aparecen en fa0/24

✔ Porque ese puerto conecta múltiples VLANs

🧩 Concepto clave final

👉 Un puerto trunk no pertenece a una sola VLAN
👉 Transporta múltiples VLANs usando etiquetas 802.1Q
