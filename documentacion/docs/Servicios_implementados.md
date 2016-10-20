#Trusted VPN

 Hemos implementado una trusted VPN entre las distintas sedes de la empresa. Los pasos a seguir son los siguientes:

1) Configuramos MPLS en las interfaces de los routers del ISP:
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
interface XX
ip vrf forwarding {Nombre de la VRF}

#Debemos asignar de nuevo la ip a la interfaz
ip add {IP} {Máscara}
```

4) Configuramos BGP entre ambos nodos

```bash
router bgp {identificador del AS}

neighbor {IP de la loopback del vecino} remote-as {identificador del AS}
neighbor {IP de la loopback del vecino} update-source loopback 0

address-family vpnv4
neighbor {IP de la loopback del vecino} activate
```

5) Configuramos EIGRP entre los routers frontera y el router del cliente

```bash
router eigrp {Identificador}
address-family ipv4 vrf {Nombre de la VRF}
redistribute bgp {ID del proceso BGP}
network {IP del enlace}
autonomous-system 110
```

6) Redistribuimos las rutas de EIGRP en el BGP

```
router bgp {Identificador del bgp}

address-family ipv4 vrf Blendugs
redistribute eigrp {ID del proceso EIGRP}
```
