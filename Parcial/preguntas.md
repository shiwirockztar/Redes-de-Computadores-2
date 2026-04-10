me sucede que al borrar la interfaz g0/1 en el router 1 y hacer ping de un pc en vlan de administracion y vlan 10 falla, en cambio si borro la interfaz g0/0 en el router 1 y hacer ping de un pc en vlan de administracion y vlan 10 sale exitosa


Eso que describes es coherente con la topología típica de router-on-a-stick en R1.

La diferencia suele ser esta:

Si borras g0/1 en R1:

Ahí normalmente están las subinterfaces de las VLAN (ejemplo: .10, .99 administración) con encapsulación 802.1Q.
Al eliminarla, desaparece el gateway de esas VLAN.
Resultado: el tráfico entre VLAN administración y VLAN 10 falla.
Si borras g0/0 en R1:

Esa interfaz suele ser hacia otra red/router, no la que hace el enrutamiento entre esas dos VLAN locales.
Mientras g0/1 y sus subinterfaces sigan arriba, el inter-VLAN local sigue funcionando.
Resultado: ping entre administración y VLAN 10 exitoso.
En resumen: el ping entre esas dos VLAN depende de la interfaz trunk con subinterfaces (probablemente g0/1), no de g0/0.

Para comprobarlo rápido en R1:


Qué deberías ver para que funcione:

Subinterfaces de VLAN 10 y administración en estado up/up.
encapsulation dot1Q correcto por VLAN.
IP de gateway correcta en cada subinterfaz.
En el switch, el puerto hacia R1 en trunk permitiendo esas VLAN.
Si quieres, te ayudo a validar tu configuración exacta línea por línea y te digo cuál comando está causando ese comportamiento.