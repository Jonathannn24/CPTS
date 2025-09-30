# Primeros pasos: Enumeración básica 

Nuestro primer paso al acercarnos a cualquier máquina es realizar alguna enumeración básica. Primero, comencemos con lo que sí sabemos sobre el objetivo. Ya sabemos la **dirección IP (Internet Protocol)** del objetivo, que es **Linux** y tiene un vector de ataque relacionado con la web. A esto lo llamamos **enfoque de caja gris (grey‑box)** porque tenemos cierta información sobre el objetivo. En la plataforma **HTB (Hack The Box)**, las 20 máquinas de lanzamiento semanal "activas" se abordan desde una perspectiva de **caja negra (black‑box)**. A los usuarios se les proporciona de antemano la dirección IP y el tipo de sistema operativo, pero no hay información adicional sobre el objetivo para formular sus ataques. Es por esto que la **enumeración exhaustiva** es fundamental y a menudo es un proceso iterativo.

Antes de continuar, demos un rápido paso atrás y observemos los distintos enfoques de las acciones de pruebas de penetración. Hay tres tipos principales, **black‑box**, **grey‑box**, y **white‑box**, y cada uno difiere en el objetivo y el enfoque.

| Compromiso    | Descripción                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Black‑Box** | Bajo nivel o sin conocimiento de un objetivo. El probador de penetración debe realizar un reconocimiento en profundidad para conocer el objetivo. Esta puede ser una prueba de penetración externa en la que al evaluador solo se le proporciona el nombre de la empresa y ninguna información adicional, como direcciones IP de destino, o una prueba de penetración interna en la que el evaluador tiene que eludir los controles para obtener acceso inicial a la red o puede conectarse a la red interna pero no tiene información sobre las redes/hosts internos. Este tipo de prueba de penetración simula en gran medida un ataque real, pero no es tan completa como otros tipos de evaluación y podría dejar configuraciones incorrectas o vulnerabilidades sin descubrir. |
| **Grey‑Box**  | En una prueba de caja gris, el evaluador recibe una cierta cantidad de información por adelantado. Esta puede ser una lista de direcciones/rangos IP dentro del alcance, credenciales de bajo nivel para una aplicación web o **AD (Active Directory)**, o algunos diagramas de aplicaciones/redes. Este tipo de prueba de penetración puede simular un infiltrado malintencionado o ver lo que un atacante puede hacer con un bajo nivel de acceso. En este escenario, el evaluador normalmente dedicará menos tiempo al reconocimiento y más tiempo a buscar configuraciones incorrectas e intentar explotarlas.                                                                                                                                                                  |
| **White‑Box** | En este tipo de prueba, el evaluador tiene acceso completo. En una prueba de aplicación web, se les pueden proporcionar credenciales de nivel de administrador, acceso al código fuente, diagramas de compilación, etc., para buscar vulnerabilidades lógicas y otras fallas difíciles de descubrir. En una prueba de red, es posible que se les proporcionen credenciales de nivel de administrador para investigar Active Directory u otros sistemas en busca de configuraciones incorrectas que de otro modo podrían pasar desapercibidas. Este tipo de evaluación es muy completa ya que el evaluador tendrá acceso a ambos lados de un objetivo y realizará un análisis exhaustivo.                                                                                            |

---

## Nmap (Network Mapper)

Comencemos con algo rápido: un escaneo para buscar puertos abiertos usando el comando `nmap -sV --open -oA nibbles_initial_scan <ip address>`. Esto ejecutará una **enumeración de servicios** (`-sV`) contra los **1000 puertos principales** por defecto y solo devolverá los puertos **abiertos** (`--open`). Podemos comprobar qué puertos Nmap escanea para un tipo de escaneo determinado ejecutando un escaneo **sin objetivo** especificado con `nmap -v -oG -`. Aquí enviaremos el formato **greppable** a `stdout` con `-oG -` y `-v` para resultados detallados. Dado que no se especifica ningún objetivo, el escaneo fallará pero mostrará los puertos escaneados.

```bash
nmap -v -oG -
```

Salida (resumen):

```
# Nmap 7.80 scan initiated Wed Dec 16 23:22:26 2020 as: nmap -v -oG -
# Ports scanned: TCP (Transmission Control Protocol)(1000;1,3-4,6-7,9,13,17,19-26,30,32-33,37,42-43,49,53,70,79-85,88-90,99-100,106,109-111,113,119,125,135,139,143-144,146,161,163,179,199,211-212,222,254-256,259,264,280,301,306,311,340,366,389,406-407,416-417,425,427,443-445,458,464-465,481,497,500,512-515,524,541,543-545,548,554-555,563,587,593,616-617,625,631,636,646,648,666-668,683,687,691,700,705,711,714,720,722,726,749,765,777,783,787,800-801,808,843,873,880,888,898,900-903,911-912,981,987,990,992-993,995,999-1002,1007,1009-1011,1021-1100,1102,1104-1108,1110-1114,1117,1119,1121-1124,1126,1130-1132,1137-1138,1141,1145,1147-1149,1151-1152,1154,1163-1166,1169,1174-1175,1183,1185-1187,1192,1198-1199,1201,1213,1216-1218,1233-1234,1236,1244,1247-1248,1259,1271-1272,1277,1287,1296,1300-1301,1309-1311,1322,1328,1334,1352,1417,1433-1434,1443,1455,1461,1494,1500-1501,1503,1521,1524,1533,1556,1580,1583,1594,1600,1641,1658,1666,1687-1688,1700,1717-1721,1723,1755,1761,1782-1783,1801,1805,1812,1839-1840,1862-1864,1875,1900,1914,1935,1947,1971-1972,1974,1984,1998-2010,2013,2020-2022,2030,2033-2035,2038,2040-2043,2045-2049,2065,2068,2099-2100,2103,2105-2107,2111,2119,2121,2126,2135,2144,2160-2161,2170,2179,2190-2191,2196,2200,2222,2251,2260,2288,2301,2323,2366,2381-2383,2393-2394,2399,2401,2492,2500,2522,2525,2557,2601-2602,2604-2605,2607-2608,2638,2701-2702,2710,2717-2718,2725,2800,2809,2811,2869,2875,2909-2910,2920,2967-2968,2998,3000-3001,3003,3005-3007,3011,3013,3017,3030-3031,3052,3071,3077,3128,3168,3211,3221,3260-3261,3268-3269,3283,3300-3301,3306,3322-3325,3333,3351,3367,3369-3372,3389-3390,3404,3476,3493,3517,3527,3546,3551,3580,3659,3689-3690,3703,3737,3766,3784,3800-3801,3809,3814,3826-3828,3851,3869,3871,3878,3880,3889,3905,3914,3918,3920,3945,3971,3986,3995,3998,4000-4006,4045,4111,4125-4126,4129,4224,4242,4279,4321,4343,4443-4446,4449,4550,4567,4662,4848,4899-4900,4998,5000-5004,5009,5030,5033,5050-5051,5054,5060-5061,5080,5087,5100-5102,5120,5190,5200,5214,5221-5222,5225-5226,5269,5280,5298,5357,5405,5414,5431-5432,5440,5500,5510,5544,5550,5555,5560,5566,5631,5633,5666,5678-5679,5718,5730,5800-5802,5810-5811,5815,5822,5825,5850,5859,5862,5877,5900-5904,5906-5907,5910-5911,5915,5922,5925,5950,5952,5959-5963,5987-5989,5998-6007,6009,6025,6059,6100-6101,6106,6112,6123,6129,6156,6346,6389,6502,6510,6543,6547,6565-6567,6580,6646,6666-6669,6689,6692,6699,6779,6788-6789,6792,6839,6881,6901,6969,7000-7002,7004,7007,7019,7025,7070,7100,7103,7106,7200-7201,7402,7435,7443,7496,7512,7625,7627,7676,7741,7777-7778,7800,7911,7920-7921,7937-7938,7999-8002,8007-8011,8021-8022,8031,8042,8045,8080-8090,8093,8099-8100,8180-8181,8192-8194,8200,8222,8254,8290-8292,8300,8333,8383,8400,8402,8443,8500,8600,8649,8651-8652,8654,8701,8800,8873,8888,8899,8994,9000-9003,9009-9011,9040,9050,9071,9080-9081,9090-9091,9099-9103,9110-9111,9200,9207,9220,9290,9415,9418,9485,9500,9502-9503,9535,9575,9593-9595,9618,9666,9876-9878,9898,9900,9917,9929,9943-9944,9968,9998-10004,10009-10010,10012,10024-10025,10082,10180,10215,10243,10566,10616-10617,10621,10626,10628-10629,10778,11110-11111,11967,12000,12174,12265,12345,13456,13722,13782-13783,14000,14238,14441-14442,15000,15002-15004,15660,15742,16000-16001,16012,16016,16018,16080,16113,16992-16993,17877,17988,18040,18101,18988,19101,19283,19315,19350,19780,19801,19842,20000,20005,20031,20221-20222,20828,21571,22939,23502,24444,24800,25734-25735,26214,27000,27352-27353,27355-27356,27715,28201,30000,30718,30951,31038,31337,32768-32785,33354,33899,34571-34573,35500,38292,40193,40911,41511,42510,44176,44442-44443,44501,45100,48080,49152-49161,49163,49165,49167,49175-49176,49400,49999-50003,50006,50300,50389,50500,50636,50800,51103,51493,52673,52822,52848,52869,54045,54328,55055-55056,55555,55600,56737-56738,57294,57797,58080,60020,60443,61532,61900,62078,63331,64623,64680,65000,65129,65389)
WARNING: No targets were specified, so 0 hosts scanned.
```

Finalmente, generaremos todos los formatos de escaneo utilizando `-oA`. Esto incluye salida **XML (eXtensible Markup Language)**, salida **greppable** y salida de **texto** que pueden resultarnos útiles más adelante. Es esencial adquirir el hábito de tomar **notas extensas** y **guardar toda la salida** desde el principio (con marcas de tiempo), lo que acelera el reporting y conserva evidencia.

```bash
nmap -sV --open -oA nibbles_initial_scan 10.129.42.190
```

Salida:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd <REDACTED> ((Ubuntu))
```

Los tres archivos generados:

```bash
ls
# nibbles_initial_scan.gnmap  nibbles_initial_scan.nmap  nibbles_initial_scan.xml
```

Antes de empezar con puertos abiertos, podemos ejecutar un **escaneo TCP completo**:

```bash
nmap -p- --open -oA nibbles_full_tcp_scan 10.129.42.190
```

Podemos dejarlo ejecutando y continuar la enumeración. Con **nc (netcat)** podemos hacer banner grabbing que confirma lo detectado por Nmap:

```bash
nc -nv 10.129.42.190 22
# SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.8

nc -nv 10.129.42.190 80
# (http) open
```

Ahora, un escaneo con **scripts por defecto** (`-sC`):

```bash
nmap -sC -p 22,80 -oA nibbles_script_scan 10.129.42.190
```

Salida (resumen):

```
22/tcp open  ssh
| ssh-hostkey:
|   2048 ... (RSA (Rivest–Shamir–Adleman))
|   256  ... (ECDSA (Elliptic Curve Digital Signature Algorithm))
|_  256 ... (ED25519 (Edwards-curve Digital Signature Algorithm))
80/tcp open  http
|_http-title: Site doesn't have a title (text/html).
```

Enumeración HTTP con script `http-enum`:

```bash
nmap -sV --script=http-enum -oA nibbles_nmap_http_enum 10.129.42.190
```

Salida (resumen):

```
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd <REDACTED> ((Ubuntu))
|_http-server-header: Apache/<REDACTED> (Ubuntu)
```

> Conclusión: Host probable **Ubuntu Linux** con **Apache (HTTP (Hypertext Transfer Protocol))** en 80 y **OpenSSH (Secure Shell)** en 22. Continuar con enumeración web (dirbusting/fuzzing), revisión de `robots.txt`, fingerprinting y pruebas específicas de la app.
