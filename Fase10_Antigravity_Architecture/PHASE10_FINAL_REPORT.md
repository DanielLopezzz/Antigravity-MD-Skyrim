# INFORME TÉCNICO FINAL — FASE 10: ARQUITECTURA DE TRADUCCIÓN CON ANTIGRAVITY Y DISEÑO DUAL
**Documento:** `PHASE10_FINAL_REPORT.md`  
**Fase:** Fase 10 — Análisis, Rediseño de Benchmark y Arquitectura Independiente del Proveedor  
**Fecha de Emisión:** `2026-11-03`  
**Estado:** Diseño documental completo — **validación operativa pendiente**  
**Restricciones Cumplidas:** `0` traducciones ejecutadas, `0` llamadas a APIs, `0` créditos consumidos, `0` modificaciones a `C:\Nolvus`, `STOCK GAME`, `MO2`, `workspace_data/nolvus_translator.db`, al snapshot de Fase 9 o a informes previos.

---

## 1. RESUMEN EJECUTIVO DE LO ANALIZADO

En esta **Fase 10** se ha auditado la arquitectura completa del código fuente actual de **Nolvus AI Translator** ([src/nolvus_translator/](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator)), el plan de benchmark previo ([MODEL_BENCHMARK_PLAN.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/MODEL_BENCHMARK_PLAN.md)), la estimación económica V2 ([API_COST_ESTIMATE_V2.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/API_COST_ESTIMATE_V2.md)) y los resultados definitivos de cobertura de la Fase 9 ([PHASE9_FULL_AUDIT_REPORT.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/PHASE9_FULL_AUDIT_REPORT.md)), con el objetivo de:
1. Priorizar el uso de los tres modelos disponibles en el selector actual de **Google Antigravity** bajo la suscripción del usuario (**`Gemini 4 Argon`**, **`Claude Sonnet 5.5`** y **`Claude Opus 5.5`**, excluyendo `GPT-6 Luna` y `GPT-6 Sol`).
2. Diseñar un mecanismo de integración seguro, auditable y resiliente a fallos entre `Nolvus AI Translator` y el agente de Antigravity sin utilizar APIs externas ni automatización frágil de clics de interfaz.
3. Mantener una **Arquitectura Dual Independiente del Proveedor** donde toda la infraestructura de APIs futuras permanece conservada, compatible e intercambiable en caliente sin reconstruir el proyecto ni perder una sola cadena ya validada.

---

## 2. MATRIZ EPISTEMOLÓGICA: HECHOS COMPROBADOS VS HIPÓTESIS Y PENDIENTES (SECCIÓN 11)

En cumplimiento estricto de la **Sección 11**, se diferencia con total transparencia la naturaleza de cada afirmación contenida en este diseño:

| Categoría Epistemológica | Elementos Concretos en Este Informe |
|---|---|
| **1. Resultados reportados por las inspecciones locales de Fase 9 y Fase 10 (no revalidados desde los artefactos fuente en este repositorio)** | • El volumen real consolidado tras la Fase 9 es de **`240.244` cadenas únicas en el Corte C** (`328.485` ocurrencias, incluyendo los `3.090` `.pex` con texto dentro de `.bsa`) más **`~440` cadenas únicas narrativas** del Subgrupo 4A de `JSON_OTHER` (*The Dragonborn's Bestiary*, *Alternate Perspective*, *Custom Skill Menu*) = **`~240.684` cadenas únicas netas pendientes**.<br>• Existen **`88.241` ocurrencias duplicadas** (`40.305` por parches de conflicto entre plugins) que la deduplicación por `source_hash` evita traducir dos veces.<br>• Existen **`38` cadenas en `3DNPC.esp`** con apóstrofes Windows-1252 mal codificados de origen (`\ufffd'`) y **`8.667` ocurrencias de nombres propios puros** (`protected_proper_noun`) que se resuelven localmente sin IA.<br>• En el código actual de `src/nolvus_translator/`, el informe de Fase 10 reporta como implementados y operativos (11/11 tests en verde; resultado no reejecutado durante esta revisión): `DatabaseManager` (SQLite WAL + FTS5), `GlossaryManager`, `TranslationMemoryEngine`, `ContextEngine`, `VariableProtector`, `TranslationQualityValidator`, `Mo2ModPackager` y el puerto abstracto `TranslationProvider`.<br>• Los adaptadores de proveedores externos en [future_stubs.py](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/translation/future_stubs.py) son actualmente *stubs* bloqueados por `PHASE_1_PAID_API_LOCK = True`. |
| **2. Capacidades Observadas en Google Antigravity** | • El agente de Antigravity lee y escribe archivos directamente en el workspace (`c:\Skyrim proyecto traducción`) con `read_file` y `write_file`, y ejecuta comandos de validación con `.venv\Scripts\python.exe` dentro del sandbox estándar (`BypassSandbox: false`).<br>• El usuario cuenta en su selector con **`Gemini 4 Argon`**, **`Claude Sonnet 5.5`** y **`Claude Opus 5.5`**.<br>• Los procesos en segundo plano o en memoria del agente se detienen si se reinicia el servidor de Antigravity, mientras que los archivos ya escritos en disco permanecen disponibles según el registro de esa prueba; la integridad de cualquier salida parcial debe validarse. |
| **3. Información Documental** | • La guía interna de Antigravity (`antigravity_guide`) documenta la existencia de *Skills* de proyecto, comandos slash y un paquete PyPI `google-antigravity` (no instalado actualmente en `.venv`). |
| **4. Hipótesis (A contrastar en el Benchmark)** | • **Hipótesis H1:** `Claude Opus 5.5` y `Claude Sonnet 5.5` ofrecerán una fluidez literaria muy alta en libros de lore (`BOOK:DESC`) y diálogos complejos en `ES-419`, mientras que `Gemini 4 Argon` o `Claude Sonnet 5.5` ofrecerán la mayor velocidad operativa en lotes estructurados.<br>• **Hipótesis H2:** El tamaño de lote óptimo sin degradación de atención ni truncamiento se situará entre `50` y `80` cadenas para textos cortos/medios y `15`–`20` cadenas para libros extensos. |
| **5. Estimaciones Matemáticas** | • Reducir los lotes fijos de `25` cadenas (`9.628` lotes) a **lotes adaptativos por presupuesto de tokens (`~58` cadenas de media = `~4.112` lotes)** y agruparlos en **Supertandas (`Workpacks`) de `12` lotes** reduce las interacciones humanas necesarias de `9.628` a **`~343` turnos supervisados** (`-96,4 %`). |
| **6. Aspectos Pendientes de Verificar Empíricamente** | • Las **cuotas exactas de suscripción** de Antigravity (mensajes o tokens permitidos por ventana de horas antes de un enfriamiento) para `Gemini 4 Argon`, `Claude Sonnet 5.5` y `Claude Opus 5.5`.<br>• El **tiempo real de procesamiento en segundos por lote** de cada uno de los tres modelos dentro de Antigravity.<br>• La tasa exacta de acierto en primera pasada (*First-Pass Hard Gate Pass Rate*) de cada modelo sobre las `300` cadenas reales. |

---

## 3. ARQUITECTURA RECOMENDADA Y DECISIONES PROPUESTAS

### 3.1. Método de Integración Más Viable y Por Qué (Objetivo B)
Se recomienda la **Arquitectura Híbrida «Transactional JSONL Maildrop + Local CLI Gatekeeper»** (detallada en [ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/NolvusAIT_Design/Fase10_Antigravity_Architecture/ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md)):
1. **`Nolvus AI Translator` prepara lotes de prueba** en `workspace_data/antigravity_exchange/01_inbox_pending/`, con un manifiesto de contexto JSON y un archivo JSONL de entrada con IDs/hash y texto protegido.
2. **El agente de Antigravity procesa los archivos** usando el modelo seleccionado y escribe un archivo JSONL de salida completo en `03_outbox_ready/`. No se presupone escritura incremental ni `Append=True`.
3. **El usuario o proceso local controlado ejecuta el Gatekeeper** por separado. Durante smoke test y benchmark, el Gatekeeper valida e ingresa exclusivamente en una DB aislada. El agente no ejecuta la ingesta ni recibe permisos de escritura a SQLite, glosario, TM o rutas originales.
4. **Validación y recuperación por registro:** el Gatekeeper comprueba esquema, conjunto exacto de IDs, hashes, placeholders, etiquetas, claves y reglas ES-419. Solo los registros completos y válidos pueden rescatarse; líneas defectuosas/truncadas se ponen en cuarentena y sus IDs se solicitan de nuevo.
5. **Cierre del lote:** el lote de prueba se marca completo únicamente cuando todos los IDs esperados tienen una respuesta válida. La persistencia parcial en una DB productiva no forma parte de esta etapa.


#### ¿Por qué este método es superior a las demás alternativas?
- **Recuperación por registros completos:** JSONL permite parsear líneas individualmente y recuperar solo las que estén completas y superen el esquema. No garantiza recuperar una escritura interrumpida ni la última línea truncada; estos casos se ponen en cuarentena y se reintentan.
- **Separación de permisos para SQLite:** El agente no recibe escritura a la base de datos. El Gatekeeper local escribe solo en la DB de prueba durante esta validación; cualquier ingesta productiva requerirá autorización y un procedimiento posterior.
- **Permisos a verificar:** El workspace puede permitir las operaciones de archivo, pero las reglas concretas dependen de la versión de Antigravity y del sistema operativo. Verificar los permisos Windows de lectura/escritura por carpeta; no depender de permisos automáticos.

---

### 3.2.Decisiones Arquitectónicas Clave (Objetivos C, D y E)
1. **Conservar el `100 %` de la Infraestructura API Existente (Objetivo C):**
   - No se elimina ni desactiva permanentemente nada de [translation/interfaces.py](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/translation/interfaces.py) ni de [translation/future_stubs.py](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/translation/future_stubs.py).
   - Tanto el adaptador `AntigravityFileExchangeProvider` como los adaptadores `CloudApiTranslationProvider` comparten el mismo contrato `BatchRequestEnvelope` / `BatchResponseEnvelope` ([DUAL_PROVIDER_ARCHITECTURE.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/NolvusAIT_Design/Fase10_Antigravity_Architecture/DUAL_PROVIDER_ARCHITECTURE.md)).
2. **Deduplicación Pre-Lote por Unidad de Trabajo Sensible al Contexto (`TranslationWorkUnit`):**
   - Antes de crear los lotes, las `328.485` ocurrencias se agrupan por `(source_hash, context_signature)`, evitando enviar duplicados al agente, pero separando estrictamente los textos cortos polisémicos (`"Fire"`, `"Light"`, `"Charge"`) cuando su `record_type` o dominio difiere.
3. **Orden de Traducción «Anclas Primero por Mod» (`Anchor-First Scheduling`):**
   - Dentro de cada mod, se procesan primero los nombres de lugares (`CELL`/`LCTN`), facciones (`FACT`) y objetos (`WEAP`/`ARMO`/`ALCH`), fijando su traducción en la Memoria de Traducción y Glosario antes de procesar los diálogos (`INFO:NAM1`), misiones (`QUST`) y libros (`BOOK:DESC`) de ese mismo mod.
4. **Enrutamiento Estratificado de Modelos dentro de Antigravity (`Tiered Routing`):**
   - Utilizar el modelo más veloz y de mayor cuota (a confirmar entre `Gemini 4 Argon` y `Claude Sonnet 5.5` en el benchmark) para el `~86 %` del volumen estructurado/estándar, y reservar `Claude Opus 5.5` (o el modelo con mayor puntuación literaria del benchmark) para el `~14 %` de alta complejidad literaria y narrativa.

---

## 4. ALTERNATIVAS DESCARTADAS Y MOTIVOS TÉCNICOS

| Alternativa Evaluada | Motivo Técnico de Descarte |
|---|---|
| **1. Automatización de clics o control de interfaz gráfica (PyAutoGUI / SendKeys)** | Extremadamente frágil ante pérdida de foco o cambios visuales, bloquea el ordenador del usuario, carece de validación transaccional y arriesga pérdida de datos. |
| **2. Intercambio en formato CSV** | Propenso a corrupción estructural al exportar libros (`BOOK:DESC`) con múltiples saltos de línea `\r\n`, tabuladores `\t`, comillas anidadas y etiquetas `<font face='...'>`. |
| **3. Permitir que el agente modifique directamente `nolvus_translator.db` con sentencias SQL libres** | Viola el principio de separación entre la Capa de Ejecución de Modelos y el Núcleo: un error del agente podría alterar registros originales o insertar traducciones sin haber pasado por `VariableProtector` ni `TranslationQualityValidator`. |
| **4. Lotes fijos pequeños de 25 cadenas enviadas de una en una por el usuario (`9.628` mensajes)** | Inviable por fatiga humana (`9.628` interacciones manuales). Se sustituye por Lotes Adaptativos (`15`–`80` cadenas) agrupados en Supertandas (`Workpacks`) de `10`–`15` lotes autovalidados. |
| **5. Lotes masivos de `500+` cadenas en un solo archivo de salida** | Alto riesgo de truncamiento por límite de tokens de salida del agente y pérdida de precisión en la mitad del lote (*lost-in-the-middle*). |

---

## 5. INVENTARIO DE COMPONENTES EXISTENTES QUE SE REUTILIZARÁN VS COMPONENTES A DESARROLLAR

### 5.1. Componentes Existentes en `src/nolvus_translator/` que se Reutilizan al `100 %` (Ya Implementados)
1. **Base de datos SQLite 3.50 WAL + FTS5:** [`DatabaseManager`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/core/database.py#L238-L367)
2. **Escáner VFS de MO2 y extractores binarios (`ESP/ESM/ESL`, `*STRINGS`, `BSA`, `PEX`, `MCM`, `INI`):** [`Mo2VfsScanner`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/mo2/vfs.py), [`EspPluginParser`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/formats/esp_parser.py), [`BsaArchiveReader`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/formats/bsa_archive.py), [`extract_pex_strings`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/formats/pex_parser.py), [`mcm_parser.py`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/formats/mcm_parser.py)
3. **Clasificador de cadenas:** [`StringClassifier`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/classification/classifier.py)
4. **Glosario Maestro `ES-419` y detector de términos:** [`GlossaryManager`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/lore/glossary.py#L20-L304)
5. **Motor de Contexto Narrativo:** [`ContextEngine` y `NarrativeContextPacket`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/lore/context_engine.py#L15-L95)
6. **Memoria de Traducción con firma contextual y FTS5/RapidFuzz:** [`TranslationMemoryEngine`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/memory/tm_engine.py#L46-L289)
7. **Protector determinista de variables y etiquetas (`⟦VAR_i⟧`):** [`VariableProtector`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/quality/protector.py#L60-L141)
8. **Validador determinista de calidad y registro `ES-419`:** [`TranslationQualityValidator`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/quality/validator.py#L21-L187)
9. **Empaquetador seguro para Mod Organizer 2 (`DSD JSON`, `*STRINGS`, `MCM UTF-16LE`, `PEX AST`) con Rollback:** [`Mo2ModPackager`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/export/mo2_packager.py#L64-L371) y [`build_dsd_json_by_plugin`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/export/dsd_exporter.py#L26-L77)
10. **Interfaz abstracta de proveedor y catálogo de stubs API:** [`TranslationProvider`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/translation/interfaces.py#L34-L64) y [`get_registered_providers`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/translation/future_stubs.py#L43-L53)

### 5.2. Componentes Nuevos o Ampliaciones que Requerirán Desarrollo (Cuando el Usuario lo Autorice)
1. **Extractor del Corpus de Benchmark de `300` Cadenas (`benchmark_builder.py`):** Script determinista para extraer las `300` cadenas estratificadas en las `11` categorías definidas en [BENCHMARK_PLAN_ANTIGRAVITY.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/NolvusAIT_Design/Fase10_Antigravity_Architecture/BENCHMARK_PLAN_ANTIGRAVITY.md).
2. **Adaptador `AntigravityFileExchangeProvider` (`translation/antigravity_provider.py`):** Implementación del exportador de sobres `batch_NNNN_meta.json` + `batch_NNNN_input.jsonl` hacia `01_inbox_pending/` y colector de `batch_NNNN_output.jsonl` desde `03_outbox_ready/`.
3. **Gatekeeper CLI Local de Validación e Ingesta (`translation/batch_gatekeeper.py`):** Comando invocable en el sandbox para validar línea a línea cada archivo `.jsonl` de salida, guardar en SQLite las líneas aprobadas, propagar a duplicados del mismo contexto y generar micro-lotes de reintento (`05_quarantine_retry/`) solo para las líneas con fallo técnico.
4. **Tabla de Historial Detallado (`string_translation_history` en `core/database.py`) y Planificador de Lotes Adaptativos por `WorkUnit`:** Migración no destructiva (`v1.2.0`) para registrar cada intento por lote/modelo y agrupar previamente por `(source_hash, context_signature)`.
5. **Incorporación de Hallazgos de Fase 9 en el Escáner de Producción (cuando se autorice):** Activar la extracción de los `3.090` `.pex` con texto dentro de `.bsa`, la lista blanca del Subgrupo 4A de `JSON_OTHER` (*Bestiary*, *Alternate Perspective*, *Custom Skill Menu*) y el normalizador de los `38` apóstrofes `\ufffd'` de `3DNPC.esp`.

---

## 6. ÍNDICE DE ENTREGABLES GENERADOS EN LA FASE 10

Todos los documentos solicitados en la **Sección 10** se han creado en la carpeta independiente `c:\Skyrim proyecto traducción\NolvusAIT_Design\Fase10_Antigravity_Architecture\`, sin sobrescribir ningún informe de las Fases 0 a 9:

1. [BENCHMARK_PLAN_ANTIGRAVITY.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/NolvusAIT_Design/Fase10_Antigravity_Architecture/BENCHMARK_PLAN_ANTIGRAVITY.md) — Rediseño completo del benchmark de `300` cadenas en `11` categorías para **`Gemini 4 Argon`**, **`Claude Sonnet 5.5`** y **`Claude Opus 5.5`**, con las 12 dimensiones de calidad y las 14 métricas operativas de Antigravity.
2. [ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/NolvusAIT_Design/Fase10_Antigravity_Architecture/ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md) — Evaluación de alternativas de integración sin API, comparativa `JSON` vs `JSONL` vs `CSV`, y especificación de la arquitectura híbrida recomendada (*Transactional JSONL Maildrop + Local CLI Gatekeeper*).
3. [DUAL_PROVIDER_ARCHITECTURE.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/NolvusAIT_Design/Fase10_Antigravity_Architecture/DUAL_PROVIDER_ARCHITECTURE.md) — Separación hexagonal entre el Núcleo común y la Capa de Ejecución de Modelos, contratos canónicos de entrada/salida y estrategia de conmutación sin pérdida entre Antigravity y API.
4. [TRANSLATION_EXECUTION_ENGINE.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/NolvusAIT_Design/Fase10_Antigravity_Architecture/TRANSLATION_EXECUTION_ENGINE.md) — Diseño de trazabilidad por cadena, máquina de estados de lotes, persistencia parcial, micro-reintentos, protección de contenido (Fase 9), deduplicación sensible al contexto y las 5 optimizaciones del Objetivo E.
5. [SCALABILITY_AND_RISK_ASSESSMENT.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/NolvusAIT_Design/Fase10_Antigravity_Architecture/SCALABILITY_AND_RISK_ASSESSMENT.md) — Análisis de escalabilidad sobre las `~240.684` cadenas únicas verificadas en la Fase 9, escalera de 6 pruebas progresivas, matriz de riesgos y condiciones objetivas de transición a API o modo híbrido.
6. [PHASE10_FINAL_REPORT.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/NolvusAIT_Design/Fase10_Antigravity_Architecture/PHASE10_FINAL_REPORT.md) — Presente informe ejecutivo y técnico consolidado.

---

## 7. PRÓXIMO PASO LÓGICO RECOMENDADO (DETENCIÓN OBLIGATORIA)

El siguiente paso lógico, una vez que revises y apruebes este diseño, es **autorizar únicamente la preparación y ejecución de la Etapa 1 (Prueba Pequeña de Funcionamiento de `25` cadenas) y la Etapa 2 (Benchmark Controlado de `300` cadenas)** con los tres modelos de tu selector (**`Gemini 4 Argon`**, **`Claude Sonnet 5.5`** y **`Claude Opus 5.5`**), utilizando una base de datos aislada de benchmark sin tocar la base de datos principal de producción.

**El sistema se detiene en este punto exacto en cumplimiento de la Sección 12.** No se ha ejecutado el benchmark, no se ha modificado el código fuente y no se ha iniciado ninguna traducción.


---

## ADENDA DE REVISIÓN — 2026-11-03

**Estado actualizado:** diseño documental completo; validación operativa pendiente. No se ha ejecutado el smoke test, no se ha realizado el benchmark y no se ha implementado el adaptador ni el Gatekeeper.

### Conclusión técnica consolidada

La opción preferida para la primera validación es **intercambio supervisado de archivos JSON/JSONL + Gatekeeper local**, sin automatizar clics de interfaz, sin hot-folder autónomo y sin dar al agente acceso a SQLite. El agente lee los archivos de entrada y escribe una salida completa en staging; un proceso local independiente valida y, solo en pruebas, ingresa resultados a una base aislada.

### Correcciones obligatorias al diseño original

- No asumir que `write_file(..., Append=True)` esté soportado. No se encontró ese contrato en la documentación oficial consultada.
- No calificar JSONL como recuperación al 100 %. Solo las líneas completas y válidas son recuperables; una línea parcial debe ir a cuarentena y reintentarse.
- No permitir que el agente ejecute la ingesta en DB ni otorgarle permisos de escritura a DB, glosario o memoria de producción. El Gatekeeper se ejecuta por separado con acceso mínimo.
- Para Windows, verificar los permisos reales de la versión instalada; la documentación del sistema de permisos actualizado advierte que su disponibilidad puede variar por sistema operativo.
- Antes de cada prueba, confirmar `AI Credit Overages = Never`. Las cuotas de Antigravity son variables y sujetas a cambios; el valor de cuenta debe confirmarse en Settings y registrarse en la bitácora.

### Secuencia autorizada para la siguiente evaluación

1. Preflight de permisos, carpeta de staging, modelo seleccionable y ajuste de excedentes.
2. Smoke test de 25 cadenas reales en una base de datos aislada.
3. Solo tras revisar y superar el resultado del smoke test, benchmark de 300 cadenas con cada modelo disponible.
4. Piloto de un mod y pruebas de continuidad/recuperación, cada una con revisión independiente.

La infraestructura de API permanece conservada como alternativa. Esta adenda no habilita facturación, llamadas API, descargas, modificaciones a Nolvus/MO2 ni traducción global.

### Fuentes oficiales de Antigravity consultadas

- [Permisos y herramientas de workspace](https://antigravity.google/docs/permissions)
- [Configuración del agente y diferencias por sistema operativo](https://antigravity.google/docs/agent-settings)
- [Cuotas base y AI Credit Overages](https://antigravity.google/docs/plans)
