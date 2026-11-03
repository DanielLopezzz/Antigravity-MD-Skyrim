# Formatos Soportados, Reglas de Extracción Segura y Limitaciones Transparentes

**NOLVUS AI TRANSLATOR** declara un formato como **soportado** únicamente cuando dispone de parser estructural determinista, validador de integridad y pruebas automatizadas que lo demuestran.

---

## 1. Matriz de Soporte de Formatos (Estado Verificado por Pruebas)

| Familia | Extensión / Ruta | Extracción | Reconstrucción / Exportación | Método de Salida en MO2 | Nivel de Soporte |
| :--- | :--- | :---: | :---: | :--- | :--- |
| **Plugins Bethesda** | `.esp`, `.esm`, `.esl` (incl. `ESPFE` y registros comprimidos `zlib`) | Sí (Binario nativo) | Sí (No invasiva) | **DSD JSON** (`SKSE/Plugins/DynamicStringDistributor/`) o `.STRINGS` | **Soportado y Verificado** |
| **Tablas de Cadenas** | `.strings`, `.dlstrings`, `.ilstrings` (`UTF-8` / `CP1252` fallback) | Sí (Binario nativo) | Sí (`UTF-8` exacto) | `Strings/<Plugin>_Spanish.<ext>` | **Soportado y Verificado** |
| **Scripts Papyrus Compilados** | `.pex` (Big-Endian `0xFA57C0DE`, v3.1/v3.2) | Sí (AST completo 36 opcodes) | Sí (Append seguro en `StringTable` sin tocar IDs) | `Scripts/<Script>.pex` *(opcional, bajo confirmación explícita)* | **Soportado y Verificado** |
| **Scripts Papyrus Fuente** | `.psc` (`Scripts/Source/`) | Sí (Literales de cadena) | Solo referencia contextual | Usado como contexto auxiliar para `.pex` (no altera binarios) | **Lectura Contextual Verificada** |
| **Traducciones MCM / SkyUI** | `Interface/Translations/*_<LANG>.txt` | Sí (`UTF-16LE BOM` / `UTF-8` fallback) | Sí (`UTF-16LE BOM` + `\t` estricto) | `Interface/Translations/<Mod>_SPANISH.txt` | **Soportado y Verificado** |
| **Configuración MCM Helper** | `MCM/Config/<Mod>/config.json` | Sí (Campos UI + cruce con `$tokens`) | Sí (Genera `<Mod>_SPANISH.txt` o JSON seguro) | `Interface/Translations/<Mod>_SPANISH.txt` | **Soportado y Verificado** |
| **Archivos XML de Mods** | `.xml` (MCM XML, SkyUI configs, FOMOD `ModuleConfig.xml`) | Sí (`lxml` selectivo por esquema) | Sí (Solo esquemas localizables confirmados) | Clasifica XML técnicos (ej. `Nemesis`/`Precision`) como `no traducibles` | **Soportado Selectivamente** |
| **Archivos Comprimidos Bethesda** | `.bsa` (v104 `zlib` y v105 `LZ4`) | Sí (Inventario + extracción selectiva en RAM) | Nunca modifica el `.bsa` original | Exporta las traducciones como **archivos sueltos** en el mod de MO2 | **Soportado (Solo Lectura Segura)** |
| **INIs de Distribución / Texto** | `*_DISTR.ini` (*Description Framework*), `.ini` con claves de texto UI | Sí (Claves textuales explícitas) | Inventario y parche suelto opcional | Archivos `.ini` técnicos del motor se marcan como `no traducibles` | **Soportado Selectivamente** |
| **Interfaces Scaleform Compiladas** | `.swf` (Flash binario compilado) | Inventariado | No | Marcado explícitamente como **`no compatible` (`UNSUPPORTED_SWF`)** | **Inventariado / No Compatible** |
| **Audio y Voces Lip-Sync** | `.fuz`, `.xwm`, `.wav` | Inventariado | No (los subtítulos van en `INFO NAM1` / `.ILSTRINGS`) | Subtítulos soportados vía plugin/DSD; audio original intacto | **Audio No Procesable (Subtítulos Sí)** |
| **Plugins Nativos SKSE** | `.dll` (C++ compilado) | Inspección PE de versión | No | Marcado como **`no traducible` (`BINARY_CODE`)** | **Solo Inspección de Compatibilidad** |

---

## 2. Subregistros Traducibles Soportados en Plugins (`ESP / ESM / ESL`)

El parser binario inspecciona grupos (`GRUP`) recursivamente y extrae exclusivamente subregistros textuales verificados:

* **Misiones (`QUST`)**: `FULL` (título), `CNAM` (entradas del diario), `NNAM` (objetivos de etapa).
* **Diálogos (`DIAL`, `INFO`)**: `DIAL FULL` (tópico), `INFO RNAM` (prompt), `INFO NAM1` (respuesta hablada/subtítulo, vinculada automáticamente a su `DIAL` padre mediante `GRUP` tipo 7).
* **Libros, Cartas y Notas (`BOOK`)**: `FULL` (título), `DESC` (cuerpo completo con etiquetas `<font>`, `[pagebreak]`, `<img>`), `CNAM` (descripción).
* **Equipo y Objetos (`WEAP`, `ARMO`, `AMMO`, `ALCH`, `INGR`, `MISC`, `SLGM`, `KEYM`, `LIGH`, `SCRL`, `APPA`)**: `FULL`, `DESC`.
* **Magia, Gritos y Ventajas (`SPEL`, `MGEF`, `ENCH`, `PERK`, `SHOU`, `WOOP`, `AVIF`, `RACE`, `CLAS`)**:
  * `SPEL`: `FULL`, `DESC`
  * `MGEF`: `FULL`, `DNAM`
  * `ENCH`: `FULL`
  * `PERK`: `FULL`, `DESC`, `EPF2`, y **`EPFD` únicamente cuando el subregistro `EPFT` precedente tiene valor `0x07` (cadena)** — garantizando que jamás se corrompan multiplicadores `float` de perks.
  * `SHOU`: `FULL`, `DESC` | `WOOP`: `FULL`, `TNAM` | `AVIF` / `RACE` / `CLAS`: `FULL`, `DESC`.
* **Personajes, Criaturas y Facciones (`NPC_`, `FACT`)**: `NPC_ FULL`, `NPC_ SHRT`, `FACT FULL`, `FACT MNAM`, `FACT FNAM`.
* **Mundo y Pantallas de Carga (`CELL`, `WRLD`, `LCTN`, `REGN`, `LSCR`)**: `CELL FULL`, `WRLD FULL`, `LCTN FULL`, `REGN RDMP`, `LSCR DESC`.
* **Mensajes, Ajustes e Interacción (`MESG`, `GMST`, `ACTI`, `CONT`, `DOOR`, `FURN`, `FLOR`, `TACT`, `TREE`, `HAZD`, `PROJ`)**:
  * `MESG`: `FULL`, `DESC`, `ITXT` (botones indexados).
  * `GMST`: **`DATA` únicamente cuando `EDID` comienza con `'s'`** (descartando ajustes `f*` float, `i*` int y `b*` bool).
  * `ACTI`, `FLOR`: `FULL`, `RNAM` (verbo de activación). Resto: `FULL`.

---

## 3. Seguridad Matemática en Scripts Papyrus (`.pex`)

En un archivo `.pex`, la `StringTable` global mezcla nombres de clases, estados, funciones, propiedades y variables con literales de cadena mostrados al jugador. Incluso peor: el compilador Papyrus deduplica cadenas, por lo que un identificador interno y un literal visible pueden compartir el **mismo índice `uint16`**.

Nuestro motor `pex_parser.py` aplica tres barreras de seguridad:
1. **Decodificación completa de `Objects` e `Instructions` (`0x00`..`0x23`)**: Verifica que el cursor binario coincida al byte exacto con `Object.size`. Si hay cualquier discrepancia, aborta la modificación y marca el archivo como `EXTRACTION_FAILED`.
2. **Filtro de Contexto de Llamada**: Solo clasifica como traducibles los operandos `VariableData(type=2)` pasados a funciones visibles al usuario (ej. `Debug.Notification`, `Debug.MessageBox`, `AddTextOption`, `SetTitleText`, variables de mensaje), bloqueando automáticamente llamadas técnicas al motor (`SendAnimationEvent`, `GetAnimationVariable*`, `NetImmerse`, `UI.Invoke*` rutas ActionScript, `ConsoleUtil`, `JContainers`).
3. **Estrategia Append-Only en Índices Compartidos**: Al reconstruir un `.pex`, si el índice de un literal traducido es referenciado en cualquier otro punto como identificador (`type=1` o metadato estructural), el índice original queda intacto; se añade una nueva entrada al final de `StringTable` y se actualiza únicamente el operando `uint16` de la instrucción `VariableData(type=2)` traducida.
