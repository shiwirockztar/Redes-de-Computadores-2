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
