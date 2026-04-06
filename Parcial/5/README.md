# Parcial 5 - Direccionamiento exclusivo 10.10.8.0/24

## 1) Objetivo
Continuando con los puntos anteriores, toda la topologia debe operar unicamente con la red:

- 10.10.8.0/24

Restriccion obligatoria:

- No se pueden usar recursos de otras redes (ni subredes fuera de 10.10.8.0/24, ni gateways externos, ni DNS externos).

## 2) Criterio de diseno

Como ya vienes trabajando con VLAN 10, 20, 30 y 40, una asignacion limpia es dividir el /24 en 4 subredes /26.

Ventaja:

- Se usan solo bloques internos de 10.10.8.0/24.
- Se separa trafico por VLAN.
- Se mantiene orden para gestion y pruebas.

## 3) Plan de direccionamiento propuesto (VLSM)

| VLAN | Uso sugerido | Subred | Rango de hosts | Gateway sugerido |
|---|---|---|---|---|
| 10 | Usuarios | 10.10.8.0/26 | 10.10.8.1 - 10.10.8.62 | 10.10.8.1 |
| 20 | Usuarios | 10.10.8.64/26 | 10.10.8.65 - 10.10.8.126 | 10.10.8.65 |
| 30 | Usuarios/Nativa | 10.10.8.128/26 | 10.10.8.129 - 10.10.8.190 | 10.10.8.129 |
| 40 | Administracion | 10.10.8.192/26 | 10.10.8.193 - 10.10.8.254 | 10.10.8.193 |

Mascara para todas las VLAN: 255.255.255.192

## 4) Asignacion sugerida para switches (gestion VLAN 40)

- SW1: 10.10.8.194/26
- SW2: 10.10.8.195/26
- SW3: 10.10.8.196/26
- SW4: 10.10.8.197/26
- SW5: 10.10.8.198/26
- SW6: 10.10.8.199/26
- Gateway de gestion: 10.10.8.193

## 5) Ejemplo de configuracion (SVI de administracion)

En cada switch (cambiando IP segun corresponda):

```bash
enable
configure terminal
interface vlan 40
 ip address 10.10.8.194 255.255.255.192
 no shutdown
exit
ip default-gateway 10.10.8.193
end
copy running-config startup-config
```

## 6) Ejemplo de hosts en acceso

- VLAN 10: 10.10.8.10/26, gateway 10.10.8.1
- VLAN 20: 10.10.8.70/26, gateway 10.10.8.65
- VLAN 30: 10.10.8.130/26, gateway 10.10.8.129
- VLAN 40: 10.10.8.200/26, gateway 10.10.8.193

## 7) Regla de aislamiento respecto a otras redes

Para cumplir el enunciado:

- No configures rutas por defecto hacia redes externas.
- No uses servidores DNS fuera de 10.10.8.0/24.
- No asignes IPs de redes como 192.168.x.x, 172.16.x.x o 10.10.9.0/24.
- Todas las pruebas de conectividad deben hacerse entre equipos dentro de 10.10.8.0/24.

## 8) Verificacion

En switches:

```bash
show ip interface brief
show running-config | include ip default-gateway
show running-config | include ip route
```

En PCs:

- Revisar IP, mascara y gateway de su VLAN.
- Probar ping a su gateway.
- Probar ping entre hosts permitidos del mismo esquema interno.

Checklist final:

- Todas las IP pertenecen a 10.10.8.0/24.
- Cada VLAN usa su /26 asignado.
- No hay rutas o recursos configurados fuera de la red permitida.
