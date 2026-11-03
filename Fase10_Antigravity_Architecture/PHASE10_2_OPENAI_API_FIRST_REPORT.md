# Informe de Implementación — Fase 10.2: Infraestructura OpenAI API-First

**Proyecto:** Nolvus AI Translator (ES-419)  
**Fecha:** 2026-11-03  
**Rama del repositorio de código:** `feat/phase10-2-openai-api-first`  
**Commit de referencia:** `2766b73` (sobre base `0b4ff8d` de `feat/phase10-1b-antigravity-exchange`)  
**Estado de ejecución:** `COMPLETADO` (`48/48` métodos de prueba en verde en `5.482s`; `0` llamadas HTTP reales; `$0.00 USD` consumidos; `PHASE_1_PAID_API_LOCK = True`)

---

## 1. Resumen del Cambio Oficial de Estrategia (API-First)

Por decisión arquitectónica de la Fase 10.2, el proyecto evoluciona de una estrategia *Antigravity-first* a una arquitectura **OpenAI API-First** con separación estricta de responsabilidades, control presupuestario local obligatorio antes de cada envío y conservación íntegra de la tubería determinista local (`VariableProtector`, `PlaceholderVault`, `GlossaryManager`, `TranslationMemoryEngine`, `TranslationQualityValidator`, `LocalGatekeeper` de 12 puertas y cuarentena).

### 1.1 Distribución oficial de proveedores y modelos
1. **`GPT-6 Luna` (`gpt-6-luna`) — Proveedor principal de traducción masiva (`openai_luna`):**
   - Destinado a la traducción masiva del corpus de Nolvus Awakening 6.0.21 tanto por vía **Responses API (`/v1/responses`)** para lotes síncronos controlados como por **Batch API (`/v1/files` + `/v1/batches`)** para procesamiento asíncrono masivo.
   - Tarifa oficial configurada: **$0.10 USD / 1M tokens de entrada** y **$0.50 USD / 1M tokens de salida** (tarifa Standard usada siempre como base de reserva de seguridad; **$0.05 / $0.25 USD por 1M** en Batch API registrado como ahorro potencial no garantizado).
2. **`GPT-6.1 Sol` (`gpt-6.1-sol`) — Proveedor de escalamiento selectivo (`openai_sol`):**
   - Reservado exclusivamente para cadenas complejas, ambiguas, de alta sensibilidad al lore o que hayan fallado controles específicos de calidad tras un intento en Luna, y **únicamente cuando una regla verificable (`EscalationRuleCode`) lo justifique y `allow_sol_escalation=True` esté expresamente autorizado**.
   - Tarifa oficial configurada: **$2.00 USD / 1M tokens de entrada** y **$10.00 USD / 1M tokens de salida** (Standard) / **$1.00 / $5.00 USD por 1M** (Batch).
   - **Prohibiciones estrictas aplicadas en código (`ModelRouter`):**
     - Nunca se envía una misma cadena a Luna y Sol simultáneamente.
     - Un fallo HTTP, de red, timeout, 429 o 5xx de Luna **jamás** dispara fallback automático a Sol (`ProviderFailureAction.ABORT_OR_RETRY_SAME_MODEL`).
3. **`Antigravity Argon` (`gemini-4-argon`) — Ruta complementaria y alternativa (`antigravity_argon`):**
   - Se conserva como vía complementaria mediante el mecanismo de intercambio supervisado por archivos (`AntigravityExchangeService` implementado en la Fase 10.1-B.1).
4. **Exclusión de Claude (`claude-sonnet-5.5`, `claude-opus-5.5`, `claude*`):**
   - Excluido explícitamente del flujo principal de traducción y del enrutamiento automático (`assert_allowed_model_identifier` rechaza cualquier identificador `claude*` con `ModelRoutingError`).

---

## 2. Regla Fundamental: «Las API solo traducen»

La integración OpenAI (`openai_provider.py`, `openai_batch_coordinator.py`, `openai_credentials.py`) aplica un aislamiento estricto de responsabilidades validado por `assert_translation_only_payload()` antes de cualquier serialización o transporte:
- **`tools=[]` y `tool_choice="none"` obligatorios:** Cualquier intento de incluir herramientas (`web_search`, `file_search`, `code_interpreter`, `computer_use`, `function`, `mcp`) lanza inmediatamente `OpenAISecurityViolationError`.
- **`store=False` por defecto:** Evita almacenamiento remoto innecesario de payloads en el endpoint `/v1/responses`.
- **Bloqueo de rutas locales:** Se inspecciona recursivamente todo el payload saliente para impedir la filtración de rutas locales de Windows/POSIX (`C:\...`, `STOCK GAME`, `ModOrganizer.ini`, `.db`) hacia OpenAI.
- **Sin acceso a Nolvus, MO2, sistema de archivos ni base de datos:** Los adaptadores OpenAI solo reciben pares inmutables `(string_uid, source_text_protected)` ya preprocesados con placeholders `⟦VAR_0⟧..⟦VAR_n⟧`, las reglas compactas ES-419 y el subconjunto de glosario aplicable al lote.

---

## 3. Arquitectura e Inventario de Módulos Implementados

### 3.1 Archivos creados y modificados en `feat/phase10-2-openai-api-first`

| Archivo | Tipo | Responsabilidad principal |
|---|---|---|
| `src/nolvus_translator/core/exceptions.py` | Modificado | Conserva `PHASE_1_PAID_API_LOCK = True` y añade la jerarquía de control económico y seguridad OpenAI (`EconomicControlError`, `BudgetNotConfiguredError`, `BudgetExhaustedError`, `InsufficientBudgetError`, `RequestLimitExceededError`, `BudgetReconciliationError`, `OpenAISecurityViolationError`, `ModelRoutingError`). |
| `src/nolvus_translator/core/config.py` | Modificado | Añade configuración API-First (`primary_api_model="gpt-6-luna"`, `escalation_api_model="gpt-6.1-sol"`, `complementary_exchange_model="gemini-4-argon"`, `allow_sol_escalation=False`, `project_budget_usd=None`, `max_job_budget_usd=None`, `budget_safety_margin=1.15`, `explicit_user_billing_authorization=False`) y refuerza el bloqueo en `ConfigManager.save()`. |
| `src/nolvus_translator/translation/future_stubs.py` | Modificado | Registra las entradas bloqueadas `openai_luna` y `openai_sol` junto a los stubs existentes. |
| `src/nolvus_translator/translation/budget_control.py` | Nuevo | Controlador local de presupuesto (`BudgetController`) respaldado en SQLite con transacciones `BEGIN IMMEDIATE` y `threading.RLock`, estimación previa de tokens, reserva atómica con margen de seguridad (`1.15x`) sobre tarifa Standard, reconciliación post-respuesta, cuarentena y detención ante sobrecosto o respuesta sin `usage`, liberación de reservas no ejecutadas y recuperación tras caída (`recover_after_interruption`). |
| `src/nolvus_translator/translation/openai_credentials.py` | Nuevo | `OpenAIProjectSecurityProfile` (allowlist exclusiva `gpt-6-luna` / `gpt-6.1-sol`, endpoints `/v1/responses`, `/v1/files`, `/v1/batches`), `CredentialRedactor` (enmascaramiento de `sk-...` y `Bearer ...`), `SecureCredentialLoader` (carga desde variable de entorno sin auto-activación) y separación de 5 etapas (`ExecutionStage`). |
| `src/nolvus_translator/translation/model_router.py` | Nuevo | `ModelRouter`, `RoutingPolicy`, `RoutingDecision` y `EscalationRuleCode`. Enruta por defecto a `gpt-6-luna`, evalúa reglas verificables de escalamiento a `gpt-6.1-sol`, bloquea Claude, impide envío dual Luna+Sol y prohíbe usar Sol como fallback ante fallos técnicos de Luna. |
| `src/nolvus_translator/translation/openai_provider.py` | Nuevo | `OpenAIResponsesRequest` (`/v1/responses` con `json_schema` estricto `nolvus_es419_translation_batch`), `assert_translation_only_payload`, `BlockedNetworkTransport` (transporte por defecto con 0 red), `RealHttpOpenAITransport` (protegido por cuádruple candado), `MockOpenAITransport`, `GPT6LunaAdapter` y `GPT61SolAdapter` integrados con `BudgetController`, `PlaceholderVault`, `ExchangeStateStore` y `LocalGatekeeper`. |
| `src/nolvus_translator/translation/openai_batch_coordinator.py` | Nuevo | `OpenAIBatchCoordinator` para el flujo asíncrono `/v1/files` + `/v1/batches`: construcción JSONL separada por modelo, correlación biyectiva `custom_id`, reserva presupuestaria por línea, persistencia de estado para reanudación tras cierre de la app (`list_recoverable_batch_jobs`), cancelación con liberación de presupuesto (`cancel_batch_job`), reconciliación de duplicados/faltantes/errores parciales, validación en `LocalGatekeeper` y limpieza de archivos remotos (`cleanup_remote_files`). |
| `src/nolvus_translator/reports/pilot_estimator.py` | Nuevo | Estimador offline de consumo para el piloto (`fetch_stratified_real_corpus_sample` con conexión SQLite `mode=ro&immutable=1` + `estimate_pilot_from_rows`) que calcula tokens de entrada, salida, tope físico, costo Standard, reserva `+15%` y ahorro potencial Batch sin abrir sockets de red. |
| `tests/test_openai_budget_control.py` | Nuevo | 8 métodos de prueba que cubren los 10 escenarios económicos y de concurrencia de la Sección 8.C. |
| `tests/test_openai_provider_and_router.py` | Nuevo | 5 métodos de prueba que cubren Responses API, adaptadores Luna/Sol, Gatekeeper, idempotencia y reglas de enrutamiento (Sección 8.A). |
| `tests/test_openai_batch_coordinator.py` | Nuevo | 3 métodos de prueba que cubren construcción JSONL, estados de lote, errores parciales, duplicados/faltantes, cancelación, recuperación tras caída y limpieza remota (Sección 8.B). |
| `tests/test_openai_security_and_isolation.py` | Nuevo | 5 métodos de prueba que cubren redacción de secretos, bloqueo sin presupuesto/autorización, aislamiento de Nolvus/MO2/DB productiva, separación de las 5 etapas y el estimador offline del piloto (Sección 8.D). |

---

## 4. Diseño del Controlador Local de Presupuesto (`BudgetController`)

El módulo `src/nolvus_translator/translation/budget_control.py` implementa tres tablas SQLite dedicadas (`budget_config`, `budget_jobs`, `budget_reservations`) bajo `PRAGMA journal_mode=WAL` y transacciones `BEGIN IMMEDIATE` sincronizadas con `threading.RLock`:

1. **Estimación previa conservadora (`estimate_request`):**
   - Desglosa tokens de entrada en: `input_source_tokens` (cadenas protegidas con `⟦VAR_n⟧`), `input_instructions_tokens` (prompt del sistema ES-419), `input_glossary_tokens` (glosario filtrado por lote) y `input_framing_tokens` (sobrecarga del esquema JSON y metadatos).
   - Calcula `estimated_output_tokens` aplicando el factor de expansión ES-419 (`1.22x`) más sobrecarga JSON por ítem, y fija un tope físico duro `max_output_tokens_cap` (`1.25x` sobre la salida estimada).
   - Calcula siempre la reserva sobre la **tarifa Standard** del modelo (`$0.10 / $0.50` por 1M para `gpt-6-luna`; `$2.00 / $10.00` por 1M para `gpt-6.1-sol`) asumiendo el peor caso (`max_output_tokens_cap`) y multiplicando por `safety_margin = 1.15` (`+15 %`).
2. **Reserva atómica previa al envío (`reserve_budget`):**
   - Rechaza cualquier operación si el presupuesto no está configurado (`BudgetNotConfiguredError`), es `<= 0`, está agotado (`BudgetExhaustedError`), supera los límites por petición (`RequestLimitExceededError`), o si la suma `consumed_usd + reserved_usd + candidate_reserved_usd` excede el tope del trabajo o del proyecto (`InsufficientBudgetError`).
   - En ejecución concurrente (múltiples hilos), el bloqueo `BEGIN IMMEDIATE` garantiza que dos hilos no puedan reservar simultáneamente el mismo saldo remanente.
3. **Reconciliación post-respuesta (`reconcile_reservation`):**
   - Si la respuesta carece del bloque `usage` o está truncada/incompleta, la reserva pasa a `QUARANTINED_UNKNOWN_COST`, el trabajo pasa a `HALTED_UNKNOWN_COST` y se bloquean envíos posteriores hasta auditoría.
   - Si el costo real calculado con los tokens reportados supera el monto reservado (`actual_cost_usd > reserved_usd`), se registra el gasto real, la reserva pasa a `QUARANTINED_OVERRUN`, el trabajo se detiene en `HALTED_OVERRUN` y se lanza `BudgetReconciliationError`.
   - En caso normal (`actual_cost_usd <= reserved_usd`), se libera la diferencia no utilizada (`reserved_usd - actual_cost_usd`) y se consolida únicamente `actual_cost_usd` en `consumed_usd`.
4. **Liberación y recuperación tras interrupción (`release_unexecuted_reservation` y `recover_after_interruption`):**
   - Si una petición falla antes de enviarse (p. ej. bloqueo local o cancelación antes de HTTP), la reserva se libera íntegramente (`RELEASED_UNEXECUTED`) devolviendo el 100 % del saldo.
   - Si el proceso se interrumpe después de abrir una reserva:
     - Sin `provider_response_id` ni `batch_id`: se libera de forma segura como no enviada.
     - Con identificador remoto en vuelo: se marca en cuarentena (`QUARANTINED_IN_FLIGHT_INTERRUPTED`) y detiene el trabajo hasta reconciliación explícita.

---

## 5. Resultados de la Suite de Pruebas Automatizadas (`48/48 PASS`)

Comando ejecutado:
```powershell
python -m unittest discover -s tests -t . -v
```
Resultado: **`Ran 48 tests in 5.482s — OK`** (`0` fallos, `0` errores, `0` solicitudes HTTP reales).

| Suite de pruebas | Métodos | Cobertura verificada | Estado |
|---|---:|---|---|
| `tests/test_end_to_end.py` | 2 | Regresión de flujo base E2E, VFS MO2, exportación y GUI | `2/2 OK` |
| `tests/test_formats.py` | 6 | Regresión de parsers binarios/textuales (`.esp`, `.strings`, `MCM`, `.pex`, `.bsa`, `DSD`) | `6/6 OK` |
| `tests/test_protector_glossary_tm.py` | 3 | Regresión de `VariableProtector`, `GlossaryManager`, `TranslationMemoryEngine` y `TranslationQualityValidator` | `3/3 OK` |
| `tests/test_exchange_contract.py` | 3 | Regresión de contrato JSON/JSONL `1.0.0` de intercambio Antigravity | `3/3 OK` |
| `tests/test_exchange_store.py` | 4 | Regresión de `ExchangeStateStore`, bóveda de placeholders y cuarentena | `4/4 OK` |
| `tests/test_gatekeeper.py` | 4 | Regresión de `LocalGatekeeper` (12 puertas), atomicidad y reintentos multi-salto | `4/4 OK` |
| `tests/test_exchange_cli.py` | 2 | Regresión de CLI `nolvus_exchange` y bloqueo de DB productiva | `2/2 OK` |
| `tests/test_phase9_coverage_and_scale.py` | 3 | Regresión de hallazgos Fase 9 (`F9-01..F9-04`) y escala sintética (2.500 cadenas) | `3/3 OK` |
| `tests/test_openai_budget_control.py` | 8 | Presupuesto `0`/no configurado, insuficiente, reserva previa, descuento del costo real, liberación de remanente, sobrecosto real (`HALTED_OVERRUN`), respuesta sin `usage` (`HALTED_UNKNOWN_COST`), fallo pre-envío, interrupción a mitad de lote y concurrencia multi-hilo (8 hilos sobre saldo para 3 reservas) | `8/8 OK` |
| `tests/test_openai_provider_and_router.py` | 5 | Construcción de request `/v1/responses`, rechazo de tools/rutas locales, adaptador `GPT6LunaAdapter` con placeholders complejos + glosario + 12 puertas del Gatekeeper, cuarentena de respuestas truncadas/placeholders rotos, idempotencia de lote y reglas de `ModelRouter` (Luna por defecto, Sol solo con regla y permiso, no fallback Luna→Sol, bloqueo de Claude) | `5/5 OK` |
| `tests/test_openai_batch_coordinator.py` | 3 | Construcción JSONL separada por modelo, correlación `custom_id`, ciclo de estados (`created`, `in_progress`, `completed`, `failed`, `expired`, `cancelled`), errores parciales, duplicados/faltantes, cancelación con liberación de presupuesto, persistencia/recuperación tras reinicio y limpieza de archivos remotos | `3/3 OK` |
| `tests/test_openai_security_and_isolation.py` | 5 | Redacción de claves `sk-...`/`Bearer` en logs y excepciones, no auto-activación aunque exista `OPENAI_API_KEY` en el entorno, perfil de proyecto dedicado, aislamiento de Nolvus/MO2/DB productiva, separación de las 5 etapas (`ExecutionStage`) y estimador offline del piloto | `5/5 OK` |
| **Total** | **48** | **Suite completa (línea base + Fase 10.1-B.1 + Fase 9 + Fase 10.2 OpenAI API-First)** | **`48/48 OK`** |

---

## 6. Estimación Preliminar y Propuesta para el Primer Piloto Real con `gpt-6-luna`

Utilizando el módulo `src/nolvus_translator/reports/pilot_estimator.py`, se extrajo en modo **100 % offline y solo lectura inmutable (`mode=ro&immutable=1`)** una muestra estratificada de cadenas reales pendientes de `workspace_data/nolvus_translator.db` (repartida equitativamente en 5 categorías: `dialogues_and_quests`, `mcm_and_ui`, `items_spells_perks`, `books_and_notes` y `scripts_and_other`) sin abrir ninguna conexión de red ni modificar un solo byte de la base de datos (`431.386.624` bytes e instante de modificación inalterados).

### 6.1 Tabla comparativa de escalones para el primer piloto (`gpt-6-luna`, lotes de 25 cadenas)

| Métrica | Escalón 1: Micro-Piloto (`25` cadenas / `1` lote) | Escalón 2: Piloto Recomendado (`100` cadenas / `4` lotes) | Escalón 3: Piloto Ampliado (`250` cadenas / `10` lotes) |
|---|---:|---:|---:|
| **Cadenas reales (5 categorías estratificadas)** | `25` (`5` por categoría) | `100` (`20` por categoría) | `250` (`50` por categoría) |
| **Caracteres fuente totales** | `1.365` | `5.925` | `20.264` |
| **Placeholders protegidos (`⟦VAR_n⟧`)** | `25` | `120` | `222` |
| **Tokens de entrada estimados (total)** | **`1.877`** | **`7.711`** | **`20.573`** |
| — *Tokens de texto fuente protegido* | `468` | `2.106` | `6.531` |
| — *Tokens de instrucciones del sistema ES-419* | `464` | `1.856` | `4.640` |
| — *Tokens de glosario aplicable al lote* | `25` | `69` | `202` |
| — *Tokens de estructura/esquema JSON* | `920` | `3.680` | `9.200` |
| **Tokens de salida estimados (`1.22x` + JSON)** | **`1.628`** | **`6.829`** | **`18.781`** |
| **Tope físico de salida (`max_output_tokens`)** | **`2.035`** | **`8.539`** | **`23.480`** |
| **Costo base esperado (tarifa Standard `$0.10/$0.50` por 1M)** | `$0.001002 USD` | `$0.004186 USD` | `$0.011448 USD` |
| **Costo máximo base al tope físico (tarifa Standard)** | **`$0.001205 USD`** | **`$0.005041 USD`** | **`$0.013797 USD`** |
| **Reserva atómica requerida (`+15 %` margen de seguridad)** | **`$0.001386 USD`** | **`$0.005797 USD`** | **`$0.015867 USD`** |
| **Costo potencial con Batch API (`50 %` dto., no asumido en reserva)** | `$0.000501 USD` | `$0.002093 USD` | `$0.005724 USD` |
| **Presupuesto de trabajo recomendado a autorizar (`max_job_budget_usd`)** | **`$0.05 USD`** | **`$0.05 USD`** | **`$0.05 USD`** |

### 6.2 Recomendación operativa para la siguiente puerta (Fase 10.3 — Primer Piloto Real Autorizado)
- Se recomienda autorizar el **Escalón 2 (`100` cadenas reales en `4` lotes de `25` cadenas)** o, si se prefiere validar primero un único lote mínimo antes de los 3 restantes, iniciar con el **Escalón 1 (`25` cadenas en `1` lote)** y continuar con las `75` restantes dentro de la misma ventana.
- Incluso para `100` cadenas reales, la reserva máxima con margen del `+15 %` en tarifa Standard es de apenas **`$0.005797 USD` (~0,58 centavos de dólar)**.
- Fijar un presupuesto duro de trabajo de **`max_job_budget_usd = $0.05 USD` (5 centavos de dólar)** y un presupuesto global de proyecto inicial de **`project_budget_usd = $0.25 USD` (25 centavos de dólar)** proporciona un margen holgado para el piloto y eventuales micro-lotes de reintento, manteniendo un techo económico prácticamente nulo y 100 % fiscalizado por `BudgetController`.
