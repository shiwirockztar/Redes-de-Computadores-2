Conexiones físicas:

LAN A 172.16.0.1/24 -> R7 (eth0 ->Fa0/0)
R7 -> R6 (Fa0/1 ->Fa0/0)
R6 -> R1 (Fa0/1 ->Fa0/0)
R1 -> R4 (Fa0/1 ->Fa0/0)
R4 -> R5 (Fa0/1 ->Fa0/0)
R5 -> R2 (Fa0/1 ->Fa0/0)
R2 -> R3 (Fa0/1 ->Fa0/0)
R3 -> LAN B 10.10.10.1/24 (Fa0/1 ->eth0)
R3 -> Cloud1_internet (Fa1/0 ->eth2)


| De                    | Interface | A                     | Interface | Tipo     |
| --------------------- | --------- | --------------------- | --------- | -------- |
| LAN A (172.16.0.1/24) | eth0      | R7                    | Fa0/0     | Ethernet |
| R7                    | Fa0/1     | R6                    | Fa0/0     | Ethernet |
| R6                    | Fa0/1     | R1                    | Fa0/0     | Ethernet |
| R1                    | Fa0/1     | R4                    | Fa0/0     | Ethernet |
| R4                    | Fa0/1     | R5                    | Fa0/0     | Ethernet |
| R5                    | Fa0/1     | R2                    | Fa0/0     | Ethernet |
| R2                    | Fa0/1     | R3                    | Fa0/0     | Ethernet |
| R3                    | Fa0/1     | LAN B (10.10.10.1/24) | eth0      | Ethernet |
| R3                    | Fa1/0     | Cloud1_internet       | eth2      | Ethernet |



La red interna a utilizar es 192.168.78.0/24, . Usted debe realizar el subnetting que considere adecuado. Se recomienda utilizar máscara fija. Debe garantizar un mínimo de 4 hosts por subred. La LAN A y la LAN B ya tienen direcciones asignadas. Se recomienda utilizar el enrutador 3745, con dos puertos Fast Ethernet adicionales y dos tarjetas wic-2T para los puertos serie. Las interfaces representadas con líneas punteadas son de tipo serial y el resto, de tipo Ethernet. Su topología representa un sistema autónomo que estará identificado con el número 65078.

Perfecto, con eso ya podemos estructurar bien el plan de subnetting + asignación lógica para tu práctica en GNS3.

🧠 Subnetting propuesto – Red 192.168.78.0/24 (máscara fija)

Requisito:

Mínimo 4 hosts por subred
Máscara fija (FLSM)
Red base: 192.168.78.0/24
📌 Elección de máscara

Para cumplir mínimo 4 hosts:

/30 → 2 hosts ❌ insuficiente
/29 → 6 hosts ✔ suficiente

👉 Usamos: 255.255.255.248 (/29)
👉 Cada subred tiene 6 hosts útiles


| Subred           | Rango de hosts | Broadcast |
| ---------------- | -------------- | --------- |
| 192.168.78.0/29  | .1 – .6        | .7        |
| 192.168.78.8/29  | .9 – .14       | .15       |
| 192.168.78.16/29 | .17 – .22      | .23       |
| 192.168.78.24/29 | .25 – .30      | .31       |
| 192.168.78.32/29 | .33 – .38      | .39       |
| 192.168.78.40/29 | .41 – .46      | .47       |
| 192.168.78.48/29 | .49 – .54      | .55       |
| 192.168.78.56/29 | .57 – .62      | .63       |
| ...              | ...            | ...       |

🌐 Asignación sugerida a tu topología

Voy a mapearlo directamente a tus enlaces:

🟢 LANs ya definidas
LAN A: 172.16.0.1/24 (no entra en subnetting pedido)
LAN B: 10.10.10.1/24 (no entra en subnetting pedido)
🔵 Red interna 192.168.78.0/24 (enlaces entre routers)

| Enlace  | Subred           | IP lado A | IP lado B |
| ------- | ---------------- | --------- | --------- |
| R7 ↔ R6 | 192.168.78.0/29  | R7 .1     | R6 .2     |
| R6 ↔ R1 | 192.168.78.8/29  | R6 .9     | R1 .10    |
| R1 ↔ R4 | 192.168.78.16/29 | R1 .17    | R4 .18    |
| R4 ↔ R5 | 192.168.78.24/29 | R4 .25    | R5 .26    |
| R5 ↔ R2 | 192.168.78.32/29 | R5 .33    | R2 .34    |
| R2 ↔ R3 | 192.168.78.40/29 | R2 .41    | R3 .42    |


🌍 Enlace a Internet (fuera del /24 interno)
R3 ↔ Cloud1 (Internet)
Este enlace normalmente NO usa el /24 interno
Se deja como:
DHCP de cloud o IP pública simulada


Se describio en la topologia dos circulos protocolo 1 y protocolo 2, el circulo protocolo 1 abarca R1 y R4, el circulo protocolo 2 abarca R2 y R3
se le solicita lo siguiente: 
1. Configurar 2 IGP en su topología. debe configurar IS-IS en alguna de las dos partes
de la topología. El otro IGP es de libre elección .
2. El router R1 no ejecuta ningún protocolo de enrutamiento. Sin embargo, la
conectividad a la LAN A debe estar garantizada.


Perfecto, aquí ya estás entrando en la parte “importante” de la práctica (y donde normalmente los profes empiezan a ver si entendiste o solo conectaste cables 😄).

Te organizo la solución clara y aplicable en GNS3.

🧠 Diseño de enrutamiento solicitado

Tienes dos “círculos”:

🔵 Círculo Protocolo 1 → R1 y R4
🔴 Círculo Protocolo 2 → R2 y R3

Y además:

Debes usar 2 IGP diferentes
Uno debe ser IS-IS
El otro es libre
R1 NO ejecuta routing
Pero debe tener conectividad a LAN A
🧩 Elección recomendada (simple y típica de laboratorio)
🔵 Círculo 1 (R1–R4): IS-IS
🔴 Círculo 2 (R2–R3): OSPF

👉 Esta combinación es la más usada en prácticas porque:

IS-IS funciona bien en backbone simple
OSPF es fácil de verificar
⚠️ Punto crítico del ejercicio
🚫 R1 no puede rutear

Pero:

R1 conecta LAN A
Debe tener salida hacia el resto de la red

👉 Solución estándar:

✔️ Ruta estática en R1
🧭 SOLUCIÓN DE ENRUTAMIENTO
🔵 PROTOCOLO 1: IS-IS (R1 – R4)
Routers en IS-IS:
R1 ❌ NO participa
R4 ✔ participa (pero ojo: R1 no corre routing)
En la práctica, normalmente:
👉 IS-IS corre en R4 y hacia el core

📌 Mejor interpretación correcta de práctica:

IS-IS se ejecuta en el “backbone”
R1 solo usa rutas estáticas

👉 Entonces ajustamos correctamente:

✔ IS-IS en:
R4
R5
R6
R7 (si lo quieres incluir en backbone)

Pero como te dijeron círculo R1–R4:
👉 interpretación típica del profe:

IS-IS solo dentro del segmento R4 ↔ R5 ↔ R6 ↔ R7
R1 queda fuera
🔴 PROTOCOLO 2: OSPF (R2 – R3)
OSPF en:
R2
R3

Área sugerida:

área 0 (simple)
🌐 R1 (CASO ESPECIAL)
❗ R1 NO corre routing

Pero debe llegar a la red interna.

✔ Solución:

En R1 configuras:

ip route 0.0.0.0 0.0.0.0 192.168.78.9

👉 (next-hop = R6 o vecino directo según tu subnetting)

Y en R6 (o router vecino):

Debe existir ruta de retorno hacia LAN A:

ip route 172.16.0.0 255.255.255.0 192.168.78.10
🧠 RESUMEN FINAL
🔵 IS-IS
Backbone: R4–R5–R6–R7
Transporta red 192.168.78.0/24
🔴 OSPF
R2 ↔ R3
Intercambio de LAN B
⚪ R1 (especial)
No routing protocol
Solo rutas estáticas
Mantiene LAN A activa

te dejo la configuración IOS completa por router, lista para copiar en GNS3 y cumplir exactamente lo que te piden:

IS-IS en el círculo izquierdo (R6–R7–R4–R5 como backbone)
OSPF en el círculo derecho (R2–R3)
R1 SIN protocolo de enrutamiento
R1 con ruta estática para LAN A
Conectividad garantizada extremo a extremo
🧠 Suposiciones de IP (según tu subnetting /29)
Enlaces internos:
R7–R6: 192.168.78.0/29 → R7 .1 / R6 .2
R6–R1: 192.168.78.8/29 → R6 .9 / R1 .10
R1–R4: 192.168.78.16/29 → R1 .17 / R4 .18
R4–R5: 192.168.78.24/29 → R4 .25 / R5 .26
R5–R2: 192.168.78.32/29 → R5 .33 / R2 .34
R2–R3: 192.168.78.40/29 → R2 .41 / R3 .42


R1 (SIN routing protocol)
conf t

interface fa0/0
 ip address 192.168.78.10 255.255.255.248
 no shutdown

interface fa0/1
 ip address 192.168.78.17 255.255.255.248
 no shutdown

! Ruta estática hacia toda la red interna
ip route 0.0.0.0 0.0.0.0 192.168.78.9

end
wr

R6 (IS-IS)
conf t

interface fa0/0
 ip address 192.168.78.2 255.255.255.248
 no shutdown

interface fa0/1
 ip address 192.168.78.9 255.255.255.248
 no shutdown

router isis CORE
 net 49.0001.0000.0000.0006.00
 is-type level-2-only

interface fa0/0
 ip router isis CORE

interface fa0/1
 ip router isis CORE

end
wr

R7 (IS-IS)

conf t

interface fa0/0
 ip address 172.16.0.2 255.255.255.0
 no shutdown

interface fa0/1
 ip address 192.168.78.1 255.255.255.248
 no shutdown

router isis CORE
 net 49.0001.0000.0000.0007.00
 is-type level-2-only

interface fa0/1
 ip router isis CORE

end
wr

R4 (IS-IS backbone + frontera)

conf t

interface fa0/0
 ip address 192.168.78.18 255.255.255.248
 no shutdown

interface fa0/1
 ip address 192.168.78.25 255.255.255.248
 no shutdown

router isis CORE
 net 49.0001.0000.0000.0004.00
 is-type level-2-only

interface fa0/0
 ip router isis CORE
interface fa0/1
 ip router isis CORE

end
wr

R5 (IS-IS backbone + frontera OSPF)

conf t

interface fa0/0
 ip address 192.168.78.26 255.255.255.248
 no shutdown

interface fa0/1
 ip address 192.168.78.33 255.255.255.248
 no shutdown

router isis CORE
 net 49.0001.0000.0000.0005.00
 is-type level-2-only

interface fa0/0
 ip router isis CORE

end
wr

R2 (OSPF)

conf t

interface fa0/0
 ip address 192.168.78.34 255.255.255.248
 no shutdown

interface fa0/1
 ip address 192.168.78.41 255.255.255.248
 no shutdown

router ospf 1
 router-id 2.2.2.2
 network 192.168.78.32 0.0.0.7 area 0
 network 192.168.78.40 0.0.0.7 area 0

end
wr

R3 (OSPF + LAN B + Internet)

conf t

interface fa0/0
 ip address 192.168.78.42 255.255.255.248
 no shutdown

interface fa0/1
 ip address 10.10.10.1 255.255.255.0
 no shutdown

interface fa1/0
 ip address dhcp
 no shutdown

router ospf 1
 router-id 3.3.3.3
 network 192.168.78.40 0.0.0.7 area 0
 network 10.10.10.0 0.0.0.255 area 0

end
wr

🧪 VERIFICACIÓN RÁPIDA

En cada router:

show ip route
show ip ospf neighbor
show isis neighbors
ping 10.10.10.1
ping 172.16.0.1

