# Motor de secuencias de comandos Nmap (NSE)

## Resumen
El **Nmap Scripting Engine (NSE)** permite extender Nmap con scripts en **Lua** para interactuar con servicios, descubrir información adicional y **detectar/explotar** vulnerabilidades. NSE organiza sus scripts en categorías, que pueden ejecutarse por categoría, por nombre o mediante el set de **scripts por defecto**.

---

## Categorías de NSE
| Categoría  | Descripción |
|---|---|
| `auth` | Determinación de credenciales de autenticación. |
| `broadcast` | Descubrimiento de hosts por broadcast; añade hosts descubiertos al escaneo. |
| `brute` | Fuerza bruta de credenciales contra servicios. |
| `default` | Scripts por defecto (equivalente a `-sC`). |
| `discovery` | Enumeración de servicios accesibles. |
| `dos` | Pruebas de denegación de servicio (riesgosas). |
| `exploit` | Intentan explotar vulnerabilidades conocidas. |
| `external` | Usan servicios externos para posprocesar. |
| `fuzzer` | Envío de entradas anómalas para hallar fallos (lento). |
| `intrusive` | Intrusivos; pueden impactar el objetivo. |
| `malware` | Detección de indicios de malware. |
| `safe` | No intrusivos. |
| `version` | Extiende la detección de versiones. |
| `vuln` | Identificación de vulnerabilidades. |

---

## Uso rápido
### Scripts por defecto
```bash
sudo nmap <target> -sC
```

### Por **categoría** (uno o varias)
```bash
sudo nmap <target> --script <category>
```

### Por **nombre de script** (coma-separado)
```bash
sudo nmap <target> --script <script-name>,<script-name>,...
```

---

## Ejemplos prácticos

### 1) SMTP: banner + comandos soportados
```bash
sudo nmap 10.129.2.28 -p 25 --script banner,smtp-commands
```
Salida clave (abreviado):
```
PORT   STATE SERVICE
25/tcp open  smtp
|_banner: 220 inlane ESMTP Postfix (Ubuntu)
|_smtp-commands: inlane, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8,
```
**Qué aprendemos:** distribución (Ubuntu) y comandos SMTP disponibles (útil, p.ej., `VRFY`).

---

### 2) Escaneo **agresivo** (servicio, OS, traceroute y scripts por defecto)
```bash
sudo nmap 10.129.2.28 -p 80 -A
```
Salida clave (abreviado):
```
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.3.4
|_http-title: blog.inlanefreight.com
```
**Qué aprendemos:** servidor web, versión de WordPress, título del sitio y estimación del SO.

---

### 3) **Evaluación de vulnerabilidades** con NSE
```bash
sudo nmap 10.129.2.28 -p 80 -sV --script vuln
```
Salida clave (abreviado):
```
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-enum:
|   /wp-login.php: Possible admin folder
|   /readme.html: Wordpress version: 2
|   /: WordPress version: 5.3.4
|   /wp-includes/images/rss.png: Wordpress version 2.2 found.
|   ...
| http-wordpress-users:
|   Username found: admin
| vulners:
|   cpe:/a:apache:http_server:2.4.29:
|     CVE-2019-0211 7.2 https://vulners.com/cve/CVE-2019-0211
|     CVE-2018-1312 6.8 https://vulners.com/cve/CVE-2018-1312
```
**Qué aprendemos:** rutas/artefactos web, usuario(s) de WordPress, CVEs mapeados al servicio.

---

## Consejos y buenas prácticas
- **Guarda los resultados** (`-oA`) y ejecuta NSE de forma **selectiva** para reducir ruido.
- Usa `-sV` junto con scripts `version`, `discovery` y `vuln` para máxima cobertura inicial.
- Los scripts `dos`, `intrusive` y `exploit` pueden **interrumpir** servicios; evita ejecutarlos sin autorización explícita.
- Lista scripts disponibles en tu sistema: `ls /usr/share/nmap/scripts/` o `nmap --script-help <script>`.

---

## Comandos de referencia (copiar/pegar)
```bash
# Scripts por defecto
sudo nmap <target> -sC

# Ejecutar una categoría completa (p. ej., vuln)
sudo nmap <target> --script vuln

# Ejecutar múltiples scripts por nombre
sudo nmap <target> --script banner,smtp-commands

# Ejemplo SMTP
sudo nmap 10.129.2.28 -p 25 --script banner,smtp-commands

# Escaneo agresivo en HTTP
sudo nmap 10.129.2.28 -p 80 -A

# Vulnerabilidades en HTTP con detección de versión
sudo nmap 10.129.2.28 -p 80 -sV --script vuln
```

---

### Documentación
- NSE docs: https://nmap.org/nsedoc/index.html
