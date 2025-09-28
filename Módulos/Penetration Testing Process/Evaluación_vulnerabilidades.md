# Evaluación de Vulnerabilidades

> Fase analítica que transforma la información recopilada en **vectores de ataque concretos**. Iterativa y dependiente de la calidad del *Information Gathering*.

---

## 1. Objetivo
- Analizar hosts, servicios y configuraciones detectadas.
- Identificar vulnerabilidades conocidas (CVE, exploits).
- Estimar impacto y probabilidad.
- Preparar pruebas/PoC seguros para confirmar hallazgos.

---

## 2. Tipos de análisis aplicados
| Tipo | Enfoque |
|---|---|
| **Descriptive** | Describe datos, detecta errores y valores atípicos. |
| **Diagnostic** | Explica causas y efectos de las vulnerabilidades. |
| **Predictive** | Modela escenarios futuros basados en datos actuales/pasados. |
| **Prescriptive** | Recomienda acciones para prevenir/explotar/mitigar. |

---

## 3. Fuentes y herramientas comunes
- **CVE / NVD (NIST)**
- **Exploit DB**
- **Security advisories** de fabricantes
- **Packet Storm**, Vulners, foros especializados

---

## 4. Ejemplo práctico
- Hallazgo: puerto TCP **2121** abierto.
- Hipótesis: podría ser **FTP** en puerto no estándar.
- Acción: conectarse con **netcat/FTP client** para validar.
- Ajuste: usar `nmap --min-rtt-timeout` para confirmar servicio lento.

---

## 5. Procedimiento
1. Revisar hallazgos de IG y asociarlos con CVEs/exploits conocidos.
2. Validar con PoC o pruebas locales (simulación si es evasivo).
3. Ejecutar pruebas controladas en el sistema real (según RoE).
4. Documentar resultados, evidencias y grado de riesgo.
5. Si no hay hallazgos → volver a IG para recolectar más datos.

---

## 6. Iteración con IG
- **IG ↔ VA** se retroalimentan: sin suficiente contexto, no hay evaluación útil.
- Ejemplo: enumerar de nuevo servicios ocultos o versiones no confirmadas.

---

## 7. Buenas prácticas
- Siempre correlacionar **qué sabemos vs qué no sabemos**.
- Validar suposiciones con pruebas mínimas antes de concluir.
- Simular localmente entornos críticos antes de test en vivo.
- Documentar claramente hallazgo → prueba → confirmación → riesgo.

---
