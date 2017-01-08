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
|Loopback 6rd| 30.0.255.2 | 255.255.255.255 | Necesaria para el funcionamiento del 6rd|
|Loopback xDSL| 21.0.0.2 | 255.255.255.255 | Necesaria xDSL|
|Pool público | 30.0.0.0  | 255.255.254.0 ||

#En la sucursal principal
|Nombre | Prefijo de red | Máscara de subred | Anotaciones |
| - | ------------ | ------------- |------------- | 
|Pool público | 203.0.112.0  | 255.255.255.0 | Asignadas a equipos que proporcionan Servicios Online |
|Direcciones administrativas | 203.0.113.0 | 255.255.255.248 | Hemos dividido nuestro prefijo público en dos, uno para los equipos y otro para usos administrativos |
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
