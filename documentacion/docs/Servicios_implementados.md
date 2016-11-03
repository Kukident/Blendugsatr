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

2) Creamos las VRFs en los routers frontera:

```bash
#Repetimos estos comandos en todos los routers frontera que participen en la VPN.
ip vrf {Nombre de la VRF}
rd XXXXX:XX
route-target both XXXXX:XX
```

3) Asignamos las VRFs a las interfaces del router:

```bash
interface XX.Z
ip vrf forwarding {Nombre de la VRF}

#Debemos asignar de nuevo la ip a la interfaz
ip add {IP} {Máscara}
```

4) Configuramos BGP entre los routers frontera.

```bash
router bgp {identificador del AS}

neighbor {IP de la loopback del vecino} remote-as {identificador del AS}
neighbor {IP de la loopback del vecino} update-source loopback 0

address-family vpnv4
neighbor {IP de la loopback del vecino} activate

#Repetimos en el otro vecino
```

5) Configuramos BGP entre los routers frontera y el router del cliente.

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
