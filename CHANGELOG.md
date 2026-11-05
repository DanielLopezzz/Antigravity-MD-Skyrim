# Registro Cronológico de Cambios y Auditorías (`CHANGELOG.md`)

**Proyecto:** Nolvus AI Translator (ES-419)  
**Propósito:** Historial cronológico verificable de las fases ejecutadas, auditorías, especificaciones, implementaciones de código, commits y resultados de pruebas.  
**Nota de organización:** Este documento registra únicamente trabajo ya ejecutado y verificado. Para el estado actual consolidado consulte [PROJECT_STATUS.md](./PROJECT_STATUS.md); para las fases pendientes consulte [ROADMAP.md](./ROADMAP.md).

---

## [2026-11-04] — Barrido y finalización de la cuarta prueba

- Protección de controles reales, reutilización verificable, recuperación focalizada y aceptación atómica. Se corrigieron falsos positivos de metadatos y concordancia, versionado contextual y cardinalidad/duplicados de salida.
- Muestra completa procesada: 975 candidatas importadas pendientes y 25 miembros retenidos. Consumo de todos los intentos `US$0.1176624`, dentro de `US$0.17`; cuatro reparaciones pagadas. Una propuesta separada de formato IA sigue pendiente de revisión.
- Suite 109/109 OK antes de la API y 110/110 OK tras reforzar el esquema SHA-256 offline; 494 archivos anteriores idénticos, cero TM, aprobaciones humanas automáticas o promociones. Código y corpus privados locales; documentación agregada preparada sin push. [Informe](./Fase10_Evolucion_Linguistica/Pilot4_Completed_20261104/README.md).

## [2026-11-04] — Corrección porcentual y repetición R2

- Protector corregido y comprobado sobre las 1.000 entradas; formatos printf legítimos conservados y 14 falsos positivos eliminados. Suite antes de R2: 95/95 OK.
- R2: misma muestra y glosario, 2 solicitudes reales, 50 respuestas, 25 importadas y un lote de 25 retenido por 1 alias duplicado (24 candidatas válidas conservadas). La respuesta con porcentajes confirma la corrección; 950 cadenas no enviadas.
- Consumo R2 `US$0.00439750`; acumulado `US$0.00888310` dentro del tope total de `US$0.17`. Las dos reservas están conciliadas. Sin reintentos, Sol ni promociones.
- Tras R2 se corrigió el indicador de conciliación para distinguir rechazo de QA e incertidumbre económica; evidencia original preservada y fuente usada archivada. Suite final **96/96 OK, 26,860 s**; 345 archivos anteriores protegidos idénticos. [Informe](./Fase10_Evolucion_Linguistica/Pilot4_R2_20261104/README.md).

## [2026-11-04] — Cuarta prueba: primera etapa real y detención editorial

- Se aplicaron las tres adjudicaciones humanas autorizadas; 43 referencias anteriores aprobadas y tiempo presente narrativo conservado.
- Dos solicitudes reales, 50 cadenas recibidas y aceptadas mecánicamente; 10.866 tokens de entrada, 6.798 de salida y coste conciliado de **US$0,00448560**. Ninguna promoción productiva o a TM.
- La revisión detectó un defecto local de protección de porcentajes: 14 cadenas afectadas, 1 enviada y 13 sin enviar. No se ejecutaron las otras 950; se preservaron entradas y respuestas y se anotaron 13 casos para revisión obligatoria, incluidos diagnósticos automáticos y editoriales de IA.
- Candado restaurado y 55 archivos anteriores protegidos idénticos. [Informe y siguiente paso](./Fase10_Evolucion_Linguistica/Pilot4_Stage1_20261104/README.md).

## [2026-11-04] — Preparación de la cuarta prueba y revisión del visor

- Muestra de 1.000 cadenas (100 controles + 900 adicionales), 130 mods, 43 lotes y tres etapas; preflight offline correcto. Cero solicitudes reales y credenciales cargadas; ejecución pendiente.
- Se excluyeron literales técnicos PEX tras inspeccionar una primera selección, conservada como preparación descartada. Los tres casos conocidos de `Elder Scroll` y `HeelsFix Refresh` están en la muestra definitiva.
- Se implementaron runner por etapas, retención de respuestas antes de analizarlas, bloqueo de reenvíos inciertos y conciliación desde el libro presupuestario. El visor incorpora otras pruebas, recarga, prioridades, bloqueo e historial de guardado y compatibilidad con decisiones anteriores.
- Validación final: **92/92 pruebas, 14,736 s**; 55 archivos preexistentes protegidos idénticos. Revisiones del usuario sin reescritura y código solo local. [Informe agregado y limitaciones](./Fase10_Evolucion_Linguistica/Pilot4_Preparation_20261104/README.md).

## [2026-11-04] — Evolución integral y excelencia lingüística offline

- **Implementación local:** contrato 1.1 compatible con 1.0, contexto narrativo en payload y reintentos, perfiles compartidos, variantes explícitas, preservación de identidades técnicas, TM v2 contextual y separación de candidatas/aprobación editorial/exportabilidad.
- **Infraestructura:** límites zlib/LZ4, comparación de tamaños, huellas SHA-256 completas versionadas e índice/cursor SQLite. Se preparó y probó una primitiva reversible de publicación de directorios; su integración en el exportador activo permanece pendiente.
- **Investigación y glosario:** registro de fuentes y condiciones, revisión de 109 entradas sin modificarlas en producción; propuestas separadas y rectificación de dudas no demostradas sobre Soul Cairn/Khajiit. UESP/Imperial Library no se recopilaron por acceso o condiciones sin verificar.
- **Evaluación:** 24 ejemplos sintéticos y 19 casos privados del piloto preparados; 0 referencias humanas nuevas y ninguna comparación de modelos. No se atribuye mejora lingüística medida a los prompts.
- **Evidencia medida:** línea base 69 pruebas; suite final **80/80 OK en 22,496 s**. Benchmark de 240.684 filas sintéticas: 27,506 s y 39.157.760 bytes de pico de proceso, integridad SQLite OK y reanudación verificada.
- **Aislamiento:** 57 archivos protegidos idénticos en hash/tamaño/mtime; candado activo, 0 llamadas a proveedores, 0 créditos, sin alterar pilotos ni producción.
- **Git:** rama local `codex/linguistic-evolution`, HEAD `066ae03`, sin commit ni push de esta fase. [Paquete de informes](./Fase10_Evolucion_Linguistica/Evolution_20261104/README.md).

## [2026-11-04] — Auditoría integral de código complementaria posterior a 10.3-C

- **Alcance:** revisión estática de 65 módulos Python propios, flujos de escaneo, parsers, glosario, TM, contexto, proveedores, presupuesto, staging, Gatekeeper, GUI y exportación. No se escaneó una instalación real ni se lanzaron pilotos.
- **Ajustes locales:** validación de rutas protegidas antes de crear/escribir; protección del `term_id` y de todos los campos de términos aprobados; preservación en la GUI de metadatos del glosario no expuestos; validación de rutas del empaquetador; rechazo de tablas STRINGS/DLSTRINGS/ILSTRINGS truncadas.
- **Evaluación:** el sistema es apto para staging offline y pilotos pequeños supervisados, pero no está demostrado para procesar y exportar como aprobadas las ~240.684 cadenas. Persisten riesgos semánticos, de contexto, identidad TM, memoria de parsers y escala; el detalle y prioridades están en el [informe integral](./Fase10_1_Code_Audit/Audit_20261104_Integral_Post_Pilot/INFORME_AUDITORIA_INTEGRAL.md).
- **Pruebas:** línea base `63/63 OK`; suite posterior `69/69 OK` en `11.752s`; `0` llamadas a proveedores y `0` créditos consumidos.
- **Aislamiento:** candado `PHASE_1_PAID_API_LOCK=True`; DB principal sin cambios de tamaño/mtime; no se ejecutaron runners contra las DB de staging ni se alteraron sus resultados.
- **Estado de código:** rama `feat/phase10-2-openai-api-first`, base observada `066ae03`; trabajo ya modificado antes de iniciar y ajustes de esta auditoría permanecen locales, sin commit ni push. No se cambiaron archivos fuente en este repositorio documental.

## [2026-11-04] — Fase 10.3-C: Piloto transversal de 125 cadenas — aceptación parcial

- **Fase / Subtarea:** Ejecución real de cinco lotes de 25 cadenas con `gpt-6-luna`, más revisión lingüística transversal de los candidatos.
- **Qué se hizo y resultado medido:**
  - Se enviaron `5` solicitudes reales, sin reintentos, para `125` cadenas. Cuatro lotes (100 cadenas) superaron Gatekeeper y se importaron al staging aislado.
  - El lote de scripts recibió 25 traducciones; 23 pasaron validación individual y 2 generaron diagnóstico de incumplimiento del glosario para `Elder Scroll` (`Pergamino Antiguo`). Por la aceptación atómica, las 25 quedaron sin importar: 23 candidatas conservadas y 2 en cuarentena. El artefacto reconstruido no retuvo el texto exacto de las dos traducciones rechazadas.
  - Tokens medidos: `24.890` entrada + `17.712` salida = `42.602`. Coste conciliado de esta ejecución: **`US$0.01134500`**, frente a reserva planificada de `US$0.01833539` y tope del trabajo de `US$0.05`.
  - El reporte registra API lock restaurado y base productiva intacta. No se hicieron reintentos ni nuevas solicitudes tras la cuarentena.
  - El análisis transversal identificó un nombre de mod traducido (`HeelsFix Refresh`) y una forma extensa para revisar (`Nightingale Armor Full Set` → `Conjunto completo de armadura de Ruiseñor`). Esas alertas requieren revisión contextual; no equivalen por sí mismas a errores confirmados.
- **Informe:** [PHASE10_3C_LIVE_PILOT_125_REPORT.md](./Fase10_Antigravity_Architecture/PHASE10_3C_LIVE_PILOT_125_REPORT.md).
- **Estado:** `COMPLETADO CON ACEPTACIÓN PARCIAL`; producción no autorizada.

---

## [2026-11-03] — Fase Correctiva Pre-Piloto Offline 10.3-B (`D-CRED-01..03`, `D-BUD-01`, `D-DOC-01`, `D-ISO-01..02` · Commit `10795fb`)

- **Fase / Subtarea:** Fase Correctiva Offline basada en `PHASE10_3B_READONLY_PREFLIGHT_AUDIT_REPORT.md` — Resolución de todas las discrepancias bloqueantes y documentales previas a la autorización de la primera solicitud real a OpenAI.
- **Qué se hizo:**
  1. **`D-CRED-01` — Unificación de credenciales (`openai_credentials.py` + documentación):**
     - Consolidación de `NOLVUS_OPENAI_API_KEY` y `NOLVUS_OPENAI_PROJECT_ID` (más `NOLVUS_OPENAI_ORG_ID` opcional) como nombres canónicos en código y documentación.
     - Resolución segura de `NOLVUS_OPENAI_PROJECT_ID` desde variable de entorno, desde `external_credential_file` (formatos JSON estructurado o `.env` `KEY=VALUE`) o desde `OpenAIProjectSecurityProfile.default_project_id`, manteniendo `CredentialRedactor` activo.
  2. **`D-CRED-02` — Transporte HTTP real (`RealHttpOpenAITransport` en `openai_provider.py`):**
     - Eliminación del stub `raise ExternalAPIDisabledError("OpenAI Real HTTP Transport")` post-candado e implementación completa de `send_request()` exclusivamente hacia `POST https://api.openai.com/v1/responses`, exigiendo `assert_billable_execution_authorized()`, `assert_translation_only_payload()` (`tools=[]`, `tool_choice="none"`, `store=False`), restricción de modelo `allowed_model_id="gpt-6-luna"` y `SecureCredentialLoader.load_for_authorized_billable_call(...)`, con `urlopen_fn` inyectable para pruebas 100 % offline.
  3. **`D-CRED-03` — Orden de autorización y reserva (`BaseOpenAIModelAdapter.execute_batch` en `openai_provider.py`):**
     - Creación de la reserva atómica `reservation = self.budget.reserve_budget(...)` antes de exigir `active_reservation` en `assert_billable_execution_authorized(...)`, transmisión de `active_reservation=reservation` a `self.transport.send_request(...)`, marcación `mark_reservation_sent()` solo tras superar las validaciones previas al envío y liberación automática mediante `release_unexecuted_reservation()` ante cualquier error previo al envío (`0` reservas huérfanas).
  4. **`D-BUD-01` — Inicialización del presupuesto en el almacén aislado (`pilot_offline_staging.py`):**
     - Invocación explícita de `budget_controller.initialize_project_and_job_budget(job_id="job_pilot_10_3a_luna25")` en `pilot_10_3a_budget_store.sqlite3` (`budget_projects = $0.25 USD ACTIVE`, `budget_jobs = $0.05 USD ACTIVE`, `reserved_usd = 0.0`, `consumed_usd = 0.0`), sin modificar `workspace_data/config.json` ni `workspace_data/nolvus_translator.db`.
  5. **`D-DOC-01`, `D-ISO-01` y `D-ISO-02` — Unificación documental y de hash de manifiesto:**
     - Eliminación de la referencia inexistente a `temperature` en `PHASE10_3A_OFFLINE_PILOT_PREPARATION_REPORT.md` (`D-DOC-01`), actualización del recuento de glosario sembrado a `108` términos (`D-ISO-01`) y unificación de `manifest_sha256` en `openai_provider.py` usando `serialize_manifest_bytes(manifest)` (`f036d564398542aca64bc66b7dcffd0a606d758cf3df9dc717cf6129a284a4e7` idéntico en SQLite y disco, `D-ISO-02`).
- **Archivos afectados (repositorio de código):**
  - `src/nolvus_translator/translation/openai_credentials.py`
  - `src/nolvus_translator/translation/openai_provider.py`
  - `src/nolvus_translator/translation/pilot_offline_staging.py`
  - `tests/test_pilot_offline_staging.py`
- **Pruebas ejecutadas:** `python -m unittest discover -s tests -t . -v` (**58/58 pruebas en verde en `13.618s`**, incluyendo las 10 pruebas obligatorias de remediación preflight; `0` llamadas HTTP reales, `$0.00 USD` consumidos).
- **Resultado:** `COMPLETADO`.

---

## [2026-11-03] — Fase 10.3-A: Preparación Offline del Primer Piloto `gpt-6-luna` de 25 cadenas reales (Commit `d2b75a5`)

- **Fase / Subtarea:** Fase 10.3-A — Preparación técnica 100 % offline del primer lote piloto real de 25 cadenas (`5 × 5`) del corpus de Nolvus Awakening 6.0.21 para `gpt-6-luna` (`REQUEST_CONSTRUCTION`).
- **Qué se hizo:**
  1. **Extracción estratificada de solo lectura (`pilot_estimator.py`, `pilot_offline_staging.py`):**
     - Lectura inmutable (`mode=ro&immutable=1`) de `workspace_data/nolvus_translator.db` seleccionando `5` cadenas por cada una de las `5` categorías funcionales (`dialogues_and_quests`: `259` chars, `mcm_and_ui`: `216` chars, `items_spells_perks`: `181` chars, `books_and_notes`: `639` chars, `scripts_and_other`: `243` chars = `25` cadenas, **`1.538` caracteres totales de origen en `source_chars`** y **`1.384` caracteres enmascarados en `source_masked`**, `0` duplicados), priorizando objetivos con `<Alias=...>`, llaves posicionales `{}` de MCM Helper, tokens mágicos `<dur>`/`<mag>`, etiquetas HTML de libros y mensajes de scripts `.pex`.
  2. **Protección mejorada de placeholders y bóveda local (`protector.py`, `exchange_store.py`):**
     - Actualización del patrón `brace_placeholder` en `VariableProtector` (`r"\{[A-Za-z0-9_.:%\-]*\}"`) para proteger llaves posicionales vacías `{}` en **MCM Helper** junto con `{0}`/`{name}`.
     - Enmascaramiento de **37 placeholders** en `11/25` cadenas (`html_xml_tag`: 23, `escaped_control`: 7, `brace_placeholder`: 3, `skyrim_alias_global`: 2, `skyrim_magic_token`: 2), guardados en `isolated_dbs/pilot_10_3a_exchange_store.sqlite3` (`25` sobres en `placeholder_vault`).
  3. **Construcción offline del payload de Responses API (`openai_provider.py`, `pilot_offline_staging.py`):**
     - Inyección de instrucciones ES-419 y del subconjunto aplicable del Glosario Maestro (`Health -> Salud`, `Nocturnal -> Nocturnal`, `Soul Gem -> Gema de alma`).
     - Generación del payload `staging_payloads/batch_batch_pilot_10_3a_001_openai_responses_request.json` para `/v1/responses` con `model="gpt-6-luna"`, `tools=[]`, `tool_choice="none"`, `store=False`, `max_output_tokens=2198` y validación recursiva de hojas en `assert_translation_only_payload()`.
  4. **Cálculo de tokens y coste en escenarios esperado y máximo (Standard vs. Batch `-50 %`) (`budget_control.py`):**
     - `1.989` tokens de entrada estimados, `1.758` tokens de salida en escenario esperado y `2.198` tokens de salida en escenario máximo (`max_output_tokens`):
       - **Tarifa Standard (`$0.10/$0.50` / 1M):** escenario esperado **`$0.001078 USD`** base (`$0.001240 USD` con `+15 %`); escenario máximo **`$0.001298 USD`** base (**`$0.001493 USD`** reserva oficial con `+15 %`).
       - **Tarifa Batch (`-50 %`, `$0.05/$0.25` / 1M):** escenario esperado **`$0.000539 USD`** base (**`$0.000620 USD`** con `+15 %`); escenario máximo **`$0.000649 USD`** base (**`$0.000746 USD`** con `+15 %`).
       - Ambos escenarios quedan muy por debajo del presupuesto de referencia de `$0.05 USD` por trabajo (`$0.00 USD` reservados y `$0.00 USD` consumidos).
  5. **Verificación de paso obligatorio por `LocalGatekeeper` (`pilot_offline_staging.py`, `test_pilot_offline_staging.py`):**
     - Demostración automatizada sobre un clon efímero de la copia aislada de que (a) la importación directa sin Gatekeeper es bloqueada (`ExchangeStateTransitionError`), (b) un placeholder corrupto es rechazado en `GATE_09_PLACEHOLDER_RESTORATION` (`0` filas en TM) y (c) un lote válido supera las 12 puertas, restaura los 37 placeholders e incorpora las 25 cadenas a `translation_memory`.
- **Archivos afectados (repositorio de código):**
  - `.gitignore`
  - `src/nolvus_translator/quality/protector.py`
  - `src/nolvus_translator/reports/pilot_estimator.py`
  - `src/nolvus_translator/translation/budget_control.py`
  - `src/nolvus_translator/translation/openai_provider.py`
  - `src/nolvus_translator/translation/pilot_offline_staging.py` (nuevo)
  - `tests/test_pilot_offline_staging.py` (nuevo)
- **Commit:** `d2b75a5` en rama `feat/phase10-2-openai-api-first`.
- **Pruebas ejecutadas:** `python -m unittest discover -s tests -t . -v` (**49/49 pruebas en verde en `5.682s`**, `0` llamadas HTTP reales, `$0.00 USD` consumidos).
- **Resultado:** `COMPLETADO`.

---

## [2026-11-03] — Fase 10.2: Implementación de infraestructura OpenAI API-First, Coordinador Batch y Controlador Local de Presupuesto (Commit `2766b73`)

- **Fase / Subtarea:** Fase 10.2 — Infraestructura OpenAI API-First, seguridad de credenciales, enrutador Luna/Sol/Argon, coordinador Batch, control económico local y estimación offline del primer piloto.
- **Qué se hizo:**
  1. **Cambio arquitectónico a OpenAI API-First (`model_router.py`, `config.py`, `future_stubs.py`):**
     - Configuración de `gpt-6-luna` (`GPT-6 Luna`) como proveedor principal de traducción masiva (`openai_luna`), `gpt-6.1-sol` (`GPT-6.1 Sol`) como proveedor de escalamiento selectivo bajo reglas verificables (`EscalationRuleCode`) y permiso explícito (`allow_sol_escalation`), y `gemini-4-argon` (`Antigravity Argon`) como vía complementaria por archivos.
     - Exclusión explícita de `Claude` (`claude-sonnet-5.5`, `claude-opus-5.5`, `claude*`) del flujo principal y del enrutamiento automático.
     - Prohibición en código de enviar una misma cadena a Luna y Sol simultáneamente y de usar Sol como fallback ante errores técnicos/HTTP de Luna.
  2. **Regla «Las API solo traducen» (`openai_provider.py`, `openai_credentials.py`):**
     - Imposición obligatoria de `tools=[]`, `tool_choice="none"` y `store=False` en todas las solicitudes OpenAI (`assert_translation_only_payload`), más inspección recursiva que bloquea cualquier filtración de rutas locales (`C:\...`, `STOCK GAME`, `ModOrganizer.ini`, `.db`).
  3. **Integración de Responses API (`/v1/responses`) y Gatekeeper (`openai_provider.py`):**
     - Implementación de `GPT6LunaAdapter`, `GPT61SolAdapter`, `OpenAIResponsesRequest` (con esquema JSON estricto `nolvus_es419_translation_batch`), `BlockedNetworkTransport`, `RealHttpOpenAITransport` y `MockOpenAITransport`, conectados de extremo a extremo con `VariableProtector`, `PlaceholderVault`, `ExchangeStateStore` y `LocalGatekeeper` (12 puertas deterministas).
  4. **Coordinador de OpenAI Batch API (`openai_batch_coordinator.py`):**
     - Construcción de archivos `.jsonl` para `/v1/files` y `/v1/batches` estrictamente separados por modelo (`gpt-6-luna` vs `gpt-6.1-sol`), correlación biyectiva `custom_id`, reserva presupuestaria previa por línea, persistencia SQLite (`openai_batch_jobs`, `openai_batch_items`) para recuperación tras cierre de la aplicación (`list_recoverable_batch_jobs`), cancelación con liberación de presupuesto (`cancel_batch_job`), reconciliación de errores parciales/duplicados/faltantes y limpieza remota (`cleanup_remote_files`).
  5. **Controlador Local de Presupuesto (`budget_control.py`):**
     - Implementación de `BudgetController` sobre SQLite con transacciones `BEGIN IMMEDIATE` y `threading.RLock`, cálculo previo de tokens (fuente + instrucciones + glosario + esquema JSON), tope físico `max_output_tokens`, reserva atómica sobre tarifa Standard (`$0.10/$0.50` por 1M para Luna; `$2.00/$10.00` por 1M para Sol) más margen de seguridad del `+15 %` (`1.15x`), reconciliación con `usage` real, detención inmediata ante sobrecosto (`HALTED_OVERRUN`) o falta de `usage` (`HALTED_UNKNOWN_COST`), liberación de reservas no enviadas y recuperación tras caída (`recover_after_interruption`).
  6. **Estimador offline del primer piloto (`pilot_estimator.py`):**
     - Extracción en modo solo lectura inmutable (`mode=ro&immutable=1`) de muestras reales estratificadas de `workspace_data/nolvus_translator.db` (`25`, `100` y `250` cadenas en 5 categorías) sin tocar la base de datos (`0` bytes alterados) ni enviar un solo byte a OpenAI.
- **Archivos afectados (repositorio de código):**
  - `src/nolvus_translator/core/exceptions.py`
  - `src/nolvus_translator/core/config.py`
  - `src/nolvus_translator/translation/future_stubs.py`
  - `src/nolvus_translator/translation/budget_control.py` (nuevo)
  - `src/nolvus_translator/translation/openai_credentials.py` (nuevo)
  - `src/nolvus_translator/translation/model_router.py` (nuevo)
  - `src/nolvus_translator/translation/openai_provider.py` (nuevo)
  - `src/nolvus_translator/translation/openai_batch_coordinator.py` (nuevo)
  - `src/nolvus_translator/reports/pilot_estimator.py` (nuevo)
  - `tests/test_openai_budget_control.py` (nuevo)
  - `tests/test_openai_provider_and_router.py` (nuevo)
  - `tests/test_openai_batch_coordinator.py` (nuevo)
  - `tests/test_openai_security_and_isolation.py` (nuevo)
- **Commit:** `2766b73` en rama `feat/phase10-2-openai-api-first`.
- **Pruebas ejecutadas:** `python -m unittest discover -s tests -t . -v` (**48/48 pruebas en verde en `5.482s`**, `0` llamadas HTTP reales, `$0.00 USD` consumidos).
- **Resultado:** `COMPLETADO`.

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
  5. **Optimización de escala SQLite (`M-03`):** Se implementaron `TranslationMemoryEngine.store_approved_batch()` y `load_own_es419_index()`. La prueba sintética de escala (2.500 cadenas, 2 lotes de 250 ítems + propagación a 1.000 filas aprobadas) excedía el umbral de 15 s (`15,68 s`) antes de la optimización; tras ella, la suite completa de 27 pruebas se ejecutó en `3,746 s`.
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
  - Diseño de la arquitectura de intercambio supervisado por archivos para evaluar modelos en Google Antigravity sin llamadas HTTP externas ni consumo de créditos API, manteniendo la vía API bloqueada como contingencia futura (posteriormente evolucionada a API-first en la Fase 10.2).
- **Archivos generados:** `PHASE10_FINAL_REPORT.md`, `ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md`, `BENCHMARK_PLAN_ANTIGRAVITY.md`, `DUAL_PROVIDER_ARCHITECTURE.md`, `TRANSLATION_EXECUTION_ENGINE.md`, `SCALABILITY_AND_RISK_ASSESSMENT.md`.
- **Resultado:** `COMPLETADO`.

---

## [2026-11-03] — Fases 7, 8 y 9: Escaneo real de Nolvus Awakening 6.0.21, estimación económica y auditoría integral de mods

- **Fase / Subtarea:** Fases 7, 8 y 9 (Auditorías de solo lectura sobre la instancia real `Nolvus Awakening 6.0.21`).
- **Qué se hizo:**
  - **Fase 7 (`AUDIT_PHASE7.md`):** Primer escaneo real de solo lectura de la instalación Nolvus Awakening 6.0.21.
  - **Fase 8 (`API_COST_ESTIMATE.md`, `API_COST_ESTIMATE_V2.md`):** Estimación económica multi-proveedor.
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
