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

Como mejora hemos implementado tuneles gre multipunto en todas las sedes, de esta forma conseguimos que el trafico entre sedes secundarias no pase necesariamente por la sede principal. De esta forma liberamos al router de la sede principal de carga, evitando que tenga que desencriptar, encriptar y reenviar trafico que no va dirigido a el.

##HUB Router
1. Configuramos ipsec

		crypto isakmp policy 1
		 authentication pre-share
		crypto isakmp key cisco47 address 0.0.0.0 0.0.0.0
		crypto ipsec transform-set trans2 esp-des esp-md5-hmac
		 mode transport
		crypto ipsec profile vpnprof
		 set transform-set trans2t

2. Configuramos el tunel

		interface tunnel 0
		 bandwidth 1000
		 ip address 192.168.0.1 255.255.255.0	#IP dentro del tunel del hub router
		 ip mtu 1400
		 ip nhrp authentication test
		 ip nhrp map multicast dynamic
		 ip nhrp network-id 100000
		 ip nhrp holdtime 600
		 ip ospf network broadcast
		 ip ospf priority 2
		 delay 1000
		 tunnel source fastEthernet 0/0
		 tunnel mode gre multipoint
		 tunnel key 100000
		 tunnel protection ipsec profile vpnprof

3. Configuramos ospf que se utilizara dentro del tunel para distribuir las ips de cada sede y las ip del resto de los nodos del tunel

		router ospf 1
		 network 10.0.0.0 0.0.0.255 area 0	#Esto luego sera necesario quitarlo a la hora de incorporar el firewall de la sede principal
		 network 192.168.0.0 0.0.0.255 area 0

##Spoke Router
1. Configuramos ipsec

		crypto isakmp policy 1
		 authentication pre-share
		crypto isakmp key cisco47 address 0.0.0.0 0.0.0.0
		crypto ipsec transform-set trans2 esp-des esp-md5-hmac
		 mode transport
		crypto ipsec profile vpnprof
		 set transform-set trans2t

2. Configuramos el tunel

		interface tunnel 0
		 bandwidth 1000
		 ip address 192.168.0.3 255.255.255.0	#IP dentro del tunel del spoke router
		 ip mtu 1400
		 ip nhrp authentication test
		 ip nhrp map multicast 30.0.0.2		#IP del router backup de la sede
		 ip nhrp map 192.168.0.1 30.0.0.2	
		 ip nhrp network-id 100000
		 ip nhrp holdtime 300
		 ip nhrp nhs 192.168.0.1
		 ip ospf network broadcast
		 ip ospf priority 0
		 delay 1000
		 tunnel source fastEthernet 1/0
		 tunnel mode gre multipoint
		 tunnel key 100000
		 tunnel protection ipsec profile vpnprof

3. Configuramos ospf que se utilizara dentro del tunel para distribuir las ips de cada sede y las ip del resto de los nodos del tunel

		router ospf 1
		 network 10.0.0.0 0.0.0.255 area 0
		 network 192.168.0.0 0.0.0.255 area 0

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

4. En el caso del CPE secundario también redistribuimos lo que aprendemos de BGP por el tunel mediante OSPF y viceversa.

      	route-map bgp-into-ospf permit 10
	 	   match ip address 50
		   set metric 200
		   set metric-type type-2

		access-list 50 permit 10.0.4.0 0.0.0.255
		access-list 50 permit 10.0.2.0 0.0.0.255


      	router ospf 1
	       redistribute bgp 64501 subnets route-map bgp-into-ospf #Necesario para que la rutas
	        enviadas por el tunel mediante ospf sean menos prioritarias que las aprendidas por la vpn mpls

      	router bgp 64501
             bgp redistribute-internal
	     redistribute ospf 1

5. Por último configuramos HSRP en ambos CPEs para protegernos ante caídas de uno de ellos

 Con esas configuraciones conseguimos que el tráfico saliente a una dirección pública vaya por el router adecuado (HSRP) y que el tráfico de la VPN vaya por la principal
 siempre que sea posible


## 6RD

Para implementar IPv6 de forma sencilla sin migrar el core de IPv4 a IPv6 utilizamos 6RD. Se basa en encapsular trafico IPv6 en paquetes IPv4.
Utiliza tuneles multipunto para conectar los routers de los clientes a un router frontera conectado a una red IPv6.
Nota: Solo es necesario IOS 15 para implementar esta funcionalidad

#### Border Router

1. Activamos IPv6 en el router

   	     ipv6 unicast-routing
	     ipv6 cef

2. Creamos una Loopback con una ip publica. (Necesaria, ya que si no los paquetes irian con ips privadas) y guardamos prefijo que usa 6rd

   	     ipv6 general-prefix DELEGATED_PREFIX 6rd Tunnel0
	     interface Loopback1
	      ip address 30.0.255.2 255.255.255.255

3. Creamos el tunel

   	      interface Tunnel1
	       no ip address
	       no ip redirects
	       ipv6 address DELEGATED_PREFIX ::/128 anycast
	       tunnel source Loopback1
	       tunnel mode ipv6ip 6rd
	       tunnel 6rd ipv4 prefix-len 16
	       tunnel 6rd prefix 2001:AAAA::/40

4. Enrutamos el trafico IPv6 que llega al router por el tunel que acabamos de crear

   	     ipv6 route 2001:AAAA::/40 Tunnel1


##### CE Router

1. Activamos IPv6 en el router

   	     ipv6 unicast-routing
	     ipv6 cef

2. Guardamos prefijo que usa 6rd

   	     ipv6 general-prefix DELEGATED_PREFIX 6rd Tunnel0
	       
3. Creamos el tunel

   	      interface Tunnel1
	       no ip address
	       no ip redirects
	       ipv6 enable
	       tunnel source FastEthernet1/0
	       tunnel mode ipv6ip 6rd
	       tunnel 6rd ipv4 prefix-len 16
	       tunnel 6rd prefix 2001:AAAA::/40
	       tunnel 6rd br 30.0.255.2

4. Le damos una IPv6 a la interfaz interna del router

          ipv6 address DELEGATED_PREFIX ::/64 eui-64
	
4. Enrutamos el trafico IPv6 que llega al router por el tunel que acabamos de crear

   	     ipv6 route 2001:AAAA::/40 Tunnel1
	     ipv6 route ::/0 Tunnel1 2001:AAAA:FF:200::


##### Autoconfiguracion equipos

Una vez hecho todo esto, los equipos restantes se configuran solos automaticamente. Cogiendo el prefijo adecuadamente.

### Enlaces

+ [http://www.cisco.com/c/en/us/products/collateral/ios-nx-os-software/enterprise-ipv6-solution/whitepaper_c11-665758.html](http://www.cisco.com/c/en/us/products/collateral/ios-nx-os-software/enterprise-ipv6-solution/whitepaper_c11-665758.html)
+ [http://www.cisco.com/c/en/us/td/docs/ios-xml/ios/interface/configuration/xe-3s/ir-xe-3s-book/ip6-6rd-tunls-xe.pdf](http://www.cisco.com/c/en/us/td/docs/ios-xml/ios/interface/configuration/xe-3s/ir-xe-3s-book/ip6-6rd-tunls-xe.pdf)
+ [https://meetings.apnic.net/__data/assets/pdf_file/0017/31148/APRICOT-6rd-final.pdf](https://meetings.apnic.net/__data/assets/pdf_file/0017/31148/APRICOT-6rd-final.pdf)