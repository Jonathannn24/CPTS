# Post-Compromiso — Checklist y entregables

> Actividades finales para cerrar un compromiso de penetration testing.

## 1. Limpieza (Cleanup)
- Eliminar herramientas, scripts y cuentas creadas.  
- Revertir cambios menores de configuración realizados.  
- Documentar todo lo que no se pudo eliminar y notificar al cliente.

## 2. Documentación y entrega de informes
- Incluir: cadena de ataque completa, resumen ejecutivo para no técnicos, hallazgos con criticidad, PoC (Proof of Concept — Prueba de Concepto), evidencia (screenshots, logs, hashes, timestamps).  
- Añadir apéndices: alcance, hosts, credenciales usadas (si procede), cambios realizados, datos OSINT relevantes, output de escaneos.  
- Retirar/evitar conservar PII (Personally Identifiable Information — Información de Identificación Personal) innecesaria.

## 3. Reunión de revisión del informe
- Presentación a responsables (CTO, CISO, TI/Soporte, PoC (Point of Contact)).  
- Q&A técnico + aclaraciones; acordar comentarios para versión FINAL.

## 4. Aceptación del entregable
- Entregar **DRAFT** → cliente comenta → emitir **FINAL**.  
- Verificar cláusulas de aceptación definidas en SoW (Scope of Work — Alcance del Trabajo).

## 5. Pruebas posteriores a la remediación (Retest)
- Revisar evidencias de remediación del cliente.  
- Reprobar hallazgos críticos/altos y generar informe post-remediación con comparación before/after.

### Ejemplo tabla de estado
| # | Severidad | Hallazgo | Estado |
|---:|---|---|---|
| 1 | Alto | SQLi | Remediado |
| 2 | Alto | Auth rota | Remediado |
| 3 | Medio | Directorios listados | No remediado |

## 6. Rol del pentester
- Actuar como auditor/asesor — **no** implementar remediaciones (evitar conflicto de interés).  
- Proveer guidance general y clarificar PoC para los equipos de remediación.

## 7. Retención de datos y seguridad
- Guardar evidencia el tiempo pactado en contrato (SoW / RoE).  
- Cifrar datos en reposo y restringir accesos.  
- Borrar/ destruir artefactos locales y VMs usadas al cierre; crear VM nueva si hay retest futuro.

## 8. Cierre operativo y administrativo
- Confirmar borrado de accesos y artefactos.  
- Facturación y cobro según condiciones pactadas.  
- Envío de encuesta de satisfacción y retroalimentación.

## 9. Buenas prácticas finales
- Incluir instrucciones claras para verificación por parte del equipo de TI.  
- Guardar evidencia por el periodo legal/contractual (p. ej. PCI (Payment Card Industry — Industria de Tarjetas de Pago) puede recomendar prácticas).  
- Mantener comunicación profesional: el trato del equipo con el cliente pesa tanto como los resultados técnicos.

---
