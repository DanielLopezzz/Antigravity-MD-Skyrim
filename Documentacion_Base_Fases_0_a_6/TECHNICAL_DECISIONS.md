> **ADR — REGISTRO HISTÓRICO DE DECISIONES:** Los ADR documentan decisiones adoptadas en sus fases de origen y no deben leerse como una lista de configuración vigente en bloque. Las decisiones posteriores de Fase 9/10 y el estado canónico en [CURRENT_PROJECT_STATE.md](../documentacion/HISTORICO.md) prevalecen cuando exista conflicto. Una ADR antigua se conserva para explicar por qué se tomó una decisión, no para declarar que los paquetes o modelos siguen instalados/disponibles.

---

# Registro de Decisiones Técnicas (ADR — Technical Decisions)

Este documento recoge las decisiones arquitectónicas fundamentales adoptadas tras la investigación de la **Fase 0** para **NOLVUS AI TRANSLATOR**.

---

## ADR-001: Elección de Lenguaje y Stack Tecnológico (`Python 3.13` + `SQLite 3.50` + Stack Permisivo)

### Contexto
Se evaluaron dos ecosistemas principales para Windows, Skyrim y MO2:
1. **C# / .NET 8+ (WPF / WinUI 3 + Mutagen + Microsoft.Data.Sqlite)**
2. **Python 3.13 (`sqlite3` FTS5 + `Pydantic` + `RapidFuzz` + `lxml` + GUI nativa `ttk` oscura virtualizada / arquitectura desacoplada para `PySide6`)**

### Comparativa Técnica
* **C# / .NET**:
  * *Ventajas*: Alto rendimiento bruto y ecosistema `Mutagen` para plugins Bethesda.
  * *Desventajas críticas en este entorno*:
    1. **Mutagen es GPL-3.0** (copyleft fuerte que condiciona toda la distribución de la aplicación).
    2. **El SDK de `.NET` no está instalado** en la estación de trabajo actual, mientras que `Python 3.13.7` sí está instalado y operativo con bibliotecas clave compiladas en C/C++ (`RapidFuzz`, `pydantic_core` en Rust, `lxml` en C, `sqlite3 3.50.4`).
    3. Menor flexibilidad inmediata para pipelines futuros de IA, tokenización, heurísticas NLP de lore y prototipado rápido verificable.
* **Python 3.13**:
  * *Ventajas*:
    1. Disponible nativamente en el entorno (`Python 3.13.7`) con extensiones nativas de altísimo rendimiento (`pydantic_core` en Rust, `RapidFuzz` en C++, `lxml` en C, `sqlite3 3.50.4` con FTS5 y modo WAL).
    2. `struct`, `zlib`, `mmap` y `ctypes` permiten parsear binarios Bethesda (`ESP`, `STRINGS`, `PEX`, `BSA`) y cabeceras Win32 PE (`SkyrimSE.exe`) a velocidades de decenas de megabytes por segundo sin dependencias GPL/LGPL.
    3. Stack **100 % libre de licencias GPL y LGPL**.

### Decisión
Adoptar **Python 3.13** con arquitectura hexagonal (núcleo y servicios completamente desacoplados de la interfaz gráfica), persistencia local en **SQLite 3.50 (WAL + FTS5)** y dependencias exclusivamente permisivas (MIT / BSD / PSF).

---

## ADR-002: Interfaz Gráfica de Escritorio Desacoplada y Virtualizada

### Contexto
Una instalación de Nolvus Awakening supera los 2.000 mods y puede contener cientos de miles de cadenas. Además, se requiere evaluar cualquier impacto LGPL antes de incorporar `PySide6`.

### Decisión
1. Implementar la aplicación de escritorio completa (las 10 secciones requeridas) sobre el motor gráfico nativo de Python/Windows (`tkinter.ttk` con tema oscuro personalizado *Nordic Slate* y **paginación virtual indexada sobre SQLite**), evitando cualquier dependencia LGPLv3 y garantizando ejecución inmediata offline sin descargas externas.
2. Aislar toda la lógica de negocio tras `ProjectService`, de modo que las vistas nunca ejecuten SQL directo ni bloqueen el hilo principal (procesamiento asíncrono mediante `ThreadPoolExecutor` + cola de eventos thread-safe con soporte de cancelación y reanudación por checkpoints).

---

## ADR-003: Parsers Binarios Bethesda Propios (Clean-Room) vs. Dependencias Copyleft

### Decisión
Implementar parsers propios verificados byte a byte según las especificaciones públicas de formato (UESP):
* **`esp_parser.py`**: Soporta `ESP`, `ESM`, `ESL` y `ESPFE` (cabecera `TES4`, bandera `Localized 0x80`, bandera `ESL 0x200`, `MAST` para resolución exacta de FormID base, grupos `GRUP` recursivos para vincular respuestas `INFO` con su tópico `DIAL`, descompresión `zlib` en registros con flag `0x00040000`, subregistros extendidos `XXXX`, verificación de tipo `EPFT == 7` en `PERK EPFD` y prefijo `'s'` en `GMST EDID`).
* **`strings_table.py`**: Lectura y escritura binaria exacta de `.STRINGS` (`zstring` UTF-8) y `.DLSTRINGS`/`.ILSTRINGS` (`uint32 length` + `zstring` UTF-8).
* **`pex_parser.py`**: Parser y reconstructor completo del AST binario Big-Endian (`0xFA57C0DE`), decodificando los 36 opcodes de Skyrim (`0x00`..`0x23`), verificando `Object.size` al byte exacto y separando matemáticamente `VariableData(type=1)` (Identificadores bloqueados) de `VariableData(type=2)` (Literales de texto) junto con el contexto de llamada (`Debug.Notification` traducible vs. `SendAnimationEvent` técnico bloqueado).
* **`bsa_archive.py`**: Lector de solo lectura para archivos comprimidos `.BSA` versión `104` (`zlib`) y versión `105` (`LZ4` Frame/Block con codec propio en Python + aceleración opcional), con extracción selectiva en memoria sin alterar jamás los `.bsa` originales.
* **`mcm_parser.py` y `aux_parsers.py`**: Soporte estricto para `Interface/Translations/*.txt` (`UTF-16 LE BOM` + `\t`), configuraciones JSON de `MCM Helper` (`config.json` con distinción entre claves `$token` y literales directos), XML estructurados (MCM/FOMOD) e INIs selectivos (`*_DISTR.ini`, subtítulos/localización).

---

## ADR-004: Estrategia de Aplicación en Skyrim sin Modificar Plugins Originales

### Decisión
Establecer una jerarquía de exportación **100 % no invasiva** dentro de un mod independiente de MO2 (`Nolvus AI Spanish Translation`):
1. **Nivel 1 — Dynamic String Distributor (DSD JSON)**: Para todos los plugins `.esp`/`.esm`/`.esl` con textos integrados o sobrescritos, generando archivos en `SKSE/Plugins/DynamicStringDistributor/<PluginName>/` referenciando `FormIDBase|PluginOrigen`. Cero modificación binaria de plugins.
2. **Nivel 2 — Tablas Localizadas (`Strings/*_Spanish.*STRINGS`)**: Para plugins que ya tienen activa la bandera `Localized (0x80)` y buscan tablas `_Spanish` nativamente en `Data/Strings/`.
3. **Nivel 3 — Traducciones MCM / SkyUI (`Interface/Translations/*_SPANISH.txt`)**: Generadas en `UTF-16 LE` con BOM (`FF FE`) y tabulador estricto.
4. **Nivel 4 — Reconstrucción Segura AST de `.pex` (`Scripts/*.pex`)**: Desactivada por defecto salvo activación explícita; cuando se usa, **jamás sobrescribe un índice de `StringTable` compartido con un identificador**: añade una nueva entrada al final de `StringTable` (si `count < 65535`) y redirige exclusivamente el operando `VariableData(type=2)` traducido.

---

## ADR-005: Bloqueo Estricto contra APIs de Pago en Fase 1–7 (`MockTranslationProvider`)

### Decisión
* Definir la interfaz abstracta `TranslationProvider` y un registro de proveedores (`ProviderRegistry`).
* Activar únicamente `MockTranslationProvider`, el cual marca explícitamente cada resultado con metadatos `is_simulated=True` y prefijos verificables `[SIM-ES419]`, respetando tokens protegidos `⟦VAR_i⟧` y términos del Glosario Maestro, además de soportar inyección controlada de fallos para pruebas de reintento, reanudación y rechazo de QA.
* Bloquear por código (`PHASE_1_PAID_API_LOCK = True`) cualquier adaptador externo (`Gemini`, `OpenAI`, `DeepL`), sin solicitar claves API ni realizar peticiones HTTP.

---

## ADR-006: Política Definitiva `ES-419` — Cero Reutilización de Traducciones Comunitarias y Separación de Localización Oficial de Bethesda

### Contexto
En una instalación de Nolvus Awakening existen dos fuentes preexistentes de textos en español:
1. La **localización oficial de Bethesda** (`Skyrim.esm`, `Update.esm`, `Dawnguard.esm`, `HearthFires.esm`, `Dragonborn.esm` y los `74` plugins oficiales de Creation Club `cc*.esm`/`.esl` en `STOCK GAME (Data)`), que suma **57.261 ocurrencias** (`48.029` cadenas únicas).
2. Archivos de **traducción comunitaria de terceros** empaquetados dentro de algunos mods (`69` archivos `*_spanish.txt` de MCM y `5` tablas `_Spanish.*STRINGS` de mods no oficiales), además de `558` archivos MCM en otros idiomas extranjeros (`_chinese.txt`, `_czech.txt`, `_french.txt`, `_german.txt`, etc.).

### Decisión
1. **Exclusión del 100 % de Traducciones Comunitarias**: `src/nolvus_translator/services/project_service.py` (código fuente local no publicado en este repositorio) ignora todos los archivos `*_spanish.txt` y tablas `_Spanish.*STRINGS` de mods de terceros, así como los `558` archivos MCM en idiomas extranjeros, extrayendo exclusivamente el texto original en inglés (`*_english.txt` y registros en inglés) como `translatable` hacia `es-419`.
2. **Separación Estricta de la Localización Oficial de Bethesda**: Las `57.261` ocurrencias oficiales de Bethesda se conservan provisionalmente y se reportan en un indicador dedicado (`official_bethesda_occurrences`), sin mezclarlas automáticamente en la Memoria de Traducción `es-419` ni descontarlas silenciosamente del volumen pendiente de los mods (`unique_pending_net = unique_translatable_total - unique_approved_es419`).

---

## ADR-007: Glosario Maestro de 10 Campos y Separación de Origen en Memoria de Traducción (`schema_version = 1.1.0`)

### Decisión
1. Ampliar `src/nolvus_translator/core/models.py` (código fuente local no publicado en este repositorio) y la tabla `glossary_terms` en `src/nolvus_translator/core/database.py` (código fuente local no publicado en este repositorio) para exigir los **10 atributos auditables**: `source_term`, `approved_translation`, `category`, `definition`, `context_notes` (contexto de lore), `usage_rule`, `translation_rule`, `keep_original`, `justification`, `source_reference` y `approval_status`.
2. Añadir la columna `tm_origin_type` (`'own_es419'` vs `'official_bethesda_reference'`) en `translation_memory` (`src/nolvus_translator/memory/tm_engine.py` (código fuente local no publicado en este repositorio)), rechazando con `ValueError` cualquier intento de importar memorias comunitarias (`'community'`) y asegurando que `lookup_exact(..., only_own_es419=True)` solo aplique automáticamente traducciones propias `es-419` aprobadas.

---

## ADR-008: Resolución Exacta de Prioridades de Archivos `.bsa` (`skyrim.ini` + `loadorder.txt` + `modlist.txt`)

### Decisión
Actualizar `src/nolvus_translator/mo2/vfs.py` (código fuente local no publicado en este repositorio) para ordenar los `583` contenedores `.bsa` siguiendo exactamente la jerarquía del motor `BSResource` de Skyrim:
1. Primero los archivos base listados en `sResourceArchiveList` y `sResourceArchiveList2` de `skyrim.ini` (`18` BSAs).
2. Segundo los archivos `.bsa` emparejados con plugins activos según su `load_order_index` en `loadorder.txt` / `plugins.txt` (`562` BSAs, ordenando `<PluginStem>.bsa` seguido de `<PluginStem> - Textures.bsa`).
3. Tercero `vfs_priority` de `modlist.txt` como desempate para BSAs sin plugin homónimo directo (`3` BSAs), garantizando siempre que cualquier archivo suelto (*loose file*, `18.132` archivos) sobrescribe a cualquier archivo empaquetado en `.bsa` (`57.166` archivos).
