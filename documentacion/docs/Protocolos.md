#OSPF

 Está implementado en todo el core del ISP, otorgando conectividad entre todos los routers interiores. Es muy sencillo de configurar, los comandos a usar son los siguientes:

```python
enable
configure terminal
router ospf ID
network 0.0.0.0 255.255.255.255 area 0
end

```
#DHCP
 Usado para asignar de forma estática las IPs (tanto privadas dentro de una subred como públicas a lso routers de los clientes). La configuración es la que sigue:

```python
enable
configure terminal
ip dhcp pool nombre_del_pool
network A.B.C.D/XX
default-router A.B.C.E
end

#Si queremos que alguna dirección no se asigne:
ip dhcp excluded-address A.B.C.D
```
