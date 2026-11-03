# Hoja de ruta (`ROADMAP.md`) — Trabajo futuro y puertas pendientes

**Proyecto:** Nolvus AI Translator (ES-419)  
**Última actualización:** 2026-11-03  
**Estado vigente:** ver [PROJECT_STATUS.md](./PROJECT_STATUS.md) · **Historial ejecutado:** ver [CHANGELOG.md](./CHANGELOG.md) · **Decisiones:** ver [DECISIONS.md](./DECISIONS.md)

> Este documento contiene **solo trabajo futuro**. Las fases 0 a 10.2 (incluida la implementación y verificación completa de la infraestructura OpenAI API-First con 48/48 pruebas en verde) ya están completadas y se registran en `CHANGELOG.md`. Nada de lo descrito aquí está autorizado para generar gasto real hasta que el usuario lo apruebe expresamente fase por fase.

---

## Secuencia de puertas pendientes (Estrategia OpenAI API-First)

| Fase | Alcance | Estado | Condición de salida |
|---|---|---|---|
| **10.3** | **Primer piloto real controlado con `gpt-6-luna`** (25 a 100 cadenas reales estratificadas en lotes de 25 sobre copia SQLite aislada) | `PENDIENTE` (requiere autorización presupuestaria explícita) | 100 % de UIDs reconciliados, tasa de aceptación a primera pasada del `LocalGatekeeper` medida empíricamente, reconciliación exacta de `BudgetController` dentro del tope autorizado (`$0.05 USD`) y `0` defectos estructurales aceptados. |
| **10.4** | **Evaluación de escalamiento selectivo a `gpt-6.1-sol` y prueba de Batch API (`/v1/batches`)** | `PENDIENTE` | Validación empírica de `OpenAIBatchCoordinator` en lote real asíncrono y verificación de las reglas de escalamiento a `gpt-6.1-sol` únicamente sobre ítems en cuarentena tras intentos en `gpt-6-luna`. |
| **10.5** | **Piloto de un mod completo representativo (~1.100–1.500 cadenas)** | `PENDIENTE` | Coherencia terminológica ES-419, reutilización TM, propagación de duplicados, control presupuestario y exportación MO2 no destructiva verificada en perfil clonado de prueba. |
| **10.6** | **Recálculo económico empírico del corpus completo (~240.684 cadenas únicas) y puerta de aprobación de Producción** | `PENDIENTE` | Proyección definitiva de costos (`gpt-6-luna` Batch/Standard + tasa real medida de escalamiento a `gpt-6.1-sol`) documentada en `DECISIONS.md` y aprobación expresa de presupuesto productivo. |
| **Producción** | **Traducción global por oleadas y empaquetado del mod MO2 final** | `PENDIENTE` (no autorizada) | Requiere superar 10.3–10.6 y autorización expresa por oleada con presupuesto duro en `BudgetController`. |

---

## 10.3 — Propuesta concreta para el primer piloto real con `gpt-6-luna`

**Preflight obligatorio antes de ejecutar la primera llamada real:**
1. Confirmar que el usuario ha autorizado expresamente el presupuesto del trabajo (`max_job_budget_usd = $0.05 USD`, `project_budget_usd = $0.25 USD`).
2. Configurar la clave de API dedicada del proyecto (`OPENAI_API_KEY`) en el entorno local con permisos restringidos a `gpt-6-luna` y `gpt-6.1-sol`, verificando el enmascaramiento de `CredentialRedactor`.
3. Trabajar exclusivamente sobre una copia SQLite aislada de prueba (nunca `workspace_data/nolvus_translator.db`).
4. Mantener `allow_sol_escalation = False` durante el piloto inicial de `gpt-6-luna` para medir la tasa pura de aceptación a primera pasada de Luna.

**Opciones de tamaño medidas offline sobre el corpus real (`pilot_estimator.py`):**
- **Escalón 1 (Micro-piloto de 25 cadenas / 1 lote):** `1.877` tokens de entrada est., `1.628` tokens de salida est. (`2.035` tope físico), costo máximo Standard `$0.001205 USD`, reserva atómica requerida (`+15 %`) **`$0.001386 USD`**.
- **Escalón 2 (Piloto recomendado de 100 cadenas / 4 lotes de 25):** `7.711` tokens de entrada est., `6.829` tokens de salida est. (`8.539` tope físico), costo máximo Standard `$0.005041 USD`, reserva atómica requerida (`+15 %`) **`$0.005797 USD`** (~`0,58` centavos de dólar).
- **Escalón 3 (Piloto ampliado de 250 cadenas / 10 lotes de 25):** `20.573` tokens de entrada est., `18.781` tokens de salida est. (`23.480` tope físico), costo máximo Standard `$0.013797 USD`, reserva atómica requerida (`+15 %`) **`$0.015867 USD`** (~`1,59` centavos de dólar).

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
