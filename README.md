# Nolvus AI Translator — Documentación central (ES-419)

Documentación del proyecto para localizar **Nolvus Awakening 6.0.21** al español latinoamericano neutro (**ES-419**).

> **Punto de entrada:** consulta primero [Estado actual del proyecto](./CURRENT_PROJECT_STATE.md). Este archivo concentra las decisiones y cifras vigentes. Los informes de auditoría son evidencia del corte en que se generaron; no deben tratarse como métricas vivas.

## Estado y reglas documentales

- [CURRENT_PROJECT_STATE.md](./CURRENT_PROJECT_STATE.md) — cifras vigentes del corte de Fase 9, decisiones, controles y estado actual.
- [DOCUMENTATION_POLICY.md](./DOCUMENTATION_POLICY.md) — jerarquía, mantenimiento y tratamiento de documentos históricos.
- [DOCUMENTATION_CLEANUP_REPORT.md](./DOCUMENTATION_CLEANUP_REPORT.md) — cambios aplicados, validación realizada y pendientes.
- [ROADMAP.md](./Documentacion_Base_Fases_0_a_6/ROADMAP.md) — hitos concluidos y puertas de validación pendientes.

## Fase 10 — Diseño Antigravity-first y arquitectura dual

La Fase 10 está **completada en diseño**. El benchmark y el adaptador de intercambio todavía no se han ejecutado/implementado.

- [PHASE10_FINAL_REPORT.md](./Fase10_Antigravity_Architecture/PHASE10_FINAL_REPORT.md) — decisiones y alcance del diseño, con una adenda de validación técnica.
- [BENCHMARK_PLAN_ANTIGRAVITY.md](./Fase10_Antigravity_Architecture/BENCHMARK_PLAN_ANTIGRAVITY.md) — benchmark de 300 cadenas, tres modelos y rúbrica.
- [ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md](./Fase10_Antigravity_Architecture/ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md) — contrato de intercambio JSON/JSONL, permisos y Gatekeeper.
- [DUAL_PROVIDER_ARCHITECTURE.md](./Fase10_Antigravity_Architecture/DUAL_PROVIDER_ARCHITECTURE.md) — núcleo común y vías Antigravity/API desacopladas.
- [TRANSLATION_EXECUTION_ENGINE.md](./Fase10_Antigravity_Architecture/TRANSLATION_EXECUTION_ENGINE.md) — unidades de trabajo, lotes, QA, reintentos y trazabilidad.
- [SCALABILITY_AND_RISK_ASSESSMENT.md](./Fase10_Antigravity_Architecture/SCALABILITY_AND_RISK_ASSESSMENT.md) — pruebas progresivas, cuotas, riesgos y decisión de escala.

## Fase 9 — Auditoría integral

- [PHASE9_FULL_AUDIT_REPORT.md](./Fase9_Full_Audit/PHASE9_FULL_AUDIT_REPORT.md) — evidencia detallada del inventario y extracción.
- **Cifras clave del informe:** 3.700 mods activos, 3.717 plugins activos, 583 BSA, 1.168.337 elementos inventariados; 328.485 ocurrencias traducibles en Corte C, 240.244 cadenas únicas pendientes y unas 440 cadenas únicas narrativas adicionales del JSON 4A. El volumen recomendado se expresa como **~240.684**, porque el componente JSON adicional es aproximado.

## Políticas lingüísticas y terminológicas vigentes

- [TRANSLATION_POLICY_ES419.md](./Fase7_y_Fase8_Informes/TRANSLATION_POLICY_ES419.md) — español latinoamericano neutro, separación de traducción propia y oficial, protección técnica.
- [GLOSSARY_POLICY.md](./Fase7_y_Fase8_Informes/GLOSSARY_POLICY.md) — decisiones terminológicas y tratamiento de nombres propios.

## Evidencia histórica de Fases 7–8

- [AUDIT_PHASE7.md](./Fase7_y_Fase8_Informes/AUDIT_PHASE7.md)
- [coverage_snapshot_1.md](./Fase7_y_Fase8_Informes/coverage_snapshot_1.md)
- [MODEL_BENCHMARK_PLAN.md](./Fase7_y_Fase8_Informes/MODEL_BENCHMARK_PLAN.md) — benchmark anterior, sustituido por el plan de Fase 10.
- [API_COST_ESTIMATE_V2.md](./Fase7_y_Fase8_Informes/API_COST_ESTIMATE_V2.md) — estimación histórica; no es una cotización vigente.
- [API_COST_ESTIMATE.md](./Fase7_y_Fase8_Informes/API_COST_ESTIMATE.md) — versión histórica anterior.

## Documentación base de Fases 0–6

Los documentos de esta carpeta describen el diseño y las pruebas de las fases iniciales. Sus versiones de runtime, rutas, interfaz o estado de funciones deben contrastarse con Fase 9 y con el código actual antes de usarse como instrucciones operativas.

- [ARCHITECTURE.md](./Documentacion_Base_Fases_0_a_6/ARCHITECTURE.md)
- [SUPPORTED_FORMATS.md](./Documentacion_Base_Fases_0_a_6/SUPPORTED_FORMATS.md)
- [TECHNICAL_DECISIONS.md](./Documentacion_Base_Fases_0_a_6/TECHNICAL_DECISIONS.md)
- [COMPATIBILITY.md](./Documentacion_Base_Fases_0_a_6/COMPATIBILITY.md)
- [TESTING.md](./Documentacion_Base_Fases_0_a_6/TESTING.md)
- [USER_GUIDE.md](./Documentacion_Base_Fases_0_a_6/USER_GUIDE.md)
- [LICENSES_AND_THIRD_PARTY.md](./Documentacion_Base_Fases_0_a_6/LICENSES_AND_THIRD_PARTY.md)
