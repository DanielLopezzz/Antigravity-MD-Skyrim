# FASE 10.3-B (REMEDIACIÓN PRE-PILOTO OFFLINE) — INFORME DE CORRECCIÓN DE DISCREPANCIAS BLOQUEANTES Y VERIFICACIÓN INTEGRAL

**Proyecto:** Nolvus AI Translator  
**Instancia objetivo:** Nolvus Awakening 6.0.21  
**Idioma de destino:** Español latinoamericano neutro (`ES-419`)  
**Fase:** `10.3-B (Preflight Remediation)` — Fase Correctiva Offline Basada en `PHASE10_3B_READONLY_PREFLIGHT_AUDIT_REPORT.md`  
**Fecha:** 2026-11-03  
**Rama de implementación (repositorio de código):** `feat/phase10-2-openai-api-first` (`HEAD` en `10795fb`)  
**Suite de pruebas automatizadas:** **58/58 pruebas en verde (`OK` en `13,618 s`)** — incluye las 10 pruebas obligatorias de remediación pre-piloto  
**Llamadas HTTP / API realizadas:** **`0`** (`PHASE_1_PAID_API_LOCK = True`, `$0.000000 USD` gastados y `$0.000000 USD` reservados)

---

## 1. Resumen Ejecutivo y Cumplimiento de Reglas Obligatorias

Se ha ejecutado y verificado al 100 % en modalidad **offline** la fase correctiva derivada de `Fase10_Antigravity_Architecture/PHASE10_3B_READONLY_PREFLIGHT_AUDIT_REPORT.md`, solucionando las **4 discrepancias técnicas bloqueantes (`D-CRED-01`, `D-CRED-02`, `D-CRED-03`, `D-BUD-01`)** y las **3 discrepancias documentales/de metadatos (`D-DOC-01`, `D-ISO-01`, `D-ISO-02`)** antes de autorizar la primera solicitud real a OpenAI.

| Regla obligatoria de la fase correctiva | Estado | Evidencia técnica verificada |
|---|---|---|
| No realizar ninguna llamada HTTP ni API | **CUMPLIDO** | `real_http_requests_count = 0` en todas las corridas y pruebas unitarias (`urlopen_fn` inyectado en memoria para pruebas offline). |
| No generar traducciones reales | **CUMPLIDO** | Las `25` cadenas en `pilot_10_3a_work_db.sqlite3` permanecen en `review_status = 'unprocessed'` y `translation_memory = 0`. |
| No desbloquear `PHASE_1_PAID_API_LOCK` | **CUMPLIDO** | `PHASE_1_PAID_API_LOCK = True` en `src/nolvus_translator/core/exceptions.py`. |
| No solicitar, leer, mostrar ni copiar claves API | **CUMPLIDO** | `api_key_loaded_or_requested = False` en staging; pruebas ejecutadas exclusivamente con diccionarios efímeros en memoria y `CredentialRedactor` activo. |
| No modificar Nolvus `STOCK GAME`, mods originales, perfiles MO2 ni partidas | **CUMPLIDO** | Ningún acceso de escritura fuera de `NolvusAIT_PilotStaging/Phase10_3A_Luna25/`. |
| No modificar la base de producción `workspace_data/nolvus_translator.db` | **CUMPLIDO** | Tamaño intacto (`431.386.624` bytes) y `mtime_ns` idéntico (`1793683151811389300`, `source_db_untouched = True`). |
| No modificar `workspace_data/config.json`, glosario ni TM de producción | **CUMPLIDO** | `workspace_data/config.json` intacto (`active_provider = "mock"`). |
| Mantener el trabajo en la rama de desarrollo y en el staging aislado | **CUMPLIDO** | Rama `feat/phase10-2-openai-api-first` y directorio aislado `NolvusAIT_PilotStaging/Phase10_3A_Luna25/`. |

---

## 2. Resolución Detallada de Discrepancias (`D-CRED-01` a `D-ISO-02`)

### 2.1 `D-CRED-01` — Integración y unificación de credenciales (`openai_credentials.py` + documentación)

- **Causa raíz identificada en la auditoría:**
  1. Existía divergencia documental entre el nombre canónico en código (`NOLVUS_OPENAI_API_KEY` y `NOLVUS_OPENAI_PROJECT_ID`) y menciones genéricas a `OPENAI_API_KEY` en `PHASE10_3A_OFFLINE_PILOT_PREPARATION_REPORT.md`.
  2. `SecureCredentialLoader` solo leía una cadena plana desde `external_credential_file`, sin permitir resolver `NOLVUS_OPENAI_PROJECT_ID` (ni `NOLVUS_OPENAI_ORG_ID`) cuando se utiliza un archivo externo de credenciales o un perfil con `default_project_id`.
- **Corrección implementada (`src/nolvus_translator/translation/openai_credentials.py`):**
  - Se definieron y exportaron las constantes canónicas `CANONICAL_OPENAI_API_KEY_ENV = "NOLVUS_OPENAI_API_KEY"`, `CANONICAL_OPENAI_PROJECT_ID_ENV = "NOLVUS_OPENAI_PROJECT_ID"` y `CANONICAL_OPENAI_ORG_ID_ENV = "NOLVUS_OPENAI_ORG_ID"`.
  - Se añadieron `default_project_id: str | None = None` y `default_org_id: str | None = None` a `OpenAIProjectSecurityProfile`.
  - Se implementaron `_read_external_store_fields()`, `_resolve_project_id()` y `_resolve_org_id()` en `SecureCredentialLoader`, soportando resolución de `NOLVUS_OPENAI_API_KEY` y `NOLVUS_OPENAI_PROJECT_ID` tanto desde variables de entorno como desde un almacén externo (formato JSON estructurado, formato `.env` `KEY=VALUE` o línea plana compatible hacia atrás), manteniendo el aislamiento físico fuera de raíces prohibidas y la redacción determinista con `CredentialRedactor`.
  - Se unificó toda la documentación de la Fase 10.3-A/B en torno a `NOLVUS_OPENAI_API_KEY` y `NOLVUS_OPENAI_PROJECT_ID`.

### 2.2 `D-CRED-02` — Implementación de `RealHttpOpenAITransport` (`openai_provider.py`)

- **Causa raíz identificada en la auditoría:**
  - `RealHttpOpenAITransport.send_request()` contenía un stub que lanzaba incondicionalmente `raise ExternalAPIDisabledError("OpenAI Real HTTP Transport")` en la línea 568 incluso después de invocar `assert_billable_execution_authorized()`.
- **Corrección implementada (`src/nolvus_translator/translation/openai_provider.py`):**
  - Se eliminó el stub incondicional post-candado y se implementó el flujo completo de `RealHttpOpenAITransport.send_request()` con **5 barreras sucesivas previas al envío**:
    1. **Triple candado de autorización (`assert_billable_execution_authorized`):** bloquea inmediatamente si `PHASE_1_PAID_API_LOCK` es `True`, si `stage != ExecutionStage.BILLABLE_REAL_EXECUTION`, si `explicit_user_billing_authorization` es `False` o si `active_reservation` es `None`, inactiva o con importe `<= 0`.
    2. **Restricción exclusiva de endpoint y destino HTTPS:** exige `method == "POST"`, `path == "/v1/responses"` (`OPENAI_RESPONSES_ENDPOINT`) y `base_url == "https://api.openai.com"` (`OPENAI_DEFAULT_BASE_URL`). Cualquier otro método, ruta o dominio lanza `OpenAISecurityViolationError`.
    3. **Validación de payload «las API solo traducen» (`assert_translation_only_payload`):** parsea `body_bytes` y verifica `tools=[]`, `tool_choice="none"`, `store=False`, esquema estricto `nolvus_es419_translation_batch`, ausencia de rutas locales y ausencia de secretos.
    4. **Restricción estricta de modelo (`allowed_model_id = "gpt-6-luna"`):** verifica que `validated_req.model == self.allowed_model_id` (`"gpt-6-luna"`) y que coincida con `active_reservation.model_id`; rechaza `gpt-6.1-sol` o cualquier otro modelo con `ModelRoutingError`.
    5. **Carga segura de credenciales (`credential_loader.load_for_authorized_billable_call`):** recupera las cabeceras `Authorization: Bearer ...` y `OpenAI-Project: ...` solo tras superar las barreras anteriores, verifica que las cabeceras del llamador no contengan secretos filtrados y ejecuta la petición `POST https://api.openai.com/v1/responses` mediante `self._urlopen_fn` (inyectable para pruebas 100 % offline), redactando cualquier error de red con `CredentialRedactor`.

### 2.3 `D-CRED-03` — Orden de autorización y reserva en `execute_batch` (`openai_provider.py`)

- **Causa raíz identificada en la auditoría:**
  1. `BaseOpenAIModelAdapter.execute_batch()` llamaba a `assert_billable_execution_authorized(..., active_reservation=None)` antes de crear la reserva atómica con `self.budget.reserve_budget(...)`, provocando un fallo sistemático por `active_reservation=None`.
  2. No pasaba `active_reservation=reservation` a `self.transport.send_request(...)`.
  3. Llamaba a `self.budget.mark_reservation_sent(reservation.reservation_id)` *antes* de `self.transport.send_request(...)`, lo que cambiaba el estado a `SENT_IN_FLIGHT` antes de validar el envío e impedía que `release_unexecuted_reservation()` liberara la reserva si ocurría un fallo previo al envío.
- **Corrección implementada (`src/nolvus_translator/translation/openai_provider.py`):**
  1. `execute_batch()` valida primero que la etapa no sea de solo configuración/construcción/estimación.
  2. Crea la reserva atómica en SQLite (`reservation = self.budget.reserve_budget(...)` en estado `RESERVED_NOT_SENT`) **antes** de exigir `active_reservation`.
  3. Dentro del bloque `try:`, valida `assert_billable_execution_authorized(..., active_reservation=reservation)` e invoca `self.transport.send_request(..., active_reservation=reservation)`.
  4. Marca la reserva como enviada (`self.budget.mark_reservation_sent(reservation.reservation_id)`) **únicamente después** de que `send_request` haya superado todas las validaciones previas y devuelto respuesta.
  5. Si ocurre cualquier fallo previo al envío (candado `PHASE_1_PAID_API_LOCK`, falta de credenciales, modelo no autorizado, error de validación o transporte local), el bloque `except` ejecuta `self.budget.release_unexecuted_reservation(reservation.reservation_id, reason=...)`, dejando `reserved_usd = 0.0` y la reserva en estado `RELEASED_UNEXECUTED` sin reservas huérfanas.

### 2.4 `D-BUD-01` — Inicialización del presupuesto en el almacén aislado de staging (`pilot_offline_staging.py`)

- **Causa raíz identificada en la auditoría:**
  - `prepare_pilot_10_3a_offline()` instanciaba `BudgetController` sobre `pilot_10_3a_budget_store.sqlite3` con `EconomicLimitsConfig(project_budget_usd=0.25, max_job_budget_usd=0.05)`, pero no invocaba `budget_controller.initialize_project_and_job_budget(job_id="job_pilot_10_3a_luna25")`, dejando las tablas `budget_projects` y `budget_jobs` con `0` filas.
- **Corrección implementada (`src/nolvus_translator/translation/pilot_offline_staging.py`):**
  - Se añadió la llamada explícita `budget_controller.initialize_project_and_job_budget(job_id=job_id)` en `prepare_pilot_10_3a_offline()` y la verificación posterior de `get_project_summary()` y `get_job_summary(job_id)`.
  - Se regeneró el staging aislado `NolvusAIT_PilotStaging/Phase10_3A_Luna25/isolated_dbs/pilot_10_3a_budget_store.sqlite3` (`SHA-256: 4bb18a2c2ffc22c5b0510c3dd5084735501368068705b8aa6f6ca1177614c994`), quedando persistidas:
    - `budget_projects`: `project_id = 'nolvus_es419_project'`, `total_budget_usd = 0.25`, `reserved_usd = 0.0`, `consumed_usd = 0.0`, `status = 'ACTIVE'`.
    - `budget_jobs`: `job_id = 'job_pilot_10_3a_luna25'`, `max_job_budget_usd = 0.05`, `reserved_usd = 0.0`, `consumed_usd = 0.0`, `status = 'ACTIVE'`.
  - No se modificó `workspace_data/config.json` ni `workspace_data/nolvus_translator.db`.

### 2.5 `D-DOC-01`, `D-ISO-01` y `D-ISO-02` — Unificación documental y de hash de manifiesto

- **`D-DOC-01` (`PHASE10_3A_OFFLINE_PILOT_PREPARATION_REPORT.md` Sección 4.1):** se eliminó la fila `temperature = 0.1` (parámetro no incluido en `OpenAIResponsesRequest`) y se alinearon las claves documentadas de `metadata` (`job_id`, `batch_id`, `model_id`, `idempotency_key`, `purpose`) con el payload real.
- **`D-ISO-01` (`PHASE10_3A_OFFLINE_PILOT_PREPARATION_REPORT.md` Secciones 1.1, 3.2 y 6):** se actualizó el recuento documentado de términos sembrados en `glossary_terms` de `87` a **`108` términos oficiales ES-419** (cifra real sembrada por `GlossaryManager.ensure_seeded()` desde `seed_data.py`).
- **`D-ISO-02` (`src/nolvus_translator/translation/openai_provider.py` línea 1362):** se unificó el cálculo de `manifest_sha256` en `prepare_batch_request()` utilizando `compute_bytes_sha256(serialize_manifest_bytes(manifest))`. Ahora la columna `exchange_batches.manifest_sha256` en `pilot_10_3a_exchange_store.sqlite3` y el archivo `batch_batch_pilot_10_3a_001_meta.json` en disco poseen exactamente el mismo hash SHA-256 (`f036d564398542aca64bc66b7dcffd0a606d758cf3df9dc717cf6129a284a4e7`).

---

## 3. Resultados de las 10 Pruebas Obligatorias (100 % Offline)

Se amplió `tests/test_pilot_offline_staging.py` y se ejecutó la suite completa del proyecto (`python -m unittest discover -s tests -t . -v`), obteniendo **58/58 pruebas en verde (`OK` en `13,618 s`)**:

| # | Prueba obligatoria solicitada | Método de prueba en `tests/test_pilot_offline_staging.py` | Resultado | Evidencia verificada |
|---:|---|---|---|---|
| **1** | Verificar que el transporte no realiza llamadas si `PHASE_1_PAID_API_LOCK` está activo | `test_02_mandatory_01_transport_makes_zero_calls_when_lock_active` | **PASS (`OK`)** | `PHASE_1_PAID_API_LOCK = True` lanza `ExternalAPIDisabledError`, `real_http_requests_count == 0` y `urlopen_calls == 0`. |
| **2** | Comprobar que no se puede ejecutar sin credenciales | `test_03_mandatory_02_cannot_execute_without_credentials` | **PASS (`OK`)** | Sin `NOLVUS_OPENAI_API_KEY` o sin `NOLVUS_OPENAI_PROJECT_ID` lanza `OpenAISecurityViolationError` con `0` llamadas de red; verifica también resolución desde almacén externo JSON (`D-CRED-01`). |
| **3** | Comprobar que no se puede ejecutar sin una reserva válida | `test_04_mandatory_03_cannot_execute_without_valid_reservation` | **PASS (`OK`)** | Con `active_reservation=None`, `RELEASED_UNEXECUTED` o `reserved_usd=0.0` lanza `OpenAISecurityViolationError` y realiza `0` llamadas. |
| **4** | Comprobar que una reserva válida se transmite correctamente al transporte | `test_05_mandatory_04_valid_reservation_transmitted_to_transport` | **PASS (`OK`)** | `execute_batch()` crea primero la reserva atómica, la pasa en `active_reservation` tanto a `MockOpenAITransport` como a `RealHttpOpenAITransport` (con `urlopen_fn` offline inyectado) y concilia en `RECONCILED`. |
| **5** | Comprobar que una reserva se libera si ocurre un error previo al envío | `test_06_mandatory_05_reservation_released_on_pre_send_error` | **PASS (`OK`)** | Ante bloqueo por `PHASE_1_PAID_API_LOCK` o credenciales ausentes, `execute_batch()` libera el 100 % de la reserva (`reserved_usd == 0.0`, estado `RELEASED_UNEXECUTED`). |
| **6** | Comprobar que ningún error permite el uso automático de `gpt-6.1-sol` | `test_07_mandatory_06_no_error_allows_automatic_sol_fallback` | **PASS (`OK`)** | Errores HTTP (`400..504`), reintentos agotados y fallos post-QA jamás escalan ni hacen fallback a `gpt-6.1-sol`. |
| **7** | Comprobar que solo se acepta `gpt-6-luna` | `test_08_mandatory_07_only_gpt_6_luna_accepted` | **PASS (`OK`)** | `RealHttpOpenAITransport` (`allowed_model_id="gpt-6-luna"`) rechaza `GPT61SolAdapter` con `ModelRoutingError`, `0` llamadas y libera la reserva. |
| **8** | Comprobar que los límites de `$0.05` y `$0.25` bloquean reservas excesivas | `test_09_mandatory_08_budget_limits_005_and_025_block_excessive_reservations` | **PASS (`OK`)** | Reservas que superan `$0.05 USD` por trabajo o `$0.25 USD` acumulados en el proyecto lanzan `InsufficientBudgetError` con `0` reservas huérfanas. |
| **9** | Comprobar que la base de producción permanece intacta | `test_10_mandatory_09_production_db_and_config_untouched` | **PASS (`OK`)** | `ProductionDatabaseLockError` protege `nolvus_translator.db`; tamaño (`431.386.624` bytes), `mtime_ns` (`1793683151811389300`) y `config.json` (`active_provider="mock"`) intactos. |
| **10** | Ejecutar la suite completa de pruebas offline | `python -m unittest discover -s tests -t . -v` (`test_01` a `test_10` + 48 pruebas previas) | **PASS (`58/58 OK`)** | `58` pruebas ejecutadas en `13,618 s` con `0` fallos, `0` errores y `0` llamadas de red. |

---

## 4. Inventario Actualizado de Hashes `SHA-256` en Staging Aislado

Los artefactos de payload (`meta.json`, `input.jsonl` y `openai_responses_request.json`) conservaron exactamente el mismo contenido bit a bit, mientras que las bases SQLite de staging se actualizaron limpiamente con `D-BUD-01` y `D-ISO-02`:

| Artefacto en `NolvusAIT_PilotStaging/Phase10_3A_Luna25/` | Estado / Integridad | Hash `SHA-256` verificado |
|---|---|---|
| `isolated_dbs/pilot_10_3a_work_db.sqlite3` | `PRAGMA integrity_check = ok` (`25` cadenas `unprocessed`, `108` términos de glosario, `0` TM) | `61806c72105fbe1915d5e64e266882f4c0be6e24008b2cecd937498d6bd439f0` |
| `isolated_dbs/pilot_10_3a_exchange_store.sqlite3` | `PRAGMA integrity_check = ok` (`AWAITING_AGENT`, `25` sobres en `placeholder_vault`, `manifest_sha256` unificado) | `a7e910f51d47a6cd0048b23fbf6e1510bbd77aabf23afda9245df66307480efc` |
| `isolated_dbs/pilot_10_3a_budget_store.sqlite3` | `PRAGMA integrity_check = ok` (`budget_projects = $0.25 USD ACTIVE`, `budget_jobs = $0.05 USD ACTIVE`, `reserved = $0.00`, `consumed = $0.00`) | `4bb18a2c2ffc22c5b0510c3dd5084735501368068705b8aa6f6ca1177614c994` |
| `staging_payloads/batch_batch_pilot_10_3a_001_meta.json` | JSON válido (`UTF-8`, inalterado) | `f036d564398542aca64bc66b7dcffd0a606d758cf3df9dc717cf6129a284a4e7` |
| `staging_payloads/batch_batch_pilot_10_3a_001_input.jsonl` | JSONL válido (`25` líneas, `1.384` chars en `source_masked`, inalterado) | `22493486b8f8b274517ca4196815727d82cfbf17261c41a8dea92a69d76b5306` |
| `staging_payloads/batch_batch_pilot_10_3a_001_openai_responses_request.json` | JSON válido (`/v1/responses`, `gpt-6-luna`, `tools=[]`, `tool_choice="none"`, `store=False`, `max_output_tokens=2198`, inalterado) | `aed55a99e06ff44c69c1fadafd5be07e94a1dcb8e0da8b572b7fb7588cea66b5` |

---

## 5. Dictamen Final de Preparación Pre-Piloto

Todas las discrepancias bloqueantes y documentales identificadas en `PHASE10_3B_READONLY_PREFLIGHT_AUDIT_REPORT.md` han quedado resueltas y verificadas mediante pruebas automatizadas offline (`58/58 OK`). El sistema permanece con `PHASE_1_PAID_API_LOCK = True` y está técnicamente listo para cuando el usuario autorice la ejecución real del piloto **Fase 10.3-B** previa configuración en su entorno local de `NOLVUS_OPENAI_API_KEY` y `NOLVUS_OPENAI_PROJECT_ID`.
