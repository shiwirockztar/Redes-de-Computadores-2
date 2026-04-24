# Preguntas y respuestas - Parcial

Este documento organiza las dudas frecuentes del parcial y su explicacion tecnica.

## 1) ¿Por que al apagar G0/1 en R1 falla el ping entre VLAN de administracion y VLAN 10?

### Situacion observada

- Si se elimina o apaga G0/1 en R1, el ping entre un PC de administracion y un PC en VLAN 10 falla.
- Si se elimina o apaga G0/0 en R1, ese mismo ping puede seguir funcionando.

### Explicacion tecnica

Segun el diseno consolidado del parcial (puntos 1-2 hasta 6), el rol de interfaces en los routers es este:

- G0/0: Router-on-a-Stick para VLAN 10, 20 y 30 (subinterfaces dot1Q).
- G0/1: interfaz fisica dedicada a VLAN 40 (administracion), sin subinterfaces.

Por eso, si se apaga G0/1 en R1, R1 pierde la salida fisica que usa para la VLAN 40. Aunque exista R2 como respaldo, ese respaldo solo funciona si R2 esta configurado para tomar el rol de gateway de esa VLAN y la topologia ya convergio. Si eso no pasa todavia, el ping falla.

En cambio, apagar G0/0 en R1 no siempre tumba inmediatamente ese ping en todos los casos, porque el trafico de VLAN 10 puede seguir entrando por la ruta de respaldo que tiene HSRP con R2. Es decir, el respaldo existe para el gateway virtual, no para cualquier interfaz apagada.

Resumen simple: R2 no es un respaldo "magico". Solo toma el control si la VLAN afectada tiene su gateway virtual, su configuracion HSRP y su camino de capa 2 y 3 listos para hacerlo.

### Ejemplo facil de envio de tramas

Supongamos que un PC de administracion quiere hacer ping a un PC en VLAN 10:

1. El PC de administracion crea una trama Ethernet con:
	- MAC origen: la MAC del PC.
	- MAC destino: la MAC del gateway virtual de VLAN 40 o del router activo.
	- IP origen: la IP del PC de administracion.
	- IP destino: la IP del PC en VLAN 10.
2. El switch envia esa trama hacia el router que tenga activo el gateway.
3. Si G0/1 de R1 esta activo, R1 recibe la trama, la enruta hacia VLAN 10 y la vuelve a encapsular con una nueva trama para esa VLAN.
4. El PC de VLAN 10 responde con otra trama de vuelta hacia su gateway.
5. Si G0/1 de R1 esta apagado, pero R2 no ha tomado correctamente el rol de gateway de VLAN 40, la primera trama ya no tiene por donde salir y el ping falla.
6. Si R2 si tomo el rol de respaldo, entonces la trama entra por R2, cruza la ruta activa y el ping sigue funcionando.

## 2) Verificacion rapida recomendada

En R1:

1. `show ip interface brief`
2. `show running-config interface g0/1`
3. `show running-config interface g0/0.10`
4. `show running-config interface g0/0.20`
5. `show running-config interface g0/0.30`
6. `show standby brief`
7. `show ip arp`
8. `show ip route`

En el switch conectado a R1:

1. `show interfaces trunk`
2. `show vlan brief`
3. `show etherchannel summary`
4. `show spanning-tree vlan 10`
5. `show spanning-tree vlan 40`

## 3) ¿Que debe estar correcto para que funcione?

- Subinterfaces o interfaz de gateway en estado up/up.
- `encapsulation dot1Q` correcta por VLAN (si aplica).
- IP de gateway correcta en cada VLAN.
- Puerto hacia G0/0 en modo trunk con VLAN 10,20,30 permitidas y nativa coherente.
- Puerto hacia G0/1 en modo access VLAN 40 (sin subinterfaces para VLAN 40).
- Puerta de enlace correcta configurada en los PCs.
- HSRP operativo y con prioridades segun el parcial (R1 principal en 10/30 y R2 principal en 20/40).
- Consistencia con STP/EtherChannel para evitar bloqueos o caminos inesperados en la capa 2.

## 4) Coherencia con la secuencia del parcial (README 1-2 a 6)

- Punto 1-2: base de VLAN, troncales, gestion y acceso remoto.
- Punto 3: control del arbol STP por VLAN.
- Punto 4: capacidad con EtherChannel para evitar saturacion.
- Punto 5: direccionamiento exclusivo 10.10.8.0/24.
- Punto 6: inter-VLAN redundante con dos routers y HSRP, con VLAN 40 por G0/1 fisica.

La interpretacion de esta pregunta debe leerse con esa secuencia completa, no como un caso aislado.

## 5) Conclusión

El comportamiento reportado es coherente con el diseno: al apagar la interfaz que presta servicio de gateway a las VLAN involucradas, el trafico inter-VLAN deja de funcionar. Si se apaga una interfaz no usada en ese trayecto, el ping se mantiene.

## 6) Respuesta corta para sustentacion oral

"En este parcial, G0/0 en los routers transporta VLAN 10, 20 y 30 por subinterfaces dot1Q, mientras que VLAN 40 va por G0/1 fisica, sin subinterfaces. Si bajo G0/1 en R1, elimino la salida directa de R1 para administracion. R2 solo reemplaza ese camino si HSRP ya le dio el rol activo y la red convergio. Si bajo G0/0 en R1, no siempre se cae todo porque el gateway virtual puede seguir por R2. En resumen, no basta con tener un respaldo: ese respaldo debe estar activo, configurado y alcanzable por capa 2 y 3." 

## 7) Como se podria corregir si falla el ping

Si al apagar o recuperar interfaces el ping entre VLAN 10 y VLAN 40 no responde, aplicar esta secuencia:

1. Verificar estado de interfaces en ambos routers:
	- `show ip interface brief`
	- Corregir interfaces caidas con `no shutdown` en `g0/0`, subinterfaces `g0/0.x` y `g0/1` segun el caso.

2. Validar configuracion de VLAN por interfaz en router:
	- En G0/0 deben existir subinterfaces de VLAN 10,20,30 con `encapsulation dot1Q` correcta.
	- En G0/1 debe existir IP de VLAN 40 fisica, sin subinterfaces.

3. Confirmar HSRP y roles esperado del parcial:
	- `show standby brief`
	- Esperado: R1 activo en VLAN 10/30 y R2 activo en VLAN 20/40 (con `preempt`).

4. Revisar puertos de switch hacia routers:
	- Puerto hacia G0/0: `switchport mode trunk`, VLAN 10,20,30 permitidas, nativa coherente.
	- Puerto hacia G0/1: `switchport mode access`, `switchport access vlan 40`.
	- Comandos utiles: `show interfaces trunk`, `show vlan brief`.

5. Verificar capa 2 del parcial (STP/EtherChannel):
	- `show spanning-tree vlan 10`
	- `show spanning-tree vlan 40`
	- `show etherchannel summary`
	- Corregir puertos en estado inconsistente, bloqueo inesperado o channel-group mal armado.

6. Confirmar direccionamiento de hosts y gateways:
	- Host VLAN 10 con gateway `10.10.8.1`.
	- Host VLAN 40 con gateway `10.10.8.193`.
	- Todo dentro de `10.10.8.0/24`.

7. Limpiar aprendizaje temporal y reintentar prueba:
	- `clear arp` (router)
	- repetir ping entre hosts de VLAN 10 y 40.

Regla practica: primero recupera capa 1/2 (interfaces, trunk, access, STP, EtherChannel) y luego capa 3 (subinterfaces, IPs y HSRP). Asi se resuelve mas rapido y con menor riesgo de cambiar algo que ya estaba bien.