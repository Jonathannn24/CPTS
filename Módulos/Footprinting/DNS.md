# DNS — Resumen

> Contenido resumido y con los **comandos incluidos**. Copia y pega directamente.

---

## ¿Qué es DNS?
DNS (Domain Name System) resuelve nombres de dominio (p. ej. `www.ejemplo.com`) en direcciones IP. Es un sistema distribuido, no centralizado, con miles de servidores de nombres que cooperan para resolver consultas.

### Tipos de servidores DNS
- **Root servers** — manejan los TLD (13 conjuntos a nivel mundial).  
- **Authoritative nameserver** — autoritativos para una zona (respuesta vinculante).  
- **Non-authoritative nameserver** — servidores recursivos/no autoritativos que consultan a otros.  
- **Caching DNS server** — guarda respuestas por TTL.  
- **Forwarding server** — reenvía consultas a otros servidores.  
- **Resolver** — realiza resolución local (cliente/OS/router).

### Privacidad / cifrado
Por defecto DNS no está cifrado. Opciones modernas: **DoT (DNS over TLS)**, **DoH (DNS over HTTPS)**, **DNSCrypt**.

---

## Tipos de registros DNS (los más importantes)
- `A` — IPv4
- `AAAA` — IPv6
- `MX` — servidores de correo
- `NS` — servidores de nombres de la zona
- `TXT` — texto libre (SPF, verificaciones, etc.)
- `CNAME` — alias a otro nombre
- `PTR` — búsqueda inversa (IP → nombre)
- `SOA` — información de la zona (administrador, serial, tiempos)

Ejemplo rápido: obtener el SOA
```bash
dig soa www.inlanefreight.com
```

---

## Archivos y configuración (bind9 / BIND)
- `named.conf` y ficheros incluidos (`named.conf.local`, `named.conf.options`, etc.)
- **zone files** (archivo de zona): formato BIND, debe contener 1 `SOA` y al menos 1 `NS`.
- **reverse zone files**: contienen registros `PTR` para búsquedas inversas.

Ejemplo de entrada en `named.conf.local`:
```text
zone "domain.com" {
    type master;
    file "/etc/bind/db.domain.com";
    allow-update { key rndc-key; };
};
```

Ejemplo de archivo de zona (forward):
```text
$ORIGIN domain.com
$TTL 86400
@     IN SOA dns1.domain.com. hostmaster.domain.com. (
          2001062501 ; serial
          21600      ; refresh
          3600       ; retry
          604800     ; expire
          86400 )    ; minimum TTL

      IN NS ns1.domain.com.
      IN NS ns2.domain.com.
      IN MX 10 mx.domain.com.
server1 IN A 10.129.14.5
www     IN CNAME server2
```

Ejemplo de reverse zone (fragmento):
```text
$ORIGIN 14.129.10.in-addr.arpa
5    IN PTR server1.domain.com.
```

---

## Configuraciones peligrosas (a revisar)
- `allow-query` — quién puede consultar el servidor.
- `allow-recursion` — quién puede solicitar resolución recursiva (no dejarlo abierto al público).
- `allow-transfer` — quién puede hacer transferencias de zona (AXFR).
- `zone-statistics` — puede filtrar información sensible si se expone.

---

## Huella y enumeración (comandos útiles)
> Incluyo los comandos mostrados en el texto de origen. Úsalos contra objetivos que tengas permiso de auditar.

### Consultas básicas con `dig`
- Obtener `NS` de una zona (especificando servidor):
```bash
dig ns inlanefreight.htb @10.129.14.128
```

- Obtener `SOA`:
```bash
dig soa www.inlanefreight.com
```

- Consultar versión (CHAOS TXT) — puede no estar expuesto:
```bash
dig CH TXT version.bind @10.129.120.85
```

- `ANY` (muestra registros que el servidor revela):
```bash
dig any inlanefreight.htb @10.129.14.128
```

### Transferencia de zona (AXFR)
> Si `allow-transfer` está mal configurado, se puede obtener todo el archivo de zona.
```bash
dig axfr inlanefreight.htb @10.129.14.128
# o zona interna
dig axfr internal.inlanefreight.htb @10.129.14.128
```

### Fuerza bruta / enumeración de subdominios
- **Bucle Bash** con una lista de subdominios (SecLists):
```bash
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt); do
  dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt
done
```

- Herramienta `dnsenum`:
```bash
dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt   -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb
```

(El ejemplo anterior muestra cómo `dnsenum` intentó transferencias de zona y fuerza bruta.)

---

## Buenas prácticas / recomendaciones
- No permitir recursion para clientes desconocidos (`allow-recursion` restringido).
- Restringir `allow-transfer` solo a servidores secundarios de confianza.
- No exponer información de versión (evitar `version.bind` expuesto).
- Revisar registros `TXT` (SPF/DMARC) por datos que indiquen proveedores usados (p. ej. `include:_spf.google.com`).
- Monitorizar cambios de serial en `SOA` y revisar logs.
- Usar DoT/DoH si la privacidad del cliente es una preocupación.

---

## Ejemplos prácticos (resumen rápido de comandos mostrados)
```bash
# SOA
dig soa www.inlanefreight.com

# NS
dig ns inlanefreight.htb @10.129.14.128

# version.bind (CHAOS TXT)
dig CH TXT version.bind @10.129.120.85

# ANY
dig any inlanefreight.htb @10.129.14.128

# AXFR (transferencia de zona)
dig axfr inlanefreight.htb @10.129.14.128
dig axfr internal.inlanefreight.htb @10.129.14.128

# Fuerza bruta (bucle con SecLists)
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt);do
  dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt
done

# dnsenum (enumeración automatizada)
dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt   -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb
```
