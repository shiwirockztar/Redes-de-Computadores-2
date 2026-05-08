1 - **Respecto a las rutas configuradas en R4 para llegar a LAN A es correcto afirmar * 

La opción correcta es: B.

: en ese segmento Ethernet compartido (R1-R2-R3), usar next-hop evita ambigüedad y ARP innecesario al destino final; se resuelve la MAC del vecino correcto (por ejemplo R1 para llegar a LAN_A).

opciones

A-Es más sencillo configurarlas con interfaz de salida porque la interfaz que conecta a R3 con R2 no es multiacceso
B-Deben configurarse con siguiente salto, porque la interfaz que conecta a R3 con R2 es multiacceso
C-Cuando R3 reciba un paquete con una IP destino en la LAN1 no realizará resolución ARP para conocer la MAC del siguiente salto sino que enviará inmediatamente el paquete a través de su interfaz serial
D-Es necesario configurar mas de una ruta en R3 para llegar a LAN1



2- **Si se configuran rutas  por defecto en R3 y R4 para llegar a LAN A: *

La opción correcta es: B.

: Si en R3 y R4 usas solo rutas por defecto, puede ocurrir un bucle: un destino desconocido en R3 se envía a R4 y, como R4 tampoco lo conoce de forma específica, lo devuelve a R3, quedando el tráfico rebotando entre ambos.


opciones

A-Es la forma más sencilla de habilitar la conectividad entre todos los hosts de la red sin problemas potenciales.
B-Es posible ocasionar un bucle de enrutamiento, pues un paquete que no conozcan específicamente ambos routers se quedará "rebotando" entre ambos.
C-No presentan ningún riesgo de provocar bucles de enrutamiento.
D-Sólamente funcionan si se configuran a través de interfaces seriales




Escriba el esquema de direccionamiento utilizado en la práctica. *

Mi cedula termina en 78 > red base 192.168.78.0/24: 
LAN_A 192.168.78.0/26, 
LAN_B 192.168.78.64/26, 
Switch 192.168.78.128/26, 
Serial R3-R4 192.168.78.192/29  
LAN_C 192.168.78.200/29; 
Gateways :
R1: 192.168.78.1  - 192.168.78.129, 
R2: 192.168.78.65 - 192.168.78.130, 
R3: 192.168.78.131 - 192.168.78.193, 
R4: 192.168.78.194 - 192.168.78.201.


¿Cuántas redes se deben crear para completar el literal b de la práctica (sin incluir la nube)? Escriba solamente el número : 5


