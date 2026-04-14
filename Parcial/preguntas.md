# Preguntas y respuestas - Parcial

Este documento organiza las dudas frecuentes del parcial y su explicacion tecnica.

## 1) ¿Por que al apagar G0/1 en R1 falla el ping entre VLAN de administracion y VLAN 10?

### Situacion observada

- Si se elimina o apaga G0/1 en R1, el ping entre un PC de administracion y un PC en VLAN 10 falla.
- Si se elimina o apaga G0/0 en R1, ese mismo ping puede seguir funcionando.

### Explicacion tecnica

Esto es normal cuando la interfaz que transporta el enrutamiento inter-VLAN (router-on-a-stick o enlace de gateway para esas VLAN) es G0/1.

- Si G0/1 contiene las subinterfaces de VLAN (por ejemplo .10 y la de administracion), al caer esa interfaz se pierde el gateway de esas redes.
- Sin gateway, no hay enrutamiento entre VLANs, por eso el ping falla.
- Si G0/0 no participa en ese flujo local entre VLAN 10 y administracion, su caida no afecta ese ping especifico.

Resumen: la conectividad entre VLAN 10 y administracion depende de la interfaz donde viven sus gateways (fisicos o subinterfaces), no de cualquier interfaz del router.

## 2) Verificacion rapida recomendada

En R1:

1. `show ip interface brief`
2. `show running-config interface g0/1`
3. `show running-config interface g0/1.10`
4. `show running-config interface g0/1.40`
5. `show ip route`

En el switch conectado a R1:

1. `show interfaces trunk`
2. `show vlan brief`

## 3) ¿Que debe estar correcto para que funcione?

- Subinterfaces o interfaz de gateway en estado up/up.
- `encapsulation dot1Q` correcta por VLAN (si aplica).
- IP de gateway correcta en cada VLAN.
- Puerto del switch hacia R1 en modo trunk (si aplica) y con VLAN 10 + VLAN de administracion permitidas.
- Puerta de enlace correcta configurada en los PCs.

## 4) Conclusión

El comportamiento reportado es coherente con el diseno: al apagar la interfaz que presta servicio de gateway a las VLAN involucradas, el trafico inter-VLAN deja de funcionar. Si se apaga una interfaz no usada en ese trayecto, el ping se mantiene.