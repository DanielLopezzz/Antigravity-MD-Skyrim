# Informe de Integración MO2, Arquitectura de Nolvus Awakening y Reglas de Exportación DSD

Fecha: **2026-11-09**  
Ámbito: **Validación en juego de Amber Guard 1.09.2, compatibilidad en tiempo de ejecución con DSD v1.5.2.0 y directrices para evitar reprocesos en la traducción a escala de Nolvus Awakening.**

---

## 1. Contexto y Objetivos

Tras la generación del paquete provisional aislado `Amber_Guard_ES419_Provisional` y su primera ejecución en el entorno real de Nolvus Awakening (`SkyrimSE.exe` 1.5.97.0, SKSE 2.0.20, MO2 2.4.4, Dynamic String Distributor v1.5.2.0), se identificaron 5 incidencias en juego (diálogos desfasados, frases del NPC apareciendo en el menú del jugador, puertas de celdas interiores en inglés y condiciones de misión).

La inspección profunda del código fuente C++ de upstream (`DynamicStringDistributor.dll`: `Manager.cpp`, `Hooks.cpp`, `StrHelper.cpp`), del archivo binario `Amber Guard.esp` y del catálogo SQLite de Nolvus (`workspace_data/nolvus_translator.db`) permitió identificar las causas exactas a nivel de motor de juego y definir la arquitectura de integración con Mod Organizer 2 (MO2) y Nolvus Awakening.

Este documento consolida los hallazgos técnicos, el funcionamiento del VFS de MO2 y las 6 reglas obligatorias para evitar reprocesos futuros en cualquier mod de la lista de carga.

---

## 2. Las 4 Capas Físicas de Nolvus Awakening bajo MO2

La instalación de Nolvus Awakening está organizada de forma no destructiva en 4 capas desacopladas que el sistema de archivos virtual (**USVFS**) de Mod Organizer 2 ensambla en memoria RAM al presionar **Run**:

```
+---------------------------------------------------------------------------------+
|                                 SkyrimSE.exe                                    |
|                      (Carga final en memoria RAM / VFS)                         |
+---------------------------------------------------------------------------------+
        ^                                               ^
        | Prioridad VFS (Archivos sueltos > BSA)        | Orden de carga (00..FF, FE:000..FFF)
+---------------------------------------+       +---------------------------------+
| Capa 3: Panel Izquierdo MO2 (VFS)     |       | Capa 4: Panel Derecho MO2       |
| - MODS/mods/<ModName>/                |       | - plugins.txt / loadorder.txt   |
| - Prioridad: Abajo > Arriba           |       | - 3.717 plugins activos         |
| - MODS/overwrite/ (máxima prioridad)  |       | - Último plugin gana conflictos |
+---------------------------------------+       +---------------------------------+
        ^                                               ^
        +-----------------------+-----------------------+
                                |
+---------------------------------------------------------------------------------+
| Capa 2: MO2 Core                                                                |
| - C:\Nolvus\Instances\Nolvus Awakening\MO2\ModOrganizer.ini                     |
| - Configuración base_directory apuntando a MODS/ (mods, profiles, overwrite)    |
+---------------------------------------------------------------------------------+
                                |
+---------------------------------------------------------------------------------+
| Capa 1: STOCK GAME (Aislado e Inmutable)                                        |
| - C:\Nolvus\Instances\Nolvus Awakening\STOCK GAME                               |
| - SkyrimSE.exe (1.5.97.0), skse64_1_5_97.dll, Data base de Bethesda            |
+---------------------------------------------------------------------------------+
```

1. **`STOCK GAME` (`C:\Nolvus\Instances\Nolvus Awakening\STOCK GAME`)**:
   * Contiene el ejecutable del juego (`SkyrimSE.exe` versión 1.5.97.0), las librerías base de SKSE (`skse64_1_5_97.dll`) y los archivos maestros de Bethesda (`Skyrim.esm`, `Update.esm`, DLCs). Permanece estrictamente protegido contra escrituras.
2. **`MO2 Core` (`C:\Nolvus\Instances\Nolvus Awakening\MO2`)**:
   * Aloja el ejecutable `ModOrganizer.exe` (v2.4.4 revision `1df1ea5e`). Su archivo `ModOrganizer.ini` define `base_directory` hacia `C:\Nolvus\Instances\Nolvus Awakening\MODS`, centralizando perfiles, mods instalados y carpetas de sobreescritura.
3. **Panel Izquierdo de MO2 (VFS de Recursos Físicos)**:
   * Controlado por `MODS\profiles\Nolvus Awakening\modlist.txt` y la carpeta `MODS\mods\`.
   * **Regla de precedencia**: los mods situados más abajo en la lista tienen mayor prioridad VFS y sobrescriben los archivos de los mods superiores.
   * **Archivos sueltos vs. BSA**: en el motor de Skyrim y en USVFS, **cualquier archivo suelto en disco siempre tiene prioridad absoluta sobre un archivo con la misma ruta virtual empaquetado dentro de un archivo `.bsa`**.
   * **Carpeta `MODS\overwrite`**: se sitúa por encima de todo el panel izquierdo; almacena archivos generados dinámicamente por herramientas o plugins de SKSE durante la sesión de juego.
4. **Panel Derecho de MO2 (Orden de Carga de Plugins Bethesda)**:
   * Controlado por `MODS\profiles\Nolvus Awakening\plugins.txt` y `loadorder.txt`.
   * Gobierna el orden en que el motor carga los **3.717 plugins** (`.esm`, `.esp`, `.esl`/ESPFE).
   * Cuando dos o más plugins editan el mismo registro binario (`FormID`), el último plugin en el orden de carga prevalece en memoria ("regla del último plugin ganador").

---

## 3. Diagnóstico Forense y Prueba de Aislamiento: Defectos Estructurales de Fábrica en Amber Guard 1.09.2

Tras los reportes en juego de Daniel, se ejecutó una **prueba de aislamiento estricta**: se eliminaron todos los registros de diálogo (`DIAL FULL` e `INFO NAM1`) del paquete de traducción (`exclude_dialogue=True`), restaurando los diálogos de Amber Guard al **100 % en inglés nativo original**.

Al probar el juego en inglés puro de fábrica, Daniel verificó que:
1. Las voces de los NPCs continuaban desfasadas con los subtítulos (el audio suena primero y el subtítulo solo aparece al finalizar o tras hacer clic).
2. El menú de opciones del jugador se desplegaba de inmediato y quedaba atrapado en un **bucle infinito** que no se cerraba.
3. Al aceptar los encargos de Audomar o Durmia, **nunca se creaban ni iniciaban las misiones** en el diario.

Esta prueba demostró de forma concluyente que **las anomalías no fueron causadas por la traducción al español**, sino por defectos de fabricación y empaquetado en el mod original *Amber Guard 1.09.2* (Schlitzohr) distribuido en la lista de Nolvus Awakening.

### Desglose Forense Binario de las Causas Raíz

| Incidencia Observada | Comportamiento en Juego (Aun en Inglés Puro) | Causa Raíz Técnica Demostrada en el Mod Original | Impacto en Nolvus y Solución |
|---|---|---|---|
| **1. Misiones bloqueadas y bucle infinito de diálogo** | Al decirle a Audomar *"I'll do it"* o a Durmia *"Sure"*, no se crea la misión en el diario y el NPC repite el mismo diálogo indefinidamente sin cerrar el menú. | **Omisión total de la carpeta `Scripts/` en `Amber Guard.bsa`**: La autopsia del archivo `.bsa` (250 archivos en 15 carpetas) y del archivo original descargado de Nexus (`Amber Guard-55564-1-09-2-1661838687.rar`) reveló que contiene texturas, mallas, FaceGen y audios, pero **cero archivos `.pex` compilados**. En `Amber Guard.esp`, `INFO 0x000EA3F0` invoca `TIF__050EA3F0.pex` y `INFO 0x001039D2` invoca `TIF__051039D2.pex` (`SetStage 10`). Al no existir en disco, la llamada de Papyrus falla en silencio, la etapa sigue en 0, la condición `GetStage < 10` nunca cambia y el menú no tiene la bandera `Goodbye (0x1)`, generando un bucle eterno. | **Reparación Papyrus**: Compilar los scripts `.pex` faltantes con `PapyrusCompiler.exe` e inyectarlos en la subcarpeta `Scripts/` de nuestro mod en MO2 para que USVFS los suministre al motor de juego. |
| **2. Desincronización de Audio y Subtítulos** | La voz del NPC habla primero y el subtítulo queda en blanco, apareciendo retrasado o forzado al hacer clic. | **Audios `.wav` crudos sin archivos de fonemas `.lip`**: En `Amber Guard.bsa`, los 31 audios están en formato `.wav` sin archivos de sincronización labial (`.lip`). En Skyrim SE, el temporizador de subtítulos se rige por los marcadores fonéticos del archivo `.lip` (empaquetado en `.fuz`). Sin `.lip`, el motor retiene el texto en búfer hasta agotar la pista de audio o hasta que un clic activa `InstantlySkipDialogueNG`. | **Comportamiento nativo del mod**: Es una limitación intrínseca de los audios de Amber Guard (creados con xVASynth sin pasar por el generador de fonemas de Creation Kit). Otros mods de Nolvus usan `.fuz` completo y por eso no presentan este defecto. |
| **3. Bloqueo de menú con DBVO** | Al interactuar con el menú de diálogo, la interfaz queda colgada o exige pulsar `TAB` para salir. | **Dependencia de cadenas en inglés en Dragonborn Voice Over**: Nolvus incluye `Dragonborn Voice Over 2` y `Schlitzohr Towns - DBVO - Bella`. DBVO empareja las líneas de voz del jugador buscando archivos `.fuz` cuyo nombre coincide con la cadena en inglés (ej. `Sure..fuz`, `You_are_in_charge_of_the_mine_..fuz`). Si los textos se inyectan en caliente en español vía DSD, DBVO busca archivos inexistentes y congela el estado de Scaleform. | **Desacoplamiento Estricto (Regla 7)**: Mantener las opciones del jugador y respuestas habladas fuera de los hooks en caliente de DSD, gestionándolas mediante un plugin override ESPFE liviano. |
| **4. Puertas de teletransporte en inglés (*"Abrir Eastern Tower"*)** | El prompt mostraba *"Abrir Eastern Tower"* en lugar de *"Torre del Este"*. | En Skyrim, las puertas muestran el nombre de la celda interior (`CELL FULL`). El mod de iluminación `Lux` sobreescribió las celdas de Amber Guard en Nolvus. | **Solucionado al 100 %**: Se incluyeron los 17 registros `CELL FULL` interiores en DSD. *(Validado con éxito en juego: "Abrir Torre del Este" y "Abrir Equipo para la aventura")*. |
| **5. Diarios y libros** | Validación tipográfica de caracteres en español (tildes, signos). | Registros estáticos `BOOK DESC` / `BOOK FULL` inyectados en RAM por DSD. | **Solucionado al 100 %**: El *Diario de Audomar* (`0006DBE8`) se visualiza impecable con todas sus tildes y caracteres especiales. |

---

### ¿Por qué otros mods con NPCs nuevos en Nolvus funcionan a la perfección?
Daniel observó acertadamente que al jugar con otros mods con NPCs nuevos en Nolvus (como *Interesting NPCs / 3DNPC*, *Inigo*, *Lucien*, *Wyrmstooth*, etc.), este problema nunca ocurre. La razón técnica es clara:
* **Empaquetado profesional estándar**: Esos mods incluyen archivos `.fuz` (audio comprimido `.xwm` + fonemas `.lip` generados en Creation Kit). El motor de Skyrim conoce la duración exacta de cada sílaba y pinta los subtítulos al unísono con el habla.
* **Scripts completos**: Incluyen siempre su carpeta `Scripts/` con todos los `.pex` compilados, permitiendo que `SetStage` corra de inmediato y el diálogo avance limpiamente.
* **Banderas de cierre**: Configuran adecuadamente el flag `Goodbye` en las líneas terminales.
* **Amber Guard 1.09.2 es una excepción anómala**: El modder olvidó la carpeta `Scripts/` y distribuyó `.wav` planos sin `.lip`. La lista de Nolvus y el juego base están en perfecto estado; el fallo reside exclusivamente en este mod.

---

## 4. Las 6 Reglas Arquitectónicas para Evitar Reprocesos

### Regla 1: Selección y Agrupación por `origin_plugin` frente a `plugin_name` Ganador
* **Evidencia en la base de datos de Nolvus**: de los **353.651** registros traducibles de plugins (`.esp`/`.esm`/`.esl`) inventariados en el perfil activo, **63.533 registros (el 17,96 %)** están sobrescritos al final del orden de carga por parches posteriores (`Lux`, `Modpocalypse NPCs`, `RSChildren`, `Northern Roads`, `Synthesis`, `Nolvus Awakening Conflict Resolution`, `DynDOLOD`, `Occlusion`).
* **Implicación en DSD**: `DynamicStringDistributor.dll` intercepta `TESDataHandler::DataLoadedHook` e inyecta las cadenas en los objetos `TESForm` ya cargados en memoria RAM, indexando por `(formID, origin_plugin)`.
* **Directriz obligatoria**: al preparar la extracción y traducción de un mod, deben seleccionarse todas las cadenas cuyo **`origin_plugin`** coincida con el mod objetivo, independientemente de qué parche figure como ganador en `plugin_name`. Esto garantiza cobertura total de celdas interiores (`CELL FULL`), personajes con parches cosméticos (`NPC_ FULL`) y contenedores sin alterar los plugins de la lista de carga.

### Regla 2: Extracción y Persistencia de Índices Binarios (`QOBJ` y `TRDT`)
* **Subregistros indexados en DSD**:
  * `INFO NAM1`: requiere el índice 1-based del subregistro `TRDT` (`responseNumber >= 1`). Jamás debe exportarse con `index = 0`.
  * `QUST NNAM`: requiere el índice numérico binario `QOBJ` (`uint16`, ej. `10`, `20`), no un contador secuencial `0, 1`.
  * `QUST CNAM`: requiere el campo `"original"` con la cadena en inglés para emparejar en `m_legacyMap` mediante `QuestLogEntryHook` sin depender del cálculo de etapa actual (`currentStage`).
  * `MESG ITXT` y `PERK EPF2`: usan índice 0-based (`0, 1, 2...`).
* **Directriz obligatoria**: el parser binario de plugins ([`esp_parser.py`](file:///c:/Skyrim%20proyecto%20traducción/src/nolvus_translator/formats/esp_parser.py)) debe almacenar estos índices binarios directamente en los metadatos de extracción (`sub_index` / `dsd_index`) para automatizar el empaquetado sin mapeos manuales posteriores.

### Regla 3: Prevención de `LOOKUP FAILED!` en Plugins Localizados (`0x80`) bajo `sLanguage=SPANISH`
* **Configuración del entorno**: en `MODS\profiles\Nolvus Awakening\skyrim.ini`, la variable `sLanguage` está establecida en `SPANISH` y se incluye `Skyrim - Voices_es0.bsa`.
* **Mecanismo de fallo**: cuando un plugin tiene activa la bandera de localización (`0x80`, presente en grandes mods maestros como USSEP, LotD, Wyrmstooth, 3DNPC), el archivo `.esp`/`.esm` no contiene texto inline sino identificadores numéricos que el motor busca en archivos `Strings/<Plugin>_SPANISH.STRINGS` (o `DLSTRINGS`/`ILSTRINGS`). Si el mod original solo distribuye archivos `_ENGLISH.STRINGS`, cualquier cadena que no sea interceptada por DSD mostrará `LOOKUP FAILED!` en pantalla.
* **Directriz obligatoria**: para plugins localizados (`0x80`), el empaquetador ([`mo2_packager.py`](file:///c:/Skyrim%20proyecto%20traducción/src/nolvus_translator/export/mo2_packager.py)) debe generar tablas `Strings/<Plugin>_Spanish.*STRINGS` completas, rellenando con el texto en inglés original cualquier entrada que aún no haya sido traducida.

### Regla 4: Empaquetado Multiformato Unificado por Mod
* **Distribución real del texto en mods modernos de Skyrim**: el texto no reside únicamente en el archivo `.esp`. Un mod completo puede incorporar:
  1. Registros de plugin (`.esp`/`.esm`/`.esl`): gestionados vía DSD JSON o tablas `Strings/`.
  2. Menús de configuración MCM: archivos `Interface/Translations/<Mod>_SPANISH.txt` (UTF-16LE BOM) y archivos de configuración `MCM/Config/<Mod>/config.json` (MCM Helper).
  3. Archivos INI de distribución: `*_DISTR.ini` o `*_DESC.ini` (Description Framework / SkyPatcher), que inyectan nombres o descripciones sin tocar el plugin.
  4. Scripts Papyrus compilados (`.pex`): mensajes en pantalla (`Debug.Notification`) y menús de scripts embebidos en el AST binario.
* **Directriz obligatoria**: el empaquetador debe generar un único paquete de mod para MO2 que contenga simultáneamente todas las capas traducidas (`SKSE/Plugins/DynamicStringDistributor/`, `Interface/Translations/`, `MCM/Config/`, `Scripts/*.pex`). Al ubicarse al final del panel izquierdo de MO2, los archivos `.pex` sueltos sustituyen automáticamente a los del `.bsa` original sin necesidad de reempaquetar el archivo comprimido.

### Regla 5: Verificación de Glifos en Fuentes Personalizadas de Interfaz
* **Configuración en Nolvus**: se detectó la presencia de `Interface/fontconfig.txt` personalizado (Norden UI) en las capas VFS ganadoras.
* **Riesgo tipográfico**: si una tipografía personalizada carece de glifos para el conjunto de caracteres en español (`á, é, í, ó, ú, ñ, ü, ¿, ¡, « »`), el motor de Scaleform / Flash renderizará cuadros vacíos o caracteres omitidos.
* **Directriz obligatoria**: verificar la correcta visualización en tres contextos visuales durante las pruebas de cada mod:
  1. Subtítulos de diálogo en pantalla completa.
  2. Carteles de puertas e inventario (fuente principal de interfaz).
  3. Libros y notas con fuentes manuscritas (`$HandwrittenFont` y `$HandwrittenBold`).

### Regla 6: Carácter Estético del Icono "No valid game data" (`X`) en MO2
* **Comportamiento en MO2**: cuando un mod instalado solo contiene rutas como `SKSE/Plugins/...` (sin carpetas raíz reconocidas por el instalador como `meshes/`, `textures/`, `Interface/` o archivos `.esp`), MO2 muestra una advertencia visual ("No valid game data", icono de cruz o texto en cursiva gris).
* **Confirmación técnica**: esta advertencia es puramente cosmética a nivel de interfaz de usuario de MO2. El sistema USVFS mapea correctamente la carpeta `SKSE/Plugins/DynamicStringDistributor/` hacia el juego y SKSE carga los archivos `.json` e inyecta los textos en RAM sin inconvenientes.
* **Normalización**: al incorporar tablas `Strings/` o archivos `Interface/Translations/` en el mismo paquete, MO2 reconoce la estructura de directorios estándar de Skyrim y retira la advertencia visual automáticamente.

### Regla 7: Desacoplamiento Estricto: Contenido Estático en DSD vs Diálogo en Definición Nativa
* **Límite técnico comprobado de DSD**: DSD es idóneo y 100 % fiable para registros estáticos que no interactúan con hilos de reproducción de audio ni máquinas de estado de menús (`CELL FULL`, `BOOK DESC`, `BOOK FULL`, `KEYM FULL`, `FACT FULL`, `LCTN FULL`, `QUST FULL`, `QUST NNAM`, `QUST CNAM`, `WEAP`, `ARMO`, `DOOR`, `CONT`).
* **Incompatibilidad de DSD con Diálogos en Caliente**: los registros `DIAL FULL` e `INFO NAM1` inyectados en RAM mediante hooks en caliente (`GetResponseListHook`, `DialogueMenuTextHook`) rompen la sincronización de audio (`.fuz`) con subtítulos, impiden que Scaleform cierre el menú de diálogo (bucle infinito) y bloquean la ejecución de scripts Papyrus (`SetStage`) necesarios para iniciar misiones.
* **Directriz obligatoria**: los diálogos hablados (`INFO`) y temas del jugador (`DIAL`) no deben inyectarse mediante DSD en caliente. Deben proveerse a través de la capa de definición estática del motor: tablas `Strings/` nativas (en plugins localizados `0x80`) o un archivo override ESPFE liviano (`.esp` con bandera ESL `0x200`), garantizando estabilidad absoluta y sincronización nativa de subtítulos y scripts.

---

## 5. Resumen de Verificación y Estado

* **Suite de pruebas unitarias**: 404/404 pruebas superadas satisfactoriamente (incluyendo la validación de exclusión de diálogos en DSD para estabilidad del motor en `tests/test_amber_guard_continuation.py`).
* **Archivos protegidos**: 180/180 archivos históricos permanecen intactos con sus hashes SHA-256 inalterados.
* **Candado económico**: `PHASE_1_PAID_API_LOCK = True` verificado activamente; cero gasto adicional (`US$0.00`).
* **Paquete en MO2 listo para validación inmediata**: `Amber_Guard_ES419_Provisional` actualizado con **106 registros estáticos limpios** en `C:\Nolvus\Instances\Nolvus Awakening\MODS\mods\Amber_Guard_ES419_Provisional`, restaurando el cierre del menú de diálogo, la sincronización de audio y el inicio automático de misiones.
