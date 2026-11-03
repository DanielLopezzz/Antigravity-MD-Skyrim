# FASE 10.3-A — INFORME TÉCNICO DE PREPARACIÓN OFFLINE DEL PRIMER PILOTO (`GPT-6 LUNA` · 25 CADENAS REALES)

**Proyecto:** Nolvus AI Translator  
**Instancia objetivo:** Nolvus Awakening 6.0.21  
**Idioma de destino:** Español latinoamericano neutro (`ES-419`)  
**Fase:** `10.3-A` — Preparación Técnica Offline del Primer Piloto (`REQUEST_CONSTRUCTION`)  
**Fecha:** 2026-11-03  
**Rama de implementación (repositorio de código):** `feat/phase10-2-openai-api-first`  
**Commit de implementación:** `d2b75a5` (`feat(pilot-10.3a): implement offline staging preparation for 25-string gpt-6-luna pilot and verify gatekeeper enforcement`)  
**Suite de pruebas automatizadas:** **49/49 pruebas en verde (`OK` en `5,682 s`)**  
**Llamadas HTTP a OpenAI ejecutadas:** **`0`** (`PHASE_1_PAID_API_LOCK = True`, `$0.000000 USD` gastados y `$0.000000 USD` reservados)

---

## 1. Resumen Ejecutivo y Cumplimiento de Controles Obligatorios

Se ha completado y verificado de extremo a extremo la preparación técnica **100 % offline** del primer lote piloto real de **25 cadenas** del corpus de **Nolvus Awakening 6.0.21** (`5` cadenas por cada una de las `5` categorías funcionales), listo para una futura ejecución mediante **OpenAI Responses API (`/v1/responses`)** con **`gpt-6-luna`** únicamente cuando el usuario autorice la **Fase 10.3-B**.

### 1.1 Verificación de los 10 controles obligatorios de Fase 10.3-A

| # | Control obligatorio | Estado verificado | Evidencia técnica (`staging_manifest_10_3a.json` y `test_pilot_offline_staging.py`) |
|---:|---|---|---|
| 1 | Utilizar **`gpt-6-luna`** como único modelo previsto | **CUMPLIDO** | `model_id = "gpt-6-luna"` en el lote `batch_pilot_10_3a_001` y en el payload `/v1/responses`. `ModelRouter` enrutó las `25/25` cadenas a `openai_luna` (`gpt-6-luna`). |
| 2 | Mantener **`PHASE_1_PAID_API_LOCK = True`** | **CUMPLIDO** | `paid_api_lock_active = True` verificado en `src/nolvus_translator/core/exceptions.py` y durante toda la ejecución de `prepare_pilot_10_3a_offline()`. |
| 3 | No cargar ni solicitar **`OPENAI_API_KEY`** | **CUMPLIDO** | `api_key_loaded_or_requested = False`. `OpenAICredentialManager` no se invocó en modo lectura de clave y el proceso se ejecutó con `OPENAI_API_KEY` ausente. |
| 4 | No realizar solicitudes HTTP (ni reales ni simuladas) | **CUMPLIDO** | `real_http_requests_count = 0`, `simulated_http_requests_count = 0`. La preparación se detuvo exactamente en `stage = "REQUEST_CONSTRUCTION"`. |
| 5 | No crear trabajos Batch remotos (`/v1/files`, `/v1/batches`) | **CUMPLIDO** | `batch_api_jobs_created = 0`. |
| 6 | No consumir créditos ni efectuar llamadas facturables | **CUMPLIDO** | `consumed_usd = $0.000000 USD`, `reserved_usd = $0.000000 USD` en `pilot_10_3a_budget_store.sqlite3`. |
| 7 | No habilitar **`gpt-6.1-sol`** (`GPT-6.1 Sol`) | **CUMPLIDO** | `sol_escalation_enabled = False` (`RoutingPolicy(allow_sol_escalation=False, allow_post_qa_sol_escalation=False)`). |
| 8 | No modificar la base de datos de producción (`workspace_data/nolvus_translator.db`) | **CUMPLIDO** | Abierta exclusivamente mediante URI SQLite de solo lectura inmutable (`file:...?mode=ro&immutable=1`). Tamaño inalterado (`431.386.624` bytes) y `mtime_ns` idéntico antes y después (`source_db_untouched = True`). |
| 9 | No modificar el Glosario Maestro ni la TM de producción | **CUMPLIDO** | La semilla del Glosario Maestro (`87` términos ES-419) se sembró únicamente dentro de la copia aislada `pilot_10_3a_work_db.sqlite3`. |
| 10 | No modificar MO2, Nolvus, `STOCK GAME`, mods ni partidas | **CUMPLIDO** | Todos los artefactos generados residen exclusivamente en el directorio de staging aislado `NolvusAIT_PilotStaging/Phase10_3A_Luna25/`. |

---

## 2. Lista Completa de las 25 Cadenas Seleccionadas (`5 × 5`)

Las 25 cadenas fueron extraídas en modo solo lectura de `workspace_data/nolvus_translator.db` aplicando estratificación determinista sobre las 5 categorías requeridas (`5` cadenas por categoría, **`0` duplicados por `string_uid`**, **`0` duplicados por `(source_sha256, context_signature)`**, **`1.538` caracteres totales de origen (`source_chars`)** y **`1.384` caracteres totales enmascarados (`source_masked`)**):

### 2.1 Categoría 1 — Diálogos y misiones (`dialogues_and_quests` · 5 cadenas)

| `#` | `string_uid` | Mod / Formato | Dominio / Registro | Firma de contexto | Chars | Hash `SHA-256` (prefijo) | Texto de origen (`source_text`) | Texto protegido (`masked_text`) | Placeholders / Glosario |
|---:|---|---|---|---|---:|---|---|---|---|
| **1** | `Relationship Dialogue Overhaul.esp:Skyrim.esm:00105385:QUST:NNAM:1` | Relationship Dialogue Overhaul (`esp`) | `quests` (`QUST:NNAM`) | `quests\|qust\|nnam\|` | `42` | `034218d466e1...` | `Bring an Iron Dagger to <Alias=QuestGiver>` | `Bring an Iron Dagger to ⟦VAR_0⟧` | `1` ph (`⟦VAR_0⟧` = `<Alias=QuestGiver>`) |
| **2** | `Relationship Dialogue Overhaul.esp:Skyrim.esm:00105385:QUST:NNAM:3` | Relationship Dialogue Overhaul (`esp`) | `quests` (`QUST:NNAM`) | `quests\|qust\|nnam\|` | `50` | `055ff8ef7381...` | `Bring a Tempered Iron Dagger to <Alias=QuestGiver>` | `Bring a Tempered Iron Dagger to ⟦VAR_0⟧` | `1` ph (`⟦VAR_0⟧` = `<Alias=QuestGiver>`) |
| **3** | `Unofficial Skyrim Special Edition Patch.esp:Skyrim.esm:00013118:INFO:NAM1:0` | Unofficial Skyrim Special Edition Patch (`esp`) | `dialogues` (`INFO:NAM1`) | `dialogues\|info\|nam1\|` | `41` | `c34a5be34a94...` | `Wonderful! Remarkably well preserved too.` | `Wonderful! Remarkably well preserved too.` | `0` ph |
| **4** | `Unofficial Skyrim Special Edition Patch.esp:Skyrim.esm:0001362F:INFO:NAM1:0` | Unofficial Skyrim Special Edition Patch (`esp`) | `dialogues` (`INFO:NAM1`) | `dialogues\|info\|nam1\|` | `54` | `0a827ab7164c...` | `I can't believe you did that. You people are monsters!` | `I can't believe you did that. You people are monsters!` | `0` ph |
| **5** | `Unofficial Skyrim Special Edition Patch.esp:Skyrim.esm:0001362F:INFO:NAM1:1` | Unofficial Skyrim Special Edition Patch (`esp`) | `dialogues` (`INFO:NAM1`) | `dialogues\|info\|nam1\|` | `72` | `e36cccb97c25...` | `You demand payment for protection and you can't even protect yourselves.` | `You demand payment for protection and you can't even protect yourselves.` | `0` ph |

### 2.2 Categoría 2 — MCM e interfaz (`mcm_and_ui` · 5 cadenas)

| `#` | `string_uid` | Mod / Formato | Dominio / Clave | Firma de contexto | Chars | Hash `SHA-256` (prefijo) | Texto de origen (`source_text`) | Texto protegido (`masked_text`) | Placeholders / Glosario |
|---:|---|---|---|---|---:|---|---|---|---|
| **6** | `MCM_TXT:interface/translations/mcmhelper_english.txt:$MCM_KeyAlreadyMapped{}` | MCM Helper (`mcm_txt`) | `mcm_menus` (`$MCM_KeyAlreadyMapped{}`) | `mcm_menus\|\|\|` | `72` | `d84b23dc985c...` | `This key is already mapped to:\n{}\n\nAre you sure you want to continue?` | `This key is already mapped to:⟦VAR_0⟧⟦VAR_1⟧⟦VAR_2⟧⟦VAR_3⟧Are you sure you want to continue?` | `4` ph (`3×\n`, `1×{}`) |
| **7** | `MCM_TXT:interface/translations/mcmhelper_english.txt:$MCM_KeyAlreadyMappedByMod{}{}` | MCM Helper (`mcm_txt`) | `mcm_menus` (`$MCM_KeyAlreadyMappedByMod{}{}`) | `mcm_menus\|\|\|` | `78` | `643dfcb0b350...` | `This key is already mapped to:\n{}\n({})\n\nAre you sure you want to continue?` | `This key is already mapped to:⟦VAR_0⟧⟦VAR_1⟧⟦VAR_2⟧(⟦VAR_3⟧)⟦VAR_4⟧⟦VAR_5⟧Are you sure you want to continue?` | `6` ph (`4×\n`, `2×{}`) |
| **8** | `MCM_TXT:interface/translations/dynamiccollisionadjustment_english.txt:$DynamicCollisionAdjustment` | Dynamic Collision Adjustment (`mcm_txt`) | `mcm_menus` (`$DynamicCollisionAdjustment`) | `mcm_menus\|\|\|` | `28` | `4ab5abd87993...` | `Dynamic Collision Adjustment` | `Dynamic Collision Adjustment` | `0` ph |
| **9** | `MCM_TXT:interface/translations/dynamiccollisionadjustment_english.txt:$DynamicCollisionAdjustment_General_HeaderText` | Dynamic Collision Adjustment (`mcm_txt`) | `mcm_menus` (`$..._General_HeaderText`) | `mcm_menus\|\|\|` | `16` | `de322f87b5f5...` | `General Settings` | `General Settings` | `0` ph |
| **10** | `MCM_TXT:interface/translations/dynamiccollisionadjustment_english.txt:$DynamicCollisionAdjustment_EnableActorScaleFix_OptionText` | Dynamic Collision Adjustment (`mcm_txt`) | `mcm_menus` (`$..._EnableActorScaleFix_OptionText`) | `mcm_menus\|\|\|` | `22` | `5074f35f6105...` | `Enable Actor Scale Fix` | `Enable Actor Scale Fix` | `0` ph |

### 2.3 Categoría 3 — Objetos, hechizos y ventajas (`items_spells_perks` · 5 cadenas)

| `#` | `string_uid` | Mod / Formato | Dominio / Registro | Firma de contexto | Chars | Hash `SHA-256` (prefijo) | Texto de origen (`source_text`) | Texto protegido (`masked_text`) | Placeholders / Glosario |
|---:|---|---|---|---|---:|---|---|---|---|
| **11** | `Unofficial Skyrim Special Edition Patch.esp:Skyrim.esm:000173DC:MGEF:FULL:0` | Unofficial Skyrim Special Edition Patch (`esp`) | `plugins` (`MGEF:FULL`) | `plugins\|mgef\|full\|` | `22` | `fc0e2af322c8...` | `Banish - Damage Health` | `Banish - Damage Health` | `0` ph · Glosario: `Health -> Salud` |
| **12** | `YOT - Your Own Thoughts.esp:Skyrim.esm:00017121:MGEF:FULL:0` | YOT - Your Own Thoughts (`esp`) | `plugins` (`MGEF:FULL`) | `plugins\|mgef\|full\|` | `24` | `f15b8de65891...` | `Shadowcloak of Nocturnal` | `Shadowcloak of Nocturnal` | `0` ph · Glosario: `Nocturnal -> Nocturnal` (`keep_original`) |
| **13** | `YOT - Your Own Thoughts.esp:Skyrim.esm:00017121:MGEF:DNAM:0` | YOT - Your Own Thoughts (`esp`) | `plugins` (`MGEF:DNAM`) | `plugins\|mgef\|dnam\|` | `76` | `df5b207c5ac1...` | `For <dur> seconds the caster automatically becomes invisible while sneaking.` | `For ⟦VAR_0⟧ seconds the caster automatically becomes invisible while sneaking.` | `1` ph (`⟦VAR_0⟧` = `<dur>`) |
| **14** | `Unofficial Skyrim Special Edition Patch.esp:Skyrim.esm:00017120:MGEF:FULL:0` | Unofficial Skyrim Special Edition Patch (`esp`) | `plugins` (`MGEF:FULL`) | `plugins\|mgef\|full\|` | `20` | `db7940dc4f2a...` | `Fortify Armor Rating` | `Fortify Armor Rating` | `0` ph |
| **15** | `Unofficial Skyrim Special Edition Patch.esp:Skyrim.esm:00017120:MGEF:DNAM:0` | Unofficial Skyrim Special Edition Patch (`esp`) | `plugins` (`MGEF:DNAM`) | `plugins\|mgef\|dnam\|` | `39` | `f016518ec9a4...` | `Increases Armor Rating by <mag> points.` | `Increases Armor Rating by ⟦VAR_0⟧ points.` | `1` ph (`⟦VAR_0⟧` = `<mag>`) |

### 2.4 Categoría 4 — Libros y notas (`books_and_notes` · 5 cadenas)

| `#` | `string_uid` | Mod / Formato | Dominio / Registro | Firma de contexto | Chars | Hash `SHA-256` (prefijo) | Texto de origen (`source_text` resumido) | Texto protegido (`masked_text` resumido) | Placeholders / Glosario |
|---:|---|---|---|---|---:|---|---|---|---|
| **16** | `MysticismMagic.esp:Skyrim.esm:0010FD60:BOOK:DESC:0` | Mysticism - A Magic Overhaul (`esp`) | `books` (`BOOK:DESC`) | `books\|book\|desc\|` | `85` | `bdf69b9a670d...` | `<font face'$HandwrittenFont'><font size='40'><p align='center'> \n Conjure Dremora Lord` | `⟦VAR_0⟧⟦VAR_1⟧⟦VAR_2⟧ \n Conjure Dremora Lord` | `3` ph HTML (`<font...>`, `<font...>`, `<p...>`) |
| **17** | `LegacyoftheDragonborn.esm:Skyrim.esm:0010F786:BOOK:DESC:0` | Legacy of the Dragonborn (`esm`) | `books` (`BOOK:DESC`) | `books\|book\|desc\|` | `112` | `c0ecb6be09ee...` | `<p align='center'><font face='$DaedricFont'><font size='31'> ... Human Heart ... Black Soul Gem ... </font></p>` | `⟦VAR_0⟧⟦VAR_1⟧⟦VAR_2⟧ ... Human Heart ... Black Soul Gem ... ⟦VAR_3⟧⟦VAR_4⟧` | `5` ph HTML · Glosario: `Soul Gem -> Gema de alma` |
| **18** | `LegacyoftheDragonborn.esm:Skyrim.esm:0010F785:BOOK:DESC:0` | Legacy of the Dragonborn (`esm`) | `books` (`BOOK:DESC`) | `books\|book\|desc\|` | `138` | `28abc4db6f8d...` | `<p align='center'><font face='$DaedricFont'><font size='31'> ... Human Skull ... Daedra Heart ... Beastflesh ... </font></p>` | `⟦VAR_0⟧⟦VAR_1⟧⟦VAR_2⟧ ... Human Skull ... Daedra Heart ... Beastflesh ... ⟦VAR_3⟧⟦VAR_4⟧` | `5` ph HTML |
| **19** | `LegacyoftheDragonborn.esm:Skyrim.esm:0010F784:BOOK:DESC:0` | Legacy of the Dragonborn (`esm`) | `books` (`BOOK:DESC`) | `books\|book\|desc\|` | `153` | `57f5c3ec18e1...` | `<p align='center'><font face='$DaedricFont'><font size='31'> ... Ebony Ingot ... Daedra Heart ... Silver Sword ... Black Soul Gem ... </font></p>` | `⟦VAR_0⟧⟦VAR_1⟧⟦VAR_2⟧ ... Ebony Ingot ... Daedra Heart ... Silver Sword ... ⟦VAR_3⟧⟦VAR_4⟧` | `5` ph HTML · Glosario: `Soul Gem -> Gema de alma` |
| **20** | `LegacyoftheDragonborn.esm:Skyrim.esm:0010F783:BOOK:DESC:0` | Legacy of the Dragonborn (`esm`) | `books` (`BOOK:DESC`) | `books\|book\|desc\|` | `151` | `3a81bc30fd59...` | `<p align='center'><font face='$DaedricFont'><font size='31'> ... Ebony Ingot ... Daedra Heart ... Void Salts ... Greater Soul Gem ... </font></p>` | `⟦VAR_0⟧⟦VAR_1⟧⟦VAR_2⟧ ... Ebony Ingot ... Daedra Heart ... Void Salts ... ⟦VAR_3⟧⟦VAR_4⟧` | `5` ph HTML · Glosario: `Soul Gem -> Gema de alma` |

### 2.5 Categoría 5 — Scripts y otros (`scripts_and_other` · 5 cadenas)

| `#` | `string_uid` | Mod / Formato | Dominio / Función | Firma de contexto | Chars | Hash `SHA-256` (prefijo) | Texto de origen (`source_text`) | Texto protegido (`masked_text`) | Placeholders / Glosario |
|---:|---|---|---|---|---:|---|---|---|---|
| **21** | `PEX:scripts/refreshquestlogscript.pex:RefreshQuestLogScript::OnOptionHighlight:inst2:arg4` | Quest Journal Limit Bug Fixer (`pex`) | `scripts` (`OnOptionHighlight`) | `scripts\|\|\|` | `54` | `f74c4dbfce7b...` | `Click to start refreshing all active quest objectives.` | `Click to start refreshing all active quest objectives.` | `0` ph |
| **22** | `PEX:scripts/refreshquestlogscript.pex:RefreshQuestLogScript::OnOptionSelect:inst43:arg4` | Quest Journal Limit Bug Fixer (`pex`) | `scripts` (`OnOptionSelect`) | `scripts\|\|\|` | `30` | `a8085d940597...` | `Starting quest journal refresh` | `Starting quest journal refresh` | `0` ph |
| **23** | `PEX:scripts/refreshquestlogscript.pex:RefreshQuestLogScript::OnOptionSelect:inst64:arg4` | Quest Journal Limit Bug Fixer (`pex`) | `scripts` (`OnOptionSelect`) | `scripts\|\|\|` | `30` | `17c4424531c1...` | `Quest journal refresh finished` | `Quest journal refresh finished` | `0` ph |
| **24** | `PEX:scripts/refreshquestlogscript.pex:RefreshQuestLogScript:busy:OnOptionHighlight:inst2:arg4` | Quest Journal Limit Bug Fixer (`pex`) | `scripts` (`OnOptionHighlight`) | `scripts\|\|\|` | `39` | `b839001119a3...` | `Refreshing all active quest objectives.` | `Refreshing all active quest objectives.` | `0` ph |
| **25** | `PEX:scripts/apdialogueplayer.pex:APDialoguePlayer::MenuOpen:inst18:arg4` | Alternate Perspective - Alternate Start (`pex`) | `scripts` (`MenuOpen`) | `scripts\|\|\|` | `90` | `ce53714b4ea2...` | `[Alternate Perspective] \n Missing JContainers. Only default start options will be available.` | `[Alternate Perspective] \n Missing JContainers. Only default start options will be available.` | `0` ph |

### 2.6 Recuento verificado de caracteres por categoría (Manifiesto JSON vs. JSONL de entrada)

Recalculado directamente desde `staging_manifest_10_3a.json` (`items[*].source_chars`), `pilot_10_3a_work_db.sqlite3` (`len(source_text)`) y `batch_batch_pilot_10_3a_001_input.jsonl` (`len(source_masked)`):

| Categoría (`pilot_category`) | Cadenas (`seq_id`) | Desglose `source_chars` por ítem | Total `source_chars` (origen) | Total `source_masked` (JSONL protegido) |
|---|---|---|---:|---:|
| `dialogues_and_quests` | `#1` a `#5` | `42 + 50 + 41 + 54 + 72` | `259` | `237` |
| `mcm_and_ui` | `#6` a `#10` | `72 + 78 + 28 + 16 + 22` | `216` | `266` |
| `items_spells_perks` | `#11` a `#15` | `22 + 24 + 76 + 20 + 39` | `181` | `185` |
| `books_and_notes` | `#16` a `#20` | `85 + 112 + 138 + 153 + 151` | `639` | `453` |
| `scripts_and_other` | `#21` a `#25` | `54 + 30 + 30 + 39 + 90` | `243` | `243` |
| **Total exacto (`25` cadenas)** | **`#1` a `#25`** | **Suma directa de `items[*].source_chars`** | **`1.538`** | **`1.384`** |

> **Nota de reconciliación de caracteres:** En la primera edición del informe se consignó `1.576` como total de caracteres porque se habían contabilizado los `38` saltos de línea con su escape visual de dos caracteres (`\n` → `\\n`, `1.538 + 38 = 1.576`). La suma directa de los 25 valores `source_chars` en `staging_manifest_10_3a.json` y `len(source_text)` en `pilot_10_3a_work_db.sqlite3` es exactamente **`1.538` caracteres de origen** (y **`1.384` caracteres** en el campo `source_masked` de `batch_batch_pilot_10_3a_001_input.jsonl`).

---

## 3. Resumen de Placeholders Protegidos y Glosario ES-419 Aplicado

### 3.1 Distribución de los 37 placeholders protegidos (`VariableProtector`)

En total se enmascararon **37 tokens técnicos** distribuidos en **11 de las 25 cadenas** (`14` cadenas de texto limpio sin variables y `11` cadenas con tokens protegidos mediante marcadores canónicos `⟦VAR_0⟧..⟦VAR_n⟧`).

Durante esta preparación se mejoró expresamente la expresión regular `brace_placeholder` en `src/nolvus_translator/quality/protector.py` (`r"\{[A-Za-z0-9_.:%\-]*\}"`) para proteger llaves posicionales vacías `{}` empleadas por **MCM Helper** (`$MCM_KeyAlreadyMapped{}` y `$MCM_KeyAlreadyMappedByMod{}{}`), además de llaves con índice/nombre (`{0}`, `{name}`).

| Familia de placeholder (`kind`) | Cantidad protegida | Ejemplos reales en el lote `batch_pilot_10_3a_001` |
|---|---:|---|
| `html_xml_tag` | `23` | `<p align='center'>`, `<font face='$DaedricFont'>`, `<font face'$HandwrittenFont'>`, `<font size='31'>`, `<font size='40'>`, `</font>`, `</p>` |
| `escaped_control` | `7` | `\n` en cadenas MCM UTF-16LE (`$MCM_KeyAlreadyMapped{}` y `$MCM_KeyAlreadyMappedByMod{}{}`) |
| `brace_placeholder` | `3` | `{}` posicionales dinámicos en `MCM Helper` |
| `skyrim_alias_global` | `2` | `<Alias=QuestGiver>` en objetivos `QUST:NNAM` |
| `skyrim_magic_token` | `2` | `<dur>` y `<mag>` en descripciones de efectos mágicos `MGEF:DNAM` |
| **Total de tokens en bóveda (`PlaceholderVault`)** | **`37`** | Almacenados exclusivamente en la tabla `placeholder_vault` de `pilot_10_3a_exchange_store.sqlite3` (`25` sobres sellados con `vault_checksum` SHA-256). |

### 3.2 Subconjunto de Glosario Maestro ES-419 inyectado en el lote

El motor de glosario (`GlossaryManager.find_matching_terms_in_texts`) cruzó las 25 cadenas contra los **87 términos oficiales ES-419** sembrados en la base de trabajo aislada y seleccionó automáticamente el subconjunto exacto aplicable al lote (`3` reglas activas en `5` cadenas):

| Término origen (`source_term`) | Traducción aprobada (`approved_translation`) | Regla (`translation_rule`) | `keep_original` | Categoría | Cadenas donde aplica (`seq_id`) |
|---|---|---|---|---|---|
| `Health` | `Salud` | `use_official_translation` | `False` | `magic_system` | `#11` (`Banish - Damage Health`) |
| `Nocturnal` | `Nocturnal` | `preserve_untranslated` | `True` | `religion_cult` | `#12` (`Shadowcloak of Nocturnal`) |
| `Soul Gem` | `Gema de alma` | `use_official_translation` | `False` | `skyrim_term` | `#17`, `#19`, `#20` (`Black Soul Gem`, `Greater Soul Gem`) |

---

## 4. Construcción y Verificación del Payload de Responses API en Staging Local

El payload exacto para **OpenAI Responses API (`/v1/responses`)** fue construido en modo offline mediante `GPT6LunaAdapter.build_responses_request()` y serializado en `NolvusAIT_PilotStaging/Phase10_3A_Luna25/staging_payloads/batch_batch_pilot_10_3a_001_openai_responses_request.json` (`SHA-256: aed55a99e06ff44c69c1fadafd5be07e94a1dcb8e0da8b572b7fb7588cea66b5`).

### 4.1 Propiedades estructurales y de seguridad verificadas sobre el payload

| Campo del payload `/v1/responses` | Valor verificado | Garantía técnica |
|---|---|---|
| `model` | `"gpt-6-luna"` | Único modelo previsto y permitido en la Fase 10.3-A. |
| `tools` | `[]` (lista vacía) | Cumple la regla «Las API solo traducen»: prohíbe invocación de herramientas, navegación web o ejecución de código. |
| `tool_choice` | `"none"` | Bloquea cualquier selección implícita de herramientas. |
| `store` | `False` | Desactiva almacenamiento innecesario del lado del proveedor. |
| `temperature` | `0.1` | Temperatura determinista baja para preservar etiquetas, marcadores `⟦VAR_n⟧` y consistencia terminológica. |
| `max_output_tokens` | `2198` | Tope duro físico de salida calculado por `BudgetController.estimate_batch()` para impedir desbordamientos de generación. |
| `text.format` | `json_schema` (`strict: True`, `name: "nolvus_es419_translation_batch"`) | Obliga al modelo a devolver exclusivamente un objeto JSON válido con `batch_id`, `contract_version` y la lista `items` (`seq_id`, `string_uid`, `translated_text`). |
| `metadata` | `{"batch_id": "batch_pilot_10_3a_001", "job_id": "job_pilot_10_3a_luna25", "project": "nolvus_ai_translator_es419", "task": "translation_only"}` | Trazabilidad biyectiva del lote sin exponer rutas del sistema ni datos sensibles. |
| `assert_translation_only_payload()` | **APROBADO (`True`)** | Inspección recursiva de todas las cadenas hoja del payload verificando ausencia total de rutas locales (`C:\...`, `STOCK GAME`, `ModOrganizer.ini`, `.db`) y ausencia del mapa de restauración de placeholders. |

---

## 5. Cálculo de Tokens y Estimación Económica (Tarifa Standard y Batch `-50 %`, Escenarios Esperado y Máximo)

### 5.1 Desglose de tokens de entrada y salida (`batch_pilot_10_3a_001`)

| Métrica de tokens (`batch_pilot_10_3a_001`) | Valor exacto | Notas de cálculo |
|---|---:|---|
| Cantidad de cadenas en el lote (`item_count`) | `25` | `5` categorías × `5` cadenas (`1.538` caracteres totales de origen; `1.384` caracteres enmascarados en JSONL) |
| Tokens de cadenas de origen (`source_strings_tokens`) | `564` | Texto enmascarado con `⟦VAR_n⟧` |
| Tokens de instrucciones del sistema ES-419 (`instructions_tokens`) | `465` | Reglas de español neutro ES-419, prohibición de voseo/vosotros, preservación de `⟦VAR_n⟧` |
| Tokens de subconjunto de glosario y firmas de contexto (`glossary_and_context_tokens`) | `40` | `3` términos de glosario activos + metadatos de registro (`QUST:NNAM`, `MGEF:DNAM`, etc.) |
| Tokens de enmarcado JSON y esquema de respuesta (`json_framing_input_tokens`) | `920` | Estructura JSONL de entrada + definición `json_schema` estricta |
| **Total estimado de tokens de entrada (`estimated_input_tokens`)** | **`1.989`** | Suma total de entrada (`564 + 465 + 40 + 920`) |
| **Tokens de salida en escenario esperado (`estimated_output_tokens`)** | **`1.758`** | Expansión lingüística EN→ES-419 (`1.22x`) + envoltura JSON de salida |
| **Tokens de salida en escenario máximo (`max_output_tokens_cap`)** | **`2.198`** | Tope duro físico inyectado en `max_output_tokens` del payload |

### 5.2 Comparativa completa de costes y reservas (`+15 %`) en ambos escenarios (Standard vs. Batch `-50 %`)

Tarifas de referencia para **`gpt-6-luna`**:
- **Tarifa Standard (`/v1/responses` síncrono):** Entrada **`$0.10 USD / 1M`** tokens · Salida **`$0.50 USD / 1M`** tokens.
- **Tarifa Batch API (`/v1/batches` asíncrono con descuento del `50 %`):** Entrada **`$0.05 USD / 1M`** tokens · Salida **`$0.25 USD / 1M`** tokens.
- **Margen de seguridad del `BudgetController`:** **`+15 %` (`× 1.15`)**. Por política de seguridad (`D-20`), `BudgetController` realiza la reserva previa atómica tomando siempre el **Escenario Máximo con Tarifa Standard + 15 % (`$0.001493 USD`)**, sin asumir descuentos Batch.

| Modalidad y escenario | Tokens de entrada | Tokens de salida | Fórmula coste base | Coste base (`USD`) | Reserva con margen `+15 %` (`× 1.15`, `USD`) |
|---|---:|---:|---|---:|---:|
| **Standard — Escenario esperado** | `1.989` | `1.758` | `(1.989 × $0,10 + 1.758 × $0,50) / 1M` | **`$0.001078 USD`** (`$0.00107790`) | **`$0.001240 USD`** (`$0.00123959`) |
| **Standard — Escenario máximo (tope `max_output_tokens`)** | `1.989` | `2.198` | `(1.989 × $0,10 + 2.198 × $0,50) / 1M` | **`$0.001298 USD`** (`$0.00129790`) | **`$0.001493 USD`** (`$0.00149259`) *(reserva oficial `BudgetController`)* |
| **Batch (`-50 %`) — Escenario esperado** | `1.989` | `1.758` | `(1.989 × $0,05 + 1.758 × $0,25) / 1M` | **`$0.000539 USD`** (`$0.00053895`) | **`$0.000620 USD`** (`$0.00061979` / `$0.00061980`) |
| **Batch (`-50 %`) — Escenario máximo (tope `max_output_tokens`)** | `1.989` | `2.198` | `(1.989 × $0,05 + 2.198 × $0,25) / 1M` | **`$0.000649 USD`** (`$0.00064895`) | **`$0.000746 USD`** (`$0.00074629`) |

| Control presupuestario de referencia (todavía no autorizado) | Importe (`USD`) | Estado de verificación |
|---|---:|---|
| Presupuesto máximo de referencia por trabajo (`reference_max_job_budget_usd`) | `$0.050000 USD` | `within_reference_job_budget = True` (el techo máximo Standard con `+15 %` de `$0.001493 USD` representa solo el **`2,99 %`** del tope de `$0.05 USD`) |
| Presupuesto interno de referencia del proyecto (`reference_project_budget_usd`) | `$0.250000 USD` | Reserva actual en base de datos: **`$0.000000 USD`** · Consumido: **`$0.000000 USD`** |

---

## 6. Estado e Integridad de la Copia de Trabajo Aislada

Todos los artefactos para el futuro piloto quedaron preparados y verificados en el directorio aislado `NolvusAIT_PilotStaging/Phase10_3A_Luna25/` (excluido de Git mediante `.gitignore`):

| Artefacto aislado en `NolvusAIT_PilotStaging/Phase10_3A_Luna25/` | Rol técnico | Integridad SQLite / Formato | Hash `SHA-256` completo |
|---|---|---|---|
| `isolated_dbs/pilot_10_3a_work_db.sqlite3` | Copia SQLite de trabajo aislada con `scan_snapshots` (`1`), `scanned_mods` (`8`), `scanned_files` (`9`), `string_entries` (`25` en estado `unprocessed`), `glossary_terms` (`87` términos oficiales ES-419) y `translation_memory` (`0` entradas iniciales). | `PRAGMA integrity_check = ok` | `f6d5baec6f5b83892a2976d3062a08f1c863590e629a024d088d198428fcf00f` |
| `isolated_dbs/pilot_10_3a_exchange_store.sqlite3` | Almacén SQLite independiente de intercambio (`ExchangeStateStore`) con el trabajo `job_pilot_10_3a_luna25`, el lote `batch_pilot_10_3a_001` en estado `AWAITING_AGENT` y los **25 sobres de bóveda** (`placeholder_vault`) con los `37` placeholders protegidos. | `PRAGMA integrity_check = ok` | `8f1f0010118ef0cd60b97c3e2f1979090218db3b81ea5c3eb6eed71189dc7b90` |
| `isolated_dbs/pilot_10_3a_budget_store.sqlite3` | Almacén SQLite del `BudgetController` inicializado con los topes de referencia (`$0.05 USD` / `$0.25 USD`), `$0.000000 USD` reservados y `$0.000000 USD` consumidos. | `PRAGMA integrity_check = ok` | `df5210bfd247fe3bb7af2c375329e24d54ca6d656e3fbbc7a96e0e21a03aa36a` |
| `staging_payloads/batch_batch_pilot_10_3a_001_meta.json` | Manifiesto versionado (`1.0.0`) del lote exportado para `gpt-6-luna`. | JSON válido (`UTF-8`) | `f036d564398542aca64bc66b7dcffd0a606d758cf3df9dc717cf6129a284a4e7` |
| `staging_payloads/batch_batch_pilot_10_3a_001_input.jsonl` | Las 25 líneas `InputBatchItemV1` con texto enmascarado (`⟦VAR_n⟧`), hash de origen y glosario por ítem (`1.384` caracteres en `source_masked`). | JSONL válido (`25` líneas) | `22493486b8f8b274517ca4196815727d82cfbf17261c41a8dea92a69d76b5306` |
| `staging_payloads/batch_batch_pilot_10_3a_001_openai_responses_request.json` | Payload completo de **OpenAI Responses API (`/v1/responses`)** listo para envío futuro. | JSON válido (`UTF-8`) | `aed55a99e06ff44c69c1fadafd5be07e94a1dcb8e0da8b572b7fb7588cea66b5` |
| `staging_payloads/staging_manifest_10_3a.json` | Auditoría completa JSON con controles, métricas de tokens/coste, hashes e inventario de las 25 cadenas. | JSON válido (`UTF-8`) | Generado en cada corrida determinista |

---

## 7. Verificación del Paso Obligatorio por `LocalGatekeeper` antes de la Memoria de Traducción

Para demostrar formalmente el requisito 10 («Verificar que todas las respuestas futuras deberán pasar por `LocalGatekeeper` antes de poder incorporarse a la TM») sin ensuciar la copia de trabajo preparada, `prepare_pilot_10_3a_offline()` clona `pilot_10_3a_work_db.sqlite3` y `pilot_10_3a_exchange_store.sqlite3` en un directorio temporal efímero y ejecuta tres pruebas de barrera reales sobre las 25 cadenas del lote:

1. **Bloqueo de importación directa sin validación del Gatekeeper (`unvalidated_direct_import_blocked = True`):**
   - Cualquier intento de llamar a `AntigravityExchangeService.import_validated_exchange_results()` mientras el lote está en `AWAITING_AGENT` (sin haber superado `LocalGatekeeper`) lanza `ExchangeStateTransitionError` y deja `0` filas en `translation_memory`.
2. **Rechazo y cuarentena ante corrupción de placeholders o violación ES-419 (`corrupted_output_rejected_by_gatekeeper = True`):**
   - Al inyectar una respuesta donde el ítem `#1` omite el placeholder `⟦VAR_0⟧` (`<Alias=QuestGiver>`), `LocalGatekeeper` rechaza la respuesta en la puerta de restauración (`GATE_09_PLACEHOLDER_RESTORATION`), envía el registro a `quarantine_items`, bloquea la importación del lote y mantiene **`tm_rows_after_corrupted_attempt = 0`** en `translation_memory`.
3. **Aceptación atómica tras superar las 12 puertas deterministas (`valid_output_passed_12_gates = True`):**
   - Al validar un lote que preserva los `37` placeholders `⟦VAR_n⟧`, respeta los términos obligatorios del Glosario Maestro (`Salud`, `Nocturnal`, `Gema de alma`) y cumple las reglas ES-419, `LocalGatekeeper` aprueba los `25/25` ítems, restaura los `37` tokens originales desde `PlaceholderVault` (`<Alias=QuestGiver>`, `{}`, `\n`, `<dur>`, `<mag>`, etiquetas HTML `<font...>/<p...>`) e incorpora las **`25` entradas en `translation_memory`** (`tm_rows_after_valid_import = 25`).
   - Mientras tanto, la base aislada oficial `pilot_10_3a_work_db.sqlite3` permanece intacta con **`0` entradas en `translation_memory`** y las **`25` cadenas en estado `unprocessed`**, lista para el piloto real.

---

## 8. Lista de Comprobación Previa a una Futura Fase 10.3-B (Ejecución Real Controlada)

El proyecto se detiene en este punto. Para pasar en el futuro a la **Fase 10.3-B (ejecución real del lote `batch_pilot_10_3a_001` con `gpt-6-luna`)** se requerirá únicamente:

- [x] **Lote real de 25 cadenas (`5 × 5`) extraído y deduplicado en copia SQLite aislada** (`pilot_10_3a_work_db.sqlite3`).
- [x] **37 placeholders protegidos en bóveda local** (`pilot_10_3a_exchange_store.sqlite3`) incluyendo llaves vacías `{}` de MCM Helper.
- [x] **Subconjunto de Glosario Maestro ES-419 e instrucciones del sistema inyectados** en el payload `/v1/responses`.
- [x] **Payload `/v1/responses` verificado** (`model="gpt-6-luna"`, `tools=[]`, `tool_choice="none"`, `store=False`, `max_output_tokens=2198`).
- [x] **Coste máximo calculado con tarifa Standard + margen `+15 %`:** **`$0.001493 USD`** (dentro del presupuesto de referencia de `$0.05 USD` por trabajo).
- [x] **Paso obligatorio por `LocalGatekeeper` (12 puertas) verificado** antes de cualquier escritura en `translation_memory`.
- [x] **Suite completa de pruebas en verde:** **`49/49 OK`** (`tests/test_pilot_offline_staging.py` incluida).
- [ ] **Pendiente de decisión del usuario (Puerta Fase 10.3-B):** Autorización explícita para habilitar el presupuesto (`max_job_budget_usd = $0.05 USD`), configurar `OPENAI_API_KEY` y enviar la solicitud preparada `staging_payloads/batch_batch_pilot_10_3a_001_openai_responses_request.json` a `gpt-6-luna`.
