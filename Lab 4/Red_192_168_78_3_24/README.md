# Laboratorio 4 - Telnet en Windows

## 1. Objetivo

Configurar y utilizar Telnet para conectarse a dispositivos de red desde Windows.

## 2. Plan de direccionamiento

### 2.1 Red base y subdivisión

**Red asignada:** `192.168.78.3/24`.

Para efectos de subneteo, la red base sigue siendo `192.168.78.0/24`, y desde allí se obtienen las subredes necesarias para la misma topología.

**Cinco redes requeridas:**

| # | RED | CIDR | Rango | Hosts útiles | Dispositivos |
| --- | --- | --- | --- | --- | --- |
| 1 | LAN_A | /26 | 192.168.78.0 - 192.168.78.63 | 62 | LAN_A ↔ R1 |
| 2 | LAN_B | /26 | 192.168.78.64 - 192.168.78.127 | 62 | LAN_B ↔ R2 |
| 3 | Switch1 | /26 | 192.168.78.128 - 192.168.78.191 | 62 | R1, R2, R3 ↔ Switch1 |
| 4 | R3-R4 | /29 | 192.168.78.192 - 192.168.78.199 | 6 | R3 ↔ R4 (Serial) |
| 5 | LAN_C | /29 | 192.168.78.200 - 192.168.78.207 | 6 | R4 ↔ LAN_C |

### 2.2 Asignación de direcciones por segmento

#### **Red 1: LAN_A (192.168.78.0/26)**
| Dispositivo | Interfaz | Dirección IP | Rol |
| --- | --- | --- | --- |
| R1 | Fa0/0 | 192.168.78.1 | Gateway |
| LAN_A | Ethernet | 192.168.78.10 | PC cliente |
| Broadcast | - | 192.168.78.63 | - |

#### **Red 2: LAN_B (192.168.78.64/26)**
| Dispositivo | Interfaz | Dirección IP | Rol |
| --- | --- | --- | --- |
| R2 | Fa0/0 | 192.168.78.65 | Gateway |
| LAN_B | Ethernet | 192.168.78.74 | PC cliente |
| Broadcast | - | 192.168.78.127 | - |

#### **Red 3: Switch1 (192.168.78.128/26)**
| Dispositivo | Interfaz | Dirección IP | Rol |
| --- | --- | --- | --- |
| R1 | Fa0/1 | 192.168.78.129 | Router |
| R2 | Fa0/1 | 192.168.78.130 | Router |
| R3 | Fa0/0 | 192.168.78.131 | Router |
| Switch1 | VLAN 1 (mgmt) | 192.168.78.132 | Switch gestión |
| Broadcast | - | 192.168.78.191 | - |

#### **Red 4: Enlace R3-R4 (192.168.78.192/29)**
| Dispositivo | Interfaz | Dirección IP | Rol |
| --- | --- | --- | --- |
| R3 | s0/0 | 192.168.78.193 | Router |
| R4 | s0/0 | 192.168.78.194 | Router |
| Broadcast | - | 192.168.78.199 | - |

#### **Red 5: LAN_C (192.168.78.200/29)**
| Dispositivo | Interfaz | Dirección IP | Rol |
| --- | --- | --- | --- |
| R4 | Fa0/0 | 192.168.78.201 | Gateway |
| LAN_C | Ethernet | 192.168.78.202 | PC cliente |
| Broadcast | - | 192.168.78.207 | - |

### 2.3 Tabla de direcciones resumen

| Dispositivo | Interfaz | Dirección IP | Máscara | Red |
| --- | --- | --- | --- | --- |
| R1 | Fa0/0 | 192.168.78.1 | 255.255.255.192 | LAN_A |
| R1 | Fa0/1 | 192.168.78.129 | 255.255.255.192 | Switch1 |
| R2 | Fa0/0 | 192.168.78.65 | 255.255.255.192 | LAN_B |
| R2 | Fa0/1 | 192.168.78.130 | 255.255.255.192 | Switch1 |
| R3 | Fa0/0 | 192.168.78.131 | 255.255.255.192 | Switch1 |
| R3 | s0/0 | 192.168.78.193 | 255.255.255.248 | R3-R4 |
| R4 | Fa0/0 | 192.168.78.201 | 255.255.255.248 | LAN_C |
| R4 | s0/0 | 192.168.78.194 | 255.255.255.248 | R3-R4 |
| Switch1 | VLAN 1 | 192.168.78.132 | 255.255.255.192 | Switch1 |
| LAN_A | Ethernet | 192.168.78.10 | 255.255.255.192 | LAN_A |
| LAN_B | Ethernet | 192.168.78.74 | 255.255.255.192 | LAN_B |
| LAN_C | Ethernet | 192.168.78.202 | 255.255.255.248 | LAN_C |

## 3. Topología GNS3

### 3.1 Diagrama de conexiones

```
LAN_A              LAN_B                                  LAN_C
  |                  |                                      |
  |                  |                                      |
Fa0/0              Fa0/0                                 Fa0/0
  |                  |                                      |
 R1                 R2                 R3                 R4
  |                  |                  |                  |
Fa0/1              Fa0/1               Fa0/0            s0/0
  |                  |                  |                  |
  +------------------+--Switch1---------+------------------+
         e0              e1              e2
                                         |
                                        s0/0
                                         |
                                        R4
```

### 3.2 Tabla de conexiones

| Dispositivo A | Puerto A | Dispositivo B | Puerto B | Tipo de conexión |
| --- | --- | --- | --- | --- |
| LAN_A | Ethernet | R1 | Fa0/0 | Red local |
| LAN_B | Ethernet | R2 | Fa0/0 | Red local |
| LAN_C | Ethernet | R4 | Fa0/0 | Red local |
| R1 | Fa0/1 | Switch1 | e0 | Fast Ethernet |
| R2 | Fa0/1 | Switch1 | e1 | Fast Ethernet |
| R3 | Fa0/0 | Switch1 | e2 | Fast Ethernet |
| R3 | s0/0 | R4 | s0/0 | Serial (WAN) |

## 4. Configuración de equipos (PCs)

### 4.1 Comando para asignar dirección IP y gateway

En sistemas Linux/Unix, se utiliza el comando `ip` para configurar la dirección IP y el gateway en una interfaz de red.

**Sintaxis general:**
```bash
ip <dirección_ip>/<máscara_bits> <dirección_gateway>
```

**Parámetros:**
- `<dirección_ip>`: Dirección IP a asignar (ej: 192.168.78.10)
- `/<máscara_bits>`: Notación CIDR de la máscara (ej: /26 = 255.255.255.192)
- `<dirección_gateway>`: IP del gateway por defecto

### 4.2 Acceso por Telnet a los equipos (antes de configurar IP)

En GNS3, cada equipo puede abrirse por consola Telnet con el comando:

```bash
telnet localhost 500x
```

Donde `500x` es el puerto asignado al dispositivo.

#### **Puertos de conexión por Telnet:**

| Dispositivo | Puerto | Comando |
| --- | --- | --- |
| LAN_A | 5004 | `telnet localhost 5004` |
| LAN_B | 5006 | `telnet localhost 5006` |
| LAN_C | 5008 | `telnet localhost 5008` |
| R1 | 5000 | `telnet localhost 5000` |
| R2 | 5001 | `telnet localhost 5001` |
| R3 | 5002 | `telnet localhost 5002` |
| R4 | 5003 | `telnet localhost 5003` |
| Switch1 | none | N/A |

Nota: los números de puerto pueden variar según la configuración de GNS3. Switch1 sí tiene 3 conexiones físicas (con R1, R2 y R3), pero en esta práctica no tiene puerto Telnet local asignado (`none`).

### 4.3 Flujo recomendado: Telnet + configuración IP en PCs

1. Conéctate al equipo por Telnet.
2. Ejecuta el comando `ip` con su máscara y gateway.
3. Verifica conectividad con `ping` al gateway.

#### **LAN_A**
```bash
telnet localhost 5004
ip 192.168.78.10/26 192.168.78.1
save
ping 192.168.78.1
```

#### **LAN_B**
```bash
telnet localhost 5006
ip 192.168.78.74/26 192.168.78.65
save
ping 192.168.78.65
```

#### **LAN_C**
```bash
telnet localhost 5008
ip 192.168.78.202/29 192.168.78.201
save
ping 192.168.78.201
```

### 4.4 Configuración por equipos

#### **Configuración de LAN_A**
```bash
ip 192.168.78.10/26 192.168.78.1
```
- Dirección IP: 192.168.78.10
- Máscara: 255.255.255.192 (/26)
- Gateway: 192.168.78.1 (R1)
- Red: LAN_A

#### **Configuración de LAN_B**
```bash
ip 192.168.78.74/26 192.168.78.65
```
- Dirección IP: 192.168.78.74
- Máscara: 255.255.255.192 (/26)
- Gateway: 192.168.78.65 (R2)
- Red: LAN_B

#### **Configuración de LAN_C**
```bash
ip 192.168.78.202/29 192.168.78.201
```
- Dirección IP: 192.168.78.202
- Máscara: 255.255.255.248 (/29)
- Gateway: 192.168.78.201 (R4)
- Red: LAN_C

### 4.5 ¿Por qué se usa /26 si la red asignada es /24?

La red `192.168.78.3/24` identifica la base usada en el laboratorio. Luego se divide en subredes más pequeñas (subneteo/VLSM) para separar los segmentos de la topología.

En este laboratorio se definieron 5 redes:
- LAN_A: `192.168.78.0/26`
- LAN_B: `192.168.78.64/26`
- Switch1: `192.168.78.128/26`
- Enlace R3-R4: `192.168.78.192/29`
- LAN_C: `192.168.78.200/29`

Por eso, en LAN_A el comando correcto es `ip 192.168.78.10/26 192.168.78.1`: la IP del host y su gateway pertenecen a la subred `192.168.78.0/26`.

Si se configurara `/24` en el host, el equipo asumiría que direcciones de otras subredes están en su misma red local, afectando la segmentación y el enrutamiento esperado.

### 4.6 Configuración de R1 (Cisco IOS)

En router Cisco, los comandos correctos son `configure terminal`, `interface fa...` e `ip address`.

```bash
enable
configure terminal

interface fa0/0
 ip address 192.168.78.1 255.255.255.192
 no shutdown

interface fa0/1
 ip address 192.168.78.129 255.255.255.192
 no shutdown

end
write memory
```

Comandos abreviados equivalentes:

```bash
conf t
int fa0/0
 ip add 192.168.78.1 255.255.255.192
 no shut
int fa0/1
 ip add 192.168.78.129 255.255.255.192
 no shut
end
wr
```

### 4.7 Configuración de R2 (Cisco IOS)

```bash
enable
configure terminal

interface fa0/0
 ip address 192.168.78.65 255.255.255.192
 no shutdown

interface fa0/1
 ip address 192.168.78.130 255.255.255.192
 no shutdown

end
write memory
```

Comandos abreviados equivalentes:

```bash
conf t
int fa0/0
 ip add 192.168.78.65 255.255.255.192
 no shut
int fa0/1
 ip add 192.168.78.130 255.255.255.192
 no shut
end
wr
```

### 4.8 Configuración de R3 (Cisco IOS)

Nota: si tu interfaz serial es DCE, agrega `clock rate 64000` en `s0/0`.

```bash
enable
configure terminal

interface fa0/0
 ip address 192.168.78.131 255.255.255.192
 no shutdown

interface s0/0
 ip address 192.168.78.193 255.255.255.248
 no shutdown

end
write memory
```

Comandos abreviados equivalentes:

```bash
conf t
int fa0/0
 ip add 192.168.78.131 255.255.255.192
 no shut
int s0/0
 ip add 192.168.78.193 255.255.255.248
 no shut
end
wr
```

### 4.9 Configuración de R4 (Cisco IOS)

Nota: si tu interfaz serial es DCE, agrega `clock rate 64000` en `s0/0`.

```bash
enable
configure terminal

interface fa0/0
 ip address 192.168.78.201 255.255.255.248
 no shutdown

interface s0/0
 ip address 192.168.78.194 255.255.255.248
 no shutdown

end
write memory
```

Comandos abreviados equivalentes:

```bash
conf t
int fa0/0
 ip add 192.168.78.201 255.255.255.248
 no shut
int s0/0
 ip add 192.168.78.194 255.255.255.248
 no shut
end
wr
```

### 4.10 Configuración de Switch1 (gestión)

```bash
enable
configure terminal

interface vlan 1
 ip address 192.168.78.132 255.255.255.192
 no shutdown

ip default-gateway 192.168.78.129

end
write memory
```

Comandos abreviados equivalentes:

```bash
conf t
int vlan 1
 ip add 192.168.78.132 255.255.255.192
 no shut
ip default-gateway 192.168.78.129
end
wr
```

### 4.11 Rutas estáticas mínimas para conectividad entre redes

Estas rutas permiten que todas las LAN se alcancen entre sí.

#### **R1**
```bash
conf t
ip route 192.168.78.64 255.255.255.192 192.168.78.130
ip route 192.168.78.192 255.255.255.248 192.168.78.131
ip route 192.168.78.200 255.255.255.248 192.168.78.131
end
wr
```

#### **R2**
```bash
conf t
ip route 192.168.78.0 255.255.255.192 192.168.78.129
ip route 192.168.78.192 255.255.255.248 192.168.78.131
ip route 192.168.78.200 255.255.255.248 192.168.78.131
end
wr
```

#### **R3**
```bash
conf t
ip route 192.168.78.0 255.255.255.192 192.168.78.129
ip route 192.168.78.64 255.255.255.192 192.168.78.130
ip route 192.168.78.200 255.255.255.248 192.168.78.194
end
wr
```

#### **R4**
```bash
conf t
ip route 192.168.78.0 255.255.255.192 192.168.78.193
ip route 192.168.78.64 255.255.255.192 192.168.78.193
ip route 192.168.78.128 255.255.255.192 192.168.78.193
end
wr
```

### 4.12 Verificación rápida

En cada router:

```bash
show ip interface brief
show ip route
```

Pruebas sugeridas:

```bash
ping 192.168.78.1
ping 192.168.78.65
ping 192.168.78.131
ping 192.168.78.194
ping 192.168.78.201
ping 192.168.78.202
```

### 4.13 Orden corto de configuración

1. Abre la consola por Telnet con el puerto de cada equipo.
2. Configura primero los routers R1, R2, R3 y R4.
3. Configura Switch1 con su VLAN de gestión.
4. Configura las PCs con `ip dirección/máscara gateway`.
5. Agrega las rutas estáticas.
6. Verifica con `show ip interface brief`, `show ip route` y `ping`.

### 4.14 Pasos para conectar ambas redes con VirtualBox

Si vas a unir esta topología con la anterior para hacer pruebas, puedes usar VirtualBox en modo puente para que la VM quede dentro de la misma red de capa 2 que el enlace del cloud.

#### **Lista de pasos recomendada**

1. En GNS3, conecta la interfaz de R4 al objeto Cloud.
2. En el Cloud, selecciona la interfaz física o virtual que usará el puente con la red de pruebas.
3. En VirtualBox, abre la configuración de la máquina virtual que vas a usar para las pruebas.
4. En la sección de red, activa el adaptador de red que conectará con la topología.
5. Cambia el modo del adaptador a **Adaptador puente**.
6. Selecciona la interfaz correcta de tu PC, por ejemplo Ethernet o Wi-Fi, según dónde esté conectado el puente.
7. Inicia la máquina virtual y verifica que reciba o tenga configurada una IP válida dentro de la red que compartirás con GNS3.
8. En R4, confirma que la interfaz conectada al Cloud esté arriba y tenga la dirección IP correcta.
9. Comprueba que ambas puntas estén en la misma red o que exista enrutamiento entre ellas.
10. Haz `ping` desde la VM hacia R4 y desde R4 hacia la VM para confirmar conectividad.

#### **Verificaciones mínimas**

- Revisar que el adaptador de VirtualBox esté en **Modo puente**.
- Confirmar que la interfaz seleccionada sea la misma red física o virtual usada por GNS3.
- Validar que no haya conflicto de direcciones IP.
- Si el ping no responde, revisar firewall de la VM, máscara, gateway y la interfaz del Cloud.

#### **Ejemplo de prueba**

Si la VM queda en la misma red de prueba, puedes usar una IP libre del mismo segmento y probar:

```bash
ping 192.168.78.194
ping 192.168.78.201
```

Si la topología anterior también está conectada al mismo Cloud, entonces ambas redes podrán intercambiar tráfico a través de R4 y del segmento de prueba de VirtualBox.

## 5. Habilitar Telnet en Windows

### 5.1 Pasos:

1. Abre **Características de Windows** (Windows Features)
2. Busca **Telnet Client**
3. Marca la casilla para activar
4. Aplica los cambios y reinicia si es necesario

La conexión por Telnet a equipos y puertos de GNS3 está documentada en la sección **4.2**.

## 6. Salir de Telnet

### Comando para desconectar:

Cuando estés conectado por Telnet:

1. Presiona `Ctrl + ]`
   - Esto abre el prompt de Telnet: `telnet>`

2. Escribe:
   ```
   quit
   ```

3. Presiona Enter para desconectar

## 7. Notas importantes

- Telnet es un protocolo no encriptado, usar solo en redes internas de prueba
- SSH es la alternativa segura recomendada para producción