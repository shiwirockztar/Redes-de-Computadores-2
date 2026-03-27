(a) Gateway por defecto y nodo que cumple la función
En esta topología no existe ningún dispositivo de capa 3 que realice funciones de enrutamiento entre VLANs.
🔹 ¿Es necesario configurar gateway por defecto?
Computadores:
❌ No tiene efecto práctico configurarlo, porque no existe un dispositivo que actúe como gateway.
Switches (SVI de administración):
❌ Tampoco tiene efecto funcional, por la misma razón.
🔹 ¿Quién cumple la función de gateway?
👉 Ningún nodo en la topología cumple la función de gateway.
🔹 Explicación
Un gateway por defecto debe ser:
Un dispositivo capaz de reenviar tráfico entre redes diferentes (capa 3)
En esta topología:
Los switches operan en capa 2
Solo se permite una interfaz VLAN por switch, usada para administración
No hay:
Router
Switch multilayer con ip routing
🔹 Conclusión (para el informe)
No es necesario ni funcional configurar un gateway por defecto en los computadores ni en los conmutadores, debido a que no existe en la topología un dispositivo de capa 3 que pueda cumplir dicha función. En consecuencia, ningún nodo actúa como gateway, lo que impide la comunicación entre diferentes VLAN.

(b) Camino del paquete (PC → interfaz VLAN del conmutador)
Supuesto:
PC en VLAN 10
Interfaz virtual del switch (SVI) en VLAN 99
No existe dispositivo de capa 3
🔁 Proceso detallado

1. Generación del paquete (Capa 3 – ICMP)
   El computador genera un paquete:
   Protocolo: ICMP (echo request) o Telnet
   IP origen: IP del PC
   IP destino: IP de la interfaz VLAN del switch
   Luego:
   Consulta su tabla de enrutamiento local
2. Decisión de enrutamiento
   El PC determina que:
   La IP destino no pertenece a su red local (VLAN 10)
   Por lo tanto:
   Debe enviar el paquete al gateway por defecto
3. Problema: ausencia de gateway
   En esta topología:
   ❌ No existe un gateway real (no hay router)
   Entonces:
   Si no hay gateway configurado → el paquete se descarta inmediatamente
   Si hay gateway configurado (pero inexistente) → se intenta ARP sin éxito
4. Proceso ARP (Capa 2)
   En caso de que el PC tenga gateway configurado:
   El PC envía un ARP Request:
   “¿Quién tiene la IP del gateway?”
   Este mensaje:
   Se envía como broadcast dentro de la VLAN 10
   NO sale a otras VLAN
   Resultado:
   ❌ Nadie responde (no hay gateway)
   ❌ No se obtiene dirección MAC
5. Encapsulamiento (NO ocurre completamente)
   Debido a que no hay MAC destino:
   ❌ No se puede construir la trama Ethernet completa
   ❌ El paquete IP no se encapsula correctamente en capa 2
6. Rol de 802.1Q
   Los enlaces entre switches pueden usar 802.1Q (trunk)
   Sin embargo:
   👉 En este caso:
   El tráfico nunca sale del PC
   Por lo tanto:
   ❌ No se genera tráfico etiquetado
   ❌ No hay uso efectivo de 802.1Q
7. Tránsito por el switch (no ocurre)
   Dado que:
   El paquete no fue enviado correctamente
   👉 Entonces:
   ❌ No llega al switch
   ❌ No hay conmutación de tramas
   ❌ La interfaz VLAN del switch nunca recibe el paquete
   🔴 Resultado final
   ❌ No hay comunicación entre el PC y la SVI
   ❌ El proceso se detiene en capa 3 (decisión de enrutamiento)
   ❌ ARP falla
   ❌ No hay transmisión en capa 2
   ✅ Conclusión (lista para escribir)
   El computador genera un paquete ICMP dirigido a la interfaz VLAN del conmutador y consulta su tabla de enrutamiento, determinando que el destino pertenece a otra red. Debido a esto, intenta enviar el paquete al gateway por defecto. Sin embargo, al no existir un dispositivo de capa 3 en la topología, no hay un gateway funcional. En consecuencia, el computador realiza un proceso ARP para resolver la dirección MAC del gateway, pero no recibe respuesta. Al no poder resolver la dirección MAC de destino, no se completa el encapsulamiento en capa 2 y el paquete no es transmitido. Por lo tanto, no hay uso efectivo de 802.1Q ni tránsito por los switches, y la comunicación falla completamente.

(c) Uso de una única interfaz entre conmutador y enrutador
🔹 ¿Qué se debe hacer?
Debes implementar router-on-a-stick, es decir:
Un solo enlace físico entre switch y router
Ese enlace configurado como trunk (802.1Q)
El router maneja múltiples VLAN mediante subinterfaces
🔧 Configuración conceptual
En el switch
El puerto hacia el router debe ser:
Bash
switchport mode trunk
Permite el paso de múltiples VLAN (10, 20, 99, etc.)
En el router
Se crean subinterfaces, una por VLAN:
Bash
interface fa0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0

interface fa0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0

interface fa0/0.99
encapsulation dot1Q 99
ip address 192.168.99.1 255.255.255.0
🔹 Resultado
El router ahora:
Recibe tramas etiquetadas (802.1Q)
Identifica la VLAN
Enruta entre ellas
👉 Se mantiene una sola interfaz física, pero múltiples redes lógicas.
(d) Camino del paquete (PC → interfaz VLAN del switch)
Supongamos:
PC en VLAN 10
Switch con SVI en VLAN 99
Router-on-a-stick configurado
🔁 Proceso completo

1. Generación del paquete (Capa 3 – ICMP)
   El PC genera:
   ICMP Echo Request
   IP origen: PC
   IP destino: SVI del switch
   Luego:
   Consulta su tabla de enrutamiento
   👉 Detecta que el destino está en otra red
2. Envío al gateway
   El PC decide enviar el paquete al:
   👉 Gateway (router, subinterfaz VLAN 10)
3. ARP (Capa 2)
   Si no conoce la MAC del gateway:
   Envía ARP Request (broadcast en VLAN 10)
   El switch:
   Solo reenvía dentro de VLAN 10
   El router:
   Responde con su MAC (subinterfaz VLAN 10)
4. Encapsulamiento inicial
   El PC envía la trama:
   Capa 3:
   Origen: IP PC
   Destino: IP SVI
   Capa 2:
   Origen: MAC PC
   Destino: MAC del router
   Sin etiqueta (porque el puerto del PC es access)
5. Paso por el switch (Capa 2 + 802.1Q)
   Cuando la trama llega al switch:
   Entra por un puerto access (VLAN 10)
   Al salir por el puerto trunk hacia el router:
   👉 El switch agrega etiqueta 802.1Q (VLAN 10)
6. Recepción en el router
   El router:
   Recibe la trama etiquetada
   Identifica VLAN 10
   La envía a la subinterfaz fa0/0.10
7. Enrutamiento (Capa 3)
   El router:
   Revisa su tabla de enrutamiento
   Determina que la red destino es VLAN 99
   👉 Decide reenviar por la subinterfaz fa0/0.99
8. ARP hacia el switch (VLAN 99)
   Si no conoce la MAC del switch:
   Envía ARP en VLAN 99
   El switch responde con:
   👉 MAC de su interfaz VLAN (SVI)
9. Envío de vuelta al switch
   El router envía:
   Capa 3:
   Origen: IP PC
   Destino: IP SVI
   Capa 2:
   Origen: MAC router
   Destino: MAC switch
   Con etiqueta 802.1Q VLAN 99
10. Switch recibe la trama
    El switch:
    Quita la etiqueta 802.1Q
    Detecta que el destino es su propia interfaz VLAN
    👉 Procesa el paquete
11. Respuesta
    El switch responde:
    Envía al router (gateway VLAN 99)
    El router reenvía al PC (VLAN 10)
    👉 Mismo proceso en sentido inverso
    🔹 Resumen clave
    Capa 2
    ARP:
    PC → Router
    Router → Switch
    802.1Q:
    Se agrega en enlaces trunk
    Permite identificar VLAN
    Capa 3
    ICMP (o Telnet)
    Uso de tabla de enrutamiento en el router
    Decisión de siguiente salto
    ✅ Conclusión (lista para entregar)
    El computador genera un paquete ICMP dirigido a la interfaz VLAN del conmutador y determina, mediante su tabla de enrutamiento, que el destino pertenece a otra red. Por ello, envía el paquete a su gateway por defecto. Tras resolver la dirección MAC del router mediante ARP, el paquete es encapsulado y enviado al switch, el cual añade una etiqueta 802.1Q al reenviarlo por el enlace trunk. El router recibe la trama, identifica la VLAN de origen, elimina la etiqueta y enruta el paquete hacia la VLAN destino utilizando su tabla de enrutamiento. Luego, encapsula nuevamente el paquete con la etiqueta correspondiente a la VLAN destino y lo envía al switch, que finalmente lo entrega a su interfaz VLAN. Este proceso involucra ARP para resolución de direcciones MAC, uso de 802.1Q en enlaces troncales y decisiones de enrutamiento en capa 3.
