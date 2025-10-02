# Recursos en la nube

El uso de la nube, como **AWS**, **GCP**, **Azure**, y otros, es hoy en día uno de los componentes esenciales para muchas empresas. Al fin y al cabo, todas las empresas quieren poder realizar su trabajo desde cualquier lugar, por lo que necesitan un punto central para toda la dirección. Por eso los servicios de Amazon (AWS), Google (GCP) y Microsoft (Azure) son ideales para este propósito.

Aunque los proveedores de la nube protegen su infraestructura de forma centralizada, esto no significa que las empresas estén libres de vulnerabilidades. No obstante, las configuraciones realizadas por los administradores pueden hacer vulnerables los recursos en la nube de la empresa. Esto a menudo comienza con **S3 buckets (AWS)**, **blobs (Azure)**, **Cloud Storage (GCP)**, al que se puede acceder sin autenticación si se configura incorrectamente.

---

## Servidores alojados por la empresa (ejemplo)

```bash
for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4;done
```

**Salida de ejemplo:**

```
blog.inlanefreight.com 10.129.24.93
inlanefreight.com 10.129.27.33
matomo.inlanefreight.com 10.129.127.22
www.inlanefreight.com 10.129.127.33
s3-website-us-west-2.amazonaws.com 10.129.95.250
```

A menudo, el almacenamiento en la nube se agrega a la lista DNS cuando otros empleados lo utilizan con fines administrativos. Este paso hace que sea mucho más fácil para los empleados llegar a ellos y gestionarlos. Sigamos con el caso de que una empresa nos haya contratado, y durante la búsqueda de IP ya hemos visto que una dirección IP pertenece al servidor `s3-website-us-west-2.amazonaws.com`.

---

## Búsqueda pasiva: Google Dorks y ejemplos

Existen muchas formas de encontrar dicho almacenamiento en la nube. Una de las más fáciles y utilizadas es la **búsqueda de Google combinada con Google Dorks**. Por ejemplo, podemos utilizar `inurl:` e `intext:` para limitar la búsqueda.

* **Buscar S3 público vinculado a la empresa:**

```
inurl:amazonaws.com "NOMBRE_EMPRESA"
```

* **Buscar Azure Blob público:**

```
inurl:blob.core.windows.net "NOMBRE_EMPRESA"
```

* **Ejemplo (buscar PDFs en S3):**

```
inurl:amazonaws.com filetype:pdf "NOMBRE_EMPRESA"
```

En los ejemplos del texto, los resultados de búsqueda de Google muestran enlaces a archivos PDF y otros documentos alojados en S3 o Azure Blob.

---

## Código fuente del sitio y referencias a la nube

A menudo los recursos en la nube aparecen referenciados en el **código fuente** del sitio (precarga, preconnect, scripts, imágenes, etc.). Por ejemplo, fragmentos de HTML pueden contener `preconnect` o `preload` apuntando a `blob.core.windows.net` o `amazonaws.com`.

Esto facilita a los administradores gestionar recursos y, a su vez, facilita la detección de endpoints en la nube durante la enumeración pasiva.

---

## Proveedores y herramientas externas útiles

* **Domain.Glass / DomainGlass**: muestra estado de seguridad (ej. Cloudflare), IPs, detalles del certificado SSL, redes sociales y herramientas externas asociadas al dominio.
* **GrayHatWarfare**: búsquedas indexadas de S3/Azure/GCP, permite filtrar por formato de archivo y ver listados públicos.
* **Shodan**: consultar IPs para puertos abiertos/servicios expuestos.

**Resultados típicos:** paneles que muestran depósitos S3 con recuentos de archivos (ej.: 1, 73, 0) y listados de archivos encontrados.

---

## Riesgo: claves y secretos filtrados

Muchos incidentes vienen por errores humanos: empleados que suben por error claves privadas (`id_rsa`, `.pem`), backups o archivos de configuración (`.env`, `config.json`) a buckets públicos. Ejemplo típico en GrayHatWarfare muestra entradas `id_rsa` e `id_rsa.pub` en un bucket público (fecha: agosto de 2021).

**Impacto:** descargar una clave privada filtrada puede permitir acceso SSH a máquinas de la empresa sin necesidad de contraseña.

---

## Recomendaciones rápidas

1. Revisar y poner *privado* todo el almacenamiento por defecto (S3, Blob, Cloud Storage).
2. Auditar y rotar claves/credenciales expuestas.
3. Implementar políticas de acceso mínimo (IAM) y controles de logging/alertas para buckets públicos.
4. Remover archivos sensibles del almacenamiento público y educar a empleados sobre la gestión segura de recursos en la nube.

---

## Notas finales

* La enumeración pasiva es una primera y valiosa técnica para identificar recursos en la nube sin interactuar directamente con el objetivo.
* Siempre documenta fuentes, fechas y comandos usados para reproducibilidad y para el informe posterior.
* Solo realizar pruebas activas (descargas, pruebas de acceso) si cuentas con autorización explícita.
