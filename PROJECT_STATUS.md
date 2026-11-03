# Estado Oficial del Proyecto (`PROJECT_STATUS.md`) — Fuente Única de Verdad

**Proyecto:** Nolvus AI Translator  
**Objetivo:** Traducción propia de Nolvus Awakening 6.0.21 al español latinoamericano neutro (`ES-419`)  
**Última actualización:** 2026-11-03  
**Fase actual:** Consolidación técnica posterior a Fase 10.1-B.1 completada — Listo para puerta de autorización de **Fase 10.2 (Smoke test aislado de 25 cadenas)**  
**Rama activa (repositorio de código):** `feat/phase10-1b-antigravity-exchange` (`HEAD` en `0b4ff8d`; rama base `master` en `fdea297`)  
**Estado de suite de pruebas:** **27/27 métodos de prueba en verde (`OK` en `3.746s`)**  
**Candado de API de pago:** `PHASE_1_PAID_API_LOCK = True` (activo)  
**Traducciones reales ejecutadas sobre corpus de Nolvus:** **`0` cadenas** (pendiente de inicio autorizado en Fase 10.2)

---

## 1. Recuento histórico y estado global por componente y fase (Fases 0 a 10.1-B.1)

Clasificación canónica aplicada:
- `COMPLETADO`: Implementado, verificado con evidencia automatizada/auditable y cerrado.
- `IMPLEMENTADO, PENDIENTE DE VALIDACIÓN`: Implementado y probado con datos sintéticos/unitarios, pendiente de ejecución sobre lote real de Nolvus en la fase correspondiente.
- `EN CURSO`: Actividad actualmente abierta.
- `PENDIENTE`: Etapa futura aún no iniciada.
- `BLOQUEADO`: Deshabilitado intencionalmente por candado técnico o política del proyecto.
- `APLAZADO`: Conservado como vía futura de contingencia sin activación actual.
- `DESCARTADO`: Excluido definitivamente por política del proyecto.

### 1.1 Estado de los 14 subsistemas principales del proyecto

| # | Subsistema / Actividad | Fase origen | Estado oficial | Evidencia técnica en código y pruebas |
|---:|---|---|---|---|
| 1 | **Detección del entorno, MO2 y VFS** (`detector.py`, `vfs.py`) | Fases 0–1 | `COMPLETADO` | Resolución de `ModOrganizer.ini`, prioridad invertida de `modlist.txt`, orden de `plugins.txt`/`loadorder.txt`, conflicto de archivos sueltos vs `.bsa` y verificación de solo lectura verificados en `test_end_to_end.py` y sobre la instalación real de Nolvus Awakening 6.0.21 (Fase 7 y Fase 9). |
| 2 | **Parsers de formatos Bethesda y auxiliares** (`esp_parser.py`, `strings_table.py`, `mcm_parser.py`, `pex_parser.py`, `bsa_archive.py`, `aux_parsers.py`) | Fases 1–2 y Consolidación Post-10.1-B.1 | `COMPLETADO` | Parsers nativos para `.esp/.esm/.esl` (incluyendo compresión zlib, `XXXX`, `GMST s*`, `PERK EPFT==7`), `.strings/.dlstrings/.ilstrings`, `MCM .txt` UTF-16LE, `MCM Helper config.json`, `.pex` (AST seguro), `.bsa` (v104 zlib / v105 LZ4), `.psc` contextual, `*_DESC.ini`/`*_DISTR.ini`/`SkyPatcher/*.ini`, XML selectivo y `JSON_OTHER` narrativo (Subgrupo 4A vs 4B). Verificados en `test_formats.py` y `test_phase9_coverage_and_scale.py`. |
| 3 | **Clasificación de cadenas y protección de variables** (`classifier.py`, `protector.py`) | Fases 2–3 y Consolidación Post-10.1-B.1 | `COMPLETADO` | Clasificación en 9 estados (`ClassificationStatus`) y enmascaramiento determinista `⟦VAR_0⟧..⟦VAR_n⟧` de 10 familias de patrones técnicos (`<Alias=...>`, `<mag>`, `<font...>`, `[pagebreak]`, `%.1f`, `{0}`, `$MCMKey`, `0x...`, `FormID\|Plugin`, `\n`) más detección de inyección cruda fuera de placeholders. Verificado en `test_protector_glossary_tm.py` y `test_gatekeeper.py`. |
| 4 | **Glosario Maestro y Memoria de Traducción (TM)** (`glossary.py`, `seed_data.py`, `tm_engine.py`, `context_engine.py`) | Fases 3–4 y Consolidación Post-10.1-B.1 | `COMPLETADO` | Semilla oficial ES-419, protección contra sobrescritura accidental (`GlossaryConflictError`), caché de regex por límite de palabra, índice en memoria `load_own_es419_index()`, escritura masiva en transacción única `store_approved_batch()`, historial de versiones `tm_versions`, búsqueda difusa `RapidFuzz + FTS5` y bloqueo estricto de traducciones comunitarias (`community` rechazado). Verificado en `test_protector_glossary_tm.py`. |
| 5 | **Validación QA y reglas ES-419** (`validator.py`) | Fase 4 y Consolidación Post-10.1-B.1 | `COMPLETADO` | Rúbrica determinista y auditable: integridad de variables/etiquetas, codificación destino, cumplimiento del Glosario Maestro con tolerancia morfológica plural (`s`/`es`), ratio de longitud, puntuación interrogativa/exclamativa y rechazo de voseo/formas peninsulares (`vosotros`, `habéis`, `tenéis`, `podéis`, `coged`, etc.). Verificado en `test_protector_glossary_tm.py` y `test_gatekeeper.py`. |
| 6 | **Exportación a MO2 y reversibilidad** (`mo2_packager.py`, `dsd_exporter.py`) | Fase 5 y Consolidación Post-10.1-B.1 | `COMPLETADO` | Generación de mod independiente para MO2 con vista previa en memoria, backup previo automático, manifiesto `translation_manifest.json` con hashes SHA-256, reversión completa (`rollback_export`) y constructores no destructivos para `DSD JSON`, `.STRINGS/.DLSTRINGS/.ILSTRINGS`, `MCM _SPANISH.txt` (UTF-16LE BOM), `MCM Helper JSON`, `.pex` (AST seguro), `INI_LOC`, `XML` selectivo y `JSON_OTHER` (Subgrupo 4A). Verificado en `test_end_to_end.py` y `test_phase9_coverage_and_scale.py`. |
| 7 | **Interfaz gráfica de escritorio (10 pestañas)** (`ui/app.py`, `ui/theme.py`) | Fase 6 y Consolidación Post-10.1-B.1 | `COMPLETADO` | Aplicación Tkinter de 10 pestañas con tema oscuro nórdico, ejecución en hilo de fondo (`ThreadPoolExecutor`), preservación de `AppConfig` al guardar y cierre limpio de recursos (`WM_DELETE_WINDOW`). Verificada en `test_end_to_end.py`. |
| 8 | **Escaneo real de Nolvus Awakening 6.0.21** | Fase 7 | `COMPLETADO` | Primer escaneo real de solo lectura (`AUDIT_PHASE7.md`, `coverage_snapshot_1.md`); sus métricas son históricas y fueron ampliadas y reconciliadas en la Fase 9. |
| 9 | **Estimación económica API (contingencia)** | Fase 8 | `APLAZADO` | Informe económico `API_COST_ESTIMATE_V2.md` completado y auditado. La integración de APIs de pago queda **aplazada como contingencia futura** y bloqueada en código (`PHASE_1_PAID_API_LOCK = True`). |
| 10 | **Auditoría integral de mods y cobertura** | Fase 9 y Consolidación Post-10.1-B.1 | `COMPLETADO` | Auditoría exhaustiva del corte Fase 9 (`3.700` mods activos, `3.717` plugins activos, `583` BSA activos, `1.168.337` elementos inventariados; `PHASE9_FULL_AUDIT_REPORT.md`). Sus 4 hallazgos técnicos de cobertura en el código (`F9-01` `.pex` en `.bsa` de mods, `F9-02` `JSON_OTHER` Subgrupo 4A vs 4B, `F9-03` exportadores auxiliares no destructivos, `F9-04` normalización focalizada de codificación en `3DNPC.esp`) quedaron implementados y probados con datos sintéticos en `test_phase9_coverage_and_scale.py`. |
| 11 | **Arquitectura de integración con Google Antigravity** | Fase 10.0 | `COMPLETADO` | Diseño de intercambio supervisado por archivos fuera de `BaseTranslationProvider` documentado en `ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md`. |
| 12 | **Auditoría estática del código y reconciliación** | Fase 10.1-A | `COMPLETADO` | Auditoría de los 57 archivos base (`PHASE10_1_CODE_AUDIT_REPORT.md`) y reconciliación (`AUDIT_10_1A_RECONCILIATION.md`) completadas. |
| 13 | **Especificación técnica de Fase 10.1-B** | Fase 10.1-B | `COMPLETADO` | Especificación de contratos, aislamiento, bóveda, Gatekeeper de 12 puertas, recuperación y CLI publicada en `PHASE10_1B_IMPLEMENTATION_SPEC.md`. |
| 14 | **Implementación y consolidación de Fase 10.1-B.1** (`exchange_contract.py`, `exchange_store.py`, `gatekeeper.py`, `exchange_service.py`, `cli.py`) | Fase 10.1-B.1 | `COMPLETADO` | Implementado y consolidado en la rama `feat/phase10-1b-antigravity-exchange` (commits `8ff5bf7`, `844aa1b`, `0b4ff8d`). Puertas `B1` a `B7` superadas con **27/27 pruebas en verde**. Pendiente únicamente de validación operativa con lote real en la Fase 10.2. |

### 1.2 Elementos bloqueados, aplazados o descartados por política

| Elemento | Estado oficial | Motivo |
|---|---|---|
| **Proveedores API de pago externos** | `BLOQUEADO` / `APLAZADO` | Bloqueados en código por `PHASE_1_PAID_API_LOCK = True`. Conservados únicamente como vía arquitectónica de contingencia si Antigravity no supera las puertas de Fase 10. |
| **Modelos `GPT-6 Luna` y `GPT-6 Sol` en Antigravity** | `DESCARTADO` | Confirmados por el usuario como ausentes en su selector real de Google Antigravity. Solo se consideran `Gemini 4 Argon`, `Claude Sonnet 5.5` y `Claude Opus 5.5`. |
| **Reutilización de traducciones comunitarias (`*_spanish.txt`, mods traducidos por terceros)** | `DESCARTADO` | Prohibido por la política de traducción propia ES-419 (`TRANSLATION_POLICY_ES419.md` y bloqueo en `TranslationMemoryEngine`). |
| **Inferencia local en la RX 9070 XT** | `APLAZADO` | Pausada por decisión de Fase 10; no se instalan herramientas ni se descargan modelos. |
| **Edición binaria directa de `.swf` (Flash Scaleform compilado)** | `DESCARTADO` | Inventariado explícitamente como `SWF_UNSUPPORTED` para preservar la estabilidad binaria de la interfaz; los textos externalizados de UI se traducen vía `Interface/Translations/*.txt`, `MCM Helper JSON` y `GMST`. |

---

## 2. Estado de cobertura de traducción sobre el corpus de Nolvus Awakening 6.0.21

| Métrica del corpus auditado (Fase 9 + consolidación de cobertura) | Valor | Estado de traducción real |
|---|---:|---|
| Ocurrencias traducibles totales identificadas (Corte C) | `328.485` | `0` traducidas (`PENDIENTE` de inicio en Fase 10.2) |
| Cadenas únicas netas pendientes de traducir (Corte C base) | `240.244` | `0` traducidas (`PENDIENTE`) |
| Cadenas únicas adicionales en archivos JSON narrativos (Subgrupo 4A) | `~440` | Parser y exportador `COMPLETADO` (`F9-02`/`F9-03`); `0` traducidas aún |
| **Total estimado de cadenas únicas traducibles (Corte C + Subgrupo 4A)** | **`~240.684`** | **`0,00 %` traducido (ninguna traducción real ejecutada aún)** |
| Cadenas con codificación corrupta identificadas en `3DNPC.esp` | `38` | Normalización focalizada implementada en código (`F9-04`: `normalize_targeted_plugin_encoding`) |
| Scripts `.pex` ganadores dentro de `.bsa` (corte Fase 9) | `38.406` (`25.365` de mods; `3.090` con texto visible = `14.036` ocurrencias) | El escaneo productivo del código omitía los `.pex` en `.bsa` en modlists grandes; ahora solo omite los de `STOCK GAME (Data)` (`F9-01`) |
| Formatos con extracción + exportación no destructiva en código | 9 familias | `DSD JSON`, `.STRINGS/.DLSTRINGS/.ILSTRINGS`, `MCM .txt`, `MCM Helper JSON`, `.pex`, `INI_LOC`, `XML` selectivo, `JSON_OTHER` (4A). Verificados con datos sintéticos; pendiente de verificación sobre snapshot real |

---

## 3. Estado técnico actual del repositorio de código

### 3.1 Ramas y commits verificados

| Rama | Commit | Descripción |
|---|---|---|
| `master` | `fdea297` | Línea base `v0.6.0` previa a Fase 10.1-B (`57` archivos rastreados, `11/11` pruebas en verde). |
| `feat/phase10-1b-antigravity-exchange` | `8ff5bf7` | Implementación funcional de Fase 10.1-B.1 (`exchange_contract`, `exchange_store`, `gatekeeper`, `exchange_service`, `cli` y 12 pruebas nuevas). |
| `feat/phase10-1b-antigravity-exchange` | `844aa1b` | Corrección de defectos de concurrencia/atomicidad en `ExchangeStateStore`/`LocalGatekeeper`, reconciliación de reintentos multi-salto (`retry1 -> retry2`), deduplicación de lotes consecutivos, detección de inyección cruda en `VariableProtector` y mejoras en `ui/app.py`. |
| `feat/phase10-1b-antigravity-exchange` | `0b4ff8d` | Implementación de los 4 hallazgos de cobertura de Fase 9 (`F9-01` a `F9-04`), optimización masiva SQLite en `TranslationMemoryEngine`/`TranslationPipeline`/`ExchangeStateStore` y suite de pruebas de escala (`test_phase9_coverage_and_scale.py`). |

### 3.2 Resumen del esquema SQLite real (reconciliado)

1. **Base de datos principal del traductor (`src/nolvus_translator/core/database.py`):**
   - `SCHEMA_SQL` declara **12 objetos DDL explícitos**: **11 tablas regulares** (`schema_meta`, `scan_snapshots`, `scanned_mods`, `scanned_files`, `string_entries`, `glossary_terms`, `translation_memory`, `tm_versions`, `translation_jobs`, `translation_batches`, `export_history`) + **1 tabla virtual FTS5** (`tm_fts`).
   - Al instanciarse en SQLite, `tm_fts` genera 4 tablas sombra internas (`tm_fts_data`, `tm_fts_idx`, `tm_fts_docsize`, `tm_fts_config`) y `AUTOINCREMENT` genera `sqlite_sequence`, sumando **17 entradas en `sqlite_master WHERE type='table'`**.
2. **Almacén aislado de intercambio (`src/nolvus_translator/translation/exchange_store.py`):**
   - Opera en un archivo SQLite físicamente separado y define **8 tablas regulares**: `exchange_meta`, `exchange_jobs`, `exchange_batches`, `exchange_items`, `placeholder_vault`, `quarantine_items`, `ingestion_ledger`, `exchange_state_transitions`.

### 3.3 Estado de la suite de pruebas automatizadas (`27/27 OK`)

- **Ejecución completa:** `python -m unittest discover -s tests -t . -v` → **`Ran 27 tests in 3.746s — OK`**.
  - **11 pruebas originales de línea base (`tests/test_end_to_end.py` [2], `tests/test_formats.py` [6], `tests/test_protector_glossary_tm.py` [3]):** `11/11 OK` (archivos originales intactos con `0` bytes modificados).
  - **13 pruebas del subsistema de intercambio Antigravity (`tests/test_exchange_contract.py` [3], `tests/test_exchange_store.py` [4], `tests/test_gatekeeper.py` [4], `tests/test_exchange_cli.py` [2]):** `13/13 OK`.
  - **3 pruebas de cobertura Fase 9 y escala sintética (`tests/test_phase9_coverage_and_scale.py` [3]):** `3/3 OK` (incluye exportación, Gatekeeper de 12 puertas e importación con propagación de duplicados sobre un snapshot sintético de **2.500 cadenas**, con umbral de tiempo de 15 s dentro de una suite completa de 3,746 s).

---

## 4. Riesgos abiertos bajo control

1. **Rendimiento real y adherencia al contrato de los modelos de Antigravity (`Gemini 4 Argon`, `Claude Sonnet 5.5`, `Claude Opus 5.5`):**
   - Aunque el `LocalGatekeeper` de 12 puertas y la recuperación por micro-lotes están verificados al 100 % con pruebas sintéticas, la tasa real de aceptación a primera pasada con cadenas reales de Skyrim debe medirse empíricamente en las Fases **10.2** (25 cadenas) y **10.3** (300 cadenas).
2. **Re-escaneo de snapshot tras mejoras de Fase 9 (`F9-01`, `F9-02`, `F9-04`):**
   - Para que las ~440 cadenas de `JSON_OTHER` (Subgrupo 4A), los `.pex` dentro de `.bsa` de mods activos y las 38 cadenas normalizadas de `3DNPC.esp` queden reflejados en un snapshot de trabajo, se deberá generar un snapshot de prueba aislado cuando se autorice la etapa correspondiente (sin tocar la base productiva hasta superar todas las puertas de la Fase 10).

---

## 5. Próxima puerta de aprobación

- **Siguiente paso:** **Fase 10.2 — Preflight y smoke test aislado de 25 cadenas reales** (5 diálogos, 5 libros con etiquetas HTML/`[pagebreak]`, 5 entradas MCM con `$claves`, 5 descripciones de hechizos/perks con `<mag>/<dur>` y 5 cadenas de scripts/misiones con `<Alias=...>`), operando exclusivamente sobre una copia SQLite de prueba y un workspace externo aislado.
- **Requisito para avanzar:** Autorización expresa del usuario tras revisar este estado consolidado.
