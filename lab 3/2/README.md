# Camino del Paquete (PC -> Interfaz VLAN del Conmutador con HSRP)

## Supuestos

- PC en VLAN 10.
- SVI del switch en VLAN 99.
- Dos routers (R1 y R2).
- HSRP configurado en las VLAN involucradas.
- Existe una IP virtual como gateway HSRP.

## Proceso Completo

1. **Generacion del paquete (Capa 3 - ICMP)**
   - El PC genera un paquete ICMP (echo request).
   - IP origen: PC.
   - IP destino: SVI del switch.
   - El PC consulta su tabla de enrutamiento, detecta que el destino esta en otra red y decide enviarlo al gateway por defecto.

2. **Gateway HSRP (concepto clave)**
   - El gateway configurado en el PC es la **IP virtual de HSRP**, no la IP fisica de un router.
   - Solo un router esta activo.
   - El otro router esta en estado standby.

3. **ARP hacia el gateway virtual (Capa 2)**
   - El PC envia un ARP Request: "Quien tiene la IP del gateway?".
   - El router activo responde con la **MAC virtual de HSRP**.
   - Punto clave: el PC no conoce routers reales; solo conoce IP virtual y MAC virtual.

4. **Envio del paquete al switch**
   - Capa 3:
     - Origen: IP del PC.
     - Destino: IP de la SVI.
   - Capa 2:
     - Origen: MAC del PC.
     - Destino: MAC virtual HSRP.

5. **Paso por el switch (802.1Q)**
   - El switch recibe la trama por puerto access (VLAN 10).
   - Al reenviarla por el enlace trunk hacia los routers, agrega etiqueta 802.1Q de VLAN 10.

6. **Tramo critico: switch -> routers (HSRP)**
   - La trama llega a ambos routers (si comparten el mismo dominio L2).
   - Solo el router activo procesa el paquete porque posee la MAC virtual.
   - El router standby ignora ese trafico.

7. **Procesamiento en el router activo (Capa 3)**
   - El router activo elimina la etiqueta 802.1Q.
   - Identifica que el trafico viene de VLAN 10.
   - Consulta su tabla de enrutamiento y determina que el destino esta en VLAN 99.

8. **ARP hacia el switch (VLAN 99)**
   - Si el router no conoce la MAC del switch, envia ARP en VLAN 99.
   - El switch responde con la MAC de su interfaz VLAN (SVI).

9. **Reenvio hacia el switch**
   - Capa 3:
     - Origen: IP del PC.
     - Destino: IP de la SVI.
   - Capa 2:
     - Origen: MAC del router.
     - Destino: MAC del switch.
   - Se encapsula con etiqueta 802.1Q de VLAN 99.

10. **Recepcion en el switch**
    - El switch quita la etiqueta.
    - Reconoce su propia IP (SVI).
    - Procesa el paquete.

11. **Respuesta y alta disponibilidad**
    - El switch responde y envia la respuesta al gateway virtual HSRP.
    - El proceso se repite en sentido inverso.
    - Si falla el router activo, el standby asume IP y MAC virtuales.
    - El PC no requiere cambios ni nuevo ARP, por lo que la comunicacion continua.

## Resumen por Capas

### Capa 2

- **ARP**:
  - PC -> MAC virtual HSRP.
  - Router -> MAC del switch.
- **802.1Q**:
  - Se usa en enlaces trunk.
  - Identifica la VLAN en transito.

### Capa 3

- ICMP (ping).
- Consulta de tabla de enrutamiento en el router.
- Decision de siguiente salto.

## Conclusion (lista para entregar)

El computador genera un paquete ICMP hacia la interfaz VLAN del conmutador y, al determinar que el destino pertenece a otra red, lo envia al gateway por defecto configurado como una direccion IP virtual de HSRP. Mediante ARP, el computador obtiene la direccion MAC virtual asociada a dicha IP, la cual es atendida por el router activo. El paquete es enviado al switch, que añade una etiqueta 802.1Q al reenviarlo por el enlace troncal hacia los routers. Ambos routers reciben la trama, pero solo el activo la procesa. Este elimina la etiqueta, consulta su tabla de enrutamiento y reenvia el paquete hacia la VLAN destino, encapsulandolo nuevamente con la etiqueta correspondiente. Finalmente, el switch recibe la trama, elimina la etiqueta y entrega el paquete a su interfaz VLAN. En caso de falla del router activo, el router en espera asume la IP y MAC virtuales, garantizando continuidad sin afectar al host.
