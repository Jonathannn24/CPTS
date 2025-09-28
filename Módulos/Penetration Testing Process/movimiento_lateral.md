# Movimiento Lateral (Lateral Movement)

> Objetivo: determinar hasta dónde puede llegar un atacante dentro de la red interna tras un compromiso inicial.

---

## 1. Resumen rápido
- Usar hosts comprometidos como **pivot** (proxy/tunnel) para alcanzar redes no ruteables.  
- Repetir **IG (Information Gathering)** y **VA (Vulnerability Assessment)** desde perspectiva interna.  
- Priorizar rutas según facilidad, impacto y detección.  
- Documentar y demostrar rutas de ataque para que el cliente pueda remediar.

---

## 2. Pasos principales
1. **Pivot / Tunneling**  
   - Crear túneles / proxies desde el host comprometido (SSH, SOCKS, port-forwarding, proxychains).  
   - Escanear la red interna a través del pivot.

2. **Pruebas evasivas (Evasive Testing)**  
   - Modos: **Non-Evasive** (visible), **Hybrid-Evasive**, **Evasive** (oculto).  
   - Medir respuesta de EDR (Endpoint Detection and Response), IDS/IPS (Intrusion Detection/Prevention System) y logging.

3. **Recopilación de información interna (IG)**  
   - Enumerar hosts, shares, dominios, servicios internos, controladores de dominio, roles y rutas de comunicación.

4. **Evaluación de vulnerabilidades interna (VA)**  
   - Buscar misconfiguraciones, servicios legacy y permisos compartidos que no son visibles desde Internet.

5. **(Privilegio) Explotación**  
   - Usar credenciales, pass-the-hash, NTLM relay o exploits locales para elevar privilegios.  
   - Evitar acciones destructivas sin aprobación (DoS (Denial of Service)).

6. **Post-Explotación**  
   - Pillaging (saqueo): buscar credenciales, claves SSH, backups, datos sensibles.  
   - Establecer persistencia si procede y documentar evidencia.

---

## 3. Técnicas y herramientas comunes (ejemplos)
- **Pivot**: SSH tunneling, socat, proxychains, Meterpreter (con túnel).  
- **Enumeración interna**: nmap, smbclient, rpcclient, bloodhound.  
- **Movimiento**: pass-the-hash, pass-the-ticket, Responder (SMB/NetNTLM capture).  
- **Evasión**: ajustar timing, ofuscación de payloads, living-off-the-land (usar binarios legítimos).

---

## 4. Riesgos y mitigación
- Riesgo: propagación de ransomware, exfiltración de datos, impacto operativo.  
- Mitigaciones a validar por cliente: segmentación de red (microsegmentation), control de privilegios, monitoreo de lateralidad (detección de autenticaciones inusuales), EDR y políticas de contraseñas.

---

## 5. Entregables/Pruebas
- Diagrama de pivots y rutas explotadas.  
- Evidencia de acceso y elevación (screenshots, hashes, logs).  
- Recomendaciones prácticas: segmentación, rotación de credenciales, listas blancas/negación de servicios internos.

---
