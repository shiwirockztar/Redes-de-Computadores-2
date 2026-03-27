a) Camino del paquete (PC → interfaz VLAN del conmutador con HSRP)
🔹 Supuesto
PC en VLAN 10
SVI del switch en VLAN 99
Dos routers (R1 y R2)
HSRP configurado en todas las VLAN
Existe una IP virtual (gateway HSRP)
🔁 Proceso completo

1. Generación del paquete (Capa 3 – ICMP)
   El PC genera un paquete:
   Protocolo: ICMP (echo request)
   IP origen: PC
   IP destino: SVI del switch
   Luego:
   Consulta su tabla de enrutamiento
   Detecta que el destino está en otra red
   👉 Decide enviar al gateway por defecto
2. Gateway HSRP (concepto clave)
   El PC tiene configurado como gateway:
   👉 IP virtual de HSRP, no la IP física de un router
   Solo un router está activo
   El otro está en standby
3. ARP hacia el gateway virtual (Capa 2)
   El PC envía:
   ARP Request: ¿Quién tiene la IP del gateway?
   👉 Respuesta:
   El router activo responde
   Pero responde con una MAC virtual HSRP
   🔴 Punto clave HSRP
   El PC no conoce routers reales
   Solo conoce:
   IP virtual
   MAC virtual
4. Envío del paquete al switch
   El PC envía:
   Capa 3:
   Origen: IP PC
   Destino: IP SVI
   Capa 2:
   Origen: MAC PC
   Destino: MAC virtual HSRP
5. Paso por el switch (802.1Q)
   El switch:
   Recibe trama por puerto access (VLAN 10)
   Al enviarla por el trunk hacia los routers:
   👉 Agrega etiqueta 802.1Q (VLAN 10)
6. Tramo crítico: switch → routers (HSRP)
   El frame llega a ambos routers (si están conectados al mismo dominio L2), pero:
   Solo el router activo:
   Tiene la MAC virtual
   Procesa el paquete
   El router standby:
   Ignora el tráfico
7. Procesamiento en el router activo (Capa 3)
   El router activo:
   Elimina la etiqueta 802.1Q
   Identifica VLAN 10
   Revisa su tabla de enrutamiento
   👉 Determina que el destino está en VLAN 99
8. ARP hacia el switch (VLAN 99)
   Si no conoce la MAC del switch:
   Envía ARP en VLAN 99
   El switch responde con:
   👉 MAC de su interfaz VLAN (SVI)
9. Reenvío hacia el switch
   El router envía:
   Capa 3:
   Origen: IP PC
   Destino: IP SVI
   Capa 2:
   Origen: MAC del router
   Destino: MAC del switch
   Con etiqueta: 👉 802.1Q VLAN 99
10. Recepción en el switch
    El switch:
    Quita la etiqueta
    Reconoce su propia IP (SVI)
    👉 Procesa el paquete
11. Respuesta
    El switch responde:
    Envía al gateway (IP virtual HSRP)
    El proceso se repite
    🔄 ¿Qué pasa si falla el router activo?
    Aquí entra HSRP:
    El router standby:
    Toma la IP virtual
    Toma la MAC virtual
    👉 Resultado:
    El PC no cambia nada
    No necesita nuevo ARP
    La comunicación continúa
    🔹 Resumen de procesos
    Capa 2
    ARP:
    PC → MAC virtual HSRP
    Router → MAC del switch
    802.1Q:
    Se usa en enlaces trunk
    Identifica VLAN en tránsito
    Capa 3
    ICMP (ping)
    Tabla de enrutamiento en el router
    Decisión de siguiente salto
    ✅ Conclusión (lista para entregar)
    El computador genera un paquete ICMP hacia la interfaz VLAN del conmutador y, al determinar que el destino pertenece a otra red, lo envía al gateway por defecto configurado como una dirección IP virtual de HSRP. Mediante ARP, el computador obtiene la dirección MAC virtual asociada a dicha IP, la cual es atendida por el router activo. El paquete es enviado al switch, que añade una etiqueta 802.1Q al reenviarlo por el enlace troncal hacia los routers. Ambos routers reciben la trama, pero solo el activo la procesa. Este elimina la etiqueta, consulta su tabla de enrutamiento y reenvía el paquete hacia la VLAN destino, encapsulándolo nuevamente con la etiqueta correspondiente. Finalmente, el switch recibe la trama, elimina la etiqueta y entrega el paquete a su interfaz VLAN. En caso de falla del router activo, el router en espera asume la IP y MAC virtuales, garantizando continuidad sin afectar al host.
