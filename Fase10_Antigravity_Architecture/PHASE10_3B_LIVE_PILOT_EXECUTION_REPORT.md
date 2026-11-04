# Informe Oficial de Ejecución — Piloto Real de 25 Cadenas con `gpt-6-luna` (Fase 10.3-B)

**Fecha de ejecución (UTC):** `2026-11-04T01:17:04Z`  
**Trabajo (`job_id`):** `job_pilot_10_3a_luna25`  
**Lote (`batch_id`):** `batch_pilot_10_3a_001`  
**Modelo utilizado:** `gpt-6-luna` (exclusivo; escalamiento a `gpt-6.1-sol` deshabilitado)  
**Endpoint invocado:** `POST https://api.openai.com/v1/responses`  
**ID de respuesta OpenAI (`api_response_id`):** `resp_0a6ee1133950df8d016aea87823b9487d298c9ca44dbd0ad02`  
**Solicitudes HTTP reales enviadas:** `1` (cero reintentos automáticos)  
**Estado posterior del candado global:** `PHASE_1_PAID_API_LOCK = True` (restaurado automáticamente tras la solicitud única)

---

## 1. Resultado de la Carga de Credenciales (Sin Valores Secretos)

La verificación se realizó a través de `SecureCredentialLoader.check_environment_presence_without_activating()` y `CredentialRedactor`, garantizando **cero exposición** de claves o identificadores en consola, logs, base de datos o informes:

| Parámetro / Variable | Ruta / Origen | Estado Detectado | Valores Secretos Expuestos / Copiados |
|---|---|---|---|
| Plantilla solicitada | `%LOCALAPPDATA%\NolvusAIT\secrets\openai-credentials.txt` | Resuelto a `%APPDATA%\NolvusAIT\secrets\openai-credentials.txt` | `0` |
| Archivo externo de credenciales | `%APPDATA%\NolvusAIT\secrets\openai-credentials.txt` | **`PRESENTE`** | `0` |
| `NOLVUS_OPENAI_API_KEY` | Archivo externo (`KEY=VALUE`) | **`PRESENTE`** | `0` |
| `NOLVUS_OPENAI_PROJECT_ID` | Archivo externo (`KEY=VALUE`) | **`PRESENTE`** | `0` |

---

## 2. Resultado de Cada Validación Previa Obligatoria (Sección A)

Antes de abrir cualquier conexión de red, `execute_authorized_pilot_25_once()` verificó las 10 condiciones obligatorias de la Sección A:

| # | Validación Previa (Sección A) | Resultado | Evidencia Técnica |
|---|---|---|---|
| **A.1** | Carga desde almacén externo de credenciales fuera del workspace | **`APROBADO` (`True`)** | Archivo detectado fuera de `<WORKSPACE_ROOT>` y sin symlinks (`has_symlink_or_reparse_point = False`). |
| **A.2** | Confirmar que `NOLVUS_OPENAI_API_KEY` está presente | **`APROBADO` (`True`)** | `api_key_configured = True` (`PRESENTE`). |
| **A.3** | Confirmar que `NOLVUS_OPENAI_PROJECT_ID` está presente | **`APROBADO` (`True`)** | `project_id_configured = True` (`PRESENTE`). |
| **A.4** | No mostrar, registrar ni copiar valores de credenciales | **`APROBADO` (`True`)** | `CredentialRedactor` activo; `0` copias locales o temporales creadas. |
| **A.5** | Confirmar que el modelo permitido es exclusivamente `gpt-6-luna` | **`APROBADO` (`True`)** | `allowed_model_id = "gpt-6-luna"`, `allow_sol_escalation = False`, `25/25` cadenas enrutadas a `gpt-6-luna`. |
| **A.6** | Confirmar que el API lock permite esta ejecución autorizada | **`APROBADO` (`True`)** | Contexto `_scoped_single_pilot_api_unlock()` habilitó la llamada única y restauró `PHASE_1_PAID_API_LOCK = True` en `finally`. |
| **A.7** | Confirmar presupuesto interno (`US$0.05` trabajo / `US$0.25` proyecto) | **`APROBADO` (`True`)** | `job_max_budget_usd = $0.05`, `project_total_budget_usd = $0.25`, saldos iniciales reservados/consumidos en `$0.00`. |
| **A.8** | Confirmar creación de reserva presupuestaria antes del envío HTTP | **`APROBADO` (`True`)** | Reserva `res_job_pilot_10_3a_luna25_batch_pilot_10_3a_001_att1` creada en estado `RESERVED_NOT_SENT` por `$0.001493 USD` (`$0.00149259`) antes de invocar `urlopen`. |
| **A.9** | Confirmar que el conjunto aislado contiene exactamente 25 cadenas pendientes | **`APROBADO` (`True`)** | `25` filas en `string_entries` con `review_status = 'unprocessed'` y `0` entradas en `translation_memory`. |
| **A.10** | Confirmar uso exclusivo de la base SQLite de staging del piloto | **`APROBADO` (`True`)** | `assert_safe_test_db_path()` verificado sobre `NolvusAIT_PilotStaging/Phase10_3A_Luna25/isolated_dbs/pilot_10_3a_work_db.sqlite3`. |

---

## 3. Confirmación de Envío de la Solicitud Única (Sección B)

- **¿Se envió la solicitud HTTP?** **SÍ (`http_request_sent = True`)**.
- **Número total de solicitudes HTTP enviadas:** **`1`** (`real_http_requests_count = 1`, cero reintentos).
- **Endpoint y método:** `POST https://api.openai.com/v1/responses`.
- **Parámetros verificados en el payload enviado:**
  - `model`: `"gpt-6-luna"`
  - `max_output_tokens`: `2198`
  - `tools`: `[]`
  - `tool_choice`: `"none"`
  - `store`: `false`
  - Reintentos automáticos: `0` (`max_retries = 0`, `max_retries_per_model = 0`)
  - Uso de Batch API: `No` (`0` trabajos Batch creados)
  - Uso de `gpt-6.1-sol` o modelos alternativos: `No`
- **Respuesta HTTP del servidor OpenAI:** `HTTP 200 OK`
- **ID de respuesta (`api_response_id`):** `resp_0a6ee1133950df8d016aea87823b9487d298c9ca44dbd0ad02`
- **Estado del objeto Response devuelto por OpenAI (`status`):** `"incomplete"` (la generación alcanzó el tope físico `max_output_tokens = 2,198` antes de cerrar el documento JSON estructurado).

---

## 4. Conteo de Cadenas (Enviadas, Recibidas, Aprobadas y Rechazadas)

| Métrica de Cadenas | Cantidad | Detalle |
|---|---:|---|
| **Cadenas pendientes en staging** | `25` | 5 categorías × 5 cadenas reales estratificadas |
| **Cadenas enviadas en la solicitud** | `25` | `seq_id = 1..25` con variables protegidas (`⟦VAR_n⟧`) |
| **Cadenas recibidas (JSON completo)** | `0` | La respuesta de OpenAI finalizó en `status = "incomplete"` al consumir los `2,198` tokens de salida permitidos |
| **Cadenas aprobadas por `LocalGatekeeper`** | `0` | `BudgetController` detuvo y puso en cuarentena el lote al detectar `status = "incomplete"` antes de importar |
| **Cadenas rechazadas / no importadas** | `25` | Las 25 cadenas permanecen intactas en estado `unprocessed` en la base aislada |
| **Cadenas importadas en DB aislada (`pilot_10_3a_work_db.sqlite3`)** | `0` | `translation_memory` permanece en `0` registros |
| **Cadenas importadas en DB oficial de producción** | `0` | Base oficial de producción estrictamente intacta |

---

## 5. Tokens de Entrada y Salida Reportados por OpenAI

| Concepto de Tokens | Estimación Previa Offline | Reportado Real por OpenAI (`usage`) | Diferencia / Observación |
|---|---:|---:|---|
| **Tokens de entrada (`input_tokens`)** | `1,989` | **`4,693`** | `+2,704` tokens (+135.9 %): incluye el sobrecoste real del servidor de OpenAI por la definición estricta de `json_schema` (`text.format`) y los metadatos por ítem (`source_sha256`, `glossary_context`, `placeholder_hints`). |
| **Tokens de salida (`output_tokens`)** | `1,758` (tope `2,198`) | **`2,198`** | Alcanzó exactamente el límite físico `max_output_tokens = 2,198` (100 % del tope autorizado), provocando `status = "incomplete"`. |
| **Tokens totales (`total_tokens`)** | `3,747` (máx `4,187`) | **`6,891`** | Suma real reportada por OpenAI (`4,693 + 2,198`). |

---

## 6. Coste Real Calculado según Uso Reportado por OpenAI

OpenAI reportó en el campo `usage` de la respuesta `resp_0a6ee1133950df8d016aea87823b9487d298c9ca44dbd0ad02` exactamente `input_tokens = 4,693` y `output_tokens = 2,198`. Aplicando la tarifa Standard configurada para `gpt-6-luna` (`US$0.10 / 1M` tokens de entrada y `US$0.50 / 1M` tokens de salida):

$$\text{Coste de entrada} = \frac{4{,}693 \times \$0.10}{1{,}000{,}000} = \$0.00046930\text{ USD}$$

$$\text{Coste de salida} = \frac{2{,}198 \times \$0.50}{1{,}000{,}000} = \$0.00109900\text{ USD}$$

$$\text{Coste total por tokens reportados} = \$0.00046930 + \$0.00109900 = \mathbf{\$0.00156830\text{ USD}}\text{ }(\approx \$0.001568\text{ USD})$$

> **Nota sobre la diferencia frente a la reserva de `US$0.001493`:**  
> La reserva previa (`$0.00149259 USD`, redondeada a `$0.001493 USD`) cubría exactamente el máximo de salida (`2,198` tokens = `$0.00109900 USD` base) más `1,989` tokens de entrada estimados (`$0.00019890 USD` base) con margen del `+15 %`. Como el tokenizador real de OpenAI contabilizó `4,693` tokens de entrada (`+$0.00027040 USD` sobre la estimación de entrada), el coste bruto calculado por tokens (`$0.00156830 USD`) superó en `$0.00007571 USD` (`7.57` milésimas de centavo) la reserva de `$0.00149259 USD`, manteniéndose muy por debajo del límite de trabajo de `US$0.05` (`3.14 %` del límite de trabajo).

---

## 7. Presupuesto Reservado, Consumido, En Cuarentena y Liberado

Al recibir `status = "incomplete"` desde OpenAI, `BudgetController.reconcile_reservation()` aplicó la regla de seguridad de la Fase 10.2 (Sección 5): **no liberar como saldo disponible una reserva cuya respuesta llegó incompleta**, transfiriendo el importe reservado a estado `QUARANTINED_UNKNOWN_COST` y deteniendo el trabajo en `HALTED_QUARANTINE`:

| Métrica Presupuestaria (`pilot_10_3a_budget_store.sqlite3`) | Importe (USD) | Estado / Detalle |
|---|---:|---|
| **Límite de presupuesto del proyecto (`project_total_budget_usd`)** | `$0.25000000` | Respetado |
| **Límite de presupuesto del trabajo (`job_max_budget_usd`)** | `$0.05000000` | Respetado |
| **Presupuesto reservado antes del envío (`reserved_usd`)** | **`$0.00149259`** (`~$0.001493`) | Creado en `RESERVED_NOT_SENT` y transicionado a `SENT_IN_FLIGHT` |
| **Presupuesto bloqueado en cuarentena (`quarantined_usd`)** | **`$0.00149259`** | Estado final de la reserva: `QUARANTINED_UNKNOWN_COST` (`status='incomplete'`) |
| **Presupuesto conciliado como completado (`consumed_usd`)** | **`$0.00000000`** | Coste real por tokens reportados (`4,693 in / 2,198 out`): `$0.00156830 USD` |
| **Presupuesto liberado (`released_usd`)** | **`$0.00000000`** | No se libera automáticamente porque la llamada sí se ejecutó en el servidor y devolvió `status='incomplete'` |
| **Saldo disponible restante del trabajo (`job_available_usd`)** | **`$0.04850741`** | Trabajo detenido de forma segura en estado `HALTED_QUARANTINE` |
| **Saldo disponible restante del proyecto (`project_available_usd`)** | **`$0.24850741`** | Proyecto en estado `ACTIVE` |

---

## 8. Errores y Advertencias Registrados

### Errores (1)
1. **`BudgetReconciliationError` (Bloqueo y cuarentena por respuesta incompleta en `max_output_tokens = 2,198`):**
   - Mensaje registrado:
     `BudgetReconciliationError: BLOQUEO Y CUARENTENA ECONÓMICA: Coste desconocido o respuesta incompleta (status='incomplete', input_tokens=4693, output_tokens=2198).`
   - **Causa técnica diagnosticada:** Con `max_output_tokens = 2,198`, el modelo `gpt-6-luna` en `POST /v1/responses` agotó los `2,198` tokens de salida antes de terminar de serializar los 25 objetos JSON del lote (cada objeto incluye `seq_id`, `string_uid`, `source_sha256` de 64 caracteres, `translated_masked_text` y `translator_note`, además de los tokens internos de razonamiento contabilizados dentro de `output_tokens` por la API Responses).
   - **Acción tomada por el sistema:** Siguiendo las reglas de la autorización (*"Si ocurre un timeout, error HTTP, 429 o error de conexión, detenerse. No repetir la solicitud"*), el sistema **no realizó ningún reintento**, puso la reserva en cuarentena auditable (`QUARANTINED_UNKNOWN_COST`), detuvo el trabajo (`HALTED_QUARANTINE`) y restauró `PHASE_1_PAID_API_LOCK = True`.

### Advertencias (1)
1. **Resolución transparente de ruta de credenciales en Windows AppData:**
   - La plantilla `%LOCALAPPDATA%\NolvusAIT\secrets\openai-credentials.txt` no existía en `AppData\Local`; `SecureCredentialLoader` resolvió automáticamente la ruta existente configurada en la tarea previa (`%APPDATA%\NolvusAIT\secrets\openai-credentials.txt`, en `AppData\Roaming`) sin copiar ni duplicar el archivo de secretos.

### Recomendación Técnica para el Próximo Intento (Cuando el Usuario lo Autorice)
- Para que las 25 cadenas completen su salida JSON estructurada en `POST /v1/responses` sin truncarse por `max_output_tokens`:
  1. Dividir el piloto de 25 cadenas en **5 micro-lotes de 5 cadenas** (o elevar `max_output_tokens` a `4,096–6,144` / configurar `reasoning={"effort": "low"}` o `"minimal"` si el modelo contabiliza tokens de razonamiento en `output_tokens`), y
  2. Ajustar el estimador local de tokens de entrada para incluir el factor real medido de `json_schema` + envoltorio (`~4,700` tokens de entrada para 25 ítems).

---

## 9. Confirmación de Integridad de Producción (Sección D)

Se certifica que **ningún recurso de producción ni del entorno del juego fue modificado**:

| Recurso Protegido | Estado Antes | Estado Después | Veredicto |
|---|---|---|---|
| Base oficial `workspace_data/nolvus_translator.db` | `431,386,624` bytes (`mtime_ns = 1793683151811389300`) | `431,386,624` bytes (`mtime_ns = 1793683151811389300`) | **`INTACTO` (0 bytes / 0 ns alterados)** |
| Glosario Maestro y TM de producción | Sin conexión de escritura | Sin conexión de escritura | **`INTACTO`** |
| `STOCK GAME` / Instalación funcional de Nolvus | Solo lectura | Solo lectura | **`INTACTO`** |
| Archivos originales de mods / Plugins / Load order | Solo lectura | Solo lectura | **`INTACTO`** |
| Perfiles de Mod Organizer 2 (MO2) y partidas guardadas | Solo lectura | Solo lectura | **`INTACTO`** |
| Candado global `PHASE_1_PAID_API_LOCK` | `True` (antes de ejecución) | `True` (restaurado al finalizar) | **`ACTIVO Y BLOQUEADO`** |
