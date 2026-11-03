# Hoja de ruta (`ROADMAP.md`) — Trabajo futuro y puertas pendientes

**Proyecto:** Nolvus AI Translator (ES-419)  
**Última actualización:** 2026-11-03  
**Estado vigente:** ver [PROJECT_STATUS.md](./PROJECT_STATUS.md) · **Historial ejecutado:** ver [CHANGELOG.md](./CHANGELOG.md) · **Decisiones:** ver [DECISIONS.md](./DECISIONS.md)

> Este documento contiene **solo trabajo futuro**. Las fases 0 a 10.3-A (incluidas la implementación de la infraestructura OpenAI API-First y la preparación offline completa del primer piloto de 25 cadenas reales con 49/49 pruebas en verde) ya están completadas y se registran en `CHANGELOG.md`. Nada de lo descrito aquí está autorizado para generar gasto real hasta que el usuario lo apruebe expresamente fase por fase.

---

## Secuencia de puertas pendientes (Estrategia OpenAI API-First)

| Fase | Alcance | Estado | Condición de salida |
|---|---|---|---|
| **10.3-B** | **Ejecución real controlada del primer piloto con `gpt-6-luna`** (lote `batch_pilot_10_3a_001` de 25 cadenas reales `5×5` ya preparado en staging local sobre `pilot_10_3a_work_db.sqlite3`, coste máximo con margen `+15 %`: **`$0.001493 USD`**) | `PENDIENTE` (requiere autorización presupuestaria explícita) | 100 % de los 25 UIDs reconciliados, tasa de aceptación a primera pasada del `LocalGatekeeper` medida empíricamente, reconciliación exacta de `BudgetController` dentro del tope autorizado (`$0.05 USD`) y `0` defectos estructurales aceptados. |
| **10.3-C** | **Ampliación opcional del piloto a 100 cadenas (`4 × 25`) con `gpt-6-luna`** | `PENDIENTE` | Confirmación de estabilidad estadística de calidad ES-419 y consumo en 4 lotes consecutivos bajo el mismo tope de `$0.05 USD`. |
| **10.4** | **Evaluación de escalamiento selectivo a `gpt-6.1-sol` y prueba de Batch API (`/v1/batches`)** | `PENDIENTE` | Validación empírica de `OpenAIBatchCoordinator` en lote real asíncrono y verificación de las reglas de escalamiento a `gpt-6.1-sol` únicamente sobre ítems en cuarentena tras intentos en `gpt-6-luna`. |
| **10.5** | **Piloto de un mod completo representativo (~1.100–1.500 cadenas)** | `PENDIENTE` | Coherencia terminológica ES-419, reutilización TM, propagación de duplicados, control presupuestario y exportación MO2 no destructiva verificada en perfil clonado de prueba. |
| **10.6** | **Recálculo económico empírico del corpus completo (~240.684 cadenas únicas) y puerta de aprobación de Producción** | `PENDIENTE` | Proyección definitiva de costos (`gpt-6-luna` Batch/Standard + tasa real medida de escalamiento a `gpt-6.1-sol`) documentada en `DECISIONS.md` y aprobación expresa de presupuesto productivo. |
| **Producción** | **Traducción global por oleadas y empaquetado del mod MO2 final** | `PENDIENTE` (no autorizada) | Requiere superar 10.3-B–10.6 y autorización expresa por oleada con presupuesto duro en `BudgetController`. |

---

## 10.3-B — Ejecución real del lote preparado `batch_pilot_10_3a_001` (`25` cadenas reales) con `gpt-6-luna`

**Estado previo ya completado en Fase 10.3-A ([PHASE10_3A_OFFLINE_PILOT_PREPARATION_REPORT.md](./Fase10_Antigravity_Architecture/PHASE10_3A_OFFLINE_PILOT_PREPARATION_REPORT.md)):**
- Copia SQLite aislada lista (`pilot_10_3a_work_db.sqlite3`, `25` cadenas `unprocessed`, `87` términos de glosario ES-419, `0` entradas TM).
- Bóveda de placeholders lista (`pilot_10_3a_exchange_store.sqlite3`, `25` sobres, `37` placeholders protegidos).
- Payload `/v1/responses` listo (`batch_pilot_10_3a_001_responses_request.json`, `1.989` tokens de entrada est., `max_output_tokens = 2.198`, coste máximo Standard `$0.001298 USD`, coste máximo con margen `+15 %`: **`$0.001493 USD`**).

**Preflight obligatorio antes de ejecutar la llamada real en Fase 10.3-B:**
1. Confirmar que el usuario ha autorizado expresamente el presupuesto del trabajo (`max_job_budget_usd = $0.05 USD`, `project_budget_usd = $0.25 USD`) y el levantamiento controlado del candado para `job_pilot_10_3a_luna25`.
2. Configurar la clave de API dedicada del proyecto (`OPENAI_API_KEY`) en el entorno local con permisos restringidos a `gpt-6-luna`, verificando el enmascaramiento de `CredentialRedactor`.
3. Operar exclusivamente sobre `pilot_10_3a_work_db.sqlite3`, `pilot_10_3a_exchange_store.sqlite3` y `pilot_10_3a_budget_store.sqlite3` (nunca `workspace_data/nolvus_translator.db`).
4. Mantener `allow_sol_escalation = False` durante toda la Fase 10.3-B para medir la tasa pura de aceptación a primera pasada de `gpt-6-luna` frente al `LocalGatekeeper` de 12 puertas.

## 10.4 — Validación de Batch API y escalamiento selectivo a `gpt-6.1-sol`

- Ejecutar un lote de prueba a través de `OpenAIBatchCoordinator` (`/v1/files` + `/v1/batches`) con `gpt-6-luna` para medir tiempos reales de cola asíncrona y verificar el descuento del 50 % en reconciliación post-respuesta.
- Si del piloto 10.3 resultan cadenas en cuarentena tras agotar reintentos en `gpt-6-luna` (`LUNA_MAX_RETRIES_EXHAUSTED`), evaluar con autorización expresa el escalamiento puntual a `gpt-6.1-sol`.

## 10.5 — Piloto de un mod completo

- Seleccionar un mod completo representativo (~1.100–1.500 cadenas), ejecutar su traducción completa bajo presupuesto acotado, validar la reutilización de la Memoria de Traducción y el Glosario Maestro ES-419, y generar el paquete de salida para MO2 en un entorno aislado.

## 10.6 — Recálculo económico empírico y puerta de Producción

- Con las métricas reales de tokens consumidos, tasa de aprobación a primera pasada en `gpt-6-luna` y porcentaje real de escalamiento a `gpt-6.1-sol`, actualizar la proyección económica total para las `~240.684` cadenas únicas antes de solicitar autorización para la traducción masiva por oleadas.

---

## Invariantes de seguridad (aplican a todas las fases)

- **Las API solo traducen:** `tools=[]`, `tool_choice="none"`, `store=False`, sin acceso a Nolvus, MO2, sistema de archivos ni base de datos.
- **Control presupuestario duro:** Ninguna solicitud puede enviarse sin reserva previa atómica en `BudgetController` (tarifa Standard + margen `+15 %`).
- **Protección de fuentes:** No modificar `STOCK GAME`, mods originales, BSA, load order, perfiles MO2 ni partidas.
- **Cada fase se detiene en su puerta de salida** y requiere aprobación expresa del usuario antes de continuar.
