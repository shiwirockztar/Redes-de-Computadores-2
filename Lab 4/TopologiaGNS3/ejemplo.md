# Ejemplo: conexión entre red física (192.168.78.0/24) y VirtualBox (192.168.99.0/24)

## Resumen
Objetivo: interconectar la topología local (XX=78) y la topología en VirtualBox (YY=99) usando `R4` de cada lado como punto de enlace mediante una red de tránsito.

- Red local: 192.168.78.0/24
- Red VirtualBox: 192.168.99.0/24
- Red de tránsito recomendada: 10.10.10.0/30 (10.10.10.1 <-> 10.10.10.2)

## Requisitos previos
- Ambos R4 deben estar encendidos en GNS3 y conectados al mismo Cloud/segmento L2 del host (o mediante un puente host-only que conecte los dos R4).
- Asegúrate de que no haya conflicto de IP en la interfaz `Fa0/1` (no usar DHCP si vas a asignar estático).

## Pasos (comandos para pegar en cada R4)

1) Configurar la interfaz de tránsito en Fa0/1

- En R4 (red 192.168.78.0/24 — tu PC):
```
conf t
interface FastEthernet0/1
 ip address 10.10.10.1 255.255.255.252
 no shutdown
end
wr
```

- En R4 (VirtualBox — 192.168.99.0/24):
```
conf t
interface FastEthernet0/1
 ip address 10.10.10.2 255.255.255.252
 no shutdown
end
wr
```

2) Añadir rutas estáticas entre R4s

- En R4 (78):
```
conf t
ip route 192.168.99.0 255.255.255.0 10.10.10.2
end
wr
```

- En R4 (99):
```
conf t
ip route 192.168.78.0 255.255.255.0 10.10.10.1
end
wr
```

3) Añadir rutas de retorno en R3s (o usar default hacia R3 si ya está configurado)

- En R3 (topología 78):
```
conf t
ip route 192.168.99.0 255.255.255.0 192.168.78.194
end
wr
```

- En R3 (topología 99):
```
conf t
ip route 192.168.78.0 255.255.255.0 192.168.99.194
end
wr
```

(Opcional) Si R1/R2 no tienen ruta por defecto hacia R3, agregar en R1/R2 la ruta hacia la red remota vía 192.168.78.131 / 192.168.99.131 según corresponda.

## Verificaciones
- En ambos R4:
```
show ip interface brief
ping 10.10.10.2   # desde 10.10.10.1
ping 10.10.10.1   # desde 10.10.10.2
show ip route
```
- Desde un host de la red 78 (por ejemplo 192.168.78.10):
```
ping 192.168.99.10
traceroute 192.168.99.10
```

## Troubleshooting rápido
- Si `ping 10.10.10.2` falla: verifica que Fa0/1 de ambos R4 estén en el mismo segmento L2 (Cloud/bridge) y `no shutdown`.
- Si Fa0/1 obtiene `192.168.1.56` por DHCP en vez de la IP estática: revisa que la configuración está cargada y que aplicaste `wr` (o reinicia la interfaz).
- Si R3 no tiene la ruta de retorno: añade la ruta estática en R3 como se indica en el paso 3.
- Usa `show ip route` en cada router para validar que la ruta hacia la red remota exista y apunte al siguiente salto correcto.

## Notas finales
- Puedes usar otra subred de tránsito (por ejemplo 10.10.20.0/30) si ya existe 10.10.10.0 en tu host.
- Si conectas mediante VirtualBox Host-only o puente, asegúrate de que no haya firewalls en el host que bloqueen la comunicación entre las dos interfaces.

---
Archivo creado: ejemplo.md

## Cumplimiento del numeral (e)

Objetivo: cada topología debe poder alcanzar las topologías de los compañeros desde cualquier punto de su red, usando una sola ruta estática por *topología vecina* en el router que hace de salida (R4).

- En `R4` (por cada compañero) **agrega una ruta estática** hacia la red del compañero usando la IP del Cloud/peer como next-hop. Si hay 7 estudiantes, en tu `R4` tendrás 6 rutas (una por cada compañero). Ejemplo:

```
conf t
ip route 192.168.YY1.0 255.255.255.0 <IP_CLOUD_COMPANERO_1>
ip route 192.168.YY2.0 255.255.255.0 <IP_CLOUD_COMPANERO_2>
ip route 192.168.YY3.0 255.255.255.0 <IP_CLOUD_COMPANERO_3>
ip route 192.168.YY4.0 255.255.255.0 <IP_CLOUD_COMPANERO_4>
ip route 192.168.YY5.0 255.255.255.0 <IP_CLOUD_COMPANERO_5>
ip route 192.168.YY6.0 255.255.255.0 <IP_CLOUD_COMPANERO_6>
end
wr
```

- En los routers internos de tu topología (`R1`, `R2`) **usa una sola ruta por defecto** hacia `R3` para que todo el tráfico hacia redes externas se encamine por `R3` (no hace falta una ruta por cada compañero en R1/R2):

```
conf t
ip route 0.0.0.0 0.0.0.0 192.168.78.131
end
wr
```

- En `R3` añade **una ruta estática por cada topología vecina** que tu R4 conoce, apuntando al `siguiente salto` R4 (esto cumple "una ruta estática por router para conectarse con una topología vecina"). Ejemplo:

```
conf t
ip route 192.168.YY1.0 255.255.255.0 192.168.78.194
ip route 192.168.YY2.0 255.255.255.0 192.168.78.194
! repetir una línea por cada topología vecina
end
wr
```

Notas clave:
- Sólo `R4` necesita conocer las rutas hacia las redes remotas (una por cada red vecina). Internamente, `R3` puede tener una ruta por cada vecina o reenviarlas hacia `R4` según diseño.
- `R1` y `R2` evitan tener N rutas estáticas si usan la `default` hacia `R3`.
- Sustituye `192.168.YYx.0` y `<IP_CLOUD_COMPANERO_x>` por las redes e IPs reales de tus compañeros.

Con esto cumples el numeral (e): cada topología se conecta con las demás usando únicamente una ruta estática por topología vecina en los routers de salida (`R4`) y mantienes configuraciones simples en los routers internos.

## Caso real aplicado: 78 (física) <-> 99 (VirtualBox)

Sí, el escenario que hiciste (tu red `192.168.78.0/24` con la de tu compañero `192.168.99.0/24`) **sí cumple** el numeral (e) para el caso de 1 vecino.

Interpretación correcta del requisito en tu caso:
- Tienes 1 topología vecina (la 99).
- Entonces en tu `R4` necesitas 1 ruta estática hacia `192.168.99.0/24`.
- En el `R4` de tu compañero, 1 ruta estática hacia `192.168.78.0/24`.

Configuración mínima exacta para ese caso:

En R4 de la topología 78:
```
conf t
interface FastEthernet0/1
 ip address 10.10.10.1 255.255.255.252
 no shutdown
ip route 192.168.99.0 255.255.255.0 10.10.10.2
end
wr
```

En R4 de la topología 99 (VirtualBox):
```
conf t
interface FastEthernet0/1
 ip address 10.10.10.2 255.255.255.252
 no shutdown
ip route 192.168.78.0 255.255.255.0 10.10.10.1
end
wr
```

Rutas internas para alcanzar "desde cualquier punto" de la red:

En R3 de la topología 78:
```
conf t
ip route 192.168.99.0 255.255.255.0 192.168.78.194
end
wr
```

En R3 de la topología 99:
```
conf t
ip route 192.168.78.0 255.255.255.0 192.168.99.194
end
wr
```

En R1 y R2 (de cada topología), si no tienen default, agrega:
```
conf t
ip route 0.0.0.0 0.0.0.0 <IP_DE_R3_EN_SWITCH1>
end
wr
```

Con eso, tu ejemplo 78 <-> 99 queda alineado con el numeral (e). Si luego agregas más compañeros, sólo agregas una ruta más por cada nueva topología vecina en `R4` (y su respectiva propagación interna).
