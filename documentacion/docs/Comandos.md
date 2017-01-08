#Visualizar rutas

```bash
show ip route          # Muestra todas la tabla de encaminamiento
show ip route ospf     # Muestra las rutas aprendidas por OSPF
show ip route A.B.C.D  # Muestra la información sobre la ruta a ese host   
```


#Debugear BGP

```bash
debug bgp all							#Activar el debug de BGP
show ip route {IP del vecino}			#Comprobar que hay conexión con el vecino
show ip bgp neighbors [IP del vecino]	#Datos sobre vecino/s
```

#Trusted VPN

```bash
show ip route vrf "Nombre de la VRF"	#Muestra las entradas de la VRF
```
#Tunnel 6rd

```bash
show tunnel 6rd											#Muestra informacion del tunel 6rd
show tunnel 6rd destination 2001:AAAA:2:200:: tunnel 1	#Traduce una direccion IPv6 en la direccion IPv4 asociada
show tunnel 6rd prefix 30.0.2.2 tunnel 1      	        #Traduce una direccion IPv4 en el prefijo IPv6 correspondiente
```