# Pre-Compromiso (Penetration Testing) — Guía y Checklists

> El pre-compromiso es la fase de preparación que convierte una idea de prueba en un **proyecto ejecutable, legal y seguro**. Estructura: **NDA → Cuestionario de alcance → Reunión previa → Propuesta/SoW + RoE → Kick-off**.

## Contenido
- [1. Objetivo del pre-compromiso](#1-objetivo-del-pre-compromiso)
- [2. Flujo del proceso](#2-flujo-del-proceso)
- [3. NDA (tipos y notas legales)](#3-nda-tipos-y-notas-legales)
- [4. Autoridad para contratar (quién puede firmar)](#4-autoridad-para-contratar-quién-puede-firmar)
- [5. Documentación requerida y cuándo crearla](#5-documentación-requerida-y-cuándo-crear-la)
- [6. Cuestionario de alcance (servicios y preguntas clave)](#6-cuestionario-de-alcance-servicios-y-preguntas-clave)
- [7. Reunión previa al compromiso (Contract Checklist)](#7-reunión-previa-al-compromiso-contract-checklist)
- [8. Reglas de Compromiso (RoE) — Checklist](#8-reglas-de-compromiso-roe--checklist)
- [9. Reunión de inicio (Kick-off)](#9-reunión-de-inicio-kick-off)
- [10. Contractor’s Agreement (pruebas físicas)](#10-contractors-agreement-pruebas-físicas)
- [11. Notas legales y de riesgo](#11-notas-legales-y-de-riesgo)

---

## 1. Objetivo del pre-compromiso
- Alinear **expectativas, alcance, objetivos y limitaciones**.
- Cubrir **legalmente** (NDA, permisos de terceros, autorización del firmante).
- Definir **métodos, ventanas de prueba, comunicación y escalado**.
- Producir **SoW/Contrato**, **RoE** y cronograma de trabajo.

---

## 2. Flujo del proceso
1. **NDA firmado** (antes de compartir detalles sensibles).
2. **Cuestionario de alcance** (servicios, superficies, tamaños).
3. **Reunión previa** → **Propuesta/Contrato (SoW)** y **RoE**.
4. **Kick-off** (confirmación final, logística, ventanas y contactos).

---

## 3. NDA (tipos y notas legales)

| Tipo | Descripción |
|---|---|
| **Unilateral NDA** | Obliga a una parte a mantener confidencialidad; la otra puede compartir con terceros según términos. |
| **Bilateral NDA** | Ambas partes mantienen confidencial la información (el más común para pentesting). |
| **Multilateral NDA** | Más de dos partes (ej. redes/entornos cooperativos). |

> **Requisito**: el NDA debe estar **firmado por todas las partes** antes de discutir detalles técnicos.

---

## 4. Autoridad para contratar (quién puede firmar)
Ejemplos de roles válidos (varía por organización):
- **CEO, CTO, CISO, CSO, CRO, CIO**
- **VP Auditoría Interna, Gerente de Auditoría**
- **VP/Director de TI o Seguridad de la Información**

> **Crítico**: confirmar **signatario con autoridad** para el contrato, RoE y permisos. Evita riesgos legales por autorizaciones inválidas.

---

## 5. Documentación requerida y cuándo crearla

| Documento | Momento |
|---|---|
| **Non-Disclosure Agreement (NDA)** | **Después** del contacto inicial (y **antes** de compartir detalles) |
| **Scoping Questionnaire** | **Antes** de la reunión previa |
| **Scoping Document** | **Durante** la reunión previa |
| **Penetration Testing Proposal (Contrato / SoW)** | **Durante** la reunión previa |
| **Rules of Engagement (RoE)** | **Antes** del kick-off |
| **Contractor’s Agreement** (si hay pruebas físicas) | **Antes** del kick-off |
| **Reports** | **Durante** y **después** de la prueba |

> El cliente puede aportar un documento de alcance (IPs/URLs/credenciales). Inclúyelo como **apéndice del RoE**.

---

## 6. Cuestionario de alcance (servicios y preguntas clave)

### 6.1 Selección de servicios (marcar lo aplicable)
- [ ] Evaluación de vulnerabilidad **interna**
- [ ] Evaluación de vulnerabilidad **externa**
- [ ] Prueba de penetración **interna**
- [ ] Prueba de penetración **externa**
- [ ] Seguridad **inalámbrica**
- [ ] Seguridad de **aplicaciones** / **aplicaciones web**
- [ ] **Ingeniería social** (phishing/vishing)
- [ ] **Red Team**
- [ ] **Física** (controles perimetrales, acceso, etc.)
- [ ] **Revisión de código** / **móviles**

### 6.2 Detalle por servicio (ejemplos)
- Web vs móvil, autenticado vs no autenticado, roles (usuario/admin), caja **negra/gris/blanca**.
- Nivel de **evasión**: no evasivo / híbrido-evasivo (progresivo) / totalmente evasivo.

### 6.3 Dimensión del alcance (datos críticos)
- # hosts en vivo, rangos IP/CIDR, dominios/subdominios.
- # SSID, # apps web/móviles y roles.
- Phishing: # usuarios objetivo, ¿lista del cliente u **OSINT**?
- Evaluación física: # ubicaciones y dispersión geográfica.
- Objetivos de **Red Team** y exclusiones (p. ej., sin phishing físico).
- ¿Evaluación separada de **Active Directory**?
- Pruebas de red: ¿usuario anónimo o de dominio estándar?
- ¿Necesidad de evadir/controlar **NAC**?

> Estos datos calibran **recursos, cronograma y coste** (p. ej., VA externa de 10 IP vs pentest interno con múltiples /24).

---

## 7. Reunión previa al compromiso (Contract Checklist)

| Punto de control | Descripción |
|---|---|
| [ ] **NDA** | Firmado antes o durante la reunión (previo a discutir detalles). |
| [ ] **Goals** | Hitos de alto nivel → objetivos detallados. |
| [ ] **Scope** | Dominios, IPs/rangos, hosts, cuentas, sistemas críticos; base legal para probar. |
| [ ] **Penetration Testing Type** | Externa/interna/VA/ingeniería social; recomendación justificada. |
| [ ] **Methodologies** | OSSTMM, PTES, OWASP; análisis manual/automático; verificación y explotación. |
| [ ] **Locations** | Remoto (VPN), interno, híbrido. |
| [ ] **Time Estimation** | Fechas de inicio/fin y **ventanas por fase** (Explotación, Post-Explotación, Movimiento Lateral). |
| [ ] **Third Parties** | Proveedores (cloud/ISP/hosting); **permisos por escrito** y verificación. |
| [ ] **Evasive Testing** | Definir si se permite y en qué grado. |
| [ ] **Risks** | Riesgos y salvaguardas; criterios de pausa/rollback. |
| [ ] **Scope Limitations** | Sistemas críticos a **no afectar** (producción/servicios esenciales). |
| [ ] **Information Handling** | Cumplimientos (HIPAA, PCI, NIST/HITRUST, etc.). |
| [ ] **Contact Information** | Lista de contactos, orden de escalado, correos y teléfonos. |
| [ ] **Lines of Communication** | Email, llamadas, reuniones, canales seguros. |
| [ ] **Reporting** | Estructura, lectores objetivo, presentación de resultados. |
| [ ] **Payment Terms** | Precios y condiciones. |

> **Claves**: priorizar **preferencias del cliente** (no “cocinar el bistec a gusto del chef”), evitar **DoS** salvo acuerdo expreso, y documentar criterios para **pausar** por hallazgos críticos.

---

## 8. Reglas de Compromiso (RoE) — Checklist

| Punto de control | Contenido |
|---|---|
| [ ] **Introduction** | Propósito y alcance del documento. |
| [ ] **Contractor** | Empresa y datos del contratista. |
| [ ] **Penetration Testers** | Nombres y roles del equipo. |
| [ ] **Contact Information** | Contactos de ambas partes (emails/teléfonos). |
| [ ] **Purpose** | Propósito de la prueba. |
| [ ] **Goals** | Objetivos/flags a alcanzar. |
| [ ] **Scope** | IPs, dominios, URLs, rangos CIDR. |
| [ ] **Lines of Communication** | Canales y frecuencia. |
| [ ] **Time Estimation** | Fechas de inicio/fin. |
| [ ] **Time of the Day to Test** | Ventanas horarias. |
| [ ] **Testing Type** | Externa/interna/VA/ingeniería social. |
| [ ] **Locations** | Cómo se conectará (VPN, on-site). |
| [ ] **Methodologies** | OSSTMM, PTES, OWASP, etc. |
| [ ] **Objectives / Flags** | Usuarios/archivos/infos objetivo. |
| [ ] **Evidence Handling** | Cifrado y protocolos seguros. |
| [ ] **System Backups** | Respaldos de config/DB, etc. |
| [ ] **Information Handling** | Requisitos de cifrado fuerte. |
| [ ] **Incident Handling & Reporting** | Contactos de emergencia, pausas, tipos de informe. |
| [ ] **Status Meetings** | Cadencia de reuniones/fechas. |
| [ ] **Reporting** | Tipo, audiencia, enfoque. |
| [ ] **Retesting** | Fechas y condiciones. |
| [ ] **Disclaimers / Liability** | Limitaciones y exenciones. |
| [ ] **Permission to Test** | Contrato y permisos firmados. |

---

## 9. Reunión de inicio (Kick-off)
- Ocurre **tras firmar** contrato/RoE/NDA.
- Participan PoC(s) del cliente (Auditoría, InfoSec, TI, Riesgo), soporte técnico (devs/sysadmins/net), **equipo de pentest** (líder, testers, PM/ventas si procede).
- Reglas operativas:
  - Normalmente **sin DoS**.
  - **Pausa** si hay **vulnerabilidad crítica** (p. ej., RCE no autenticado, SQLi crítica, fuga de datos sensible). Se emite **informe de notificación** y se contacta con emergencias.
  - En interno, pausar si hay caída de sistemas, evidencia de actividad ilegal o actor de amenaza activo.
- **Riesgos operativos** a comunicar:
  - Generación de **logs y alertas**.
  - Posible **bloqueo de cuentas** por fuerza bruta u otras técnicas.
  - Cliente debe **avisar de inmediato** si detecta impacto negativo.

---

## 10. Contractor’s Agreement (pruebas físicas)
Documento adicional y obligatorio para **intrusión física** (leyes distintas y personal no informado). Sirve como **permiso explícito** frente a seguridad/policía.

**Checklist (físico)**
- [ ] Introduction
- [ ] Contractor
- [ ] Purpose
- [ ] Goal
- [ ] Penetration Testers
- [ ] Contact Information
- [ ] Physical Addresses / Building / Floors / Rooms
- [ ] Physical Components
- [ ] Timeline
- [ ] Notarization
- [ ] Permission to Test

---

## 11. Notas legales y de riesgo
- **Asesoría legal**: todos los documentos deben ser **revisados por un abogado**.
- **Permisos de terceros** (cloud/ISP/hosting) **por escrito**, y **verificados**.
- **Autorización del signatario** confirmada antes de iniciar.
- Mantener **evidencias** y **cadenas de custodia** conforme a RoE.

---
