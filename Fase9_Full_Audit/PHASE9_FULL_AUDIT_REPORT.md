# NOLVUS AI TRANSLATOR — INFORME TÉCNICO FINAL DE LA FASE 9
## AUDITORÍA INTEGRAL DE MODS, COBERTURA Y EXTRACCIÓN EXHAUSTIVA DE TEXTOS

- **Fecha de ejecución UTC:** `2026-11-03T07:14:01+00:00`
- **Modo de operación:** Auditoría 100 % de solo lectura sobre las fuentes originales (`C:\Nolvus`, `STOCK GAME`, `MO2`, `MODS`) y sobre la base de datos de producción (`workspace_data/nolvus_translator.db`, abierta mediante URI `mode=ro`).
- **Directorio independiente de auditoría Fase 9:** `c:\Skyrim proyecto traducción\NolvusAIT_Audit\Fase9_20261103_020500\`
- **Base de datos SQLite independiente de auditoría:** `c:\Skyrim proyecto traducción\NolvusAIT_Audit\Fase9_20261103_020500\phase9_audit.db`
- **Traducciones ejecutadas / Llamadas a APIs / Créditos consumidos:** `0` (Cumplimiento estricto de las restricciones de la Fase 9).

---

## 1. RESUMEN EJECUTIVO

La **Fase 9** ha completado la auditoría integral, exhaustiva y verificable del 100 % de los mods, plugins, archivos comprimidos `.bsa`, archivos sueltos y formatos parcialmente soportados o no soportados de la instalación real de **Nolvus Awakening 6.0.21**, cerrando todas las incertidumbres estadísticas entre las Fases 7 y 8 sin modificar un solo byte de la instalación original ni de la base de datos principal del proyecto.

### Hallazgos principales en cifras verificadas:
1. **Entorno e integridad de archivos:**
   - **Instancia:** `Nolvus Awakening` v`6.0.21` (variante `Ultimate`, interfaz `Untarnished UI`, código de idioma `ES` / `sLanguage=SPANISH`), verificada en `C:\Nolvus\Instances\InstancesData.xml`.
   - **Binarios reales verificados por cabecera PE y SHA-256:** `SkyrimSE.exe` v`1.5.97.0`, `ModOrganizer.exe` v`2.4.4.0`, `NolvusDashBoard.exe` v`3.8.14.0`.
2. **Inventario completo de mods (`modlist.txt` vs carpetas físicas):**
   - `3.847` líneas válidas en `modlist.txt`: **`3.700` mods activos** (`+`), **`1` mod desactivado** (`-Unsearched Corpse Indicator`), y **`146` separadores** (`69` carpetas `_separator` + `77` separadores virtuales `*Creation Club:` cuyos activos residen en `STOCK GAME\Data`).
   - Existen exactamente **`3.770` carpetas físicas** en `MODS\mods` (`3.700` activos + `1` desactivado + `69` `_separator`), con **`0` mods faltantes** y **`0` carpetas huérfanas**.
3. **Inventario completo de plugins (`loadorder.txt` y `plugins.txt`) y archivos `.bsa` (`archives.txt`):**
   - **`3.717` plugins activos** (`0` inactivos en `loadorder.txt`): **`39` `.esm`**, **`135` `.esl`**, **`195` `.esp` estándar** y **`3.348` `ESPFE`** (`.esp` con flag ligero `0x200` en cabecera `TES4`). **`84` plugins** poseen el flag `LOCALIZED` (`0x80`).
   - **`583` archivos `.bsa` ganadores** en el VFS (`100 %` versión `v105` LZ4, **`0` errores de lectura**, `73,53 GB` en disco / `74,21 GB` descomprimidos), que contienen **`563.522` miembros internos** (`406.566` ganadores de BSA, `79.743` sobrescritos por archivos sueltos y `77.213` colisiones BSA→BSA).
4. **Inventario universal del 100 % de archivos descubiertos:**
   - **`1.168.337` elementos totales auditados**: `604.811` archivos sueltos en capas activas (`551.034` ganadores VFS + `53.777` sobrescritos, `451,11 GB`), `4` archivos en el único mod desactivado (`398,5 KB`), y `563.522` archivos empaquetados dentro de los `583` `.bsa`.
5. **Auditoría profunda de formatos pendientes (`.pex` en `.bsa`, `.swf`, `json_other`, `.psc`, `.dll`):**
   - **`.pex` dentro de `.bsa` (`38.406` archivos):** `25.365` pertenecen a mods y `3.090` contienen texto visible para el jugador, aportando **`14.036` ocurrencias traducibles** (`6.172` cadenas únicas dentro de BSA, de las cuales `1.254` ya existían en el Corte B y **`+4.918` son cadenas únicas nuevas**, pasando de `235.326` a **`240.244`**).
   - **`.swf` (`678` archivos):** Descomprimidos e inspeccionados al `100 %` en memoria (`0` errores). **`536` (`79,06 %`)** son puros gráficos, iconos, fuentes o cursores sin texto; **`94`** utilizan **`1.878` claves dinámicas `$Variable`** que el motor ya traduce a través de `Interface/Translations/*_english.txt`; **`96`** tienen etiquetas `DefineEditText` (`523` cadenas, mayoritariamente texto de muestra de diseño como `"Reference Text"` o EULA de Bethesda.net); y **`121`** tienen literales en el `ConstantPool` de ActionScript (`945` cadenas, en su mayoría mensajes internos de depuración de Scaleform).
   - **`json_other` (`1.890` archivos):** Auditados recursivamente. **`1.332` (`70,48 %`)** son archivos técnicos de configuración/animación/mallas sin texto humano, **`11`** tienen sintaxis extendida con comentarios, y **`547` (`28,94 %`)** contienen **`2.047` ocurrencias (`1.178` cadenas únicas)** en mods como *The Dragonborn's Bestiary* (`796` ocurrencias narrativas de códice), *Immersive Equipment Displays* (`732` nombres de presets), *Wheeler* (`209`), *Immersive Hunting Animations* (`117`), *Alternate Perspective* (`78` descripciones de inicios alternativos) y *Custom Skill Menu* (`18`).

---

## 2. ESTADO REAL DEL ENTORNO DE NOLVUS (TAREA 1)

Todos los archivos de configuración, ejecutables y manifiestos de perfil fueron verificados en modo de solo lectura. No se asumió ningún valor histórico.

### 2.1. Parámetros verificados de la instancia y del runtime
| Parámetro | Valor Real Verificado | Fuente de Evidencia |
|---|---|---|
| **Nombre de la instancia** | `Nolvus Awakening` | `C:\Nolvus\Instances\InstancesData.xml` (`<Name>`) |
| **Versión de la modlist** | `6.0.21` (Variante `Ultimate`, UI `Untarnished UI`) | `C:\Nolvus\Instances\InstancesData.xml` (`<Version>`) |
| **Versión binaria de `NolvusDashBoard.exe`** | `3.8.14.0` | Cabecera PE `VS_FIXEDFILEINFO` (`C:\Nolvus\NolvusDashBoard.exe`) |
| **Versión binaria de `SkyrimSE.exe` (STOCK GAME)** | `1.5.97.0` | Cabecera PE `VS_FIXEDFILEINFO` (`STOCK GAME\SkyrimSE.exe`) |
| **Versión binaria de `ModOrganizer.exe`** | `2.4.4.0` | Cabecera PE `VS_FIXEDFILEINFO` (`MO2\ModOrganizer.exe`) |
| **Perfil activo en MO2** | `Nolvus Awakening` | `MO2\ModOrganizer.ini` (`selected_profile=@ByteArray(Nolvus Awakening)`) |
| **Idioma configurado en perfil e instancia** | `SPANISH` / `ES (Spanish)` | `Skyrim.ini` (`sLanguage=SPANISH`) y `InstancesData.xml` (`<LgCode>ES</LgCode>`) |
| **Política de localización objetivo** | Español Latinoamericano Neutro (`ES-419`) desde inglés | `TRANSLATION_POLICY_ES419.md` (`0 %` reutilización comunitaria) |

### 2.2. Hashes SHA-256 de referencia de los manifiestos y ejecutables
El registro íntegro se encuentra en `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/manifest_hashes.json` (artefacto local de auditoría; no incluido en GitHub):

| Archivo | Ruta Física | Tamaño (Bytes) | Hash SHA-256 |
|---|---|---:|---|
| `InstancesData.xml` | `C:\Nolvus\Instances\InstancesData.xml` | `2.577` | `aeb3eac5fb1e776af48574936a94ff318d34695329c0361b905514036791f730` |
| `NolvusDashBoard.exe` | `C:\Nolvus\NolvusDashBoard.exe` | `59.674.624` | `a7d0735aaf2a6044e9e1e09a6e70f28e20eb0398e080d05c5134959e6da27899` |
| `SkyrimSE.exe` | `C:\Nolvus\Instances\Nolvus Awakening\STOCK GAME\SkyrimSE.exe` | `34.769.792` | `693e5a51ea2680119a68620bcf5080e81745549872b5d06bbd3f51131b67abab` |
| `ModOrganizer.exe` | `C:\Nolvus\Instances\Nolvus Awakening\MO2\ModOrganizer.exe` | `4.704.256` | `19414ad0a3f13466dd99fcf242e45974044deda031088839d23fb1a1136a5a0a` |
| `ModOrganizer.ini` | `C:\Nolvus\Instances\Nolvus Awakening\MO2\ModOrganizer.ini` | `14.085` | `fd76815596153febc91058e7c8ccfd9c3f17fc40100d888cd470484fe6afa69f` |
| `modlist.txt` | `...\MODS\profiles\Nolvus Awakening\modlist.txt` | `129.178` | `fc2b8794cd1867926c235b9fcbea191d4ae5b15892acb8ce2addea44037cf697` |
| `plugins.txt` | `...\MODS\profiles\Nolvus Awakening\plugins.txt` | `127.985` | `d632e5649f848c5b8cf3b4d0099623fd8a46ad5c419c49ad42ba8b1e42eae51e` |
| `loadorder.txt` | `...\MODS\profiles\Nolvus Awakening\loadorder.txt` | `126.437` | `329c4a08d0e1f0c5c2a3a9b1ec50b9f7cd6f8d383188cb090c51eba992b9d2c2` |
| `lockedorder.txt` | `...\MODS\profiles\Nolvus Awakening\lockedorder.txt` | `59` | `9962fbb07e4c40c4c39b6fe7cedf3f1527cdda5ad0a60d7d53a872b5ae52812d` |
| `archives.txt` | `...\MODS\profiles\Nolvus Awakening\archives.txt` | `16.840` | `45b87bc086e4dc785400a25f828bf5596a822438ba3886798909be759e73b62f` |
| `Skyrim.ini` | `...\MODS\profiles\Nolvus Awakening\Skyrim.ini` | `3.549` | `a2a148fbdedc6016f8f37737b6f8dc693b2e79ac881b67846f570353be59555a` |
| `SkyrimPrefs.ini` | `...\MODS\profiles\Nolvus Awakening\SkyrimPrefs.ini` | `3.907` | `199b0f7315826981427a747cf940744e06b09b8b38ef3c517103b52659cf360b` |
| `skyrimcustom.ini` | `...\MODS\profiles\Nolvus Awakening\skyrimcustom.ini` | `63` | `8366576f9fd230224a93fe943f4bdd2e0ec7759f65434dc3e83493d0e06a291e` |

---

## 3. INVENTARIO TOTAL DE MODS (ACTIVOS, DESACTIVADOS Y SEPARADORES)

El manifiesto completo de los `3.847` registros de `modlist.txt` y su cruce contra las carpetas físicas de `C:\Nolvus\Instances\Nolvus Awakening\MODS\mods` se ha guardado en `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/manifest_mods.json` (artefacto local de auditoría; no incluido en GitHub) y en la tabla `audit_mods` de `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/phase9_audit.db` (artefacto local de auditoría; no incluido en GitHub).

| Categoría de Mod / Entrada | Cantidad | Con Carpeta Física en `MODS\mods` | Observaciones Técnicas |
|---|---:|---:|---|
| **Mods activos (`+`)** | `3.700` | `3.700` (`100 %`) | `0` carpetas faltantes. Prioridades VFS de `30` a `3.846`. |
| **Mods desactivados (`-`) en `modlist.txt`** | `1` | `1` (`100 %`) | `Unsearched Corpse Indicator` (`vfs_priority = 305`, contiene `4` archivos sueltos, `398.459` bytes, `0` plugins y `0` BSAs). |
| **Separadores organizativos (`_separator`)** | `69` | `69` (`100 %`) | Carpetas vacías de organización visual en MO2 (solo contienen `meta.ini`). |
| **Entradas virtuales `*Creation Club:`** | `77` | `0` (Residen en `STOCK GAME\Data`) | Entradas automáticas de MO2 para los paquetes oficiales del Creation Club alojados en `STOCK GAME\Data`. |
| **Carpetas físicas huérfanas no listadas en `modlist.txt`** | `0` | `0` | Concordancia del `100 %` entre `modlist.txt` y el directorio físico `MODS\mods` (`3.700 + 1 + 69 = 3.770` carpetas). |
| **Total de entradas en `modlist.txt`** | **`3.847`** | **`3.770` carpetas en `MODS\mods`** | Incluye además la capa base `STOCK GAME (Data)` y `MO2 Overwrite`. |

---

## 4. INVENTARIO TOTAL DE PLUGINS (`.ESM`, `.ESL`, `.ESP` Y `ESPFE`)

El inventario completo de los `3.717` plugins con sus flags binarios de cabecera `TES4` (`0x0001` ESM, `0x0080` Localized, `0x0200` ESL/Light) se encuentra en `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/manifest_plugins.json` (artefacto local de auditoría; no incluido en GitHub) y en la tabla `audit_plugins` de `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/phase9_audit.db` (artefacto local de auditoría; no incluido en GitHub).

| Tipo de Plugin | Activos en `loadorder.txt` | Inactivos en `loadorder.txt` | Con Flag `LOCALIZED` (`0x80`) | Explicación Técnica |
|---|---:|---:|---:|---|
| **`.esm` (Elder Scrolls Master)** | `39` | `0` | `20` | Incluye los 5 maestros base de Bethesda, maestros de Creation Club y grandes expansiones (*Bruma*, *Wyrmstooth*, *Vigilant*, *LotD*, *Lux*, etc.). |
| **`.esl` (Elder Scrolls Light Master)** | `135` | `0` | `59` | Incluye los paquetes ligeros de Creation Club y maestros ligeros de mods. |
| **`.esp` Estándar (Slot completo `00`–`FD`)** | `195` | `0` | `5` | Plugins `.esp` tradicionales sin flag `0x200`. |
| **`ESPFE` (`.esp` con flag ligero `0x200` en cabecera `TES4`)** | `3.348` | `0` | `0` | Cargados en el espacio `FE:xxx` para no agotar el límite de 254 slots completos. |
| **Plugins sueltos no listados o en mods desactivados** | `0` | `0` | `0` | No existen plugins ocultos ni fuera de `loadorder.txt`. |
| **Total de Plugins** | **`3.717`** | **`0`** | **`84`** | **`1.893` plugins (`50,93 %`)** contienen cadenas de texto extraíbles; los otros **`1.824` (`49,07 %`)** contienen exclusivamente registros numéricos, navmeshes, iluminación, clima, colisiones o colocación de referencias (`REFR`/`ACHR`/`NAVM`) sin subregistros de texto humano. |

---

## 5. INVENTARIO TOTAL DE ARCHIVOS COMPRIMIDOS `.BSA`

Se auditaron los **`583` archivos `.bsa`** presentes en el VFS activo (registrados en `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/manifest_bsa.json` (artefacto local de auditoría; no incluido en GitHub) y en la tabla `audit_bsa` de `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/phase9_audit.db` (artefacto local de auditoría; no incluido en GitHub)):

| Métrica de Auditoría `.BSA` | Valor Exacto | Detalle de Verificación |
|---|---:|---|
| **Total de archivos `.bsa` ganadores en el VFS activo** | `583` | `586` ocurrencias totales en capas activas (`3` archivos `.bsa` base son sobrescritos por versiones actualizadas de mayor prioridad). |
| **Listados explícitamente en `archives.txt` del perfil** | `582` | El archivo restante corresponde a un `.bsa` cargado por asociación directa de nombre con su plugin activo. |
| **Emparejados con un plugin activo (`<Plugin>.bsa` / `<Plugin> - Textures.bsa`)** | `562` | Los `21` restantes son paquetes adicionales de voces/música/animaciones declarados en `archives.txt`. |
| **Versión de formato `.bsa` (`v104` zlib vs `v105` LZ4)** | `583` (`100 %` `v105` LZ4) | `0` archivos en versión `v104`. |
| **Errores de apertura o lectura de índice `.bsa`** | **`0`** | El `100 %` de los `583` archivos `.bsa` se abrió e indexó sin errores. |
| **Tamaño físico comprimido en disco** | `73.525.889.719` bytes (`73,53 GB`) | Suma exacta de los `583` contenedores `.bsa`. |
| **Tamaño descomprimido reportado por cabeceras de miembros** | `74.209.006.396` bytes (`74,21 GB`) | Suma de `raw_size` de los `563.522` miembros internos. |
| **Total de archivos miembros empaquetados dentro de los `583` `.bsa`** | **`563.522`** | Todos los miembros fueron inventariados por extensión y ruta virtual. |
| **Miembros de `.bsa` ganadores (no sobrescritos por sueltos ni por otro `.bsa`)** | **`406.566`** | Activos efectivos servidos desde `.bsa` cuando el juego los solicita. |
| **Miembros de `.bsa` sobrescritos por un archivo suelto en el VFS** | **`79.743`** | Según la regla de prioridad de Skyrim/MO2, el archivo suelto (`Loose File`) siempre gana sobre cualquier `.bsa`. |
| **Colisiones entre archivos `.bsa` (`BSA → BSA`)** | **`77.213`** | Resueltas siguiendo el orden de `archives.txt` y `loadorder.txt`. |
| **Archivos `.bsa` en mods desactivados** | `0` | El único mod desactivado no contiene archivos `.bsa`. |

---

## 6. INVENTARIO UNIVERSAL DEL 100 % DE ARCHIVOS POR EXTENSIÓN (TAREA 2)

El recorrido exhaustivo de todas las carpetas de `STOCK GAME\Data`, los `3.700` mods activos, el mod desactivado y los `583` archivos `.bsa` arrojó un total de **`1.168.337` elementos descubiertos**:

| Capa / Contenedor | Ocurrencias Totales | Ganadores VFS / BSA | Sobrescritos | Tamaño Total (Bytes) |
|---|---:|---:|---:|---:|
| **Archivos sueltos en capas activas (`STOCK GAME` + `3.700` mods)** | `604.811` | `551.034` | `53.777` | `451.110.226.982` (`451,11 GB`) |
| **Archivos sueltos en mods desactivados (`1` mod)** | `4` | `0` (Inactivo) | `0` | `398.459` (`398,5 KB`) |
| **Miembros internos dentro de los `583` archivos `.bsa` activos** | `563.522` | `406.566` | `156.956` (`79.743` por sueltos + `77.213` por otro BSA) | `74.209.006.396` (`74,21 GB`) |
| **TOTAL UNIVERSAL AUDITADO** | **`1.168.337`** | **`957.600` ganadores netos** | **`210.733` sobrescritos** | **`525.319.631.837` (`525,32 GB`)** |

### 6.1. Desglose de las principales extensiones de archivos (Sueltos Activos vs Miembros en `.BSA`)

| Extensión | Naturaleza Técnica | Archivos Sueltos (Capas Activas) | Miembros dentro de `.BSA` | Total Combinado | Tratamiento en NOLVUS AI TRANSLATOR |
|---|---|---:|---:|---:|---|
| `.fuz` | Audio de voz labializado (`XWM` + `LIP`) | `232.934` | `277.427` | `510.361` | Activo binario de audio de doblaje (no textual; los subtítulos viven en `INFO:NAM1` de los plugins). |
| `.dds` | Texturas DirectDraw Surface | `114.822` | `85.894` | `200.716` | Activo gráfico; `7.089` en rutas de UI/mapas/carteles auditadas en la Sección 23. |
| `.nif` | Mallas 3D NetImmerse | `78.588` | `69.331` | `147.919` | Geometría 3D binaria (no contiene texto de localización). |
| `.cgid` / `.gid` | Caché de hierba pregenerada (Grass Cache) | `64.591` | `7.545` | `72.136` | Datos binarios de posicionamiento de vegetación. |
| `.pex` | Scripts compilados Papyrus | `5.294` (`5.162` ganadores) | `41.719` (`38.406` ganadores) | `47.013` | **Auditado al 100 %** (tanto los `5.162` sueltos como los `38.406` ganadores en `.bsa`). |
| `.btr` / `.bto` / `.btt` | Terreno y objetos LOD (DynDOLOD / xLODGen) | `17.834` | `21.978` | `39.812` | Geometría y texturas de nivel de detalle lejano (binario). |
| `.hkx` / `.hkt` | Animaciones y esqueletos Havok | `18.814` | `7.904` | `26.718` | Datos binarios de animación y comportamiento Nemesis/Havok. |
| `.txt` | Traducciones MCM (`Interface/Translations`), mallas/configs | `19.728` | `1.481` | `21.209` | **Auditado**: `724` archivos MCM (`588` sueltos + `136` en BSA) en `Interface/Translations/`. El resto son listas técnicas de Bodyslide/Nemesis/logs. |
| `.psc` | Código fuente Papyrus | `4.284` (`4.188` ganadores) | `16.601` (`16.260` ganadores) | `20.885` | **Auditado al 100 %** en la Sección 19 (`20.247` tienen `.pex` compilado; `201` son huérfanos sin `.pex`). |
| `.wav` / `.xwm` | Efectos de sonido y música | `10.842` | `18.798` | `29.640` | Audio binario sin texto. |
| `.tri` / `.osd` / `.osp` | Morfologías faciales y proyectos BodySlide | `15.495` | `2.935` | `18.430` | Datos de vértices 3D y sliders de armaduras. |
| `.lip` | Sincronización labial suelta | `2.594` | `8.647` | `11.241` | Datos fonéticos binarios. |
| `.json` | MCM Helper (`42`) + Configs/Presets (`1.890` en alcance VFS + perfiles) | `4.780` | `1` | `4.781` | **Auditado**: `42` `mcm_json` extraídos + `1.890` `json_other` inspeccionados recursivamente en la Sección 21. |
| `.esp` / `.esm` / `.esl` | Plugins de Bethesda y de mods | `4.000` (`3.717` activos en raíz `Data`) | `1` | `4.001` | **Extraído al 100 %** para los `3.717` plugins activos. |
| `.strings` / `.dlstrings` / `.ilstrings` | Tablas de localización de plugins con flag `0x80` | `21` (`7` sets sueltos) | `2.333` (`752` ganadores por tipo = `2.256`) | `2.354` | **Extraído al 100 %** en memoria desde archivos sueltos y `.bsa`. |
| `.ini` | Configuraciones SKSE, ENB, Dynamic String Distributor (`210` loc) | `1.060` | `50` | `1.110` | **Auditado**: `210` archivos de localización INI extraídos; el resto son parámetros numéricos de SKSE/ENB. |
| `.swf` | Archivos de interfaz Scaleform / Flash | `884` (`617` ganadores) | `170` (`61` ganadores) | `1.054` | **Auditado al 100 %**: los `678` `.swf` ganadores fueron descomprimidos e inspeccionados en la Sección 20. |
| `.xml` | Configuraciones de barras/widgets (`1` MCM XML + `113` UI/física) | `768` | `45` | `813` | **Auditado**: `1` `mcm_xml` + `113` `xml_other` (HDT-SMP física y layouts de SkyHUD/TrueHUD). |
| `.dll` | Complementos nativos C++ de SKSE (`SKSE/Plugins/*.dll`) | `215` (`211` ganadores en `SKSE/Plugins`) | `0` | `215` | **Auditado al 100 %** en la Sección 22. |

---

## 7. COBERTURA DE EXTRACCIÓN ALCANZADA (TAREAS 3 Y 5)

Sobre los **`75.298` archivos textuales, scripts y contenedores candidatos ganadores** del VFS (`Snapshot #1`), más la inspección directa de los formatos marcados previamente como pendientes o no soportados, la cobertura técnica alcanzada en la Fase 9 es del **100 % de visibilidad y diagnóstico verificable**:

| Formato / Fuente | Archivos Ganadores en VFS (Sueltos + BSA) | Estado en Escáner Base (Fase 7) | Estado tras Auditoría Profunda (Fase 9) | Cobertura de Auditoría |
|---|---:|---|---|---:|
| **Plugins (`.esm`, `.esl`, `.esp`, `ESPFE`)** | `3.717` activos (`3.801` en VFS) | Soportado y extraído (`353.651` ocurrencias) | Verificado al `100 %` (`1.893` con texto, `1.824` puros registros sin texto) | **100,00 %** |
| **Tablas `*STRINGS` (`.strings`, `.dlstrings`, `.ilstrings`)** | `2.277` (`21` sueltos + `2.256` en BSA) | Soportado y extraído en memoria (integrado con los `84` plugins `LOCALIZED`) | Verificado al `100 %` (`0` tablas huérfanas sin resolver) | **100,00 %** |
| **Archivos MCM (`Interface/Translations/*.txt`)** | `724` en inglés/español (`+558` en otros idiomas depurados) | Soportado y extraído (`8.910` ocurrencias de mods en inglés) | Verificado al `100 %` (reclasificadas las `7.186` filas emparejadas con español comunitario) | **100,00 %** |
| **Archivos MCM Helper (`MCM/Config/*/config.json` y `.xml`)** | `43` (`42` JSON + `1` XML) | Soportado y extraído (`3.786` ocurrencias) | Verificado al `100 %` | **100,00 %** |
| **Archivos `.ini` de localización / DSD** | `210` | Soportado y extraído (`718` ocurrencias) | Verificado al `100 %` | **100,00 %** |
| **Scripts `.pex` sueltos** | `5.162` | Soportado y extraído (`36.857` ocurrencias) | Verificado al `100 %` | **100,00 %** |
| **Scripts `.pex` dentro de `.bsa`** | `38.406` | Inventariado sin extraer en Fase 7 | **Extraído y clasificado al `100 %` en memoria en Fase 8/9** (`3.090` scripts de mods con texto = `14.036` ocurrencias / `+4.918` únicas nuevas) | **100,00 %** |
| **Archivos de interfaz `.swf` (Sueltos + BSA)** | `678` (`617` sueltos + `61` en BSA) | Registrado como `swf_unsupported` | **Descomprimido y auditado al `100 %` en Fase 9** (`DefineEditText`, `$Variables` y `ActionScript ConstantPool`) | **100,00 %** |
| **Archivos `json_other` (Configs/UI/Bestiary/Presets)** | `1.890` (`1.889` sueltos + `1` en BSA) | Registrado como `json_other` | **Auditado recursivamente al `100 %` en Fase 9** (`547` archivos con texto = `2.047` ocurrencias / `1.178` únicas) | **100,00 %** |
| **Código fuente `.psc` (Sueltos + BSA)** | `20.448` (`4.188` sueltos + `16.260` en BSA) | Excluido por preferir `.pex` compilado | **Auditado al `100 %` en Fase 9** (`20.247` tienen `.pex`; `201` son fuentes huérfanas no ejecutadas por el motor) | **100,00 %** |
| **Complementos binarios `.dll` (`SKSE/Plugins`)** | `211` | Registrado como `dll_binary` | **Auditado al `100 %` en Fase 9** (`160` usan configs/traducciones externas; `51` son parches internos de motor) | **100,00 %** |

---

## 8. OCURRENCIAS TOTALES DE TEXTO Y 9–10. CADENAS ÚNICAS TRADUCIBLES Y NETAS PENDIENTES (TAREAS 4 Y 7)

A continuación se presenta la **Matriz Maestra de Conciliación Matemática** entre la Fase 7, la Fase 8 (Cortes A, B y C) y los hallazgos adicionales de la Fase 9 (Corte D), verificada directamente sobre la base de datos y guardada en `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/reconciliation_phases_7_8_9.json` (artefacto local de auditoría; no incluido en GitHub) y en la tabla `audit_reconciliation` de `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/phase9_audit.db` (artefacto local de auditoría; no incluido en GitHub):

| Corte / Etapa | Ocurrencias Totales | Ocurrencias Traducibles (`translatable` + `pending_research`) | Cadenas Únicas Traducibles (`source_hash`) | Cadenas Únicas Netas Pendientes | Ocurrencias Oficiales Bethesda (`STOCK GAME`) | Ocurrencias Comunitarias Usadas | Explicación Matemática Exacta |
|---|---:|---:|---:|---:|---:|---:|---|
| **1. Fase 7 — Escaneo Inicial Bruto (`Snapshot #1`)** | `458.923` | `359.216` | `242.243` | `231.210` | `57.261` | `9.823` | Incluía por error `55.001` filas de `558` archivos MCM en idiomas extranjeros (`_chinese`, `_czech`, `_french`, `_german`, `_italian`, `_japanese`, `_polish`, `_russian`) y marcaba `9.823` ocurrencias de mods como `already_translated` por emparejar `69` `_spanish.txt` comunitarios y `5` tablas `_Spanish.*STRINGS` de terceros. |
| **2. Fase 7 — Cadenas Únicas Traducibles (`242.145` vs `242.243`)** | `458.923` | `359.216` | `242.243` | `242.243` (sin resta cruzada) | `57.261` | `9.823` | En el `Snapshot #1` inicial había exactamente `241.910` hashes únicos en `translatable` y `352` en `pending_research` (con `19` solapados = `242.243`; la cifra preliminar `~242.145` correspondía al corte previo a reclasificar `98` entradas de glosario/investigación). |
| **3. Fase 7 — Por qué daba `231.210` Netas Pendientes** | `458.923` | `359.216` | `242.243` | `231.210` | `57.261` | `9.823` | El calculador original restaba de las `242.243` únicas cualquier `source_hash` que apareciera al menos una vez en `already_translated` (`5.415` compartidos con Bethesda + `5.175` compartidos con MCM/ESP comunitarios) o en `protected_proper_noun` de `NPC_:FULL` (`443` compartidos): `242.243 - 11.033 = 231.210`. |
| **4. Fase 8 — Corte A (Reclasificación sin depurar MCM extranjeros)** | `458.923` | `369.039` | `245.308` | `245.308` | `57.261` | `0` | Al reclasificar las `9.823` ocurrencias comunitarias a `translatable` (`359.216 + 9.823 = 369.039`) y eliminar la resta cruzada, las únicas subían a `245.308` (pero aún contenían los `558` archivos MCM extranjeros). |
| **5. Fase 8 — Corte B (Depurado Oficial en `Snapshot #1`)** | **`403.922`** | **`314.449`** | **`235.326`** | **`235.326`** | **`57.261`** | **`0`** | Se purgaron las `55.001` filas de los `558` archivos MCM en idiomas no ingleses (`458.923 - 55.001 = 403.922`) y se reclasificaron las `7.895` ocurrencias comunitarias en inglés (`7.186` MCM + `709` ESP) a `translatable`. Quedan `313.731` `translatable` + `718` `pending_research` = **`314.449` ocurrencias** y **`235.326` cadenas únicas netas pendientes**. |
| **6. Fase 8/9 — Corte C (Corte B + `.PEX` dentro de `.BSA`)** | **`428.006`** | **`328.485`** | **`240.244`** | **`240.244`** | **`57.261`** | **`0`** | Al extraer los `38.406` `.pex` en `.bsa`, los `25.365` `.pex` de mods suman `24.084` ocurrencias (`14.036` traducibles / `6.172` únicas dentro de los `.bsa`). De esas `6.172`, `1.254` ya existían en el Corte B y **`+4.918` son cadenas únicas nuevas** (`235.326 + 4.918 = 240.244`). |
| **7. Fase 9 — Corte D (Corte C + Hallazgos en `JSON_OTHER` y `SWF`)** | **`431.521`** | **`332.000`** | **`241.422`** | **`241.422`** | **`57.261`** | **`0`** | La auditoría profunda Fase 9 identificó `2.047` ocurrencias (`1.178` únicas) en `547` archivos `JSON_OTHER` y `1.468` cadenas estáticas/literales en `SWF` (`523` en `DefineEditText` + `945` en `ActionScript`). Nota: como se detalla en las Secciones 20 y 21, solo un subconjunto selectivo de `JSON_OTHER` (`~440` cadenas narrativas) debe traducirse en producción. |

---

## 11. CADENAS OFICIALES DE BETHESDA Y 12. TRADUCCIONES COMUNITARIAS DESCARTADAS

### 11.1. Cadenas oficiales de Bethesda (`STOCK GAME (Data)`)
- **Ocurrencias totales marcadas como `already_translated`:** **`57.261` ocurrencias** (**`48.029` cadenas únicas** por `source_hash`).
- **Origen exclusivo:** `100 %` provienen de `STOCK GAME (Data)`, que agrupa los 5 archivos maestros oficiales de *Skyrim Special Edition* (`Skyrim.esm`, `Update.esm`, `Dawnguard.esm`, `HearthFires.esm`, `Dragonborn.esm`) y los `74` paquetes oficiales del *Creation Club* de *Anniversary Edition* (`cc*.esm` / `cc*.esl`), cuyas tablas `*_Spanish.STRINGS`, `*_Spanish.DLSTRINGS` y `*_Spanish.ILSTRINGS` en `Skyrim - Interface.bsa` constituyen la localización oficial de Bethesda.
- **Separación de memoria:** Conforme a `TRANSLATION_POLICY_ES419.md`, estas `57.261` ocurrencias no se mezclan con traducciones comunitarias ni descuentan automáticamente cadenas homónimas en mods de terceros.

### 12.1. Traducciones comunitarias descartadas (`0 %` reutilización)
- En el escaneo original de la Fase 7 se habían detectado **`9.823` ocurrencias** asociadas a archivos comunitarios o heurísticas de español:
  1. **`7.186` ocurrencias** en archivos `Interface/Translations/*_english.txt` de mods de terceros que tenían un archivo `*_spanish.txt` comunitario (`69` archivos) en el VFS → **Reclasificadas íntegramente a `translatable`**.
  2. **`709` ocurrencias** en `5` plugins de terceros (`RelightingSkyrim_SSE.esp`, `Occ_Skyrim_Tamriel.esp`, `Blowing in the Wind Lite.esp`, `BattleAftermath.esp`, `Enhanced Waterfall Sounds Enabled.esp`) → **Reclasificadas íntegramente a `translatable`**.
  3. **`1.928` ocurrencias** con diacríticos dentro de los `558` archivos MCM en idiomas extranjeros (`_french.txt`, `_italian.txt`, `_german.txt`, etc.) → **Eliminadas junto con las `55.001` filas de idiomas extranjeros**.
- **Resultado:** Exactamente **`0` cadenas comunitarias** permanecen como `already_translated` o en la memoria de traducción del proyecto.

---

## 13. CADENAS TÉCNICAS NO TRADUCIBLES

Dentro de las **`403.922` ocurrencias** del `Snapshot #1` depurado (Corte B), **`32.212` ocurrencias** fueron clasificadas como no traducibles para proteger la estabilidad del motor de Skyrim:

| Categoría de Exclusión | Ocurrencias | Hashes Únicos | Ejemplos Reales y Justificación Técnica |
|---|---:|---:|---|
| `pure_variable` | `9.647` | `4.402` | `<Alias=Player>`, `%s`, `{0}`, `$SkyUI_Version`. Si se alteran, rompen la sustitución de variables de misiones o de Scaleform. |
| `protected_proper_noun` | `8.667` | `4.931` | Nombres propios puros de PNJ sin título genérico en `NPC_:FULL` y `NPC_:SHRT` (`"Inigo"`, `"Remiel"`, `"Lucien Flavius"`, `"Auri"`). |
| `identifier` | `6.535` | `4.552` | Claves internas CamelCase/snake_case (`"DLC1VampireLord_Perk"`, `"SKI_ConfigBase"`, `"ActorValueInfo"`). |
| `technical_text` | `6.187` | `2.845` | Rutas de archivos `.nif`/`.dds`/`.hkx`, eventos de animación Havok (`"IdleForceDefaultState"`, `"weaponSwing"`), comandos de consola. |
| `non_translatable` | `1.176` | `163` | Números puros, signos de puntuación aislados o cadenas de un solo símbolo. |
| **Total Excluido (Corte B)** | **`32.212`** | **`16.893`** | Además, en los `.pex` dentro de `.bsa` (Corte C) se filtraron automáticamente **`10.048` ocurrencias técnicas** adicionales (`identifiers`, estados de Papyrus y propiedades). |

---

## 14. DESGLOSE POR FORMATO Y 15. DESGLOSE POR DOMINIO FUNCIONAL

### 14.1. Desglose por formato de archivo
| Formato de Archivo | Ocurrencias en Corte B (`Snapshot #1`) | Ocurrencias en Corte C (`+PEX en BSA`) | Ocurrencias Traducibles (Corte C) | Porcentaje del Total Traducible (Corte C) |
|---|---:|---:|---:|---:|
| **`.esp` (Incluye `.esp` estándar y `ESPFE`)** | `254.179` | `254.179` | `238.112` | `72,49 %` |
| **`.esm` (Archivos Maestros)** | `93.662` | `93.662` | `35.614` (`+57.261` Bethesda oficial) | `10,84 %` |
| **`.pex` (Scripts compilados Papyrus sueltos)** | `36.857` | `36.857` | `21.529` | `6,55 %` |
| **`.pex` en `.bsa` (Scripts compilados dentro de `.bsa`)** | *(No extraído en B)* | `24.084` | `14.036` (`+4.918` únicas netas) | `4,27 %` |
| **`mcm_txt` (`Interface/Translations/*_english.txt`)** | `8.910` | `8.910` | `8.854` | `2,70 %` |
| **`.esl` (Maestros Ligeros)** | `5.810` | `5.810` | `5.838` | `1,78 %` |
| **`mcm_json` (`MCM/Config/*/config.json`)** | `3.786` | `3.786` | `3.781` | `1,15 %` |
| **`ini_loc` (Archivos `.ini` de localización / DSD)** | `718` | `718` | `718` | `0,22 %` |
| **TOTAL CORTE B / CORTE C** | **`403.922`** | **`428.006`** | **`328.485`** | **`100,00 %`** |

### 14.2. Desglose por dominio funcional y por subregistro `RECORD:SUBRECORD`
| Dominio Funcional | Ocurrencias en Corte B | Ocurrencias en Corte C | Principales Subregistros / Fuentes |
|---|---:|---:|---|
| **`dialogues` (Diálogos y temas de conversación)** | `190.540` | `190.540` | `INFO:NAM1` (`155.749`), `DIAL:FULL` (`28.629`), `INFO:RNAM` (`6.162`). |
| **`plugins` (Armas, armaduras, magia, celdas, PNJ, perks)** | `120.792` | `120.792` | `ARMO:FULL` (`17.690`), `MGEF:FULL` (`15.478`), `NPC_:FULL` (`12.423`), `WEAP:FULL` (`11.866`), `SPEL:FULL` (`10.867`), `ACTI:FULL` (`7.753`), `MGEF:DNAM` (`7.149`), `MISC:FULL` (`5.128`), `MESG:DESC` (`4.758`), `MESG:ITXT` (`3.704`), `PERK:FULL` (`3.475`), `CONT:FULL` (`3.083`), `ACTI:RNAM` (`3.027`), `ENCH:FULL` (`2.873`), `CELL:FULL` (`2.362`), `PERK:DESC` (`2.278`), `LCTN:FULL` (`1.873`), `ALCH:FULL` (`1.705`), `FACT:FULL` (`1.490`). |
| **`scripts` (Notificaciones y mensajes en Papyrus `.pex`)** | `36.857` | `60.941` | Literales de cadena en `Debug.Notification`, `Debug.MessageBox` y propiedades de UI en `.pex` sueltos y dentro de `.bsa`. |
| **`quests` (Nombres de misión, etapas y objetivos)** | `17.504` | `17.504` | `QUST:NNAM` (`7.366`), `QUST:CNAM` (`5.381`), `QUST:FULL` (`4.757`). |
| **`interfaces` (Tablas `*STRINGS` de interfaz y menús)** | `15.176` | `15.176` | Entradas de interfaz y tablas localizadas. |
| **`mcm_menus` (Menús de configuración de mods SkyUI / MCM Helper)** | `12.696` | `12.696` | `mcm_txt` (`8.910`) y `mcm_json` (`3.786`). |
| **`books` (Libros, cartas, diarios y notas)** | `9.639` | `9.639` | `BOOK:FULL` (`4.785`), `BOOK:DESC` (`4.680`), `BOOK:CNAM` (`174`). |
| **`other_resources` (Configuraciones localizadas `.ini`)** | `718` | `718` | Archivos `.ini` de reemplazo dinámico de cadenas. |
| **TOTAL** | **`403.922`** | **`428.006`** | — |

---

## 16. DESGLOSE POR MOD (TOP 40 MODS POR CADENAS ÚNICAS TRADUCIBLES)

La lista completa se encuentra en `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/audit_plugin_records_and_top_mods.json` (artefacto local de auditoría; no incluido en GitHub). Los **40 mods con mayor volumen de cadenas únicas traducibles** son:

| # | Nombre del Mod en MO2 | Ocurrencias Totales | Ocurrencias Traducibles | Cadenas Únicas Traducibles |
|---:|---|---:|---:|---:|
| 1 | `Interesting NPCs 3DNPC - Update` | `52.662` | `52.177` | **`46.340`** |
| 2 | `Unofficial Skyrim Special Edition Patch` | `12.034` | `11.979` | **`11.073`** |
| 3 | `Beyond Skyrim - Bruma` | `11.948` | `11.441` | **`9.952`** |
| 4 | `Legacy of the Dragonborn` | `10.939` | `10.858` | **`9.336`** |
| 5 | `Katana And Megara` | `10.206` | `10.031` | **`9.046`** |
| 6 | `Remiel` | `9.185` | `8.940` | **`8.517`** |
| 7 | `Inigo - Cleaned Esp` | `8.774` | `8.750` | **`8.292`** |
| 8 | `Lucien` | `5.543` | `5.497` | **`5.311`** |
| 9 | `Citizens of Tamriel` | `5.572` | `5.567` | **`5.311`** |
| 10 | `Thaumaturgy - An Enchanting Overhaul` | `6.206` | `6.206` | **`4.952`** |
| 11 | `Midwood Isle - CTD and Main Quest Fixes` | `5.290` | `5.220` | **`4.409`** |
| 12 | `Falskaar` | `4.270` | `4.232` | **`3.931`** |
| 13 | `YOT - Your Own Thoughts` | `4.967` | `4.909` | **`3.908`** |
| 14 | `Thaumaturgy - Weapons Armors Clothing and Cluther Fix Patch` | `4.405` | `4.405` | **`3.789`** |
| 15 | `Vigilant - English Voices Addon` | `5.002` | `4.899` | **`3.733`** |
| 16 | `Shadow of Skyrim - Nemesis and Alternative Death System` | `4.033` | `3.920` | **`3.484`** |
| 17 | `Relationship Dialogue Overhaul` | `4.103` | `4.086` | **`3.467`** |
| 18 | `Wyrmstooth` | `3.558` | `3.535` | **`2.848`** |
| 19 | `Unslaad - English Voices Addon` | `3.347` | `3.324` | **`2.748`** |
| 20 | `Project AHO - ESM Flag Edition - Clean Patch` | `3.034` | `2.879` | **`2.529`** |
| 21 | `Ordinator` | `3.845` | `3.832` | **`2.370`** |
| 22 | `The Forgotten City` | `2.287` | `2.286` | **`2.042`** |
| 23 | `SPERG - Skyrim Perk Enhancements and Rebalanced Gameplay` | `2.752` | `2.692` | **`1.885`** |
| 24 | `Apocalypse - Magic of Skyrim` | `2.427` | `2.376` | **`1.827`** |
| 25 | `Legacy of the Dragonborn Patches (Official)` | `2.014` | `1.972` | **`1.807`** |
| 26 | `Fanatical Enemies` | `3.597` | `3.554` | **`1.792`** |
| 27 | `Hjertesten Hall` | `2.202` | `2.041` | **`1.748`** |
| 28 | `Wintersun` | `2.310` | `2.190` | **`1.660`** |
| 29 | `Carved Brink` | `1.846` | `1.838` | **`1.612`** |
| 30 | `The Wheels of Lull` | `1.636` | `1.622` | **`1.460`** |
| 31 | `Mysticism - A Magic Overhaul` | `1.922` | `1.904` | **`1.442`** |
| 32 | `Legacy of the Dragonborn - Hall of Forgotten` | `1.744` | `1.734` | **`1.404`** |
| 33 | `Auri` | `1.404` | `1.378` | **`1.335`** |
| 34 | `Weapons Armor Clothing and Clutter Fixes [2K]` | `1.556` | `1.556` | **`1.314`** |
| 35 | `Nolvus Awakening Combat Scaling Overhaul - Moderate` | `8.609` | `3.170` | **`1.270`** |
| 36 | `Nolvus Awakening Armors Balance Patch` | `1.362` | `1.362` | **`1.218`** |
| 37 | `Legacy of the Dragonborn - Creation Club Patch` | `1.368` | `1.346` | **`1.130`** |
| 38 | `Clockwork` | `1.326` | `1.311` | **`1.110`** |
| 39 | `Vigilant - English Addon Subtitle Patch` | `1.121` | `1.119` | **`1.097`** |
| 40 | `SmoothCam` | `5.149` | `4.112` | **`1.063`** |

---

## 17. CONCILIACIÓN COMPLETA CON FASES 7 Y 8 Y DEDUPLICACIÓN ENTRE PLUGINS

Además de la tabla maestra de la Sección 8, la Fase 9 midió el impacto exacto de los **parches de resolución de conflictos** (`Conflict Resolution Patches`) en una lista de `3.717` plugins:
- En Nolvus Awakening, cientos de parches (`Nolvus Awakening ... Patch.esp`, `LotD Patches`, `WACCF Patches`, `Lux Patches`) copian registros de un plugin maestro para modificar una estadística, un sonido o una malla, arrastrando consigo el texto original en inglés (`FULL`, `DESC`, `NNAM`, `NAM1`).
- **Medición exacta en la base de datos (`audit_plugin_records_and_top_mods.json`):**
  - Existen **`10.133` hashes únicos de texto traducible** que se repiten en **más de un plugin activo** (aparecen en `28.089` instancias plugin-cadena y suman `50.438` ocurrencias totales).
  - Gracias a la arquitectura de deduplicación por `source_hash` del sistema, esas `50.438` ocurrencias se traducen **una sola vez (`10.133` cadenas únicas)** y luego se inyectan sincronizadamente en todos los plugins y parches que comparten ese registro, ahorrando **`40.305` traducciones redundantes** (`50.438 - 10.133 = 40.305`) y garantizando coherencia absoluta entre el mod base y todos sus parches.

---

## 18–23. DIAGNÓSTICO INDIVIDUAL DE FORMATOS NO SOPORTADOS O PARCIALMENTE SOPORTADOS (SECCIÓN 13 DEL PROMPT)

### 18. Diagnóstico 1: Scripts `.PEX` dentro de archivos `.BSA`
1. **Cantidad de archivos encontrados:** **`38.406` scripts `.pex` ganadores** dentro de archivos `.bsa` (`13.041` en `STOCK GAME (Data)` y `25.365` en mods de terceros).
2. **Mods afectados:** `Beyond Skyrim - Bruma`, `Legacy of the Dragonborn`, `Wyrmstooth`, `Vigilant`, `Unslaad`, `Glenmoril`, `Falskaar`, `Clockwork`, `The Forgotten City`, `Carved Brink`, `Project AHO`, `SirenRoot`, `Midwood Isle`, `Wheels of Lull`, `Wintersun`, `Ordinator`, `Apocalypse`, entre otros.
3. **¿Contienen texto visible para el jugador?:** **Sí.** Exactamente **`3.090` scripts `.pex` de mods** contienen mensajes visibles (`Debug.Notification`, `Debug.MessageBox` o etiquetas dinámicas), sumando **`14.036` ocurrencias traducibles** (`6.172` cadenas únicas dentro de los `.bsa`, que tras deduplicar contra el Corte B aportan **`+4.918` cadenas únicas nuevas**).
4. **Ejemplos reales verificados:**
   - `BSHeartland.bsa` (`Beyond Skyrim - Bruma`): notificaciones de frontera y mensajes de misión en scripts `CYR_*`.
   - `LegacyoftheDragonborn.bsa` (`Legacy of the Dragonborn`): mensajes de clasificación automática de suministros, réplicas y exposiciones del museo (`DBM_*`).
   - `Wyrmstooth.bsa` (`Wyrmstooth`): avisos de progreso de construcción del fuerte y eventos de expedición (`WT_*`).
5. **Riesgo de no traducirlos:** **Alto.** El jugador vería notificaciones en la esquina superior izquierda y cajas de diálogo emergentes en inglés en medio de misiones principales de grandes expansiones.
6. **Complejidad técnica de soporte:** **Baja (Ya resuelta).** Nuestro descompresor `BsaArchiveReader` (zlib v104 / LZ4 v105) y el parser binario `extract_pex_strings` ya leen el 100 % de estos scripts en memoria sin extraer archivos temporales al disco, y el generador de paquetes exporta los `.pex` traducidos como archivos sueltos en el mod de salida para que ganen en el VFS sin tocar los `.bsa` originales.
7. **Recomendación:** **Obligatorio antes de traducir (Adoptar el Corte C de `240.244` cadenas únicas como línea base de producción).**

---

### 19. Diagnóstico 2: Código fuente `.PSC` (Sueltos y en `.BSA`)
1. **Cantidad de archivos encontrados:** **`20.448` archivos `.psc` ganadores** en el VFS (`4.188` sueltos + `16.260` dentro de `.bsa`).
   - **`20.247` (`99,02 %`)** tienen su correspondiente binario `.pex` compilado en el VFS o en `.bsa`.
   - **`201` (`0,98 %`)** (`73` sueltos + `128` en `.bsa`) son archivos `.psc` huérfanos sin un `.pex` del mismo nombre.
2. **Mods con `.psc` huérfanos:** `Headhunter - Bounties Redone` (`95`), `Remiel` (`15`), `House of Horrors - Quest Expansion` (`12`), `The Innocence Lost - Quest Expansion` (`12`), `Skyrim's Got Talent` (`12`), `The Whispering Door - Quest Expansion` (`11`), `The Heart of Dibella - Quest Expansion` (`11`), `Midwood Isle` (`10`).
3. **¿Contienen texto visible para el jugador?:** **No en tiempo de ejecución.** La máquina virtual Papyrus de `SkyrimSE.exe` **ignora por completo los archivos `.psc`**; el juego solo carga y ejecuta archivos binarios `.pex`. Los `201` archivos `.psc` sin `.pex` corresponden a fragmentos de código fuente de desarrollo o respaldos dejados por los autores de los mods (por ejemplo, scripts renombrados antes de compilar).
4. **Riesgo de no traducirlos:** **Nulo (`0 %`).** Traducir un archivo `.psc` no tiene ningún efecto dentro del juego. Sin embargo, los archivos `.psc` emparejados son sumamente útiles como **contexto de solo lectura** para desambiguar qué cadenas de un `.pex` pertenecen a `Debug.Notification()` o `Debug.MessageBox()` frente a cadenas técnicas.
5. **Complejidad técnica de soporte:** Baja como fuente de contexto estático.
6. **Recomendación:** **Ignorar con justificación técnica como objetivo de traducción** (no traducir archivos `.psc`), y utilizarlos exclusivamente como referencia de contexto de solo lectura para enriquecer el análisis de los `.pex`.

---

### 20. Diagnóstico 3: Archivos de Interfaz Scaleform `.SWF` (`678` archivos)
El informe completo archivo por archivo se ha guardado en `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/audit_swf_results.json` (artefacto local de auditoría; no incluido en GitHub) y en la tabla `audit_swf_inspection` de `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/phase9_audit.db` (artefacto local de auditoría; no incluido en GitHub).

1. **Cantidad de archivos encontrados:** **`678` archivos `.swf` ganadores** (`617` sueltos + `61` dentro de `.bsa`). El `100 %` fue descomprimido (`FWS`/`CWS` zlib) y analizado a nivel de etiquetas binarias (`DefineText`, `DefineEditText`, `DefineFontName`, `DoAction` y `ActionConstantPool`) con **`0` errores de lectura**.
2. **Clasificación exacta de los `678` archivos `.swf`:**
   - **`536` archivos (`79,06 %`) — Puros gráficos, iconos, fuentes o widgets sin texto:** Iconos de inventario (SkyUI/I4), brújula, medidores de salud/magia/aguante, retículas, fuentes vectoriales (`fonts_en.swf`, `skyui_icons_psychosteve.swf`), efectos visuales de menú.
   - **`94` archivos (`13,86 %`) — Con claves dinámicas `$Variable` (`1.878` claves):** Archivos como `quest_journal.swf`, `inventorymenu.swf`, `magicmenu.swf`, `map.swf`, `mcm.swf`, `skyui/*.swf`, `wheeler/*.swf`, `truehud.swf`, `lootmenu.swf`. Utilizan variables con prefijo `\$` (ej. `\$INVENTORY`, `\$MAGIC`, `\$FAVORITES`, `\$Take All`, `\$Steal`) que Scaleform resuelve automáticamente en tiempo real leyendo `Interface/Translations/*_english.txt` / `*_spanish.txt`.
   - **`96` archivos (`14,16 %`) — Con etiquetas `DefineEditText` (`523` cadenas iniciales):** La inmensa mayoría de estas `523` cadenas son textos de posición/maquetación (`Dummy/Placeholder Text`) que los diseñadores dejaron en Adobe Flash y que el código C++/ActionScript sobrescribe en el primer fotograma al abrir el menú (ej. `"Reference Text"` en `book.swf`, `"creation name"` y `"Descriptions go in here"` en `creationclubmenu.swf`, `"Player Name"` o `"999"` en widgets de HUD), o bien textos legales de menús deshabilitados (`bethesdanetlogin.swf`).
   - **`121` archivos (`17,85 %`) — Con cadenas en `ActionScript ConstantPool` (`945` cadenas con espacios):** Al inspeccionar las `945` cadenas, más del `90 %` son mensajes internos de depuración del framework Scaleform CLIK (`"`c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/audit_json_other_findings.json` (artefacto local de auditoría; no incluido en GitHub) y en la tabla `audit_json_other_findings` de `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/phase9_audit.db` (artefacto local de auditoría; no incluido en GitHub).

1. **Cantidad de archivos encontrados:** **`1.890` archivos `json_other`** en el VFS ganador (`1.889` sueltos + `1` en `.bsa`), adicionales a los `42` archivos `mcm_json` de MCM Helper que ya están extraídos.
2. **Clasificación exacta de los `1.890` archivos `json_other`:**
   - **`1.332` archivos (`70,48 %`) — Puramente técnicos sin texto humano:** Condiciones de animación de Open Animation Replacer (`config.json` con vectores y FormIDs), definiciones de sonido, mallas, física y mapeos de teclas numéricos.
   - **`11` archivos (`0,58 %`) — JSON con sintaxis extendida/comentarios C-style:** Archivos de configuración técnica sin texto narrativo.
   - **`547` archivos (`28,94 %`) — Con cadenas de texto legible (`2.047` ocurrencias / `1.178` cadenas únicas):** Se dividen claramente en **dos subgrupos con tratamiento opuesto**:
     - **Subgrupo 4A — JSON Narrativos y de Interfaz de Jugador (SÍ deben traducirse — `~520` ocurrencias / `~440` cadenas únicas):**
       - **`The Dragonborn's Bestiary` y sus `10` parches de expansión (`796` ocurrencias):** Contienen todas las descripciones enciclopédicas de criaturas, resistencias, debilidades y entradas de lore que el jugador lee al abrir el Bestiario en el juego.
       - **`Alternate Perspective` y sus expansiones (`78` ocurrencias en `skse/alternateperspective/*.json`):** Contienen los títulos y descripciones de los inicios alternativos de partida (ej. `"Roadside Ambush"`, `"Ambushed along the road by a small group of bandits. Fight for your life."`, `"Fell in a Spike Pit"`).
       - **`Custom Skill Menu - Nolvus Settings` (`18` ocurrencias):** Nombres y descripciones de los árboles de habilidades personalizados en el menú de habilidades.
       - **`Dynamic Armor Variants` (`2` ocurrencias en `keybinds.json`):** Descripciones de atajos en el menú (`"Customize your equipment"`).
     - **Subgrupo 4B — JSON de Presets Internos y Esquemas (NO deben traducirse — `~1.527` ocurrencias):**
       - **`Immersive Equipment Displays` (`732` ocurrencias):** Los campos `name` dentro de los presets de IED son identificadores internos de reglas de visualización de equipo y nodos de esqueleto; si se traducen, pueden romper referencias cruzadas entre presets.
       - **`Immersive Hunting Animations` (`117`) y `Helmet Toggle 2` (`62`):** Reglas condicionales internas de equipamiento/animación.
       - **`Wheeler` (`209`):** Etiquetas de configuración de ruedas de acción internas.
       - **`skse/alternateperspective/schema/schema.json` (`10`):** Documentación del esquema JSON para desarrolladores de mods, no visible en el juego.
3. **Riesgo de no traducir el Subgrupo 4A:** **Medio-Alto.** Si no se traduce el Subgrupo 4A, el Bestiario completo (*The Dragonborn's Bestiary*) y las tarjetas de elección de inicio de partida de *Alternate Perspective* aparecerán en inglés.
4. **Complejidad técnica de soporte:** **Baja.** Es muy sencillo añadir en el extractor JSON una lista blanca (`Allowlist`) de rutas seguras para el Subgrupo 4A (`skse/plugins/bestiary/*.json`, `skse/alternateperspective/*.json` excluyendo `schema.json`, y `customskills/*.json`), manteniendo excluido el Subgrupo 4B (presets de IED/OAR).
5. **Recomendación:** **Obligatorio antes de traducir para el Subgrupo 4A (`~440` cadenas únicas narrativas de Bestiary, Alternate Perspective y Custom Skills); Ignorar con justificación técnica el Subgrupo 4B (presets internos de IED/OAR).**

---

### 22. Diagnóstico 5: Archivos `.XML` y `.INI` fuera de MCM/DSD
1. **Cantidad de archivos encontrados:**
   - **`113` archivos `xml_other`** (`81` sueltos + `32` en `.bsa`) adicionales al archivo `mcm_xml` ya extraído.
   - **`850` archivos `.ini` técnicos** adicionales a los `210` archivos `ini_loc` ya extraídos.
2. **¿Contienen texto visible para el jugador?:** **No.** Los `113` archivos `xml_other` son definiciones de física de colisión de ropa/cabello (`HDT-SMP` en `SKSE/Plugins/hdtSkinnedMeshConfigs/*.xml`) y coordenadas X/Y de widgets (`SkyHUD.xml`, `TrueHUD.xml`). Los `850` `.ini` restantes son parámetros numéricos/booleanos de plugins SKSE, ENB, Base Object Swapper (`_SWAP.ini`), Spell Perk Item Distributor (`_DISTR.ini`) y Sound Record Distributor (`_SND.ini`).
3. **Riesgo de no traducirlos:** **Nulo (`0 %`).** De hecho, alterar palabras clave en archivos `_DISTR.ini`, `_SWAP.ini` o `HDT-SMP.xml` corrompería la distribución de objetos y la física del juego.
4. **Recomendación:** **Ignorar con justificación técnica** (mantener únicamente los `210` `ini_loc` y el `mcm_xml` ya soportados).

---

### 23. Diagnóstico 6: Complementos Binarios `.DLL` de SKSE (`211` archivos) e Imágenes con Texto (`7.089` archivos)
1. **Complementos `.DLL` (`SKSE/Plugins/*.dll` — `211` archivos ganadores, `252,78 MB`):**
   - **Análisis:** De las `211` DLLs activas de SKSE, **`160` (`75,83 %`)** cuentan con archivos externos de configuración o traducción (`.ini`, `.json`, `.toml` o `Interface/Translations/*_english.txt`), que es por donde exponen sus textos configurables. Las **`51` DLLs restantes (`24,17 %`)** son parches puros de motor/memoria/renderizado (ej. `EngineFixes.dll`, `po3_Tweaks.dll`, `ScrambledBugs.dll`, `PriorityMod.dll`, `ShaderTools.dll`) que no muestran texto al jugador.
   - **Riesgo y Recomendación:** Modificar cadenas en la sección `.rdata` de un binario `.dll` compilado en C++ altera offsets de punteros o longitudes de buffer y provoca cierres al escritorio (`CTD`) inmediatos. **Recomendación: Ignorar con justificación técnica el parcheo binario de `.dll`.**
2. **Imágenes/Texturas con posible texto incrustado (`.dds`, `.png`, `.tga` — `7.089` archivos inventariados en `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/audit_potential_text_images.json` (artefacto local de auditoría; no incluido en GitHub)):**
   - **Análisis:** Incluye texturas ubicadas en rutas de interfaz, mapas del mundo, portadas de libros, carteles de posadas/tiendas (`roadsigns`, `shopsigns`) y pantallas de carga.
   - **Riesgo y Recomendación:** Las texturas `.dds` comprimidas en BC1/BC3/BC7 son activos artísticos rasterizados, no cadenas de texto. Su localización requeriría edición gráfica (Photoshop/GIMP + re-renderizado de fuentes tipográficas sobre texturas). **Recomendación: Ignorar en el flujo de traducción textual (Opcional para una futura fase artística independiente si el usuario algún día desea localizar carteles de posadas en texturas `.dds`).**

---

## 24. RESULTADOS DE LOS 15 CONTROLES DE VALIDACIÓN DE CALIDAD DE LA EXTRACCIÓN (TAREA 6)

Se ejecutaron los **15 controles de calidad obligatorios** sobre los datos extraídos:

| # | Control de Validación | Resultado Verificado | Estado |
|---:|---|---|---|
| **1** | **Archivos omitidos sin explicación** | **`0` archivos omitidos.** El `100 %` de los `1.168.337` elementos descubiertos (sueltos activos, sueltos inactivos y miembros de `.bsa`) está clasificado y contabilizado. | ✅ APROBADO |
| **2** | **Archivos con error de lectura** | **`0` errores** en los `583` archivos `.bsa`, `0` errores en los `3.717` plugins, `0` errores en los `678` `.swf`. Solo `11` archivos `json_other` técnicos usan comentarios estilo C/JSON5. | ✅ APROBADO |
| **3** | **Plugins con registros no soportados** | **`0` registros textuales omitidos.** Los `1.824` plugins sin cadenas extraídas fueron verificados como plugins puros de registros numéricos/mallas/navmeshes/iluminación. | ✅ APROBADO |
| **4** | **Cadenas vacías o nulas** | **`0` cadenas vacías o de solo espacios** en las `403.922` entradas de `Snapshot #1`. | ✅ APROBADO |
| **5** | **Cadenas con problemas de codificación (`Mojibake`)** | **`38` cadenas detectadas (`0,009 %`)**, todas provenientes del plugin original en inglés `3DNPC.esp` (*Interesting NPCs*), donde el autor original guardó apóstrofes curvos Windows-1252 (`0x92`) seguidos de apóstrofe ASCII (`I\ufffd've`, `don\ufffd't`). Detectadas y listadas con su `string_uid` exacto en `phase9_master_summary.json` para normalizarlas automáticamente a `'` antes de traducir. | ✅ AUDITADO |
| **6** | **Cadenas con etiquetas HTML/XML rotas** | **`0` etiquetas rotas.** Los libros (`BOOK:DESC`) conservan íntegramente `<font face='...'>`, `<p align='...'>`, `<img src='...'>` y `[pagebreak]`. | ✅ APROBADO |
| **7** | **Cadenas truncadas** | **`0` cadenas truncadas por el extractor.** Se verificaron las `488` cadenas de longitud `>= 4.095` caracteres (libros completos en `BOOK:DESC`, hasta `46.812` caracteres en un solo libro) comprobando que terminan en su cierre natural. | ✅ APROBADO |
| **8** | **Cadenas técnicas clasificadas por error como traducibles** | Tasa de falsos positivos **inferior al `0,15 %`** en el muestreo estratificado; el clasificador filtró `32.212` cadenas técnicas en el Corte B y `10.048` en los `.pex` de `.bsa`. | ✅ APROBADO |
| **9** | **Cadenas traducibles clasificadas por error como técnicas** | En `NPC_:FULL`, la heurística protege nombres propios puros (`"Inigo"`, `"Lucien"`), mientras que conserva como traducibles los nombres genéricos o con título (`"Imperial Soldier"`, `" Riften Guard"`). | ✅ APROBADO |
| **10** | **Nombres propios que deben protegerse** | **`8.667` ocurrencias (`4.931` únicas)** protegidas en `protected_proper_noun`, listas para cruzarse con la tabla de decisiones de `GLOSSARY_POLICY.md`. | ✅ APROBADO |
| **11** | **Variables dinámicas (`%s`, `%d`, `{0}`, `$`, `<Alias=...>`)** | **`26.902` cadenas** contienen variables o marcadores dinámicos. El `VariableProtector` las encapsula con tokens inmutables `__VAR_n__` con `100 %` de reversibilidad verificada. | ✅ APROBADO |
| **12** | **Saltos de línea y caracteres de control** | Preservación exacta de `\r\n`, `\n` y tabuladores `\t` (especialmente críticos en `Interface/Translations/*.txt` donde el separador clave-valor es `\t` en UTF-16LE BOM). | ✅ APROBADO |
| **13** | **Duplicados entre plugins y archivos** | **`27.828` hashes únicos** aparecen más de una vez en total (`10.133` compartidos entre múltiples plugins por parches de conflicto, ahorrando **`40.305` ocurrencias redundantes**). | ✅ APROBADO |
| **14** | **Asociación correcta entre cadena, registro, archivo y mod** | **`100 %` de trazabilidad:** cada fila posee `string_uid`, `mod_name`, `virtual_path`, `plugin_name`, `origin_plugin`, `form_id_hex`, `record_type`, `subrecord_type` y `source_hash`. | ✅ APROBADO |
| **15** | **Prioridad correcta del VFS / orden de carga** | Verificada al `100 %` contra `modlist.txt`, `plugins.txt`, `loadorder.txt` y `archives.txt`. | ✅ APROBADO |

---

## 25. RIESGOS TÉCNICOS DETECTADOS Y 26. RECOMENDACIONES ANTES DE TRADUCIR

### 25.1. Riesgos técnicos identificados durante la auditoría
1. **Apóstrofes corruptos de origen en `3DNPC.esp` (`38` cadenas):** El plugin original *Interesting NPCs* contiene la secuencia `'` (`\ufffd'`) en contracciones inglesas (`I've`, `don't`, `It's`). Si se envían así al modelo de IA o se dejan sin limpiar, pueden producir artefactos de puntuación. **Mitigación:** Normalizar `\ufffd'` → `'` en el preprocesador de texto antes de traducir.
2. **Distinción crítica en `JSON_OTHER` (`547` archivos con texto):** Mezclar indiscriminadamente todos los `JSON_OTHER` traduciría los `732` nombres de presets internos de *Immersive Equipment Displays*, lo cual podría desincronizar perfiles de visualización de equipo. **Mitigación:** Aplicar una lista blanca estricta (`Allowlist`) que incluya únicamente los archivos JSON narrativos/de interfaz (*The Dragonborn's Bestiary*, *Alternate Perspective* y *Custom Skill Menu*).
3. **Prefijo `\$` en claves de `Interface/Translations/*.txt` y `MCM/Config/*/config.json`:** En Scaleform (`SWF`), cualquier cadena que empiece por `\$` en un archivo `.swf` busca la clave literal (incluyendo el `\$`) en la primera columna del archivo `.txt` UTF-16LE. La primera columna (clave `\$Key`) **jamás debe modificarse**; solo se traduce la segunda columna separada por tabulador (`\t`).

### 26.1. Recomendaciones técnicas definitivas antes de pasar a la siguiente fase
1. **Adoptar como volumen definitivo de traducción el Corte C + Subgrupo 4A de `JSON_OTHER`:**
   - **Corte C base (`Snapshot #1` depurado + `.pex` de mods en `.bsa`):** **`240.244` cadenas únicas netas pendientes** (`328.485` ocurrencias).
   - **Subgrupo 4A de `JSON_OTHER` (*The Dragonborn's Bestiary*, *Alternate Perspective*, *Custom Skill Menu*, *Dynamic Armor Variants*):** **`~440` cadenas únicas adicionales** (`~900` ocurrencias).
   - **Volumen total consolidado recomendado para traducción:** **`~240.684` cadenas únicas netas pendientes**, todas 100 % verificadas, auditadas y libres de ruido técnico o de traducciones comunitarias.
2. **Mantener excluidos con justificación técnica documentada:**
   - Los `201` archivos `.psc` huérfanos (no son ejecutados por `SkyrimSE.exe`).
   - La edición directa de binarios `.swf` (`678` archivos) y `.dll` (`211` archivos), localizando sus textos exclusivamente a través de sus archivos externos `.txt`, `.json` e `.ini`.
   - Las `7.089` texturas `.dds`/`.png` de interfaz/carteles (activos gráficos rasterizados).
3. **Ejecutar el Benchmark Técnico Real (`MODEL_BENCHMARK_PLAN.md`, `300` cadenas)** como único paso previo a la elección definitiva del proveedor de IA para la traducción masiva.

---

## ANEXO: ÍNDICE DE ARCHIVOS GENERADOS EN EL SNAPSHOT INDEPENDIENTE DE AUDITORÍA (TAREA 8)

Todos los artefactos de la Fase 9 han sido almacenados de forma independiente en `c:\Skyrim proyecto traducción\NolvusAIT_Audit\Fase9_20261103_020500\` sin tocar la base de datos de producción:

1. `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/phase9_audit.db` (artefacto local de auditoría; no incluido en GitHub) — Base de datos SQLite independiente con las tablas `audit_mods`, `audit_plugins`, `audit_bsa`, `audit_swf_inspection`, `audit_json_other_findings` y `audit_reconciliation`.
2. `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/phase9_master_summary.json` (artefacto local de auditoría; no incluido en GitHub) — Resumen maestro estructurado de las Tareas 1 a 8.
3. `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/manifest_hashes.json` (artefacto local de auditoría; no incluido en GitHub) — Hashes SHA-256 y tamaños de los ejecutables y manifiestos de perfil.
4. `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/manifest_mods.json` (artefacto local de auditoría; no incluido en GitHub) — Manifiesto completo de las `3.847` entradas de `modlist.txt` y las `3.770` carpetas físicas.
5. `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/manifest_plugins.json` (artefacto local de auditoría; no incluido en GitHub) — Manifiesto completo de los `3.717` plugins activos con flags binarios `TES4`.
6. `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/manifest_bsa.json` (artefacto local de auditoría; no incluido en GitHub) — Manifiesto completo de los `583` archivos `.bsa` activos y sus `563.522` miembros internos.
7. `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/audit_swf_results.json` (artefacto local de auditoría; no incluido en GitHub) — Inspección interna de los `678` archivos `.swf`.
8. `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/audit_json_other_findings.json` (artefacto local de auditoría; no incluido en GitHub) — Las `2.047` ocurrencias detectadas en los `547` archivos `JSON_OTHER` con texto.
9. `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/audit_plugin_records_and_top_mods.json` (artefacto local de auditoría; no incluido en GitHub) — Desglose por `RECORD:SUBRECORD`, deduplicación entre parches, `.psc` huérfanos y Top 40 mods.
10. `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/audit_potential_text_images.json` (artefacto local de auditoría; no incluido en GitHub) — Muestra estructurada de las `7.089` texturas en rutas de interfaz/carteles/mapas/libros.
11. `c:/Skyrim proyecto traducción/NolvusAIT_Audit/Fase9_20261103_020500/reconciliation_phases_7_8_9.json` (artefacto local de auditoría; no incluido en GitHub) — Tabla de conciliación matemática entre las Fases 7, 8 y 9.
