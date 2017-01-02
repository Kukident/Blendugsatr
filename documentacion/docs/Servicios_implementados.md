#Trusted VPN

 Hemos implementado una trusted VPN entre las distintas sedes de la empresa. Los pasos a seguir son los siguientes:

1) Creamos dos subinterfaces en el enlace entre frontera y cliente.

```bash
interface XX.Y
ip address {IP que queramos usar para tráfico normal} {Máscara}

interface XX.Z
ip address {IP de la conexión BGP para la VRF} {Máscara}
```

2) Configuramos MPLS en las interfaces de los routers del ISP:
```bash
interface XX
mpls ip

#Comandos de comprobación
show mpls interfaces
show mpls ldp neighbor
```

3) Creamos las VRFs en los routers frontera:

```bash
#Repetimos estos comandos en todos los routers frontera que participen en la VPN.
ip vrf {Nombre de la VRF}
rd XXXXX:XX
route-target both XXXXX:XX
```

4) Asignamos las VRFs a las interfaces del router:

```bash
interface XX.Z
ip vrf forwarding {Nombre de la VRF}

#Debemos asignar de nuevo la ip a la interfaz
ip add {IP} {Máscara}
```

5) Configuramos BGP entre los routers frontera.

```bash
router bgp {identificador del AS}

neighbor {IP de la loopback del vecino} remote-as {identificador del AS}
neighbor {IP de la loopback del vecino} update-source loopback 0

address-family vpnv4
neighbor {IP de la loopback del vecino} activate

#Repetimos en el otro vecino
```

6) Configuramos BGP entre los routers frontera y el router del cliente.

```bash
#En el router frontera
router bgp {identificador del AS}

address-family ipv4 vrf Blendugs
neighbor {IP de la subinterfaz VRF del vecino} remote-as {ID del AS del vecino}
neighbor {IP de la subinterfaz VRF del vecino} activate

#En el router del cliente
router bgp {Identificador del AS}

#Se repite tantas veces como redes se quieran compartir
network {Prefijo de la VPN} mask {Máscara del prefjo}

neighbor {IP de la subinterfaz VRF del router frontera} remote-as {ID del AS del ISP}
```

Opcional: Filtrado de las rutas que compartes con cada vecino

```bash

#En el router del cliente

#Rutas que NO queremos enviar, una línea por cada
access-list {1-99} deny {Prefijo que no queremos enviar} {Wildcard de la máscara}
access-list {1-99} deny {Prefijo que no queremos enviar} {Wildcard de la máscara}
access-list {1-99} permit any

router bgp {ID del AS}

#En caso de no haberlo hecho ya añadir todas las rutas que queramos transmitir
network {Prefijo a transmitir} mask {Máscara del prefijo}

#Le asignamos una lista de distribución que filtre las direcciones que se le envían a ese vecino
neighbor {IP del vecino} distribute-list {1-99} out
```

#VPN GRE Multipunto con IPSEC

Spoke Router
```bash

SedeDerecha(config)#crypto isakmp policy 1
SedeDerecha(config-isakmp)#authentication pre-share
SedeDerecha(config-isakmp)#exit
SedeDerecha(config)#crypto isakmp key cisco47 address 0.0.0.0 0.0.0.0
SedeDerecha(config)#crypto ipsec transform-set trans2 esp-des esp-md5-hmac
SedeDerecha(cfg-crypto-trans)#mode transport
SedeDerecha(cfg-crypto-trans)#exit
SedeDerecha(config)#crypto ipsec profile vpnprof
SedeDerecha(ipsec-profile)#set transform-set trans2
SedeDerecha(ipsec-profile)#exit

SedeDerecha(config)#interface tunnel 0
SedeDerecha(config-if)#bandwidth 1000
SedeDerecha(config-if)#ip address 192.168.0.3 255.255.255.0
SedeDerecha(config-if)#ip mtu 1400
SedeDerecha(config-if)#ip nhrp authentication test
SedeDerecha(config-if)#ip nhrp map multicast 30.0.0.2
SedeDerecha(config-if)#ip nhrp map 192.168.0.1 30.0.0.2
SedeDerecha(config-if)#ip nhrp network-id 100000
SedeDerecha(config-if)#ip nhrp holdtime 300
SedeDerecha(config-if)#ip nhrp nhs 192.168.0.1
SedeDerecha(config-if)#ip ospf network broadcast
SedeDerecha(config-if)#ip ospf priority 0
SedeDerecha(config-if)#delay 1000
SedeDerecha(config-if)#tunnel source fastEthernet 1/0
SedeDerecha(config-if)#tunnel mode gre multipoint
SedeDerecha(config-if)#tunnel key 100000
SedeDerecha(config-if)#tunnel protection ipsec profile vpnprof
SedeDerecha(config-if)#exit

SedeDerecha(config)#router ospf 1
SedeDerecha(config-router)#network 10.0.0.0 0.0.0.255 area 0
SedeDerecha(config-router)#network 192.168.0.0 0.0.0.255 area 0
SedeDerecha(config-router)#exit

```

Falta añadir la del HUB Router


Fuente: http://www.cisco.com/c/en/us/support/docs/security-vpn/ipsec-negotiation-ike-protocols/41940-dmvpn.html

#Elección del router de entrada a la sede principal

 Dado que tenemos dos routers desde los que se puede entrar a la sede debemos tener algún mecanismo para indicar por cuál queremos que entre el trafico. 
El método que vamos a usar es modificar el AS-PATH que comparte el router frontera de backup, para que así tenga un coste superior a las rutas compartidas desde el otro. Estas rutas tendrán un coste superior
porque el primer parámetro que examina BGP para calcular el coste es el AS-PATH, no el número de saltos.

1. Creamos un route-map en el router de backup
    
        route-map {Nombre} permit {Número, por ejemplo 10}

2. Dentro del route-map le indicamos que concatene varias veces nuestro número de AS en el AS-PATH

        set as-path prepend 64501 64501 64501

3. A continuación entramos en BGP, y le indicamos que use con el vecino que queramos el route-map que acabamos de crear.

        router bgp {ID del AS}
            neighboor {IP del vecino} route-map {Nombre del route-map}

#Incorporación de un Firewall en la sede principal

 El objetivo de este equipo es completar nuestros servicios orientados a 
la protección antifallos de la sede principal. Nos va a pemitir mantener
la conectividad con el exterior incluso cuando se caiga la conexión con 
uno de los ISPs (ya sea porque se caiga el PE del ISP o nuestro CPE). Para conseguirlo hemos seguido estos pasos:

1. Creamos una red /29 entre el firewall y ambos CPE, usando direcciones de nuestro
pool público
        

2. Configuramos una conexión BGP entre cada CPE y el firewall

        router bgp 64501
            neighbor 203.0.113.9 remote-as 64501
            neighbor 203.0.113.9 next-hop-self


3. En el caso del CPE principal redistribuimos las rutas a direcciones
privadas, ya que son las pertenecientes a la trusted VPN.

        access-list 11 permit 192.168.0.0 0.0.255.255
        access-list 11 permit 10.0.0.0 0.255.255.255
        access-list 11 permit 172.16.0.0 0.15.255.255
        access-list 11 permit 169.254.0.0 0.0.255.255
        access-list 11 deny   any

        router bgp 64501
            neighbor 203.0.113.9 distribute-list 11 out

4. En el caso del CPE secundario también redistribuimos algo *Peite completa aquí*

        router bgp 64501
             bgp redistribute-internal

5. Por último configuramos HSRP en ambos CPEs para protegernos ante caídas de uno de ellos

 Con esas configuraciones conseguimos que el tráfico saliente a una dirección pública vaya por el router adecuado (HSRP) y que el tráfico de la VPN vaya por la principal
 siempre que sea posible