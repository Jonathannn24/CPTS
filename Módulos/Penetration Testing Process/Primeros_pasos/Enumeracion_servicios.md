
## Nmap: ejemplos básicos y avanzados

### Escaneo básico (1000 puertos más comunes)

Comando:

```bash
nmap 10.129.42.253
```

Salida de ejemplo:

```
Starting Nmap 7.80 ( https://nmap.org ) at 2021-02-25 16:07 EST
Nmap scan report for 10.129.42.253
Host is up (0.11s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 2.19 seconds
```

Por defecto, Nmap realizará un escaneo **TCP (Transmission Control Protocol)** a menos que se le solicite específicamente que realice un escaneo UDP.

* El encabezado `STATE` confirma si los puertos están `open` (abiertos) o en estados como `filtered`.
* El encabezado `SERVICE` indica el nombre asociado normalmente al número de puerto, pero no garantiza que el servicio real coincida hasta que se realicen comprobaciones adicionales.

### Escaneo completo con versión y scripts

Comando para escanear todos los puertos TCP, detectar versiones y ejecutar scripts predeterminados:

```bash
nmap -sV -sC -p- 10.129.42.253
```

Ejemplo de salida (resumen relevante):

```
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    2 ftp      ftp          4096 Feb 25 19:25 pub
22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2

Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

> Notas:
>
> * `-sC` ejecuta scripts por defecto para obtener más información.
> * `-sV` realiza detección de versiones de servicios.
> * `-p-` indica escanear los **65.535** puertos TCP.

Este tipo de escaneo tarda mucho más que el escaneo por defecto de 1.000 puertos porque se realizan comprobaciones adicionales y huellas dactilares de servicios.

---

## Interpretación de resultados y huellas dactilares

La información de versión puede ayudar a inferir el sistema operativo y su versión. Por ejemplo, una salida de **OpenSSH 8.2p1 Ubuntu 4ubuntu0.1** sugiere que la máquina podría ejecutar **Ubuntu 20.04 (Focal Fossa)**, aunque esta técnica no es infalible (los paquetes pueden actualizarse en distribuciones antiguas).

El script `-sC` puede revelar encabezados HTTP (`http-server-header`) o títulos de página (`http-title`), lo que puede indicar sitios como `phpinfo()` que exponen versiones de PHP vulnerables.

---

## Scripts específicos de Nmap

Si necesitamos comprobar una vulnerabilidad o una configuración concreta (por ejemplo, Citrix NetScaler CVE-2019-19781), Nmap incluye scripts especializados ubicados típicamente en `/usr/share/nmap/scripts/`.

Sintaxis para ejecutar un script concreto:

```bash
nmap --script <script-name>.nse -p <port> <host>
```

---

## Técnicas de enumeración y ataque a servicios

### Banner grabbing (captura de banners)

Nmap puede capturar banners con `--script=banner` o podemos hacerlo manualmente con `netcat (nc)`:

```bash
nc -nv 10.129.42.253 21
# ejemplo de salida: 220 (vsFTPd 3.0.3)
```

### FTP (File Transfer Protocol)

FTP suele ofrecer datos interesantes y, si permite `anonymous` (anónimo), se pueden listar y descargar ficheros:

```bash
nmap -sC -sV -p21 10.129.42.253
ftp -p 10.129.42.253
# login: anonymous
# ls, cd, get login.txt
```

Ejemplo de archivo `login.txt` obtenido:

```
admin:ftp@dmin123
```

### SMB (Server Message Block)

SMB es muy común en entornos Windows y ofrece vectores para movimiento lateral. Nmap tiene scripts para enumerar SMB:

```bash
nmap --script smb-os-discovery.nse -p445 10.10.10.40
```

Salida de ejemplo:

```
Host script results:
| smb-os-discovery:
|   OS: Windows 7 Professional 7601 Service Pack 1
|   Computer name: CEO-PC
```

Enumeración de recursos con `smbclient`:

```bash
smbclient -N -L \\\\10.129.42.253
smbclient \\\\10.129.42.253\\users
# autenticación con bob:Welcome1
# get passwords.txt
```

---

### SNMP (Simple Network Management Protocol)

Las cadenas de comunidad `public` y `private` en SNMP v1/v2c suelen ser valores por defecto y permiten obtener información del dispositivo (nombre, versión del kernel, procesos). Ejemplo con `snmpwalk`:

```bash
snmpwalk -v 2c -c public 10.129.42.253 1.3.6.1.2.1.1.5.0
# iso.3.6.1.2.1.1.5.0 = STRING: "gs-svcscan"
```

Para fuerza bruta de cadenas comunitarias se puede usar `onesixtyone` con un diccionario de cadenas comunes.

---

## Buenas prácticas y consideraciones éticas

* Realizar siempre este tipo de escaneos dentro del **alcance contractual** y con **autorización explícita** del propietario del sistema.
* Registrar todas las acciones, conservar evidencias y realizar limpieza (remover herramientas, restaurar cambios menores) tras finalizar las pruebas.
* Priorizar la no interrupción de servicios críticos y coordinar ventanas de prueba si es necesario.

---

## Referencias y recursos adicionales

* Documentación oficial de Nmap: [https://nmap.org](https://nmap.org)
* Repositorio de scripts y ejemplos de Nmap

---

*Fin del documento.*
