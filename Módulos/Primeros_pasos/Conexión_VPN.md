# Conexión mediante VPN (Virtual Private Network)

Una **VPN (Virtual Private Network)** nos permite conectarnos a una red privada como si estuviéramos físicamente en ella.  
Es un canal seguro sobre redes públicas que cifra las comunicaciones para evitar escuchas y accesos no autorizados.

---

## Concepto básico
- Una VPN enruta la conexión de nuestro dispositivo a través de un **servidor VPN** en lugar del ISP (Internet Service Provider).  
- El tráfico parecerá originarse desde la IP pública del servidor VPN, no desde la nuestra.  
- Tipos principales:
  - **VPN basada en cliente:** requiere software en el host para conectarse y acceder a recursos de red completos.  
  - **VPN SSL (Secure Sockets Layer):** usa el navegador como cliente, sin necesidad de software adicional; puede limitarse a aplicaciones web internas o abrir acceso parcial a la red.

---

## ¿Por qué usar una VPN?
- Acceso remoto seguro a redes corporativas.  
- Cifrado del tráfico en redes hostiles (ej. WiFi de aeropuertos).  
- Evitar restricciones de red o cortafuegos.  
- Cierta privacidad al ocultar nuestra IP pública.  

⚠️ Importante: un servicio VPN comercial **no garantiza anonimato total**. El proveedor podría registrar tráfico o incumplir lo que anuncia.

---

## Conexión a HTB VPN
En Hack The Box (HTB), el acceso a las máquinas vulnerables requiere conectarse a la red privada del laboratorio mediante VPN.  
**Buenas prácticas:**
- Conectarse siempre desde una VM (Máquina Virtual).  
- No usar la misma máquina que empleamos para clientes reales.  
- Deshabilitar autenticación por contraseña en SSH.  
- No dejar servicios web abiertos ni información sensible en la VM.

---

## Ejemplo de conexión
```bash
sudo openvpn user.ovpn
