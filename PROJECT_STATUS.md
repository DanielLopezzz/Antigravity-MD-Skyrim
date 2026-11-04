# Estado Oficial del Proyecto (`PROJECT_STATUS.md`) — Fuente Única de Verdad

**Proyecto:** Nolvus AI Translator  
**Objetivo:** Traducción propia de Nolvus Awakening 6.0.21 al español latinoamericano neutro (`ES-419`)  
**Última actualización:** 2026-11-04  
**Fase actual:** **Fase 10.3-C completada con aceptación parcial** — 100/125 cadenas importadas al staging aislado; 23 candidatas retenidas por atomicidad y 2 en cuarentena. Siguiente puerta: remediación dirigida 10.3-D.  
**Estrategia de proveedores vigente:** **API-First** (`gpt-6-luna` principal, `gpt-6.1-sol` escalamiento selectivo por regla verificable, `gemini-4-argon` vía complemento de intercambio de archivos; `Claude` excluido)  
**Rama activa (repositorio de código):** `feat/phase10-2-openai-api-first` (`HEAD` en `10795fb`; rama previa `feat/phase10-1b-antigravity-exchange` en `0b4ff8d`; rama base `master` en `fdea297`)  
**Estado de suite de pruebas:** **63/63 métodos en verde en la última ejecución registrada anterior a los ajustes locales posteriores al piloto; pendiente repetir verificación tras esos ajustes**  
**Candado de API de pago:** `PHASE_1_PAID_API_LOCK = True` restaurado tras el piloto; producción continúa bloqueada  
**Traducciones reales en staging de prueba:** **100 aprobadas e importadas + 23 candidatas no importadas + 2 en cuarentena**; coste de la Fase 10.3-C: **`US$0.01134500`**. Traducciones promovidas a producción: `0`.

---

## 1. Recuento histórico y estado global por componente y fase (Fases 0 a 10.3-A)

Clasificación canónica aplicada:
- `COMPLETADO`: Implementado, verificado con evidencia automatizada/auditable y cerrado.
- `IMPLEMENTADO, PENDIENTE DE VALIDACIÓN`: Implementado y probado con datos sintéticos/unitarios, pendiente de ejecución sobre lote real de Nolvus en la fase correspondiente.
- `EN CURSO`: Actividad actualmente abierta.
- `PENDIENTE`: Etapa futura aún no iniciada.
- `BLOQUEADO`: Deshabilitado intencionalmente por candado técnico o política del proyecto hasta autorización expresa.
- `APLAZADO`: Conservado como vía futura o complementaria sin activación masiva actual.
- `DESCARTADO`: Excluido definitivamente por política del proyecto.

### 1.1 Estado de los 16 subsistemas principales del proyecto

| # | Subsistema / Actividad | Fase origen | Estado oficial | Evidencia técnica en código y pruebas |
|---:|---|---|---|---|
| 1 | **Detección del entorno, MO2 y VFS** (`detector.py`, `vfs.py`) | Fases 0–1 | `COMPLETADO` | Resolución de `ModOrganizer.ini`, prioridad invertida de `modlist.txt`, orden de `plugins.txt`/`loadorder.txt`, conflicto de archivos sueltos vs `.bsa` y verificación de solo lectura verificados en `test_end_to_end.py` y sobre la instalación real de Nolvus Awakening 6.0.21 (Fase 7 y Fase 9). |
| 2 | **Parsers de formatos Bethesda y auxiliares** (`esp_parser.py`, `strings_table.py`, `mcm_parser.py`, `pex_parser.py`, `bsa_archive.py`, `aux_parsers.py`) | Fases 1–2 y Consolidación Post-10.1-B.1 | `COMPLETADO` | Parsers nativos para `.esp/.esm/.esl` (incluyendo compresión zlib, `XXXX`, `GMST s*`, `PERK EPFT==7`), `.strings/.dlstrings/.ilstrings`, `MCM .txt` UTF-16LE, `MCM Helper config.json`, `.pex` (AST seguro), `.bsa` (v104 zlib / v105 LZ4), `.psc` contextual, `*_DESC.ini`/`*_DISTR.ini`/`SkyPatcher/*.ini`, XML selectivo y `JSON_OTHER` narrativo (Subgrupo 4A vs 4B). Verificados en `test_formats.py` y `test_phase9_coverage_and_scale.py`. |
| 3 | **Clasificación de cadenas y protección de variables** (`classifier.py`, `protector.py`) | Fases 2–3, Consolidación Post-10.1-B.1 y Fase 10.3-A | `COMPLETADO` | Clasificación en 9 estados (`ClassificationStatus`) y enmascaramiento determinista `⟦VAR_0⟧..⟦VAR_n⟧` de 10 familias de patrones técnicos (`<Alias=...>`, `<mag>`, `<font...>`, `[pagebreak]`, `%.1f`, `{}`/`{0}`/`{name}`, `$MCMKey`, `0x...`, `FormID\|Plugin`, `\n`) más detección de inyección cruda fuera de placeholders. Verificado en `test_protector_glossary_tm.py`, `test_gatekeeper.py` y `test_pilot_offline_staging.py`. |
| 4 | **Glosario Maestro y Memoria de Traducción (TM)** (`glossary.py`, `seed_data.py`, `tm_engine.py`, `context_engine.py`) | Fases 3–4 y Consolidación Post-10.1-B.1 | `COMPLETADO` | Semilla oficial ES-419, protección contra sobrescritura accidental (`GlossaryConflictError`), caché de regex por límite de palabra, índice en memoria `load_own_es419_index()`, escritura masiva en transacción única `store_approved_batch()`, historial de versiones `tm_versions`, búsqueda difusa `RapidFuzz + FTS5` y bloqueo estricto de traducciones comunitarias (`community` rechazado). Verificado en `test_protector_glossary_tm.py`. |
| 5 | **Validación QA y reglas ES-419** (`validator.py`) | Fase 4 y Consolidación Post-10.1-B.1 | `COMPLETADO` | Rúbrica determinista y auditable: integridad de variables/etiquetas, codificación destino, cumplimiento del Glosario Maestro con tolerancia morfológica plural (`s`/`es`), ratio de longitud, puntuación interrogativa/exclamativa y rechazo de voseo/formas peninsulares (`vosotros`, `habéis`, `tenéis`, `podéis`, `coged`, etc.). Verificado en `test_protector_glossary_tm.py` y `test_gatekeeper.py`. |
| 6 | **Exportación a MO2 y reversibilidad** (`mo2_packager.py`, `dsd_exporter.py`) | Fase 5 y Consolidación Post-10.1-B.1 | `COMPLETADO` | Generación de mod independiente para MO2 con vista previa en memoria, backup previo automático, manifiesto `translation_manifest.json` con hashes SHA-256, reversión completa (`rollback_export`) y constructores no destructivos para `DSD JSON`, `.STRINGS/.DLSTRINGS/.ILSTRINGS`, `MCM _SPANISH.txt` (UTF-16LE BOM), `MCM Helper JSON`, `.pex` (AST seguro), `INI_LOC`, `XML` selectivo y `JSON_OTHER` (Subgrupo 4A). Verificado en `test_end_to_end.py` y `test_phase9_coverage_and_scale.py`. |
| 7 | **Interfaz gráfica de escritorio (10 pestañas)** (`ui/app.py`, `ui/theme.py`) | Fase 6 y Consolidación Post-10.1-B.1 | `COMPLETADO` | Aplicación Tkinter de 10 pestañas con tema oscuro nórdico, ejecución en hilo de fondo (`ThreadPoolExecutor`), preservación de `AppConfig` al guardar y cierre limpio de recursos (`WM_DELETE_WINDOW`). Verificada en `test_end_to_end.py`. |
| 8 | **Escaneo real de Nolvus Awakening 6.0.21** | Fase 7 | `COMPLETADO` | Primer escaneo real de solo lectura (`AUDIT_PHASE7.md`, `coverage_snapshot_1.md`); sus métricas son históricas y fueron ampliadas y reconciliadas en la Fase 9. |
| 9 | **Estimación económica API (histórica Fase 8)** | Fase 8 | `COMPLETADO` | Informe económico histórico `API_COST_ESTIMATE_V2.md` completado y complementado en la Fase 10.2 por el módulo en vivo `BudgetController` y el estimador offline `pilot_estimator.py`. |
| 10 | **Auditoría integral de mods y cobertura** | Fase 9 y Consolidación Post-10.1-B.1 | `COMPLETADO` | Auditoría exhaustiva del corte Fase 9 (`3.700` mods activos, `3.717` plugins activos, `583` BSA activos, `1.168.337` elementos inventariados; `PHASE9_FULL_AUDIT_REPORT.md`). Sus 4 hallazgos técnicos (`F9-01` a `F9-04`) quedaron implementados y probados en `test_phase9_coverage_and_scale.py`. |
| 11 | **Arquitectura de integración con Google Antigravity** | Fase 10.0 | `COMPLETADO` | Diseño de intercambio supervisado por archivos documentado en `ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md` (ahora ruta complementaria para `Antigravity Argon`). |
| 12 | **Auditoría estática del código y reconciliación** | Fase 10.1-A | `COMPLETADO` | Auditoría de los 57 archivos base (`PHASE10_1_CODE_AUDIT_REPORT.md`) y reconciliación (`AUDIT_10_1A_RECONCILIATION.md`) completadas. |
| 13 | **Especificación técnica de Fase 10.1-B** | Fase 10.1-B | `COMPLETADO` | Especificación de contratos, aislamiento, bóveda, Gatekeeper de 12 puertas, recuperación y CLI publicada en `PHASE10_1B_IMPLEMENTATION_SPEC.md`. |
| 14 | **Infraestructura de intercambio Antigravity y Gatekeeper de 12 puertas** (`exchange_contract.py`, `exchange_store.py`, `gatekeeper.py`, `exchange_service.py`, `cli.py`) | Fase 10.1-B.1 | `COMPLETADO` | Implementado y consolidado (commits `8ff5bf7`, `844aa1b`, `0b4ff8d`). Conservado íntegro como núcleo de validación local (`LocalGatekeeper`) para las respuestas de OpenAI y como ruta complementaria de intercambio con `Antigravity Argon`. |
| 15 | **Infraestructura OpenAI API-First, Coordinador Batch y Control Presupuestario Local** (`openai_provider.py`, `openai_batch_coordinator.py`, `budget_control.py`, `model_router.py`, `openai_credentials.py`, `pilot_estimator.py`) | Fase 10.2 y Remediación Preflight 10.3-B | `COMPLETADO` | Implementado en la rama `feat/phase10-2-openai-api-first`. Incluye `GPT6LunaAdapter`, `GPT61SolAdapter`, `RealHttpOpenAITransport` (restringido a `POST https://api.openai.com/v1/responses` y `gpt-6-luna`), `OpenAIBatchCoordinator`, `BudgetController` con reserva atómica SQLite (`BEGIN IMMEDIATE` + margen `1.15x` sobre tarifa Standard), `ModelRouter`, aislamiento `tools=[]`/`tool_choice="none"`, resolución canónica de `NOLVUS_OPENAI_API_KEY` y `NOLVUS_OPENAI_PROJECT_ID` y liberación automática de reservas ante cualquier error previo al envío. Permanece con candado `PHASE_1_PAID_API_LOCK = True` hasta autorización expresa. |
| 16 | **Preparación Offline del Primer Piloto `gpt-6-luna` (25 cadenas reales 5×5) y Remediación Pre-Piloto** (`pilot_offline_staging.py`, `test_pilot_offline_staging.py`) | Fase 10.3-A / 10.3-B Preflight | `COMPLETADO` | Implementado y verificado en modo 100 % offline (**58/58 pruebas en verde**). Extrae 25 cadenas reales (`5 × 5`, `1.538` caracteres de origen / `1.384` enmascarados), protege `37` placeholders en bóveda local (`PlaceholderVault`), siembra los `108` términos del Glosario Maestro ES-419, construye el payload `/v1/responses` (`batch_batch_pilot_10_3a_001_openai_responses_request.json`, `max_output_tokens = 2198`, coste máximo Standard con `+15 %`: **`$0.001493 USD`**), inicializa los presupuestos de proyecto (`$0.25 USD`) y trabajo (`$0.05 USD`) en `pilot_10_3a_budget_store.sqlite3` y verifica en clon efímero el paso obligatorio por `LocalGatekeeper`. |
| 17 | **Pilotos reales 10.3-B/10.3-C y análisis lingüístico transversal** | Fases 10.3-B / 10.3-C | `COMPLETADO CON ACEPTACIÓN PARCIAL` | Informe 10.3-B conserva los dos intentos de 25 cadenas. Piloto 10.3-C: 5 lotes, 125 cadenas, 123 respuestas recibidas, 100 importadas en staging, 23 candidatas retenidas y 2 cuarentenadas por glosario; coste real `US$0.01134500`. Ver `PHASE10_3C_LIVE_PILOT_125_REPORT.md`. |

### 1.2 Elementos bloqueados, complementarios, aplazados o descartados por política

| Elemento | Estado oficial | Motivo |
|---|---|---|
| **Más ejecuciones facturables y producción (`gpt-6-luna`, `gpt-6.1-sol`)** | `BLOQUEADO` hasta la siguiente autorización | Ya hubo pilotos reales autorizados 10.3-B/10.3-C. El candado vuelve a estar activo; la fase 10.3-D requiere remediación y aprobación específica antes de nuevas solicitudes. La producción sigue sin traducciones promovidas. |
| **Escalamiento a `GPT-6.1 Sol` (`gpt-6.1-sol`)** | `BLOQUEADO` por defecto (`allow_sol_escalation=False`) | Solo interviene cuando una regla verificable (`EscalationRuleCode`) lo justifica y el usuario habilita explícitamente el escalamiento. Prohibido usar Sol como fallback ante errores técnicos/HTTP de Luna. |
| **Ruta `Antigravity Argon` (`gemini-4-argon`)** | `COMPLEMENTARIO` | Conservada como ruta complementaria y alternativa mediante el intercambio supervisado por archivos (`AntigravityExchangeService`). |
| **Modelos `Claude` (`Claude Sonnet 5.5`, `Claude Opus 5.5`, `claude*`)** | `DESCARTADO` | Excluidos del flujo principal de traducción y del enrutamiento automático (`ModelRouter` y `OpenAIProjectSecurityProfile` bloquean cualquier identificador `claude*`). |
| **Reutilización de traducciones comunitarias (`*_spanish.txt`, mods traducidos por terceros)** | `DESCARTADO` | Prohibido por la política de traducción propia ES-419 (`TRANSLATION_POLICY_ES419.md` y bloqueo en `TranslationMemoryEngine`). |
| **Inferencia local en la RX 9070 XT** | `APLAZADO` | Pausada por decisión de Fase 10; no se instalan herramientas ni se descargan modelos. |
| **Edición binaria directa de `.swf` (Flash Scaleform compilado)** | `DESCARTADO` | Inventariado explícitamente como `SWF_UNSUPPORTED` para preservar la estabilidad binaria de la interfaz; los textos externalizados de UI se traducen vía `Interface/Translations/*.txt`, `MCM Helper JSON` y `GMST`. |

---

## 2. Estado de cobertura de traducción sobre el corpus de Nolvus Awakening 6.0.21

| Métrica del corpus auditado (Fase 9 + consolidación de cobertura) | Valor | Estado de traducción real |
|---|---:|---|
| Ocurrencias traducibles totales identificadas (Corte C) | `328.485` | `0` traducidas (`PENDIENTE` de inicio en Fase 10.3) |
| Cadenas únicas netas pendientes de traducir (Corte C base) | `240.244` | `0` traducidas (`PENDIENTE`) |
| Cadenas únicas adicionales en archivos JSON narrativos (Subgrupo 4A) | `~440` | Parser y exportador `COMPLETADO` (`F9-02`/`F9-03`); `0` traducidas aún |
| **Total estimado de cadenas únicas traducibles (Corte C + Subgrupo 4A)** | **`~240.684`** | **Producción: `0,00 %` promovido. Pilotos aislados 10.3-B/10.3-C documentados; sus cifras no alteran el estado del corpus productivo.** |
| Cadenas con codificación corrupta identificadas en `3DNPC.esp` | `38` | Normalización focalizada implementada en código (`F9-04`: `normalize_targeted_plugin_encoding`) |
| Scripts `.pex` ganadores dentro de `.bsa` (corte Fase 9) | `38.406` (`25.365` de mods; `3.090` con texto visible = `14.036` ocurrencias) | El escaneo productivo del código ahora solo omite los `.pex` en `.bsa` de `STOCK GAME (Data)` (`F9-01`) |
| Formatos con extracción + exportación no destructiva en código | 9 familias | `DSD JSON`, `.STRINGS/.DLSTRINGS/.ILSTRINGS`, `MCM .txt`, `MCM Helper JSON`, `.pex`, `INI_LOC`, `XML` selectivo, `JSON_OTHER` (4A). Verificados con datos sintéticos |

---

## 3. Estado técnico actual del repositorio de código

### 3.1 Ramas y commits verificados

| Rama | Commit | Descripción |
|---|---|---|
| `master` | `fdea297` | Línea base `v0.6.0` previa a Fase 10.1-B (`57` archivos rastreados, `11/11` pruebas en verde). |
| `feat/phase10-1b-antigravity-exchange` | `8ff5bf7` | Implementación funcional de Fase 10.1-B.1 (`exchange_contract`, `exchange_store`, `gatekeeper`, `exchange_service`, `cli` y 12 pruebas nuevas). |
| `feat/phase10-1b-antigravity-exchange` | `844aa1b` | Corrección de concurrencia/atomicidad en `ExchangeStateStore`/`LocalGatekeeper`, reconciliación de reintentos multi-salto (`retry1 -> retry2`), deduplicación de lotes consecutivos, detección de inyección cruda en `VariableProtector` y mejoras en `ui/app.py`. |
| `feat/phase10-1b-antigravity-exchange` | `0b4ff8d` | Implementación de los 4 hallazgos de cobertura de Fase 9 (`F9-01` a `F9-04`), optimización masiva SQLite en `TranslationMemoryEngine`/`TranslationPipeline`/`ExchangeStateStore` y suite de pruebas de escala (`test_phase9_coverage_and_scale.py`). |
| `feat/phase10-2-openai-api-first` | `2766b73` | **Implementación completa de Fase 10.2 (Infraestructura OpenAI API-First):** `budget_control.py`, `openai_credentials.py`, `model_router.py`, `openai_provider.py`, `openai_batch_coordinator.py`, `pilot_estimator.py` y 4 nuevas suites de pruebas (`21` métodos nuevos -> **48/48 en verde**). |
| `feat/phase10-2-openai-api-first` | `780b643` | Sincronización de los 5 archivos documentales canónicos de la raíz tras la Fase 10.2. |
| `feat/phase10-2-openai-api-first` | `d2b75a5` | **Implementación y ejecución offline de Fase 10.3-A (Preparación del primer piloto `gpt-6-luna` de 25 cadenas reales):** `pilot_offline_staging.py`, mejora de `protector.py` para llaves vacías `{}` de MCM Helper, mejora de `pilot_estimator.py` y `openai_provider.py`, y suite `test_pilot_offline_staging.py` (**49/49 pruebas en verde**). |
| `feat/phase10-2-openai-api-first` | `10795fb` | **Remediación offline de las discrepancias preflight (`D-CRED-01..03`, `D-BUD-01`, `D-DOC-01`, `D-ISO-01..02`):** implementación de `RealHttpOpenAITransport` (`POST https://api.openai.com/v1/responses`), corrección del orden de reserva y autorización en `execute_batch()`, inicialización presupuestaria en `pilot_10_3a_budget_store.sqlite3`, unificación de `manifest_sha256` y ampliación de `test_pilot_offline_staging.py` con las 10 pruebas obligatorias (**58/58 pruebas en verde**). |

### 3.2 Resumen del esquema SQLite real (reconciliado)

1. **Base de datos principal del traductor (`src/nolvus_translator/core/database.py`):**
   - `SCHEMA_SQL` declara **12 objetos DDL explícitos**: **11 tablas regulares** (`schema_meta`, `scan_snapshots`, `scanned_mods`, `scanned_files`, `string_entries`, `glossary_terms`, `translation_memory`, `tm_versions`, `translation_jobs`, `translation_batches`, `export_history`) + **1 tabla virtual FTS5** (`tm_fts`), que generan **17 entradas en `sqlite_master WHERE type='table'`**.
2. **Almacén aislado de intercambio y lotes (`src/nolvus_translator/translation/exchange_store.py`):**
   - Opera en un archivo SQLite físicamente separado y define **8 tablas regulares**: `exchange_meta`, `exchange_jobs`, `exchange_batches`, `exchange_items`, `placeholder_vault`, `quarantine_items`, `ingestion_ledger`, `exchange_state_transitions`.
3. **Almacén de control presupuestario y coordinación Batch (`budget_control.py` + `openai_batch_coordinator.py`):**
   - `BudgetController` define **4 tablas regulares** (`budget_projects`, `budget_jobs`, `budget_reservations`, `budget_audit_log`) bajo `BEGIN IMMEDIATE`.
   - `OpenAIBatchCoordinator` define **2 tablas regulares** (`openai_batch_jobs`, `openai_batch_items`) para persistencia y recuperación de trabajos asíncronos `/v1/batches`.

### 3.3 Estado de la suite de pruebas automatizadas (`58/58 OK`)

- **Ejecución completa:** `python -m unittest discover -s tests -t . -v` → **`Ran 58 tests in 13.618s — OK`**.
  - **11 pruebas originales de línea base (`test_end_to_end.py` [2], `test_formats.py` [6], `test_protector_glossary_tm.py` [3]):** `11/11 OK`.
  - **13 pruebas del subsistema de intercambio y Gatekeeper (`test_exchange_contract.py` [3], `test_exchange_store.py` [4], `test_gatekeeper.py` [4], `test_exchange_cli.py` [2]):** `13/13 OK`.
  - **3 pruebas de cobertura Fase 9 y escala sintética (`test_phase9_coverage_and_scale.py` [3]):** `3/3 OK`.
  - **21 pruebas de infraestructura OpenAI API-First, Batch, Presupuesto y Seguridad (`test_openai_budget_control.py` [8], `test_openai_provider_and_router.py` [5], `test_openai_batch_coordinator.py` [3], `test_openai_security_and_isolation.py` [5]):** `21/21 OK` (`0` llamadas de red reales).
  - **10 pruebas de preparación offline del piloto de 25 cadenas Fase 10.3-A y remediación preflight 10.3-B (`test_pilot_offline_staging.py` [10]):** `10/10 OK` (`0` llamadas de red reales).

---

## 4. Lote real preparado en Fase 10.3-A (`batch_pilot_10_3a_001`) y estimación económica

Preparado en modo 100 % offline (`mode=ro&immutable=1`) sobre `workspace_data/nolvus_translator.db` en el entorno aislado `NolvusAIT_PilotStaging/Phase10_3A_Luna25/` (`25` cadenas `5×5`, **`1.538` caracteres totales de origen** en `source_chars` y **`1.384` caracteres enmascarados** en `source_masked`, `37` placeholders protegidos en `11/25` cadenas):

### 4.1 Desglose de escenarios del lote `batch_pilot_10_3a_001` (`25` cadenas) — Tarifa Standard vs. Batch (`-50 %`)

| Modalidad y escenario (`batch_pilot_10_3a_001`) | Tokens entrada | Tokens salida | Coste base (`USD`) | Reserva con margen `+15 %` (`× 1.15`, `USD`) |
|---|---:|---:|---:|---:|
| **Standard (`/v1/responses`) — Escenario esperado** | `1.989` | `1.758` | `$0.001078 USD` (`$0.00107790`) | **`$0.001240 USD`** (`$0.00123959`) |
| **Standard (`/v1/responses`) — Escenario máximo (`max_output_tokens`)** | `1.989` | `2.198` | **`$0.001298 USD`** (`$0.00129790`) | **`$0.001493 USD`** (`$0.00149259`) *(reserva oficial `BudgetController`)* |
| **Batch (`-50 %`, `/v1/batches`) — Escenario esperado** | `1.989` | `1.758` | `$0.000539 USD` (`$0.00053895`) | **`$0.000620 USD`** (`$0.00061979` / `$0.00061980`) |
| **Batch (`-50 %`, `/v1/batches`) — Escenario máximo (`max_output_tokens`)** | `1.989` | `2.198` | `$0.000649 USD` (`$0.00064895`) | **`$0.000746 USD`** (`$0.00074629`) |

### 4.2 Escalones comparativos del piloto (`25`, `100` y `250` cadenas)

| Lote / Escalón | Cadenas / Lotes | Caracteres origen / Placeholders | Tokens entrada est. | Tokens salida (esp. / máx.) | Coste máx. Standard (`2.198` out) / Reserva `+15 %` | Coste Batch `-50 %` esp. (`+15 %`) / máx. (`+15 %`) | Presupuesto de referencia (`max_job_budget_usd`) |
|---|---:|---:|---:|---:|---:|---:|---:|
| **`batch_pilot_10_3a_001` (Preparado en Fase 10.3-A)** | **`25` (`5×5`)** | **`1.538` chars / `37` ph** | **`1.989`** | **`1.758` / `2.198`** | **`$0.001298` / `$0.001493 USD`** | `$0.000539` (`$0.000620`) / `$0.000649` (`$0.000746 USD`) | **`$0.05 USD`** (no autorizado aún) |
| **Escalón 2 (Ampliación a 100 cadenas)** | `100` (`4×25`) | — | `7.711` | `6.829` / `8.539` | `$0.005041` / **`$0.005797 USD`** | `$0.002093` (`$0.002407 USD`) / — | **`$0.05 USD`** |
| **Escalón 3 (Ampliación a 250 cadenas)** | `250` (`10×25`) | — | `20.573` | `18.781` / `23.480` | `$0.013797` / **`$0.015867 USD`** | `$0.005724` (`$0.006583 USD`) / — | **`$0.05 USD`** |

---

## 5. Próxima puerta de aprobación

- **Siguiente paso:** **Fase 10.3-D — Remediar los dos hallazgos del lote de scripts y revisar traducciones candidatas** en staging aislado, conservando evidencia y sin reejecutar los lotes ya aprobados.
- **Requisitos técnicos:** reconciliar `Elder Scroll` con su contexto y validación; proteger `HeelsFix Refresh` sin impedir traducir la descripción; mantener aceptación atómica hasta analizar dependencias; revisar manualmente las candidatas por contexto, registro y legibilidad.
- **Puerta facturable:** cualquier nueva solicitud requiere reserva presupuestaria y autorización expresa. Antes de ampliar, reprueba dirigida de hasta 25 cadenas del dominio de scripts; la producción continúa no autorizada.
