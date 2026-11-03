# Informe de depuración y validación documental

**Fecha:** 2026-11-03  
**Rama de trabajo:** `docs/cleanup-and-antigravity-validation-2026-11-03`  
**Estado:** Cambios documentales preparados para revisión; no fusionados en `main`.

## 1. Objetivo

Consolidar la documentación del proyecto Nolvus AI Translator, separar estado vigente de evidencia histórica, corregir afirmaciones no demostradas y definir la opción más segura para validar Antigravity sin API.

## 2. Decisión técnica resultante

La ruta preferida para validar Antigravity es **intercambio supervisado JSON + JSONL con un Gatekeeper local separado**:

1. El núcleo exporta un manifiesto JSON y un JSONL de entrada con IDs estables, hashes, contexto y texto protegido.
2. El agente Antigravity solo lee el protocolo/entrada y escribe un archivo JSONL completo en staging.
3. No se depende de `write_file(..., Append=True)`, hot-folder autónomo ni automatización de clics.
4. El usuario o un proceso local controlado ejecuta el Gatekeeper por separado. Durante pruebas solo se permite ingesta a DB aislada.
5. El Gatekeeper revisa esquema, IDs esperados, hashes, duplicados, placeholders, etiquetas, claves no traducibles, glosario y ES-419.
6. Solo las líneas completas y válidas pueden rescatarse. Las líneas parciales o defectuosas se ponen en cuarentena y se reintentan por UID.
7. El agente nunca recibe permisos de escritura sobre DB principal, glosario, memoria de traducción, STOCK GAME, mods originales ni exportación productiva.

**Por qué es la mejor opción para esta fase:** minimiza permisos, no requiere API ni automatización de interfaz, conserva trazabilidad y permite probar recuperación sin conceder al agente acceso a SQLite. JSONL facilita recuperar registros completos por línea, pero no es una transacción ni garantiza por sí solo recuperación del 100 %.

## 3. Cuotas y permisos de Antigravity

La documentación oficial consultada indica que:
- Los límites de uso pueden variar y dependen del esfuerzo de cada tarea; no se debe convertir una cuota observada en una cifra permanente.
- Los planes Google AI Pro/Ultra pueden consumir créditos adicionales cuando se agota la cuota base si AI Credit Overages está configurado para ello.
- La opción **AI Credit Overages = Never** evita el consumo automático de créditos al agotarse la cuota base; debe comprobarse manualmente en Settings antes de la prueba.
- La documentación de permisos actualizada tiene diferencias por sistema operativo; en Windows debe verificarse la interfaz/configuración efectiva de la versión instalada.

Fuentes:
- [Antigravity Plans and Overages](https://antigravity.google/docs/plans)
- [Antigravity Permissions](https://antigravity.google/docs/permissions)
- [Antigravity Agent Settings](https://antigravity.google/docs/agent-settings)

## 4. Correcciones documentales aplicadas

- README: dejó de publicar el resumen incorrecto de Fase 9 (`2.587` mods/`2.371` plugins/`464.597` cadenas) y ahora enlaza cifras del informe Fase 9: `3.700` mods activos, `3.717` plugins, `583` BSA, `1.168.337` elementos, `328.485` ocurrencias de Corte C y `240.244` únicas pendientes + unas `440` JSON 4A adicionales.
- Roadmap: dejó de decir que la Fase 9 era el piloto de traducción. Ahora refleja Fase 9 como auditoría completada, Fase 10 como diseño documental completado y las pruebas como pendientes.
- Fase 10: se retiraron afirmaciones de soporte de `Append=True`, recuperación garantizada del 100 %, ejecución del Gatekeeper por el agente y permisos automáticos universales.
- Escalabilidad: los valores de cadenas/día, sesiones, Workpacks y umbrales de decisión se identifican como hipótesis/estimaciones; ya no activan API automáticamente.
- Benchmark, motor y arquitectura dual: se identifican como planes/diseños pendientes de validación de código y corpus, no como sistemas operativos certificados por esta revisión.
- Fases 7–8 y documentos base 0–6: se marcaron como históricos cuando sus números, modelos, precios, versiones o instrucciones pueden estar sustituidos.
- En los documentos depurados se reemplazaron enlaces `file:///` no portables: se usan enlaces relativos cuando el destino está en GitHub y referencias de ruta local no incluida cuando el código/artefacto permanece en el PC.
- Políticas ES-419 y glosario: se mantienen como fuentes documentales, sin afirmar que el estado actual de DB/código esté sincronizado hasta comprobarlo.

## 5. Documentos actualizados

**Índice y fuente de verdad**
- `README.md`
- `CURRENT_PROJECT_STATE.md`
- `DOCUMENTATION_POLICY.md`
- `Documentacion_Base_Fases_0_a_6/ROADMAP.md`

**Fase 10**
- `Fase10_Antigravity_Architecture/PHASE10_FINAL_REPORT.md`
- `Fase10_Antigravity_Architecture/ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md`
- `Fase10_Antigravity_Architecture/BENCHMARK_PLAN_ANTIGRAVITY.md`
- `Fase10_Antigravity_Architecture/DUAL_PROVIDER_ARCHITECTURE.md`
- `Fase10_Antigravity_Architecture/TRANSLATION_EXECUTION_ENGINE.md`
- `Fase10_Antigravity_Architecture/SCALABILITY_AND_RISK_ASSESSMENT.md`

**Fases anteriores y políticas**
- `Fase7_y_Fase8_Informes/AUDIT_PHASE7.md`
- `Fase7_y_Fase8_Informes/coverage_snapshot_1.md`
- `Fase7_y_Fase8_Informes/MODEL_BENCHMARK_PLAN.md`
- `Fase7_y_Fase8_Informes/API_COST_ESTIMATE.md`
- `Fase7_y_Fase8_Informes/API_COST_ESTIMATE_V2.md`
- `Fase7_y_Fase8_Informes/TRANSLATION_POLICY_ES419.md`
- `Fase7_y_Fase8_Informes/GLOSSARY_POLICY.md`
- `Documentacion_Base_Fases_0_a_6/ARCHITECTURE.md`
- `Documentacion_Base_Fases_0_a_6/SUPPORTED_FORMATS.md`
- `Documentacion_Base_Fases_0_a_6/TECHNICAL_DECISIONS.md`
- `Documentacion_Base_Fases_0_a_6/COMPATIBILITY.md`
- `Documentacion_Base_Fases_0_a_6/TESTING.md`
- `Documentacion_Base_Fases_0_a_6/USER_GUIDE.md`
- `Documentacion_Base_Fases_0_a_6/LICENSES_AND_THIRD_PARTY.md`

Los informes detallados de Fase 9 y los snapshots se preservan como evidencia; no se reescribieron los resultados de auditoría cerrados.

## 6. Validaciones que sí se hicieron

- Confirmación de que la rama de limpieza existe y que los documentos principales pueden leerse desde ella.
- Verificación de que el README ya no contiene el resumen numérico erróneo de `2.587` mods y `2.371` plugins.
- Búsqueda en los documentos principales de las afirmaciones no soportadas sobre `Append=True`, recuperación del 100 % y ejecución del Gatekeeper por el agente; se corrigieron en los documentos de integración y Fase 10.
- Confirmación de que el estado documental distingue diseño, propuesta, medición reportada e implementación aún no verificada.

## 7. Lo que no se ha hecho ni debe darse por hecho

- No se ejecutó el smoke test de 25 cadenas.
- No se ejecutó el benchmark de 300 cadenas.
- No se construyó el corpus real desde la DB local.
- No se verificó directamente el código fuente ni se reejecutaron sus tests, porque no está publicado en este repositorio documental.
- No se volvió a escanear Nolvus ni se comprobaron los manifiestos/DB del snapshot de Fase 9.
- No se modificó STOCK GAME, MO2, la DB principal, glosario, TM ni partidas.
- No se activó API, facturación, créditos, modelos locales ni traducción masiva.

## 8. Siguiente puerta técnica

Antes del smoke test, inspeccionar el código local e implementar o completar —con autorización— el adaptador JSON/JSONL y Gatekeeper en entorno aislado. Después ejecutar preflight y 25 cadenas en una DB de prueba; comprobar modelo, AI Credit Overages = Never, permisos Windows, integridad JSONL y recuperación ante interrupción. Solo si supera esta puerta avanzar al benchmark de 300 cadenas.

La rama requiere revisión antes de fusionarse en `main`.
