# Hoja de ruta (`ROADMAP.md`) — Trabajo futuro y puertas pendientes

**Proyecto:** Nolvus AI Translator (ES-419)  
**Última actualización:** 2026-11-03  
**Estado vigente:** ver [PROJECT_STATUS.md](./PROJECT_STATUS.md) · **Historial ejecutado:** ver [CHANGELOG.md](./CHANGELOG.md) · **Decisiones:** ver [DECISIONS.md](./DECISIONS.md)

> Este documento contiene **solo trabajo futuro**. Las fases 0 a 10.1-B.1 y la consolidación técnica posterior ya están completadas y se registran en `CHANGELOG.md`. Nada de lo descrito aquí está autorizado hasta que el usuario lo apruebe expresamente fase por fase.

---

## Secuencia de puertas pendientes

| Fase | Alcance | Estado | Condición de salida |
|---|---|---|---|
| **10.2** | Preflight + smoke test aislado de 25 cadenas reales | `PENDIENTE` (requiere autorización) | 25/25 UIDs completos y válidos al cierre, cero defectos estructurales aceptados, cero accesos a producción y recuperación documentada ante una interrupción controlada. |
| **10.3** | Benchmark controlado: 300 cadenas × 3 modelos (`Gemini 4 Argon`, `Claude Sonnet 5.5`, `Claude Opus 5.5`) | `PENDIENTE` | Comparación ciega de calidad y métricas operativas según [BENCHMARK_PLAN_ANTIGRAVITY.md](./Fase10_Antigravity_Architecture/BENCHMARK_PLAN_ANTIGRAVITY.md). |
| **10.4** | Piloto de un mod completo representativo (~1.100–1.500 cadenas) | `PENDIENTE` | Coherencia terminológica y QA a escala de mod. |
| **10.5** | Continuidad y recuperación: 12–15 lotes consecutivos y fallos controlados | `PENDIENTE` | Sin pérdida silenciosa; recuperación medida y reproducible. |
| **10.6** | Decisión de escala | `PENDIENTE` | Decisión documentada en `DECISIONS.md`: Antigravity, híbrido o API, con datos medidos y nueva estimación de costes si aplica. |
| **Producción** | Traducción global (~240.684 cadenas únicas) y exportación del paquete MO2 | `PENDIENTE` (no autorizada) | Requiere superar 10.2–10.6 y una autorización expresa e independiente. |

---

## 10.2 — Preflight y smoke test de 25 cadenas

**Preflight obligatorio (antes de enviar cualquier solicitud a un modelo):**
1. Confirmar los modelos visibles en el selector de Antigravity el día de la prueba.
2. Confirmar **AI Credit Overages = Never**. Si no puede confirmarse, no ejecutar solicitudes.
3. Crear una copia SQLite de prueba (nunca `workspace_data/nolvus_translator.db`) y un workspace de intercambio externo y aislado; abrir **solo** esa carpeta en Antigravity.
4. Verificar permisos reales de lectura/escritura de Windows en el workspace.

**Composición propuesta de la muestra (25 cadenas):** 5 diálogos `INFO`, 5 libros con etiquetas HTML/`[pagebreak]`, 5 entradas MCM con `$claves`, 5 descripciones de hechizos/perks con `<mag>`/`<dur>`, 5 cadenas de misiones/scripts con `<Alias=...>`.

**Flujo con la CLI ya implementada:** `init-test-store` → `export-batch` → (Antigravity genera `02_OUTPUT/batch_<id>_output.jsonl`) → `validate-output` → `retry-quarantine` (si aplica) → `report`.

**Trabajo técnico previo recomendado (sin traducir):** generar un snapshot de prueba aislado con el código actual para que los cambios `F9-01`, `F9-02` y `F9-04` queden reflejados en las cadenas candidatas.

## 10.3 — Benchmark de 300 cadenas

Mismas entradas y contexto para los tres modelos en sesiones separadas. Medir calidad (fidelidad, lore, ES-419), tasa de aceptación a primera pasada del Gatekeeper, truncamientos, reintentos, tiempos, cuota observada e intervención humana. Los umbrales se fijan antes de ver resultados.

## 10.4 — Piloto de un mod completo

Probar un mod representativo tras aprobar modelo y protocolo. Verificar reutilización TM, consistencia del Glosario Maestro y exportación MO2 en un perfil clonado de prueba.

## 10.5 — Continuidad y recuperación

12–15 lotes consecutivos con interrupciones controladas (salida truncada, UID ausente, reinicio del proceso). Medir el ritmo sostenible real.

## 10.6 — Decisión de escala

Elegir Antigravity, híbrido o API únicamente con mediciones reales. La infraestructura API permanece bloqueada (`PHASE_1_PAID_API_LOCK = True`) hasta esa decisión.

---

## Invariantes de seguridad (aplican a todas las fases)

- No modificar `STOCK GAME`, mods originales, BSA, load order, perfiles MO2 ni partidas.
- No escribir desde el agente en la DB productiva, el Glosario Maestro, la Memoria de Traducción ni carpetas de exportación productiva.
- No instalar herramientas, descargar modelos locales, activar APIs, comprar créditos ni ejecutar traducciones masivas sin autorización expresa.
- Cada fase se detiene en su puerta de salida y entrega un informe antes de continuar.
