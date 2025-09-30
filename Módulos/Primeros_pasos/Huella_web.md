# Web Footprinting (huella web)

## Resumen breve

Enumeramos una aplicación web en `10.129.42.190` y detectamos **Nibbleblog** (motor de blog en **PHP (Hypertext Preprocessor)**). Vimos pistas en el **código fuente HTML** y en directorios listados que revelaron `nibbleblog/`, confirmamos la **versión v4.0.3** (vulnerable a subida de ficheros autenticada), identificamos el **portal admin** (`/admin.php`), comprobamos **listas negras por fuerza bruta** y descubrimos el usuario `admin`. Finalmente, inferimos la contraseña **`nibbles`** a partir de indicios en `config.xml`/títulos del sitio.

Puntos clave:

* Identificación de tecnologías con **WhatWeb**.
* Confirmación de directorios y portal con **Gobuster**.
* Verificación de contenido/versión con **cURL (Client URL)** + `xmllint`.
* Protección anti‑fuerza bruta (blacklist) activa.
* Usuario `admin` confirmado; contraseña deducida por contexto ("nibbles").

---

## Comandos usados

### Fingerprinting con WhatWeb

```bash
whatweb 10.129.42.190
```

Salida esperada (resumen):

```
http://10.129.42.190 [200 OK] Apache[2.4.18], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.129.42.190]
```

### Ver contenido con cURL

```bash
curl http://10.129.42.190
```

Salida (resumen):

```
<b>Hello world!</b>

<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

### Comprobar ruta /nibbleblog con WhatWeb

```bash
whatweb http://10.129.42.190/nibbleblog
```

Salida (resumen):

```
http://10.129.42.190/nibbleblog [301 Moved Permanently] ... RedirectLocation[http://10.129.42.190/nibbleblog/]
http://10.129.42.190/nibbleblog/ [200 OK] ... Cookies[PHPSESSID] ... MetaGenerator[Nibbleblog] ... Title[Nibbles - Yum yum]
```

### Enumeración de directorios con Gobuster

```bash
gobuster dir -u http://10.129.42.190/nibbleblog/ --wordlist /usr/share/seclists/Discovery/Web-Content/common.txt
```

Resultados relevantes:

```
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/admin (Status: 301)
/admin.php (Status: 200)
/content (Status: 301)
/index.php (Status: 200)
/languages (Status: 301)
/plugins (Status: 301)
/README (Status: 200)
/themes (Status: 301)
```

### Confirmar versión en README

```bash
curl http://10.129.42.190/nibbleblog/README
```

Salida (clave):

```
====== Nibbleblog ======
Version: v4.0.3
...
```

### Listado en `content/private` → `users.xml` (formateado)

```bash
curl -s http://10.129.42.190/nibbleblog/content/private/users.xml | xmllint --format -
```

Salida (resumen):

```
<users>
  <user username="admin"> ... </user>
  <blacklist ip="10.10.10.1"> ... </blacklist>
  <blacklist ip="10.10.14.2"> ... </blacklist>
</users>
```

### Enumeración adicional en raíz del sitio

```bash
gobuster dir -u http://10.129.42.190/ --wordlist /usr/share/seclists/Discovery/Web-Content/common.txt
```

Resultados (resumen):

```
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/index.html (Status: 200)
/server-status (Status: 403)
```

### Revisar `config.xml` en `content/private`

```bash
curl -s http://10.129.42.190/nibbleblog/content/private/config.xml | xmllint --format -
```

Salida (fragmento):

```
<config>
  <name>Nibbles</name>
  <slogan>Yum yum</slogan>
  <url>http://10.129.42.190/nibbleblog/</url>
  <notification_email_to>admin@nibbles.com</notification_email_to>
  <notification_email_from>noreply@10.10.10.134</notification_email_from>
  <theme>simpler</theme>
  ...
</config>
```

---

## Conclusiones

* **Tecnología:** Apache + PHP + Nibbleblog (v4.0.3).
* **Páginas clave:** `/nibbleblog/`, `/nibbleblog/admin.php`, `README`, `content/private/users.xml`, `content/private/config.xml`.
* **Usuario válido:** `admin`.
* **Contraseña probable:** `nibbles` (derivado de branding y correo), con lo que se habilita la vía para el módulo **MSF (Metasploit Framework)** de **subida de ficheros** autenticada para Nibbleblog 4.0.3.

> Siguiente paso sugerido: intentar autenticación en `/nibbleblog/admin.php` con `admin:nibbles` y, si procede, ejecutar explotación de *file upload* (CVE (Common Vulnerabilities and Exposures) asociado a Nibbleblog 4.0.3) para obtener **RCE (Remote Code Execution)**.
