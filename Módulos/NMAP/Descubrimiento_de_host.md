# Descubrimiento del anfitrión — Resumen y comandos

## Objetivo

El descubrimiento de anfitriones (host discovery) busca identificar qué máquinas están activas en una red antes de proceder con escaneos más profundos. Es la primera fase en una prueba interna de red para priorizar recursos y evitar escaneos innecesarios.

---

## Principios clave

* Guarda siempre cada escaneo (`-oA`) para documentación y comparación.
* Un host "no respondiente" puede estar detrás de un firewall que bloquea ICMP o evita respuestas a sondas. No descartes hosts sin más comprobaciones manuales.
* Nmap usa diferentes técnicas (ARP, ICMP, TCP) según el contexto de la red; entiende cuál se aplica y por qué.

---

## Comandos y ejemplos útiles

### 1) Escanear un rango de red buscando hosts activos

```bash
sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5
```

* `-sn`: desactiva el escaneo de puertos (solo host discovery).
* `-oA tnet`: guarda resultados en `tnet.nmap`, `tnet.gnmap`, `tnet.xml`.
* El `grep | cut` extrae las IPs encontradas.

### 2) Escanear a partir de una lista de hosts

Crear `hosts.lst` con IPs y luego:

```bash
sudo nmap -sn -oA tnet -iL hosts.lst | grep for | cut -d" " -f5
```

* `-iL hosts.lst`: lee objetivos desde el fichero.

### 3) Escanear múltiples IPs o un rango pequeño

```bash
sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20
# o usando rango
sudo nmap -sn -oA tnet 10.129.2.18-20
```

### 4) Escanear una sola IP (ping scan) y ver MAC

```bash
sudo nmap 10.129.2.18 -sn -oA host
```

Salida ejemplo:

```
Nmap scan report for 10.129.2.18
Host is up (0.087s latency).
MAC Address: DE:AD:00:00:BE:EF
```

### 5) Forzar uso de ICMP Echo Request y ver trazas de paquetes

Por defecto Nmap puede usar ARP en redes locales. Para forzar ICMP y ver paquetes:

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace
```

* `-PE`: usar ICMP Echo Requests.
* `--packet-trace`: muestra todos los paquetes enviados/recibidos.

### 6) Ver razón del resultado (por qué Nmap considera alive)

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --reason
```

Salida ejemplo:

```
Host is up, received arp-response (0.028s latency).
```

* `--reason` explica por qué Nmap clasificó el estado.

### 7) Deshabilitar ARP pings (para evitar ARP automatico en LAN)

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping
```

* `--disable-arp-ping` evita ARP probes y fuerza el uso del método de descubrimiento especificado (por ejemplo, ICMP).

---

## Consideraciones prácticas

* En redes LAN, Nmap suele usar ARP para descubrir hosts (rápido y fiable). Si necesitas probar con ICMP por alguna razón (evitar ARP que pueda ser filtrado o para reproducir condiciones remotas), usa `--disable-arp-ping` + `-PE`.
* Si un host aparece "down", intenta varias técnicas (ICMP, TCP SYN a puertos comunes, ARP). Los IDS/firewalls pueden responder selectivamente.
* Cuando trabajes con listas o rangos grandes, guarda los resultados y procesa las salidas con scripts (grep/cut/awk) para extraer listas limpias de hosts activos.

---

## Recursos

* Estrategias de descubrimiento de hosts (Nmap Book): [https://nmap.org/book/host-discovery-strategies.html](https://nmap.org/book/host-discovery-strategies.html)

*Fin del documento.*
