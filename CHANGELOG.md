# Registro Cronológico de Cambios y Auditorías (`CHANGELOG.md`)

**Proyecto:** Nolvus AI Translator (ES-419)  
**Propósito:** Historial cronológico verificable de las fases ejecutadas, auditorías, especificaciones, implementaciones de código, commits y resultados de pruebas.  
**Nota de organización:** Este documento registra únicamente trabajo ya ejecutado y verificado. Para el estado actual consolidado consulte [PROJECT_STATUS.md](./PROJECT_STATUS.md); para las fases pendientes consulte [ROADMAP.md](./ROADMAP.md).

---

## [2026-11-03] — Revisión técnica integral, resolución de cobertura Fase 9 y consolidación post-Fase 10.1-B.1

### 1. Reconciliación documental y reorganización centralizada (Partes I–III)
- **Fase / Subtarea:** Consolidación documental post-Fase 10.1-B.1.
- **Qué se hizo:**
  1. Se reconciliaron las tres discrepancias detectadas en la primera edición de `PHASE10_1B1_IMPLEMENTATION_REPORT.md`:
     - **11 pruebas originales:** Se verificó en Git (`fdea297..0b4ff8d`) que `tests/test_end_to_end.py` (2 métodos), `tests/test_formats.py` (6 métodos) y `tests/test_protector_glossary_tm.py` (3 métodos) permanecen 100 % intactos (`0` bytes alterados) y se corrigieron los nombres reales de los 27 métodos de prueba.
     - **Esquema SQLite:** Se documentó la relación exacta entre los **12 objetos DDL explícitos** de `core/database.py` (11 tablas regulares + 1 tabla virtual FTS5 `tm_fts`), las **17 entradas en `sqlite_master`** (al sumar las 4 tablas sombra FTS5 y `sqlite_sequence`), el listado previo de 10 tablas de la Auditoría 10.1-A y las **8 tablas regulares** del nuevo `ExchangeStateStore` (`translation/exchange_store.py`).
     - **Puertas B1–B7:** Se alineó la tabla de aceptación con la numeración y definición exacta de `PHASE10_1B_IMPLEMENTATION_SPEC.md` (`B1: Contrato`, `B2: Aislamiento`, `B3: Protección`, `B4: Gatekeeper`, `B5: Recuperación`, `B6: CLI`, `B7: Regresión`).
  2. Se implantó el nuevo sistema documental centralizado con 5 documentos canónicos en la raíz (`README.md`, `PROJECT_STATUS.md`, `CHANGELOG.md`, `ROADMAP.md`, `DECISIONS.md`) preservando intactos los informes históricos de auditoría.
- **Archivos afectados (repositorio documental):** `README.md`, `PROJECT_STATUS.md`, `CHANGELOG.md`, `ROADMAP.md`, `DECISIONS.md`, `CURRENT_PROJECT_STATE.md`, `DOCUMENTATION_POLICY.md`, `Documentacion_Base_Fases_0_a_6/ROADMAP.md`, `Fase10_Antigravity_Architecture/PHASE10_1B1_IMPLEMENTATION_REPORT.md`.
- **Resultado:** Documentación reconciliada y libre de contradicciones entre estado actual, historial y trabajo futuro.

### 2. Corrección de defectos de concurrencia, reconciliación multi-salto, deduplicación y QA (Commit `844aa1b`)
- **Fase / Subtarea:** Revisión técnica integral — Bloque Antigravity Exchange, QA y UI (Partes IV y V).
- **Qué se hizo:**
  1. **`ExchangeStateStore` y `LocalGatekeeper` (`A-01`):** Se añadió `threading.RLock()`, se trasladó `PRAGMA journal_mode=WAL;` a `_init_schema()`, y se crearon `store_batch_export_bundle()`, `commit_gatekeeper_verdict()` y `load_vault_envelopes_for_batch()` para que toda la evaluación del Gatekeeper (transiciones `OUTPUT_RECEIVED -> VALIDATING -> resultado`, candidatos válidos, cuarentena e `ingestion_ledger`) se persista en una única transacción SQLite atómica.
  2. **Reconciliación de reintentos multi-salto (`A-02`):** Se corrigió `AntigravityExchangeService.validate_and_process_output()` y `_reconcile_retry_into_parent_batch()` para propagar al lote padre los candidatos válidos parciales de un micro-lote (`retry1`) incluso cuando queda algún ítem pendiente para `retry2`, y para propagar el estado `QUARANTINED` al padre si se agota el límite de reintentos.
  3. **Deduplicación de lotes consecutivos y propagación TM (`A-03`):** `export_batch_from_test_db()` ahora pre-aplica coincidencias exactas de TM, excluye `string_uid` ya activos en el mismo `job_id` (`get_active_exported_uids`) y deduplica `(source_hash, context_signature)` dentro del lote; `import_validated_exchange_results()` propaga automáticamente las traducciones `APPROVED` a las ocurrencias pendientes idénticas del mismo snapshot.
  4. **Detección de inyección cruda y caché de glosario (`A-04`, `M-01`):** `VariableProtector.restore_and_verify()` ahora detecta y rechaza etiquetas HTML/XML o tokens técnicos (`<br>`, `<Alias=...>`, `<mag>`, `[pagebreak]`, `$MCMKey`, `FormID|Plugin`) inyectados fuera de los placeholders `⟦VAR_n⟧`; `TranslationQualityValidator.validate()` valida términos de glosario por límite de palabra con tolerancia plural (`s`/`es`); `GlossaryManager` cachea las expresiones regulares compiladas.
  5. **Robustez de interfaz gráfica (`M-02`):** `NolvusTranslatorApp._action_save_config()` valida `batch_size` dentro de `try/except` y usa `model_copy(update=...)` sin sobrescribir campos de configuración no expuestos en el formulario; se añadió `_on_close_window()` (`WM_DELETE_WINDOW`) para cerrar `ThreadPoolExecutor` limpiamente.
- **Archivos afectados (repositorio de código):**
  - `src/nolvus_translator/translation/exchange_store.py`
  - `src/nolvus_translator/quality/gatekeeper.py`
  - `src/nolvus_translator/translation/exchange_service.py`
  - `src/nolvus_translator/quality/protector.py`
  - `src/nolvus_translator/quality/validator.py`
  - `src/nolvus_translator/lore/glossary.py`
  - `src/nolvus_translator/translation/pipeline.py`
  - `src/nolvus_translator/ui/app.py`
  - `tests/test_gatekeeper.py`
- **Commit:** `844aa1b` en rama `feat/phase10-1b-antigravity-exchange`.
- **Pruebas ejecutadas:** `python -m unittest discover -s tests -t . -v` (`24/24 OK` en `3.016s`).
- **Resultado:** `COMPLETADO`.

### 3. Implementación de hallazgos de cobertura de Fase 9 y optimización de escala SQLite (Commit `0b4ff8d`)
- **Fase / Subtarea:** Revisión técnica integral — Bloque Cobertura Fase 9 (`F9-01..F9-04`) y Escalabilidad Sintética (Partes IV y V).
- **Qué se hizo:**
  1. **Hallazgo `F9-01` (`.pex` dentro de `.bsa` de mods activos):** En `ProjectService.scan_and_extract()` se ajustó el filtro de modlists masivos para omitir únicamente los `.pex` empaquetados en `.bsa` cuando `fe.origin_mod == "STOCK GAME (Data)"`, extrayendo todos los `.pex` ganadores de mods activos.
  2. **Hallazgo `F9-02` (`FileFormat.JSON_OTHER` Subgrupo 4A vs 4B):** Se implementó `parse_narrative_json_other()` (y el alias `parse_json_localization_bytes`) en `src/nolvus_translator/formats/aux_parsers.py` con lista blanca para JSON narrativos/subtítulos/customskills (Subgrupo 4A) y lista negra estricta para configuraciones técnicas OAR/DAR/Racemenu/BodySlide/SRD/SPID/KID (Subgrupo 4B), conectándolo a `ProjectService.scan_and_extract()`.
  3. **Hallazgo `F9-03` (Exportadores no destructivos para formatos auxiliares):** Se implementaron `build_mcm_helper_json_bytes()` en `mcm_parser.py` y `build_localization_ini_bytes()`, `build_selective_xml_bytes()` y `build_narrative_json_bytes()` en `aux_parsers.py`, conectándolos a `Mo2ModPackager.build_export_plan()`.
  4. **Hallazgo `F9-04` (Normalización focalizada de codificación en `3DNPC.esp`):** Se implementó `normalize_targeted_plugin_encoding()` en `strings_table.py` y se integró en `EspPluginParser` (`esp_parser.py`) para corregir de forma focalizada y reversible las secuencias CP1252/mojibake de `3DNPC.esp` sin reemplazos globales indiscriminados.
  5. **Optimización de escala SQLite (`M-03`):** Se implementaron `TranslationMemoryEngine.store_approved_batch()` y `load_own_es419_index()`. La prueba sintética de escala (2.500 cadenas, 2 lotes de 250 ítems + propagación a 1.000 filas aprobadas) excedía el umbral de 15 s (`15,68 s`) antes de la optimización; tras ella, la suite completa de 27 pruebas se ejecuta en `3,746 s`.
- **Archivos afectados (repositorio de código):**
  - `src/nolvus_translator/formats/strings_table.py`
  - `src/nolvus_translator/formats/esp_parser.py`
  - `src/nolvus_translator/formats/mcm_parser.py`
  - `src/nolvus_translator/formats/aux_parsers.py`
  - `src/nolvus_translator/services/project_service.py`
  - `src/nolvus_translator/export/mo2_packager.py`
  - `src/nolvus_translator/memory/tm_engine.py`
  - `src/nolvus_translator/translation/pipeline.py`
  - `src/nolvus_translator/translation/exchange_store.py`
  - `src/nolvus_translator/quality/gatekeeper.py`
  - `tests/test_phase9_coverage_and_scale.py` (nuevo)
- **Commit:** `0b4ff8d` en rama `feat/phase10-1b-antigravity-exchange`.
- **Pruebas ejecutadas:** `python -m unittest discover -s tests -t . -v` (**27/27 pruebas en verde en `3.746s`**).
- **Resultado:** `COMPLETADO`.

---

## [2026-11-03] — Fase 10.1-B.1: Implementación de infraestructura de intercambio supervisado con Antigravity

- **Fase / Subtarea:** Fase 10.1-B.1 (Implementación en código de las puertas B1–B7).
- **Qué se hizo:**
  - Se creó el commit de línea base `fdea297` en `master` con los 57 archivos previos (`v0.6.0`).
  - Se creó la rama `feat/phase10-1b-antigravity-exchange` y se implementaron el contrato JSON/JSONL `1.0.0` (`exchange_contract.py`), el almacén SQLite aislado con bóveda de placeholders, cuarentena y ledger de idempotencia (`exchange_store.py`), el `LocalGatekeeper` de 12 puertas (`gatekeeper.py`), el coordinador `AntigravityExchangeService` (`exchange_service.py`) y la CLI sin cabecera (`cli.py`).
- **Archivos afectados:** 13 archivos en el repositorio de código (9 nuevos, 4 modificados).
- **Commit:** `8ff5bf78b7e09e344c291bcdddc9f5a8d8b85515` (sobre base `fdea29750e81c9f71322a7488e2e39e6fbd80707`).
- **Pruebas ejecutadas:** `python -m unittest discover -s tests -t . -v` (`23/23 OK` en `2.656s`).
- **Resultado:** `COMPLETADO`.

---

## [2026-11-03] — Fases 10.1-A y 10.1-B: Auditoría estática del código, reconciliación y especificación

- **Fase / Subtarea:** Fases 10.1-A y 10.1-B (Solo lectura y diseño técnico).
- **Qué se hizo:**
  - Auditoría estática exhaustiva de los 57 archivos de `Nolvus AI Translator` (`PHASE10_1_CODE_AUDIT_REPORT.md` y anexos CSV/JSON).
  - Reconciliación de métricas y conteos (`AUDIT_10_1A_RECONCILIATION.md`).
  - Especificación técnica de implementación para el intercambio con Google Antigravity (`PHASE10_1B_IMPLEMENTATION_SPEC.md`).
- **Commits (repositorio documental):** `4db3018` (publicación de la auditoría 10.1-A), `be28524`…`6fdf81b` (reconciliación de contadores), `1ea457e` (especificación 10.1-B), fusionados en `main` mediante los PR #4 y #5.
- **Resultado:** `COMPLETADO`.

---

## [2026-11-03] — Fase 10.0: Rediseño arquitectónico para Google Antigravity

- **Fase / Subtarea:** Fase 10.0 (Arquitectura y plan de benchmark sin API externa).
- **Qué se hizo:**
  - Diseño de la arquitectura de intercambio supervisado por archivos para evaluar `Gemini 4 Argon`, `Claude Sonnet 5.5` y `Claude Opus 5.5` en Google Antigravity sin llamadas HTTP externas ni consumo de créditos API, manteniendo la vía API bloqueada como contingencia futura.
- **Archivos generados:** `PHASE10_FINAL_REPORT.md`, `ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md`, `BENCHMARK_PLAN_ANTIGRAVITY.md`, `DUAL_PROVIDER_ARCHITECTURE.md`, `TRANSLATION_EXECUTION_ENGINE.md`, `SCALABILITY_AND_RISK_ASSESSMENT.md`.
- **Resultado:** `COMPLETADO`.

---

## [2026-11-03] — Fases 7, 8 y 9: Escaneo real de Nolvus Awakening 6.0.21, estimación económica y auditoría integral de mods

- **Fase / Subtarea:** Fases 7, 8 y 9 (Auditorías de solo lectura sobre la instancia real `Nolvus Awakening 6.0.21`).
- **Qué se hizo:**
  - **Fase 7 (`AUDIT_PHASE7.md`):** Primer escaneo real de solo lectura de la instalación Nolvus Awakening 6.0.21.
  - **Fase 8 (`API_COST_ESTIMATE.md`, `API_COST_ESTIMATE_V2.md`):** Estimación económica multi-proveedor (posteriormente archivada como contingencia tras priorizar Antigravity).
  - **Fase 9 (`PHASE9_FULL_AUDIT_REPORT.md`):** Auditoría integral del corte Fase 9 (`3.700` mods activos, `3.717` plugins activos, `583` BSA, `1.168.337` elementos inventariados), estableciendo **328.485 ocurrencias traducibles** y **240.244 cadenas únicas (Corte C)** más **~440 cadenas únicas en JSON narrativos (Subgrupo 4A)**.
- **Resultado:** `COMPLETADO`.

---

## [2026-11-03] — Fases 0 a 6: Construcción de la plataforma base `Nolvus AI Translator v0.6.0`

- **Fase / Subtarea:** Fases 0 a 6 (Arquitectura base, VFS MO2, parsers binarios/textuales, clasificador, protector de variables, Glosario Maestro ES-419, Memoria de Traducción, validador QA, empaquetador MO2 y GUI de 10 pestañas).
- **Qué se hizo:**
  - Implementación completa de los 57 archivos base del proyecto con bloqueo duro de APIs de pago (`PHASE_1_PAID_API_LOCK = True`) y `MockTranslationProvider` para validación estructural sin costo.
- **Commit de referencia:** `fdea297` (`chore: baseline v0.6.0 source tree before phase 10.1-b`).
- **Pruebas ejecutadas:** `11/11 OK` (`tests/test_end_to_end.py`, `tests/test_formats.py`, `tests/test_protector_glossary_tm.py`).
- **Resultado:** `COMPLETADO`.
