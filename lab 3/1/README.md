# Laboratorio 3 - Punto 1

## 1. Contexto general

- PC1 esta en VLAN 10.
- PC2 esta en VLAN 20.
- La administracion de switches usa VLAN 99 (SVI de gestion).
- Los switches del escenario son capa 2.

Por lo tanto, para comunicar redes diferentes (VLAN 10, 20 y 99) se necesita un equipo de capa 3, en este caso el router.

---

## 2. Numeral (a)

Inicialmente se conecta el switch al router usando solo puertos access (una interfaz fisica del router por VLAN).

### 2.1 Direccionamiento y gateways

| VLAN | Red | Gateway en el router |
| --- | --- | --- |
| 10 | 192.168.10.0/24 | 192.168.10.1 |
| 20 | 192.168.20.0/24 | 192.168.20.1 |
| 99 | 192.168.99.0/24 | 192.168.99.1 |

### 2.2 Explicacion solicitada

Si es necesario configurar gateway por defecto en los computadores y en los switches de gestion.

- Computadores: necesitan gateway por defecto para enviar trafico a redes remotas (otras VLAN).
- Switches capa 2: usan ip default-gateway para poder ser administrados desde otra red.
- Router: no requiere gateway por defecto para enrutar entre sus redes directamente conectadas.

Nodo que cumple la funcion de gateway:

- El router es el gateway de la topologia.
- Cada interfaz del router asociada a una VLAN es la puerta de enlace de esa VLAN.

### 2.3 Configuracion base (escenario con interfaces access)

Router (una interfaz por VLAN):

```bash
enable
configure terminal
interface g0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface g0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown

interface g0/2
 ip address 192.168.99.1 255.255.255.0
 no shutdown
end
copy running-config startup-config
```

Switch conectado al router (puertos access):

```bash
enable
configure terminal
interface fa0/10
 switchport mode access
 switchport access vlan 10

interface fa0/11
 switchport mode access
 switchport access vlan 20

interface fa0/12
 switchport mode access
 switchport access vlan 99
end
copy running-config startup-config
```

### 2.4 Configuracion de hosts

PC1 (VLAN 10):

| Campo | Valor |
| --- | --- |
| IP | 192.168.10.10 |
| Mascara | 255.255.255.0 |
| Gateway | 192.168.10.1 |

PC2 (VLAN 20):

| Campo | Valor |
| --- | --- |
| IP | 192.168.20.10 |
| Mascara | 255.255.255.0 |
| Gateway | 192.168.20.1 |

### 2.5 Switches (gateway de administracion)

En S1, S2 y S3:

```bash
ip default-gateway 192.168.99.1
```

### 2.6 Acceso remoto por Telnet (VTY)

En los switches administrables (S1, S2 y S3):

```bash
enable
configure terminal
line vty 0 4
 password cisco
 login
end
copy running-config startup-config
```

### 2.7 Pruebas sugeridas (despues de configurar Telnet)

```bash
ping 192.168.20.10
ping 192.168.99.11
telnet 192.168.99.11
telnet 192.168.99.12
telnet 192.168.99.13
```

### 2.8 Verificacion final (escenario inicial)

En el router:

```bash
show ip interface brief
```

Esperado (numeral a, con interfaces fisicas separadas):

```text
G0/0  192.168.10.1  up up
G0/1  192.168.20.1  up up
G0/2  192.168.99.1  up up
```

---

## 3. Numeral (b)

Camino completo de un paquete desde un computador hasta la interfaz virtual del switch (SVI), en el escenario del numeral (a) con puertos access al router.

Supuesto de ejemplo:

- Origen: PC1 en VLAN 10.
- Destino: SVI del switch en VLAN 99 (ejemplo 192.168.99.11).

### 3.1 Proceso detallado capa 3 y capa 2

1. PC1 genera un ICMP Echo Request con destino 192.168.99.11.
2. PC1 revisa su mascara y detecta que 192.168.99.11 no esta en su red local 192.168.10.0/24.
3. PC1 consulta su tabla de enrutamiento local y usa su ruta por defecto hacia 192.168.10.1.
4. Si PC1 no conoce la MAC del gateway, envia ARP Request en VLAN 10 preguntando por 192.168.10.1.
5. El router responde ARP Reply con la MAC de su interfaz en VLAN 10.
6. PC1 encapsula el ICMP en una trama Ethernet con MAC destino del router y la envia por su puerto access VLAN 10.
7. El switch reenvia la trama dentro de VLAN 10 hasta el puerto access que conecta al router para esa VLAN (sin etiqueta 802.1Q en ese enlace, por ser access).
8. El router recibe la trama, desencapsula, analiza el paquete IP y busca en su tabla de enrutamiento.
9. El router determina que la red 192.168.99.0/24 esta conectada directamente por su interfaz de VLAN 99.
10. Si no conoce la MAC de la SVI destino o del equipo de gestion correspondiente, hace ARP en VLAN 99.
11. El router reencapsula y envia el paquete por su interfaz fisica de VLAN 99 hacia el switch.
12. El switch entrega el paquete a su proceso interno de SVI VLAN 99 y responde con ICMP Echo Reply.
13. La respuesta regresa por el camino inverso, usando nuevamente ARP y tabla de enrutamiento cuando aplique.

Nota sobre 802.1Q en este numeral:

- En el tramo switch-router del numeral (a) no hay etiqueta 802.1Q, porque los enlaces al router son access.
- La etiqueta 802.1Q aparece en enlaces trunk entre switches si esos enlaces transportan varias VLAN.

---

## 4. Numeral (c)

Se eliminan las conexiones previas switch-router y se migra a una unica interfaz fisica entre switch y router (router-on-a-stick).

### 4.1 Cambios necesarios

- Dejar un solo enlace fisico entre switch y router.
- Configurar ese puerto del switch en modo trunk.
- Crear subinterfaces en el router, una por VLAN, con encapsulacion dot1Q.

### 4.2 Configuracion de referencia

Switch (puerto hacia el router):

```bash
enable
configure terminal
interface fa0/24
 switchport mode trunk
end
copy running-config startup-config
```

Router (subinterfaces):

```bash
enable
configure terminal
interface g0/0
 no shutdown

interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface g0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface g0/0.99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0
end
copy running-config startup-config
```

---

## 5. Numeral (d)

Camino completo de un paquete desde un computador hasta la interfaz virtual del switch (SVI), ahora en el escenario del numeral (c) con un unico enlace trunk switch-router.

Supuesto de ejemplo:

- Origen: PC1 en VLAN 10.
- Destino: SVI de gestion en VLAN 99.

### 5.1 Proceso detallado capa 3 y capa 2

1. PC1 crea un ICMP Echo Request hacia la IP de la SVI en VLAN 99.
2. PC1 detecta que el destino esta en otra red y decide enviar al gateway 192.168.10.1.
3. Si no tiene la MAC del gateway, ejecuta ARP en VLAN 10.
4. El router responde por su subinterfaz g0/0.10 y PC1 arma la trama con MAC destino del gateway.
5. PC1 envia la trama sin etiqueta al switch por puerto access VLAN 10.
6. Al salir del switch por el enlace trunk hacia el router, la trama se marca con etiqueta 802.1Q VLAN 10.
7. El router recibe la trama etiquetada, la asocia a la subinterfaz g0/0.10, desencapsula y revisa la cabecera IP.
8. Usando su tabla de enrutamiento, el router decide reenviar hacia la red 192.168.99.0/24 por la subinterfaz g0/0.99.
9. Si es necesario, el router realiza ARP en VLAN 99 para resolver la MAC del destino.
10. El router envia la nueva trama por el mismo enlace fisico, ahora etiquetada con 802.1Q VLAN 99.
11. El switch recibe la trama trunk, identifica VLAN 99 y la entrega a la SVI de gestion.
12. La respuesta ICMP Echo Reply vuelve por el camino inverso con el mismo principio: switching L2 por VLAN y enrutamiento L3 en el router.

---

## 6. Verificacion recomendada

### 6.1 Comandos utiles

```bash
show ip interface brief
show vlan brief
show interfaces trunk
show ip route
show arp
```

Si se implemento el numeral (c), en este comando se esperan subinterfaces activas en una sola interfaz fisica (router-on-a-stick).

---

## 7. Errores comunes

- No configurar gateway por defecto en PCs o switches de gestion.
- Dejar interfaces o subinterfaces en shutdown.
- Configurar mal el modo del puerto (access o trunk).
- Omitir encapsulation dot1Q en subinterfaces del router.
- Asignar VLAN incorrecta a un puerto de acceso.
