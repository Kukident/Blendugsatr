# Welcome to Blendugsatr project

# Semana 1

En esta primera semana los elementos a implementar son los siguientes:

1. Un ISP
	1. Con redundancia 
	2. Encaminamiento dinámico
2. La sede principal de una empresa
	1. Tres subredes distintas (Data Center, Servicios Online e intranet)
	2. Asignación de parámetros de red automático
	3. Configuración de NAT
3. Una sede secundaria conectada a otro router frontera
	1. Asignación de parámetros de red automático
	2. Configuración de NAT


# Semana 2

En esta segunda semana la cosa se complica. Además de lo ya implementado anteriormente hay que implementar:

1. Redundancia y protección ante caídas en la sede principal
2. Conexión con un ISP de backup
3. Dos VPNs entre las sedes
	1. Una Trusted VPN a través del ISP principal
	2. Una VPN corporativa a través del ISP secundario

# Semana 3

En esta tercera y ultima semana es necesario implementar ademas de todo lo anterior:

1. Implementar IPv6 mediante 6rd en el ISP 2
2. Acceso mediante xDSL

## Enunciados

### Semana 1

La empresa Blendugsargh se dedica a la venta de BlendugsThings. El negocio va bien y son ya 4 sedes las que tiene diseminadas en su país de origen, Blendugsland. Para su conectividad han elegido una topología «hub and spoke,» y han contratado el servicio de red al operador BlendungsComm (que también operará como ISP).

En la sede principal, se establecen tres subredes diferentes para dar cobijo a su Data Center, servicios online e intranet corporativa, respectivamente. La empresa contrata con el RIPE el uso del prefijo de red 203.0.112.0/23 (que utilizará en la vlan en la que ofrece sus servicios al exterior), así como el identificador de sistema autónomo 64501 y utiliza direccionamiento privado para sus otras subredes. En las demás sucursales, solo implementan un subconjunto de la intranet corporativa.

Implementad un escenario que comprenda la sede principal, una sucursal y el operador de red contratado. La maqueta debe incluir al menos dos equipos finales por cada subred. Además, la red del operador debe presentar redundancia con encaminamiento dinámico, y la asignación de los parámetros de red en las redes internas debe ser automático.

Varias aclaraciones:
a) Recomendamos usar gns3 para hacer la simulación, utilizando
diversos modelos de routers haciendo labores de equipos finales y/o switches.
c) Podéis emplear cualquier tipo de enlace para unir los dispositivos
d) La simulación puede ampliarse con interfaces de red nativas para conectar con otros simuladores y/o máquinas virtuales o incluso procesos en vuestro propio equipo. ¡No hay límite!


### Semana 2

El modelo de negocio de nuestra empresa se basa principalmente en la venta online, y los recursos
que se han dispuesto para ello (servidores, bases de datos, ...) residen exclusivamente en la sede principal.
En consecuencia, la disponiblidad en este punto es crítica.

Para evitar quedarnos sin conectividad por algún fallo en la infraestructura de comunicaciones, en la sede
principal se duplican los equipos de red (switches/routers) proveyendo redundancia que soporte al menos
la caída de uno de dichos dispositivos.

En la misma línea, se contrata con dos proveedores de servicio de red la conexión a Internet, de tal forma
que uno de ellos actuará de backup en caso de que el ISP principal (ISP1) deje de dar servicio. Al ISP primario –que
implementa MPLS en su infraestructura– también se le contrata un servicio de VPN corporativo (Trusted VPN), que
posibilite la visibilidad de las subredes internas del Hub (intranet y data center) en cada una de las sucursales.

Como también es importante no dejar sin conectividad con la sede principal a los trabajadores de las sucursales
(en caso de fallo del ISP1), los ingenieros telemáticos de la empresa implementan per se (ahorro de costes)
el servicio de VPN corporativo usando el acceso a Internet alternativo contratado con el ISP2.

+ Tareas mínimas a realizar:
    - Redundancia de salida en la sede
    - VPN MPLS
    -  Túneles a través del ISP 2 entre todas las sedes

+ Tareas opcionales:
    - Salida a Internet de las sucursales sólo a través de la sede
    - Uso de GRE multipunto para los túneles a través del ISP2 (sin IPSEC)
    - Uso de GRE multipunto para los túneles a través del ISP2 (con IPSEC)
    - Acceso a Internet a través de la VPN MPLS (adicional a la del ISP2)


##### Palabras clave:

VRRP/HSRP, MPLS, vrf, mgre/nhrp, ipsec, BGP

Información sobre GRE multipunto:
http://www.cisco.com/c/en/us/support/docs/security-vpn/ipsec-negotiation-ike-protocols/41940-dmvpn.html

### Semana 3

#### Despliegue IPv6

+ El ISP2, que utilizamos como backup para la sede principal y para alguna/s de la/s sucursales, consciente de la competencia de otros operadores, decide ofrecer a sus clientes la posibilidad de disfrutar de conectividad IPv6. El servicio, que comercializan bajo el lema «InjOy-iT, IoT is in da house», es ofrecido sin aumento de tarifa para los usuarios/empresas que ya son clientes, y con un coste mínimo adicional para los advenedizos. Tal generosidad obedece realmente a que dicho operador tendrá que hacer mínimos cambios en la infraestructura de su red de transporte (que seguirá siendo una red IPv4) al elegir como solución tecnológica «6rd» (IPv6 Rapid Deployment).

+ Implementa el despliegue de esta tecnología/servicio para que las sucursales de nuestra empresa tengan doble conectividad (IPv4/IPv6) en su red interna. (Los clientes 6rd –CPEs– recibirán un prefijo /56 que subdividirán internamente según sus necesidades; y los terminales deberán obtener su dirección IPv6 de forma dinámica). Considere que para ofrecer el servicio IPv6, el ISP2 obtiene el rango 2001:aaaa::/38 y usa el subrango 2001:aaaa::/40 para el servicio 6rd propiamente dicho. Para probar el servicio, dote de IPv6 también a un tercer ISP para probar la conectivad.

    - (OPCIONAL) Ídem para la sede principal, donde se definirán tantos rangos/subredes IPv6 como redes internas tenga definidas. (Palabras clave: L2TPv3, pseudowire, local switching)

#### Acceso xDSL

+ Maritta Blendugsbürger, trabajadora de la sede principal de Blendugsargh, tiene contratado un acceso xDSL en su domicilio con un ISP que no posee infraestructura propia de comunicaciones cerca de su vecindario. Pero el OBLA, que sí llega hasta su hogar, le aporta el acceso indirecto que necesita a través de un enlace Ethernet entre CPE y NAS, y una red de transporte IP entre el NAS y el correspondiente BBRAS del ISP.

+ Implementa dicho servicio de acceso indirecto y prueba su funcionamiento accediendo al servicio de ventas online de nuestra empresa.

+ Para las autenticaciones de los clientes (CPE) utilizaremos duplas «usuario,password», donde el usuario tendrá la estructura username@ISPdomain (por ejemplo: 7749@isp99.com); utilizando la segunda parte del nombre de usuario para localizar en el LAC (NAS) la IP del LNS (BBRAS), y la primera para propiamente autenticar al cliente ante el ISP.

    - (OPCIONAL) Para facilitar el teletrabajo, nuestra empresa implementa un *servicio VPN de acceso remoto en la sede principal.

	     Implementa la solución «vpdn» en el servidor/concentrador de vpns del hub, y pruébala usando cualquier cliente ppp ya preinstalado en el terminal que utilizará Maritta para conectarse remotamente a su intranet.

    - (OPCIONAL) Utiliza un servidor AAA (freeradius) para realizar de forma centralizada las tareas de autenticación (acreditación de los clientes/usuarios) y autorización (p.e., localización de la IP del router de acceso del ISP) inherentes al protocolo PPP y al servicio vpdn.

        Acerca del servidor freeradius: Configurad el servidor freeradius en una máquina Linux (en un ordenador real que esté conectado a Internet, en una máquina virtual que conectéis en algún enclave de la red de nuestro ISP, o en el propio equipo donde ejecutáis la simulación).

##### Acrónimos:
IoT Internet of Things
OBLA Operador del Bucle Local de Abonados
NAS Network Access Server (véase como el BRAS del operador de acceso en una solución PPPoEoE con BRAS dual)
BBRAS BroadBand Remote Access Server
LAC L2TP Access Concentrator
LNS L2TP Network Server
AAA Authentication, Authorization, Accounting
vpdn virtual private data network
CPE Customer Premise Equipment