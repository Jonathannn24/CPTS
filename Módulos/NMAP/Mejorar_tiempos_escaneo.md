# Rendimiento de Nmap — optimización y comandos útiles

**Resumen rápido**

El rendimiento del escaneo es clave cuando escaneas redes grandes o tienes ancho de banda limitado. Nmap permite controlar la agresividad y tiempos mediante opciones como `-T`, `--min-rate`, `--max-retries`, y los parámetros de RTT (`--initial-rtt-timeout`, `--max-rtt-timeout`). Estos ajustes aceleran escaneos pero pueden provocar falsos negativos si se usan de forma agresiva.

---

## 1. Conceptos principales

* **Plantillas de tiempo (`-T 0..5`)**: facilitan elegir un perfil de velocidad/ruido.

  * `-T0` paranoid — extremadamente lento y sigiloso
  * `-T1` sneaky
  * `-T2` polite
  * `-T3` normal (por defecto)
  * `-T4` aggressive
  * `-T5` insane — muy rápido y ruidoso

* **Tasa mínima de paquetes (`--min-rate <num>`)**: número mínimo de paquetes por segundo que Nmap intentará mantener.

* **Reintentos (`--max-retries <num>`)**: cuántas veces volver a intentar un puerto que no responde (por defecto 10). `0` evita reintentos.

* **RTT (tiempos de ida y vuelta)**:

  * `--initial-rtt-timeout <time>`: primer timeout usado.
  * `--max-rtt-timeout <time>`: timeout máximo durante la optimización.

* **Paralelismo**: `--min-parallelism <num>` y otras opciones internas controlan cuántas sondas paralelas se permiten.

---

## 2. Ejemplos y comparaciones

### Escaneo por defecto (red /24, 100 puertos principales)

```
sudo nmap 10.129.2.0/24 -F
# Resultado: ~39s (ejemplo)
```

### Optimizar RTT (más rápido, pero puede perder hosts)

```
sudo nmap 10.129.2.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms
# Resultado: ~12s (ejemplo). Posible pérdida de detección si tiempos demasiado cortos.
```

### Reducir reintentos (más rápido; riesgo de falsos negativos)

```
sudo nmap 10.129.2.0/24 -F --max-retries 0
```

### Forzar tasa de envío mínima (acelera mucho si tu red lo soporta)

```
sudo nmap 10.129.2.0/24 -F --min-rate 300 -oN tnet.minrate300
```

### Plantilla de tiempo agresiva vs por defecto

```
sudo nmap 10.129.2.0/24 -F -oN tnet.default      # -T3 por defecto
sudo nmap 10.129.2.0/24 -F -oN tnet.T5 -T 5       # muy rápido (-T5)
```

---

## 3. Recomendaciones prácticas

* En pruebas **black-box** (externas) comienza con `-T2/-T3` y escaneos ligeros; sube agresividad solo si sabes que no activarás IDS/IPS.
* En pruebas **white-box** o con permiso para alto tráfico, usar `--min-rate`, `-T4/-T5` y reducir `--max-retries` para acelerar.
* Ajusta `--initial-rtt-timeout` y `--max-rtt-timeout` según latencia esperada de la red (p. ej., redes lentas requieren valores mayores).
* Guarda siempre los resultados (`-oA <prefijo>`) para comparación.

---

## 4. Comandos útiles para copiar

> **Bloque listo para copiar** — pega en tu terminal (ajusta IPs/puertos según tu entorno):

```bash
# Guardar todos los puertos en todos los formatos
sudo nmap 10.129.2.28 -p- -oA target

# Escaneo rápido (100 puertos) - salida normal
sudo nmap 10.129.2.0/24 -F -oN tnet.default

# Optimizar RTT (más agresivo en tiempos)
sudo nmap 10.129.2.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms -oN tnet.rtt

# Reducir reintentos a 0 (más rápido; puede perder puertos)
sudo nmap 10.129.2.0/24 -F --max-retries 0 -oN tnet.noretries

# Aumentar la tasa mínima de paquetes (requiere buen ancho de banda)
sudo nmap 10.129.2.0/24 -F --min-rate 300 -oN tnet.minrate300

# Usar plantilla de tiempo extrema (-T5) para mayor velocidad
sudo nmap 10.129.2.0/24 -F -T 5 -oN tnet.T5

# Medir progreso cada 5s y aumentar verbosidad
sudo nmap 10.129.2.0/24 -p- -sV --stats-every=5s -v -oA full_scan

# Guardar resultados y convertir XML a HTML (xsltproc debe estar instalado)
sudo nmap 10.129.2.28 -p- -oA target
xsltproc target.xml -o target.html
```

---

## 5. Lecturas recomendadas

* Documentación oficial de rendimiento de Nmap: [https://nmap.org/book/man-performance.html](https://nmap.org/book/man-performance.html)
* Plantillas de tiempo y explicación: [https://nmap.org/book/performance-timing-templates.html](https://nmap.org/book/performance-timing-templates.html)

---

*Fin del resumen sobre rendimiento y comandos de Nmap.*
