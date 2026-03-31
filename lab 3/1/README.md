# Laboratorio 3 - Punto 1

🧠 📌 Contexto clave del ejercicio PC1 → VLAN 10 PC2 → VLAN 20 VLAN 99 → Administración (SVI en switches) Solo 1 interfaz VLAN por switch (SVI) Se requiere conectividad total → esto SOLO se logra cuando entra el router

🔹 1. Situación inicial (SIN router)
PC1 → VLAN 10
PC2 → VLAN 20
VLAN 99 → administración (SVI en switches)

👉 En este estado:

❌ NO hay conectividad entre VLANs
Porque:

Los switches operan en capa 2
Las VLANs aíslan el tráfico

✔ Solo hay comunicación dentro de la misma VLAN
(a): inclusión del router usando interfaces ACCESS
📌 Diseño

Como no se permite trunk, se implementa:

👉 Una interfaz física del router por cada VLAN

VLAN	Red	Interfaz Router
VLAN 10	192.168.10.0/24	Fa0/0
VLAN 20	192.168.20.0/24	Fa0/1
VLAN 99	192.168.99.0/24	Fa1/0

🔹 3. Procedimiento de configuración
✅ 3.1 Configuración en TODOS los switches
🔸 Crear VLANs

```bash
enable
configure terminal
vlan 10
name VLAN10
vlan 20
name VLAN20
vlan 99
name ADMIN
end
copy running-config startup-config
```

🔸 Configuración de puertos de acceso a VLAN

En S1 (PC1)

enable
configure terminal
interface fa0/1
switchport mode access
switchport access vlan 10
end
copy running-config startup-config

En S2 (PC2)

enable
configure terminal
interface fa0/1
switchport mode access
switchport access vlan 20
end
copy running-config startup-config

3.3 Enlaces entre switches (TRUNK)

enable
configure terminal
interface fa0/2
switchport mode trunk
interface fa0/3
switchport mode trunk
end
copy running-config startup-config


3.4 VLAN de administración (SVI)

⚠️ SOLO UNA interfaz VLAN por switch (como pide el ejercicio)

S1

enable
configure terminal
interface vlan 99
ip address 192.168.99.11 255.255.255.0
no shutdown
end
copy running-config startup-config

S2

enable
configure terminal
interface vlan 99
ip address 192.168.99.12 255.255.255.0
ip default-gateway 192.168.99.1
no shutdown
end
copy running-config startup-config


S3

enable
configure terminal
interface vlan 99
ip address 192.168.99.13 255.255.255.0
no shutdown
end
copy running-config startup-config


## a) Gateway por defecto y nodo que cumple la función

En esta topología no existe ningún dispositivo de capa 3 que realice funciones de enrutamiento entre VLAN.

🔹 3.6 Puertos switch → router (ACCESS)
📍 En cada switch conectar así:
VLAN 10 → puerto hacia Fa0/0 del router
VLAN 20 → puerto hacia Fa0/1
VLAN 99 → puerto hacia Fa1/0

Ejemplo:

interface fa0/x
switchport mode access
switchport access vlan 10

(ajustar según conexión física)

🔐 🔹 3.7 Configuración de Telnet (OBLIGATORIO)

En cada switch:

enable
configure terminal

line vty 0 4
password cisco
login

end
🧪 🔹 4. Pruebas
✔ Desde PC1:
ping 192.168.20.10
ping 192.168.99.11
✔ Telnet:
telnet 192.168.99.11
telnet 192.168.99.12
telnet 192.168.99.13
🧠 🔹 5. Explicación teórica (lo que te piden)
✔ ¿Se necesita gateway?
✔ PCs → SÍ necesitan gateway
✔ Switches → SÍ (para administración)
❌ Router → NO necesita
✔ ¿Quién es el gateway?

👉 El router

192.168.10.1 → VLAN 10
192.168.20.1 → VLAN 20
192.168.99.1 → VLAN 99
✔ ¿Por qué?

Porque el router:

Opera en capa 3
Conecta dominios de broadcast distintos
Usa su tabla de enrutamiento
🚨 🔹 6. Errores comunes (muy importante)

❌ No poner ip default-gateway en switches
❌ Apagar interfaces del router
❌ No asignar VLAN correcta a puertos
❌ Olvidar no shutdown en SVI
❌ Usar trunk hacia router (NO permitido aquí)

🎯 🔹 7. Resultado final esperado

✔ PC1 ↔ PC2 → comunicación OK
✔ PCs ↔ switches → Telnet OK
✔ Switches ↔ router → OK
✔ Toda la red interconectada

### ¿Es necesario configurar gateway por defecto?

- Computadores: no tiene efecto práctico configurarlo, porque no existe un dispositivo que actúe como gateway.
- Switches (SVI de administración): tampoco tiene efecto funcional, por la misma razón.

### ¿Quién cumple la función de gateway?

Ningún nodo en la topología cumple la función de gateway.

### Explicación

Un gateway por defecto debe ser un dispositivo capaz de reenviar tráfico entre redes diferentes (capa 3).

En esta topología:

- Los switches operan en capa 2.
- Solo se permite una interfaz VLAN por switch, usada para administración.
- No hay router.
- No hay switch multilayer con ip routing.

### Conclusión para el informe

No es necesario ni funcional configurar un gateway por defecto en los computadores ni en los conmutadores, debido a que no existe en la topología un dispositivo de capa 3 que pueda cumplir dicha función. En consecuencia, ningún nodo actúa como gateway, lo que impide la comunicación entre diferentes VLAN.

## b) Camino del paquete (PC - interfaz VLAN del conmutador)

### Supuesto

- PC en VLAN 10.
- Interfaz virtual del switch (SVI) en VLAN 99.
- No existe dispositivo de capa 3.

### Proceso detallado

1. Generación del paquete (capa 3 - ICMP)
   - El computador genera un paquete.
   - Protocolo: ICMP (echo request) o Telnet.
   - IP origen: IP del PC.
   - IP destino: IP de la interfaz VLAN del switch.
   - Luego consulta su tabla de enrutamiento local.
2. Decisión de enrutamiento
   - El PC determina que la IP destino no pertenece a su red local (VLAN 10).
   - Por lo tanto, debe enviar el paquete al gateway por defecto.
3. Problema: ausencia de gateway
   - No existe un gateway real (no hay router).
   - Si no hay gateway configurado, el paquete se descarta inmediatamente.
   - Si hay gateway configurado (pero inexistente), se intenta ARP sin éxito.
4. Proceso ARP (capa 2)
   - En caso de que el PC tenga gateway configurado, envía un ARP Request para resolver la IP del gateway.
   - Este mensaje se envía como broadcast dentro de la VLAN 10 y no sale a otras VLAN.
   - Resultado: nadie responde y no se obtiene dirección MAC.
5. Encapsulamiento (no ocurre completamente)
   - Debido a que no hay MAC destino, no se puede construir la trama Ethernet completa.
   - El paquete IP no se encapsula correctamente en capa 2.
6. Rol de 802.1Q
   - Los enlaces entre switches pueden usar 802.1Q (trunk).
   - En este caso, el tráfico nunca sale del PC.
   - No se genera tráfico etiquetado y no hay uso efectivo de 802.1Q.
7. Tránsito por el switch (no ocurre)
   - El paquete no fue enviado correctamente.
   - No llega al switch, no hay conmutación de tramas y la SVI del switch nunca recibe el paquete.

### Resultado final

- No hay comunicación entre el PC y la SVI.
- El proceso se detiene en capa 3 (decisión de enrutamiento).
- ARP falla.
- No hay transmisión en capa 2.

### Conclusión lista para escribir

El computador genera un paquete ICMP dirigido a la interfaz VLAN del conmutador y consulta su tabla de enrutamiento, determinando que el destino pertenece a otra red. Debido a esto, intenta enviar el paquete al gateway por defecto. Sin embargo, al no existir un dispositivo de capa 3 en la topología, no hay un gateway funcional. En consecuencia, el computador realiza un proceso ARP para resolver la dirección MAC del gateway, pero no recibe respuesta. Al no poder resolver la dirección MAC de destino, no se completa el encapsulamiento en capa 2 y el paquete no es transmitido. Por lo tanto, no hay uso efectivo de 802.1Q ni tránsito por los switches, y la comunicación falla completamente.

## c) Uso de una única interfaz entre conmutador y enrutador

### ¿Qué se debe hacer?

Debes implementar router-on-a-stick, es decir:

- Un solo enlace físico entre switch y router.
- Ese enlace configurado como trunk (802.1Q).
- El router maneja múltiples VLAN mediante subinterfaces.

### Configuración conceptual

En el switch, el puerto hacia el router debe ser:

```bash
switchport mode trunk
```

Esto permite el paso de múltiples VLAN (10, 20, 99, etc.).

En el router se crean subinterfaces, una por VLAN:

```bash
interface fa0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0

interface fa0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0

interface fa0/0.99
encapsulation dot1Q 99
ip address 192.168.99.1 255.255.255.0
```

### Resultado

El router ahora:

- Recibe tramas etiquetadas (802.1Q).
- Identifica la VLAN.
- Enruta entre ellas.

Se mantiene una sola interfaz física, pero múltiples redes lógicas.

## d) Camino del paquete (PC - interfaz VLAN del switch)

### Supongamos

- PC en VLAN 10.
- Switch con SVI en VLAN 99.
- Router-on-a-stick configurado.

### Proceso completo

1. Generación del paquete (capa 3 - ICMP)
   - El PC genera un ICMP Echo Request.
   - IP origen: PC.
   - IP destino: SVI del switch.
   - Consulta su tabla de enrutamiento y detecta que el destino está en otra red.
2. Envío al gateway
   - El PC decide enviar el paquete al gateway (router, subinterfaz VLAN 10).
3. ARP (capa 2)
   - Si no conoce la MAC del gateway, envía ARP Request (broadcast en VLAN 10).
   - El switch solo reenvía dentro de VLAN 10.
   - El router responde con su MAC (subinterfaz VLAN 10).
4. Encapsulamiento inicial
   - Capa 3: origen IP del PC, destino IP de la SVI.
   - Capa 2: origen MAC del PC, destino MAC del router.
   - Sin etiqueta, porque el puerto del PC es access.
5. Paso por el switch (capa 2 + 802.1Q)
   - La trama entra por puerto access (VLAN 10).
   - Al salir por el puerto trunk hacia el router, el switch agrega etiqueta 802.1Q (VLAN 10).
6. Recepción en el router
   - El router recibe la trama etiquetada.
   - Identifica VLAN 10 y la envía a la subinterfaz fa0/0.10.
7. Enrutamiento (capa 3)
   - El router revisa su tabla de enrutamiento.
   - Determina que la red destino es VLAN 99.
   - Decide reenviar por la subinterfaz fa0/0.99.
8. ARP hacia el switch (VLAN 99)
   - Si no conoce la MAC del switch, envía ARP en VLAN 99.
   - El switch responde con la MAC de su interfaz VLAN (SVI).
9. Envío de vuelta al switch
   - Capa 3: origen IP del PC, destino IP de la SVI.
   - Capa 2: origen MAC del router, destino MAC del switch.
   - Con etiqueta 802.1Q VLAN 99.
10. Switch recibe la trama
    - El switch quita la etiqueta 802.1Q.
    - Detecta que el destino es su propia interfaz VLAN.
    - Procesa el paquete.
11. Respuesta
    - El switch responde y envía al router (gateway VLAN 99).
    - El router reenvía al PC (VLAN 10).
    - Es el mismo proceso en sentido inverso.

### Resumen clave

En capa 2:

- ARP: PC -> Router y Router -> Switch.
- 802.1Q: se agrega en enlaces trunk para identificar VLAN.

En capa 3:

- ICMP (o Telnet).
- Uso de tabla de enrutamiento en el router.
- Decisión de siguiente salto.

### Conclusión lista para entregar

El computador genera un paquete ICMP dirigido a la interfaz VLAN del conmutador y determina, mediante su tabla de enrutamiento, que el destino pertenece a otra red. Por ello, envía el paquete a su gateway por defecto. Tras resolver la dirección MAC del router mediante ARP, el paquete es encapsulado y enviado al switch, el cual añade una etiqueta 802.1Q al reenviarlo por el enlace trunk. El router recibe la trama, identifica la VLAN de origen, elimina la etiqueta y enruta el paquete hacia la VLAN destino utilizando su tabla de enrutamiento. Luego, encapsula nuevamente el paquete con la etiqueta correspondiente a la VLAN destino y lo envía al switch, que finalmente lo entrega a su interfaz VLAN. Este proceso involucra ARP para resolución de direcciones MAC, uso de 802.1Q en enlaces troncales y decisiones de enrutamiento en capa 3.
