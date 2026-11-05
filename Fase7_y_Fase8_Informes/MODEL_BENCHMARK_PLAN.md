> **BENCHMARK ORIGINAL — SUSTITUIDO:** Este plan de 90 cadenas corresponde a una estrategia anterior orientada a APIs y modelos locales. Ha sido sustituido para la evaluación Antigravity-first por [BENCHMARK_PLAN_ANTIGRAVITY.md](../Fase10_Antigravity_Architecture/BENCHMARK_PLAN_ANTIGRAVITY.md), que define el corpus y los candidatos confirmados para aquel diseño. Conservar este archivo únicamente como antecedente; no mezclar sus modelos, muestras, costes ni criterios con el benchmark vigente.

---

# PLAN DE BENCHMARK LINGÜÍSTICO Y TÉCNICO DE MODELOS (`ES-419`)
**Proyecto**: NOLVUS AI TRANSLATOR — Nolvus Awakening 6.0.21  
**Estado**: Diseñado y Listo para Revisión (0 llamadas a APIs externas realizadas; 0 modelos locales descargados)  
**Informe Económico Asociado**: `c:/Skyrim proyecto traducción/API_COST_ESTIMATE_V2.md` (ruta/artefacto local no incluido en GitHub) y `c:/Skyrim proyecto traducción/API_COST_COMPARISON_V2.xlsx` (ruta/artefacto local no incluido en GitHub)

---

## 1. Objetivo del Benchmark

Evaluar de forma empírica, ciega, reproducible y cuantitativa a los modelos candidatos (`Gemini 3.5 Flash-Lite`, `Gemini 3.8 Flash`, `Gemini 4 Argon`, `GPT-6 Luna`, `GPT-6.1 Terra`, `GPT-6.1 Sol` y los modelos locales `Gemma 3 12B IT` / `Qwen 2.5 14B IT` en tu `RX 9070 XT`) sobre **un mismo corpus fijo de cadenas originales en inglés** extraídas de `Snapshot #1`, antes de comprometer presupuesto o tiempo de cómputo en las `235.326` cadenas de producción.

> [!IMPORTANT]
> **Garantía de Cumplimiento de Restricciones**: Este documento define el diseño completo de la prueba, las muestras reales extraídas de tu base de datos local (`c:/Skyrim proyecto traducción/workspace_data/nolvus_translator.db` (ruta/artefacto local no incluido en GitHub)) y la rúbrica matemática de calificación. **Ningún texto ha sido enviado a ningún proveedor externo ni se ha consumido ninguna API**.

---

## 2. Estructura del Corpus de Prueba (`90` Cadenas Estratificadas en las 9 Categorías Obligatorias)

El conjunto de prueba consta de **90 cadenas reales** (10 cadenas por cada una de las 9 categorías exigidas, sumando `~3.800` tokens de entrada y `~2.900` tokens de salida por modelo evaluado, con un costo de prueba inferior a `\$0,03 USD` por modelo en nube cuando se autorice).

A continuación se especifican las **9 categorías** y ejemplos representativos reales del corpus junto con el reto lingüístico/técnico exacto que miden:

### Categoría 1 — Objetos e Inventario (`WEAP:FULL`, `ARMO:FULL`, `ALCH:FULL`, `MISC:FULL`)
- **Muestras de referencia**:
  1. `"Iron Greatsword of Scorching"` (`WEAP:FULL`)
  2. `"Ancient Nord Helmet of Unburned"` (`ARMO:FULL`)
  3. `"Potion of Extreme Stamina"` (`ALCH:FULL`)
  4. `" blackened steel ingot"` (`MISC:FULL` — incluye espacio inicial intencional de ordenación en inventario)
- **Qué evalúa**: Orden sintáctico natural en español (`Mandoble de hierro de abrasión`, no calcos literales), uso estricto del Glosario Maestro (`Stamina` -> `Aguante`, nunca *Estamina* ni *Resistencia*), y conservación de espacios iniciales de ordenación de inventario cuando aplica.

### Categoría 2 — Nombres de Hechizos y Gritos (`SPEL:FULL`, `SHOU:FULL`, `WOOP:TNAM`)
- **Muestras de referencia**:
  1. `"Conjure Flame Atronach"` (`SPEL:FULL`)
  2. `"Icy Spear"` (`SPEL:FULL`)
  3. `"Wither"` (`SPEL:FULL` — *Apocalypse / Mysticism*)
  4. `"Throw Voice"` (`SHOU:FULL`)
- **Qué evalúa**: Distinción entre verbo imperativo e infinitivo/sustantivo en nombres de hechizos (`Conjurar atronach de las llamas`, `Lanza helada`, `Marchitar`), evitando confundir nombres de hechizos con órdenes de diálogo.

### Categoría 3 — Descripciones de Habilidades, Ventajas (`PERK:DESC`) y Efectos Mágicos (`MGEF:DNAM`)
- **Muestras de referencia**:
  1. `"Two-handed weapons do <mag>% more damage, and power attacks with two-handed weapons cost <1>% less Stamina."` (`PERK:DESC`)
  2. `"Targets on fire take <mag> points of bonus damage for <dur> seconds."` (`MGEF:DNAM`)
  3. `"Grants the ability to craft Dwarven armor and weapons at the forge, and improves them twice as much."` (`PERK:DESC`)
- **Qué evalúa**: Concisión técnica, preservación exacta de `<mag>`, `<dur>`, `<1>` y del símbolo `%` adyacente (`<mag>%`), y traducción canónica de *Dwarven* (`enano/a` o `dwemer` según contexto de herrería oficial).

### Categoría 4 — Diálogos Cotidianos (`INFO:NAM1` — Registro Coloquial / Tabernas / Ciudadanos / Guardias)
- **Muestras de referencia**:
  1. `"Watch your step around here, stranger. The guards in Riften don't work for free, if you catch my drift."` (`INFO:NAM1`)
  2. `"Pull up a chair by the fire! What'll it be? Honningbrew mead, or something stronger?"` (`INFO:NAM1`)
  3. `"You guys better clear out before the Jarl's men arrive. Take your gear and go!"` (`INFO:NAM1`)
- **Qué evalúa**: Neutralidad `ES-419` en habla cotidiana (uso obligatorio de **ustedes** en la muestra 3: *«Será mejor que ustedes se marchen... Tomen su equipo y váyanse»*, penalizando con `0` cualquier uso de *vosotros/tomad/id* o modismos como *coger*, *vale*, *tío*, *ahorita*, *che*).

### Categoría 5 — Diálogos Narrativos y Solemnes (`INFO:NAM1` — Jarls, Barbas Grises, Príncipes Daedra, Compañeros)
- **Muestras de referencia**:
  1. `"Speak, mortal. You stand before the shrine of Azura, Mother of the Rose and Queen of the Night Sky. Why do you seek her favor?"` (`INFO:NAM1`)
  2. `"My Jarl, the scouts from Windhelm confirm it. Ulfric Stormcloak marches toward Whiterun with his veteran legions."` (`INFO:NAM1`)
  3. `"Inigo is glad you are still breathing, my friend. Shall we continue our journey to High Hrothgar?"` (`INFO:NAM1` — *Inigo*)
- **Qué evalúa**: Selección adecuada de `tú` vs `usted` según jerarquía narrativa (deferencia ante un Jarl en la muestra 2; solemnidad arcaica/divina en la muestra 1; cercanía amistosa y rasgo de hablar en tercera persona de *Inigo* en la muestra 3), además de aplicar el Glosario Maestro (*Windhelm* -> *Ventalia*, *Ulfric Stormcloak* -> *Ulfric Capa de la Tormenta*, *Whiterun* -> *Carrera Blanca*, *High Hrothgar* -> *Alto Hrothgar*).

### Categoría 6 — Misiones, Etapas de Diario y Objetivos (`QUST:FULL`, `QUST:NNAM`, `QUST:CNAM`)
- **Muestras de referencia**:
  1. `"Retrieve the Horn of Jurgen Windcaller from Ustengrav"` (`QUST:NNAM`)
  2. `"Speak to <Alias=Jarl> in <Alias=HoldCapital>"` (`QUST:NNAM`)
  3. `"I spoke with the curator at the Dragonborn Gallery in Solitude. He asked me to recover three lost relics from the ruins of Blackreach before the Thalmor expedition finds them."` (`QUST:CNAM` — *Legacy of the Dragonborn*)
- **Qué evalúa**: Redacción de diario en primera persona **sin asumir género masculino/femenino para el jugador** (*«Hablé con el curador... Me pidió recuperar...»* en lugar de expresiones marcadas por género), formato verbal consistente en objetivos de misión y preservación intacta de `<Alias=Jarl>` y `<Alias=HoldCapital>`.

### Categoría 7 — Libros, Cartas, Notas y Documentos de Lore (`BOOK:DESC`)
- **Muestras de referencia**:
  1. Fragmento de crónica histórica imperial con etiquetas de formato y salto de página:
     `"<p align='center'><font face='$HandwrittenFont' size='22'>On the Oblivion Crisis</font></p>[pagebreak]<p align='left'>When the fires of Mehrunes Dagon engulfed the Imperial City in 3E 433, Martin Septim shattered the Amulet of Kings and invoked the avatar of Akatosh...</p>"`
  2. Poema/canción de bardo con rima o ritmo métrico (*Interesting NPCs / Beyond Skyrim: Bruma*).
  3. Nota manuscrita de un bandido con tono rudo y urgente.
- **Qué evalúa**: Registro literario elevado en crónicas de lore, preservación de todas las etiquetas HTML/Scaleform (`<p align='...'>`, `<font face='$HandwrittenFont' size='22'>`, `[pagebreak]`) sin traducir el nombre de la fuente `$HandwrittenFont`, y adaptación natural de textos largos.

### Categoría 8 — Expresiones Ambiguas, Polisemia y Términos Sin Contexto Aparente
- **Muestras de referencia**:
  1. `"Fire"` en un registro `SPEL:FULL` vs `"Fire!"` en un registro `INFO:NAM1` (orden de disparo de arqueros imperiales) vs `"Fire"` en `MCM_TXT` (elemento mágico).
  2. `"Charge"` en `WEAP:DESC` (carga de encantamiento) vs `"Charge!"` en `INFO:NAM1` (carga de caballería).
  3. `"Light"` en `ARMO` (armadura ligera) vs `"Light"` en `LIGH` / `SPEL` (luz / iluminar).
  4. `"Draw"` en `MCM_TXT` (desenfundar arma) vs `"Draw"` (empate / dibujar).
- **Qué evalúa**: Capacidad real del modelo para aprovechar los metadatos contextuales que le inyecta nuestro pipeline (`record_type`, `subrecord_type`, `content_domain`, `speaker_hint`, `context_group_id`) para desambiguar términos polisémicos cortos.

### Categoría 9 — Variables Complejas, Especificadores y Etiquetas Especiales (`MCM_TXT`, `INI_LOC`, `PEX`)
- **Muestras de referencia**:
  1. `"$TDM_TargetLock_Key\tTarget Lock Hotkey ({0}) [Default: {1}]"` (`MCM_TXT`)
  2. `"Weight: %.1f | Value: %d septims (<Alias.ShortName=Follower> carries %.0f%%)"` (`PEX` / `MESG`)
  3. `"<font color='#FF4444'>WARNING:</font> Updating Sanguine Symphony from v%.0f.%.0f.%.0f to v%.0f.%.0f.%.0f requires a clean save."` (`MESG:DESC`)
- **Qué evalúa**: Preservación al 100 % de todos los marcadores enmascarados (`⟦VAR_0⟧`..`⟦VAR_n⟧`), doble signo de porcentaje (`%.0f%%`), colores hexadecimales (`#FF4444`) y no traducción de claves `$TDM_TargetLock_Key`.

---

## 3. Rúbrica Matemática de Evaluación (8 Criterios Auditables)

Cada una de las 90 cadenas traducidas por un modelo recibe una calificación entre `0,0` y `100,0` calculada mediante dos niveles: **Puertas Duras Eliminatorias (*Hard Gates*)** y **Puntuación Ponderada de Calidad**.

### 3.1. Nivel 1 — Puertas Duras (*Hard Gates* Automáticas)
Si una cadena falla cualquiera de estos dos criterios técnicos, su puntuación técnica cae automáticamente a `0` y cuenta como **Fallo Crítico de Integridad**:
1. **Protección de Variables y Etiquetas (`Criterio 7`)**: El 100 % de los placeholders `⟦VAR_i⟧`, etiquetas `<Alias=...>`, `<font ...>`, `[pagebreak]`, `{0}` y `%s`/`%.1f%%` debe restaurarse sin pérdida, duplicación ni alteración.
2. **Integridad del Formato de Salida (`Criterio 8`)**: La respuesta del lote debe ser JSON sintácticamente válido (`finish_reason == "stop"`), cumplir el esquema, contener exactamente los mismos IDs solicitados sin omisiones ni truncamientos, y no devolver cadenas vacías.

### 3.2. Nivel 2 — Matriz de Ponderación de los 8 Criterios (`0` a `100` puntos)

| # | Criterio de Evaluación | Peso (%) | Método de Medición (Automático + Revisión Ciega) | Penalización por Incumplimiento |
| :-: | :--- | :---: | :--- | :--- |
| **1** | **Naturalidad `ES-419`** | **20 %** | Auditoría léxica y sintáctica: ausencia de `vosotros/os/habéis`, ausencia de regionalismos de España (*coger*, *vale*, *tío*) y ausencia de modismos locales (*ahorita*, *che*, *parce*, *cachai*); fluidez natural sin calcos sintácticos del inglés. | `-50 pts` por uso de *vosotros*; `-25 pts` por regionalismo marcado o calco literal antinatural. |
| **2** | **Fidelidad al Original** | **15 %** | Precisión semántica completa: sin omisión de información, sin adición de datos inventados (*hallucinations*) y sin inversión del sentido condicional o numérico. | `-40 a -100 pts` según gravedad de la distorsión de significado. |
| **3** | **Coherencia Narrativa y Registro** | **15 %** | Adecuación del tono al personaje y situación (`tú` vs `usted`, solemnidad en príncipes Daedra/Jarls, neutralidad de género para el protagonista en entradas de diario `QUST:CNAM`, concisión en UI/MCM). | `-30 pts` por pérdida de tono del personaje o asunción indebida de género del jugador. |
| **4** | **Respeto al Lore de *The Elder Scrolls*** | **15 %** | Uso correcto de conceptos cosmológicos, históricos y mágicos de Tamriel (*Aetherius*, *Oblivion*, *Kalpa*, *CHIM*, *Thu'um*, *Draugr*, *Dwemer*) y desambiguación correcta en la Categoría 8. | `-40 pts` por error conceptual de lore o desambiguación fallida (*Fire* hechizo vs orden). |
| **5** | **Consistencia del Glosario Maestro** | **15 %** | Verificación automática contra `src/nolvus_translator/lore/glossary.py` (código fuente local no publicado en este repositorio): todo término del Glosario Maestro presente en la cadena original debe aparecer con su `approved_translation` exacta. | `-50 pts` por cada término obligatorio del glosario ignorado o traducido de forma divergente. |
| **6** | **Conservación de Nombres Propios** | **10 %** | Verificación automática de entidades con `keep_original = True` (*Delphine*, *Paarthurnax*, *Serana*, *Inigo*, *Lucien*, *Jorrvaskr*, *Sovngarde*, *Apocrypha*). | `-100 pts` en este criterio si se traduce o hispaniza un nombre propio protegido. |
| **7** | **Protección de Variables y Etiquetas** | **5 %** *(+ Hard Gate)* | Verificación automática por `src/nolvus_translator/quality/protector.py` (código fuente local no publicado en este repositorio). | **Eliminatorio** (marca la cadena como `INCORRECT` y exige reintento). |
| **8** | **Integridad del Formato de Salida** | **5 %** *(+ Hard Gate)* | Verificación de validez JSON, cumplimiento de esquema, completitud de IDs del lote, ausencia de truncamiento y ratio de longitud visual en UI ($\le 1,35\times$). | **Eliminatorio** si falta el ID o el JSON está roto; `-25 pts` si excede el límite visual de UI. |

---

## 4. Protocolo de Ejecución Ciega (Solo cuando el Usuario lo Autorice)

1. **Extracción Determinista del Corpus (`benchmark_dataset_90.json`)**:
   - Se seleccionan las 90 cadenas reales desde `Snapshot #1` en `c:/Skyrim proyecto traducción/workspace_data/nolvus_translator.db` (ruta/artefacto local no incluido en GitHub) con su contexto completo (`record_type`, `subrecord_type`, `content_domain`, `speaker_hint`, términos aplicables del Glosario Maestro y tokens `⟦VAR_i⟧` enmascarados).
2. **Evaluación en Igualdad Estricta de Condiciones**:
   - Todos los modelos autorizados reciben **exactamente el mismo prompt de sistema `ES-419`**, **el mismo subconjunto del Glosario Maestro** y **la misma agrupación en lotes**.
   - Para los modelos locales en tu **RX 9070 XT** (si autorizas su prueba), se registran adicionalmente con cronómetro de alta resolución (`time.perf_counter()`):
     - Tiempo real de carga del modelo en VRAM ($T_{\text{load}}$).
     - Velocidad real de procesamiento de contexto en tokens/s ($R_{\text{prefill}}$).
     - Velocidad real de generación en tokens/s ($R_{\text{gen}}$).
     - Uso pico real de VRAM en GB.
3. **Evaluación Ciega (*Blind Review*)**:
   - Las salidas de los modelos se anonimizan como `Modelo A`, `Modelo B`, `Modelo C`... antes de la calificación humana/asistida de los criterios 1 a 4, mientras que los criterios 5, 6, 7 y 8 se miden de forma 100 % determinista por `src/nolvus_translator/quality/validator.py` (código fuente local no publicado en este repositorio).
4. **Umbrales de Aprobación para Producción**:
   - **Tasa de integridad de variables y esquema JSON (Criterios 7 y 8)**: $\ge 99,0\,\%$ en primera pasada (`100 %` tras reintento automático).
   - **Cumplimiento de Glosario y Nombres Propios (Criterios 5 y 6)**: $\ge 97,0\,\%$.
   - **Puntuación Global Ponderada**: $\ge 90,0 / 100$ para el tramo estándar (UI/Objetos/MCM/Diálogos cortos) y $\ge 94,0 / 100$ para el tramo literario (Libros de lore y diálogos complejos).
