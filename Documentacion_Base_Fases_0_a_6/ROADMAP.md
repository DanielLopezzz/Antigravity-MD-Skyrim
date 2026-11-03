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
| 10 | Evaluación de estrategia Antigravity-first y arquitectura dual | Diseño documental completado | No se ha ejecutado benchmark ni se ha implementado el adaptador/Gatekeeper. Las correcciones están registradas en el informe final. |
| 10.1 | Implementación mínima aislada del adaptador JSON/JSONL y Gatekeeper | Pendiente de autorización | Desarrollar únicamente en el workspace de pruebas, con DB aislada, permisos mínimos y sin tocar producción. |
| 10.2 | Preflight y smoke test aislado de 25 cadenas | Pendiente | Solo después de implementar y probar el contrato; confirmar cuota/overages, permisos Windows, escritura real y recuperación. |
| 10.3 | Benchmark controlado de 300 cadenas × 3 modelos | Pendiente | Solo después de superar 10.2; comparar calidad y operación en sesiones aisladas. |
| 10.4 | Piloto de un mod completo (~1.100–1.500 cadenas) | Pendiente | Solo después de revisar resultados del benchmark y aprobar modelo/protocolo. |
| 10.5 | Continuidad y recuperación | Pendiente | 12–15 lotes consecutivos y prueba de interrupción, sin corrupción ni pérdida silenciosa. |
| 10.6 | Decisión de escala | Pendiente | Elegir Antigravity, híbrido o API con mediciones reales; conservar infraestructura API. |
| Producción | Traducción global y exportación de paquetes de prueba | No autorizada/iniciada | Requiere decisión expresa después de las puertas anteriores. |

## Siguiente secuencia

### 10.1 — Implementación mínima, aislada y autorizada

Antes de ejecutar pruebas se necesita confirmar qué componentes del código actual ya existen. El repositorio publicado no contiene el código fuente, por lo que la inspección debe hacerse en el workspace local. Solo después de esa inspección y autorización se implementará lo que falte:

- Adaptador de exportación/recepción JSON y JSONL.
- Workspace de intercambio físicamente separado (por ejemplo, `C:\\NolvusAIT_Exchange\\job_<job_id>\\`), abierto como único workspace de Antigravity; no abrir la raíz del proyecto.
- Contrato de lote versionado con IDs estables, hashes y contexto.
- Gatekeeper local independiente, con DB de prueba y permisos mínimos.
- Validadores para conteo, conjunto de IDs, duplicados, placeholders, etiquetas, claves, glosario y ES-419.
- Cuarentena y micro-reintentos para líneas incompletas o rechazadas.
- Pruebas unitarias y de recuperación sin acceder a la DB productiva.

No dar al agente acceso de escritura a DB principal, glosario, TM, STOCK GAME, MO2 ni mods originales. No activar proveedores API.

### 10.2 — Preflight y smoke test de 25 cadenas

Solo cuando 10.1 supere revisión:
1. Confirmar modelos visibles en el selector en la fecha de la prueba.
2. Comprobar que **AI Credit Overages = Never**; si no puede confirmarse, no ejecutar solicitudes de modelos.
3. Abrir solo el workspace de intercambio aislado y verificar los permisos reales de la versión Windows instalada; limitar la escritura del agente al directorio de salida de esa carpeta.
4. Procesar 25 cadenas reales en una DB aislada.
5. Interrumpir una escritura de prueba y comprobar que solo se recuperan líneas completas válidas; la línea incompleta se pone en cuarentena.

**Criterio:** 25/25 IDs completos y válidos al cierre, cero defectos estructurales aceptados, ningún acceso a producción y recuperación documentada.

### 10.3 — Benchmark de 300 cadenas

Ejecutar las mismas entradas/contexto con cada modelo que siga disponible, en sesiones separadas y con revisión ciega. Medir calidad, primera pasada, errores, latencia, cuota, reintentos y acciones humanas.

### 10.4 — Piloto de un mod completo

Probar un mod representativo (~1.100–1.500 cadenas) solo después de revisar el benchmark y autorizar modelo/protocolo. Verificar coherencia terminológica entre nombres, objetivos, diálogos y libros.

### 10.5 — Continuidad y recuperación

Procesar 12–15 lotes consecutivos y ejecutar fallos controlados. Medir continuidad, truncamientos, tasa real de recuperación, reintentos y cuota.

### 10.6 — Decisión de escala

Calcular el ritmo sostenible con cadenas validadas por hora/día y cuota observada. Documentar una decisión entre Antigravity, híbrido o API. La conmutación a API requiere nueva cotización y autorización expresa.

## Invariantes de seguridad

- No modificar STOCK GAME, mods originales, BSA, load order, perfiles principales MO2 ni partidas.
- No escribir desde el agente en la DB principal, glosario maestro, memoria de traducción ni carpetas de exportación productiva.
- No instalar herramientas, descargar modelos locales, activar APIs, comprar créditos ni ejecutar traducciones masivas sin autorización expresa.
- La infraestructura API se conserva como alternativa; las estimaciones de Fase 8 no autorizan activarla.
- Cada fase se detiene en su puerta de salida y entrega un informe antes de continuar.
