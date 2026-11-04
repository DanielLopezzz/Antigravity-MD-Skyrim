# FASE 10.3-B — INFORME DE CORRECCIÓN TÉCNICA Y EJECUCIÓN CONTROLADA DEL PILOTO REAL (25 CADENAS — `gpt-6-luna`)

- **Fecha y hora de ejecución (Intento 2):** `2026-11-04T01:50:52.983372+00:00` (`20:50:52 UTC-5`)
- **Identificador del trabajo (Intento 1 preservado):** `job_pilot_10_3a_luna25` (`HALTED_QUARANTINE`)
- **Identificador del trabajo (Intento 2 controlado):** `job_pilot_10_3b_luna25_att2` (`ACTIVE` / `RECONCILED`)
- **Identificador del lote:** `batch_pilot_10_3a_001`
- **Modelo utilizado:** `gpt-6-luna` (exclusivo)
- **Endpoint utilizado:** `POST https://api.openai.com/v1/responses`

---

## 1. Diagnóstico y Causa Raíz del Intento 1 (`resp_0a6ee1133950df8d016aea87823b9487d298c9ca44dbd0ad02`)

### 1.1 Por qué `consumed_usd` quedó en `0.0` aunque OpenAI reportó `usage`
- **Ubicación exacta:** `src/nolvus_translator/translation/budget_control.py`, método `BudgetController.reconcile_reservation()`.
- **Causa técnica:** En la implementación previa de `reconcile_reservation()`, la condición del Caso 1 comprobaba:
  ```python
  if (
      response_status != "completed"
      or actual_input_tokens is None
      or actual_output_tokens is None
      or actual_input_tokens < 0
      or actual_output_tokens < 0
  ):
  ```
  antes de calcular `actual_billed_cost`. Cuando OpenAI devolvió HTTP `200` con `status = "incomplete"` pero con el objeto `usage` poblado (`input_tokens = 4,693`, `output_tokens = 2,198`), el método entró en esa rama y llamó a `_quarantine_reservation_and_halt_job_in_tx(..., actual_cost=None)`. Dicho método auxiliar movió los `US$0.00149259` reservados de `reserved_usd` a `quarantined_usd` sin sumar el coste real facturado (`US$0.00156830`) a `consumed_usd` ni registrar el exceso (`+US$0.00007571`) en `reconciliation_delta_usd`.

### 1.2 Por qué la estimación de entrada (`1,989` tokens) subestimó el uso real (`4,693` tokens)
- **Ubicación exacta:** `src/nolvus_translator/translation/openai_provider.py` (`prepare_batch_request`) y `src/nolvus_translator/translation/budget_control.py` (`estimate_batch`).
- **Causa técnica:** `prepare_batch_request()` invocaba `estimate_batch()` antes de construir el diccionario `user_payload`, pasando únicamente las 25 cadenas enmascaradas crudas (`602` tokens), las instrucciones (`465` tokens) y una lista corta de pares de glosario (`467` tokens), más una constante fija de `120 + 32 * 25 = 920` tokens (`total = 1,989` tokens). En la solicitud real enviada a `/v1/responses`, el `user_payload` serializado incluye por cada uno de los 25 ítems los metadatos (`string_uid`, `source_sha256`, `record_type`, `field_name`), la lista completa de objetos `glossary_context` y `placeholder_hints` (`12,372` caracteres = `3,757` tokens BPE), más el esquema `text.format` (`json_schema` estricto) y el encuadre interno de Responses API (`4,693` tokens reales).

### 1.3 Por qué la salida se truncó en `2,198` tokens (`status = "incomplete"`)
- **Causa técnica:** El límite `max_output_tokens = 2,198` se calculó asumiendo únicamente la longitud del texto traducido multiplicada por `1.35 * 1.25`, sin contemplar la sobrecarga de llaves JSON por cada uno de los 25 objetos (`seq_id`, `string_uid`, `source_sha256` de 64 caracteres hexadecimales, `translated_masked_text` y `translator_note`) ni los tokens de salida/razonamiento consumidos por `gpt-6-luna` antes de cerrar el contenedor JSON. En el Intento 2, con el campo redundante `translator_note` eliminado del esquema de respuesta de OpenAI, la respuesta completa de los 25 ítems requirió **`3,056` tokens de salida** (`858` tokens por encima del tope de `2,198` del Intento 1).

---

## 2. Correcciones Aplicadas y Verificadas Offline

1. **Conciliación contable estricta con `usage` en `BudgetController.reconcile_reservation()` (`src/nolvus_translator/translation/budget_control.py`):**
   - Si `actual_input_tokens` o `actual_output_tokens` es `None` (o `< 0`), se mantiene el bloqueo por coste desconocido (`QUARANTINED_UNKNOWN_COST` / `HALTED_QUARANTINE`).
   - Si OpenAI reporta `usage` válido (`>= 0`) y `response_status != "completed"`, se calcula siempre `actual_billed_cost` con la tarifa configurada, se suma `actual_billed_cost` a `consumed_usd` en `budget_projects` y `budget_jobs`, se persiste `actual_cost_usd` y `reconciliation_delta_usd = actual_billed_cost - reserved_usd` en `budget_reservations`, se mantiene cualquier remanente no gastado (`max(0.0, reserved_usd - actual_billed_cost)`) en `quarantined_usd` sin liberarlo nunca como disponible, se detiene el trabajo (`HALTED_RECONCILIATION_ERROR` si hubo sobrecoste `QUARANTINED_OVERRUN`, o `HALTED_QUARANTINE` si quedó bajo la reserva) y se lanza `BudgetReconciliationError`.
2. **Calibración del estimador de entrada y soporte de `explicit_max_output_tokens` (`budget_control.py` y `openai_provider.py`):**
   - `prepare_batch_request()` construye primero el `user_payload` JSON serializado y el `response_schema` JSON y los pasa a `estimate_batch()`.
   - `estimate_batch()` mide los tokens BPE reales del `user_payload` serializado + `instructions` + `response_schema` más la sobrecarga estructural del servidor (`max(350 + 8 * N, ceil(raw * 0.10))`), arrojando **`4,897` tokens de entrada estimados** para las 25 cadenas (frente a los **`4,694` tokens reales** reportados por OpenAI en el Intento 2).
   - Se eliminó el campo redundante `"translator_note"` del `json_schema` solicitado al modelo (`build_response_json_schema`), conservando únicamente los 4 campos obligatorios (`seq_id`, `string_uid`, `source_sha256`, `translated_masked_text`).
   - Con `explicit_max_output_tokens = 8192`, la reserva calculada antes del envío es de **`US$0.00527356`** (`US$0.00458570` base + `15%` margen), muy por debajo del límite de trabajo de `US$0.05` y del límite de proyecto de `US$0.25`.
3. **Preservación íntegra de la evidencia del Intento 1:**
   - No se borró ni alteró el trabajo `job_pilot_10_3a_luna25`, ni la reserva `res_job_pilot_10_3a_luna25_batch_pilot_10_3a_001_att1`, ni el archivo `pilot_10_3b_live_execution_summary.json`.
   - El Intento 2 se ejecutó bajo el identificador limpio y trazable `job_pilot_10_3b_luna25_att2` (`res_job_pilot_10_3b_luna25_att2_batch_pilot_10_3a_001_att1`).
4. **Suite de pruebas offline:**
   - Se añadieron las pruebas unitarias `test_09_incomplete_response_with_usage_records_actual_consumed_cost_and_overrun` y `test_10_incomplete_response_under_reservation_records_consumed_and_quarantines_remainder` en `tests/test_openai_budget_control.py`.
   - Resultado de la suite completa antes y después: **`61/61` pruebas en verde (`OK`) con `0` llamadas de red en los tests**.

---

## 3. Estado Previo al Envío (Intento 2 — `job_pilot_10_3b_luna25_att2`)

| Verificación Pre-Flight | Estado | Detalle Verificado |
| :--- | :---: | :--- |
| **A.1** Archivo externo de credenciales cargado | **CUMPLIDO** | `PRESENTE` (resuelto desde `%APPDATA%\NolvusAIT\secrets\openai-credentials.txt` sin copiar ni exponer secretos) |
| **A.2** `NOLVUS_OPENAI_API_KEY` | **PRESENTE** | Formato válido verificado en memoria; valor jamás impreso ni registrado |
| **A.3** `NOLVUS_OPENAI_PROJECT_ID` | **PRESENTE** | Formato válido verificado en memoria; valor jamás impreso ni registrado |
| **A.4** Cero exposición o copia de secretos | **CUMPLIDO** | Verificado por `CredentialRedactor.assert_no_credentials_present` |
| **A.5** Modelo exclusivo `gpt-6-luna` | **CUMPLIDO** | `allow_sol_escalation=False`, `allow_post_qa_sol_escalation=False`, `max_retries=0` |
| **A.6** Desbloqueo acotado y restauración de `PHASE_1_PAID_API_LOCK` | **CUMPLIDO** | Desbloqueado solo dentro de `_scoped_single_pilot_api_unlock()` y restaurado a `True` en `finally` |
| **A.7** Presupuesto disponible y evidencia previa preservada | **CUMPLIDO** | Proyecto: `US$0.25` (`US$0.00149259` en cuarentena del Intento 1; disponible inicial `US$0.24850741`). Trabajo `job_pilot_10_3b_luna25_att2`: `US$0.05` disponible |
| **A.8** Reserva creada antes de enviar la solicitud | **CUMPLIDO** | `res_job_pilot_10_3b_luna25_att2_batch_pilot_10_3a_001_att1` en estado `RESERVED_NOT_SENT` por `US$0.00527356` antes de abrir el socket HTTPS |
| **A.9** Conjunto aislado con exactamente 25 cadenas pendientes | **CUMPLIDO** | `25` filas con `review_status='unprocessed'` y `0` entradas en `translation_memory` antes del envío |
| **A.10** Base de datos aislada de staging exclusivamente | **CUMPLIDO** | `pilot_10_3a_work_db.sqlite3` (producción `workspace_data/nolvus_translator.db` intacta) |

---

## 4. Resultado de la Única Solicitud Enviada (Intento 2)

| Parámetro / Métrica | Valor Registrado |
| :--- | :--- |
| **Solicitudes HTTP reales enviadas en esta prueba** | `1` (`real_http_requests_count = 1`, `0` reintentos) |
| **Endpoint y método** | `POST https://api.openai.com/v1/responses` |
| **Modelo solicitado y confirmado** | `gpt-6-luna` |
| **Parámetros de control del payload** | `max_output_tokens = 8192`, `tools = []`, `tool_choice = "none"`, `store = false` |
| **Código de estado HTTP** | `200 OK` |
| **ID de respuesta (`response_id`)** | `resp_016c38d304293029016aea8f6856c887d2963d23486eaf69b9` |
| **Estado de la respuesta (`status`)** | `completed` |
| **Tokens de entrada estimados vs reales (`input_tokens`)** | Estimados: `4,897` \| Reales: **`4,694`** |
| **Tokens de salida estimados / tope vs reales (`output_tokens`)** | Estimados: `1,758` \| Tope (`max_output_tokens`): `8,192` \| Reales: **`3,056`** |
| **Tokens totales reales (`total_tokens`)** | **`7,750`** |

---

## 5. Conciliación Presupuestaria y Gasto Acumulado Auditable (Intentos 1 y 2)

### 5.1 Conciliación del Intento 2 (`job_pilot_10_3b_luna25_att2`)
- **ID de reserva:** `res_job_pilot_10_3b_luna25_att2_batch_pilot_10_3a_001_att1`
- **Estado inicial -> final de la reserva:** `RESERVED_NOT_SENT` -> `INTO_TRANSPORT` -> **`RECONCILED`**
- **Coste base estimado (con tope `8,192` salida):** `US$0.00458570` (`4,897` in × `$0.10/1M` + `8,192` out × `$0.50/1M`)
- **Monto reservado antes del envío (`+15%` margen):** **`US$0.00527356`**
- **Coste real facturado (`consumed_usd`):**
  $$\frac{4,694}{1,000,000} \times 0.10 + \frac{3,056}{1,000,000} \times 0.50 = 0.00046940 + 0.00152800 = \mathbf{US\$0.00199740}$$
- **Margen liberado tras conciliación (`released_usd`):** **`US$0.00327616`** (`US$0.00527356 - US$0.00199740`)
- **Delta frente al coste base estimado (`reconciliation_delta_usd`):** `-US$0.00258830` (ahorro respecto al tope reservado)
- **Saldo restante del trabajo (`job_remaining_usd` sobre `US$0.05`):** **`US$0.04800260`**

### 5.2 Balance acumulado del proyecto (`nolvus_es419_default` sobre `US$0.25`)

| Concepto | Intento 1 (`job_pilot_10_3a_luna25`) | Intento 2 (`job_pilot_10_3b_luna25_att2`) | Total Acumulado Proyecto |
| :--- | :---: | :---: | :---: |
| **`response_id`** | `resp_0a6ee1133950df8d016aea87823b9487d298c9ca44dbd0ad02` | `resp_016c38d304293029016aea8f6856c887d2963d23486eaf69b9` | 2 llamadas históricas (1 por autorización) |
| **Estado del registro en SQLite** | `QUARANTINED_UNKNOWN_COST` (intacto) | `RECONCILED` | `ACTIVE` |
| **Tokens reales (`in` / `out` / `total`)** | `4,693` / `2,198` / `6,891` | `4,694` / `3,056` / `7,750` | `9,387` / `5,254` / `14,641` |
| **Reserva original (`reserved_usd`)** | `US$0.00149259` | `US$0.00527356` | — |
| **En cuarentena en libro mayor (`quarantined_usd`)** | `US$0.00149259` | `US$0.00000000` | **`US$0.00149259`** |
| **Consumido conciliado en libro mayor (`consumed_usd`)** | `US$0.00000000` *(pre-fix)* | `US$0.00199740` | **`US$0.00199740`** |
| **Coste real por `usage` de OpenAI** | `US$0.00156830` *(exceso `+$0.00007571`)* | `US$0.00199740` | **`US$0.00356570`** |
| **Saldo disponible en `budget_projects` (`project_remaining_usd`)** | — | — | **`US$0.24651001`** *(o `US$0.24643430` descontando los `+$0.00007571` del Intento 1)* |

---

## 6. Validación por `LocalGatekeeper` e Importación en Base Aislada

### 6.1 Éxito técnico de ejecución e importación
- **Cadenas enviadas:** `25`
- **Cadenas recibidas de `gpt-6-luna`:** `25`
- **Veredicto de `LocalGatekeeper` (12 puertas duras):** `passed_all_gates = True`
  - **Cadenas aprobadas (`valid_candidates`):** **`25`** (`25/25`)
  - **Cadenas rechazadas / en cuarentena (`quarantine_records`):** **`0`**
  - **Integridad de placeholders y etiquetas (`⟦VAR_0⟧...⟦VAR_5⟧` -> `<Alias=QuestGiver>`, `<dur>`, `<mag>`, `\n{}\n`, `<font ...>`, `<p align='center'>`):** `25/25` restauradas de forma bit-a-bit exacta (`variables_intact = True`, `tags_balanced = True`).
- **Importación en base aislada (`pilot_10_3a_work_db.sqlite3`):**
  - `string_entries` actualizadas con traducción: **`25 / 25`** (`review_status = 'approved'`, `provider_id = 'openai_luna'`, `is_simulated = 0`).
  - `translation_memory` poblada en staging: **`25` entradas**.

### 6.2 Revisión lingüística de calidad (separada del éxito técnico)

El éxito técnico del pipeline fue de `25/25` cadenas recibidas, validadas e importadas en la base aislada. Al realizar la **revisión lingüística humana/experta** de las 25 traducciones devueltas por `gpt-6-luna` (ver tabla completa abajo), se distinguen **22 cadenas con traducción natural y precisa en ES-419** y **3 observaciones lingüísticas/terminológicas** que deben tenerse en cuenta antes de escalar a lotes mayores:

1. **Ítem #02 (`QUST:NNAM`, `Bring a Tempered Iron Dagger to <Alias=QuestGiver>` -> `Lleva una daga de hierro templado a <Alias=QuestGiver>`):**
   - El adjetivo *Tempered* modifica a la daga forjada/mejorada en la piedra de afilar (*daga de hierro templada* o *mejorada*), aunque en español también se usa *hierro templado*. Es comprensible y gramaticalmente válida, pero conviene fijar convención en glosario para armas mejoradas.
2. **Ítem #16 (`BOOK:DESC`, `Conjure Dremora Lord` -> `Invocar señor dremora`):**
   - La traducción literal es correcta y preserva intactas las etiquetas `<font face'$HandwrittenFont'><font size='40'><p align='center'>`, pero en la terminología oficial de Skyrim los grimorios y hechizos de conjuración suelen capitalizar el rango o usar la forma canónica (*Invocar a un señor dremora* / *Conjurar señor dremora*).
3. **Ítem #25 (`PEX`, `[Alternate Perspective]\nMissing JContainers...` -> `[Perspectiva alternativa]\nFalta JContainers...`):**
   - El texto dentro de corchetes `[Alternate Perspective]` es el nombre propio del mod de inicio alternativo (*Alternate Perspective*). El modelo lo tradujo como `[Perspectiva alternativa]`. Para mensajes técnicos de mods en scripts Papyrus, los nombres de mods entre corchetes `[Mod Name]` deben protegerse como tokens invariables o añadirse a la regla de preservación para no traducir el título del mod.

### 6.3 Tabla completa de las 25 cadenas traducidas en `pilot_10_3a_work_db.sqlite3`

| # | Categoría | UID | Texto Original (`source_text`) | Traducción Restaurada (`translated_text`) | Score QA |
| :---: | :--- | :--- | :--- | :--- | :---: |
| 01 | Diálogos/Misiones | `Relationship Dialogue Overhaul.esp:Skyrim.esm:00105385:QUST:NNAM:1` | `Bring an Iron Dagger to <Alias=QuestGiver>` | `Lleva una daga de hierro a <Alias=QuestGiver>` | `1.000` |
| 02 | Diálogos/Misiones | `Relationship Dialogue Overhaul.esp:Skyrim.esm:00105385:QUST:NNAM:3` | `Bring a Tempered Iron Dagger to <Alias=QuestGiver>` | `Lleva una daga de hierro templado a <Alias=QuestGiver>` | `1.000` |
| 03 | Diálogos/Misiones | `Unofficial Skyrim Special Edition Patch.esp:Skyrim.esm:00013118:INFO:NAM1:0` | `Wonderful! Remarkably well preserved too.` | `¡Maravilloso! También está increíblemente bien conservado.` | `1.000` |
| 04 | Diálogos/Misiones | `Unofficial Skyrim Special Edition Patch.esp:Skyrim.esm:0001362F:INFO:NAM1:0` | `I can't believe you did that. You people are monsters!` | `No puedo creer que hayas hecho eso. ¡Ustedes son unos monstruos!` | `1.000` |
| 05 | Diálogos/Misiones | `Unofficial Skyrim Special Edition Patch.esp:Skyrim.esm:0001362F:INFO:NAM1:1` | `You demand payment for protection and you can't even protect yourselves.` | `Exigen un pago por protección y ni siquiera pueden protegerse ustedes mismos.` | `1.000` |
| 06 | MCM e Interfaz | `MCM_TXT:interface/translations/mcmhelper_english.txt:$MCM_KeyAlreadyMapped{}` | `This key is already mapped to:\n{}\n\nAre you sure you want to continue?` | `Esta tecla ya está asignada a:\n{}\n\n¿Seguro que quieres continuar?` | `1.000` |
| 07 | MCM e Interfaz | `MCM_TXT:interface/translations/mcmhelper_english.txt:$MCM_KeyAlreadyMappedByMod{}{}` | `This key is already mapped to:\n{}\n({})\n\nAre you sure you want to continue?` | `Esta tecla ya está asignada a:\n{}\n({})\n\n¿Seguro que quieres continuar?` | `1.000` |
| 08 | MCM e Interfaz | `MCM_TXT:interface/translations/dynamiccollisionadjustment_english.txt:$DynamicCollisionAdjustment` | `Dynamic Collision Adjustment` | `Ajuste dinámico de colisiones` | `1.000` |
| 09 | MCM e Interfaz | `MCM_TXT:interface/translations/dynamiccollisionadjustment_english.txt:$DynamicCollisionAdjustment_General_HeaderText` | `General Settings` | `Configuración general` | `1.000` |
| 10 | MCM e Interfaz | `MCM_TXT:interface/translations/dynamiccollisionadjustment_english.txt:$DynamicCollisionAdjustment_EnableActorScaleFix_OptionText` | `Enable Actor Scale Fix` | `Activar corrección de escala de actores` | `0.940` |
| 11 | Objetos/Hechizos | `Unofficial Skyrim Special Edition Patch.esp:Skyrim.esm:000173DC:MGEF:FULL:0` | `Banish - Damage Health` | `Destierro: daño a la Salud` | `1.000` |
| 12 | Objetos/Hechizos | `YOT - Your Own Thoughts.esp:Skyrim.esm:00017121:MGEF:FULL:0` | `Shadowcloak of Nocturnal` | `Manto de sombras de Nocturnal` | `1.000` |
| 13 | Objetos/Hechizos | `YOT - Your Own Thoughts.esp:Skyrim.esm:00017121:MGEF:DNAM:0` | `For <dur> seconds the caster automatically becomes invisible while sneaking.` | `Durante <dur> segundos, el lanzador se vuelve invisible automáticamente al escabullirse.` | `1.000` |
| 14 | Objetos/Hechizos | `Unofficial Skyrim Special Edition Patch.esp:Skyrim.esm:00017120:MGEF:FULL:0` | `Fortify Armor Rating` | `Fortificar índice de armadura` | `1.000` |
| 15 | Objetos/Hechizos | `Unofficial Skyrim Special Edition Patch.esp:Skyrim.esm:00017120:MGEF:DNAM:0` | `Increases Armor Rating by <mag> points.` | `Aumenta el índice de armadura en <mag> puntos.` | `0.955` |
| 16 | Libros y Notas | `MysticismMagic.esp:Skyrim.esm:0010FD60:BOOK:DESC:0` | `<font face'$HandwrittenFont'><font size='40'><p align='center'>\r\nConjure Dremora Lord` | `<font face'$HandwrittenFont'><font size='40'><p align='center'>\r\nInvocar señor dremora` | `1.000` |
| 17 | Libros y Notas | `LegacyoftheDragonborn.esm:Skyrim.esm:0010F786:BOOK:DESC:0` | `...Human Heart / Black Soul Gem...` | `...Corazón humano / Gema de alma negra...` | `1.000` |
| 18 | Libros y Notas | `LegacyoftheDragonborn.esm:Skyrim.esm:0010F785:BOOK:DESC:0` | `...Human Skull / Daedra Heart / Beastflesh / Beastflesh...` | `...Cráneo humano / Corazón de daedra / Carne de bestia / Carne de bestia...` | `1.000` |
| 19 | Libros y Notas | `LegacyoftheDragonborn.esm:Skyrim.esm:0010F784:BOOK:DESC:0` | `...Ebony Ingot / Daedra Heart / Silver Sword / Greater Soul Gem Filled...` | `...Lingote de ébano / Corazón de daedra / Espada de plata / Gema de alma superior llena...` | `0.955` |
| 20 | Libros y Notas | `LegacyoftheDragonborn.esm:Skyrim.esm:0010F783:BOOK:DESC:0` | `...Ebony Ingot / Daedra Heart / Void Salts / Greater Soul Gem Filled...` | `...Lingote de ébano / Corazón de daedra / Sales del vacío / Gema de alma superior llena...` | `0.955` |
| 21 | Scripts y Otros | `PEX:scripts/refreshquestlogscript.pex:RefreshQuestLogScript::OnOptionHighlight:inst2:arg4` | `Click to start refreshing all active quest objectives.` | `Haz clic para comenzar a actualizar todos los objetivos de misiones activas.` | `1.000` |
| 22 | Scripts y Otros | `PEX:scripts/refreshquestlogscript.pex:RefreshQuestLogScript::OnOptionSelect:inst43:arg4` | `Starting quest journal refresh` | `Iniciando actualización del diario de misiones` | `1.000` |
| 23 | Scripts y Otros | `PEX:scripts/refreshquestlogscript.pex:RefreshQuestLogScript::OnOptionSelect:inst64:arg4` | `Quest journal refresh finished` | `Terminó la actualización del diario de misiones` | `1.000` |
| 24 | Scripts y Otros | `PEX:scripts/refreshquestlogscript.pex:RefreshQuestLogScript:busy:OnOptionHighlight:inst2:arg4` | `Refreshing all active quest objectives.` | `Actualizando todos los objetivos de misiones activas.` | `1.000` |
| 25 | Scripts y Otros | `PEX:scripts/apdialogueplayer.pex:APDialoguePlayer::MenuOpen:inst18:arg4` | `[Alternate Perspective]\nMissing JContainers. Only default start options will be available.` | `[Perspectiva alternativa]\nFalta JContainers. Solo estarán disponibles las opciones de inicio predeterminadas.` | `0.955` |

---

## 7. Garantías de Aislamiento y Seguridad de Producción

- **Candado global en reposo (`PHASE_1_PAID_API_LOCK`):** `True` (restaurado inmediatamente tras la llamada única).
- **Base de datos de producción (`workspace_data/nolvus_translator.db`):** Intacta (`431,386,624` bytes antes y después; `mtime_ns = 1793683151811389300` idéntico).
- **Configuración de producción (`workspace_data/config.json`):** Intacta (`active_provider = "mock"`).
- **Instalación Nolvus / STOCK GAME / MO2 / mods / partidas:** `0` archivos tocados.
- **Credenciales:** `0` secretos impresos, copiados o expuestos en logs o artefactos.
