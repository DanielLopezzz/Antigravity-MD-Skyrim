> **AUDITORÍA FASE 7 — CORTE HISTÓRICO:** Este documento conserva la auditoría complementaria del Snapshot #1 y la reconciliación realizada en la Fase 7. Sus métricas fueron ampliadas/conciliadas en el informe posterior de [Fase 9](../../Fase9_Full_Audit/PHASE9_FULL_AUDIT_REPORT.md). Úsalo para rastrear el origen de los cortes B/C, no como inventario vigente independiente.

---

# AUDITORÍA TÉCNICA COMPLEMENTARIA DE LA FASE 7 Y CONSOLIDACIÓN DE MÉTRICAS (`ES-419`)
**Proyecto**: NOLVUS AI TRANSLATOR — Nolvus Awakening 6.0.21  
**Base de Datos Auditada**: `c:/Skyrim proyecto traducción/workspace_data/nolvus_translator.db` (ruta/artefacto local no incluido en GitHub) (`Snapshot #1`)  
**Datos Empíricos de Respaldo**: `C:/Users/PC/.gemini/antigravity/brain/bbb1e814-f767-477e-92fe-3df8fb16c877/scratch/audit_phase7_results.json` (ruta/artefacto local no incluido en GitHub) y `c:/Skyrim proyecto traducción/workspace_data/output/reports/coverage_snapshot_1.md` (ruta/artefacto local no incluido en GitHub)

---

## 1. Auditoría A — Scripts `.pex` dentro de Archivos `.bsa` (`38.406` archivos)

Durante el escaneo inicial de la Fase 7 se procesaron los **5.162 archivos `.pex` sueltos** ganadores del VFS y se inventariaron **38.406 archivos `.pex` empaquetados dentro de archivos `.bsa`** como diferidos. En esta auditoría complementaria se extrajeron y analizaron empíricamente en memoria (solo lectura) los **38.406 archivos `.pex`** alojados en **253 contenedores `.bsa`** utilizando `src/nolvus_translator/formats/bsa_archive.py` (código fuente local no publicado en este repositorio) y `src/nolvus_translator/formats/pex_parser.py` (código fuente local no publicado en este repositorio) en `18,24 segundos`.

### 1.1. Respuestas Técnicas a los 5 Puntos Solicitados

| Punto de Auditoría | Resultado Empírico Exacto | Explicación Técnica |
| :--- | :---: | :--- |
| **1. ¿Por qué se excluyeron inicialmente?** | Optimización de primera pasada VFS | En listas masivas (`>500` archivos), descomprimir `38.406` scripts de `253` archivos `.bsa` sin filtrar antes `STOCK GAME` ni fragmentos `QF_`/`TIF_` añadía sobrecarga de E/S sobre miles de scripts base sin texto de interfaz. |
| **2. ¿Cuántos pertenecen a archivos base (`STOCK GAME`) o redundantes?** | **`13.041`** `.pex` (`33,96 %`) | Alojados en `Skyrim - Misc.bsa` y BSAs oficiales de Creation Club dentro de `STOCK GAME (Data)`. Sus mensajes de juego base ya están cubiertos por registros `GMST`/`MESG`/`QUST` de `Skyrim.esm` y DLCs. |
| **3. ¿Cuántos pertenecen a fragmentos automáticos (`QF_*`, `TIF_*`, `SF_*`, `PF_*`)?** | **`16.567`** `.pex` (`43,14 %`) | Scripts autogenerados por Creation Kit para etapas de misión (`QF_`), diálogos (`TIF_`), escenas (`SF_`) y paquetes de IA (`PF_`), los cuales solo ejecutan llamadas `SetStage()`, `SetObjectiveDisplayed()` o `GetOwningQuest()` sin literales visibles al jugador. |
| **4. ¿Cuántos pertenecen a mods y contienen realmente mensajes al jugador?** | **`25.365`** `.pex` en BSAs de mods, de los cuales **`3.090` scripts** contienen literales traducibles (**`14.036` ocurrencias** / **`6.172` cadenas únicas**) | De los `38.406` `.pex` en BSA (`38.405` parseados sin error, `99,997 %`), solo `3.923` poseen alguna literal de texto y **`3.090` scripts** de mods contienen llamadas reales a `Debug.Notification`, `Debug.MessageBox` o propiedades visibles. |
| **5. Método técnico para sobrescribir un `.pex` de un `.bsa` sin recompilar ni reconstruir el `.bsa`** | Inyección binaria en tabla de strings + exportación como archivo suelto (*Loose File Override*) | Ver detalle arquitectónico paso a paso en la subsección **1.3**. |

### 1.2. Principales Mods cuyos `.bsa` contienen Scripts `.pex` con Mensajes al Jugador
El análisis identificó exactamente qué contenedores `.bsa` de mods concentran las `14.036` ocurrencias traducibles (`6.172` cadenas únicas):
1. `Unofficial Skyrim Special Edition Patch.bsa`: **1.942** ocurrencias.
2. `Legacy of the Dragonborn - The Curators Companion (DBM_RelicNotifications.bsa)`: **1.228** ocurrencias.
3. `Legacy of the Dragonborn (LegacyoftheDragonborn.bsa)`: **1.191** ocurrencias.
4. `Campfire (Campfire.bsa)`: **527** ocurrencias.
5. `Lucien (Lucien.bsa)`: **517** ocurrencias.
6. `Beyond Skyrim - Bruma (BSHeartland.bsa)`: **479** ocurrencias.
7. `SPERG - Skyrim Perk Enhancements (SPERG-SSE.bsa)`: **401** ocurrencias.
8. `Interesting NPCs 3DNPC (3DNPC.bsa)`: **399** ocurrencias.
9. `A Cat's Life (ACatsLife.bsa)`: **355** ocurrencias.
10. `Project AHO (Dwarfsphere.bsa)` + `Bugfix Patch`: **696** ocurrencias.

### 1.3. Método Técnico de Sobrescritura Segura de `.pex` desde `.bsa` en MO2
El motor `BSResource` de Skyrim aplica una jerarquía inmutable: **cualquier archivo suelto (*loose file*) en `Data/Scripts/<Nombre>.pex` tiene prioridad absoluta sobre cualquier archivo `Scripts/<Nombre>.pex` empaquetado dentro de un `.bsa`**, sin importar el orden de carga del `.bsa`.

Por tanto, el flujo implementado en `src/nolvus_translator/formats/bsa_archive.py` (código fuente local no publicado en este repositorio), `src/nolvus_translator/formats/pex_parser.py` (código fuente local no publicado en este repositorio) y `src/nolvus_translator/export/mo2_packager.py` (código fuente local no publicado en este repositorio) opera en 4 pasos sin tocar el `.bsa` original ni requerir el compilador Papyrus:
1. **Extracción en memoria desde el `.bsa`**: `BsaArchiveReader.extract_file("scripts/<script>.pex")` localiza el bloque por hash/offset y descomprime el flujo LZ4 (`v105` SSE) o zlib (`v104`) en memoria RAM sin escribir nada en el mod original.
2. **Aislamiento AST de la Tabla de Strings (`.pex` `0xFA57C0DE`)**: `PexBinaryParser` recorre las instrucciones de bytecode (`CallMethod` / `CallStatic` / `Assign`) y verifica qué índices `uint16` de la tabla de cadenas son argumentos exclusivos de `Debug.Notification`, `Debug.MessageBox` o valores iniciales de propiedades de texto. Si un índice `uint16` es compartido con el nombre de un estado, función, variable, evento o `SendAnimationEvent`, queda bloqueado (`safe_indices`).
3. **Reconstrucción binaria sin recompilar**: `rebuild_pex_with_translations()` actualiza únicamente la longitud (`uint16` Big-Endian) y los bytes UTF-8 de los índices `safe_indices` en la tabla de cadenas de la cabecera del `.pex`, dejando intactos el árbol de objetos, estados, variables e instrucciones de bytecode.
4. **Empaquetado como archivo suelto en MO2**: `Mo2ModPackager` escribe el `.pex` resultante como archivo suelto en `Nolvus_AI_Spanish_ES419_Patch/Scripts/<script>.pex`. Al colocar nuestro parche al final de `modlist.txt` en MO2, el VFS de MO2 presenta este archivo suelto al ejecutable `SkyrimSE.exe`, sobrescribiendo limpiamente al `.pex` del `.bsa` sin modificar ni reconstruir el archivo `.bsa`.

---

## 2. Auditoría B — Reglas de Prioridad de Archivos `.bsa` (`583` contenedores)

Se auditaron los **583 archivos `.bsa`** activos en el perfil `Nolvus Awakening` y se actualizó `src/nolvus_translator/mo2/vfs.py` (código fuente local no publicado en este repositorio) para reflejar con exactitud el comportamiento del motor `BSResource` de Skyrim:

1. **Lista Base de `skyrim.ini` (`18` BSAs)**: Se cargan primero en el orden exacto definido por `sResourceArchiveList` y `sResourceArchiveList2` en `profiles/Nolvus Awakening/skyrim.ini` (`Skyrim - Misc.bsa` -> ... -> `Skyrim - Voices_es0.bsa` -> `Skyrim - Textures0..8.bsa` -> `Skyrim - Patch.bsa`).
2. **Orden de Carga de Plugins (`562` BSAs)**: Se ordenan según el índice `load_order_index` de su plugin asociado (`.esm` -> `.esl` -> `.esp`) en `loadorder.txt` / `plugins.txt`, emparejando tanto `<PluginStem>.bsa` como `<PluginStem> - Textures.bsa`.
3. **BSAs sin Plugin Activo Directo (`3` BSAs)**: Se identificaron exactamente 3 BSAs sin plugin homónimo activo (`Skyrim - Voices_en0.bsa` en `STOCK GAME`, `Midwood Isle - Cats0.bsa` y `Dangerous NPCs MI Patch.bsa`), para los cuales el escáner aplica `vfs_priority` de `modlist.txt` como criterio de desempate.
4. **Colisiones Internas BSA-contra-BSA (`2.595` archivos)** y **Prioridad de Archivos Sueltos**: Se verificaron `2.595` recursos textuales/interfaz que colisionan entre dos o más BSAs (resueltos a favor del BSA de mayor índice de carga) y se confirmó que los **18.132 archivos sueltos** ganan siempre sobre los **57.166 archivos en BSA**.

---

## 3. Auditoría C — Archivos Pendientes de Clasificación Fina (`SWF`, `JSON`, `PSC`, `XML/TXT`)

| Formato | Total Inventariado | Desglose Empírico en `Snapshot #1` | Decisión Técnica Definitiva |
| :--- | :---: | :--- | :--- |
| **`.swf` (`SWF_UNSUPPORTED`)** | **`678`** (`617` sueltos + `61` en BSA) | `578` comprimidos con zlib (`CWS`), `38` sin comprimir (`FWS`) y `1` Scaleform (`CFX`). Concentrados en `Untarnished UI` (`223`), `Edge UI` (`200`), `Dear Diary` (`69`), `The Handy Icon Collection` (`27`) y `STOCK GAME` (`23`). | La inmensa mayoría son hojas de iconos vectoriales y widgets HUD cuyas etiquetas provienen de `translate_spanish.txt`, `$variables` MCM o `GMST`. Se mantienen como `unsupported` para edición binaria directa y se penalizan honestamente en el cálculo de cobertura real. |
| **`.json` (`MCM_JSON` vs `JSON_OTHER`)** | **`1.932`** (`42` `MCM_JSON` + `1.890` `JSON_OTHER`) | Los **`42` archivos `mcm/config/*/config.json`** (MCM Helper) ya fueron extraídos (`3.786` cadenas). De los `1.890` `JSON_OTHER`: `1.600` son coordenadas/layouts geométricos de widgets SkyUI/Interface, `50` son preajustes de posición de `Immersive Equipment Displays (IED)`, `41` son mapeos técnicos de `DBVO`, `56` son configs técnicas SKSE, y solo `~33` (`Custom Skills` / `Inventory Injector`) contienen etiquetas menores opcionales. | Se procesan los `42` `MCM_JSON` y se excluyen los `1.890` `JSON_OTHER` de geometría/configuración SKSE para evitar corromper claves estructurales de plugins C++. |
| **`.psc` (`PSC`)** | **`20.448`** | **`20.247` (`99,02 %`)** tienen su correspondiente `.pex` compilado ya presente en el VFS; solo `201` son fuentes huérfanos sin `.pex` activo (scripts de desarrollo descartados por los autores). | Skyrim ejecuta exclusivamente `.pex`, nunca `.psc`. Los `.psc` se conservan en `scanned_files` como referencia contextual de código fuente sin duplicar cadenas en `string_entries`. |
| **`.xml` y `.txt` (`MCM_XML`, `XML_OTHER`, `OTHER`)** | **`114` XML** + **`743` TXT** | Los `114` XML extraen `4.864` cadenas válidas (`DialogueViews` y configuraciones). Los `743` `.txt` fuera de `Interface/Translations/` son `302` *readmes/changelogs* en raíz, `260` logs/listas técnicas SKSE, `76` ajustes internos MCM y `73` listas de estaciones (`Seasons of Skyrim`). | Ninguno de los `743` `.txt` auxiliares contiene texto jugable; se mantienen excluidos de la traducción. |

---

## 4. Auditoría D y Sección 7 — Consolidación de Métricas y Discrepancias Internas Corregidas

### 4.1. Hallazgos Críticos de la Auditoría sobre `Snapshot #1`
La inspección cruzada de las `458.923` filas originales de `Snapshot #1` reveló tres causas exactas que explicaban tanto la presencia de traducciones comunitarias como la diferencia entre cadenas únicas traducibles y cadenas únicas pendientes:

1. **Separación Exacta entre Localización Oficial de Bethesda y Traducciones Comunitarias (`67.084` ocurrencias originales)**:
   - **`57.261` ocurrencias (`48.029` cadenas únicas)** provienen de `STOCK GAME (Data)`: `Skyrim.esm` (`37.767`), `Dragonborn.esm` (`6.546`), `Dawnguard.esm` (`5.314`), `HearthFires.esm` (`1.429`), `Update.esm` (`488`) y los **74 plugins oficiales de Creation Club** (`cc*.esm` y `cc*.esl`, `5.717`). Estas constituyen la **Localización Oficial de Bethesda preexistente**.
   - **`9.823` ocurrencias** provenían de mods de terceros: `7.186` en archivos MCM `*_english.txt` que habían sido emparejados automáticamente con archivos comunitarios `*_spanish.txt` (incluidos en `69` mods), `1.928` en archivos MCM de otros idiomas con tildes (`_czech.txt`, `_french.txt`, `_polish.txt`), y `709` en plugins de terceros (`hearthfiremultikid.esp` con `394`, `unofficial skyrim creation club content patch.esl` con `255`, `dismembering framework.esm` con `14`, y `46` por detección heurística de palabras cortas como *"la"* o *"una"*).
2. **Depuración de Archivos MCM en Idiomas Extranjeros (`55.001` ocurrencias eliminadas)**:
   - De los `724` archivos `Interface/Translations/*.txt` del VFS, **`97` son los archivos fuente en inglés (`*_english.txt`)**, `69` son `*_spanish.txt` comunitarios, y **`558` son archivos en otros idiomas** (`_chinese.txt`, `_czech.txt`, `_french.txt`, `_german.txt`, `_italian.txt`, `_japanese.txt`, `_polish.txt`, `_russian.txt`, `_turkish.txt`, etc.).
   - El 100 % de los mods con menú MCM en Nolvus posee su archivo fuente `*_english.txt` (`8.910` cadenas en total). Los `558` archivos en otros idiomas añadían **55.001 ocurrencias espurias** (en chino, ruso, checo, alemán, francés, etc.) que han sido eliminadas de `Snapshot #1` y bloqueadas en `src/nolvus_translator/services/project_service.py` (código fuente local no publicado en este repositorio).
3. **Explicación de la Fórmula de Deduplicación (`242.243` vs `231.210` en el informe inicial)**:
   - En el informe inicial de Fase 7, `CoverageReporter` calculaba `unique_pending_net = len(unique_translatable_hashes - unique_covered_hashes)`, donde `unique_covered_hashes` incluía las cadenas de Bethesda (`ALREADY_TRANSLATED`) y los nombres propios de `NPC_:FULL` (`PROTECTED_PROPER_NOUN`). Como `11.033` cadenas en inglés aparecían tanto en registros de Bethesda/MCM como en mods de terceros, se descontaban silenciosamente aunque `translated_approved` fuera `0`.
   - Bajo la nueva política (`Sección 3` y `Sección 5`), la localización oficial de Bethesda **no se mezcla automáticamente en la Memoria de Traducción ES-419**, por lo que la fórmula matemática exacta e inequívoca es:
     $$\text{Cadenas Únicas Pendientes Netas} = \text{Cadenas Únicas Traducibles de Mods} - \text{Cadenas Únicas Aprobadas ES-419}$$

### 4.2. Definición Exacta de las Categorías y Cifras Auditadas en `Snapshot #1`

| Categoría / Estado | Ocurrencias (Antes) | Ocurrencias (Auditadas ES-419) | Cadenas Únicas (`source_hash`) | Significado Exacto bajo Política ES-419 |
| :--- | ---: | ---: | ---: | :--- |
| **`already_translated` (Solo Bethesda Oficial)** | 67.084 *(mezclado)* | **57.261** | **48.029** | Exclusivamente localización oficial preexistente de Bethesda (`Skyrim.esm`, DLCs y `74` plugins Creation Club en `STOCK GAME`). **0 % traducciones comunitarias reutilizadas**. |
| **`translatable` (Traducibles de Mods)** | 358.498 | **313.731** | **235.091** | Texto en inglés original de mods apto para traducción directa a `es-419` (incluye las `7.895` ocurrencias comunitarias reclasificadas a traducibles desde su inglés original y excluye las `52.662` filas MCM en idiomas extranjeros). |
| **`pending_research` (Requieren Investigación)** | 718 | **718** | **352** | Topónimos o términos cortos capitalizados en registros `LCTN`, `CELL`, `WRLD`, `FACT` o `BOOK` no presentes aún en el Glosario Maestro que requieren verificación de lore antes de decidir si se traducen o conservan. |
| **Subtotal Acción de Traducción (`translatable + pending_research`)** | **359.216** | **314.449** | **235.326** | Total de ocurrencias y cadenas únicas de mods que requieren procesamiento hacia `es-419` (nota: `117` hashes coinciden entre `translatable` y `pending_research`, por lo que `235.091 + 352 - 117 = 235.326`). |
| **`protected_proper_noun` (Protegidas)** | 8.795 | **8.667** | **4.877** | Nombres propios de personajes (`NPC_:FULL`) y entidades protegidas por el Glosario Maestro con regla `keep_original = True`. |
| **`pure_variable` (Variables Puras)** | 9.647 | **9.647** | **4.402** | Cadenas compuestas únicamente por marcadores dinámicos (`<Alias=...>`, `%s`, `%.0f`) y símbolos sin texto humano traducible. |
| **`identifier` (Identificadores)** | 6.788 | **6.535** | **4.327** | Identificadores internos en `camelCase` o `snake_case` (eventos SKSE, claves de animación, variables de script). |
| **`technical_text` (Técnicas)** | 6.187 | **6.187** | **2.845** | Rutas internas de archivos (`.nif`, `.dds`, `.hkx`) y referencias a nodos Scaleform (`_root.WidgetContainer`). |
| **`non_translatable` (No Traducibles)** | 1.206 | **1.176** | **165** | Expresiones puramente numéricas, signos de puntuación aislados o cadenas vacías. |
| **TOTAL CADENAS EN `SNAPSHOT #1`** | **458.923** | **403.922** | **292.192** | `458.923 - 55.001 (MCM extranjeros depurados) = 403.922` (`scan_snapshots.total_strings == COUNT(*) FROM string_entries`). |

---

## 5. Reconciliación de los Tres Cortes de Volumen Pendiente

Para ofrecer trazabilidad absoluta entre el informe inicial de Fase 7 y la evaluación económica de Fase 8, se presentan los tres cortes exactos:

| Métrica | Corte A (Sobre las `458.923` filas originales reclasificando comunitarias) | **Corte B (Inventario Oficial Depurado en `Snapshot #1` — Base de Referencia)** | Corte C (Inventario Depurado + `.pex` de Mods dentro de `.bsa`) |
| :--- | ---: | ---: | ---: |
| **Ocurrencias totales en inventario** | 458.923 | **403.922** | 428.006 |
| **Localización oficial de Bethesda conservada** | 57.261 | **57.261** | 57.261 |
| **Traducciones comunitarias reutilizadas** | 0 | **0** | 0 |
| **Ocurrencias traducibles de mods (`translatable + pending_research`)** | 369.039 | **314.449** | 328.485 |
| **Ocurrencias duplicadas ahorradas por Memoria de Traducción** | 123.731 | **79.123** | 88.241 |
| **Cadenas únicas netas pendientes de traducir (`unique_pending_net`)** | 245.308 | **235.326** | **240.244** |
| **Caracteres únicos pendientes en inglés** | 20.348.900 | **20.062.309** | **20.314.890** |
| **Lotes estimados (a 25 cadenas/lote)** | 9.813 | **9.414** | 9.610 |
| **Lotes estimados (a 40 cadenas/lote)** | 6.133 | **5.884** | 6.007 |
| **Tokens de entrada estimados (Base, 25/lote)** | 13,51 M | **13,09 M** | 13,35 M |
| **Tokens de salida estimados (Base)** | 9,70 M | **9,52 M** | 9,68 M |
| **Tokens de entrada con margen de revisión (+15 %)** | 15,54 M | **15,06 M** | 15,35 M |
| **Tokens de salida con margen de revisión (+15 %)** | 11,15 M | **10,95 M** | 11,13 M |

> [!NOTE]
> El **Corte B (`235.326` cadenas únicas / `20.062.309` caracteres)** es el estado activo persistido en `c:/Skyrim proyecto traducción/workspace_data/nolvus_translator.db` (ruta/artefacto local no incluido en GitHub). Si se decide incluir también los mensajes de scripts `.pex` empaquetados dentro de los `.bsa` de mods (**Corte C**), el volumen neto aumenta únicamente en **`+4.918` cadenas únicas nuevas** (`+252.581` caracteres, un `+1,26 %` en caracteres) gracias a la deduplicación contra las cadenas ya presentes en los plugins `.esp`.
