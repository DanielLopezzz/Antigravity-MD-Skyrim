# Nolvus AI Translator — Informes Técnicos y Arquitectura (ES-419)

Repositorio de documentación e informes técnicos en formato Markdown del proyecto **Nolvus AI Translator** para la localización integral de **Nolvus Awakening 6.0.21 (Skyrim AE)** al español latinoamericano neutro (ES-419).

---

## 1. Fase 10 — Estrategia Antigravity-First y Arquitectura Dual de Proveedores

Diseño completo para ejecutar las traducciones utilizando los modelos de la suscripción de **Google Antigravity** (Gemini 4 Argon, Claude Sonnet 5.5 y Claude Opus 5.5) mediante intercambio transaccional .json/.jsonl y validación local automática, conservando la compatibilidad con APIs externas:

- [**PHASE10_FINAL_REPORT.md**](./Fase10_Antigravity_Architecture/PHASE10_FINAL_REPORT.md) — Informe Ejecutivo Final de la Fase 10.
- [**BENCHMARK_PLAN_ANTIGRAVITY.md**](./Fase10_Antigravity_Architecture/BENCHMARK_PLAN_ANTIGRAVITY.md) — Plan de Benchmark de 300 cadenas en 11 categorías para Gemini 4 Argon, Claude Sonnet 5.5 y Claude Opus 5.5.
- [**ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md**](./Fase10_Antigravity_Architecture/ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md) — Arquitectura de Integración Transaccional (.json + .jsonl + Gatekeeper CLI Local).
- [**DUAL_PROVIDER_ARCHITECTURE.md**](./Fase10_Antigravity_Architecture/DUAL_PROVIDER_ARCHITECTURE.md) — Arquitectura Dual Desacoplada (AntigravityExchangeProvider, APIProvider, LocalLLMProvider).
- [**TRANSLATION_EXECUTION_ENGINE.md**](./Fase10_Antigravity_Architecture/TRANSLATION_EXECUTION_ENGINE.md) — Motor de Ejecución, Deduplicación TranslationWorkUnit, Lotes Adaptativos y Ordenamiento *Anchor-First*.
- [**SCALABILITY_AND_RISK_ASSESSMENT.md**](./Fase10_Antigravity_Architecture/SCALABILITY_AND_RISK_ASSESSMENT.md) — Evaluación de Escalabilidad (Workpacks), Riesgos y Criterios de Conmutación.

---

## 2. Fase 9 — Auditoría Integral de Mods, Cobertura y Extracción Exhaustiva

- [**PHASE9_FULL_AUDIT_REPORT.md**](./Fase9_Full_Audit/PHASE9_FULL_AUDIT_REPORT.md) — Informe de los 26 puntos obligatorios sobre los 2.587 mods de MO2, 2.371 plugins activos, 583 archivos .bsa y 464.597 cadenas extraídas.

---

## 3. Fases 7 y 8 — Escaneo Global, Políticas ES-419 y Estimación Económica API

- [**AUDIT_PHASE7.md**](./Fase7_y_Fase8_Informes/AUDIT_PHASE7.md) — Auditoría del Escaneo Real de la Fase 7 (Snapshot #1).
- [**coverage_snapshot_1.md**](./Fase7_y_Fase8_Informes/coverage_snapshot_1.md) — Reporte de Cobertura del Snapshot #1.
- [**TRANSLATION_POLICY_ES419.md**](./Fase7_y_Fase8_Informes/TRANSLATION_POLICY_ES419.md) — Política Lingüística y Estilística Español Latinoamericano Neutro (ES-419).
- [**GLOSSARY_POLICY.md**](./Fase7_y_Fase8_Informes/GLOSSARY_POLICY.md) — Política del Glosario Maestro y Jerarquía Terminológica.
- [**MODEL_BENCHMARK_PLAN.md**](./Fase7_y_Fase8_Informes/MODEL_BENCHMARK_PLAN.md) — Plan Original de Benchmark (Histórico pre-Antigravity).
- [**API_COST_ESTIMATE_V2.md**](./Fase7_y_Fase8_Informes/API_COST_ESTIMATE_V2.md) — Estimación Económica Corregida (Versión 2) para Proveedores API y Locales.
- [**API_COST_ESTIMATE.md**](./Fase7_y_Fase8_Informes/API_COST_ESTIMATE.md) — Estimación Económica Original (Versión 1).

---

## 4. Documentación Base del Sistema (Fases 0 a 6)

- [**ARCHITECTURE.md**](./Documentacion_Base_Fases_0_a_6/ARCHITECTURE.md)
- [**SUPPORTED_FORMATS.md**](./Documentacion_Base_Fases_0_a_6/SUPPORTED_FORMATS.md)
- [**TECHNICAL_DECISIONS.md**](./Documentacion_Base_Fases_0_a_6/TECHNICAL_DECISIONS.md)
- [**COMPATIBILITY.md**](./Documentacion_Base_Fases_0_a_6/COMPATIBILITY.md)
- [**TESTING.md**](./Documentacion_Base_Fases_0_a_6/TESTING.md)
- [**USER_GUIDE.md**](./Documentacion_Base_Fases_0_a_6/USER_GUIDE.md)
- [**ROADMAP.md**](./Documentacion_Base_Fases_0_a_6/ROADMAP.md)
- [**LICENSES_AND_THIRD_PARTY.md**](./Documentacion_Base_Fases_0_a_6/LICENSES_AND_THIRD_PARTY.md)
