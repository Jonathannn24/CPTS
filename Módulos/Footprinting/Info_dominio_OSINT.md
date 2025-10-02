# Información del dominio (OSINT pasivo)

> **Objetivo:** recopilar, sin tocar el objetivo de forma activa, todo lo posible sobre la presencia en Internet de una empresa (dominios, subdominios, servicios externos y pistas técnicas) para preparar la enumeración activa.

---

## Resumen rápido

* Revisar el **sitio principal** y anotar servicios/tecnologías mencionadas.
* Extraer **SANs** de certificados SSL para descubrir subdominios.
* Consultar **crt.sh** (Certificate Transparency) y normalizar resultados.
* Resolver DNS y **filtrar hosts propios** (excluir proveedores externos).
* Pasar IPs a **Shodan** para huellas rápidas de puertos/servicios.
* Enumerar **registros DNS** (A/MX/NS/TXT/SOA) en busca de más pistas y terceros.

---

## 1) Certificado SSL del dominio principal

* Los certificados a menudo incluyen múltiples **Subject Alternative Names (SANs)** que revelan subdominios activos.
* Recolecta fechas de validez, emisores y todos los **DNS Names**.

> *Ejemplo observado*: SANs como `inlanefreight.htb`, `www.inlanefreight.htb`, `support.inlanefreight.htb`.

---

## 2) Certificate Transparency con crt.sh

Consulta web: `https://crt.sh/?q=<dominio>`

```bash
# Salida en JSON y formateo
curl -s "https://crt.sh/?q=inlanefreight.com&output=json" | jq .

# Subdominios únicos normalizados a partir del JSON
curl -s "https://crt.sh/?q=inlanefreight.com&output=json" | jq . \
  | grep name \
  | cut -d":" -f2 \
  | grep -v "CN=" \
  | cut -d'"' -f2 \
  | awk '{gsub(/\\n/,"\n");}1;' \
  | sort -u
```

> **Salida típica (recortada):**

```
account.ttn.inlanefreight.com
blog.inlanefreight.com
bots.inlanefreight.com
console.ttn.inlanefreight.com
ct.inlanefreight.com
...
www.inlanefreight.com
```

**Notas:**

* Certificados de **Let's Encrypt (R3)** aparecen con frecuencia y son buenos candidatos para pivotes temporales.
* Fechas `not_before`/`not_after` ayudan a inferir actividad reciente.

---

## 3) Resolver y aislar hosts **propios**

Objetivo: quedarnos con subdominios **alojados por la empresa** (evitar SaaS/proveedores que quedan fuera del alcance contractual).

```bash
# Para cada subdominio, mostrar A-records y filtrar aquellos bajo el propio dominio/ASN
for i in $(cat subdomainlist); do
  host "$i" | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4
done
```

> **Ejemplo de resultado:**

```
blog.inlanefreight.com 10.129.24.93
inlanefreight.com 10.129.27.33
matomo.inlanefreight.com 10.129.127.22
www.inlanefreight.com 10.129.127.33
```

Crear lista de IPs para otras consultas:

```bash
# Construir ip-addresses.txt con las IPs filtradas
: > ip-addresses.txt
for i in $(cat subdomainlist); do
  host "$i" | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt
done
```

---

## 4) Huella rápida con Shodan

Usa la CLI autenticada de Shodan para revisar puertos/servicios expuestos y banners.

```bash
for i in $(cat ip-addresses.txt); do
  shodan host "$i"
done
```

> **Qué guardar:** puertos, servicios, versiones, suites TLS, grupos Diffie‑Hellman, ciudad/ASN/Org, y **IPs clave** para pruebas activas (p. ej. `10.129.127.22`).

---

## 5) Registros DNS (visión global)

Inspecciona **A**, **MX**, **NS**, **TXT**, **SOA** para revelar terceros, verificaciones y rangos adicionales.

```bash
dig any inlanefreight.com
```

**Interpretación útil:**

* **A**: IPs de frontend o balanceadores.
* **MX**: gestor de correo (p. ej., Google Workspace) → normalmente fuera de alcance.
* **NS**: proveedor de DNS (p. ej., INWX) → útil para entender delegaciones.
* **TXT**: verificaciones y políticas (SPF/DMARC/DKIM) que **delatan proveedores**: `mailgun.org`, `_spf.google.com`, `spf.protection.outlook.com`, `_spf.atlassian.net`, etc.

> **Pistas extraídas de TXT (ejemplo):**

* **Atlassian** → posible Jira/Confluence/Bitbucket (dev & colab).
* **Google Gmail** → gestión de correo; atención a enlaces públicos **GDrive**.
* **LogMeIn**/**Central/Rescue** → acceso remoto centralizado (alto impacto si credenciales reutilizadas).
* **Mailgun** → APIs SMTP/Webhooks expuestos (probar IDOR/SSRF/verbos HTTP).
* **Outlook/O365** → OneDrive/Azure Blob/SMB (ruta a malconfiguraciones de almacenamiento).
* **INWX** → probable registrador/DNS; valores `MS=` suelen mapear a IDs/usuarios de portal.

---

## 6) Checklist para tu libreta de notas

* [ ] Lista de **subdominios** únicos desde CT logs.
* [ ] Mapa de **proveedores externos** detectados (SaaS, correo, DNS, devtools).
* [ ] Tabla **subdominio → IP → servicio** (Shodan/Nmap posterior).
* [ ] TXT/ SPF/ DMARC/ DKIM archivados con fecha/hora.
* [ ] Candidatos para **pruebas activas** (p. ej., `matomo.inlanefreight.com`).
* [ ] Exclusiones **fuera de alcance** anotadas (p. ej., Google MX, Cloudflare, etc.).

---

## Buenas prácticas

* Mantén todo **pasivo** en esta fase: nada de escaneos/dirbusting.
* Anota **fechas de certificados** y **rotaciones** (indican despliegues recientes).
* Normaliza y **desduplica** datos; versiona tus listas (git).
* Cruza con ASN/Netblocks si es aplicable antes de escaneo activo.

---

## Comandos (copia/pega)

```bash
# 1) CT logs → JSON
curl -s "https://crt.sh/?q=inlanefreight.com&output=json" | jq .

# 2) Subdominios únicos desde crt.sh
curl -s "https://crt.sh/?q=inlanefreight.com&output=json" | jq . \
  | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 \
  | awk '{gsub(/\\n/,"\n");}1;' | sort -u > subdomainlist

# 3) Resolver y filtrar hosts propios
for i in $(cat subdomainlist); do
  host "$i" | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4
done

# 4) Construir lista de IPs
: > ip-addresses.txt
for i in $(cat subdomainlist); do
  host "$i" | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt
done

# 5) Pasar IPs a Shodan
for i in $(cat ip-addresses.txt); do
  shodan host "$i"
done

# 6) Registros DNS completos
dig any inlanefreight.com
```

---

> **Siguiente paso:** con esta base pasiva, prioriza objetivos y pasa a **enumeración activa** (Nmap/HTTP enum) solo sobre los activos **dentro de alcance**.
