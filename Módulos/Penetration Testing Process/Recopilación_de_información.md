# Recopilación de Información (Information Gathering)

> La recopilación de información es la **base del pentest**. Es iterativa y acompaña todas las etapas: sin buenos datos, no hay hallazgos fiables ni explotación segura.

## Contenido
- [1. Objetivo y alcance](#1-objetivo-y-alcance)
- [2. Estructura general del proceso](#2-estructura-general-del-proceso)
- [3. OSINT (Open Source Intelligence)](#3-osint-open-source-intelligence)
- [4. Enumeración de infraestructura](#4-enumeración-de-infraestructura)
- [5. Enumeración de servicios](#5-enumeración-de-servicios)
- [6. Enumeración del host](#6-enumeración-del-host)
- [7. Pillaging (saqueo) y su relación con IG/priv-esc](#7-pillaging-saqueo-y-su-relación-con-igpriv-esc)
- [8. Gestión de hallazgos críticos y RoE](#8-gestión-de-hallazgos-críticos-y-roe)
- [9. Checklist operativo](#9-checklist-operativo)
- [10. Matriz de artefactos y uso](#10-matriz-de-artefactos-y-uso)
- [11. Buenas prácticas](#11-buenas-prácticas)

---

## 1. Objetivo y alcance
- Reunir información **externa e interna** de la organización, sus personas, activos y dependencias.
- Apoyar la **identificación de superficie de ataque**, priorización de vectores y validación de hipótesis.
- Proveer insumos verificables para **Vulnerability Assessment**, **Exploitation**, **Post-Exploitation** y **Lateral Movement**.

---

## 2. Estructura general del proceso
1. **OSINT**: fuentes públicas (personas, dominios, repos, publicaciones, ofertas de empleo, metadatos).
2. **Infra**: mapa de activos/red (DNS, nube, correo, web, ASNs, rangos, controles perimetrales).
3. **Servicios**: identificación y perfilado (puertos, banners, versiones, exposición).
4. **Host**: SO, roles, relaciones, puertos internos, configuraciones locales.
5. **(Iterativo)**: volver a IG tras cada hallazgo/avance (explotación, post-explotación, credenciales, rutas nuevas).

---

## 3. OSINT (Open Source Intelligence)
- **Objetivo**: identificar datos sensibles/publicados y relaciones útiles (eventos, proveedores, tecnologías).
- **Ejemplos de hallazgos**:
  - Credenciales/secretos (contraseñas, claves SSH/API, tokens).
  - Repositorios mal configurados, gists, issues con logs o configuraciones.
  - Metadatos de documentos (usuarios, rutas internas, versiones).
  - Tecnologías declaradas en **ofertas de empleo** o páginas corporativas.
- **Acción inmediata** ante secretos expuestos:
  - Seguir el **RoE**: **pausar** si aplica, **notificar** a contactos de emergencia y documentar evidencia.
- **Consejo**: priorizar fuentes con mayor probabilidad de impacto (repos de ingeniería, tickets, wikis públicas).

---

## 4. Enumeración de infraestructura
- **Objetivo**: obtener una visión de la **postura externa/interna** y controles de seguridad.
- **Qué mapear**:
  - DNS (registros, subdominios), servidores de correo, web, **instancias cloud**, WAF/CDN, ASNs y rangos.
  - Diferenciar **en alcance** vs **fuera de alcance** según el **Scoping Document**.
  - **Controles**: WAF, FW, IDS/IPS, NAC y comportamiento frente a tráfico evasivo.
- **Valor**: define tácticas (por ejemplo, si hay WAF → ajustar payloads/velocidad; si hay NAC → plan interno).

---

## 5. Enumeración de servicios
- **Objetivo**: identificar **servicios expuestos**, **versiones** y **superficie**.
- **Puntos clave**:
  - Extraer **banners** y versiones → comparar con historiales y CVEs conocidos.
  - Registrar **métodos y funcionalidades** (p. ej., FTP anónimo, endpoints admin, RPCs habilitados).
  - Valorar **temor al cambio** en sistemas legacy (riesgo de versiones antiguas sin parchear).
- **Resultado**: lista priorizada de vectores plausibles y pruebas seguras de verificación.

---

## 6. Enumeración del host
- **Objetivo**: perfilar el **rol** y el **contexto** de cada host.
- **Qué identificar**:
  - **SO y build**, servicios locales, versiones, puertos internos, rutas de comunicación.
  - Roles (web/app/DB/DC/FS/Jump), relaciones y dependencias.
  - Configuraciones débiles: shares abiertos, cuentas por defecto, servicios sin cifrar.
- **Interno vs externo**:
  - Interno revela servicios **no expuestos** públicamente, donde abundan **misconfigurations**.
  - Tras explotación, IG local para **priv-esc** y **lateral movement**.

---

## 7. Pillaging (saqueo) y su relación con IG/priv-esc
- **No es una fase aislada**: es parte integral de **IG** y **privilege escalation** una vez hay acceso.
- **Objetivo**: recolectar artefactos que demuestren impacto y **habiliten nuevos movimientos**:
  - Credenciales (archivos de config, gestores de secretos, browsers, servicios).
  - Datos sensibles (clientes, PII, llaves, tokens, inventarios).
  - Evidencia de rutas laterales (scripts, tareas programadas, conexiones a DB).
- **Nota**: Tratar cadena de custodia y cifrado de evidencias conforme a **RoE**.

---

## 8. Gestión de hallazgos críticos y RoE
- Seguir lo pactado: **pausar**, **notificar** (contactos de emergencia), **emitir informe de notificación**.
- Ejemplos típicos: **RCE no autenticada**, **SQLi crítica**, **clave/credencial expuesta activa**, **fuga de datos**.

---

## 9. Checklist operativo

**9.1 OSINT**
- [ ] Búsqueda de secretos (repos, gists, pastes, issues, wikis públicas).
- [ ] Metadatos en docs públicos.
- [ ] Tecnologías declaradas (webs/empleo/blog/foros).
- [ ] Huella de dominios/subdominios y proveedores (cloud/CDN).
- [ ] Personas y patrones de correo (para ingeniería social si está en alcance).

**9.2 Infra**
- [ ] Inventario de dominios, rangos, ASNs, DNS/mail/web.
- [ ] Presencia en cloud (nombres/etiquetas públicas).
- [ ] Controles: WAF, CDN, FW, IDS/IPS, NAC.
- [ ] Diferenciar en alcance vs fuera de alcance.

**9.3 Servicios**
- [ ] Descubrir puertos/servicios y banners.
- [ ] Identificar versiones/funcionalidad sensible (FTP anónimo, paneles admin).
- [ ] Priorizar servicios legacy/desactualizados.

**9.4 Host**
- [ ] SO/rol/dependencias.
- [ ] Servicios internos/no expuestos.
- [ ] Misconfigs locales (shares, permisos, credenciales guardadas).
- [ ] Trazas para priv-esc/movimiento lateral.

**9.5 RoE / Riesgos**
- [ ] Criterios de **pausa/notificación** definidos.
- [ ] Manejo de evidencias (cifrado, hash, custodia).
- [ ] Respetar alcance y ventanas horarias.

---

## 10. Matriz de artefactos y uso

| Artefacto | Fuente típica | Valor para… |
|---|---|---|
| Claves/API tokens | Repos/CI/CD/logs | Acceso directo, PoC, pivoting |
| Usuarios/roles | Metadatos/AD/listados | Spraying, validación de auth |
| Inventarios/diagramas | Wikis/Docs públicos | Rutas, dependencias, priorización |
| Banners/versiones | Escaneos/headers | Mapeo de CVEs, fingerprinting |
| Configs (DB/app) | Host comprometido | Credenciales, rutas laterales |
| Logs/errores detallados | Apps/servicios | Info sensible, paths internos |

---

## 11. Buenas prácticas
- **Iterar**: volver a IG tras cada hallazgo.
- **Trazabilidad**: registrar comandos, timestamps y hashes de evidencia.
- **Mínimo impacto**: empezar por pruebas no invasivas; escalar según RoE.
- **Contexto**: separar claramente **en alcance** / **fuera de alcance**.
- **Comunicación**: informar hallazgos críticos según **canales y contactos** del RoE.

---
