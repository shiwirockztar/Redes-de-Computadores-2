# Laboratorio 4 - Telnet en Windows

## 1. Objetivo

Configurar y utilizar Telnet para conectarse a dispositivos de red desde Windows.

## 2. Plan de Direccionamiento

### 2.1 Red base y subdivisión

**Red asignada:** `192.168.78.0/24` (256 direcciones)

**División de subredes:**

| Subred | CIDR | Rango | Hosts útiles | Uso |
| --- | --- | --- | --- | --- |
| 192.168.78.0 | /26 | 192.168.78.0 - 192.168.78.63 | 62 | LAN_A |
| 192.168.78.64 | /26 | 192.168.78.64 - 192.168.78.127 | 62 | LAN_B |
| 192.168.78.128 | /26 | 192.168.78.128 - 192.168.78.191 | 62 | LAN_C |
| 192.168.78.192 | /29 | 192.168.78.192 - 192.168.78.199 | 6 | Enlace R3-R4 (Serial) |
| 192.168.78.200 | /29 | 192.168.78.200 - 192.168.78.207 | 6 | Gestión/Switch1 (Reserva) |

### 2.2 Asignación de direcciones por segmento

#### **Segmento LAN_A (192.168.78.0/26)**
| Dispositivo | Interfaz | Dirección IP | Gateway |
| --- | --- | --- | --- |
| R1 | Fa0/0 | 192.168.78.1 | - |
| LAN_A | Ethernet | 192.168.78.10 | 192.168.78.1 |
| Broadcast | - | 192.168.78.63 | - |

#### **Segmento LAN_B (192.168.78.64/26)**
| Dispositivo | Interfaz | Dirección IP | Gateway |
| --- | --- | --- | --- |
| R2 | Fa0/0 | 192.168.78.65 | - |
| LAN_B | Ethernet | 192.168.78.74 | 192.168.78.65 |
| Broadcast | - | 192.168.78.127 | - |

#### **Segmento LAN_C (192.168.78.128/26)**
| Dispositivo | Interfaz | Dirección IP | Gateway |
| --- | --- | --- | --- |
| R4 | Fa0/0 | 192.168.78.129 | - |
| LAN_C | Ethernet | 192.168.78.138 | 192.168.78.129 |
| Broadcast | - | 192.168.78.191 | - |

#### **Enlace Serial R3-R4 (192.168.78.192/29)**
| Dispositivo | Interfaz | Dirección IP |
| --- | --- | --- |
| R3 | s0/0 | 192.168.78.193 |
| R4 | s0/0 | 192.168.78.194 |
| Broadcast | - | 192.168.78.199 |

#### **Gestión Switch1 (192.168.78.200/29)** - Reserva
| Dispositivo | Interfaz | Dirección IP |
| --- | --- | --- |
| Switch1 | VLAN 1 (mgmt) | 192.168.78.201 |
| Gateway | - | 192.168.78.200 |
| Broadcast | - | 192.168.78.207 |

### 2.3 Tabla de direcciones resumen

| Dispositivo | Tipo | Dirección IP | Máscara |
| --- | --- | --- | --- |
| R1 (Fa0/0) | Router | 192.168.78.1 | 255.255.255.192 |
| R2 (Fa0/0) | Router | 192.168.78.65 | 255.255.255.192 |
| R3 (s0/0) | Router | 192.168.78.193 | 255.255.255.248 |
| R4 (Fa0/0) | Router | 192.168.78.129 | 255.255.255.192 |
| R4 (s0/0) | Router | 192.168.78.194 | 255.255.255.248 |
| LAN_A | PC | 192.168.78.10 | 255.255.255.192 |
| LAN_B | PC | 192.168.78.74 | 255.255.255.192 |
| LAN_C | PC | 192.168.78.138 | 255.255.255.192 |

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

### 4.1 Comando para asignar dirección IP y Gateway

En sistemas Linux/Unix, se utiliza el comando `ip` para configurar la dirección IP y el gateway en una interfaz de red.

**Sintaxis general:**
```bash
ip <dirección_ip>/<máscara_bits> <dirección_gateway>
```

**Parámetros:**
- `<dirección_ip>`: Dirección IP a asignar (ej: 192.168.78.10)
- `/<máscara_bits>`: Notación CIDR de la máscara (ej: /26 = 255.255.255.192)
- `<dirección_gateway>`: IP del gateway por defecto

### 4.2 Configuración por equipos

#### **Configuración de LAN_A**
```bash
ip 192.168.78.10/26 192.168.78.1
```
- Dirección IP: 192.168.78.10
- Máscara: 255.255.255.192 (/26)
- Gateway: 192.168.78.1 (R1)

#### **Configuración de LAN_B**
```bash
ip 192.168.78.74/26 192.168.78.65
```
- Dirección IP: 192.168.78.74
- Máscara: 255.255.255.192 (/26)
- Gateway: 192.168.78.65 (R2)

#### **Configuración de LAN_C**
```bash
ip 192.168.78.138/26 192.168.78.129
```
- Dirección IP: 192.168.78.138
- Máscara: 255.255.255.192 (/26)
- Gateway: 192.168.78.129 (R4)

## 5. Habilitar Telnet en Windows

### Pasos:

1. Abre **Características de Windows** (Windows Features)
2. Busca **Telnet Client**
3. Marca la casilla para activar
4. Aplica los cambios y reinicia si es necesario

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