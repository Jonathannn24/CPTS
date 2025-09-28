# Prueba de Concepto (PoC (Proof of Concept))

> La PoC (Proof of Concept) demuestra que una vulnerabilidad es explotable y qué impacto tiene, facilitando la remediación.

---

## 1. Objetivo
- Probar de forma reproducible que la vulnerabilidad existe.  
- Mostrar impacto real y pasos para reproducirlo.  
- Facilitar la priorización y remediación por parte de desarrolladores/administradores.

---

## 2. Formas comunes de PoC
- **Documental**: descripción paso a paso con capturas y logs.  
- **Script / Código**: automatiza la explotación (más claro, pero con cuidado).  
- **Prueba simulada**: datos falsos para validar controles (ej. exfiltración simulada).  

---

## 3. Contenido mínimo recomendado en la PoC
- Identificador del hallazgo (por ejemplo, CVE (Common Vulnerabilities and Exposures) si aplica).  
- Requisitos previos (credenciales, puerto, versión).  
- Pasos reproducibles y comandos exactos.  
- Evidencia (screenshots, hashes, logs, timestamps).  
- Riesgos al ejecutar el PoC (posibles DoS (Denial of Service) u otros impactos).  
- Recomendaciones de mitigación y verificación post-fix.

---

## 4. Buenas prácticas y precauciones
- **No** entregar exploits destructivos sin autorización.  
- Preferir **PoC seguras** (entornos locales o datos falsos) cuando sea posible.  
- Aclarar que parchear un vector no siempre elimina otras rutas (ej.: cambiar una contraseña no corrige la **password policy**).  
- Incluir variantes/alternativas cuando existan múltiples formas de explotación para evitar falsas sensaciones de seguridad.

---

## 5. Entregables típicos
- Archivo `.md` con pasos y evidencia.  
- Script comentado (si procede) con instrucciones de uso y advertencias.  
- Video/screencast corto (opcional) demostrando la explotación.  
- Checklist de verificación para el equipo de remediación.

---

## 6. Ejemplo breve (estructura de PoC)
1. Título + identificador.  
2. Resumen del impacto.  
3. Requisitos previos.  
4. Comandos/Script.  
5. Evidencia.  
6. Remediación sugerida.

---
