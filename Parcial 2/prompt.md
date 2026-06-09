Estoy trabajando en una topología de red en GNS3 y necesito que generes la configuración completa y corregida de todos los routers.

📌 TOPOLOGÍA GENERAL

LAN A (172.16.0.0/24)
   |
R7 (Fa0/0 172.16.0.2)
   |
R6 (IS-IS L2)
   |
R1 (NO debe ejecutar ningún IGP)
   |
R4 (OSPF)
   |
R5 (OSPF + IS-IS redistribución)
   |
R2 (OSPF)
   |
R3 (OSPF)
   |
LAN B (10.10.10.0/24)
Cloud Internet

📌 REQUISITOS OBLIGATORIOS

1. Deben coexistir 2 IGP:
   - IS-IS: en R7, R6 y hasta el borde hacia R1/R5
   - OSPF área 0: en R5, R4, R2, R3

2. R1:
   - NO puede ejecutar IS-IS ni OSPF
   - Solo puede usar rutas estáticas
   - Debe permitir tránsito completo entre IS-IS y OSPF

3. R5:
   - Es el router de redistribución entre IS-IS ↔ OSPF
   - Debe redistribuir:
     - IS-IS → OSPF (subnets + metric-type 2)
     - OSPF → IS-IS (level-2 + metric definida)
     - connected routes si es necesario

4. IS-IS:
   - Debe ser level-2-only en todos los routers IS-IS
   - Debe tener NET configurado correctamente
   - Interfaces correctamente activadas con "ip router isis CORE"

5. OSPF:
   - Área única (area 0)
   - Redistribución correcta en R5
   - Incluir redes conectadas

6. Conectividad final obligatoria:
   - R7 debe hacer ping a 10.10.10.10
   - LAN A (172.16.0.1) debe hacer ping a LAN B
   - Traceroute debe cruzar toda la topología sin saltos perdidos

📌 PROBLEMA ACTUAL

- R7 no puede llegar a 10.10.10.10
- R7 no tiene ruta hacia redes OSPF
- Redistribución en R5 está incompleta o asimétrica
- R1 puede estar rompiendo el forwarding por rutas estáticas incorrectas o incompletas
- IS-IS solo conoce redes internas, no OSPF externas

📌 LO QUE QUIERO QUE GENERES

Quiero que me devuelvas:

1. Configuración completa de:
   - R7
   - R6
   - R1 (solo static routing)
   - R4
   - R5 (redistribution core)
   - R2
   - R3

2. Incluye:
   - Configuración de interfaces (IP + no shutdown)
   - IS-IS correcto (NET, level-2, adjacency)
   - OSPF area 0 completo
   - Redistribución bidireccional en R5 correctamente filtrada y consistente
   - Rutas estáticas necesarias en R1 para garantizar tránsito

3. Incluye comandos de verificación:
   - show ip route
   - show isis database
   - show ip ospf database
   - ping end-to-end
   - traceroute end-to-end

4. Corrige cualquier error de diseño típico:
   - loops de redistribución
   - rutas faltantes de retorno
   - next-hop incorrectos
   - interfaces mal asignadas
   - subredes inconsistentes

📌 OBJETIVO FINAL

Que toda la red tenga conectividad total bidireccional entre:
- LAN A (172.16.0.0/24)
- LAN B (10.10.10.0/24)
- Internet cloud

Sin necesidad de ajustes manuales adicionales después de aplicar la configuración.