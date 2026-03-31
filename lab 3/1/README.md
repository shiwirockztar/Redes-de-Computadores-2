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

### 2.1 Diseno

En este escenario se usa una interfaz fisica del router por cada VLAN.

### 2.2 Interfaces del router Cisco 2911

Normalmente:

- GigabitEthernet0/0
- GigabitEthernet0/1
- GigabitEthernet0/2

Se puede conectar una por VLAN (porque no se permite trunk en este escenario).

### 2.3 Conexion sugerida (router conectado a S3)

| Router 2911 | Switch S3 | VLAN |
| --- | --- | --- |
| G0/0 | Fa0/10 | VLAN 10 |
| G0/1 | Fa0/11 | VLAN 20 |
| G0/2 | Fa0/12 | VLAN 99 |

### 2.4 Direccionamiento y gateways

| VLAN | Red | Gateway en el router |
| --- | --- | --- |
| 10 | 192.168.10.0/24 | 192.168.10.1 |
| 20 | 192.168.20.0/24 | 192.168.20.1 |
| 99 | 192.168.99.0/24 | 192.168.99.1 |

### 2.5 Explicacion solicitada

Si es necesario configurar gateway por defecto en los computadores y en los switches de gestion.

- Computadores: necesitan gateway por defecto para enviar trafico a redes remotas (otras VLAN).
- Switches capa 2: usan ip default-gateway para poder ser administrados desde otra red.
- Router: no requiere gateway por defecto para enrutar entre sus redes directamente conectadas.

Nodo que cumple la funcion de gateway:

- El router es el gateway de la topologia.
- Cada interfaz del router asociada a una VLAN es la puerta de enlace de esa VLAN.

### 2.6 Configuracion base en switches

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

### 2.7 Puertos de acceso para hosts

S1 (PC1 en VLAN 10):

```bash
enable
configure terminal
interface fa0/1
 switchport mode access
 switchport access vlan 10
end
copy running-config startup-config
```

S2 (PC2 en VLAN 20):

```bash
enable
configure terminal
interface fa0/1
 switchport mode access
 switchport access vlan 20
end
copy running-config startup-config
```

### 2.8 Enlaces entre switches (trunk)

```bash
enable
configure terminal
interface fa0/2
 switchport mode trunk
interface fa0/3
 switchport mode trunk
end
copy running-config startup-config
```

### 2.9 SVI de administracion (VLAN 99)

S1:

```bash
enable
configure terminal
interface vlan 99
 ip address 192.168.99.11 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.99.1
end
copy running-config startup-config
```

S2:

```bash
enable
configure terminal
interface vlan 99
 ip address 192.168.99.12 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.99.1
end
copy running-config startup-config
```

S3:

```bash
enable
configure terminal
interface vlan 99
 ip address 192.168.99.13 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.99.1
end
copy running-config startup-config
```

### 2.10 Puertos de S3 hacia el router (access)

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

### 2.11 Router (una interfaz por VLAN)

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

### 2.12 Configuracion de hosts

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

### 2.13 Acceso remoto por Telnet (VTY)

En los switches administrables (S1, S2 y S3):

```bash
enable
configure terminal
line vty 0 4
 password 123
 login
end
copy running-config startup-config
```

### 2.14 Pruebas sugeridas (despues de configurar Telnet)

```bash
ping 192.168.20.10
ping 192.168.99.11
telnet 192.168.99.11
telnet 192.168.99.12
telnet 192.168.99.13
```

### 2.15 Verificacion final (escenario inicial)

En el router:

```bash
show ip interface brief
```

Esperado (numeral a, con interfaces fisicas separadas):

```text
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     192.168.10.1    YES manual up                    up 
GigabitEthernet0/1     192.168.20.1    YES manual up                    up 
GigabitEthernet0/2     192.168.99.1    YES manual up                    up 
Vlan1                  unassigned      YES unset  administratively down down
```

---

## 3. Numeral (b)

Camino completo de un paquete desde un computador hasta la interfaz virtual del switch (SVI), en el escenario del numeral (a) con puertos access al router.

Supuesto de ejemplo:

- Origen: PC1 en VLAN 10.
- Destino: SVI de S2 en VLAN 99 (192.168.99.12).

Camino general del paquete:

PC1 -> S1 -> S3 -> Router -> S3 -> S2 -> Interfaz VLAN 99 (SVI)

Punto clave del escenario:

- Al cambiar de VLAN, el trafico debe pasar por el router.
- Como el destino es una direccion de administracion, el trafico termina en una SVI.

### 3.1 Proceso detallado capa 3 y capa 2

1. PC1 quiere enviar trafico a 192.168.99.12.
2. PC1 compara esa IP con su red local 192.168.10.0/24 y detecta que el destino esta en otra red.
3. PC1 decide enviar el paquete a su gateway por defecto: 192.168.10.1.
4. Si PC1 no conoce la MAC del gateway, envia ARP Request (broadcast) en VLAN 10 preguntando por 192.168.10.1.
5. El router responde con ARP Reply y su direccion MAC para VLAN 10.
6. PC1 encapsula y envia el paquete al router:

	- Capa 3 (IP): origen 192.168.10.10, destino 192.168.99.12.
	- Capa 2 (Ethernet): MAC origen PC1, MAC destino router.

7. Entre switches (S1, S3 y S2) el trafico de enlaces troncales se transporta con 802.1Q, etiquetando la VLAN correspondiente.
8. El router recibe la trama por su interfaz de VLAN 10, quita la encapsulacion de capa 2 y revisa el destino IP 192.168.99.12.
9. El router consulta su tabla de enrutamiento y determina que 192.168.99.0/24 esta conectada directamente por su interfaz hacia VLAN 99.
10. Si el router no conoce la MAC de 192.168.99.12, realiza ARP en VLAN 99; S2 responde con la MAC de su SVI.
11. El router reencapsula el paquete para VLAN 99:

	- Capa 3 (IP): origen 192.168.10.10, destino 192.168.99.12.
	- Capa 2 (Ethernet): MAC origen router, MAC destino S2.

12. El paquete llega a la interfaz VLAN 99 de S2, donde se procesa.
13. Si la prueba es ping, S2 responde con ICMP Echo Reply y el trafico vuelve por el camino inverso.

### 3.2 Resumen para entrega

Camino del paquete:

PC -> Switch de acceso -> Switch de distribucion -> Router -> Switch -> SVI destino.

Procesos de capa 3:

- El host detecta red remota y usa su puerta de enlace predeterminada.
- Las IP origen/destino se mantienen durante el trayecto: 192.168.10.10 -> 192.168.99.12.
- El router consulta su tabla de enrutamiento y reenvia por la interfaz de la red destino.
- En pruebas de conectividad se usa ICMP (Echo Request / Echo Reply).

Procesos de capa 2:

- ARP inicial para resolver la MAC del gateway.
- Reenvio por switches segun VLAN.
- Uso de 802.1Q en enlaces trunk para identificar trafico de VLAN.
- Nuevo ARP del router en VLAN 99 para resolver la MAC de la SVI destino.
- Reencapsulacion con nuevas MAC en cada salto de capa 3.

Frase clave:

"Cada salto implica cambio de encapsulamiento de capa 2, mientras que la direccion IP se mantiene constante durante todo el recorrido."

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

Nota de equivalencia con tu guia:

- Si en tu topologia el router usa FastEthernet, la configuracion es la misma cambiando el nombre de la interfaz (por ejemplo fa0/0.10, fa0/0.20, fa0/0.99).

---

## 5. Numeral (d)

Camino completo de un paquete desde un computador hasta la interfaz virtual del switch (SVI), ahora en el escenario del numeral (c) con un unico enlace trunk switch-router.

Supuesto de ejemplo:

- Origen: PC1 en VLAN 10.
- Destino: SVI de gestion en VLAN 99 (ejemplo 192.168.99.3).

### 5.1 Proceso detallado capa 3 y capa 2

1. PC1 crea un ICMP Echo Request hacia 192.168.99.3.
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

### 5.2 Conclusiones clave (b, c y d)

- Sin router no hay comunicacion entre VLANs.
- El router es el gateway de toda la red en ambos escenarios (interfaces fisicas separadas o router-on-a-stick).
- Las VLAN separan dominios de broadcast.
- 802.1Q permite transportar multiples VLAN en un mismo enlace trunk.
- El enrutamiento inter-VLAN lo realiza el router.
- La SVI de gestion permite administracion remota (por ejemplo, Telnet).

Frase clave tipo examen:

"La conectividad total entre equipos en diferentes VLANs se logra unicamente mediante un dispositivo de capa 3 (router), el cual actua como gateway y permite el enrutamiento entre redes, ya sea usando interfaces fisicas o mediante router-on-a-stick con encapsulacion 802.1Q."

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
