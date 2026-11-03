> **ESTADO: PLAN NO EJECUTADO.** El corpus de 300 cadenas todavía debe extraerse y verificarse desde la DB y el snapshot local. Este repositorio publica el plan, no el corpus, ni la DB ni los scripts de construcción. Las afirmaciones de que ciertos validadores ya están implementados proceden de los informes previos y deben revalidarse en el código actual. Reconfirmar los modelos del selector antes de cada corrida.

---

# PLAN DE BENCHMARK REDISEÑADO PARA GOOGLE ANTIGRAVITY SIN API (`ES-419`)
**Documento:** `BENCHMARK_PLAN_ANTIGRAVITY.md`  
**Fase:** Fase 10 — Análisis, Diseño y Arquitectura sin API  
**Estado:** Diseño Técnico Completo (0 traducciones ejecutadas; 0 llamadas a APIs; 0 modificaciones a la base de datos de producción)  
**Antecedente Histórico Conservado:** `c:/Skyrim proyecto traducción/MODEL_BENCHMARK_PLAN.md` (referencia local no incluida en este repositorio) *(Versión Fase 8 conservada intacta)*

---

## 1. REVISIÓN PREVIA DEL BENCHMARK ANTERIOR (`MODEL_BENCHMARK_PLAN.md`)

De acuerdo con la **Sección 2 (Revisión Previa Obligatoria)**, se inspeccionó el documento anterior `c:/Skyrim proyecto traducción/MODEL_BENCHMARK_PLAN.md` (referencia local no incluida en este repositorio) y su informe económico asociado `c:/Skyrim proyecto traducción/API_COST_ESTIMATE_V2.md` (referencia local no incluida en este repositorio). A continuación se documenta qué elementos dependían del enfoque de API/inferencia local, cuáles siguen siendo útiles y cuáles se rediseñan en esta nueva versión:

### 1.1. Elementos del plan anterior que dependían de APIs o hardware local (Desplazados en esta fase)
1. **Catálogo de modelos orientado a API y GPU local:** El plan anterior evaluaba `Gemini 3.5 Flash-Lite`, `Gemini 3.8 Flash`, `GPT-6 Luna`, `GPT-6.1 Terra`, `GPT-6.1 Sol` y modelos locales GGUF (`Gemma 3 12B IT`, `Qwen 2.5 14B IT` sobre la `RX 9070 XT`). En esta Fase 10, la evaluación se centra exclusivamente en los **tres modelos confirmados por el usuario en su selector actual de Google Antigravity**: **`Gemini 4 Argon`**, **`Claude Sonnet 5.5`** y **`Claude Opus 5.5`** (excluyendo expresamente `GPT-6 Luna` y `GPT-6 Sol`).
2. **Métricas de facturación por token y telemetría de VRAM:** El plan anterior incluía presupuestos en dólares (`< \$0,03 USD` por corrida de prueba) y mediciones de VRAM/velocidad de *prefill* local ($T_{\text{load}}, R_{\text{prefill}}, R_{\text{gen}}$). En Antigravity mediante suscripción, esas métricas se sustituyen por las **14 Métricas Operativas de Antigravity** (tiempos de preparación y respuesta del agente, estabilidad de lotes consecutivos, intervenciones humanas, errores de herramientas y límites/cuotas reales de la suscripción).
3. **Tamaño del corpus en el documento previo (`90` vs `300` cadenas):** Aunque en la Fase 8 se proyectó una prueba de `300` cadenas, el archivo `c:/Skyrim proyecto traducción/MODEL_BENCHMARK_PLAN.md` (referencia local no incluida en este repositorio) había definido únicamente `90` cadenas (`10` por cada una de `9` categorías) y era previo a los descubrimientos de la Fase 9 (`.pex` dentro de `.bsa`, `JSON_OTHER` del Bestiario/Alternate Perspective y apóstrofes corruptos en `3DNPC.esp`). En este nuevo plan se diseña formalmente el corpus completo de **`300` cadenas reales estratificadas en `11` categorías**.

### 1.2. Elementos del plan anterior que siguen siendo útiles y se reutilizan íntegramente
1. **Política lingüística `ES-419` (`c:/Skyrim proyecto traducción/TRANSLATION_POLICY_ES419.md` (referencia local no incluida en este repositorio)) y Glosario Maestro (`c:/Skyrim proyecto traducción/GLOSSARY_POLICY.md` (referencia local no incluida en este repositorio)):** Se mantienen sin cambios las reglas de neutralidad latinoamericana, prohibición de voseo/españolismos peninsulares (`vosotros`, `os`, `habéis`, `coged`, `vale`, `tío`) y respeto obligatorio al Glosario Maestro.
2. **Puertas Duras deterministas (*Hard Gates*) implementadas en código:** Se reutilizan al `100 %` los módulos ya construidos y verificados en `src/nolvus_translator/quality/protector.py` (ruta de código local; el código fuente no está publicado aquí) (`VariableProtector.protect()` y `restore_and_verify()`) y `src/nolvus_translator/quality/validator.py` (ruta de código local; el código fuente no está publicado aquí) (`TranslationQualityValidator.validate()`).
3. **Evaluación ciega (*Blind Review*) para calidad narrativa:** Se conserva el principio de anonimizar las salidas de los tres modelos antes de calificar la naturalidad y el estilo literario.

---

## 2. MODELOS CANDIDATOS A EVALUAR EN ANTIGRAVITY (SECCIÓN 3.1)

El benchmark evaluará exclusivamente los tres modelos confirmados en el selector de Google Antigravity del usuario:

| ID Interno en Benchmark | Nombre en Selector de Antigravity | Familia / Proveedor Subyacente | Hipótesis Inicial a Verificar en el Benchmark (No Presupuesta) |
|---|---|---|---|
| `ag_gemini_4_argon` | **Gemini 4 Argon** | Google DeepMind (Gemini 4) | Posible ventaja en velocidad de procesamiento de lotes grandes y seguimiento nativo de herramientas de archivo en Antigravity; debe verificarse su naturalidad literaria en `ES-419` frente a la familia Claude. |
| `ag_claude_sonnet_5_5` | **Claude Sonnet 5.5** | Anthropic (Claude 5.5) | Posible equilibrio óptimo entre velocidad operativa, precisión en formato estructurado `JSONL` y naturalidad narrativa en español neutro. |
| `ag_claude_opus_5_5` | **Claude Opus 5.5** | Anthropic (Claude 5.5) | Posible máxima calidad literaria en libros de lore (`BOOK:DESC`), poesía, diálogos complejos y desambiguación contextual, pero con mayor latencia por lote y posible consumo más rápido de cuota de suscripción. |

> [!IMPORTANT]
> **Reglas estrictas sobre los modelos (Sección 3.1):**
> 1. **Exclusión expresa:** No se incluyen `GPT-6 Luna` ni `GPT-6 Sol` por confirmación directa del usuario de que no figuran en su selector de Antigravity.
> 2. **Verificación de disponibilidad en tiempo real:** Antes de iniciar cada corrida del benchmark, se comprobará que los tres modelos estén seleccionables en la interfaz de Antigravity. Si alguno cambiara de nombre o disponibilidad, se documentará antes de ejecutar.
> 3. **Sin suposiciones de equivalencia:** No se asume que los tres modelos tengan la misma ventana de contexto efectiva, la misma velocidad de generación, ni los mismos límites de uso por hora/día dentro de la suscripción de Antigravity.

---

## 3. CORPUS DE PRUEBA REDISEÑADO: `300` CADENAS REALES EN `11` CATEGORÍAS (SECCIÓN 3.2)

El corpus de prueba (`benchmark_corpus_300.jsonl`) se extraerá de forma determinista y en modo de solo lectura a partir de las cadenas reales de **Nolvus Awakening 6.0.21** auditadas en el `Snapshot #1` (`c:/Skyrim proyecto traducción/workspace_data/nolvus_translator.db` (referencia local no incluida en este repositorio)) y los hallazgos de la Fase 9 (`c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/phase9_audit.db` (referencia local no incluida en este repositorio)). **No se inventará ninguna cadena artificial.**

### 3.1. Distribución Estratificada de las `300` Cadenas por Categoría y Origen Real

| # | Categoría Obligatoria (Sección 3.2) | Cadenas | Registros / Formatos Reales de Origen | Qué Evalúa Específicamente en el Modelo |
|---:|---|---:|---|---|
| **01** | **Objetos, armas, armaduras e inventarios** | `30` | `WEAP:FULL`, `ARMO:FULL`, `ALCH:FULL`, `INGR:FULL`, `MISC:FULL`, `AMMO:FULL` (*LotD*, *WACCF*, *Thaumaturgy*, *Heavy Armory*) | Sintaxis natural de modificadores nominales (`Mandoble de hierro de abrasión`), conservación de espacios iniciales de ordenación (`" blackened steel ingot"`) y terminología oficial (`Stamina` → `Aguante`, `Magicka` → `Magia`). |
| **02** | **Descripciones y mensajes de interfaz** | `30` | `MESG:DESC`, `MESG:ITXT`, `PERK:DESC`, `MGEF:DNAM`, `MCM_TXT`, `MCM_JSON`, `INI_LOC`, `JSON_OTHER` (*Custom Skill Menu*, *Alternate Perspective*) | Concisión para cajas de UI/HUD, límite de expansión visual ($\le 1,35\times$ en botones `MESG:ITXT` y opciones MCM), y tono impersonal de interfaz. |
| **03** | **Diálogos (Coloquiales, Solemnes y Compañeros)** | `45` | `INFO:NAM1`, `DIAL:FULL`, `INFO:RNAM` (*3DNPC*, *Inigo*, *Remiel*, *Lucien*, *Bruma*, *RDO*, *Citizens of Tamriel*) | `25` cadenas con segunda persona plural (*you guys / all of you*) e imperativos para auditar `ustedes` vs `vosotros`; `20` cadenas con contraste de jerarquía (`tú` entre compañeros/tabernas vs `usted` ante Jarls/Príncipes Daedra) y voz propia de personaje (*Inigo* hablando en tercera persona). |
| **04** | **Misiones (Títulos, Objetivos y Diario)** | `30` | `QUST:FULL`, `QUST:NNAM`, `QUST:CNAM` (*Wyrmstooth*, *Vigilant*, *LotD*, *Falskaar*, *Midwood Isle*) | Uso de primera persona neutra en entradas de diario `QUST:CNAM` (**sin asumir género masculino ni femenino para el jugador**: *«Me encontré con...»* en vez de *«Estaba cansado/a»*), y formato verbal homogéneo en objetivos `QUST:NNAM`. |
| **05** | **Libros, códices y textos extensos** | `25` | `BOOK:FULL`, `BOOK:DESC` (`1.500` a `8.000` chars) y entradas largas de *The Dragonborn's Bestiary* (`JSON_OTHER` Fase 9) | Calidad literaria sostenida en párrafos largos, adaptación de poemas/canciones de bardo, preservación intacta de `<font face='...'>`, `<p align='...'>`, `<img>`, `[pagebreak]` y ausencia de truncamiento en textos extensos. |
| **06** | **Lore y referencias culturales de Skyrim** | `25` | `BOOK:DESC`, `INFO:NAM1`, `MGEF:DNAM`, `QUST:CNAM` (*Wintersun*, *Unslaad*, *Wheels of Lull*, *Bruma*) | Precisión en conceptos teológicos, históricos y metafísicos de Tamriel (*Aetherius*, *Oblivion*, *Kalpa*, *Dragon Break*, *Thu'um*, *Aedra/Daedra*, *Chim*, *Hist*, *Sovngarde*). |
| **07** | **Textos con ambigüedades y polisemia** | `25` | Pares reales con el mismo texto corto en distintos registros (`SPEL:FULL` vs `INFO:NAM1` vs `ARMO:FULL` vs `MCM_TXT`) | Desambiguación obligatoria guiada por `record_type`, `subrecord_type` y `content_domain`: `"Fire"` (hechizo Fuego vs orden ¡Disparen!), `"Light"` (Luz vs Ligera), `"Charge"` (Carga mágica vs ¡Carguen!), `"Ward"` (Custodia mágica vs Pabellón), `"Bound"` (Vinculado vs Límite), `"Scroll"` (Pergamino vs Desplazar). |
| **08** | **Nombres propios y títulos compuestos** | `25` | `NPC_:FULL`, `NPC_:SHRT`, `CELL:FULL`, `LCTN:FULL`, `FACT:FULL` | Respeto simultáneo de nombres propios inmutables (`keep_original=True`: *Paarthurnax*, *Inigo*, *Remiel*, *Arkngthamz*) y traducción parcial en compuestos con rango/epíteto (*"Balgruuf the Greater"* → *"Balgruuf el Grande"*, *"Whiterun Guard"* → *"Guardia de Carrera Blanca"*). |
| **09** | **Variables, placeholders y etiquetas** | `25` | `PERK:DESC`, `MGEF:DNAM`, `MCM_TXT`, `.pex` sueltos y `.pex` dentro de `.bsa` (hallazgo Fase 9) | Conservación exacta 1:1 de tokens enmascarados `⟦VAR_0⟧`..`⟦VAR_n⟧` correspondientes a `<Alias=...>`, `<Global=...>`, `<mag>%`, `<dur>`, `{0}`, `%.1f%%`, `\n`, y llamadas `Debug.Notification` de `.pex` en `.bsa`. |
| **10** | **Cadenas repetidas y términos del glosario (Inter-Lote)** | `20` | Cadenas con alta densidad de términos del Glosario Maestro y duplicados intencionales repartidos entre los lotes `1`, `3` y `6` | Medición determinista de **consistencia entre lotes distintos** (comprobar si el modelo traduce exactamente igual un término del glosario y una frase recurrente en el Lote 1 y en el Lote 6) y manejo de duplicados con distinto contexto. |
| **11** | **Textos técnicos híbridos y casos borde (Fase 9)** | `20` | `MCM_TXT` con claves `\$`, `INI_LOC` con referencias `FormID\|Plugin.esp`, y muestras de `3DNPC.esp` con apóstrofes `\ufffd'` | Verificar que el modelo jamás altere claves técnicas ni referencias hexadecimales, que traduzca únicamente el segmento de valor humano y que maneje correctamente entradas pre-normalizadas de `3DNPC.esp`. |
| — | **TOTAL DEL CORPUS DE BENCHMARK** | **`300`** | **100 % cadenas reales de Nolvus Awakening 6.0.21** | — |

---

## 4. CONDICIONES ESTRICTAS DE COMPARACIÓN (SECCIÓN 3.3)

Para garantizar una comparación científica e imparcial entre **Gemini 4 Argon**, **Claude Sonnet 5.5** y **Claude Opus 5.5**:

1. **Paridad absoluta de entrada (`Input Parity`):**
   - Los tres modelos recibirán **exactamente los mismos archivos de entrada**, con:
     - El mismo texto original enmascarado (`masked_source_text`).
     - El mismo paquete de contexto narrativo y técnico (`NarrativeContextPacket`: `mod_name`, `plugin_name`, `content_domain`, `record_type`, `subrecord_type`, `quest_id`, `topic_id`, `speaker_hint`).
     - El mismo subconjunto de términos aplicables del Glosario Maestro (`applicable_glossary`).
     - Las mismas instrucciones de traducción `ES-419` y las mismas reglas de protección de `⟦VAR_i⟧`.
     - El mismo contrato de formato de salida (`JSONL` con campos `string_uid`, `translated_masked_text`).
2. **Estructura de Lotes para Medir Efecto de Tamaño (`Batch Size Sensitivity`):**
   - Las `300` cadenas se organizarán en **3 configuraciones de tamaño de lote** dentro de la misma corrida de `300` cadenas:
     - **Lotes 1 y 2 (`2 × 25 = 50` cadenas):** Tamaño estándar actual (`25` cadenas/lote).
     - **Lotes 3, 4 y 5 (`3 × 50 = 150` cadenas):** Tamaño medio optimizado (`50` cadenas/lote).
     - **Lote 6 (`1 × 100 = 100` cadenas):** Prueba de estrés de tamaño de lote (`100` cadenas/lote) para observar si algún modelo sufre degradación, omisión de registros intermedios o truncamiento al procesar 100 cadenas en un solo archivo.
3. **Aislamiento de Sesiones y Carpetas:**
   - Cada modelo se evaluará en una **conversación nueva e independiente** de Antigravity.
   - Los archivos de entrada y salida de cada modelo se alojarán en carpetas separadas (`benchmark_runs/gemini_4_argon/`, `benchmark_runs/claude_sonnet_5_5/`, `benchmark_runs/claude_opus_5_5/`) para impedir que un modelo lea las respuestas generadas por otro modelo.
4. **Documentación de diferencias no igualables:**
   - Cualquier parámetro interno propio de la plataforma Antigravity que no sea configurable por el usuario (por ejemplo, temperatura interna por defecto del selector o presupuesto de razonamiento interno de cada proveedor) quedará documentado explícitamente como variable inherente al modelo dentro de Antigravity.

---

## 5. MÉTRICAS DE CALIDAD LINGÜÍSTICA Y TÉCNICA (SECCIÓN 3.4)

La calidad **no** se reducirá a una única puntuación opaca. Cada modelo será evaluado mediante una **arquitectura de 3 capas de evaluación** que separa tajantemente los **Errores Críticos (Bloqueantes)** de los **Errores Semánticos/Narrativos** y los **Errores Estilísticos**.

### 5.1. Clasificación de Severidad de Errores

| Nivel de Severidad | Tipo de Error | Definición y Ejemplos Concretos | Consecuencia en Producción |
|---|---|---|---|
| **S0 — Crítico Estructural (Hard Gate)** | Rotura técnica o de contrato | Pérdida, duplicación o alteración de cualquier token `⟦VAR_i⟧`; desbalanceo de etiquetas `<font>`/`[pagebreak]`; traducción de una clave `\$`; omisión de un `string_uid`; cadena vacía; error de codificación `UTF-16LE`/`UTF-8`. | **Rechazo automático inmediato (`REJECTED`)**. La cadena no puede exportarse a Skyrim bajo ninguna circunstancia. |
| **S1 — Crítico de Política o Lore (Hard Gate Lingüístico)** | Violación de regla innegociable | Uso de formas del español de España (`vosotros`, `habéis`, `coged`); traducción/hispanización de un nombre propio protegido (`keep_original=True`, ej. *Paarthurnax*, *Inigo*); violación de un término obligatorio del Glosario Maestro (*Whiterun* no traducido como *Carrera Blanca*); texto devuelto en inglés sin traducir; alucinación de contenido inexistente. | **Rechazo o marcado obligatorio (`INCORRECT` / `DUBIOUS`)**. Exige corrección antes de aprobar. |
| **S2 — Mayor Semántico / Contextual** | Error de sentido o registro | Inversión del sentido de una frase; desambiguación incorrecta en polisemia (traducir `"Fire!"` de arqueros como `"¡Fuego [elemento]!"` o `"Light"` en armadura ligera como `"Luz"`); asumir género masculino/femenino para el jugador en el diario de misiones (`QUST:CNAM`); pérdida del trato de respeto (`tú` vs `usted`) ante un Jarl o Daedra. | Requiere revisión humana o retraducción con modelo superior. |
| **S3 — Menor Estilístico / Fluidez** | Preferencia de redacción | Giro sintáctico ligeramente literal pero comprensible y gramaticalmente correcto; repetición léxica menor en un libro largo; longitud visual entre `1,35x` y `1,60x` en descripciones amplias. | Aceptable con nota de mejora estilística; no bloquea jugabilidad ni lore. |

### 5.2. Las 12 Dimensiones de Calidad Evaluadas (Automáticas + Revisión Humana Ciega)

| # | Dimensión de Calidad (Sección 3.4) | Tipo de Medición | Herramienta / Protocolo de Evaluación | Métrica Reportada |
|---:|---|---|---|---|
| **1** | **Conservación de variables y placeholders** | `100 %` Determinista | `VariableProtector.restore_and_verify()` sobre las `300` cadenas | Tasa de exactitud (`%` de cadenas con `0` errores de `⟦VAR_i⟧`) |
| **2** | **Conservación de etiquetas y estructura** | `100 %` Determinista | Verificador de árboles `<font>`, `<p>`, `<b>`, `<i>`, `[pagebreak]` y saltos `\r\n` / `\t` | Tasa de preservación estructural (`%`) |
| **3** | **Ausencia de omisiones** | `100 %` Determinista | Cruce de conjuntos `set(input_uids) == set(output_uids)` y verificación de no vacíos ni fragmentos sin traducir | Conteo de omisiones (`0` exigido) |
| **4** | **Conservación de nombres propios** | `100 %` Determinista | Cruce contra entidades `keep_original=True` del Glosario Maestro y Categoría 08 | `%` de nombres propios conservados intactos |
| **5** | **Consistencia terminológica (Glosario)** | `100 %` Determinista | `GlossaryManager.match_terms_in_text()` + verificador de `approved_translation` | `%` de cumplimiento del Glosario Maestro |
| **6** | **Consistencia entre diferentes lotes** | Determinista + Comparativa | Comparación de hash de traducción para las cadenas y términos repetidos en los Lotes 1, 3 y 6 (Categoría 10) | `%` de coincidencia inter-lote |
| **7** | **Naturalidad del español latinoamericano neutro (`ES-419`)** | Híbrida (Regex + Humana Ciega) | Detector automático `_PENINSULAR_FORMS_RE` + puntuación humana ciega (`1–5`) en fluidez natural sin modismos locales | `%` libre de españolismos + Media (`1–5`) |
| **8** | **Fidelidad al significado original** | Revisión Humana Ciega | Cotejo semántico oración por oración frente al inglés original en muestra estratificada (`120` de `300` cadenas) | `%` sin distorsión semántica + Media (`1–5`) |
| **9** | **Ausencia de traducciones inventadas (*Hallucinations*)** | Híbrida (Longitud + Humana Ciega) | Control de ratio de expansión y verificación de que no se añadan frases, notas del traductor ni explicaciones extra | Conteo de alucinaciones detectadas (`0` exigido) |
| **10** | **Fidelidad al lore de Skyrim** | Revisión Experta Lore + Glosario | Evaluación de las `25` cadenas de Categoría 06 (Lore) y `25` de Categoría 07 (Ambigüedades/Polisemia) | `%` de acierto conceptual y desambiguación |
| **11** | **Calidad narrativa y registro de personaje** | Revisión Humana Ciega | Evaluación de `tú`/`usted`, género neutro del protagonista en `QUST:CNAM` y voz de compañeros en Categorías 03 y 04 | Puntuación narrativa (`1–5`) |
| **12** | **Calidad de los textos extensos** | Revisión Humana Ciega | Evaluación literaria de los libros completos, poemas y entradas del Bestiario (Categoría 05) | Puntuación literaria en textos largos (`1–5`) |

---

## 6. MÉTRICAS OPERATIVAS DE ANTIGRAVITY (SECCIÓN 3.5)

Dado que el objetivo central es determinar si **Google Antigravity puede actuar como intermediario de traducción estable sin API**, durante la ejecución del benchmark se registrarán empíricamente (con marcas de tiempo exactas en JSON y bitácora operativa) las siguientes **14 métricas operativas** para cada uno de los tres modelos:

| # | Métrica Operativa de Antigravity | Unidad / Registro | Cómo se Medirá Empíricamente durante el Benchmark |
|---:|---|---|---|
| **1** | **Tiempo de preparación de cada lote** | Milisegundos (`ms`) / Segundos (`s`) | Cronometraje automático en Python (`time.perf_counter()`) desde la consulta a SQLite hasta la escritura del archivo `batch_NNN_input.jsonl`. |
| **2** | **Tiempo de procesamiento del agente por lote** | Segundos (`s`) y `s / cadena` | Diferencia entre el `timestamp` de envío de la instrucción al agente y el `mtime` final del archivo `batch_NNN_output.jsonl` validado en disco. |
| **3** | **Cantidad de cadenas procesadas** | `cadenas / lote` y `cadenas / minuto` | Conteo verificado de registros válidos producidos por minuto de sesión activa. |
| **4** | **Comportamiento según tamaño del lote** | Comparativa `25` vs `50` vs `100` cadenas | Tasa de éxito a la primera, latencia por cadena y presencia de errores al escalar de `25` a `50` y `100` cadenas por archivo. |
| **5** | **Cantidad de resultados completos a la primera** | Conteo y `%` (`First-Pass Complete`) | Lotes devueltos con el `100 %` de los `string_uid` presentes y `100 %` de Puertas Duras aprobadas en el primer intento. |
| **6** | **Resultados incompletos o defectuosos** | Conteo y desglose por causa | Registro de líneas truncadas, JSONL malformado, IDs omitidos o placeholders `⟦VAR_i⟧` alterados antes de corrección. |
| **7** | **Intervenciones humanas necesarias** | Conteo de acciones humanas / lote | Número exacto de mensajes, confirmaciones o correcciones manuales que el usuario tuvo que realizar por lote o por tanda. |
| **8** | **Errores del agente** | Conteo y tipo de error | Fallos al invocar herramientas (`write_file`, `edit_file`, `run_command`), confusión de rutas o comentarios añadidos fuera de formato. |
| **9** | **Interrupciones** | Conteo y causa | Cortes de generación, timeouts de herramienta, desconexiones o reinicios del servidor de Antigravity durante la corrida. |
| **10** | **Capacidad de recuperación** | Tiempo (`s`) y `%` de datos salvados | Prueba real de reanudación tras interrupción: medir qué líneas completas y válidas sobreviven a la interrupción; reintentar los IDs faltantes y poner en cuarentena cualquier línea parcial. |
| **11** | **Cuotas o límites encontrados** | Observación empírica documentada | Registro fiel de cualquier aviso de límite de mensajes, enfriamiento temporal (*cooldown*), degradación de prioridad o bloqueo de modelo en la interfaz de Antigravity. **No se presupone ninguna cifra previa.** |
| **12** | **Comportamiento ante solicitudes extensas** | Observación sobre Lote 5 (Libros) y Lote 6 (`100` cadenas) | Verificar si el agente divide la escritura en varios pasos, si trunca textos largos (`BOOK:DESC` de `8.000` chars) o si mantiene la integridad completa. |
| **13** | **Estabilidad al procesar varios lotes consecutivos** | Evolución del Lote 1 al Lote 6 | Medir si la precisión terminológica, el respeto a placeholders o la velocidad se degradan conforme avanza la conversación del agente. |
| **14** | **Consistencia entre ejecuciones** | Repetición del Lote 1 en sesión nueva | Ejecutar el Lote 1 (`25` cadenas) en dos sesiones distintas con el mismo modelo para medir la variabilidad de salida y estabilidad del flujo. |

---

## 7. PROCEDIMIENTO PASO A PASO Y CRITERIOS DE ACEPTACIÓN (SECCIÓN 3.6)

### 7.1. Procedimiento de Ejecución Controlada (Solo tras Autorización del Usuario)
1. **Fase P1 (Exportación Determinista):** Un script local de solo lectura extrae las `300` cadenas reales y genera los `6` archivos de lote (`2×25`, `3×50`, `1×100`) en subdirectorios idénticos para cada uno de los 3 modelos.
2. **Fase P2 (Corrida por Modelo en Antigravity):**
   - Seleccionar **Modelo 1 (`Gemini 4 Argon`)** en una sesión limpia → procesar los 6 lotes mediante el protocolo de intercambio de archivos → ejecutar el validador determinista local y registrar las 14 métricas operativas.
   - Seleccionar **Modelo 2 (`Claude Sonnet 5.5`)** en una sesión limpia → procesar los mismos 6 lotes → validar y registrar métricas.
   - Seleccionar **Modelo 3 (`Claude Opus 5.5`)** en una sesión limpia → procesar los mismos 6 lotes → validar y registrar métricas.
3. **Fase P3 (Auditoría Automática + Evaluación Ciega):**
   - El validador local calcula automáticamente las métricas deterministas (Dimensiones 1 a 6 y detección de españolismos).
   - Se genera una tabla comparativa anónima (`Modelo A`, `Modelo B`, `Modelo C`) para calificar las Dimensiones 7 a 12 y responder con evidencia a las **10 preguntas obligatorias de la Sección 3.6**.

### 7.2. Criterios Cuantitativos de Aceptación para Aprobar un Modelo / Flujo
- **Integridad Técnica post-validación local (Dimensiones 1, 2 y 3):** **`100,0 %`** (ninguna cadena con variables rotas ni IDs omitidos puede pasar a la base de datos).
- **Tasa de Éxito en Primera Pasada (*First-Pass Hard Gate Pass Rate*):** **$\ge 97,0\,\%$** antes de micro-reintento automático.
- **Cumplimiento de Glosario Maestro y Nombres Propios (Dimensiones 4 y 5):** **$\ge 96,0\,\%$**.
- **Ausencia de Formas Peninsulares (`vosotros/os/habéis`):** **`100,0 %` (`0` ocurrencias permitidas)**.
- **Viabilidad Operativa en Antigravity:** Procesamiento estable de los 6 lotes consecutivos sin pérdida de datos y con un máximo de `1` instrucción del usuario por tanda de lotes.
