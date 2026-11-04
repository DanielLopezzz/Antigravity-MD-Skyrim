# Nolvus AI Translator — Documentación central (ES-419)

Documentación del proyecto para localizar **Nolvus Awakening 6.0.21** al español latinoamericano neutro (**ES-419**). Este repositorio contiene solo documentación; el código fuente vive en un repositorio local separado.

## Empieza aquí

| Documento | Para qué sirve |
|---|---|
| [PROJECT_STATUS.md](./PROJECT_STATUS.md) | **Fuente única de verdad**: estado de cada fase y subsistema, rama/commits, pruebas, cobertura, control económico, riesgos y próxima puerta. |
| [CHANGELOG.md](./CHANGELOG.md) | Historial cronológico de trabajo verificado (fases, archivos, commits, pruebas, resultados). |
| [ROADMAP.md](./ROADMAP.md) | Solo trabajo futuro y puertas pendientes (10.3-B Ejecución Piloto Real → Producción). |
| [DECISIONS.md](./DECISIONS.md) | Decisiones técnicas y operativas vigentes e históricas, con fecha y motivo. |
| [DOCUMENTATION_POLICY.md](./DOCUMENTATION_POLICY.md) | Reglas de mantenimiento documental. |

**Estado en una línea (2026-11-04):** Fase 10.3-C ejecutó cinco lotes reales de 25 cadenas (`gpt-6-luna`): 100 se aprobaron e importaron al staging aislado, 23 candidatas válidas quedaron retenidas por la cuarentena atómica de dos incidencias del glosario y el coste medido fue `$0.011345 USD`; la producción sigue bloqueada. La puerta siguiente es la remediación dirigida del lote de scripts, protección de nombres propios y revisión lingüística, no una ampliación masiva.

## Evidencia por fase (informes históricos preservados)

### Fase 10.3 — Preparación Offline (10.3-A), Auditoría Preflight y Remediación Offline (10.3-B)
- [Informe del piloto transversal real de 125 cadenas — Fase 10.3-C (resultado parcial, consumo medido, hallazgos y próxima puerta)](./Fase10_Antigravity_Architecture/PHASE10_3C_LIVE_PILOT_125_REPORT.md)
- [Informe del piloto real de 25 cadenas — Fase 10.3-B (ejecución y remediación)](./Fase10_Antigravity_Architecture/PHASE10_3B_LIVE_PILOT_EXECUTION_REPORT.md)
- [Informe de remediación pre-piloto offline Fase 10.3-B (resolución de discrepancias `D-CRED-01..03`, `D-BUD-01`, `D-DOC-01`, `D-ISO-01..02` y 10 pruebas obligatorias)](./Fase10_Antigravity_Architecture/PHASE10_3B_PREFLIGHT_REMEDIATION_REPORT.md)
- [Auditoría preflight de solo lectura para Fase 10.3-B (credenciales, modelo, candado API, presupuesto, aislamiento y discrepancias)](./Fase10_Antigravity_Architecture/PHASE10_3B_READONLY_PREFLIGHT_AUDIT_REPORT.md)
- [Informe técnico de preparación offline Fase 10.3-A (25 cadenas reales 5×5, bóveda de 37 placeholders, payload `/v1/responses` y verificación de Gatekeeper)](./Fase10_Antigravity_Architecture/PHASE10_3A_OFFLINE_PILOT_PREPARATION_REPORT.md)

### Fase 10.2 — Infraestructura OpenAI API-First y Control Presupuestario Local
- [Informe de implementación Fase 10.2 (OpenAI API-First, BudgetController, Batch API y estimación de piloto)](./Fase10_Antigravity_Architecture/PHASE10_2_OPENAI_API_FIRST_REPORT.md)

### Fase 10.1 — Auditoría de código, especificación e implementación de intercambio Antigravity
- [Auditoría 10.1-A](./Fase10_1_Code_Audit/Audit_20261103_113100/PHASE10_1_CODE_AUDIT_REPORT.md) · [Reconciliación 10.1-A](./Fase10_1_Code_Audit/AUDIT_10_1A_RECONCILIATION.md)
- [Especificación 10.1-B](./Fase10_Antigravity_Architecture/PHASE10_1B_IMPLEMENTATION_SPEC.md)
- [Informe de implementación 10.1-B.1 (consolidado y reconciliado)](./Fase10_Antigravity_Architecture/PHASE10_1B1_IMPLEMENTATION_REPORT.md)

### Fase 10.0 — Antigravity-first y arquitectura dual (antecedente histórico)
- [PHASE10_FINAL_REPORT.md](./Fase10_Antigravity_Architecture/PHASE10_FINAL_REPORT.md) · [ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md](./Fase10_Antigravity_Architecture/ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md) · [DUAL_PROVIDER_ARCHITECTURE.md](./Fase10_Antigravity_Architecture/DUAL_PROVIDER_ARCHITECTURE.md) · [TRANSLATION_EXECUTION_ENGINE.md](./Fase10_Antigravity_Architecture/TRANSLATION_EXECUTION_ENGINE.md) · [SCALABILITY_AND_RISK_ASSESSMENT.md](./Fase10_Antigravity_Architecture/SCALABILITY_AND_RISK_ASSESSMENT.md) · [BENCHMARK_PLAN_ANTIGRAVITY.md](./Fase10_Antigravity_Architecture/BENCHMARK_PLAN_ANTIGRAVITY.md)

### Fase 9 — Auditoría integral de mods
- [PHASE9_FULL_AUDIT_REPORT.md](./Fase9_Full_Audit/PHASE9_FULL_AUDIT_REPORT.md)

### Fases 7–8 — Escaneo real y estimación económica (histórico)
- [AUDIT_PHASE7.md](./Fase7_y_Fase8_Informes/AUDIT_PHASE7.md) · [coverage_snapshot_1.md](./Fase7_y_Fase8_Informes/coverage_snapshot_1.md) · [API_COST_ESTIMATE_V2.md](./Fase7_y_Fase8_Informes/API_COST_ESTIMATE_V2.md) · [API_COST_ESTIMATE.md](./Fase7_y_Fase8_Informes/API_COST_ESTIMATE.md) · [MODEL_BENCHMARK_PLAN.md](./Fase7_y_Fase8_Informes/MODEL_BENCHMARK_PLAN.md)

### Políticas lingüísticas vigentes
- [TRANSLATION_POLICY_ES419.md](./Fase7_y_Fase8_Informes/TRANSLATION_POLICY_ES419.md) · [GLOSSARY_POLICY.md](./Fase7_y_Fase8_Informes/GLOSSARY_POLICY.md)

### Fases 0–6 — Documentación base (histórico; contrastar con el código actual)
- [ARCHITECTURE.md](./Documentacion_Base_Fases_0_a_6/ARCHITECTURE.md) · [SUPPORTED_FORMATS.md](./Documentacion_Base_Fases_0_a_6/SUPPORTED_FORMATS.md) · [TECHNICAL_DECISIONS.md](./Documentacion_Base_Fases_0_a_6/TECHNICAL_DECISIONS.md) · [COMPATIBILITY.md](./Documentacion_Base_Fases_0_a_6/COMPATIBILITY.md) · [TESTING.md](./Documentacion_Base_Fases_0_a_6/TESTING.md) · [USER_GUIDE.md](./Documentacion_Base_Fases_0_a_6/USER_GUIDE.md) · [LICENSES_AND_THIRD_PARTY.md](./Documentacion_Base_Fases_0_a_6/LICENSES_AND_THIRD_PARTY.md)

### Documentos de transición (sustituidos, conservados por trazabilidad)
- [CURRENT_PROJECT_STATE.md](./CURRENT_PROJECT_STATE.md) — estado y decisiones previos a la centralización; su función pasa a `PROJECT_STATUS.md` y `DECISIONS.md`.
- [Documentacion_Base_Fases_0_a_6/ROADMAP.md](./Documentacion_Base_Fases_0_a_6/ROADMAP.md) — roadmap anterior; sustituido por `ROADMAP.md` (raíz).
- [DOCUMENTATION_CLEANUP_REPORT.md](./DOCUMENTATION_CLEANUP_REPORT.md) — registro de la depuración documental anterior.
