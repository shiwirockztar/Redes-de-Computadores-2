# Parcial Integrador - Sustentacion tecnica (Puntos a, b, c, d)

Este documento consolida lo desarrollado en los parciales [1-2](1-2/README.md), [3](3/README.md), [4](4/README.md), [5](5/README.md) y [6](6/README.md).

## a) Esquema de direccionamiento

Se usa exclusivamente el bloque **10.10.8.0/24**, dividido en 4 subredes /26, una por VLAN:

| VLAN | Proposito | Red | Mascara | Rango de hosts | Broadcast | Gateway virtual (HSRP) |
|---|---|---|---|---|---|---|
| 10 | Usuarios | 10.10.8.0/26 | 255.255.255.192 | 10.10.8.1 - 10.10.8.62 | 10.10.8.63 | 10.10.8.1 |
| 20 | Usuarios | 10.10.8.64/26 | 255.255.255.192 | 10.10.8.65 - 10.10.8.126 | 10.10.8.127 | 10.10.8.65 |
| 30 | Usuarios / Nativa de troncales | 10.10.8.128/26 | 255.255.255.192 | 10.10.8.129 - 10.10.8.190 | 10.10.8.191 | 10.10.8.129 |
| 40 | Administracion | 10.10.8.192/26 | 255.255.255.192 | 10.10.8.193 - 10.10.8.254 | 10.10.8.255 | 10.10.8.193 |

### Hosts de infraestructura (tambien cuentan como hosts de VLAN)

1. **Gateways virtuales (HSRP):**
- VLAN 10: 10.10.8.1
- VLAN 20: 10.10.8.65
- VLAN 30: 10.10.8.129
- VLAN 40: 10.10.8.193

2. **Interfaces de enrutadores (inter-VLAN + redundancia):**
- R1:
  - G0/0.10 = 10.10.8.2/26
  - G0/0.20 = 10.10.8.66/26
  - G0/0.30 = 10.10.8.130/26
  - G0/1 (VLAN 40 fisica) = 10.10.8.194/26
- R2:
  - G0/0.10 = 10.10.8.3/26
  - G0/0.20 = 10.10.8.67/26
  - G0/0.30 = 10.10.8.131/26
  - G0/1 (VLAN 40 fisica) = 10.10.8.195/26

3. **Interfaces de administracion de conmutadores (SVI VLAN 40):**
- SW1 = 10.10.8.200/26
- SW2 = 10.10.8.201/26
- SW3 = 10.10.8.202/26
- SW4 = 10.10.8.203/26
- SW5 = 10.10.8.204/26
- SW6 = 10.10.8.205/26

4. **Ejemplos de hosts finales:**
- PC VLAN 10: 10.10.8.10/26, GW 10.10.8.1
- PC VLAN 20: 10.10.8.70/26, GW 10.10.8.65
- PC VLAN 30: 10.10.8.150/26, GW 10.10.8.129
- PC VLAN 40: 10.10.8.210/26, GW 10.10.8.193

Nota tecnica: se normalizaron IPs de gestion y ejemplos de hosts para evitar duplicidad con las IP de R1/R2 en VLAN 40 y VLAN 30.

## b) Como se logro la conectividad entre VLAN y desde que equipos hay Telnet/SSH

### Conectividad entre VLAN

La conectividad inter-VLAN se logro con **dos routers en esquema principal/backup** usando **HSRP**:

- R1 es principal para VLAN 10 y 30.
- R2 es principal para VLAN 20 y 40.
- En caso de falla del router activo en una VLAN, el router en espera asume el gateway virtual y mantiene la comunicacion.

Mecanismo de transporte:

1. VLAN 10, 20 y 30 via **Router-on-a-Stick** (subinterfaces 802.1Q sobre G0/0).
2. VLAN 40 via **interfaz fisica dedicada** (sin subinterfaces), cumpliendo el requisito del enunciado.
3. Troncales entre switches con VLAN nativa 30 y VLANs permitidas 10,20,30,40.

### Desde que equipos se puede hacer Telnet y/o SSH a los switches

- Los switches tienen gestion en **VLAN 40** y configuracion de acceso remoto por **Telnet y SSH**.
- Por diseno, el acceso de administracion se realiza principalmente desde los hosts de VLAN 40 (por ejemplo PC de administracion).
- Como existe inter-VLAN y no se definieron ACL de restriccion en los README base, tambien podria alcanzarse desde otras VLAN, siempre que haya ruta y resolucion ARP hacia la IP de gestion.

Credenciales de gestion documentadas:

- `enable secret`: `redes`
- VTY/Telnet: `udea`
- SSH: usuario `admin`, clave `martes`

## c) Prueba de conectividad (ping) y flujo completo L2/L3

### Escenario elegido

Ping desde un equipo en VLAN 10 hacia otro equipo en VLAN 20:

- Origen: PC-A VLAN 10 = 10.10.8.10/26, GW 10.10.8.1
- Destino: PC-B VLAN 20 = 10.10.8.70/26, GW 10.10.8.65

### Flujo de la peticion (Echo Request)

1. **Decision de capa 3 en el origen:**
- PC-A compara 10.10.8.70 con su red local 10.10.8.0/26.
- Como el destino esta en otra subred, envia al gateway 10.10.8.1 (HSRP VLAN 10).

2. **ARP en VLAN 10:**
- Si PC-A no conoce la MAC del gateway virtual, emite ARP Request broadcast en VLAN 10.
- El router activo de HSRP para VLAN 10 responde con la MAC virtual del grupo.

3. **Conmutacion de capa 2 en acceso/distribucion/core:**
- El frame entra al switch de acceso por puerto access VLAN 10.
- Cruza la red por enlaces trunk/port-channel permitiendo VLAN 10.
- STP solo deja activos los caminos sin bucles; los puertos bloqueados no reenvian datos de usuario.

4. **Enrutamiento en el router activo:**
- El router recibe el paquete IP (src 10.10.8.10, dst 10.10.8.70).
- Busca la red 10.10.8.64/26 (VLAN 20) en la tabla de enrutamiento conectada.
- Decrementa TTL, recalcula checksum y reencapsula en VLAN 20 por la subinterfaz correspondiente.

5. **ARP en VLAN 20 (si hace falta):**
- Si el router no conoce la MAC de PC-B, hace ARP Request en VLAN 20.
- PC-B responde con su MAC.

6. **Entrega al destino:**
- El switch reenvia unicast hasta el puerto access VLAN 20 donde esta PC-B.
- PC-B recibe el ICMP Echo Request.

### Flujo de la respuesta (Echo Reply)

1. PC-B detecta que 10.10.8.10 esta fuera de su red local 10.10.8.64/26.
2. Envia al gateway 10.10.8.65 (HSRP VLAN 20).
3. El router activo de VLAN 20 enruta hacia 10.10.8.0/26.
4. Reencapsula en VLAN 10 y entrega a PC-A.

### Consideracion de redundancia entre routers

- Si el router principal de una VLAN cae, HSRP conmuta al backup.
- El host no cambia su gateway configurado (sigue usando la IP virtual).
- Tras convergencia ARP/HSRP, el ping vuelve a responder por el nuevo router activo.

## d) Configuracion correcta de STP y estrategia usada

Se aplico una estrategia **PVST/Rapid-PVST por VLAN** con prioridades manuales para controlar el arbol y evitar depender de MAC aleatoria.

### Estrategia de raiz por VLAN

1. **VLAN 10 y 20:** raiz en Core (SW1).
- Objetivo: trayectorias estables para trafico de usuarios y control del arbol desde el nivel superior.

2. **VLAN 30 y 40:** raiz en distribucion (SW2) y secundario SW3.
- Objetivo: separar dominios de control y mejorar resiliencia operativa.

### Resultado operativo esperado

- Existe un unico camino activo por VLAN entre dos puntos (sin bucles L2).
- Enlaces redundantes permanecen disponibles como backup (estado bloqueado/alternate cuando corresponde).
- Ante falla de enlace/equipo, STP reconverge y habilita una ruta alternativa.
- Con EtherChannel, STP ve el Port-Channel como enlace logico, mejorando estabilidad y capacidad.

### Coherencia con el resto del parcial

La combinacion de **STP + EtherChannel + HSRP** permite:

1. Evitar bucles en capa 2.
2. Mantener capacidad de transporte en uplinks (sin perdida por saturacion segun dimensionamiento).
3. Sostener la conectividad inter-VLAN ante falla de un router.

---

## Conclusiones

- El direccionamiento cumple la restriccion de usar solo 10.10.8.0/24.
- La interconectividad entre VLAN se implemento con redundancia real (principal/backup por VLAN).
- La gestion remota de switches se centraliza en VLAN 40 con Telnet/SSH.
- El STP fue disenado de forma intencional por VLAN para estabilidad, convergencia y soporte de enlaces redundantes.
