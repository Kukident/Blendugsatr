#Visualizar rutas

```bash
show ip route          # Muestra todas la tabla de encaminamiento
show ip route ospf     # Muestra las rutas aprendidas por OSPF
show ip route A.B.C.D  # Muestra la información sobre la ruta a ese host   
```


#Debugear BGP

```bash
#Activar el debug de BGP
debug bgp all

# Comprobar que hay conexión con el vecino
show ip route {IP del vecino}

# Datos sobre vecino/s
show ip bgp neighbors [IP del vecino]


```
