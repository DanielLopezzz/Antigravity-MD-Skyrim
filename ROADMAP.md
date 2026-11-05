# Hoja de ruta (`ROADMAP.md`) — Trabajo futuro y puertas pendientes

**Proyecto:** Nolvus AI Translator (ES-419)  
**Última actualización:** 2026-11-04  
**Estado vigente:** ver [PROJECT_STATUS.md](./PROJECT_STATUS.md) · **Historial ejecutado:** ver [CHANGELOG.md](./CHANGELOG.md) · **Decisiones:** ver [DECISIONS.md](./DECISIONS.md)

> Este documento contiene **solo trabajo futuro**. La ejecución 10.3-B de 25 cadenas y el piloto 10.3-C de 125 cadenas ya ocurrieron y están registrados en `CHANGELOG.md` y sus informes. El piloto 10.3-C tuvo aceptación parcial: 100 cadenas importadas en staging, 23 candidatas retenidas por atomicidad y 2 en cuarentena. Nada de lo siguiente autoriza gasto real ni cambios en producción; cada ejecución facturable requiere aprobación expresa.

---

## Secuencia de puertas pendientes (Estrategia OpenAI API-First)

| Fase | Alcance | Estado | Condición de salida |
|---|---|---|---|
| **10.3-B** | **Ejecución real controlada de 25 cadenas con `gpt-6-luna`** | `COMPLETADO CON INCIDENCIA` | Dos intentos y su conciliación se documentan en el informe 10.3-B. No se considera puerta de producción. |
| **10.3-C** | **Piloto transversal real de 125 cadenas (`5 × 25`) con `gpt-6-luna`** | `COMPLETADO CON ACEPTACIÓN PARCIAL` | Cinco solicitudes, 100 importadas en staging, 23 candidatas retenidas por lote atómico, 2 en cuarentena; coste real `$0.011345 USD`. Informe 10.3-C. |
| **10.3-D / cuarta prueba** | **Evaluación humana de la muestra completada** | `PENDIENTE` | Revisar obligatorios y muestra sin alertas; medir gravedad y categorías de error antes de escalar. [Resultado completo](./Fase10_Evolucion_Linguistica/Pilot4_Completed_20261104/README.md). |
| **10.4** | **Evaluación de escalamiento selectivo a `gpt-6.1-sol` y prueba de Batch API (`/v1/batches`)** | `PENDIENTE` | Validación empírica de `OpenAIBatchCoordinator` en lote real asíncrono y verificación de reglas de escalamiento solo bajo autorización y condiciones comprobables. |
| **10.5** | **Piloto de un mod completo representativo (~1.100–1.500 cadenas)** | `PENDIENTE` | Coherencia terminológica ES-419, reutilización TM, propagación de duplicados, control presupuestario y exportación MO2 no destructiva verificada en perfil clonado de prueba. |
| **10.6** | **Recálculo económico empírico del corpus completo (~240.684 cadenas únicas) y puerta de aprobación de Producción** | `PENDIENTE` | Proyección definitiva de costos (`gpt-6-luna` Batch/Standard + tasa real medida de escalamiento a `gpt-6.1-sol`) documentada en `DECISIONS.md` y aprobación expresa de presupuesto productivo. |
| **Producción** | **Traducción global por oleadas y empaquetado del mod MO2 final** | `PENDIENTE` (no autorizada) | Requiere superar 10.3-B–10.6 y autorización expresa por oleada con presupuesto duro en `BudgetController`. |

---

## 10.3-D — Evaluación humana y cuarta prueba

**Prioridad actual:** revisar los casos obligatorios y la muestra de calidad de la continuación activa. La ejecución de las 1.000 entradas ya concluyó; no repetir las candidatas válidas. Separar aceptación mecánica, reparación posterior y aprobación humana. Ver el informe completo enlazado arriba.

- Completar las revisiones previas y decidir qué patrones exigen corrección. Si cambian prompt, glosario o código, regenerar el plan en otra carpeta; no modificar reglas durante el experimento.
- Reconfirmar modelo/tarifas y autorizar presupuesto antes de la primera etapa. Ejecutar por separado las etapas preparadas y evaluar las 50 iniciales antes de avanzar.
- Revisar los tres casos conocidos de `Elder Scroll` y `HeelsFix Refresh`, todas las alertas y la muestra de calidad. Comparar los 100 controles distinguiendo aprobación original de corrección humana.
- Elaborar recuentos por categoría y cohorte de significado, naturalidad, voz, terminología, contexto y formato. La aceptación mecánica no demuestra calidad lingüística.
- Auditar la clasificación general PEX antes de producción. El filtro de esta prueba evita los literales técnicos detectados, pero no reclasifica el inventario ni acredita cobertura completa.
- Conservar aceptación atómica y no reenviar solicitudes fallidas o inciertas sin reconciliación. Los controles ya implementados están en el [informe de preparación](./Fase10_Evolucion_Linguistica/Pilot4_Preparation_20261104/README.md).

## Antecedente histórico: piloto de 25 cadenas

### 10.3-B — Piloto real de 25 cadenas

**Estado previo ya completado en Fase 10.3-A ([PHASE10_3A_OFFLINE_PILOT_PREPARATION_REPORT.md](./Fase10_Antigravity_Architecture/PHASE10_3A_OFFLINE_PILOT_PREPARATION_REPORT.md)) y remediación preflight ([PHASE10_3B_PREFLIGHT_REMEDIATION_REPORT.md](./Fase10_Antigravity_Architecture/PHASE10_3B_PREFLIGHT_REMEDIATION_REPORT.md)):**
- Copia SQLite aislada lista (`isolated_dbs/pilot_10_3a_work_db.sqlite3`, `25` cadenas `unprocessed`, `1.538` caracteres de origen, `108` términos de glosario ES-419, `0` entradas TM).
- Bóveda de placeholders lista (`isolated_dbs/pilot_10_3a_exchange_store.sqlite3`, `25` sobres, `37` placeholders protegidos, `manifest_sha256` unificado).
- Almacén presupuestario aislado inicializado (`isolated_dbs/pilot_10_3a_budget_store.sqlite3`, `budget_projects = $0.25 USD ACTIVE`, `budget_jobs = $0.05 USD ACTIVE`, `reserved_usd = $0.00`, `consumed_usd = $0.00`).
- Payload `/v1/responses` listo (`staging_payloads/batch_batch_pilot_10_3a_001_openai_responses_request.json`, `1.989` tokens de entrada est., `1.758` salida esperada / `2.198` tope `max_output_tokens`; coste Standard esperado `$0.001078 USD` [`$0.001240 USD` con `+15 %`] y máximo `$0.001298 USD` [**`$0.001493 USD`** reserva oficial con `+15 %`]; coste Batch `-50 %` esperado `$0.000539 USD` [`$0.000620 USD` con `+15 %`] y máximo `$0.000649 USD` [`$0.000746 USD` con `+15 %`]).

**Preflight obligatorio antes de ejecutar la llamada real en Fase 10.3-B:**
1. Confirmar que el usuario ha autorizado expresamente el levantamiento controlado del candado `PHASE_1_PAID_API_LOCK` para `job_pilot_10_3a_luna25`.
2. Configurar las credenciales canónicas dedicadas del proyecto (`NOLVUS_OPENAI_API_KEY` y `NOLVUS_OPENAI_PROJECT_ID`) en el entorno local o almacén externo aislado, verificando el enmascaramiento de `CredentialRedactor`.
3. Operar exclusivamente sobre `pilot_10_3a_work_db.sqlite3`, `pilot_10_3a_exchange_store.sqlite3` y `pilot_10_3a_budget_store.sqlite3` (nunca `workspace_data/nolvus_translator.db`).
4. Mantener `allow_sol_escalation = False` durante toda la Fase 10.3-B para medir la tasa pura de aceptación a primera pasada de `gpt-6-luna` frente al `LocalGatekeeper` de 12 puertas.

## 10.4 — Validación de Batch API y escalamiento selectivo a `gpt-6.1-sol`

- Ejecutar un lote de prueba a través de `OpenAIBatchCoordinator` (`/v1/files` + `/v1/batches`) con `gpt-6-luna` para medir tiempos reales de cola asíncrona y verificar el descuento del 50 % en reconciliación post-respuesta.
- Si del piloto 10.3 resultan cadenas en cuarentena tras agotar reintentos en `gpt-6-luna` (`LUNA_MAX_RETRIES_EXHAUSTED`), evaluar con autorización expresa el escalamiento puntual a `gpt-6.1-sol`.

## 10.5 — Piloto de un mod completo

- Seleccionar un mod completo representativo (~1.100–1.500 cadenas), ejecutar su traducción completa bajo presupuesto acotado, validar la reutilización de la Memoria de Traducción y el Glosario Maestro ES-419, y generar el paquete de salida para MO2 en un entorno aislado.

## 10.6 — Recálculo económico empírico y puerta de Producción

- Con las métricas reales de tokens consumidos, tasa de aprobación a primera pasada en `gpt-6-luna` y porcentaje real de escalamiento a `gpt-6.1-sol`, actualizar la proyección económica total para las `~240.684` cadenas únicas antes de solicitar autorización para la traducción masiva por oleadas.

## Pendientes tras la evolución lingüística offline

Antes de ampliar el piloto, completar las puertas que siguen abiertas según el [estado de aceptación](./Fase10_Evolucion_Linguistica/Evolution_20261104/05_ESTADO_FINAL.md):

- **P1 — Calidad demostrada:** completar las revisiones humanas en curso, adjudicar discrepancias y preparar comparación A/B ciega. Todavía no hay métricas comparativas de mejora de modelos.
- **P1 — Terminología y contexto:** completar evidencia por entrada, obtener fuentes de lore consultables y evaluar segmentación/entidades/relevancia para recuperar pasajes locales de libros.
- **P2 — Cobertura real:** medir tamaños y memoria de parsers con archivos representativos, añadir pruebas de cabeceras/checksums y medir UI/exportación completa. El benchmark SQLite no cubre esos recorridos.
- **P2 — Operación y migración:** integrar la publicación reversible preparada con bloqueo y recuperación durable; llevar cursor a consultas necesarias y preparar migraciones generales de la DB principal sobre copias.

La implementación ya realizada está en [CHANGELOG.md](./CHANGELOG.md) y su paquete de evidencia. Estas tareas pendientes no habilitan llamadas facturables ni escrituras de producción.

---

## Invariantes de seguridad (aplican a todas las fases)

- **Las API solo traducen:** `tools=[]`, `tool_choice="none"`, `store=False`, sin acceso a Nolvus, MO2, sistema de archivos ni base de datos.
- **Control presupuestario duro:** Ninguna solicitud puede enviarse sin reserva previa atómica en `BudgetController` (tarifa Standard + margen `+15 %`).
- **Protección de fuentes:** No modificar `STOCK GAME`, mods originales, BSA, load order, perfiles MO2 ni partidas.
- **Cada fase se detiene en su puerta de salida** y requiere aprobación expresa del usuario antes de continuar.
