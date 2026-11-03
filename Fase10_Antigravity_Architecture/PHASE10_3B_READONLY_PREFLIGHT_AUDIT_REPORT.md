# AUDITORÍA DE SOLO LECTURA — PREPARACIÓN PARA LA FASE 10.3-B (PRIMER PILOTO REAL `GPT-6 LUNA`)

**Proyecto:** Nolvus AI Translator  
**Instancia objetivo:** Nolvus Awakening 6.0.21 (`ES-419`)  
**Alcance:** Auditoría técnica preflight de **solo lectura** previa a la primera ejecución real del lote piloto `batch_pilot_10_3a_001` (`25` cadenas reales)  
**Fecha:** 2026-11-03  
**Rama inspeccionada (repositorio de código):** `feat/phase10-2-openai-api-first` (`HEAD` en `050ba6d` / código en `d2b75a5`)  
**Llamadas HTTP / API realizadas durante esta auditoría:** **`0`**  
**Costo generado:** **`$0.000000 USD`**  
**Archivos de código, bases de datos, configuración o credenciales modificados:** **`0`**

---

## 1. Resumen Ejecutivo de Estado por Comprobación

| # | Bloque auditado | Estado | Resumen ejecutivo |
|---:|---|---|---|
| **1** | **Credenciales (`openai_credentials.py` y `openai_provider.py`)** | **`BLOQUEADO`** | Estado de la credencial en el entorno (`Process`, `User` y `Machine`): **`AUSENTE`** (tanto para `NOLVUS_OPENAI_API_KEY` como para `OPENAI_API_KEY` y `NOLVUS_OPENAI_PROJECT_ID`). Además, se identificaron **3 discrepancias técnicas** en el cableado de `RealHttpOpenAITransport` y `BaseOpenAIModelAdapter.execute_batch()` que impiden recuperar la clave y ejecutar el transporte HTTP real incluso si se desbloqueara el candado. |
| **2** | **Modelo y restricciones (`gpt-6-luna`, `/v1/responses`, sandbox)** | **`APROBADO`** | Único modelo configurado y registrado en el lote: **`gpt-6-luna`**. Endpoint exclusivo: **`POST /v1/responses`**. Payload verificado con **`tools=[]`**, **`tool_choice="none"`**, **`store=False`** y **`max_output_tokens=2198`**. Sin escalamiento automático a `gpt-6.1-sol` (`allow_sol_escalation=False`) y sin fallback a otro modelo ante fallos HTTP (`ProviderFailureAction.RETRY_SAME_MODEL` / `HALT_AND_REPORT`). |
| **3** | **Bloqueo de API (`PHASE_1_PAID_API_LOCK`)** | **`BLOQUEADO`** | Valor actual verificado sin modificar: **`PHASE_1_PAID_API_LOCK = True`** en `src/nolvus_translator/core/exceptions.py`. Impide toda ejecución facturable hasta autorización explícita. |
| **4** | **Presupuesto (`BudgetController` y `pilot_10_3a_budget_store.sqlite3`)** | **`ADVERTENCIA`** | Reservas (`reserved_usd`) y consumo (`consumed_usd`) confirmados en **`$0.000000 USD`**. Los topes de referencia del piloto (`$0.05 USD` trabajo / `$0.25 USD` proyecto, margen `1.15x`) están definidos en `pilot_offline_staging.py` y `staging_manifest_10_3a.json`, pero las tablas `budget_projects` y `budget_jobs` de `pilot_10_3a_budget_store.sqlite3` están aún vacías (`0` filas) porque `initialize_project_and_job_budget()` no se invocó durante el staging offline; además, en `workspace_data/config.json` siguen en `null`. |
| **5** | **Aislamiento del piloto (`Phase10_3A_Luna25` e integridad)** | **`APROBADO`** | Las 3 bases SQLite aisladas (`pilot_10_3a_work_db.sqlite3`, `pilot_10_3a_exchange_store.sqlite3`, `pilot_10_3a_budget_store.sqlite3`) tienen `PRAGMA integrity_check = ok` y los **6 hashes `SHA-256`** coinciden al 100 % con `staging_manifest_10_3a.json`. Confirmadas **`25` cadenas en estado `unprocessed`** (`translated_text = ''`), **`0` entradas en `translation_memory`** y **`0` conexiones/modificaciones sobre la base de producción** `workspace_data/nolvus_translator.db`. |

---

## 2. Detalle Técnico por Punto Auditado

### Punto 1 — Credenciales (`BLOQUEADO`)

- **Archivos examinados:**
  - `src/nolvus_translator/translation/openai_credentials.py` (líneas `30–244`)
  - `src/nolvus_translator/translation/openai_provider.py` (líneas `529–569`, `986–1019`, `1271–1348`)
- **Estado de presencia de credenciales (sin leer, mostrar, copiar ni registrar valores):**
  - `NOLVUS_OPENAI_API_KEY` (entorno `Process` / `User` / `Machine`): **`AUSENTE`**
  - `OPENAI_API_KEY` (entorno `Process` / `User` / `Machine`): **`AUSENTE`**
  - `NOLVUS_OPENAI_PROJECT_ID` (entorno `Process` / `User` / `Machine`): **`AUSENTE`**
  - `NOLVUS_OPENAI_ORG_ID` (entorno `Process` / `User` / `Machine`): **`AUSENTE`**
  - `OpenAIProjectSecurityProfile.external_credential_file`: **`AUSENTE`** (`None`)
  - Resultado de `SecureCredentialLoader.check_environment_presence_without_activating()`:
    - `api_key_configured`: **`AUSENTE`** (`False`)
    - `project_id_configured`: **`AUSENTE`** (`False`)
- **Evaluación del mecanismo de integración entre `openai_credentials.py` y `openai_provider.py`:**
  1. En `openai_credentials.py`, `OpenAIProjectSecurityProfile` (líneas `95–107`) define como nombres de variables esperadas:
     - `api_key_env_var = "NOLVUS_OPENAI_API_KEY"`
     - `project_id_env_var = "NOLVUS_OPENAI_PROJECT_ID"`
     - `org_id_env_var = "NOLVUS_OPENAI_ORG_ID"`
  2. `SecureCredentialLoader.load_for_authorized_billable_call()` (líneas `204–244`) exige simultáneamente:
     - Superar `assert_billable_execution_authorized()` (`PHASE_1_PAID_API_LOCK == False`, `stage == ExecutionStage.BILLABLE_REAL_EXECUTION`, `explicit_user_billing_authorization == True` y `active_reservation` válida con `reserved_usd > 0`).
     - Que exista clave en `NOLVUS_OPENAI_API_KEY` (o en `external_credential_file`).
     - Que exista identificador de proyecto en `NOLVUS_OPENAI_PROJECT_ID` (obligatorio en líneas `231–235`).
  3. **Discrepancias técnicas encontradas en la integración con `openai_provider.py` (bloqueantes para cuando se autorice la Fase 10.3-B):**
     - **`D-CRED-01` (Nombre de variable de entorno y `PROJECT_ID` obligatorio):** La documentación general menciona `OPENAI_API_KEY`, pero `OpenAIProjectSecurityProfile` lee por defecto `NOLVUS_OPENAI_API_KEY` y exige además `NOLVUS_OPENAI_PROJECT_ID`.
     - **`D-CRED-02` (`RealHttpOpenAITransport.send_request` es un stub cerrado sin cliente HTTP):** En `src/nolvus_translator/translation/openai_provider.py` (líneas `552–569`), `RealHttpOpenAITransport.send_request()` ejecuta `assert_billable_execution_authorized(...)` y en la línea `568` lanza incondicionalmente `raise ExternalAPIDisabledError("OpenAI Real HTTP Transport")`. No invoca todavía `self.credential_loader.load_for_authorized_billable_call(...)` ni realiza la petición HTTPS a `https://api.openai.com/v1/responses`.
     - **`D-CRED-03` (Orden de validación de `active_reservation` en `BaseOpenAIModelAdapter.execute_batch`):**
       - En `src/nolvus_translator/translation/openai_provider.py` (líneas `1289–1295`), `execute_batch()` llama a `assert_billable_execution_authorized(..., active_reservation=None)` **antes** de crear la reserva presupuestaria en la línea `1315` (`self.budget.reserve_budget(...)`). Como `assert_billable_execution_authorized()` exige `active_reservation is not None` (líneas `159–162` de `openai_credentials.py`), esa llamada preliminar fallará siempre con `OpenAISecurityViolationError`.
       - Adicionalmente, en la línea `1326`, `self.transport.send_request(...)` no pasa el argumento `active_reservation=reservation` que `RealHttpOpenAITransport.send_request()` espera recibir.

---

### Punto 2 — Modelo y Restricciones (`APROBADO`)

- **Archivos examinados:**
  - `src/nolvus_translator/translation/model_router.py` (líneas `23–30`, `69–94`, `96–287`)
  - `src/nolvus_translator/translation/openai_provider.py` (líneas `81–263`, `1189–1223`, `1362–1379`, `1465–1516`)
  - `NolvusAIT_PilotStaging/Phase10_3A_Luna25/staging_payloads/batch_batch_pilot_10_3a_001_openai_responses_request.json`
  - `NolvusAIT_PilotStaging/Phase10_3A_Luna25/staging_payloads/batch_batch_pilot_10_3a_001_meta.json`
  - `NolvusAIT_PilotStaging/Phase10_3A_Luna25/staging_payloads/staging_manifest_10_3a.json`
- **Evidencias verificadas:**
  1. **Único modelo habilitado para el piloto:**
     - `batch_batch_pilot_10_3a_001_openai_responses_request.json`: `"model": "gpt-6-luna"`, `"metadata.model_id": "gpt-6-luna"`.
     - `batch_batch_pilot_10_3a_001_meta.json`: `"recorded_model_selection": "gpt-6-luna"`.
     - `staging_manifest_10_3a.json`: `"model_id": "gpt-6-luna"`.
     - `pilot_10_3a_exchange_store.sqlite3`: `recorded_model_selection = "gpt-6-luna"` tanto en `exchange_jobs` como en `exchange_batches`.
  2. **Uso exclusivo de la API Responses (`/v1/responses`):**
     - `OPENAI_RESPONSES_ENDPOINT = "/v1/responses"` (`openai_provider.py` línea `81`).
     - `GPT6LunaAdapter` envía únicamente `POST /v1/responses` (`openai_provider.py` líneas `1326–1328`) validado por el modelo Pydantic estricto `OpenAIResponsesRequest` (`extra="forbid"`, `strict=True`).
  3. **Restricciones de herramientas y almacenamiento confirmadas en código y en el payload de staging:**
     - `"tools": []` (validado por `_validate_empty_tools` y `assert_translation_only_payload`).
     - `"tool_choice": "none"` (validado por `_validate_tool_choice_none` y `assert_translation_only_payload`).
     - `"store": false` (validado por `_validate_store_false`).
     - `"max_output_tokens": 2198`.
     - `"text": {"format": {"type": "json_schema", "name": "nolvus_es419_translation_batch", "strict": true, ...}}`.
     - Bloqueo explícito de `_FORBIDDEN_REQUEST_KEYS` (`functions`, `function_call`, `web_search`, `web_search_options`, `file_search`, `code_interpreter`, `computer_use`, `mcp`, `mcp_servers`, `reasoning`, `background_agent`).
  4. **Ausencia de escalamiento automático a Sol y de fallback a otro modelo:**
     - `RoutingPolicy` (`model_router.py` líneas `100–102`) establece por defecto `allow_sol_escalation = False`, `allow_post_qa_sol_escalation = False`, `use_complementary_argon_exchange = False`.
     - `GPT61SolAdapter.execute_batch()` (`openai_provider.py` líneas `1506–1510`) lanza `ModelRoutingError` si `allow_sol_escalation` es `False`.
     - `ModelRouter.handle_provider_failure()` (`model_router.py` líneas `252–286`) prohíbe cambiar de modelo ante errores HTTP/red (`ModelRoutingError` en líneas `268–273`) y solo permite `ProviderFailureAction.RETRY_SAME_MODEL` (máximo `2` reintentos en errores transitorios `408/429/500/502/503/504`) o `ProviderFailureAction.HALT_AND_REPORT` (en `400/401/403/404/422`).
- **Discrepancia documental menor detectada (`D-DOC-01`):**
  - La tabla descriptiva de la Sección 4.1 de `PHASE10_3A_OFFLINE_PILOT_PREPARATION_REPORT.md` mencionaba `temperature = 0.1` y un ejemplo de `metadata` con claves `project`/`task`. En el código real (`OpenAIResponsesRequest` con `extra="forbid"`) y en el archivo real `batch_batch_pilot_10_3a_001_openai_responses_request.json` no se incluye el campo `temperature` (las 9 claves exactas son `model`, `instructions`, `input`, `text`, `max_output_tokens`, `tools`, `tool_choice`, `store`, `metadata`), y `metadata` contiene `{"job_id": "job_pilot_10_3a_luna25", "batch_id": "batch_pilot_10_3a_001", "model_id": "gpt-6-luna", "idempotency_key": "04d8af437f0f5bb22288e3555caf1201936d58fd0cd03455f29a9777bcc0a464", "purpose": "translation_only"}`.

---

### Punto 3 — Bloqueo de API (`BLOQUEADO`)

- **Archivo examinado:**
  - `src/nolvus_translator/core/exceptions.py` (línea `10`)
- **Valor actual verificado (sin modificar):**
  - **`PHASE_1_PAID_API_LOCK: bool = True`**
- **Efecto verificado:**
  - Bloquea en tiempo de ejecución `assert_billable_execution_authorized()` (`openai_credentials.py` líneas `151–152`), `BaseOpenAIModelAdapter.__init__()` cuando se solicita ejecución real o autorización de facturación (`openai_provider.py` líneas `1001–1003`) y `ConfigManager.save()` si se intenta cambiar `active_provider` o `explicit_user_billing_authorization` (`config.py` líneas `101–104`).

---

### Punto 4 — Presupuesto (`ADVERTENCIA`)

- **Archivos examinados:**
  - `src/nolvus_translator/translation/budget_control.py` (líneas `85–106`, `141–214`, `440–523`, `632–800`)
  - `src/nolvus_translator/translation/pilot_offline_staging.py` (líneas `413–430`, `592–598`)
  - `NolvusAIT_PilotStaging/Phase10_3A_Luna25/isolated_dbs/pilot_10_3a_budget_store.sqlite3` (inspeccionado con `mode=ro&immutable=1`)
  - `NolvusAIT_PilotStaging/Phase10_3A_Luna25/staging_payloads/staging_manifest_10_3a.json`
  - `workspace_data/config.json`
- **Límites configurados de trabajo y proyecto:**
  1. **En la preparación del piloto (`pilot_offline_staging.py` líneas `413–423` y `staging_manifest_10_3a.json`):**
     - Presupuesto máximo por trabajo (`max_job_budget_usd`): **`$0.05 USD`** (`0.05`)
     - Presupuesto interno del proyecto (`project_budget_usd`): **`$0.25 USD`** (`0.25`)
     - Factor de margen de seguridad (`safety_margin_factor`): **`1.15`** (`+15 %`)
     - Límites operativos por lote: `max_strings_per_batch = 25`, `max_tokens_per_string = 2.000`, `max_input_tokens_per_batch = 24.000`, `max_output_tokens_per_request = 12.000`, `max_retries = 2`, `max_concurrent_requests = 1`.
  2. **En la configuración general del workspace (`workspace_data/config.json`):**
     - `project_budget_usd`: `null`
     - `max_job_budget_usd`: `null`
     - `explicit_user_billing_authorization`: `null`
     - `active_provider`: `"mock"`
- **Estado de reservas y consumo en `pilot_10_3a_budget_store.sqlite3`:**
  - `budget_schema_meta`: `[{"key": "schema_version", "value": "10.2.v1"}]`
  - `budget_projects`: **`0` filas**
  - `budget_jobs`: **`0` filas**
  - `budget_reservations`: **`0` filas**
  - `budget_audit_log`: **`0` filas**
  - **Reservas registradas (`reserved_usd`):** **`$0.000000 USD` (`0.0`)**
  - **Consumo registrado (`consumed_usd`):** **`$0.000000 USD` (`0.0`)**
- **Motivo de la clasificación `ADVERTENCIA` (`D-BUD-01`):**
  - Durante la Fase 10.3-A no se llamó a `budget_controller.initialize_project_and_job_budget(job_id="job_pilot_10_3a_luna25")` para mantener las tablas contables en cero. Cuando se autorice la Fase 10.3-B, será obligatorio invocar primero `initialize_project_and_job_budget(job_id="job_pilot_10_3a_luna25")` sobre `pilot_10_3a_budget_store.sqlite3` antes de llamar a `reserve_budget()`; de lo contrario, `reserve_budget()` rechazará la operación con `BudgetNotConfiguredError` (líneas `654–667` de `budget_control.py`).

---

### Punto 5 — Aislamiento del Piloto e Integridad (`APROBADO`)

- **Archivos examinados (lectura estricta `mode=ro&immutable=1`):**
  - `NolvusAIT_PilotStaging/Phase10_3A_Luna25/isolated_dbs/pilot_10_3a_work_db.sqlite3`
  - `NolvusAIT_PilotStaging/Phase10_3A_Luna25/isolated_dbs/pilot_10_3a_exchange_store.sqlite3`
  - `NolvusAIT_PilotStaging/Phase10_3A_Luna25/isolated_dbs/pilot_10_3a_budget_store.sqlite3`
  - `NolvusAIT_PilotStaging/Phase10_3A_Luna25/staging_payloads/batch_batch_pilot_10_3a_001_meta.json`
  - `NolvusAIT_PilotStaging/Phase10_3A_Luna25/staging_payloads/batch_batch_pilot_10_3a_001_input.jsonl`
  - `NolvusAIT_PilotStaging/Phase10_3A_Luna25/staging_payloads/batch_batch_pilot_10_3a_001_openai_responses_request.json`
  - `NolvusAIT_PilotStaging/Phase10_3A_Luna25/staging_payloads/staging_manifest_10_3a.json`
  - `workspace_data/nolvus_translator.db` (solo metadatos `stat()` del sistema de archivos)
- **Verificaciones de aislamiento y estado de las 25 cadenas:**
  1. **Separación respecto a la base de producción:**
     - `ExchangeStateStore` y `BudgetController` validan sus rutas con `assert_safe_test_db_path()`, que bloquea cualquier archivo llamado `nolvus_translator.db*` o ubicado dentro de `workspace_data/`.
     - En `pilot_10_3a_exchange_store.sqlite3`, el trabajo `job_pilot_10_3a_luna25` apunta exclusivamente al directorio de staging de `Phase10_3A_Luna25` (`snapshot_id = 1`).
     - La base de producción `workspace_data/nolvus_translator.db` permanece intacta con `size_bytes = 431.386.624` y `mtime_ns = 1793683151811389300` (idénticos al registro previo en `staging_manifest_10_3a.json`).
  2. **Estado verificado de `pilot_10_3a_work_db.sqlite3`:**
     - Total de filas en `string_entries`: **`25`**
     - Filas con `review_status = 'unprocessed'`: **`25`** (`100 %` sin procesar)
     - Filas con `translated_text = ''` y `existing_translation = ''`: **`25`** (`100 %` sin traducir)
     - Filas con `is_simulated = 0` y `provider_id = ''`: **`25`**
     - Filas en `translation_memory`: **`0`** (vacía)
     - Filas en `tm_versions`: **`0`** (vacía)
     - Filas en `translation_jobs` / `translation_batches`: **`0` / `0`**
     - Filas en `glossary_terms`: **`108`** términos sembrados
  3. **Estado verificado de `pilot_10_3a_exchange_store.sqlite3`:**
     - `exchange_jobs`: **`1`** (`job_pilot_10_3a_luna25`, `status = 'ACTIVE'`, `recorded_model_selection = 'gpt-6-luna'`, `max_retries = 2`).
     - `exchange_batches`: **`1`** (`batch_pilot_10_3a_001`, `item_count = 25`, `state = 'AWAITING_AGENT'`, `output_sha256 = ''`).
     - `exchange_items`: **`25`** ítems registrados.
     - `placeholder_vault`: **`25`** sobres sellados (`37` placeholders protegidos).
     - `quarantine_items`: **`0`** filas.
     - `ingestion_ledger`: **`0`** filas.
  4. **Integridad SQLite y verificación de hashes `SHA-256` (`6/6` exactos):**

| Archivo en `NolvusAIT_PilotStaging/Phase10_3A_Luna25/` | `PRAGMA integrity_check` / Formato | Hash `SHA-256` verificado en disco | ¿Coincide con `staging_manifest_10_3a.json`? |
|---|---|---|---|
| `isolated_dbs/pilot_10_3a_work_db.sqlite3` | `ok` | `f6d5baec6f5b83892a2976d3062a08f1c863590e629a024d088d198428fcf00f` | **SÍ (`True`)** |
| `isolated_dbs/pilot_10_3a_exchange_store.sqlite3` | `ok` | `8f1f0010118ef0cd60b97c3e2f1979090218db3b81ea5c3eb6eed71189dc7b90` | **SÍ (`True`)** |
| `isolated_dbs/pilot_10_3a_budget_store.sqlite3` | `ok` | `df5210bfd247fe3bb7af2c375329e24d54ca6d656e3fbbc7a96e0e21a03aa36a` | **SÍ (`True`)** |
| `staging_payloads/batch_batch_pilot_10_3a_001_meta.json` | JSON (`UTF-8`) | `f036d564398542aca64bc66b7dcffd0a606d758cf3df9dc717cf6129a284a4e7` | **SÍ (`True`)** |
| `staging_payloads/batch_batch_pilot_10_3a_001_input.jsonl` | JSONL (`25` líneas) | `22493486b8f8b274517ca4196815727d82cfbf17261c41a8dea92a69d76b5306` | **SÍ (`True`)** |
| `staging_payloads/batch_batch_pilot_10_3a_001_openai_responses_request.json` | JSON (`UTF-8`) | `aed55a99e06ff44c69c1fadafd5be07e94a1dcb8e0da8b572b7fb7588cea66b5` | **SÍ (`True`)** |

---

## 3. Consolidado de Discrepancias Encontradas

| Código | Severidad | Componente | Descripción exacta de la discrepancia |
|---|---|---|---|
| **`D-CRED-01`** | **Alta (Preflight 10.3-B)** | `openai_credentials.py` (`OpenAIProjectSecurityProfile`, líneas `99–100`, `219–235`) | El cargador `SecureCredentialLoader` busca por defecto la variable `NOLVUS_OPENAI_API_KEY` (no `OPENAI_API_KEY`) y exige obligatoriamente `NOLVUS_OPENAI_PROJECT_ID`. Actualmente ambas están **`AUSENTE`** en el sistema. |
| **`D-CRED-02`** | **Alta (Preflight 10.3-B)** | `openai_provider.py` (`RealHttpOpenAITransport.send_request`, líneas `552–569`) | `RealHttpOpenAITransport.send_request()` es actualmente un stub de seguridad que tras validar `assert_billable_execution_authorized()` lanza siempre `raise ExternalAPIDisabledError("OpenAI Real HTTP Transport")` (línea `568`), sin llamar aún a `self.credential_loader.load_for_authorized_billable_call()` ni enviar la petición HTTPS a `https://api.openai.com/v1/responses`. |
| **`D-CRED-03`** | **Alta (Preflight 10.3-B)** | `openai_provider.py` (`BaseOpenAIModelAdapter.execute_batch`, líneas `1289–1334`) | En `execute_batch()`, la línea `1290` invoca `assert_billable_execution_authorized(..., active_reservation=None)` **antes** de crear la reserva (`self.budget.reserve_budget(...)` en la línea `1315`), lo que causa que `assert_billable_execution_authorized()` falle siempre por `active_reservation is None`. Además, la línea `1326` invoca `self.transport.send_request(...)` sin pasar `active_reservation=reservation`. |
| **`D-BUD-01`** | **Media (Operativa)** | `pilot_10_3a_budget_store.sqlite3` / `pilot_offline_staging.py` | Las tablas `budget_projects` y `budget_jobs` de `pilot_10_3a_budget_store.sqlite3` tienen `0` filas porque en la Fase 10.3-A solo se estimó el lote sin inicializar saldos. Antes de llamar a `reserve_budget()` en la Fase 10.3-B, debe invocarse `budget_controller.initialize_project_and_job_budget(job_id="job_pilot_10_3a_luna25")`. |
| **`D-DOC-01`** | **Baja (Documental)** | `PHASE10_3A_OFFLINE_PILOT_PREPARATION_REPORT.md` (Sección 4.1) | El informe previo listó una fila descriptiva `temperature = 0.1` en la tabla del payload `/v1/responses`, pero el modelo real `OpenAIResponsesRequest` (`extra="forbid"`) y el archivo `batch_batch_pilot_10_3a_001_openai_responses_request.json` no incluyen la clave `temperature`. |
| **`D-ISO-01`** | **Baja (Documental)** | `pilot_10_3a_work_db.sqlite3` (`glossary_terms`) | La tabla `glossary_terms` de `pilot_10_3a_work_db.sqlite3` contiene **`108` términos** sembrados por `GlossaryManager.ensure_seeded()` (el informe de la Fase 10.3-A citaba `87`). |
| **`D-ISO-02`** | **Baja (Trazabilidad)** | `pilot_10_3a_exchange_store.sqlite3` (`exchange_batches.manifest_sha256`) | La columna `manifest_sha256` en `exchange_batches` guarda el hash del manifiesto compacto (`637068aee029...`), mientras que `batch_batch_pilot_10_3a_001_meta.json` en disco está formateado con `indent=2` (`f036d5643985...`). El `input_sha256` (`22493486b8f8...`) coincide en ambos y es el que verifica `LocalGatekeeper`. |

---

## 4. Declaración Expresa de Cero Llamadas Externas y Cero Costo

Se confirma expresamente que durante esta auditoría:

1. **No se realizó ninguna llamada HTTP ni solicitud a la API de OpenAI ni a ningún servicio externo (`0` solicitudes de red).**
2. **No se generó ningún costo ni se reservó saldo (`$0.000000 USD` gastados, `$0.000000 USD` reservados).**
3. **No se imprimió, mostró, copió, registró ni expuso ninguna clave o secreto.**
4. **No se modificó ningún archivo de código fuente, configuración, credencial ni base de datos (los 6 hashes `SHA-256` del directorio de staging `Phase10_3A_Luna25` y los metadatos `size_bytes`/`mtime_ns` de `workspace_data/nolvus_translator.db` permanecen 100 % idénticos).**
