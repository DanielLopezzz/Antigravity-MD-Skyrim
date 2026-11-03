# ESTIMACIÓN ECONÓMICA Y COMPARATIVA DE PROVEEDORES IA — VERSIÓN 2 CORREGIDA (`FASE 8`)
**Proyecto**: NOLVUS AI TRANSLATOR — Nolvus Awakening 6.0.21  
**Documento Previo Conservado**: [API_COST_ESTIMATE.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/API_COST_ESTIMATE.md) *(Versión 1 conservada para trazabilidad histórica)*  
**Libro de Cálculo Comparativo**: [API_COST_COMPARISON_V2.xlsx](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/API_COST_COMPARISON_V2.xlsx)  
**Plan de Benchmark Lingüístico y Técnico**: [MODEL_BENCHMARK_PLAN.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/MODEL_BENCHMARK_PLAN.md)  
**Motor de Cálculo en Código**: [`CostEstimator`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/reports/cost_estimator.py)  

---

## 0. Registro de Correcciones Documentadas respecto a la Versión 1 (`API_COST_ESTIMATE.md`)

1. **Sustitución de Modelos Antiguos o en Retirada por la Generación Vigente (Sección 1)**:
   - En la Versión 1 se habían listado como candidatos principales `Gemini 2.5 Flash`, `Gemini 2.5 Pro`, `GPT-4o mini`, `GPT-4.1 mini` y `GPT-4.1`.
   - En esta **Versión 2**, dichos modelos pasan a una sección de **Modelos Legacy / En Retirada** con sus fechas oficiales de apagado o redirección (`gemini-2.5-flash` redirige automáticamente a `gemini-3.5-flash-lite` el **1 de diciembre de 2026**; `gpt-4o-mini` y `gpt-4.1` tienen apagado definitivo anunciado para el **14 de enero de 2027**; la primera ola `gpt-5` se apaga el **11 de diciembre de 2026**).
   - Se incorporan como **candidatos principales vigentes**: **`Gemini 3.5 Flash-Lite` (`gemini-3.5-flash-lite`)**, **`Gemini 3.8 Flash` (`gemini-3.8-flash`)**, **`Gemini 4 Argon` (`gemini-4-argon`)**, **`GPT-6 Luna` (`gpt-6-luna`)**, **`GPT-6.1 Terra` (`gpt-6.1-terra`)**, **`GPT-6.1 Sol` (`gpt-6.1-sol`)** y **`GPT-6 Astra` (`gpt-6-astra`)**, distinguiendo explícitamente entre **tarifas promocionales temporales (2026)** y **tarifas permanentes (2026–2027)**.
2. **Recálculo Completo para Corte B y Corte C sin Asumir Descuentos Batch Uniformes (Sección 2)**:
   - Se presentan tablas completas con costo sin margen, costo con margen del `+15 %`, costo por cada `1.000` cadenas únicas y diferencia exacta entre **Corte B (`235.326` cadenas)** y **Corte C (`240.244` cadenas)**.
   - Se documenta que mientras Google Gemini y OpenAI aplican un **50 % de descuento en Batch API**, **DeepL aplica un 0 % de descuento por lotes** (cobra exactamente igual por carácter).
3. **Corrección Matemática Rigurosa del Tiempo de Inferencia Local (Sección 3)**:
   - En la Versión 1 existía una inconsistencia matemática: se indicaban `~9,52 M` de tokens de salida y una velocidad de `55–70 tokens/s`, pero se reportaban `12–15 horas` porque se había estimado erróneamente el tiempo por lote sobre `~300` tokens en lugar de los `1.011,7` tokens de salida reales por lote de 25 cadenas.
   - En esta Versión 2 se recalcula mediante $\text{Tokens Totales de Salida} / \text{Tokens Generados por Segundo}$, sumando además el **tiempo de procesamiento de contexto (*prefill*)**, **tiempo de carga del modelo** y **tiempo de validación QA + escritura en SQLite WAL**, tanto sin margen como con el margen del `+15 %`.
   - Para `55–70 tokens/s` teóricos, el tiempo real de generación pura es de **`37,80 a 48,10 horas` sin margen** (**`43,46 a 55,32 horas` con margen `+15 %`**), y el tiempo total de ejecución incluyendo contexto y almacenamiento es de **`40,98 a 52,60 horas` sin margen** (**`47,13 a 60,48 horas` con margen `+15 %`**). Además, se deja explícito que estas velocidades son **estimaciones teóricas** y que la velocidad real en tu **RX 9070 XT** queda pendiente de medición práctica autorizada.
4. **Eliminación de Calificaciones Subjetivas de Calidad sin Benchmark (Sección 4)**:
   - Se eliminan etiquetas subjetivas previas (*"buena"*, *"muy buena"*, *"excelente"*) y se diseña el protocolo formal de evaluación comparativa en [MODEL_BENCHMARK_PLAN.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/MODEL_BENCHMARK_PLAN.md).
5. **Revisión Técnica de *Structured Outputs* y Validación Post-Respuesta (Sección 5)**:
   - Se corrige la afirmación de que los *Structured Outputs* JSON garantizan "cero errores": una respuesta puede ser sintácticamente válida en JSON y cumplir el esquema (`json_schema`), pero fallar por truncamiento (`finish_reason = length`), rechazo de filtro de seguridad (`refusal`), omisión de IDs del lote, mutación de variables `<Alias=...>` o traducción semánticamente errónea.
6. **Precisión Regional y Comercial de DeepL para Colombia (Sección 6)**:
   - Se diferencia `API Developer`, `API Growth` (mensual y anual) y el antiguo `API Pro`, aclarando las implicaciones de facturación internacional desde **Colombia** (divisa USD/EUR, IVA digital del 19 % en servicios transfronterizos, TRM bancaria e imposibilidad de pago por transferencia local en planes mensuales).

---

## 1. Catálogo Actualizado de Modelos, Vigencia, Promociones y Fechas de Retirada

### 1.1. Candidatos Principales Vigentes (`2026–2027`)

| Proveedor | Modelo | Identificador de API | Disponibilidad y Modalidad Batch | Tipo de Tarifa y Vigencia | Input Estándar / Batch (\$/1M) | Output Estándar / Batch (\$/1M) | Input en Caché (\$/1M) |
| :--- | :--- | :--- | :--- | :--- | :---: | :---: | :---: |
| **Google Gemini** | **Gemini 3.5 Flash-Lite** | `gemini-3.5-flash-lite` | GA Activo. Soporta **Batch API (50 % dto.)** y *Context Caching*. | **Permanente** (Objetivo oficial de migración de `2.5-flash` desde `01/12/2026`). | \$0,30 / **\$0,15** | \$2,50 / **\$1,25** | \$0,03 |
| **Google Gemini** | **Gemini 3.8 Flash** *(Promo 2026)* | `gemini-3.8-flash` | GA Activo. Soporta **Batch API (50 % dto.)** y *Context Caching*. | **Promocional Temporal** (Vigente únicamente **hasta el 31/12/2026**). | \$0,75 / **\$0,375** | \$3,75 / **\$1,875** | \$0,075 |
| **Google Gemini** | **Gemini 3.8 Flash** *(Estándar 2027)* | `gemini-3.8-flash` | GA Activo. Soporta **Batch API (50 % dto.)** y *Context Caching*. | **Permanente** (Entra en vigor el **01/01/2027**). | \$1,50 / **\$0,75** | \$7,50 / **\$3,75** | \$0,15 |
| **Google Gemini** | **Gemini 4 Argon** *(Promo Nov 2026)* | `gemini-4-argon` | GA Activo (Frontera). Soporta **Batch API (50 % dto.)** y *Context Caching*. | **Promocional Temporal** (Vigente únicamente **hasta el 30/11/2026**). | \$2,00 / **\$1,00** | \$10,00 / **\$5,00** | \$0,10 |
| **Google Gemini** | **Gemini 4 Argon** *(Estándar Dic 2026)* | `gemini-4-argon` | GA Activo (Frontera). Soporta **Batch API (50 % dto.)** y *Context Caching*. | **Permanente** (Entra en vigor el **01/12/2026**). | \$4,00 / **\$2,00** | \$20,00 / **\$10,00** | \$0,20 |
| **OpenAI** | **GPT-6 Luna** | `gpt-6-luna` | GA Activo ($\le 272\text{K}$ ctx). Soporta **Batch API (50 % dto.)** y *Prompt Caching*. | **Permanente** (Sucesor oficial de `gpt-4o-mini` y `gpt-4.1-mini`). | \$0,10 / **\$0,05** | \$0,50 / **\$0,25** | \$0,01 (\$0,005 Batch) |
| **OpenAI** | **GPT-6.1 Terra** | `gpt-6.1-terra` | GA Activo ($\le 272\text{K}$ ctx). Soporta **Batch API (50 % dto.)** y *Prompt Caching*. | **Permanente** (Gama media-alta de alta precisión estructural). | \$1,00 / **\$0,50** | \$5,00 / **\$2,50** | \$0,05 (\$0,025 Batch) |
| **OpenAI** | **GPT-6.1 Sol** | `gpt-6.1-sol` | GA Activo ($\le 272\text{K}$ ctx). Soporta **Batch API (50 % dto.)** y *Prompt Caching*. | **Permanente** (Sucesor oficial de `gpt-4.1` para narrativa y razonamiento). | \$2,00 / **\$1,00** | \$10,00 / **\$5,00** | \$0,10 |
| **OpenAI** | **GPT-6 Astra** | `gpt-6-astra` | GA Activo ($\sim 1,05\text{M}$ ctx). Soporta **Batch API (50 % dto.)**. | **Permanente** (Tope de gama absoluto de OpenAI). | \$10,00 / **\$5,00** | \$50,00 / **\$25,00** | \$1,00 |
| **DeepL** | **DeepL API Growth** *(Mensual / Anual)* | `deepl-v2-translate` | Activo. **Sin descuento Batch (0 % dto.)**: cobra por carácter enviado. | **Permanente** (\$32,50/mes con `1M` chars incl., o \$312/año con `12M` chars incl.). | \$27,50 / 1M chars extra | \$0,00 *(cobra sobre entrada)* | N/A |

### 1.2. Modelos Antiguos, Deprecados o con Fecha de Retirada Anunciada *(Excluidos como Candidatos Principales)*

| Proveedor | Modelo | Identificador de API | Fecha Oficial de Retirada / Migración | Ruta de Migración Oficial |
| :--- | :--- | :--- | :--- | :--- |
| **Google Gemini** | Gemini 2.0 Flash / Flash-Lite | `gemini-2.0-flash` | **Retirados definitivamente el 1 de junio de 2026** | `gemini-3.5-flash-lite` |
| **Google Gemini** | Gemini 2.5 Flash | `gemini-2.5-flash` | **Retirado en Vertex AI el 20/10/2026**; en AI Studio **redirige automáticamente a `gemini-3.5-flash-lite` el 1 de diciembre de 2026** | `gemini-3.5-flash-lite` |
| **Google Gemini** | Gemini 2.5 Pro | `gemini-2.5-pro` | **Retirado en Vertex AI el 20/10/2026**; bloqueado para proyectos nuevos en AI Studio | `gemini-3.8-flash` / `gemini-4-argon` |
| **Google Gemini** | Gemini 3.5 Flash y 3.7 Flash | `gemini-3.5-flash` / `3.7-flash` | **Deprecados en octubre de 2026** (enrutados automáticamente a versiones superiores) | `gemini-3.8-flash` |
| **OpenAI** | GPT-3.5 Turbo / GPT-4 / o1 / 4.1-nano | `gpt-4-0613`, `o1-2024-12-17` | **Retirados definitivamente el 23 de octubre de 2026** | `gpt-6-luna` / `gpt-6.1-sol` |
| **OpenAI** | GPT-5 / GPT-5 mini / GPT-5 nano / o3 | `gpt-5`, `gpt-5-mini`, `o3` | **Apagado definitivo anunciado para el 11 de diciembre de 2026** | `gpt-6-luna` / `gpt-6.1-sol` |
| **OpenAI** | **GPT-4o, GPT-4o mini y GPT-4.1** | `gpt-4o-mini-2024-07-18`, `gpt-4.1-2025-04-14` | **Apagado definitivo anunciado para el 14 de enero de 2027** | `gpt-6-luna` / `gpt-6.1-terra` / `gpt-6.1-sol` |
| **OpenAI** | GPT-5.1 / GPT-5.3 Codex / GPT-5.4 nano | `gpt-5.1`, `gpt-5.4-nano` | **Apagado definitivo anunciado para el 1 de abril de 2027** | `gpt-6-luna` / `gpt-6.1-sol` |
| **DeepL** | DeepL API Pro *(Clásico)* | Plan Legacy (\$5,49 + \$25/1M) | **Retirado comercialmente para nuevas contrataciones** | `DeepL API Growth` |

---

## 2. Recálculo Exacto de Costos para Corte B (`235.326` cadenas) y Corte C (`240.244` cadenas)

### 2.1. Parámetros Exactos de Volumen Utilizados

| Parámetro | **Corte B** *(Sin `.pex` en `.bsa`)* | **Corte C** *(Con `.pex` de mods en `.bsa`)* | Diferencia (`Corte C - Corte B`) |
| :--- | ---: | ---: | ---: |
| **Cadenas únicas netas pendientes (`unique_pending_net`)** | **`235.326`** | **`240.244`** | `+4.918` (`+2,09 %`) |
| **Caracteres únicos en inglés (Sin margen `1,00x`)** | `20.062.309` | `20.314.890` | `+252.581` (`+1,26 %`) |
| **Caracteres únicos en inglés (Con margen `+15 %`)** | `23.071.655` | `23.362.124` | `+290.469` (`+1,26 %`) |
| **Lotes a 25 cadenas/lote (Sin margen / Con margen `+15 %`)** | `9.414` / `10.826` | `9.610` / `11.052` | `+196` / `+226` (`+2,08 %`) |
| **Tokens de Entrada Totales (Sin margen `1,00x`)** | **`13.092.743`** (`13,09 M`) | **`13.323.910`** (`13,32 M`) | `+231.167` (`+1,77 %`) |
| **Tokens de Salida Totales (Sin margen `1,00x`)** | **`9.524.439`** (`9,52 M`) | **`9.681.900`** (`9,68 M`) | `+157.461` (`+1,65 %`) |
| **Tokens de Entrada Totales (Con margen `+15 %`)** | **`15.056.654`** (`15,06 M`) | **`15.322.496`** (`15,32 M`) | `+265.842` (`+1,77 %`) |
| **Tokens de Salida Totales (Con margen `+15 %`)** | **`10.953.105`** (`10,95 M`) | **`11.134.185`** (`11,13 M`) | `+181.080` (`+1,65 %`) |

### 2.2. Tabla Recalculada de Costos — Modelos Vigentes (`Corte B` y `Corte C`)

| Modelo Vigente (`API ID`) | Input / Output Estándar (\$/1M) | Input / Output Batch (\$/1M) | **Corte B Estándar** (Sin / **Con +15%**) | **Corte B Batch** (Sin / **Con +15%**) | **Corte B Costo / 1.000 Cadenas** (Estándar / **Batch +15%**) | **Corte C Estándar** (Sin / **Con +15%**) | **Corte C Batch** (Sin / **Con +15%**) | **Diferencia `C - B`** (Batch +15%) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **`GPT-6 Luna`** (`gpt-6-luna`) | \$0,10 / \$0,50 | \$0,05 / \$0,25 *(50%)* | \$6,07 / **\$6,98** | \$3,04 / **\$3,49** | \$0,0297 / **\$0,0148** | \$6,17 / **\$7,10** | \$3,09 / **\$3,55** | **+\$0,06 USD** *(+1,69%)* |
| **`Gemini 3.5 Flash-Lite`** (`gemini-3.5-flash-lite`) | \$0,30 / \$2,50 | \$0,15 / \$1,25 *(50%)* | \$27,74 / **\$31,90** | \$13,87 / **\$15,95** | \$0,1356 / **\$0,0678** | \$28,20 / **\$32,43** | \$14,10 / **\$16,22** | **+\$0,27 USD** *(+1,67%)* |
| **`Gemini 3.8 Flash`** *(Promo hasta 31/12/2026)* | \$0,75 / \$3,75 | \$0,375 / \$1,875 *(50%)* | \$45,54 / **\$52,37** | \$22,77 / **\$26,18** | \$0,2225 / **\$0,1113** | \$46,30 / **\$53,25** | \$23,15 / **\$26,62** | **+\$0,44 USD** *(+1,68%)* |
| **`GPT-6.1 Terra`** (`gpt-6.1-terra`) | \$1,00 / \$5,00 | \$0,50 / \$2,50 *(50%)* | \$60,71 / **\$69,82** | \$30,36 / **\$34,91** | \$0,2967 / **\$0,1484** | \$61,73 / **\$70,99** | \$30,87 / **\$35,50** | **+\$0,59 USD** *(+1,68%)* |
| **`Gemini 3.8 Flash`** *(Permanente desde 01/01/2027)* | \$1,50 / \$7,50 | \$0,75 / \$3,75 *(50%)* | \$91,07 / **\$104,73** | \$45,54 / **\$52,37** | \$0,4451 / **\$0,2225** | \$92,60 / **\$106,49** | \$46,30 / **\$53,25** | **+\$0,88 USD** *(+1,68%)* |
| **`GPT-6.1 Sol`** (`gpt-6.1-sol`) | \$2,00 / \$10,00 | \$1,00 / \$5,00 *(50%)* | \$121,43 / **\$139,64** | \$60,71 / **\$69,82** | \$0,5934 / **\$0,2967** | \$123,47 / **\$141,99** | \$61,73 / **\$70,99** | **+\$1,17 USD** *(+1,68%)* |
| **`Gemini 4 Argon`** *(Promo hasta 30/11/2026)* | \$2,00 / \$10,00 | \$1,00 / \$5,00 *(50%)* | \$121,43 / **\$139,64** | \$60,71 / **\$69,82** | \$0,5934 / **\$0,2967** | \$123,47 / **\$141,99** | \$61,73 / **\$70,99** | **+\$1,17 USD** *(+1,68%)* |
| **`Gemini 4 Argon`** *(Permanente desde 01/12/2026)* | \$4,00 / \$20,00 | \$2,00 / \$10,00 *(50%)* | \$242,86 / **\$279,29** | \$121,43 / **\$139,64** | \$1,1868 / **\$0,5934** | \$246,93 / **\$283,97** | \$123,47 / **\$141,99** | **+\$2,34 USD** *(+1,68%)* |
| **`GPT-6 Astra`** (`gpt-6-astra`) | \$10,00 / \$50,00 | \$5,00 / \$25,00 *(50%)* | \$607,15 / **\$698,22** | \$303,57 / **\$349,11** | \$2,9670 / **\$1,4835** | \$617,33 / **\$709,93** | \$308,67 / **\$354,97** | **+\$5,86 USD** *(+1,68%)* |
| **`DeepL API Growth`** *(Mensual: \$32,50 + \$27,50/1M)* | \$27,50 / 1M chars | **Sin dto. Batch (0%)** | \$556,71 / **\$639,47** | \$556,71 / **\$639,47** | \$2,7174 / **\$2,7174** | \$563,66 / **\$647,46** | \$563,66 / **\$647,46** | **+\$7,99 USD** *(+1,25%)* |
| **`DeepL API Growth`** *(Anual: \$312 + \$27,50/1M)* | \$27,50 / 1M chars | **Sin dto. Batch (0%)** | \$533,71 / **\$616,47** | \$533,71 / **\$616,47** | \$2,6196 / **\$2,6196** | \$540,66 / **\$624,46** | \$540,66 / **\$624,46** | **+\$7,99 USD** *(+1,30%)* |

### 2.3. Tabla de Trazabilidad — Modelos en Retirada de la Versión 1 *(No Recomendados para Inicio de Proyecto)*

| Modelo en Retirada (`API ID`) | Fecha de Retirada / Migración | **Corte B Estándar** (Sin / **Con +15%**) | **Corte B Batch** (Sin / **Con +15%**) | **Corte C Batch** (Sin / **Con +15%**) | Diferencia `C - B` (Batch +15%) |
| :--- | :--- | :---: | :---: | :---: | :---: |
| `GPT-4o mini` (`gpt-4o-mini-2024-07-18`) | Apagado: `14/01/2027` | \$7,68 / **\$8,83** | \$3,84 / **\$4,42** | \$3,90 / **\$4,49** | +\$0,07 USD |
| `GPT-4.1 mini` (`gpt-4.1-mini`) | Transición Q1 2027 | \$20,48 / **\$23,55** | \$10,24 / **\$11,77** | \$10,41 / **\$11,97** | +\$0,20 USD |
| `Gemini 2.5 Flash` (`gemini-2.5-flash`) | Redirige `01/12/2026` | \$27,74 / **\$31,90** | \$13,87 / **\$15,95** | \$14,10 / **\$16,22** | +\$0,27 USD |
| `GPT-4.1` (`gpt-4.1-2025-04-14`) | Apagado: `14/01/2027` | \$102,38 / **\$117,74** | \$51,19 / **\$58,87** | \$52,05 / **\$59,86** | +\$0,99 USD |
| `Gemini 2.5 Pro` (`gemini-2.5-pro`) | Retirado Vertex `20/10/2026` | \$111,61 / **\$128,35** | \$55,81 / **\$64,18** | \$56,74 / **\$65,25** | +\$1,07 USD |
| `GPT-5 Base` (`gpt-5`) | Apagado: `11/12/2026` | \$111,61 / **\$128,35** | \$55,81 / **\$64,18** | \$56,74 / **\$65,25** | +\$1,07 USD |
| `DeepL API Pro` *(Legacy)* | Plan retirado | \$507,05 / **\$582,28** | \$507,05 / **\$582,28** | \$513,36 / **\$589,54** | +\$7,26 USD |

---

## 3. Corrección Matemática del Tiempo de Inferencia Local (`RX 9070 XT 16 GB VRAM`)

> [!IMPORTANT]
> **Advertencia Metodológica**: No se ha descargado ni ejecutado ningún modelo local en tu equipo. Las velocidades en tokens por segundo ($R_{\text{gen}}$ y $R_{\text{prefill}}$) que se emplean a continuación son **estimaciones teóricas** basadas en el ancho de banda de memoria de una GPU arquitectura RDNA 4 de 16 GB GDDR6 (`~640 GB/s`). **La velocidad real en tu RX 9070 XT queda explícitamente pendiente de medición práctica mediante el benchmark local autorizado**.

### 3.1. Explicación de la Inconsistencia Corregida
En el informe original (`v1`), se habían proyectado `9.524.439` tokens de salida y una velocidad de `55–70 tokens/s`, pero se había indicado erróneamente un tiempo de `12–15 horas` por haber multiplicado `9.414` lotes por `~5 s/lote` (lo cual asumía solo `~300` tokens de salida por lote en lugar de los **`1.011,73` tokens de salida reales por lote**).

Aplicando la fórmula matemática exacta:
$$T_{\text{generación}} = \frac{\text{Tokens Totales de Salida}}{R_{\text{gen}} \, (\text{tokens/s})}, \quad T_{\text{contexto}} = \frac{\text{Tokens Totales de Entrada}}{R_{\text{prefill}} \, (\text{tokens/s})}$$
$$T_{\text{total}} = T_{\text{generación}} + T_{\text{contexto}} + T_{\text{carga\_modelo}} + T_{\text{almacenamiento\_SQLite\_QA}}$$

Por ejemplo, para **Corte B con margen del +15 %** (`10.953.105` tokens de salida y `15.056.654` tokens de entrada en `10.826` lotes):
- A **$70\text{ tokens/s}$** de generación y $1.300\text{ tokens/s}$ de procesamiento de contexto:
  - $T_{\text{generación}} = 10.953.105 / 70 = 156.472,9\text{ s} = \mathbf{43,46\text{ horas}}$.
  - $T_{\text{contexto}} = 15.056.654 / 1.300 = 11.582,0\text{ s} = \mathbf{3,22\text{ horas}}$.
  - $T_{\text{carga\_modelo}} = 90\text{ s acumulados entre sesiones} = \mathbf{0,03\text{ horas}}$.
  - $T_{\text{almacenamiento\_SQLite\_QA}} = 10.826 \times 0,14\text{ s} = 1.515,6\text{ s} = \mathbf{0,42\text{ horas}}$.
  - **$T_{\text{total}} = 43,46 + 3,22 + 0,03 + 0,42 = \mathbf{47,13\text{ horas}}$** ($\approx 5,9$ jornadas nocturnas de 8 horas).

### 3.2. Desglose Completo de Tiempos Teóricos de Ejecución Local

| Modelo Local y Perfil Teórico | Huella Estimada VRAM | Corte y Margen | Vel. Teórica Gen / Contexto (`tok/s`) | 1. Tiempo Generación ($T_{\text{gen}}$) | 2. Tiempo Contexto ($T_{\text{prefill}}$) | 3. Tiempo Carga Modelo ($T_{\text{load}}$) | 4. Tiempo QA + SQLite ($T_{\text{storage}}$) | **5. TIEMPO TOTAL EJECUCIÓN ($T_{\text{total}}$)** | Jornadas de 8 h | Costo Eléctrico Est. (`~320W`) |
| :--- | :---: | :--- | :---: | ---: | ---: | ---: | ---: | ---: | :---: | ---: |
| **Gemma 3 12B IT (`Q5_K_M`)** — *Teórico Optimista* | `11,2 GB` *(100% VRAM)* | Corte B (Sin margen) | `70` / `1.300` | `37,80 h` | `2,80 h` | `0,03 h` | `0,37 h` | **`40,98 horas`** | `5,1 noches` | \$2,03 USD |
| **Gemma 3 12B IT (`Q5_K_M`)** — *Teórico Optimista* | `11,2 GB` *(100% VRAM)* | **Corte B (+15% margen)** | `70` / `1.300` | **`43,46 h`** | **`3,22 h`** | **`0,03 h`** | **`0,42 h`** | **`47,13 horas`** | **`5,9 noches`** | **\$2,34 USD** |
| **Gemma 3 12B IT (`Q5_K_M`)** — *Teórico Optimista* | `11,2 GB` *(100% VRAM)* | **Corte C (+15% margen)** | `70` / `1.300` | **`44,18 h`** | **`3,27 h`** | **`0,03 h`** | **`0,43 h`** | **`47,91 horas`** | **`6,0 noches`** | **\$2,38 USD** |
| **Gemma 3 12B IT (`Q5_K_M`)** — *Teórico Conservador* | `11,2 GB` *(100% VRAM)* | Corte B (Sin margen) | `55` / `900` | `48,10 h` | `4,04 h` | `0,03 h` | `0,42 h` | **`52,60 horas`** | `6,6 noches` | \$2,69 USD |
| **Gemma 3 12B IT (`Q5_K_M`)** — *Teórico Conservador* | `11,2 GB` *(100% VRAM)* | **Corte B (+15% margen)** | `55` / `900` | **`55,32 h`** | **`4,65 h`** | **`0,03 h`** | **`0,48 h`** | **`60,48 horas`** | **`7,6 noches`** | **\$3,10 USD** |
| **Gemma 3 12B IT (`Q5_K_M`)** — *Teórico Conservador* | `11,2 GB` *(100% VRAM)* | **Corte C (+15% margen)** | `55` / `900` | **`56,23 h`** | **`4,73 h`** | **`0,03 h`** | **`0,49 h`** | **`61,48 horas`** | **`7,7 noches`** | **\$3,15 USD** |
| **Qwen 2.5 14B IT (`Q5_K_M`)** — *Teórico Optimista* | `12,6 GB` *(100% VRAM)* | **Corte B (+15% margen)** | `65` / `1.200` | **`46,81 h`** | **`3,49 h`** | **`0,03 h`** | **`0,42 h`** | **`50,74 horas`** | **`6,3 noches`** | **\$2,60 USD** |
| **Qwen 2.5 14B IT (`Q5_K_M`)** — *Teórico Conservador* | `12,6 GB` *(100% VRAM)* | **Corte B (+15% margen)** | `50` / `850` | **`60,85 h`** | **`4,92 h`** | **`0,04 h`** | **`0,48 h`** | **`66,29 horas`** | **`8,3 noches`** | **\$3,45 USD** |
| **Mistral Small 24B (`IQ4_XS`)** — *Teórico Optimista* | `15,4 GB` *(Límite VRAM)* | **Corte B (+15% margen)** | `42` / `800` | **`72,44 h`** | **`5,23 h`** | **`0,04 h`** | **`0,48 h`** | **`78,19 horas`** | **`9,8 noches`** | **\$4,25 USD** |
| **Mistral Small 24B (`IQ4_XS`)** — *Teórico Conservador* | `15,6 GB` *(Límite VRAM)* | **Corte B (+15% margen)** | `32` / `550` | **`95,08 h`** | **`7,60 h`** | **`0,05 h`** | **`0,54 h`** | **`103,27 horas`** | **`12,9 noches`** | **\$5,62 USD** |
| **Qwen 2.5 32B / Llama 3.3 70B** *(Offload parcial a RAM)* | `>16 GB` *(Desborda PCIe)* | **Corte B (+15% margen)** | `6,5` / `130` | `468,08 h` | `32,17 h` | `0,07 h` | `0,54 h` | **`500,86 horas`** | `62,6 noches` | \$22,44 USD |

---

## 4. Evaluación Objetiva de Calidad Lingüística

En lugar de asignar calificativos subjetivos predefinidos (*"buena"*, *"muy buena"* o *"excelente"*) sin evidencia empírica sobre las cadenas reales de Nolvus Awakening, el desempeño lingüístico de cada candidato se determina exclusivamente mediante el **Benchmark Estandarizado de 9 Dominios y 8 Dimensiones** especificado en **[MODEL_BENCHMARK_PLAN.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/MODEL_BENCHMARK_PLAN.md)**.

Ningún modelo recibirá una etiqueta de calidad definitiva hasta que autorices la ejecución controlada del conjunto de prueba (`90` cadenas estratificadas / `~3.800` tokens) y se midan sus puntuaciones reales en:
1. Naturalidad `ES-419` (ausencia total de voseo regional, españolismos peninsulares y calcos robóticos).
2. Fidelidad semántica al original.
3. Coherencia narrativa y preservación del registro del hablante.
4. Respeto al lore de *The Elder Scrolls*.
5. Consistencia estricta con el Glosario Maestro.
6. Conservación de nombres propios protegidos (`keep_original = True`).
7. Protección inalterada de variables y etiquetas (`⟦VAR_i⟧`).
8. Integridad estructural del formato de salida.

---

## 5. Revisión Técnica de *Structured Outputs* y Validación Posterior Obligatoria

Se corrige de forma expresa la afirmación de la Versión 1 según la cual el uso de *Structured Outputs* (`response_format: json_schema` en OpenAI o `response_schema` en Gemini) garantiza *"0 % de errores"*.

### 5.1. Distinción de los 5 Niveles de Fallo en Respuestas de Modelos IA

| Nivel Técnico | ¿Lo garantiza `json_schema` / *Structured Outputs*? | Modos de Fallo Reales que `json_schema` NO Evita | Validación Posterior Implementada en Nuestro Pipeline ([`TranslationPipeline`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/translation/pipeline.py) + [`TranslationQualityValidator`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/quality/validator.py)) |
| :--- | :---: | :--- | :--- |
| **1. Validez sintáctica JSON** | Sí *(salvo truncamiento)* | Si el modelo alcanza `max_output_tokens` (`finish_reason = "length"`), el flujo JSON se corta a mitad de cadena produciendo un `JSONDecodeError`. | Verificación de `finish_reason == "stop"` antes de parsear; si ocurre truncamiento, el lote se divide automáticamente a la mitad (*batch splitting*) y se reintenta. |
| **2. Cumplimiento del esquema (`schema`)** | Sí *(estructural)* | La gramática libre de contexto obliga a devolver `{"items": [{"id": "...", "translation": "..."}]}`, pero **no impide** que el modelo devuelva menos elementos de los enviados (p. ej. 23 de 25), duplique un `id` o invente un `id` inexistente. | Cruce biyectivo estricto de conjuntos: `set(returned_ids) == set(requested_ids)` y `len(returned_items) == len(requested_items)`. Cualquier omisión o desfase de ID rechaza el ítem faltante y lo reencola. |
| **3. Respuesta rechazada o truncada (`refusal` / `safety`)** | **No** | Textos oscuros propios de Skyrim (asesinatos de la Hermandad Oscura, sacrificios daédricos de Molag Bal/Boethiah, canibalismo de Namira, desmembramientos) pueden disparar un `refusal` o bloqueo `PROHIBITED_CONTENT`. | Detección explícita de bandera `refusal` / `safety_ratings`; aislamiento de la cadena individual conflictiva fuera del lote de 25 para no perder las otras 24 cadenas, y marcado para revisión o proveedor local sin censura. |
| **4. Integridad de variables y etiquetas técnicas** | **No** | Dentro de un string JSON perfectamente válido, el modelo puede omitir `⟦VAR_0⟧`, alterar `<Alias=Player>` a `<Alias=Jugador>`, romper `<font face='$EverywhereMediumFont'>` o borrar `[pagebreak]`. | Puerta dura (`Hard Gate`) en [`VariableProtector.restore_and_verify()`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/quality/protector.py): conteo exacto de cada token `⟦VAR_i⟧`, balanceo de etiquetas XML/Scaleform y verificación de codificación. Si falla, la traducción se rechaza automáticamente (`passed_hard_gates = False`). |
| **5. Traducción semántica y lingüísticamente correcta** | **No** | Un JSON válido puede contener una cadena vacía `""`, devolver el texto en inglés sin traducir, usar *"vosotros habéis"*, traducir *Delphine* como *Delfina* o alucinar texto adicional. | Auditoría post-respuesta en [`TranslationQualityValidator.validate()`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/quality/validator.py): detector de español peninsular (`vosotros/os/habéis`), verificador de términos obligatorios del Glosario Maestro, detector de texto idéntico al inglés y control de ratio de longitud ($0,35\times \le \text{ratio} \le 2,20\times$). |

---

## 6. Análisis Detallado de DeepL API y Condiciones para Cuenta en Colombia

### 6.1. Planes Actuales Verificados vs. Planes Antiguos
1. **`DeepL API Developer` (Plan Gratuito actual)**:
   - **Cuota**: Otorga un fondo único (*one-time pool*) de **1.000.000 de caracteres en total** para pruebas de integración (a diferencia del antiguo plan *API Free* que renovaba 500.000 caracteres cada mes). Insuficiente para Nolvus (`20,06 M` de caracteres).
2. **`DeepL API Growth` (Plan de Producción vigente que sustituyó al antiguo `API Pro` estándar)**:
   - **Modalidad Mensual**: Tarifa base fija de **\$32,50 USD/mes** que incluye el primer **1.000.000 de caracteres del mes**, y cobra **\$27,50 USD por cada millón de caracteres adicional** (*overage*).
     - Cálculo Corte B Sin Margen (`20.062.309` chars): $\$32,50 + (19,062309 \times \$27,50) = \mathbf{\$556,71\text{ USD}}$.
     - Cálculo Corte B Con Margen +15 % (`23.071.655` chars): $\$32,50 + (22,071655 \times \$27,50) = \mathbf{\$639,47\text{ USD}}$.
   - **Modalidad Anual**: Pago adelantado de **\$312,00 USD/año** (\$26,00/mes) que incluye **12.000.000 de caracteres al año**, más **\$27,50 USD por millón adicional** (Corte B con margen +15 %: **\$616,47 USD**).
   - **Descuento Batch**: **0 %** (DeepL factura por carácter enviado sin descuento por procesamiento asíncrono).
3. **`DeepL API Pro` (Plan Antiguo Legacy)**:
   - Cobraba **\$5,49 USD/mes** de suscripción base (sin caracteres incluidos) + **\$25,00 USD por millón de caracteres** (\$507,05 USD sin margen / \$582,28 USD con margen +15 % en Corte B). Ya no se ofrece en el alta estándar para nuevos suscriptores comerciales.

### 6.2. Consideraciones Específicas de Facturación para tu Cuenta en Colombia
- **Disponibilidad Regional**: Colombia figura oficialmente en la lista de países soportados por DeepL para cuentas de pago.
- **Diferencias entre la Tarifa Estadounidense Publicada y el Precio Final en Colombia**:
  1. **Divisa de Facturación (`USD` vs `EUR`)**: Dependiendo del enrutamiento regional de la pasarela de pago de DeepL SE (Alemania) para Sudamérica, la cuenta puede facturarse en dólares estadounidenses (`USD`) o en euros (`EUR`, donde la base es `29,75 €/mes` y el millón adicional es `22,00 €/1M`).
  2. **Impuesto sobre el Valor Agregado (IVA Digital en Colombia — 19 %)**: En compras de servicios digitales transfronterizos desde Colombia, el proveedor extranjero o la entidad bancaria emisora de la tarjeta en Colombia puede aplicar la retención/cobro del **19 % de IVA** sobre el valor neto de la factura (lo que elevaría \$639,47 USD a **~\$760,97 USD**).
  3. **Conversión Cambiaria (`TRM`) y Comisiones Bancarias**: El cargo en pesos colombianos (`COP`) estará sujeto a la tasa de cambio vigente el día del corte más el margen cambiario internacional (`1 %–3 %`) de tu banco emisor. Además, para cuentas en Colombia, DeepL exige tarjeta de crédito o débito internacional habilitada para compras en el exterior (no admite transferencias bancarias locales / PSE en planes mensuales).

---

## 7. Recomendación Final Comparativa y Criterios de Justificación

Sin elegir automáticamente la opción más barata, a continuación se comparan las **5 categorías estratégicas** y qué evidencia empírica concreta del benchmark ([MODEL_BENCHMARK_PLAN.md](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/MODEL_BENCHMARK_PLAN.md)) se requiere para justificar cada una:

| Categoría Estratégica | Candidato(s) Principales | Costo Corte B (Sin / **Con +15%**) | Costo Corte C (Sin / **Con +15%**) | ¿Qué Resultados Reales en el Benchmark Justificarían Elegir esta Opción? |
| :--- | :--- | :---: | :---: | :--- |
| **1. Mejor Costo Absoluto (Cloud)** | **`GPT-6 Luna` (`gpt-6-luna`) — Batch API** | \$3,04 / **\$3,49 USD** | \$3,09 / **\$3,55 USD** | Se justifica como motor único **solo si** en el benchmark alcanza **100 % de preservación de variables/etiquetas**, **$\ge 95\,\%$ de cumplimiento del Glosario Maestro** y no pierde naturalidad literaria ni tono de personaje en la categoría 7 (*Libros de lore*) y categoría 8 (*Expresiones ambiguas*). |
| **2. Mejor Calidad Potencial (Monomodelo)** | **`GPT-6.1 Sol` (`gpt-6.1-sol`)** o **`Gemini 4 Argon` (`gemini-4-argon` Promo Nov 2026)** — Batch API | \$60,71 / **\$69,82 USD** *(Argon sube a \$139,64 el 01/12/2026)* | \$61,73 / **\$70,99 USD** | Solo se justifica pagar ~\$70 USD en lugar de \$7–\$22 USD **si** el benchmark demuestra que tanto `GPT-6 Luna` como `Gemini 3.5 Flash-Lite` y `GPT-6.1 Terra` cometen errores frecuentes de concordancia de género, registro de cortesía (`tú`/`usted`) o sentido narrativo incluso en diálogos cotidianos y objetivos de misión. |
| **3. Mejor Equilibrio (Monomodelo)** | **`Gemini 3.5 Flash-Lite` Batch** (\$15,95) / **`Gemini 3.8 Flash Promo` Batch** (\$26,18) / **`GPT-6.1 Terra` Batch** (\$34,91) | \$13,87–\$30,36 / **\$15,95–\$34,91 USD** | \$14,10–\$30,87 / **\$16,22–\$35,50 USD** | Se justifica si un único modelo intermedio resuelve con alta fidelidad tanto la interfaz/objetos como los libros largos, evitando la complejidad de enrutar lotes entre dos modelos distintos. En el caso de Gemini, requiere verificar en el benchmark que fijar `thinking_budget = 0` mantiene el costo sin inflar los tokens de salida. |
| **4. Alternativa Local Gratuita (`0 %` API)** | **`Gemma 3 12B IT` o `Qwen 2.5 14B IT` (`Q5_K_M`)** en tu **RX 9070 XT (16 GB VRAM)** | **\$0,00 USD API** *(~\$2,34–\$3,45 USD electricidad; `47 a 66 h`)* | **\$0,00 USD API** *(~\$2,38–\$3,51 USD electricidad; `48 a 67 h`)* | Se justifica si deseas independencia total de APIs externas y en la prueba práctica local tu **RX 9070 XT** confirma una velocidad sostenida $\ge 55\text{ tokens/s}$ (completable en 6–8 noches con pausa/reanudación por checkpoints) y el modelo local logra **0 % de rotura de placeholders `⟦VAR_i⟧`** y **0 % de modismos peninsulares**. |
| **5. Alternativa Híbrida (88 % Volumen Estándar + 12 % Narrativo/Lore Complejo)** | • **Híbrido OpenAI**: `GPT-6 Luna Batch` (88 %) + `GPT-6.1 Terra Batch` (12 %) -> **\$7,26 USD**<br>• **Híbrido OpenAI Literario**: `GPT-6 Luna Batch` (88 %) + `GPT-6.1 Sol Batch` (12 %) -> **\$11,45 USD**<br>• **Híbrido Gemini**: `3.5 Flash-Lite Batch` (88 %) + `3.8 Flash Promo Batch` (12 %) -> **\$17,18 USD**<br>• **Híbrido Local + Cloud**: `RX 9070 XT` (88 %) + `GPT-6.1 Sol / Argon Batch` (12 %) -> **\$8,38 USD** | \$6,31–\$14,94 / **\$7,26–\$17,18 USD** | \$6,42–\$15,19 / **\$7,38–\$17,46 USD** | **Arquitectónicamente la opción más sólida**: el 88 % del volumen de Nolvus son cadenas cortas/medias estructuradas (nombres de armas, armaduras, efectos, menús MCM y diálogos breves) donde un modelo ligero suele empatar con uno frontera, mientras que el 12 % restante (`1.400` libros de lore, `2.503` textos largos y `718` cadenas ambiguas) recibe toda la capacidad literaria de un modelo superior por apenas **\$7,26 a \$17,18 USD en total**. |
