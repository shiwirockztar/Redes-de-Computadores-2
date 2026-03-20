🧩 🔷 Nueva topología

Ahora tienes:

- Switch1 → PC1, PC2, PC3, PC6, Admin

- Switch2 → PC4, PC5

- Switch3 → PC7, PC8

Conexiones:

- S1 ↔ S2 (trunk)

- S2 ↔ S3 (trunk)

---

🅰️ (a) Nueva VLAN
📌 Definiciones

VLAN 10 → PC1, PC2, PC4

VLAN 20 → PC3, PC5

VLAN 30 → Admin + PC7 (administración)

VLAN 40 → PC6 y PC8 (nueva VLAN)

---

🔧 Configuración clave
🟦 En TODOS los switches

```bash
enable
conf t
vlan 10
vlan 20
vlan 30
vlan 40
```

---

🟦 Switch1 (agregar PC6)

```bash
enable
conf t
interface fa0/4
switchport mode access
switchport access vlan 40
copy running-config startup-config
```

---

🟦 Switch3

```bash
enable
conf t
interface fa0/1   ← PC7
switchport mode access
switchport access vlan 30

interface fa0/2   ← PC8
switchport mode access
switchport access vlan 40
```

---

🅱️ (b) VLAN nativa y tráfico sin etiqueta
📘 ¿Qué pasa con tráfico SIN etiqueta?

En un trunk:

Si una trama NO tiene etiqueta (untagged)
👉 El switch la asigna a la VLAN nativa

---

📌 VLAN nativa

- Es la VLAN que viaja SIN etiqueta en un trunk

- Por defecto es VLAN 1

- Se puede cambiar

---

| Tipo                | Función                      |
| ------------------- | ---------------------------- |
| VLAN nativa         | Transporte sin etiqueta      |
| VLAN administración | Gestión remota (Telnet, SSH) |

👉 NO son lo mismo

---

🔧 Configurar VLAN 40 como nativa

En TODOS los trunks:

```bash
enable
conf t
interface fa0/24
switchport mode trunk
switchport trunk native vlan 40
```

(Y también el enlace S2 ↔ S3)

---

🅲 (c) Permitir solo VLAN necesarias

Sí se puede (y es buena práctica 🔥)

```bash
enable
conf t
interface fa0/24
switchport trunk allowed vlan 10,20,30,40
exit
end
copy running-config startup-config 
```

🟦 Switch2

```bash
enable
conf t
interface range fastEthernet 0/23-24
switchport trunk allowed vlan 10,20,30,40
exit
end
copy running-config startup-config 
```
---

🅳 (d) Conectividad + Telnet
🔑 Paso 1: Crear SVI (IP de administración)

Ejemplo Switch1

```bash
enable
conf t
interface vlan 30
ip address 192.18.3.1 255.255.255.0
exit
end
copy running-config startup-config
```

```bash
enable
conf t
interface vlan 30
ip address 192.18.3.1 255.255.255.0
no shutdown
```

Switch2:

```bash
enable
conf t
interface vlan 30
ip address 192.18.3.2 255.255.255.0
no shutdown
```

Switch3:

```bash
enable
conf t
interface vlan 30
ip address 192.18.3.3 255.255.255.0
no shutdown
```

---

🔑 Paso 2: Configurar Telnet

```bash
enable
conf t
line vty 0 4
password cisco
login
transport input telnet
```

---

🔑 Paso 3: IPs

- Admin → 192.18.3.10

- PC7 → 192.18.3.7

---

✅ Resultado

✔ PC7 y Admin pueden hacer Telnet a todos los switches
✔ VLANs tienen conectividad interna

---

🅴 (e) Ping PC1 → SVI Switch3
📡 Camino

PC1 (VLAN 10)

Switch1

Trunk S1 → S2 (con etiqueta VLAN 10)

Switch2

Trunk S2 → S3 (con etiqueta VLAN 10)

Switch3 → SVI VLAN 30 ❌

---

⚠️ IMPORTANTE

👉 NO funciona directamente

Porque:

PC1 está en VLAN 10

SVI está en VLAN 30

➡️ Necesitarías routing (inter-VLAN)

---

🅵 (f) Ping PC7 → SVI Switch1
📡 Camino correcto

1. PC7 (VLAN 30)
2. Switch3
3. Trunk S3 → S2 (tag VLAN 30)
4. Switch2
5. Trunk S2 → S1 (tag VLAN 30)
6. Switch1 → SVI VLAN 30

---

| Tramo           | Etiqueta        |
| --------------- | --------------- |
| PC7 → Switch3   | ❌ sin etiqueta |
| Switch3 → trunk | ✅ VLAN 30      |
| Trunk → Switch2 | ✅ VLAN 30      |
| Switch2 → trunk | ✅ VLAN 30      |
| Switch1 → SVI   | ❌ se elimina   |

---

🧠 Resumen clave

- PCs → tramas sin etiqueta

- Trunks → agregan etiqueta

- Switch destino → elimina etiqueta

---

🧩 CONCLUSIÓN GLOBAL

✔ VLAN 40 → ahora es nativa
✔ Trunks → transportan 10,20,30,40
✔ Administración → VLAN 30 con Telnet
✔ Sin routing → VLANs NO se comunican entre sí
