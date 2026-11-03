> **ESTIMACIÓN DE COSTES V1 — OBSOLETA Y CONSERVADA:** Esta es la primera estimación económica de la Fase 8. Fue corregida y sustituida históricamente por [API_COST_ESTIMATE_V2.md](./API_COST_ESTIMATE_V2.md), que también debe tratarse como estimación histórica y no como tarifa actual. No usar sus cifras, catálogo de modelos ni recomendaciones para decidir gasto presente.

---

# ESTIMACIÓN ECONÓMICA Y COMPARATIVA DE PROVEEDORES IA — FASE 8 (`ES-419`)
**Proyecto**: NOLVUS AI TRANSLATOR — Nolvus Awakening 6.0.21  
**Política Aplicada**: Localización 100 % propia en español latinoamericano neutro (`es-419`) desde el inglés original, conservando separada la localización oficial de Bethesda y excluyendo el 100 % de las traducciones comunitarias.  
**Restricción de Seguridad**: Ninguna API de pago ha sido activada. Este estudio se realizó en modo de simulación y cálculo matemático local mediante [`CostEstimator`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/reports/cost_estimator.py).

---

## 1. Volumen Real Auditado Base de Cálculo (`Snapshot #1` Depurado)

Los cálculos siguientes se fundamentan en el inventario real auditado de `Snapshot #1` en [`nolvus_translator.db`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/workspace_data/nolvus_translator.db) (con comparativa adicional para el Corte C que incluye los `.pex` dentro de `.bsa` de mods):

| Métrica de Volumen | Valor Exacto (`Corte B` Base) | Valor Ampliado (`Corte C` con `.pex` en `.bsa`) |
| :--- | ---: | ---: |
| **Ocurrencias traducibles totales en mods** | `314.449` | `328.485` |
| **Ocurrencias ahorradas por deduplicación (`TM`)** | `79.123` (`25,16 %`) | `88.241` (`26,86 %`) |
| **Cadenas únicas netas a traducir (`unique_pending_net`)** | **`235.326`** | **`240.244`** |
| **Caracteres únicos en inglés (`unique_pending_chars`)** | **`20.062.309`** (`20,06 M`) | **`20.314.890`** (`20,31 M`) |
| **Palabras únicas en inglés** | `~3.525.800` | `~3.568.400` |
| **Longitud media por cadena única** | `85,25 caracteres` (`14,98 palabras`) | `84,56 caracteres` |
| **Distribución por longitud** | `53.140` cortas ($\le 20$ chars), `117.508` medias ($21\text{–}80$ chars), `60.775` diálogos ($81\text{–}250$ chars), `2.503` párrafos ($251\text{–}1.000$ chars), `1.400` libros ($>1.000$ chars) | +`4.918` cadenas cortas/medias de notificaciones de script |
| **Lotes a 25 cadenas/lote** (o 40 cadenas/lote) | **`9.414` lotes** (`5.884` lotes a 40/lote) | **`9.610` lotes** (`6.007` lotes a 40/lote) |
| **Tokens de texto fuente + estructura JSON (`Input Source`)** | **`9.515.423`** (`9,52 M tokens`) | `9.672.110` (`9,67 M tokens`) |
| **Tokens de sistema + contexto + Glosario (`Input Context`, 25/lote)** | **`3.577.320`** (`3,58 M tokens`) | `3.651.800` (`3,65 M tokens`) |
| **Total Tokens de Entrada Base (`Input Total`, 25/lote)** | **`13.092.743`** (**`13,09 M tokens`**) | **`13.323.910`** (`13,32 M tokens`) |
| **Total Tokens de Salida Base (`Output Total`, expansión `1,18x` + JSON)** | **`9.524.439`** (**`9,52 M tokens`**) | **`9.681.900`** (`9,68 M tokens`) |
| **Total con Margen de Incertidumbre/Revisión (`+15 %`)** | **`15,06 M Input` / `10,95 M Output`** | **`15,32 M Input` / `11,13 M Output`** |

---

## 2. Evaluación Detallada de las 4 Alternativas de Proveedor

### Opción 1 — Google Gemini API (`Gemini 2.5 Flash` / `Gemini 2.5 Pro`)
Tarifas oficiales verificadas (`ai.google.dev/pricing`, en USD por millón de tokens para contextos $\le 200\text{K}$):

| Modelo y Modalidad | Precio Input / 1M | Precio Output / 1M | Costo Base (`13,09M In` / `9,52M Out`) | **Costo Total con Margen +15% (`15,06M In` / `10,95M Out`)** | Costo por `1.000` cadenas únicas |
| :--- | ---: | ---: | ---: | ---: | ---: |
| **Gemini 2.5 Flash — Batch API (50 % dto.)** | \$0,15 | \$1,25 | **\$13,87 USD** | **\$15,95 USD** | \$0,0678 USD |
| **Gemini 2.5 Flash — Estándar Síncrono** | \$0,30 | \$2,50 | **\$27,74 USD** | **\$31,90 USD** | \$0,1356 USD |
| **Gemini 2.5 Flash — Estándar + Context Caching** *(cacheando `3,58M` tokens de sistema/glosario a \$0,03/1M)* | \$0,30 / \$0,03 | \$2,50 | **\$26,77 USD** | **\$30,79 USD** | \$0,1308 USD |
| **Gemini 2.5 Pro — Batch API (50 % dto.)** | \$0,625 | \$5,00 | **\$55,80 USD** | **\$64,17 USD** | \$0,2727 USD |
| **Gemini 2.5 Pro — Estándar Síncrono** | \$1,25 | \$10,00 | **\$111,61 USD** | **\$128,35 USD** | \$0,5454 USD |

- **Ventajas**: Excelente seguimiento de instrucciones complejas en JSON estructurado, ventana de contexto masiva (`1M` tokens), soporte nativo para *Batch API* (50 % de descuento real) y *Context Caching* para las reglas `ES-419` y el Glosario Maestro.
- **Precaución Técnica Importante**: En la serie Gemini 2.5, los *thinking tokens* (tokens de razonamiento interno) se facturan al precio de *Output*. Para mantener el costo exacto arriba calculado en `Gemini 2.5 Flash`, el conector de Fase 9 debe fijar `thinking_budget = 0` (o bajo presupuesto de pensamiento) en lotes de interfaz/objetos y reservar el razonamiento extendido exclusivamente para libros y diálogos ambiguos.

---

### Opción 2 — OpenAI API (`GPT-4o mini` / `GPT-4.1 mini` / `GPT-4.1`)
Tarifas oficiales verificadas (`openai.com/api/pricing`, en USD por millón de tokens):

| Modelo y Modalidad | Precio Input / 1M | Precio Output / 1M | Costo Base (`13,09M In` / `9,52M Out`) | **Costo Total con Margen +15% (`15,06M In` / `10,95M Out`)** | Costo por `1.000` cadenas únicas |
| :--- | ---: | ---: | ---: | ---: | ---: |
| **GPT-4o mini — Batch API (50 % dto.)** | \$0,075 | \$0,30 | **\$3,84 USD** | **\$4,41 USD** | \$0,0188 USD |
| **GPT-4o mini — Estándar Síncrono** | \$0,15 | \$0,60 | **\$7,68 USD** | **\$8,83 USD** | \$0,0375 USD |
| **GPT-4.1 mini — Batch API (50 % dto.)** | \$0,20 | \$0,80 | **\$10,24 USD** | **\$11,77 USD** | \$0,0500 USD |
| **GPT-4.1 mini — Estándar Síncrono** | \$0,40 | \$1,60 | **\$20,48 USD** | **\$23,55 USD** | \$0,1001 USD |
| **GPT-4.1 — Batch API (50 % dto.)** | \$1,00 | \$4,00 | **\$51,19 USD** | **\$58,87 USD** | \$0,2502 USD |
| **GPT-4.1 — Estándar Síncrono** | \$2,00 | \$8,00 | **\$102,38 USD** | **\$117,74 USD** | \$0,5003 USD |

- **Ventajas**: **GPT-4.1 mini** y **GPT-4o mini** no cobran tokens ocultos de razonamiento (*non-reasoning models*), ofrecen *Structured Outputs* estrictos (`response_format: json_schema`) que garantizan un 0 % de errores de sintaxis JSON, y cuentan con *Prompt Caching* automático (75 % de descuento en el prefijo repetido de sistema/glosario) además de *Batch API* (50 % de descuento con entrega en $<24\text{h}$).

---

### Opción 3 — DeepL API (`API Growth / Pro` por Caracteres)
Tarifas oficiales verificadas (`deepl.com/pro-api`): Plan base mensual de **\$32,50 USD/mes** (incluye el primer `1,00 M` de caracteres) + **\$27,50 USD por cada millón de caracteres adicional** (o tarifa clásica API Pro de \$5,49 base + \$25,00 / 1M chars).

| Modalidad DeepL | Caracteres Únicos (`Corte B`) | Costo Base (`20,06 M chars`) | **Costo con Margen +15 % (`23,07 M chars`)** | Costo por `1.000` cadenas únicas |
| :--- | ---: | ---: | ---: | ---: |
| **DeepL API Growth (Tarifa vigente 2026)** | 20.062.309 | **\$556,71 USD** | **\$639,47 USD** | \$2,7174 USD |
| **DeepL API Pro (Tarifa clásica \$25/1M)** | 20.062.309 | **\$507,05 USD** | **\$582,29 USD** | \$2,4744 USD |

- **Desventajas Críticas para Skyrim/Nolvus**:
  1. **Entre `20x` y `50x` más caro** que Gemini 2.5 Flash o GPT-4.1 mini (\$582–\$639 USD frente a \$11–\$24 USD).
  2. **Incapacidad de razonamiento narrativo y de lore**: DeepL es un motor NMT clásico por frase; no acepta un prompt de sistema con las reglas `ES-419`, ni puede distinguir si `"Fire"` es un hechizo o una orden de arqueros según el registro `SPEL` vs `INFO`, ni comprende el lore de Tamriel más allá de un diccionario plano `source -> target`.
  3. **Riesgo en etiquetas complejas de Skyrim**: Aunque soporta etiquetas XML básicas, las construcciones anidadas de Skyrim (`<Alias.ShortName=Child1>`, `[pagebreak]`, `%.1f%%`) requieren pre-enmascaramiento externo y suelen sufrir alteraciones de orden sintáctico en motores NMT puros.

---

### Opción 4 — Modelos Locales en el Hardware del Usuario (`RX 9070 XT 16 GB VRAM` + `Ryzen 7 5700X3D` + `32 GB RAM`)
Tu equipo cuenta con una GPU **AMD Radeon RX 9070 XT (arquitectura RDNA 4, 16 GB GDDR6, ~640 GB/s de ancho de banda)**, procesador **Ryzen 7 5700X3D** y **32 GB de RAM DDR4**, ejecutando inferencia local mediante **LM Studio / llama.cpp (backend ROCm 6.3+ / Vulkan)**.

| Modelo Local Recomendado (Cuantización) | Huella en VRAM (con contexto `8K`) | Velocidad Realista en `RX 9070 XT` | Tiempo Total Estimado (`9.414` lotes / `235.326` cadenas) | Costo de API | Costo Eléctrico Estimado (`~320W` sistema, ~\$0,15/kWh) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Qwen 2.5 14B Instruct (`Q4_K_M` / `Q5_K_M`, ~9,5–11 GB)** o **Gemma 3 12B Instruct (`Q5_K_M`, ~9,2 GB)** | `10,5 – 12,8 GB` *(cabe 100 % en los 16 GB de VRAM)* | `~55 – 70 tokens/s` de generación (`~4,5 – 5,8 s` por lote de 25 cadenas) | **~12 a 15 horas** (1 o 2 noches con pausa/reanudación por checkpoints) | **\$0,00 USD** | **~\$0,60 – \$0,80 USD** |
| **Mistral Small 24B Instruct 2501 (`IQ4_XS` / `Q4_K_M`, ~13,8–14,5 GB)** | `15,2 – 15,8 GB` *(ajuste límite en 16 GB con KV cache Q8_0)* | `~32 – 42 tokens/s` de generación (`~8 – 10 s` por lote) | **~22 a 26 horas** | **\$0,00 USD** | **~\$1,10 – \$1,30 USD** |
| **Qwen 2.5 32B / Llama 3.3 70B (Offload parcial a RAM DDR4)** | Excede 16 GB VRAM -> Offload PCIe a RAM DDR4 | `~4 – 9 tokens/s` (cuello de botella de ancho de banda DDR4) | `>120 horas` *(No recomendado)* | \$0,00 USD | ~\$6,00 USD |

- **Evaluación Técnica**:
  - **Viabilidad**: **100 % viable** para modelos de **12B a 14B parámetros en `Q5_K_M`** (Gemma 3 12B IT / Qwen 2.5 14B Instruct) que caben holgadamente dentro de los **16 GB de VRAM** de tu RX 9070 XT sin tocar la RAM del sistema.
  - **Fortalezas**: Costo de facturación **\$0,00 USD**, privacidad total, sin límites de cuota por minuto (*rate limits*), y perfectamente compatible con nuestro sistema de **Checkpoints y Pausa/Reanudación por lote** ([`TranslationPipeline`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/translation/pipeline.py)).
  - **Limitaciones frente a API Cloud**: En textos literarios largos y complejos (los `1.400` libros de lore de más de `1.000` caracteres y los diálogos con juegos de palabras o poesía de bardos), un modelo local de 12B–14B tiene menor riqueza estilística y adherencia simultánea a múltiples restricciones del glosario que un modelo frontera como `Gemini 2.5 Pro` o `GPT-4.1`.

---

## 3. Tres Escenarios Estratégicos Recomendados

### Escenario 1 — Más Económico (Híbrido Local + Cloud Mini Batch, o 100 % Cloud Mini Batch)
- **Opción 1A (Cero gasto API — 100 % Local en tu RX 9070 XT)**:
  - **Motor**: `Gemma 3 12B Instruct (Q5_K_M)` o `Qwen 2.5 14B Instruct (Q5_K_M)` vía servidor local OpenAI-compatible (LM Studio / llama.cpp).
  - **Costo Total API**: **\$0,00 USD** (~14 horas de cómputo local fraccionables por mod).
- **Opción 1B (Cloud Ultrarrápido — Batch API)**:
  - **Motor**: `GPT-4o mini (Batch API)` o `GPT-4.1 mini (Batch API)`.
  - **Costo Total Estimado (con margen +15 %)**: **\$4,41 USD** (`GPT-4o mini Batch`) a **\$11,77 USD** (`GPT-4.1 mini Batch`).

### Escenario 2 — Equilibrado Calidad / Precio (**RECOMENDADO PARA PRODUCCIÓN GLOBAL**)
Combina un modelo rápido de última generación para el **88 %** del volumen (interfaz, objetos, armas, hechizos, MCM, objetivos cortos y diálogos estándar) y un modelo frontera de máxima capacidad literaria para el **12 %** más exigente (los `1.400` libros/tratados de lore, `2.503` párrafos narrativos largos y las `718` cadenas marcadas con `pending_research`):
- **Distribución del Volumen (`235.326` cadenas únicas)**:
  - **Tramo Estándar (88 % del volumen de tokens: UI, Objetos, MCM, Diálogos cortos/medios)**: Procesado con **`Gemini 2.5 Flash (Batch API)`** o **`GPT-4.1 mini (Batch API)`**.
  - **Tramo Literario y Lore Complejo (12 % del volumen de tokens: `BOOK:DESC`, `QUST:CNAM` largos y diálogos de jefes/compañeros principales)**: Procesado con **`Gemini 2.5 Pro (Batch API)`** o **`GPT-4.1 (Batch API)`**.
- **Costo Total Estimado (incluyendo +15 % de margen de revisión)**:
  - **Ruta Híbrida Gemini (`2.5 Flash Batch` 88 % + `2.5 Pro Batch` 12 %)**: **\$21,74 USD** (o **\$39,40 USD** en modo síncrono estándar).
  - **Ruta Híbrida OpenAI (`GPT-4.1 mini Batch` 88 % + `GPT-4.1 Batch` 12 %)**: **\$17,42 USD** (o **\$34,85 USD** en modo síncrono estándar).
- **Alternativa Híbrida Local + Cloud Pro**: Procesar objetos/UI/MCM gratis en tu **RX 9070 XT** (`\$0,00`) y enviar únicamente los libros y diálogos complejos (`12 %` del volumen) a `Gemini 2.5 Pro Batch` o `GPT-4.1 Batch` por solo **\$7,06 – \$7,70 USD** en total.

### Escenario 3 — Máxima Calidad para Contenido Narrativo Complejo (100 % Modelo Frontera)
- **Motor**: 100 % **`Gemini 2.5 Pro`** o **`GPT-4.1`** sobre la totalidad de las `235.326` cadenas únicas.
- **Costo Total Estimado (incluyendo +15 % de margen de revisión)**:
  - Mediante **Batch API (50 % dto.)**: **\$58,87 USD** (`GPT-4.1 Batch`) a **\$64,17 USD** (`Gemini 2.5 Pro Batch`).
  - Mediante **API Síncrona Estándar**: **\$117,74 USD** (`GPT-4.1`) a **\$128,35 USD** (`Gemini 2.5 Pro`).

---

## 4. Resumen Comparativo General

| Proveedor / Escenario | Modalidad | Costo Base (`Snapshot #1`) | **Costo Total (+15 % Revisión)** | Costo con `.pex` en `.bsa` (`Corte C`, +15 %) | Calidad Lore / `ES-419` | Velocidad Global |
| :--- | :--- | ---: | ---: | ---: | :---: | :---: |
| **Local (`RX 9070 XT 16GB`)** | `Gemma 3 12B` / `Qwen 2.5 14B Q5_K_M` | \$0,00 USD | **\$0,00 USD** *(~\$0,70 luz)* | **\$0,00 USD** | Buena (UI/Objetos) / Media-Alta (Libros) | ~12–15 horas |
| **Escenario 1B (Más Económico Cloud)** | `GPT-4o mini` (Batch API) | \$3,84 USD | **\$4,41 USD** | **\$4,49 USD** | Buena | < 6 horas (Batch) |
| **Escenario 1C (Económico Alta Precisión)** | `GPT-4.1 mini` (Batch API) | \$10,24 USD | **\$11,77 USD** | **\$11,98 USD** | Muy Buena | < 6 horas (Batch) |
| **Escenario 1D (Gemini Económico)** | `Gemini 2.5 Flash` (Batch API) | \$13,87 USD | **\$15,95 USD** | **\$16,22 USD** | Muy Buena | < 6 horas (Batch) |
| **Escenario 2A (Equilibrado Híbrido Local + Cloud Pro)** | `RX 9070 XT` (88 %) + `Gemini 2.5 Pro Batch` (12 %) | \$6,70 USD | **\$7,70 USD** | **\$7,70 USD** | Excelente | ~10 horas |
| **Escenario 2B (Equilibrado Cloud OpenAI)** | `GPT-4.1 mini Batch` (88 %) + `GPT-4.1 Batch` (12 %) | \$15,15 USD | **\$17,42 USD** | **\$17,73 USD** | Excelente | < 6 horas (Batch) |
| **Escenario 2C (Equilibrado Cloud Gemini)** | `Gemini 2.5 Flash Batch` (88 %) + `2.5 Pro Batch` (12 %) | \$18,90 USD | **\$21,74 USD** | **\$22,10 USD** | Excelente | < 6 horas (Batch) |
| **Escenario 3 (100 % Frontera Cloud)** | `GPT-4.1 Batch` / `Gemini 2.5 Pro Batch` | \$51,19 – \$55,80 | **\$58,87 – \$64,17 USD** | **\$59,90 – \$65,25 USD** | Máxima | < 12 horas (Batch) |
| **DeepL API Growth** *(No recomendado)* | Pago por carácter (`20,06 M chars`) | \$556,71 USD | **\$639,47 USD** | **\$647,40 USD** | Baja para Lore/Tags | ~2 horas |
