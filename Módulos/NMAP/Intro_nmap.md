# Introducción a Nmap

## Resumen

Nmap (Network Mapper) es una herramienta de código abierto para auditoría de seguridad y análisis de redes. Permite descubrir hosts activos, identificar puertos abiertos, detectar servicios (nombre y versión), inferir sistemas operativos y comprobar la presencia de firewalls o IDS. Nmap es fundamental tanto para administradores de red como para pentesters: ayuda a auditar seguridad, simular ataques y evaluar configuraciones.

---

## Casos de uso

* Auditoría de seguridad de redes.
* Simulación de pruebas de penetración.
* Verificación de reglas de firewall e IDS.
* Identificación de servicios y evaluación de vulnerabilidad.

---

## Arquitectura / Técnicas principales

Nmap agrupa sus capacidades en varias fases y técnicas:

* **Descubrimiento de hosts**: encontrar qué equipos están arriba en una red.
* **Escaneo de puertos**: detectar puertos TCP/UDP abiertos o filtrados.
* **Detección y enumeración de servicios**: identificar la aplicación que escucha en un puerto (ej. Apache, OpenSSH) y, cuando es posible, su versión.
* **Detección de sistema operativo**: estimar SO y versiones mediante fingerprints.
* **Nmap Scripting Engine (NSE)**: ejecutar scripts para interacción programable (vulnerabilidades, enumeración, banners, etc.).

---

## Sintaxis básica

```bash
nmap <scan types> <options> <target>
```

Ejemplo general de invocación:

```bash
nmap -sV --open -oA myscan 10.0.0.5
```

---

## Técnicas de escaneo comunes

Mostrar algunas técnicas que ofrece Nmap (resumen):

* `-sS` / `-sT`: TCP SYN (half-open) / Connect scan.
* `-sU`: UDP scan.
* `-sN` / `-sF` / `-sX`: TCP Null / FIN / Xmas scans.
* `-sI`: Idle (zombie) scan.
* `-sO`: IP protocol scan.
* `-sV`: Version detection (identificar versión del servicio).
* `-O`: OS fingerprinting (detección de sistema operativo).
* `-p-`: escanear todos los puertos (1-65535).
* `-sC`: ejecutar scripts NSE por defecto.
* `--script <script>`: ejecutar scripts NSE específicos.
* `-oA <basename>`: generar salida en los tres formatos (normal, xml, gnmap).

Ejecuta `nmap --help` para ver todas las opciones y técnicas.

---

## Cómo funciona (breve explicación del TCP-SYN scan `-sS`)

El escaneo TCP-SYN envía paquetes con la bandera SYN sin completar el handshake de tres vías:

* Si el host responde con **SYN-ACK** → puerto **open**.
* Si responde con **RST** → puerto **closed**.
* Si no responde → puerto **filtered** (posible firewall que descarta paquetes).

Este método es rápido y evita completar conexiones completas, con lo que a menudo pasa desapercibido por algunos logs.

---

## Ejemplo práctico

```bash
sudo nmap -sS localhost
```

Salida de ejemplo:

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-11 22:50 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000010s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
5432/tcp open  postgresql
5901/tcp open  vnc-1

Nmap done: 1 IP address (1 host up) scanned in 0.18 seconds
```

Explicación rápida: la primera columna es el **puerto**, la segunda el **estado** (open/closed/filtered) y la tercera el **servicio** detectado.

---

## Buenas prácticas

* Guarda la salida con `-oA` para tener formatos reutilizables en reporting.
* Comienza con un escaneo rápido (`-sV --open`) y luego amplía (por ejemplo `-p-`, `-sC`, o NSE scripts específicos).
* Ten en cuenta el impacto en la red: `-T` timing templates controlan la agresividad (`-T0` más sigiloso, `-T5` muy agresivo).
* Documenta timestamps y resultados para reproducibilidad.
* Usa `--script` con precaución en entornos productivos (algunos scripts son intrusivos).

---

## Recursos y lecturas recomendadas

* Sitio oficial: [https://nmap.org](https://nmap.org)
* Nmap Book (Guía oficial)
* Nmap Scripting Engine (NSE) docs
* Práctica: experimentar en laboratorios como HTB o entornos de pruebas aislados.

---

*Fin del documento.*
