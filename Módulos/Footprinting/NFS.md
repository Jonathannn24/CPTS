# NFS

Network File System (NFS) es un sistema de archivos de red desarrollado por Sun Microsystems y tiene el mismo propósito que SMB. Su finalidad es acceder a los sistemas de archivos a través de una red como si fueran locales. Sin embargo, utiliza un protocolo completamente diferente. NFS se utiliza entre sistemas Linux y Unix. Esto significa que los clientes NFS no pueden comunicarse directamente con los servidores SMB. NFS es un estándar de Internet que rige los procedimientos en un sistema de archivos distribuido. Mientras que el protocolo **NFS versión 3.0 (NFSv3)**, que se utiliza desde hace muchos años, autentica el ordenador cliente, esto cambia con **NFSv4**. Aquí, al igual que con el protocolo SMB de Windows, el usuario debe autenticarse.

## Versiones y características

| Versión | Características |
| --- | --- |
| **NFSv2** | Es más antiguo pero cuenta con el soporte de muchos sistemas e inicialmente funcionaba íntegramente sobre UDP. |
| **NFSv3** | Tiene más funciones, incluido un tamaño de archivo variable y mejores informes de errores, pero no es totalmente compatible con los clientes NFSv2. |
| **NFSv4** | Incluye Kerberos, funciona a través de firewalls y en Internet, ya no requiere portmappers, admite ACL, aplica operaciones basadas en estados y proporciona mejoras de rendimiento y alta seguridad. También es la primera versión que tiene un protocolo con estado. |

**NFS versión 4.1 (RFC 8881)** tiene como objetivo proporcionar soporte de protocolo para aprovechar las implementaciones de servidores en clúster, incluida la capacidad de proporcionar acceso paralelo escalable a archivos distribuidos en múltiples servidores (extensión **pNFS**). Además, **NFSv4.1** incluye un mecanismo de *trunking* de sesiones, también conocido como **multitrayecto NFS**. Una ventaja significativa de NFSv4 sobre sus predecesores es que solo usa un puerto **UDP/TCP 2049**, lo que simplifica su uso a través de *firewalls*.

NFS se basa en el **ONC-RPC/SUN-RPC** expuesto en **TCP/UDP 111**, que usa **XDR** para el intercambio de datos independiente del sistema. El protocolo NFS **no** tiene mecanismo propio de autenticación/autorización: la autenticación se deriva de RPC y la **autorización** del FS (UID/GID y ACL). Esto implica que cliente y servidor deben alinear **UID/GID** para evitar confusiones; por ello, **NFS con auth UNIX** solo debería usarse en redes de confianza.

---

## Configuración predeterminada

El archivo **`/etc/exports`** contiene la tabla de exportaciones.

```bash
# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
```

### Opciones típicas

| Opción | Descripción |
|---|---|
| `rw` | Permisos de lectura y escritura. |
| `ro` | Solo lectura. |
| `sync` | Escritura síncrona (más seguro, algo más lento). |
| `async` | Escritura asíncrona (más rápido, menos seguro ante cortes). |
| `secure` | No usar puertos >1024. |
| `insecure` | Permite puertos >1024. |
| `no_subtree_check` | Desactiva chequeo de subárbol. |
| `root_squash` | Mapea UID/GID 0 a anónimo para evitar que `root` tenga privilegios en el montaje. |

### Ejemplo de exportación y publicación

```bash
root@nfs:~# echo '/mnt/nfs  10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
root@nfs:~# systemctl restart nfs-kernel-server 
root@nfs:~# exportfs
/mnt/nfs      	10.129.14.0/24
```

---

## Configuraciones peligrosas (a vigilar)

| Opción | Motivo |
|---|---|
| `rw` | Permite escritura remota (riesgo si no controlado). |
| `insecure` | Permite puertos >1024 desde clientes. |
| `nohide` | Submontajes aparecen expuestos si no se separan entradas. |
| `no_root_squash` | `root` del cliente mantiene UID/GID 0 en el servidor (muy riesgoso). |

---

## Huella del servicio (nmap + RPC)

### Escaneo de puertos clave (111 y 2049)

```bash
JonaLF@htb[/htb]$ sudo nmap 10.129.14.128 -p111,2049 -sV -sC

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 17:12 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00018s latency).

PORT    STATE SERVICE VERSION
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      41982/udp6  mountd
|   100005  1,2,3      45837/tcp   mountd
|   100005  1,2,3      47217/tcp6  mountd
|   100005  1,2,3      58830/udp   mountd
|   100021  1,3,4      39542/udp   nlockmgr
|   100021  1,3,4      44629/tcp   nlockmgr
|   100021  1,3,4      45273/tcp6  nlockmgr
|   100021  1,3,4      47524/udp6  nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp open  nfs_acl 3 (RPC #100227)
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.58 seconds
```

### Scripts NSE útiles para NFS

```bash
JonaLF@htb[/htb]$ sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049

PORT     STATE SERVICE VERSION
111/tcp  open  rpcbind 2-4 (RPC #100000)
| nfs-ls: Volume /mnt/nfs
|   access: Read Lookup NoModify NoExtend NoDelete NoExecute
| PERMISSION  UID    GID    SIZE  TIME                 FILENAME
| rwxrwxrwx   65534  65534  4096  2021-09-19T15:28:17  .
| ??????????  ?      ?      ?     ?                    ..
| rw-r--r--   0      0      1872  2021-09-19T15:27:42  id_rsa
| rw-r--r--   0      0      348   2021-09-19T15:28:17  id_rsa.pub
| rw-r--r--   0      0      0     2021-09-19T15:22:30  nfs.share
|_
| nfs-showmount: 
|_  /mnt/nfs 10.129.14.0/24
| nfs-statfs: 
|   Filesystem  1K-blocks   Used       Available   Use%  Maxfilesize  Maxlink
|_  /mnt/nfs    30313412.0  8074868.0  20675664.0  29%   16.0T        32000
| rpcinfo: ...
2049/tcp open  nfs_acl 3 (RPC #100227)
```

---

## Montaje e inspección de recursos NFS

### Mostrar exportaciones disponibles

```bash
JonaLF@htb[/htb]$ showmount -e 10.129.14.128

Export list for 10.129.14.128:
/mnt/nfs 10.129.14.0/24
```

### Montar y listar contenido

```bash
JonaLF@htb[/htb]$ mkdir target-NFS
JonaLF@htb[/htb]$ sudo mount -t nfs 10.129.14.128:/ ./target-NFS/ -o nolock
JonaLF@htb[/htb]$ cd target-NFS
JonaLF@htb[/htb]$ tree .

.
└── mnt
    └── nfs
        ├── id_rsa
        ├── id_rsa.pub
        └── nfs.share

2 directories, 3 files
```

### Ver propietarios por nombre y por UID/GID

```bash
JonaLF@htb[/htb]$ ls -l mnt/nfs/

total 16
-rw-r--r-- 1 cry0l1t3 cry0l1t3 1872 Sep 25 00:55 cry0l1t3.priv
-rw-r--r-- 1 cry0l1t3 cry0l1t3  348 Sep 25 00:55 cry0l1t3.pub
-rw-r--r-- 1 root     root     1872 Sep 19 17:27 id_rsa
-rw-r--r-- 1 root     root      348 Sep 19 17:28 id_rsa.pub
-rw-r--r-- 1 root     root        0 Sep 19 17:22 nfs.share
```

```bash
JonaLF@htb[/htb]$ ls -n mnt/nfs/

total 16
-rw-r--r-- 1 1000 1000 1872 Sep 25 00:55 cry0l1t3.priv
-rw-r--r-- 1 1000 1000  348 Sep 25 00:55 cry0l1t3.pub
-rw-r--r-- 1    0 1000 1221 Sep 19 18:21 backup.sh
-rw-r--r-- 1    0    0 1872 Sep 19 17:27 id_rsa
-rw-r--r-- 1    0    0  348 Sep 19 17:28 id_rsa.pub
-rw-r--r-- 1    0    0    0 Sep 19 17:22 nfs.share
```

> **Nota:** Si `root_squash` está activo, incluso el usuario `root` del cliente no podrá modificar ciertos ficheros (p.ej. `backup.sh`).

### Desmontar

```bash
JonaLF@htb[/htb]$ cd ..
JonaLF@htb[/htb]$ sudo umount ./target-NFS
```

---

## Ideas de escalada con NFS

- Si podemos **escribir** en una export sin `root_squash`, es posible subir binarios/SUID para ejecutar con privilegios en el host servidor.
- Alinear **UID/GID** locales con los del servidor puede permitir leer/escribir archivos que de otro modo no serían accesibles.
- Revisar claves/credenciales (p. ej., `id_rsa`) expuestas en exportaciones.

---

## Resumen rápido

- Puertos clave: **111/tcp/udp** (*rpcbind*), **2049/tcp/udp** (*nfs*).  
- Revisar exportaciones con `showmount -e`, montar con `mount -t nfs`.  
- Validar opciones peligrosas en `/etc/exports`: `no_root_squash`, `rw`, `insecure`, `nohide`.  
- Usar NSE: `--script nfs*` para **listar** y **statfs**.
