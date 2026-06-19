R1 (LAN izquierda)

```bash
conf t
interface f0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface s0/0
 ip address 10.0.12.1 255.255.255.252
 no shutdown

interface s0/1
 ip address 10.0.13.1 255.255.255.252
 no shutdown

ip route 0.0.0.0 0.0.0.0 10.0.12.2
end
wr
```

R2 (intermedio)
```bash
conf t
interface s0/0
 ip address 10.0.12.2 255.255.255.252
 no shutdown

interface s0/1
 ip address 10.0.24.1 255.255.255.252
 no shutdown

ip route 0.0.0.0 0.0.0.0 10.0.24.2
end
wr
```

R4 (NAT + Internet — CRÍTICO)
```bash
conf t

! -------------------------
! INTERFACES
! -------------------------
interface f0/0
 ip address 192.168.40.1 255.255.255.0
 ip nat inside
 no shutdown

interface s0/0
 ip address 10.0.24.2 255.255.255.252
 ip nat inside
 no shutdown

interface s0/1
 ip address 10.0.34.2 255.255.255.252
 ip nat inside
 no shutdown

interface f0/1
 ip address dhcp
 ip nat outside
 no shutdown

! -------------------------
! RUTAS
! -------------------------
ip route 0.0.0.0 0.0.0.0 192.168.1.254

! -------------------------
! ACL NAT
! -------------------------
no access-list 1
access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.40.0 0.0.0.255
access-list 1 permit 10.0.12.0 0.0.0.255
access-list 1 permit 10.0.24.0 0.0.0.255
access-list 1 permit 10.0.13.0 0.0.0.255
access-list 1 permit 10.0.34.0 0.0.0.255

! -------------------------
! NAT OVERLOAD
! -------------------------
no ip nat inside source list 1 interface f0/1 overload
ip nat inside source list 1 interface f0/1 overload

end
wr
```

VERIFICACIÓN RÁPIDA

Después de todo:

En R2:
```bash
ping 10.0.24.2
ping 8.8.8.8
```

En R1:
```bash
ping 8.8.8.8
```

En R4:
```bash
show ip nat translations
show ip nat statistics
```

En R3:
```bash
conf t

interface s0/0
 ip address 10.0.13.2 255.255.255.252
 no shutdown

interface s0/1
 ip address 10.0.34.1 255.255.255.252
 no shutdown

#Rutas (si estás usando routing dinámico no haría falta, pero en tu caso usas estático)

ip route 192.168.10.0 255.255.255.0 10.0.13.1
ip route 192.168.40.0 255.255.255.0 10.0.34.2
ip route 10.0.12.0 255.255.255.252 10.0.13.1
ip route 10.0.24.0 255.255.255.252 10.0.34.2
end
wr
```
```bash
show ip interface brief
ping 10.0.13.1
ping 10.0.34.2
```


⚡ IMPORTANTE (para que NO falle otra vez)

✔ SOLO R4 hace NAT
✔ Solo F0/1 es outside
✔ Seriales siempre inside
✔ Default route SOLO en R4
