cada vez que llega un icmp al s3 del router no deberia salir a sw1 por la interfaz Fa0/2 de sw3



verifique ejecutando el codigo en sw3 (se evidencia que por la interfaz Fa0/2) no deberia haber trafico

Switch>show mac address-table 
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----

   1    0060.7019.9603    DYNAMIC     Fa0/3
  10    0005.5eeb.29d7    DYNAMIC     Fa0/3
  10    0060.7019.9603    DYNAMIC     Fa0/3
  # Preguntas y respuestas - Lab 3

  Este documento organiza las dudas del laboratorio con base en la topologia descrita en:

  - [lab 3/1/README.md](lab 3/1/README.md)
  - [lab 3/2/README.md](lab 3/2/README.md)

  ## 1) Recorrido de ICMP entre VLAN 10 (S1) y VLAN 20 (S2) sin ARP

  Supuesto: ARP ya fue resuelto y solo se observa ICMP en Simulation.

  ### Ida (Echo Request)

  1. PC VLAN 10 -> S1 (puerto access VLAN 10).
  2. S1 -> S3 por trunk (trama etiquetada VLAN 10).
  3. S3 -> router que enruta inter-VLAN:
    - Punto 1: router unico.
    - Punto 2: router activo HSRP (R1 o R2) por trunk.
  4. Router enruta de VLAN 10 a VLAN 20.
  5. Router -> S3 en VLAN 20.
  6. S3 -> S2 por trunk (VLAN 20).
  7. S2 -> PC VLAN 20 (puerto access VLAN 20).

  ### Vuelta (Echo Reply)

  1. PC VLAN 20 -> S2.
  2. S2 -> S3 por trunk VLAN 20.
  3. S3 -> router activo (VLAN 20).
  4. Router enruta de VLAN 20 a VLAN 10.
  5. Router -> S3 en VLAN 10.
  6. S3 -> S1 por trunk VLAN 10.
  7. S1 -> PC VLAN 10.

  Resumen: ida = VLAN10 -> gateway L3 -> VLAN20, vuelta = VLAN20 -> gateway L3 -> VLAN10.

  ## 2) Por que en S3 aparecen copias ICMP hacia varias interfaces

  No siempre es un bug. En la mayoria de casos es comportamiento normal de capa 2:

  1. Si S3 no conoce la MAC destino en esa VLAN, hace unknown unicast flooding.
  2. Ese trafico sale por todos los puertos de esa VLAN (excepto el de entrada), incluyendo trunks a S1, R1 y R2.
  3. Solo el router activo HSRP procesa la trama util del gateway virtual.
  4. El router standby recibe/copias pero no enruta como gateway activo.

  Por eso puedes ver paquetes "fallidos" en Simulation y solo una ruta valida de extremo a extremo.

  ## 3) Evidencia actual observada en S3

  Salida reportada de tabla MAC:

  ```text
  Switch>show mac address-table
         Mac Address Table
  -------------------------------------------

  Vlan    Mac Address       Type        Ports
  ----    -----------       --------    -----

    1    0060.7019.9603    DYNAMIC     Fa0/3
    10    0005.5eeb.29d7    DYNAMIC     Fa0/3
    10    0060.7019.9603    DYNAMIC     Fa0/3
    10    0090.2156.0001    DYNAMIC     Fa0/11
    10    00d0.bac0.6201    DYNAMIC     Fa0/10
    20    0060.7019.9603    DYNAMIC     Fa0/3
    20    0090.2156.0001    DYNAMIC     Fa0/11
    20    00d0.bac0.6201    DYNAMIC     Fa0/10
    99    0060.7019.9603    DYNAMIC     Fa0/3
    99    0090.2156.0001    DYNAMIC     Fa0/11
    99    00d0.bac0.6201    DYNAMIC     Fa0/10
  ```

  Lectura rapida:

  1. S3 aprende MACs por Fa0/10 y Fa0/11 (enlaces a routers).
  2. Si en algun momento no existe la MAC exacta de destino, S3 replica temporalmente el trafico dentro de la VLAN.

  ## 4) Verificacion recomendada en Lab 3

  1. En S3: `clear mac address-table dynamic`.
  2. En PCs: limpiar ARP (o reiniciar host).
  3. En R1/R2: `show standby brief` para confirmar Active/Standby.
  4. En S3: `show mac address-table dynamic vlan 10` y `show mac address-table dynamic vlan 20`.
  5. Lanzar un ping inicial (puede haber flooding).
  6. Lanzar 4-5 pings seguidos.
  7. Repetir consulta MAC en S3.
  8. En S3: `show interfaces trunk` para confirmar VLAN permitidas (10,20,99).

  ## 5) Criterio de conclusion

  1. Si el "ruido" ocurre solo al inicio y luego disminuye: comportamiento normal de aprendizaje L2.
  2. Si persiste igual despues de varios pings: revisar trunks/VLAN, aprendizaje MAC, o limitacion visual de Packet Tracer.
Resumen corto:
