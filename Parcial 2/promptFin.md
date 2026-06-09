Estoy trabajando en una topología compleja en GNS3 con IS-IS, OSPF y BGP. Necesito que generes la configuración completa correcta y optimizada de todos los routers.

========================
📌 TOPOLOGÍA GENERAL
========================

LAN A: 172.16.0.0/24
   |
R7 — R6 — R1 — R4 — R5 — R2 — R3 — LAN B: 10.10.10.0/24
                                              |
                                          Cloud / ISP (eBGP externo)

========================
📌 REQUISITOS OBLIGATORIOS (IGP)
========================

1. Deben existir 2 IGP en la topología:
   - IS-IS (lado LAN A hasta el core)
   - OSPF área 0 (lado LAN B hasta el core)

2. IS-IS:
   - Level-2-only en todos los routers del dominio IS-IS
   - Adyacencias correctas entre R7–R6–R1–R5 (según diseño)
   - Redistribución controlada en el punto de frontera

3. OSPF:
   - Área única (area 0)
   - Corre en R5–R4–R2–R3
   - Debe anunciar LAN B y redes internas

4. R1:
   - NO puede ejecutar ningún protocolo de enrutamiento (ni IS-IS ni OSPF)
   - Solo puede usar rutas estáticas
   - Debe garantizar conectividad completa entre IS-IS y OSPF mediante rutas estáticas bien definidas

========================
📌 REQUISITOS BGP
========================

5. iBGP (dentro de la topología):
   - Debe configurarse iBGP entre routers internos del dominio BGP
   - NO se permite full-mesh iBGP
   - Se debe usar una de estas soluciones:
     - Route Reflector (preferido)
     - o Confederation (si aplica)
   - Debe evitarse el diseño iBGP todos-con-todos

6. eBGP:
   - Debe establecerse una sesión eBGP con un router externo del profesor (ISP/Cloud)
   - Solo se permite anunciar mediante eBGP UNA ÚNICA red adicional
   - Esa red debe ser diferente a:
     - LAN A (172.16.0.0/24)
     - LAN B (10.10.10.0/24)
   - LAN A y LAN B NO deben anunciarse por eBGP

7. Objetivo eBGP:
   - LAN A y LAN B deben poder hacer ping a una IP externa definida en la nube del profesor

========================
📌 RESTRICCIONES IMPORTANTES
========================

- No usar full-mesh iBGP
- No redistribuir BGP de forma que genere bucles
- Evitar rutas asimétricas
- Evitar loops de redistribución entre IS-IS ↔ OSPF ↔ BGP
- R1 no puede ejecutar routing dinámico
- Debe haber consistencia de next-hop en toda la red

========================
📌 PROBLEMAS ACTUALES
========================

- Conectividad inter-dominio es inconsistente
- Rutas entre IS-IS y OSPF no siempre se propagan
- R7 no ve redes externas (10.10.10.0 o eBGP)
- Posibles fallos de retorno (return path)
- Falta de diseño iBGP correcto (sin full mesh)

========================
📌 LO QUE QUIERO QUE GENERES
========================

1. Configuración completa de todos los routers:
   - R7
   - R6
   - R1 (solo static routing)
   - R4
   - R5 (core redistribution + BGP)
   - R2
   - R3
   - Router ISP/Cloud (eBGP peer)

2. Incluir:
   - IS-IS completo (NET, level-2, interfaces)
   - OSPF area 0 completo
   - Redistribución IS-IS ↔ OSPF en R5 correctamente filtrada
   - iBGP sin full mesh (con Route Reflector o confederation)
   - eBGP con anuncio de UNA sola red externa
   - rutas estáticas necesarias en R1

3. Incluir verificación:
   - show ip route (IGP y BGP)
   - show ip bgp
   - show ip bgp summary
   - show isis database
   - show ip ospf database
   - ping end-to-end (LAN A ↔ LAN B ↔ ISP)
   - traceroute completo

========================
📌 OBJETIVO FINAL
========================

La red debe cumplir:

- LAN A puede llegar a LAN B
- LAN A y LAN B pueden llegar a la red externa del ISP vía eBGP
- IS-IS y OSPF deben coexistir correctamente
- iBGP debe funcionar sin full-mesh
- R1 no ejecuta routing dinámico pero no rompe conectividad

Todo debe quedar funcionando sin ajustes manuales posteriores.
