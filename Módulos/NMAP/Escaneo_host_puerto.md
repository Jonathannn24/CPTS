# Escaneo de host y puerto

## Resumen rápido

* **Objetivo**: pasar del simple “host vivo” a un **mapa claro de puertos/servicios**, sus **versiones** y pistas de **SO**.
* **Estados de puerto (6)**: `open`, `closed`, `filtered`, `unfiltered`, `open|filtered`, `closed|filtered`.
* **TCP**: por defecto Nmap (root) usa **SYN scan (-sS)** sobre los **1000 puertos más comunes**; sin root, usa **Connect scan (-sT)**.
* **Selección de puertos**: `-p`, rangos (`-p 22-445`), todos (`-p-`), top N (`--top-ports`), rápido (`-F`).
* **Trazas y razones**: `--packet-trace`, `--reason`, desactivar resoluciones/probes con `-n`, `-Pn`, `--disable-arp-ping` para entender respuestas.
* **UDP (-sU)**: más **lento** y con muchos `open|filtered`; mirar **ICMP type 3/code 3** para `closed`.
* **Detección de versiones (-sV)**: sondeo activo para banner/firma de servicio (p. ej., Samba 3.X–4.X), útil para pivote a exploits.

---

## Estados de puertos

* **open**: se establece conexión (TCP/UDP/SCTP).
* **closed**: en TCP suele verse **RST** de respuesta.
* **filtered**: sin respuesta o **ICMP error** → posible firewall.
* **unfiltered**: solo en **ACK scan** (accesible, estado real desconocido).
* **open|filtered**: sin respuesta concluyente.
* **closed|filtered**: típico de **Idle scan**.

---

## Escaneando los 10 principales puertos TCP

[!bash!]$ sudo nmap 10.129.2.28 --top-ports=10

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:36 CEST
Nmap scan report for 10.129.2.28
Host is up (0.021s latency).

PORT     STATE    SERVICE
21/tcp   closed   ftp
22/tcp   open     ssh
23/tcp   closed   telnet
25/tcp   open     smtp
80/tcp   open     http
110/tcp  open     pop3
139/tcp  filtered netbios-ssn
443/tcp  closed   https
445/tcp  filtered microsoft-ds
3389/tcp closed   ms-wbt-server
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 1.44 seconds
```

**Claves**: `--top-ports=10` usa la base de datos de puertos más frecuentes.

---

## Nmap: trazar paquetes (SYN cerrado en 21/tcp)

[!bash!]$ sudo nmap 10.129.2.28 -p 21 --packet-trace -Pn -n --disable-arp-ping

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:39 CEST
SENT (0.0429s) TCP 10.10.14.2:63090 > 10.129.2.28:21 S ttl=56 id=57322 iplen=44  seq=1699105818 win=1024 <mss 1460>
RCVD (0.0573s) TCP 10.129.2.28:21 > 10.10.14.2:63090 RA ttl=64 id=0 iplen=40  seq=0 win=0
Nmap scan report for 10.129.2.28
Host is up (0.014s latency).

PORT   STATE  SERVICE
21/tcp closed ftp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds
```

**Lectura**: `RA` = **RST+ACK** ⇒ **closed**. `-n`, `-Pn`, `--disable-arp-ping` quitan “ruido” de DNS/ARP/ICMP.

---

## Connect scan (-sT) en 443/tcp

[!bash!]$ sudo nmap 10.129.2.28 -p 443 --packet-trace --disable-arp-ping -Pn -n --reason -sT

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:26 CET
CONN (0.0385s) TCP localhost > 10.129.2.28:443 => Operation now in progress
CONN (0.0396s) TCP localhost > 10.129.2.28:443 => Connected
Nmap scan report for 10.129.2.28
Host is up, received user-set (0.013s latency).

PORT    STATE SERVICE REASON
443/tcp open  https   syn-ack

Nmap done: 1 IP address (1 host up) scanned in 0.04 seconds
```

**Nota**: -sT es preciso pero **menos sigiloso** (completa el 3-way handshake y suele loguearse).

---

## Puertos filtrados (drop vs reject)

### Ejemplo 139/tcp (drop → reintentos)

[!bash!]$ sudo nmap 10.129.2.28 -p 139 --packet-trace -n --disable-arp-ping -Pn

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:45 CEST
SENT (0.0381s) TCP 10.10.14.2:60277 > 10.129.2.28:139 S ttl=47 id=14523 iplen=44  seq=4175236769 win=1024 <mss 1460>
SENT (1.0411s) TCP 10.10.14.2:60278 > 10.129.2.28:139 S ttl=45 id=7372 iplen=44  seq=4175171232 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up.

PORT    STATE    SERVICE
139/tcp filtered netbios-ssn
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds
```

### Ejemplo 445/tcp (reject → ICMP type 3/code 3)

[!bash!]$ sudo nmap 10.129.2.28 -p 445 --packet-trace -n --disable-arp-ping -Pn

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:55 CEST
SENT (0.0388s) TCP 10.129.2.28:52472 > 10.129.2.28:445 S ttl=49 id=21763 iplen=44  seq=1418633433 win=1024 <mss 1460>
RCVD (0.0487s) ICMP [10.129.2.28 > 10.129.2.28 Port 445 unreachable (type=3/code=3) ] IP [ttl=64 id=20998 iplen=72 ]
Nmap scan report for 10.129.2.28
Host is up (0.0099s latency).

PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.05 seconds
```

**Claves**: `drop` ⇒ esperas/reintentos; `reject` ⇒ ICMP “port unreachable”. Ambos terminan en **filtered**.

---

## UDP (-sU): estados y tiempos

[!bash!]$ sudo nmap 10.129.2.28 -F -sU

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:01 CEST
Nmap scan report for 10.129.2.28
Host is up (0.059s latency).
Not shown: 95 closed ports
PORT     STATE         SERVICE
68/udp   open|filtered dhcpc
137/udp  open          netbios-ns
138/udp  open|filtered netbios-dgm
631/udp  open|filtered ipp
5353/udp open          zeroconf
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 98.07 seconds
```

### UDP “open” con respuesta

[!bash!]$ sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason

```
SENT (0.0367s) UDP 10.10.14.2:55478 > 10.129.2.28:137 ttl=57 id=9122 iplen=78
RCVD (0.0398s) UDP 10.129.2.28:137 > 10.10.14.2:55478 ttl=64 id=13222 iplen=257
...
137/udp open  netbios-ns udp-response ttl 64
```

### UDP “closed” por ICMP type 3/code 3

[!bash!]$ sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 100 --reason

```
RCVD (0.1498s) ICMP [10.129.2.28 > 10.10.14.2 Port unreachable (type=3/code=3) ]
...
100/udp closed unknown port-unreach ttl 64
```

### UDP “open|filtered” sin respuesta

[!bash!]$ sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 138 --reason

```
SENT ...
SENT ...
...
138/udp open|filtered netbios-dgm no-response
```

---

## Detección de versiones (-sV)

[!bash!]$ sudo nmap 10.129.2.28 -Pn -n --disable-arp-ping --packet-trace -p 445 --reason  -sV

```
Service scan sending probe SMBProgNeg ...
Service scan match ... 10.129.2.28:445 is netbios-ssn.  Version: |Samba smbd|3.X - 4.X|workgroup: WORKGROUP|
...
PORT    STATE SERVICE     REASON         VERSION
445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: Host: Ubuntu
```

**Claves**: `-sV` activa el **service probe** y matching de firmas. Útil para pivotar a **CVE/exploits**.

---

## Pistas útiles

* Para **acelerar**: delimita puertos (`-p`), usa `--top-ports`, o `-F`. En UDP, prueba primero puertos críticos (53, 123, 161, 137, 138, 500, 5353…).
* Para **fiabilidad**: repite escaneos con diferentes timings (`-T4` conservadormente) y confirma con trazas.
* Para **sigilo**: favorece `-sS` (root), desincroniza, o usa técnicas como `--scan-delay`, sin olvidar que IDS modernos detectan patrones.
* Para **reporting**: guarda siempre con `-oA` para tener `.nmap`, `.gnmap`, `.xml` y poder parsear luego.

> Más: técnicas de escaneo en [https://nmap.org/book/man-port-scanning-techniques.html](https://nmap.org/book/man-port-scanning-techniques.html)
