# Plan de hitos verificables — Nolvus AI Translator

**Última consolidación documental:** 2026-11-03  
**Fuente de estado y métricas actuales:** [CURRENT_PROJECT_STATE.md](../../CURRENT_PROJECT_STATE.md)

Este roadmap reemplaza el plan inicial de nueve fases. Distingue el trabajo histórico completado del trabajo que aún requiere validación. No implica autorización para ejecutar traducciones, activar API, modificar MO2 ni tocar la instalación original.

## Estado de fases

| Fase | Alcance | Estado documental | Observación |
|---:|---|---|---|
| 0–6 | Investigación, diseño, infraestructura, extracción, clasificación, glosario/TM, QA simulado y exportación no invasiva | Reportadas como completadas en el historial del proyecto | La implementación debe verificarse contra el código actual antes de afirmar que cada componente sigue operativo. |
| 7 | Escaneo real y snapshot inicial | Completada | Sus métricas son históricas; fueron ampliadas y reconciliadas en Fase 9. |
| 8 | Estimación económica y comparativa de proveedores | Completada como análisis histórico | Los precios, modelos y escenarios no son cotización actual. Recalcular antes de cualquier gasto. |
| 9 | Auditoría integral y extracción de formatos pendientes | Completada según informe del 2026-11-03 | Ver [informe de Fase 9](../../Fase9_Full_Audit/PHASE9_FULL_AUDIT_REPORT.md). La base SQLite/manifiestos fuente no están publicados en este repositorio. |
| 10 | Evaluación de estrategia Antigravity-first y arquitectura dual | Diseño documental completado | No se ha ejecutado benchmark con modelos aún. Las correcciones están registradas en el informe final. |
| 10.1-A | Auditoría estática del código y reconciliación de contadores | Completada | 57 archivos fuente inspeccionados; los contadores de módulos, 23 componentes y 11 métodos de prueba fueron reconciliados. Ver informe y nota de reconciliación. |
| 10.1-B | Especificación de implementación del intercambio Antigravity | Completada y aprobada | Ver `PHASE10_1B_IMPLEMENTATION_SPEC.md`. |
| 10.1-B.1 | Implementación aislada de contrato, exportación, bóveda, Gatekeeper, reintentos y CLI | Completada localmente (`feat/phase10-1b-antigravity-exchange`) | Puertas `B1–B7` superadas y `23/23` pruebas en verde (`11` previas + `12` nuevas) sin tocar producción. Ver [informe 10.1-B.1](../../Fase10_Antigravity_Architecture/PHASE10_1B1_IMPLEMENTATION_REPORT.md). |
| 10.2 | Preflight y smoke test aislado de 25 cadenas | Pendiente de autorización | Confirmar cuota/overages, permisos Windows, escritura real y recuperación en base aislada. |
| 10.3 | Benchmark controlado de 300 cadenas × 3 modelos | Pendiente | Solo después de superar 10.2; comparar calidad y operación en sesiones aisladas. |
| 10.4 | Piloto de un mod completo (~1.100–1.500 cadenas) | Pendiente | Solo después de revisar resultados del benchmark y aprobar modelo/protocolo. |
| 10.5 | Continuidad y recuperación | Pendiente | 12–15 lotes consecutivos y prueba de interrupción, sin corrupción ni pérdida silenciosa. |
| 10.6 | Decisión de escala | Pendiente | Elegir Antigravity, híbrido o API con mediciones reales; conservar infraestructura API. |
| Producción | Traducción global y exportación de paquetes de prueba | No autorizada/iniciada | Requiere decisión expresa después de las puertas anteriores. |

## Siguiente secuencia

### 10.1-A — Auditoría y reconciliación (completadas)

- [Informe técnico](../../Fase10_1_Code_Audit/Audit_20261103_113100/PHASE10_1_CODE_AUDIT_REPORT.md).
- [Reconciliación de contadores](../../Fase10_1_Code_Audit/AUDIT_10_1A_RECONCILIATION.md).
- La clasificación detallada queda en 36 módulos funcionales y 23 componentes Antigravity; las 3 suites contienen 11 métodos inspeccionados estáticamente, no ejecutados durante la auditoría.

### 10.1-B — Especificación de implementación (completada y aprobada)

Consultar [especificación técnica 10.1-B](../../Fase10_Antigravity_Architecture/PHASE10_1B_IMPLEMENTATION_SPEC.md). La decisión clave es utilizar un coordinador de intercambio supervisado, no fingir que el selector Antigravity es un proveedor API síncrono. El workspace Antigravity debe estar físicamente separado del traductor.

### 10.1-B.1 — Implementación mínima aislada (completada y verificada localmente)

Registrada en [PHASE10_1B1_IMPLEMENTATION_REPORT.md](../../Fase10_Antigravity_Architecture/PHASE10_1B1_IMPLEMENTATION_REPORT.md) sobre la rama local de código `feat/phase10-1b-antigravity-exchange` (commit `8ff5bf78b7e09e344c291bcdddc9f5a8d8b85515`). Comprende:

- Contrato JSON/JSONL versionado (`1.0.0`).
- Workspace de intercambio físicamente separado y protegido contra puntos de reparseo.
- Exportador y bóveda local de placeholders (`placeholder_vault`).
- Almacenamiento aislado e `ingestion_ledger` de lotes.
- Colector JSONL tolerante a truncamiento final y `LocalGatekeeper` de 12 puertas.
- Cuarentena, microlotes de reintento (`retry_of_batch_id`), reconciliación atómica del lote padre e idempotencia.
- CLI local sin cabecera (`exchange`) manteniendo intacta la GUI.
- Suite automatizada: **23/23 pruebas en verde** (`11` preexistentes + `12` nuevas) en directorios temporales y DB de prueba.

No se migró el esquema productivo ni se modificó la DB principal. No se habilitaron proveedores API.

**Estado de salida:** todas las puertas `B1–B7` de la especificación fueron superadas.

### 10.2 — Preflight y smoke test de 25 cadenas (pendiente de autorización)

Solo tras revisar 10.1-B.1 y contar con autorización expresa:
1. Confirmar modelos visibles en el selector en la fecha de prueba.
2. Comprobar que **AI Credit Overages = Never**; si no puede confirmarse, no ejecutar solicitudes de modelos.
3. Abrir únicamente el workspace de intercambio aislado y verificar los permisos reales de Windows.
4. Procesar 25 cadenas en una DB aislada.
5. Comprobar recuperación ante una interrupción controlada.

**Criterio:** 25/25 IDs completos y válidos al cierre, cero defectos estructurales aceptados, ningún acceso a producción y recuperación documentada.

### 10.3 — Benchmark de 300 cadenas

Comparar las mismas entradas y contexto en sesiones separadas con los modelos disponibles ese día. Medir calidad, tiempos, cuota observada, errores, reintentos e intervención humana.

### 10.4 — Piloto de un mod completo

Probar un mod representativo (~1.100–1.500 cadenas) después de revisar el benchmark y aprobar el protocolo.

### 10.5 — Continuidad y recuperación

Procesar 12–15 lotes consecutivos y fallos controlados. Medir recuperación real y ritmo sostenible.

### 10.6 — Decisión de escala

Elegir Antigravity, híbrido o API únicamente con datos medidos, nueva estimación de costes y autorización expresa.

## Invariantes de seguridad

- No modificar STOCK GAME, mods originales, BSA, load order, perfiles principales MO2 ni partidas.
- No escribir desde el agente en la DB principal, glosario maestro, memoria de traducción ni carpetas de exportación productiva.
- No instalar herramientas, descargar modelos locales, activar APIs, comprar créditos ni ejecutar traducciones masivas sin autorización expresa.
- La infraestructura API se conserva como alternativa; las estimaciones de Fase 8 no autorizan activarla.
- Cada fase se detiene en su puerta de salida y entrega un informe antes de continuar.
