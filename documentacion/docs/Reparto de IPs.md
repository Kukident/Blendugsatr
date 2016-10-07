#En el ISP

|Nombre | Prefijo de red | Máscara de subred | Anotaciones |
| - | ------------ | ------------- |------------- |
|Interiores | 192.168.1.X | 255.255.255.252  | Donde X toma valores de 4 en cuatro (0, 4, 8, ...) | 
|Pool público | 20.0.0.0  | 255.255.254.0 | |


#En la sucursal principal
|Nombre | Prefijo de red | Máscara de subred | Anotaciones |
| - | ------------ | ------------- |------------- | 
|Pool público | 203.0.113.128  | 255.255.254.0 | Asignadas a equipos que proporcionan Servicios Online |
|Intranet | 10.0.4.0 | 255.255.255.0 ||
|Data Center | 10.0.2.0  | 255.255.255.0 ||
