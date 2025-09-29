# Enumeración web

Al realizar el escaneo de servicios, a menudo nos encontraremos con servidores web que se ejecutan en los puertos **80 (HTTP)** y **443 (HTTPS)**. Los servidores web alojan aplicaciones web (a veces más de una) que proporcionan una superficie de ataque considerable y un objetivo de alto valor durante una prueba de penetración. Una enumeración web adecuada es fundamental, especialmente cuando una organización no expone muchos servicios o esos servicios están correctamente parchados.

---

## Gobuster: Directorios y archivos

Después de descubrir una aplicación web, conviene verificar si existen archivos o directorios ocultos que no estén destinados al acceso público. Podemos utilizar herramientas como **ffuf** o **GoBuster** para realizar esta enumeración.

GoBuster permite realizar fuerza bruta de DNS, vhosts y directorios. Para este módulo, nos interesa el modo `dir`.

```bash
gobuster dir -u http://10.10.10.121/ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

Salida de ejemplo:

```
/.hta (Status: 403)
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/index.php (Status: 200)
/server-status (Status: 403)
/wordpress (Status: 301)
```

* **200 OK:** recurso accesible.
* **403 Forbidden:** acceso prohibido (puede ser interesante).
* **301 Moved Permanently:** redirección.

En este caso, se detecta `/wordpress`, revelando una instalación de **WordPress**, lo que abre posibilidades de explotación si está en modo de configuración.

---

## Enumeración de subdominios (DNS)

Podemos usar **GoBuster** en modo `dns` para descubrir subdominios:

```bash
git clone https://github.com/danielmiessler/SecLists
sudo apt install seclists -y
gobuster dns -d inlanefreight.com -w /usr/share/SecLists/Discovery/DNS/namelist.txt
```

Ejemplo de resultados:

```
Found: blog.inlanefreight.com
Found: customer.inlanefreight.com
Found: my.inlanefreight.com
```

Estos subdominios pueden contener paneles de administración o servicios adicionales.

---

## Captura de banners y encabezados

Podemos usar **cURL** para recuperar encabezados HTTP:

```bash
curl -IL https://www.inlanefreight.com
```

Ejemplo de salida:

```
HTTP/1.1 200 OK
Server: Apache/2.4.29 (Ubuntu)
Content-Type: text/html; charset=UTF-8
```

Otra herramienta útil es **EyeWitness**, que captura pantallas de aplicaciones web y ayuda a identificar configuraciones por defecto.

---

## WhatWeb: fingerprinting

La herramienta **WhatWeb** detecta tecnologías y versiones:

```bash
whatweb 10.10.10.121
whatweb --no-errors 10.10.10.0/24
```

Ejemplo de salida:

```
http://10.10.10.121 [200 OK] Apache[2.4.41], HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], Title[PHP 7.4.3 - phpinfo()]
```

---

## Certificados SSL/TLS

Los certificados HTTPS pueden contener información útil como correo electrónico y organización, que podrían aprovecharse (p. ej., para phishing si está dentro del alcance).

---

## Robots.txt

El archivo `robots.txt` puede indicar directorios sensibles.

```bash
curl http://10.10.10.121/robots.txt
```

Ejemplo:

```
User-agent: *
Disallow: /private
Disallow: /uploaded_files
```

Acceder a `/private` puede revelar un panel de administración.

---

## Código fuente

Al revisar el código fuente (`CTRL+U` en el navegador), podemos encontrar comentarios con información sensible, como credenciales de prueba.

---

## Resumen

* Usa **GoBuster** para directorios y subdominios.
* Revisa códigos HTTP (200, 301, 403) cuidadosamente.
* Usa **curl** y **WhatWeb** para fingerprinting.
* No olvides inspeccionar **robots.txt**, certificados y código fuente.
* Siempre actúa dentro del **alcance autorizado**.

---


