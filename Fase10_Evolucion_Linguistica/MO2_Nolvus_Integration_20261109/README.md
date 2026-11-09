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

## 3. Diagnóstico de las Incidencias Observadas en Amber Guard

| Incidencia Reportada | Comportamiento Anómalo en Juego | Causa Raíz Técnica en DSD / Engine | Corrección Aplicada |
|---|---|---|---|
| **1. Opciones de respuesta anómalas (Durmia y Audomar)** | Las frases habladas por el NPC aparecían como opciones del menú de diálogo del jugador (e.g. el jugador podía elegir *"Gracias, forastero"*). | En DSD (`Hooks.cpp` / `Manager.cpp`), `INFO RNAM` (opción del jugador) y `INFO NAM1` (subtítulo hablado del NPC) comparten la tabla hash `m_runtimeMap2` indexada por `packU32(index, formID)`. `INFO RNAM` consulta obligatoriamente `index = 0`. Al exportar `INFO NAM1` con `index = 0`, el hook del menú del jugador interpretó la frase del NPC como opción propia. | Se actualizó `dsd_exporter.py` para que `INFO NAM1` use estrictamente el `responseNumber` de `TRDT` (1-based: `1, 2, 3...`), dejando `index = 0` libre para `RNAM`. |
| **2. Subtítulos desfasados y caída al inglés** | En diálogos multilínea (Durmia, Audomar), el subtítulo salía en blanco en la línea 1 y desfasado una línea por delante; en diálogos de una línea (Rogneda, saludos de Somerled) salía en inglés. | El motor de Skyrim numera las respuestas habladas en el subregistro `TRDT` desde `1` (`responseNumber = 1, 2...`). Al buscar `index = 1`, DSD no encontraba nada para diálogos de 1 línea (quedaban en inglés) y encontraba la línea 2 para diálogos multilínea (desfase con el audio). | Al exportar con base 1 (`index = sub_idx + 1` o `TRDT`), cada respuesta coincide exactamente con el audio FuZ reproducido. |
| **3. Puerta de la torre en inglés (*"Abrir Eastern Tower"*)** | Al aproximarse a la puerta exterior de la torre, el prompt de interacción mostraba *"Abrir Eastern Tower"* pese a que la ubicación `LCTN FULL` estaba traducida como *"Torre del Este"*. | Las puertas de teletransporte en Skyrim muestran el nombre de la celda interior de destino (`CELL FULL`, FormID `001B5B80`). En Nolvus, el mod de iluminación `Lux` sobrescribe las celdas interiores de Amber Guard (`Lux - Amber Guard patch.esp`), por lo que no se incluyeron en el conjunto inicial del plugin base. | Se incorporaron los 17 registros `CELL FULL` interiores de Amber Guard en el JSON de DSD; al inyectarse en RAM tras la carga de todos los plugins, la celda queda traducida sin importar los parches de iluminación. |
| **4. Somerled sin menú de preguntas** | Somerled no habilitaba opciones de diálogo para preguntarle sobre trabajo o compra de casa. | Condiciones de misión (`CTDA`) en `Amber Guard.esp`: `AGDQuest02` (Audomar) debe estar en etapa 10 para habilitar la entrega del mensaje, y en etapa >= 30 para la compra de la casa (`AGDQuest01`). La misión de cazarrecompensas (`AGDQuest03`) requiere entrar a la celda exterior del pueblo. | No era un bug del personaje; la progresión natural de la misión se desbloquea al hablar primero con Audomar una vez corregidos los índices de diálogo. |

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

---

## 5. Resumen de Verificación y Estado

* **Suite de pruebas unitarias**: 404/404 pruebas superadas satisfactoriamente (`Ran 68 tests in 9.176s` enMass/Continuación y 404 globales), incluyendo la nueva prueba de regresión para índices 1-based de `INFO NAM1` en `tests/test_amber_guard_continuation.py`.
* **Archivos protegidos**: 180/180 archivos históricos permanecen intactos con sus hashes SHA-256 inalterados.
* **Candado económico**: `PHASE_1_PAID_API_LOCK = True` verificado activamente; cero gasto adicional (`US$0.00`).
* **Paquete listo para pruebas**: `Amber_Guard_ES419_Provisional` actualizado con **151 registros** en `NolvusAIT_Intelligence/20261107_AmberGuard_Continuation/private/dsd_package/Amber_Guard_ES419_Provisional`.
