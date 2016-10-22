#En el ISP

|Nombre | Prefijo de red | Máscara de subred | Anotaciones |
| - | ------------ | ------------- |------------- |
|Interiores | 192.168.1.X | 255.255.255.252  | Donde X toma valores de cuatro en cuatro (0, 4, 8, ...) | 
|Loopbacks | 192.168.11.X | 255.255.255.255 | |
|Pool público | 20.0.0.0  | 255.255.254.0 |En cada enlace con un cliente del servicio Trusted VPN se usaran dos subredes, una para tráfico normal y otro para la VRF |

#En el ISP secundario
|Nombre | Prefijo de red | Máscara de subred | Anotaciones |
| - | ------------ | ------------- |------------- |
|Interiores | 192.168.2.X | 255.255.255.252 | Donde X toma valores de cuatro en cuatro (0, 4, 8, ...) |
|Loopbacks | 192.168.22.X | 255.255.255.255 ||

#En la sucursal principal
|Nombre | Prefijo de red | Máscara de subred | Anotaciones |
| - | ------------ | ------------- |------------- | 
|Pool público | 203.0.113.128  | 255.255.254.0 | Asignadas a equipos que proporcionan Servicios Online |
|Intranet | 10.0.4.0 | 255.255.255.0 ||
|Data Center | 10.0.2.0  | 255.255.255.0 ||

#En la sucursal secundaria (derecha)
|Nombre | Prefijo de red | Máscara de subred | Anotaciones |
| - | ------------ | ------------- |------------- |
|Intranet | 10.0.0.0 | 255.255.255.0 | |

#En la sucursal secundaria (abajo)
|Nombre | Prefijo de red | Máscara de subred | Anotaciones |
| - | ------------ | ------------- |------------- |
|Intranet | 10.0.3.0 | 255.255.255.0 | |
