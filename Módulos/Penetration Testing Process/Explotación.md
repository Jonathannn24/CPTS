# Explotación (Exploitation)

> Fase en la que se usan vulnerabilidades confirmadas para obtener acceso inicial, elevar privilegios o ganar persistencia.  

---

## 1. Objetivo
- Adaptar vulnerabilidades a nuestro caso de uso (punto de apoyo, shell inverso, privilegios).  
- Validar exploits en entornos controlados antes de aplicarlos.  
- Documentar cada paso y comunicar riesgos al cliente.  

---

## 2. Priorización de ataques
Se evalúa cada posible ataque según:  
1. **Probabilidad de éxito** → CVSS (Common Vulnerability Scoring System) con NVD (National Vulnerability Database).  
2. **Complejidad** → esfuerzo y experiencia necesarios.  
3. **Probabilidad de daño** → evitar interrupciones (no hacer DoS salvo autorización).  

**Ejemplo de puntuación:**

| Factor | Puntos | RFI (Remote File Inclusion) | Buffer Overflow |
|---|---|---|---|
| Probabilidad de éxito | 10 | 10 | 8 |
| Complejidad fácil | 5 | 4 | 0 |
| Complejidad media | 3 | 0 | 3 |
| Complejidad dura | 1 | 0 | 0 |
| Probabilidad de daño | -5 | 0 | -5 |
| **Total** | max. 15 | **14** | 6 |

👉 Se prioriza el **RFI (Remote File Inclusion)** por ser más simple y menos riesgoso.

---

## 3. Preparación
- Si no hay PoC (Proof of Concept) estable:  
  - Crear entorno espejo en máquina virtual.  
  - Replicar versiones de servicios y aplicaciones.  
  - Ajustar exploit y validar localmente.  
- Si el exploit es conocido/estable: usar herramienta probada.  
- Si el riesgo es alto → **consultar con el cliente**.  

---

## 4. Buenas prácticas
- **Comunicación constante**: nunca ejecutar sin informar si hay riesgo de inestabilidad.  
- **Registro completo**: notas, comandos y logs para reportes.  
- **Pruebas seguras**: usar conexiones cifradas en shells inversos.  
- **Ética**: si el cliente no aprueba la explotación, marcar hallazgo como no confirmado y documentarlo.  

---

## 5. Transición
Una vez explotado el objetivo con éxito:  
- Confirmar acceso inicial.  
- Documentar todo.  
- Continuar con **Post-Exploitation** y **Lateral Movement**.  

---
