#Semana 1
1. Ping entre dos routers frontera
2. Ping entre un equipo final y una dirección pública
3. Ping desde un equipo final al servicio online de la sede principal
4. Que no se pueda hacer ping entre un router del core y uno de fuera

#Semana 2
**Trusted VPN**

1. Ping entre un host de la sede principal y un host de la secundaria (comprobando que no se haga NAT).
2. Ping entre un host y un equipo de "Internet" (comprobando que SÍ se hace NAT).
3. Ping entre un host de una sede secundaria a los servicios públicos de la principal (comprobar que va por fuera de la VPN)
4. Ping entre un equipo fuera de la VPN y los servicios públicos. 

**Conjuntas de ambas VPN**

1. Apagar el router principal de la sede principal. Comprobar en el Firewall que la gateway para las rutas de la VPN es ahora el router de backup.
2. Comprobar que si se apaga un router necesario para la Trusted VPN en una de las sedes se pasa a usar sin problemas la VPN secundaria (sólo con la sede afectada, el resto siguen por la principal).

**Salida/entrada doble a Internet**

1. Comprobar que al hacer un ping desde un router de Internet a una dirección perteneciente a los servicios públicos de la sede principal el paquete entra por:
    * Si sólo uno de los routers están encendidos, el tráfico entrará por ahí.
    * Si están ambos encendidos entrará por el principal, ya que tiene un AS-PATH inferior al compartirlo por BGP
