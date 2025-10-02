# Metodología de enumeración

Los procesos complejos deben apoyarse en una metodología estandarizada para mantener el foco y evitar omisiones. La enumeración en pruebas de penetración es dinámica: la metodología que proponemos es flexible, práctica y está organizada en 6 capas anidadas que guían el trabajo desde la presencia externa hasta la configuración interna del sistema.

> **Objetivo general:** identificar de forma sistemática los huecos (vulnerabilidades, servicios, configuraciones) que permitan avanzar a la siguiente "capa" hasta lograr acceso y, si procede, escalada de privilegios.

---

## Niveles de enumeración

La metodología se articula en tres niveles principales:

* **Infrastructure-based enumeration** (enumeración basada en infraestructura)
* **Host-based enumeration** (enumeración basada en hosts)
* **OS-based enumeration** (enumeración basada en sistemas operativos)

Estos niveles se cruzan con las 6 capas descritas a continuación.

---

## Las 6 capas de la metodología

| Capa                         | Descripción                                                                 | Información clave a buscar                                                                            |
| ---------------------------- | --------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **1. Presencia en Internet** | Identificar la huella pública y la infraestructura accesible desde Internet | Dominios, subdominios, vHosts, ASN, netblocks, IPs, instancias cloud, medidas de seguridad (WAF, CDN) |
| **2. Puerta de enlace**      | Entender la protección perimetral y cómo el tráfico llega a los servicios   | Firewalls, DMZ, IDS/IPS, EDR, proxies, NAC, segmentación, VPN, Cloudflare                             |
| **3. Servicios accesibles**  | Enumerar interfaces y servicios expuestos (externa o internamente)          | Tipo de servicio, puerto, versión, interfaz, funcionalidad, configuraciones                           |
| **4. Procesos**              | Identificar procesos y flujos de datos internos asociados a servicios       | PID, tareas, datos procesados, orígenes/destinos, dependencias                                        |
| **5. Privilegios**           | Comprender la separación de privilegios y permisos disponibles              | Usuarios, grupos, roles, permisos, restricciones y su alcance                                         |
| **6. Configuración del SO**  | Inspección de la configuración y componentes internos una vez con acceso    | Tipo de SO, nivel de parche, configuración de red, archivos de config, datos sensibles                |

> **Nota:** para intranets (p. ej. Active Directory) las capas 1 y 2 aplican distinto; las infraestructuras internas se tratan con módulos dedicados.

---

## Enfoque práctico y recomendaciones

* **Metodología ≠ check‑list rígida.** Es un marco sistemático que facilita la toma de decisiones; las herramientas concretas (nmap, gobuster, whatweb, etc.) son hojas de trucos que se usan dentro de la metodología.

* **Iteración y adaptabilidad.** La enumeración es iterativa: los hallazgos en una capa orientan las acciones en las siguientes.

* **Priorizar vectores con mayor probabilidad de explotación.** No todas las brechas conducen a acceso real; valora coste/beneficio y el tiempo disponible.

* **Documentación continua.** Guarda resultados, timestamps y evidencia (salidas de escaneos, capturas) para reproducibilidad y reporting.

* **Pensar lateralmente.** Muchas veces la entrada no es la más obvia; piensa en dependencias entre servicios, credenciales reutilizadas, archivos expuestos, backups y recursos en la nube.

---

## Flujo sugerido de trabajo (resumido)

1. **Presencia externa:** OSINT, búsquedas de dominios/subdominios, enumeración de netblocks y servicios expuestos.
2. **Mapeo de perímetro:** identificar WAF, balanceadores, VPNs, puertos filtrados; descubrir puntos de entrada.
3. **Enumeración de servicios:** detección de versiones, banners, directorios y contenido web (nmap -sV, gobuster, whatweb).
4. **Interacción manual:** probar funcionalidades, formularios, autenticaciones, upload paths, API endpoints.
5. **Escalada a host interno:** si se obtiene acceso, enumerar SUID/sudo, crons, configuraciones, credenciales expuestas, claves SSH.
6. **Escalada de privilegios y persistencia:** usar hallazgos para escalar y documentar impacto.

---

## Consejos finales

* **No confíes solo en herramientas automáticas.** Aprender el protocolo/servicio aumenta la efectividad.
* **Elegir la herramienta adecuada para cada tarea.** Complementa herramientas automatizadas con técnicas manuales (telnet/nc, curl, tcpdump, lectura de ficheros).
* **Gestiona el ruido.** En entornos reales, potencia el equilibrio entre rapidez y sigilo (evita disparar alertas innecesarias).
* **Sé humilde y curioso.** La empresa que mejor conoces es la que has estudiado durante más tiempo. Nunca asumas que lo has cubierto todo.

---

## Recursos y lecturas recomendadas

* Documentación y cheat-sheets de Nmap, Burp, Gobuster, WhatWeb.
* Repositorios de enumeración: `GTFOBins`, `peass`, `bash-hackers` y listas de verificación (HackTricks, Awesome-PrivEsc).

---

*Documento: Metodología de enumeración — guía práctica y marco de trabajo.*
