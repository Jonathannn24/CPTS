# FTP y TFTP — Guía de Enumeración y Configuración (con comandos)

## Visión general

**FTP (File Transfer Protocol)** es uno de los protocolos más antiguos de Internet. Opera en la **capa de aplicación** de TCP/IP (como HTTP o POP). Normalmente usa **TCP/21** para el canal de control y **TCP/20** para el canal de datos. Existen dos modos:

* **Activo**: el cliente abre control por 21/TCP y el **servidor** abre la conexión de datos hacia el puerto indicado por el cliente.
* **Pasivo**: el servidor anuncia un puerto y **el cliente** inicia también el canal de datos (útil tras firewalls estrictos del lado cliente).

FTP soporta múltiples **comandos** y el servidor responde con **códigos de estado** (p.ej., `220`, `230`, etc.). Puede existir **anonymous FTP** (sin contraseña) con permisos limitados.

**TFTP (Trivial FTP)** es aún más simple: no hay autenticación, usa **UDP** (no confiable) y suele restringirse a **redes locales** seguras. No lista directorios.

---

## TFTP — Comandos del cliente

| Comando   | Descripción                                                   |
| --------- | ------------------------------------------------------------- |
| `connect` | Define host remoto y opcionalmente puerto.                    |
| `get`     | Descarga uno o más archivos del host remoto al local.         |
| `put`     | Sube uno o más archivos del host local al remoto.             |
| `quit`    | Salir del cliente `tftp`.                                     |
| `status`  | Muestra estado (modo ascii/binario, timeout, conexión, etc.). |
| `verbose` | Activa/Desactiva salida detallada.                            |

> Nota: TFTP **no** tiene listado de directorios.

---

## vsFTPd — Instalación y configuración base

### Instalar vsFTPd

```bash
sudo apt install vsftpd
```

### Archivo de configuración por defecto

Ruta: `/etc/vsftpd.conf`

```bash
cat /etc/vsftpd.conf | grep -v "#"
```

**Opciones relevantes observadas**

| Configuración                                                 | Descripción                                       |
| ------------------------------------------------------------- | ------------------------------------------------- |
| `listen=NO`                                                   | Ejecutar desde inetd o como daemon independiente. |
| `listen_ipv6=YES`                                             | Escuchar en IPv6.                                 |
| `anonymous_enable=NO`                                         | Habilitar acceso anónimo.                         |
| `local_enable=YES`                                            | Permitir inicio de sesión de usuarios locales.    |
| `dirmessage_enable=YES`                                       | Mensajes al entrar a directorios.                 |
| `use_localtime=YES`                                           | Usar hora local.                                  |
| `xferlog_enable=YES`                                          | Registrar cargas/descargas.                       |
| `connect_from_port_20=YES`                                    | Conectarse desde el puerto 20.                    |
| `secure_chroot_dir=/var/run/vsftpd/empty`                     | Directorio vacío para chroot seguro.              |
| `pam_service_name=vsftpd`                                     | Servicio PAM a usar.                              |
| `rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem`          | Certificado RSA para SSL.                         |
| `rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key` | Clave privada.                                    |
| `ssl_enable=NO`                                               | Habilitar SSL.                                    |

### Denegar usuarios mediante `/etc/ftpusers`

```bash
cat /etc/ftpusers
```

Ejemplo de contenido:

```
guest
john
kevin
```

---

## Configuraciones "peligrosas" / comunes para pruebas

Estas opciones pueden existir en entornos internos para compartir ficheros. **Úselas con cuidado** en pruebas de laboratorio:

| Configuración                  | Descripción                                                   |
| ------------------------------ | ------------------------------------------------------------- |
| `anonymous_enable=YES`         | Permitir inicio de sesión anónimo.                            |
| `anon_upload_enable=YES`       | Permitir subidas anónimas.                                    |
| `anon_mkdir_write_enable=YES`  | Permitir creación de directorios anónimos.                    |
| `no_anon_password=YES`         | No pedir contraseña a anónimos.                               |
| `anon_root=/home/username/ftp` | Directorio raíz para anónimos.                                |
| `write_enable=YES`             | Permitir comandos de escritura (`STOR`, `DELE`, `MKD`, etc.). |

Otras opciones útiles en auditoría:

| Configuración             | Descripción                                                |
| ------------------------- | ---------------------------------------------------------- |
| `dirmessage_enable=YES`   | Mostrar mensaje al entrar por primera vez a un directorio. |
| `chown_uploads=YES`       | Cambiar propietario de subidas anónimas.                   |
| `chown_username=username` | Usuario al que se asignan subidas anónimas.                |
| `local_enable=YES`        | Permitir login de usuarios locales.                        |
| `chroot_local_user=YES`   | Chroot de usuarios locales a su `$HOME`.                   |
| `chroot_list_enable=YES`  | Usar lista de chroot para usuarios locales.                |
| `hide_ids=YES`            | Mostrar UID/GID como `ftp` en listados.                    |
| `ls_recurse_enable=YES`   | Permitir `ls -R` (listado recursivo).                      |

---

## Enumeración/uso del servicio FTP (cliente interactivo)

### Inicio de sesión anónimo

```bash
ftp 10.129.14.136
# Banner esperado:
# 220 "Welcome to the HTB Academy vsFTP service."
# Name (10.129.14.136:usuario): anonymous
# 230 Login successful.

ftp> ls
```

Salida de ejemplo (acortada):

```
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 1002 1002   8138592 Sep 14 16:54 Calender.pptx
... (más entradas) ...
226 Directory send OK.
```

### Ver estado del cliente (útil para debug)

```text
ftp> status
```

### Activar depuración y traza de paquetes en el cliente

```text
ftp> debug
ftp> trace
ftp> ls
```

Ejemplo de listado con `hide_ids=YES` activo:

```
ftp> ls
-rw-rw-r-- 1 ftp ftp 8138592 Sep 14 16:54 Calender.pptx
...
```

### Listado recursivo (si `ls_recurse_enable=YES`)

```text
ftp> ls -R
```

---

## Descarga y subida de archivos

### Descargar un archivo puntual

```text
ftp> ls
ftp> get Important\ Notes.txt
ftp> exit
```

Verificar localmente:

```bash
ls | grep Notes.txt
```

### Descargar **todo** el árbol disponible (ojo con alertas DLP/IDS)

```bash
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
```

Inspeccionar estructura descargada:

```bash
tree .
```

### Subir un archivo (PUT)

```bash
touch testupload.txt
```

En la sesión FTP:

```text
ftp> put testupload.txt
ftp> ls
```

---

## Huella y scripts NSE (Nmap)

### Actualizar base de datos de scripts NSE

```bash
sudo nmap --script-updatedb
```

### Localizar scripts NSE relacionados con FTP

```bash
find / -type f -name 'ftp*' 2>/dev/null | grep scripts
```

### Escaneo típico del servicio FTP (versión + scripts default + técnicas agresivas)

```bash
sudo nmap -sV -p21 -sC -A 10.129.14.136
```

Salida esperable (resumen):

```
21/tcp open  ftp  vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ... lista de ficheros ...
| ftp-syst:
|   STAT:
|   vsFTPd 3.0.3 - secure, fast, stable
```

### Trazar lo que hacen los scripts NSE a nivel de red

```bash
sudo nmap -sV -p21 -sC -A 10.129.14.136 --script-trace
```

Ejemplo de líneas ilustrativas:

```
NSE: TCP 10.10.14.4:54228 > 10.129.14.136:21 | CONNECT
NSE: TCP 10.10.14.4:54228 < 10.129.14.136:21 | 220 Welcome to HTB-Academy FTP service.
```

---

## Interacción directa con el puerto 21

### Con `nc` o `telnet`

```bash
nc -nv 10.129.14.136 21
```

```bash
telnet 10.129.14.136 21
```

### FTP sobre TLS/SSL (STARTTLS) con `openssl`

```bash
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```

Salida ilustrativa (resumida):

```
depth=0 C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev,
CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
verify error:num=18:self signed certificate
...
-----BEGIN CERTIFICATE-----
...SNIP...
```

> Útil para identificar **CN/hostnames**, correos, OU/ubicaciones y validar la cadena de confianza.

---

## Notas operativas

* **FTP es texto claro**: credenciales y datos pueden esnifarse si no se usa TLS/SSL.
* **Anonymous FTP** puede exponer directorios, leaks de información o vectores de RCE si se combina con LFI/websync.
* **Fail2ban** u otros mecanismos suelen limitar el bruteforce; planifique intentos cuidadosos.
* **TFTP**: limite su uso a VLANs de gestión/lab (PXE/boot) y restrinja directorios de trabajo.

---

## Glosario rápido

* **220/230**: códigos típicos FTP (servicio listo / login correcto).
* **PORT/PASV**: comandos/estados para modo activo vs pasivo.
* **STAT**: estado del servidor (`ftp-syst` NSE lo usa).
* **STOR/RETR**: subir/bajar archivos en FTP.

---

> Esta guía está pensada para uso didáctico y de **pentesting autorizado**. Aplique siempre en entornos donde tenga permiso explícito.
