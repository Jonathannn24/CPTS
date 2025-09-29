# Términos comunes en Pentesting

Las pruebas de penetración (pentesting) y la piratería informática abarcan un campo muy amplio.  
Aquí se presentan algunos de los términos y tecnologías más comunes que debemos comprender firmemente para avanzar en módulos fundamentales y máquinas sencillas de HTB (Hack The Box).

---

## 🔹 ¿Qué es un Shell?
Un **shell** es un programa que recibe comandos del usuario y los pasa al sistema operativo para ejecutar funciones específicas.  
En Linux, el shell más usado es **Bash (Bourne Again Shell)**, aunque existen otros como Zsh, Tcsh, Ksh o Fish.

**“Obtener un shell”** en un sistema significa haberlo explotado hasta conseguir acceso interactivo para ejecutar comandos.

### Tipos principales de Shell
| Tipo           | Descripción |
|----------------|-------------|
| **Reverse shell** | El host comprometido inicia una conexión hacia nuestra máquina atacante, que escucha (listener). |
| **Bind shell**    | El host comprometido abre un puerto y espera que nos conectemos desde nuestra máquina atacante. |
| **Web shell**     | Permite ejecutar comandos a través de una aplicación web (ej. scripts PHP cargados en el servidor). |

> Los shells pueden estar escritos en múltiples lenguajes: Python, Perl, Go, Bash, PHP, etc.

---

## 🔹 ¿Qué es un Puerto?
Un **puerto** es como una puerta/ventana de un sistema remoto: si está abierto sin protección, puede ser un vector de ataque.  
Son puntos virtuales gestionados por el sistema operativo que permiten diferenciar tipos de tráfico.

- **HTTP**: puerto 80 (TCP).  
- **HTTPS**: puerto 443 (TCP).  
- **SSH**: puerto 22 (TCP).  

Existen **65,535 puertos TCP (Transmission Control Protocol)** y **65,535 puertos UDP (User Datagram Protocol)**.

### Diferencias entre TCP y UDP
- **TCP (Transmission Control Protocol)**: orientado a conexión (handshake, fiabilidad).  
- **UDP (User Datagram Protocol)**: sin conexión, más rápido pero menos fiable, ideal para aplicaciones en tiempo real.

### Puertos comunes
| Puerto(s) | Protocolo | Servicio |
|-----------|-----------|----------|
| 20/21 (TCP) | FTP (File Transfer Protocol) | Transferencia de archivos |
| 22 (TCP)   | SSH (Secure Shell) | Acceso remoto seguro |
| 23 (TCP)   | Telnet | Acceso remoto inseguro |
| 25 (TCP)   | SMTP (Simple Mail Transfer Protocol) | Correo electrónico |
| 80 (TCP)   | HTTP (Hypertext Transfer Protocol) | Web sin cifrado |
| 161 (TCP/UDP) | SNMP (Simple Network Management Protocol) | Administración de red |
| 389 (TCP/UDP) | LDAP (Lightweight Directory Access Protocol) | Directorios |
| 443 (TCP)  | HTTPS (Hypertext Transfer Protocol Secure) | Web cifrado (SSL/TLS) |
| 445 (TCP)  | SMB (Server Message Block) | Compartición de archivos |
| 3389 (TCP) | RDP (Remote Desktop Protocol) | Escritorio remoto Windows |

👉 Como pentesters, debemos memorizar y reconocer rápidamente los puertos más comunes.

---

## 🔹 ¿Qué es un Servidor Web?
Un **servidor web** es una aplicación que procesa tráfico HTTP/HTTPS y conecta a usuarios con una aplicación web.  
Normalmente escucha en los puertos **80 (HTTP)** y **443 (HTTPS)**.

Los servidores web son **superficie de ataque crítica**, ya que están expuestos públicamente.  
Si la aplicación web es vulnerable, puede comprometer todo el servidor back-end.

---

## 🔹 OWASP Top 10 (2021)
El **OWASP (Open Web Application Security Project)** mantiene la lista de las 10 vulnerabilidades más críticas en aplicaciones web.  
Es un referente para cualquier evaluación de seguridad.

| Nº | Categoría | Descripción |
|----|-----------|-------------|
| 1 | **Control de acceso roto** | Permite a usuarios acceder a cuentas/datos/funciones no autorizadas. |
| 2 | **Fallos criptográficos** | Uso incorrecto de criptografía → exposición de datos. |
| 3 | **Inyección** | Entrada no validada → SQLi, command injection, LDAP injection, etc. |
| 4 | **Diseño inseguro** | Aplicación sin seguridad en el diseño. |
| 5 | **Mala configuración de seguridad** | Configuraciones por defecto, errores de despliegue. |
| 6 | **Componentes vulnerables/obsoletos** | Dependencias inseguras o sin soporte. |
| 7 | **Fallos de identificación y autenticación** | Ataques contra login, gestión de sesiones. |
| 8 | **Fallas en integridad de software/datos** | Dependencias de fuentes no confiables. |
| 9 | **Fallas en registro y monitoreo** | No permite detectar y responder a ataques. |
| 10 | **SSRF (Server-Side Request Forgery)** | Permite forzar a la aplicación a realizar solicitudes no autorizadas. |

---

## 📌 Conclusión
- Un **shell** nos da control directo sobre un host.  
- Un **puerto abierto** indica servicios que pueden ser explotados.  
- Los **servidores web** son vectores críticos de ataque.  
- El **OWASP Top 10** es la base para pruebas de aplicaciones web.  

👉 Dominar estos términos es esencial para avanzar en pentesting y practicar eficazmente en HTB y entornos reales.
