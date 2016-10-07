#OSPF

 Está implementado en todo el core del ISP, otorgando conectividad entre todos los routers interiores. Es muy sencillo de configurar, los comandos a usar son los siguientes:

```bash
# Desde (config)#
router ospf ID
network 0.0.0.0 255.255.255.255 area 0

```

#BGP
 Multiples usos que requieren una configuración variada. La configuración básica es:

```bash
# Desde (config)#
router bgp {ID del AS}
network A.B.C.D mask E.F.G.H
neighbor {IP del vecino} remote-as {ID del AS al que nos conectamos}

# Si usamos la loopback (recomendado) hace falta indicarlo
neighbor {IP del vecino} update-source loopback {Nº de la loopback}

```
 * Aquí añadiremos el resto de usos de BGP

#DHCP
 Usado para asignar de forma estática las IPs (tanto privadas dentro de una subred como públicas a lso routers de los clientes). La configuración es la que sigue:

```bash
# Desde (config)#
ip dhcp pool nombre_del_pool
network A.B.C.D/XX
default-router A.B.C.E
end

#Si queremos que alguna dirección no se asigne:
ip dhcp excluded-address A.B.C.D
```

#NAT dinámico con sobrecarga
 Permite utilizar una única dirección publica cuando los equipos de una subred quieren conectarse a Internet. Los comandos son los siguientes:

```bash
# Creamos una lista de acceso
access-list {100-199} permit ip A.B.C.D {WildCard de la máscara} any

# Creamos NAT asignándole la lista de acceso
ip nat inside source list {100-199} interface {Interfaz de salida} overload

# Asignamos a cada interfaz su rol (nat inside o nat outside)
ip nat {inside | outside}

```
