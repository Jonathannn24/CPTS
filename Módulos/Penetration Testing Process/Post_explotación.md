# Post-Explotación (Post-Exploitation)

> Etapa donde, tras comprometer un sistema, se demuestra impacto real y se asegura la continuidad del acceso.

---

## 1. Objetivo
- Obtener información sensible y relevante.  
- Escalar privilegios y consolidar acceso.  
- Evaluar persistencia y exfiltración controlada.  
- Aportar valor al cliente mostrando impactos y puntos ciegos de seguridad.

---

## 2. Componentes principales
1. **Pruebas evasivas (Evasive Testing)**  
   - Tipos:  
     - Evasive → pasar desapercibido.  
     - Hybrid-Evasive → probar detección parcial.  
     - Non-Evasive → ataques visibles para medir defensas.  
   - Valor: ayuda al cliente a validar sus sistemas de detección.  

2. **Recopilación de información local**  
   - Repetir **IG (Information Gathering)** y **VA (Vulnerability Assessment)** desde el sistema comprometido.  
   - Enumerar servicios internos: impresoras, DB, virtualización, shares.  

3. **Saqueo (Pillaging)**  
   - Datos a buscar: contraseñas, configuraciones, archivos, claves SSH.  
   - Mapear el rol del host en la red y dependencias.  

4. **Persistencia**  
   - Mantener acceso en caso de perder sesión.  
   - Establecer antes de nuevos intentos de explotación.  

5. **Escalada de privilegios (Privilege Escalation)**  
   - Objetivo: root (Linux) o admin/SYSTEM (Windows).  
   - Puede ser local (exploits) o indirecta (credenciales de usuarios privilegiados).  

6. **Exfiltración de datos**  
   - Validar controles: **DLP (Data Loss Prevention)**, **EDR (Endpoint Detection and Response)**, cifrado, monitoreo de red.  
   - Usar datos falsos (ej. números de tarjeta ficticios) para pruebas.  
   - Normativas aplicables: **PCI**, **HIPAA**, **GLBA**, **FISMA**, **GDPR**, **ISO**, **NIST**, entre otras.  

---

## 3. Buenas prácticas
- **Documentar** cada paso (capturas, logs, screen recording).  
- **Comunicar hallazgos críticos** de inmediato.  
- **Adaptabilidad**: cada sistema es único, ajustar métodos.  
- **Ética**: exfiltrar datos reales solo si el cliente lo aprueba.  

---

## 4. Transición
- Una vez consolidado el acceso y validados los hallazgos:  
  - Continuar con **Movimiento Lateral (Lateral Movement)**.  
  - Preparar documentación y pruebas de concepto.  

---
