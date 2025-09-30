# Enumeración — Resumen y checklist

## Resumen breve

La **enumeración** es la fase más crítica en una evaluación de seguridad: no se trata sólo de entrar, sino de **identificar todas las vías posibles** para atacar un objetivo. Herramientas como Nmap, Gobuster, WhatWeb o LinEnum son útiles, pero solo sirven si sabemos **qué hacer** con la información que extraen. La enumeración eficaz mezcla **herramientas automáticas** y **enumeración manual** enfocada: leer respuestas, interactuar con servicios y comprender protocolos para descubrir configuraciones erróneas, datos expuestos y recursos explotables.

> La diferencia entre una búsqueda superficial y una exhaustiva puede ser como pedir "dónde están las llaves" vs. "en la sala, en el estante blanco, al lado del televisor, en el tercer cajón". Cuanta más información exacta recojas, más fácil será explotar la caja.

---

## Qué buscamos (objetivos de la enumeración)

* Funcionalidades/recursos que permitan **interacción** (formularios, APIs, uploads, servicios expuestos).
* Información que nos da **más información** (banners, headers, versiones, archivos de configuración, credenciales, certificados, robots.txt).
* Configuraciones erróneas (listados de directorios, permisos débiles, servicios con versiones vulnerables, ficheros escribibles, sudo NOPASSWD, tareas programadas, claves privadas expuestas).

---

## Principios clave

1. **Herramientas ≠ conocimiento.** Usa herramientas para recopilar datos, pero invierte tiempo en entender cómo se comunican los servicios y qué te cuenta cada respuesta.
2. **Enumeración manual es indispensable.** Muchas defensas (timeouts, rate limits, filtros) hacen que las herramientas fallen: interactúa manualmente si hace falta.
3. **Iterativa y adaptativa.** La enumeración es un proceso iterativo: cada hallazgo sugiere nuevas pruebas.
4. **Registrar todo.** Guarda salidas, timestamps y rutas para reproducibilidad y reporting.

---

## Checklist práctico (orden recomendado)

1. **Conectividad básica**: comprobar VPN/rutas/ping.
2. **Escaneo de puertos**: Nmap básico → nmap -sV --open -oA target_initial <ip>.
3. **Banner grabbing**: nc, curl, WhatWeb, whatweb <url>.
4. **Enumeración web**: gobuster/ffuf, revisar robots.txt, revisar código fuente, buscar subdominios.
5. **Enumeración de servicios**: smbclient, snmpwalk, ftp, ssh, smtp (según puertos).
6. **Búsqueda de ficheros expuestos**: config, backups, .git, credentials, keys.
7. **Scripting y automatización**: linpeas/linenum/WinPEAS para primeras pistas (con cautela).
8. **Análisis de resultados**: identificar vectores reales (uploads, comandos sudo, tareas cron, servicios con RCE conocidos).
9. **Prueba manual**: reproducir las interacciones sospechosas manualmente (requests lentas, encadenamiento de pasos).
10. **Documentar y priorizar**: prioriza vectores explotables y crea un plan de explotación.

---

## Trucos y consideraciones prácticas

* **Ajusta timeouts** en tus herramientas si crees que un servicio responde lento o aplica rate‑limit.
* **Prueba varias formas de interactuar**: muchas veces un endpoint responde diferente por User‑Agent, por método (GET/POST), por cabeceras o por autenticación parcial.
* **Fuzzying y bruteforce con cautela**: evita bloquearte a ti mismo (IP blacklist) y respeta el scope del engagement.
* **Busca datos auxiliares**: certificados TLS, metadatos, comentarios en HTML, archivos robots.txt, sitemaps y README públicos dan pistas valiosas.
* **Pensar como desarrollador/administrador**: ¿qué rutas/archivos existen por conveniencia (backups, uploads, logs)? ¿qué scripts se ejecutan periódicamente?

---

## Errores comunes al enumerar

* Confiar únicamente en un único escaneo y no volver a intentar manualmente.
* No interpretar correctamente los estados (closed/filtered) y descartarlos sin más.
* No leer los banners o no correlacionar versiones con exploits conocidos.
* No guardar salidas o anotar timestamps (pierdes evidencia y contexto).

---

## Recursos recomendados

* Nmap, Gobuster/ffuf, WhatWeb, curl, nc, nikto.
* SecLists (wordlists); CeWL (wordlist a partir del site).
* LinEnum / linpeas / WinPEAS para enumeración local.
* GTFOBins / LO LBAS para abuso de binarios.

---

## Conclusión

La enumeración es el núcleo de un pentest exitoso. No se trata de ejecutar más herramientas, sino de **interpretar mejor** las respuestas, **interactuar** con los servicios y **traducir** hallazgos en vectores prácticos de ataque. Si entiendes el servicio que estás probando, habrás ganado la mitad de la batalla.

*Fin.*
