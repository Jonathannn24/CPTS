# SMB

Server Message Block (SMB) es un protocolo cliente‑servidor que regula el acceso a archivos y directorios completos y otros recursos de red como impresoras, enrutadores o interfaces lanzadas para la red. El intercambio de información entre diferentes procesos del sistema también se puede gestionar basándose en el protocolo SMB. SMB por primera vez estuvo disponible para un público más amplio, por ejemplo, como parte del sistema operativo de red OS/2 LAN Manager y LAN Server. Desde entonces, la principal área de aplicación del protocolo ha sido en particular la serie de sistemas operativos Windows, cuyos servicios de red admiten SMB de manera compatible con versiones anteriores, lo que significa que los dispositivos con ediciones más nuevas pueden comunicarse fácilmente con dispositivos que tienen instalado un sistema operativo Microsoft más antiguo. Con el proyecto de software libre Samba también existe una solución que permite el uso de SMB en distribuciones Linux y Unix y así la comunicación multiplataforma vía SMB.

El protocolo SMB permite al cliente comunicarse con otros participantes de la misma red para acceder a archivos o servicios compartidos con él en la red. El otro sistema también debe haber implementado el protocolo de red y recibido y procesado la solicitud del cliente utilizando una aplicación de servidor SMB. Antes de eso, sin embargo, ambas partes deben establecer una conexión, por lo que primero intercambian los mensajes correspondientes.

En las redes IP, SMB utiliza el protocolo TCP para este propósito, que prevé un protocolo de enlace de tres vías entre el cliente y el servidor antes de que finalmente se establezca una conexión. Las especificaciones del protocolo TCP también regulan el transporte posterior de datos. Podemos echar un vistazo a algunos ejemplos aquí.

Un servidor SMB puede proporcionar partes arbitrarias de su sistema de archivos local como recursos compartidos. Por tanto, la jerarquía visible para un cliente es parcialmente independiente de la estructura del servidor. Los derechos de acceso están definidos por Access Control Lists (ACL). Se pueden controlar de forma detallada en función de atributos como: `execute`, `read`, y `full access` para usuarios individuales o grupos de usuarios. Las ACL se definen en función de los recursos compartidos y, por lo tanto, no corresponden a los derechos asignados localmente en el servidor.

---

## Samba

Como se mencionó anteriormente, existe una implementación alternativa del servidor SMB llamada **Samba**, que está desarrollada para sistemas operativos basados en Unix. Samba implementa el Common Internet File System (CIFS), un dialecto de SMB. Esto permite a Samba comunicarse eficazmente con los sistemas Windows. Por ello a menudo se le denomina **SMB/CIFS**.

CIFS se considera una versión específica del protocolo SMB (principalmente SMBv1). Cuando los comandos SMB se transmiten a través de Samba a un servicio NetBIOS más antiguo, las conexiones generalmente ocurren a través de los puertos TCP `137`, `138`, y `139`. Por el contrario, CIFS puede operar a través del puerto TCP `445` exclusivamente. SMB 2 y SMB 3 son versiones modernas con mejoras de rendimiento y seguridad.

### Versiones SMB (resumen)

| Versión SMB |                 Soportado en | Características                            |
| ----------- | ---------------------------: | ------------------------------------------ |
| CIFS        |               Windows NT 4.0 | Comunicación vía NetBIOS                   |
| SMB 1.0     |                 Windows 2000 | Conexión directa vía TCP                   |
| SMB 2.0     |  Windows Vista / Server 2008 | Rendimiento, firma mejorada                |
| SMB 2.1     |   Windows 7 / Server 2008 R2 | Mecanismos de bloqueo                      |
| SMB 3.0     |      Windows 8 / Server 2012 | Multicanal, cifrado, almacenamiento remoto |
| SMB 3.0.2   | Windows 8.1 / Server 2012 R2 | —                                          |
| SMB 3.1.1   |     Windows 10 / Server 2016 | Integridad, cifrado AES-128                |

Con la versión 3, Samba puede ser miembro de un dominio Active Directory y desde la versión 4 puede actuar como domain controller.

Samba usa varios *daemons*:

* `smbd` — servidor SMB (ficheros/impr.)
* `nmbd` — NetBIOS name service

Además, en redes SMB los hosts se agrupan por *workgroups* o dominios. NetBIOS/WINS proporcionan resolución de nombres en entornos legacy.

---

## Configuración predeterminada (ejemplo filtrado)

```bash
cat /etc/samba/smb.conf | grep -v "#\|\;"
```

Ejemplo (extracto):

```ini
[global]
   workgroup = DEV.INFREIGHT.HTB
   server string = DEVSMB
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d

   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes

   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .

   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes

[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700

[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
```

Parámetros comunes (explicación breve):

* `[sharename]` — nombre del recurso compartido.
* `workgroup` — grupo de trabajo o dominio.
* `path` — directorio que se comparte.
* `server string` — descripción que aparece al conectar.
* `unix password sync` — sincronizar contraseñas UNIX ↔ SMB.
* `usershare allow guests` — permitir shares definidos por usuarios no autenticados.
* `map to guest` — cómo mapear usuarios inválidos a guest.
* `browseable` — si el share aparece en la lista.
* `guest ok` — acceso sin contraseña.
* `read only` — sólo lectura.
* `create mask` — permisos por defecto para archivos nuevos.

---

## Configuraciones peligrosas (riesgos)

Ejemplos que incrementan el riesgo si se usan sin control:

* `browseable = yes` → facilita que un atacante liste recursos.
* `read only = no` / `writable = yes` → permite subir/alterar ficheros.
* `guest ok = yes` → acceso anónimo.
* `enable privileges = yes` → puede conceder permisos excesivos.
* `create mask = 0777` / `directory mask = 0777` → ficheros y directorios con permisos abiertos.
* `logon script`, `magic script`, `magic output` → ejecución automática de scripts (peligroso si no se controla).

---

## Ejemplo de share inseguro `[notes]`

```ini
[notes]
	comment = CheckIT
	path = /mnt/notes/

	browseable = yes
	read only = no
	writable = yes
	guest ok = yes

	enable privileges = yes
	create mask = 0777
	directory mask = 0777
```

Este tipo de configuración se encuentra en entornos de prueba o por descuido en entornos internos y permite una enumeración y acceso muy cómodos para un atacante.

---

## Reiniciar Samba

```bash
sudo systemctl restart smbd
# o (como root):
# root@samba:~# sudo systemctl restart smbd
```

---

## `smbclient` — listar shares e interactuar

**Listar shares (null session):**

```bash
smbclient -N -L //10.129.14.128
```

Salida ejemplo:

```
Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
home            Disk      INFREIGHT Samba
dev             Disk      DEVenv
notes           Disk      CheckIT
IPC$            IPC       IPC Service (DEVSM)
SMB1 disabled -- no workgroup available
```

**Conectarse a un share:**

```bash
smbclient //10.129.14.128/notes
# Enter WORKGROUP\<username>'s password:
# Anonymous login successful
```

Dentro de la sesión `smbclient`, comandos útiles:

```
help      # ver comandos
ls        # listar archivos
get <f>   # descargar fichero remoto
put <f>   # subir fichero local
!<cmd>    # ejecutar comando local sin salir
quit      # salir
```

Ejemplo de descarga:

```
smb: \> get prep-prod.txt
getting file \prep-prod.txt of size 71 as prep-prod.txt
smb: \> !cat prep-prod.txt
[] check your code with the templates
[] run code-assessment.py
[] …
```

---

## `smbstatus` — estado / conexiones activas en el servidor

```bash
sudo smbstatus
```

Salida ejemplo:

```
Samba version 4.11.6-Ubuntu
PID     Username     Group        Machine                                   Protocol Version  Encryption  Signing
75691   sambauser    samba        10.10.14.4 (ipv4:10.10.14.4:45564)      SMB3_11           -           -
Service  pid    Machine       Connected at                     Encryption  Signing
notes    75691  10.10.14.4   Do Sep 23 00:12:06 2021 CEST     -            -
No locked files
```

---

## Huella del servicio: Nmap

Nmap puede detectar SMB y ejecutar scripts NSE para obtener información:

```bash
sudo nmap 10.129.14.128 -sV -sC -p139,445
```

Salida ejemplo:

```
PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
|_nbstat: NetBIOS name: HTB
| smb2-security-mode: Message signing enabled but not required
| smb2-time: date: 2021-09-19T13:16:04
```

---

## `rpcclient` — MS‑RPC queries para enumeración

`rpcclient` permite ejecutar muchas consultas RPC útiles para enumerar información del servidor y del dominio.

Conectar (sesión nula):

```bash
rpcclient -U "" 10.129.14.128
# Enter WORKGROUP\'s password:  (Enter)
```

Comandos útiles dentro de `rpcclient`:

```
srvinfo             # Información del servidor
enumdomains         # Enumera dominios/workgroups
querydominfo        # Info de dominio/servidor/usuarios
netshareenumall     # Lista shares disponibles
netsharegetinfo <s> # Info de un share concreto
enumdomusers        # Lista usuarios del dominio
queryuser <RID>     # Info del usuario por RID (hex)
querygroup <RID>    # Info del grupo por RID (hex)
```

Ejemplos de uso y salidas (recortado para brevedad):

```
rpcclient $> srvinfo
rpcclient $> enumdomains
rpcclient $> querydominfo
rpcclient $> netshareenumall
rpcclient $> netsharegetinfo notes
# devuelve ACLs, path, permisos, etc.
```

Estos comandos muestran qué información es accesible incluso con sesiones anónimas y lo valioso que puede llegar a ser.

---

## Enumeración de usuarios (ejemplos)

Dentro de `rpcclient`:

```
rpcclient $> enumdomusers
user:[mrb3n] rid:[0x3e8]
user:[cry0l1t3] rid:[0x3e9]

rpcclient $> queryuser 0x3e9
# muestra atributos: User Name, Home Drive, Profile Path, Password last set Time, user_rid, group_rid, ...
```

### Fuerza bruta de RIDs (Bash loop)

```bash
for i in $(seq 500 1100); do
  rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" \
    | grep "User Name\|user_rid\|group_rid" && echo ""
done
```

Salida ejemplo (usuarios encontrados):

```
User Name   :   sambauser
user_rid :      0x1f5
group_rid:      0x201

User Name   :   mrb3n
user_rid :      0x3e8
group_rid:      0x201

User Name   :   cry0l1t3
user_rid :      0x3e9
group_rid:      0x201
```

---

## Impacket — `samrdump.py`

```bash
samrdump.py 10.129.14.128
```

Salida ejemplo resumida:

```
Found user: mrb3n, uid = 1000
Found user: cry0l1t3, uid = 1001
# Muestra atributos y timestamps de contraseña, estado de cuenta, etc.
```

---

## `smbmap` y `crackmapexec`

**smbmap** — listar shares y permisos:

```bash
smbmap -H 10.129.14.128
```

**crackmapexec** — enumerar shares con null session:

```bash
crackmapexec smb 10.129.14.128 --shares -u '' -p ''
```

Salida ejemplo (crackmapexec):

```
SMB 10.129.14.128 445 DEVSMB [*] Windows 6.1 ...
Share    Permissions  Remark
notes    READ,WRITE   CheckIT
```

---

## `enum4linux-ng` — instalación y ejecución

```bash
git clone https://github.com/cddmp/enum4linux-ng.git
cd enum4linux-ng
pip3 install -r requirements.txt
./enum4linux-ng.py 10.129.14.128 -A
```

`enum4linux-ng` automatiza muchas de las consultas SMB/NetBIOS/RPC y devuelve usuarios, shares, políticas y más.

---

## Observaciones finales

* Nunca confiar exclusivamente en herramientas automatizadas: contrastar resultados manualmente.
* Evitar habilitar SMBv1; forzar SMB 2/3 y aplicar firma/cifrado si es posible.
* No permitir `guest ok = yes` ni permisos 0777 en producción.
* Monitorizar conexiones con `smbstatus` y logs de Samba.

---

*Fin del documento.*
