# Escalada de privilegios

La entrada inicial suele darnos un usuario con pocos privilegios. Para obtener control total necesitamos elevar privilegios a **root** (Linux) o **Administrator/SYSTEM** (Windows). A continuación, un resumen práctico con técnicas, comandos y referencias.

---

## Listas de verificación (checklists)

* **HackTricks** — Linux/Windows PrivEsc: guías y comandos de enumeración.
* **PayloadsAllTheThings** — (Cargas útiles Todas Las Cosas) recopilación de listas y PoC (Proof of Concept).

> Recomendación: practicar y entender **por qué** funciona cada técnica, no solo ejecutar comandos.

---

## Scripts de enumeración

> Aviso: generan "ruido" y pueden activar AV (Antivirus) / EDR (Endpoint Detection and Response).

* **PEASS (Privilege Escalation Awesome Scripts SUITE)**: `linpeas.sh` (Linux), `winPEAS.exe` (Windows).
* **LinEnum**, **linuxprivchecker** (Linux).
* **Seatbelt**, **JAWS** (Windows).

### LinPEAS (Linux) — ejemplo de ejecución

```bash
./linpeas.sh
```

Salida típica (recorte):

```
Linux Privesc Checklist: https://book.hacktricks.xyz/linux-unix/linux-privilege-escalation-checklist
OS: Linux version 3.9.0-73-generic
User & Groups: uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

---

## Vectores frecuentes

### 1) Explotaciones del kernel (Kernel exploits)

* Buscar por versión: `uname -a` (Linux) / `systeminfo` (Windows).
* Ejemplo hallazgo: **CVE (Common Vulnerabilities and Exposures)-2016-5195 Dirty COW** para kernels antiguos.
* Precaución: pueden inestabilizar el sistema; probar antes en laboratorio y coordinar con el cliente.

### 2) Software vulnerable

* **Linux**: listar paquetes/versión

```bash
dpkg -l | less
```

* **Windows**: revisar

```
C:\Program Files\
C:\Program Files (x86)\
```

* Buscar exploits públicos (SearchSploit/Exploit‑DB, boletines del fabricante).

### 3) Privilegios del usuario actual

#### Sudo (Linux)

Comprobar privilegios:

```bash
sudo -l
```

Ejemplo (todos los comandos):

```
User user1 may run the following commands on ExampleServer:
    (ALL : ALL) ALL
```

Elevar a root:

```bash
sudo su -
```

Ejemplo con `NOPASSWD`:

```
(user : user) NOPASSWD: /bin/echo
```

Ejecutar como otro usuario:

```bash
sudo -u user /bin/echo "Hello World!"
```

> **GTFOBins**: catálogo de binarios explotables con `sudo`/`SUID`.
> **LOLBAS (Living Off The Land Binaries And Scripts)**: equivalentes en Windows.

#### SUID/SGID (Linux)

Buscar binarios SUID/SGID (SetUID/SetGID):

```bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
```

Consultar GTFOBins para técnicas de escalada según el binario.

#### Privilegios de tokens (Windows)

Listar y abusar de privilegios como `SeImpersonatePrivilege` (técnicas Juicy Potato/PrintSpoofer, etc.).

### 4) Tareas programadas / Cron

**Linux (Cron):**
Rutas habituales de interés (escritura peligrosa):

```
/etc/crontab
/etc/cron.d/
/var/spool/cron/crontabs/root
```

Idea: si un cron ejecuta un script editable por nosotros, insertar **reverse shell** o elevar permisos.

**Windows (Scheduled Tasks/Servicios):**

* Buscar tareas/servicios con binarios en rutas escribibles o sin comillas (unquoted service paths).

### 5) Credenciales expuestas

* Revisar archivos de **configuración**, **logs**, historiales: `~/.bash_history`, `PSReadLine` (Windows), `.env`, `config.php`, etc.

Ejemplo hallazgo:

```
/var/www/html/config.php: $conn = new mysqli(localhost, 'db_user', 'password123');
```

Probar **password reuse (reutilización de contraseñas)**:

```bash
su -
# Password: password123
whoami
```

### 6) Claves SSH (Secure Shell)

* Ruta típica: `/home/<user>/.ssh/` o `/root/.ssh/`.
* Si podemos **leer** `id_rsa`, usarla para acceder:

```bash
vim id_rsa
chmod 600 id_rsa
ssh root@10.10.10.10 -i id_rsa
```

* Si tenemos **escritura** en `authorized_keys`, plantar nuestra clave:

Generar par de claves:

```bash
ssh-keygen -f key
```

Añadir la pública en el remoto:

```bash
echo "ssh-rsa AAAAB...SNIP...M= user@parrot" >> /root/.ssh/authorized_keys
```

Acceder:

```bash
ssh root@10.10.10.10 -i key
```

---

## Consejos operativos

* **Siempre** dentro de alcance y con autorización.
* Registrar comandos, parámetros y evidencias.
* Si un vector falla, probar alternativas (otro binario SUID, otro servicio, técnica manual, etc.).
* **Limpieza** al finalizar (borrar scripts, claves y artefactos colocados).

---

## Mini‑checklist rápida

* [ ] Ejecutar `linpeas.sh` / `winPEAS.exe` y revisar highlights.
* [ ] `sudo -l`, binarios **SUID/SGID**, permisos de escritura en rutas críticas.
* [ ] `crontab`/`/etc/cron*`, servicios/tareas (unquoted paths, binarios en rutas escribibles).
* [ ] Configs/logs/historiales → credenciales; probar **password reuse**.
* [ ] Paquetes/software/versiones → buscar CVE/PoC.
* [ ] Claves SSH: `id_rsa` legible o `authorized_keys` escribible.

---

*Fin del documento.*
