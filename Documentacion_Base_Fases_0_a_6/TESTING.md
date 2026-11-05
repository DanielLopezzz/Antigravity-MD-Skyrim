> **TESTING — EVIDENCIA DE FASES INICIALES:** Este plan describe fixtures y pruebas de la etapa inicial. Sus comandos, nombres de tests y resultados deben contrastarse con el repositorio de código vigente; este repositorio documental no incluye el código fuente ni permite reejecutar la suite. No asumir que un resultado histórico equivale a una ejecución reciente.

---

# ESTRATEGIA Y GUÍA DE PRUEBAS AUTOMATIZADAS (`TESTING.md`)

---

## 1. Regla de Oro de Seguridad en Pruebas

**Jamás se ejecutan pruebas destructivas ni de escritura sobre la instalación real de Nolvus Awakening ni sobre `STOCK GAME`.**

Todas las pruebas automatizadas generan en directorios temporales aislados (`tempfile.TemporaryDirectory`) sus propios binarios sintéticos con cabeceras y estructuras reales de Bethesda:
- Ejecutables PE con recurso `VS_FIXEDFILEINFO` (`1.7.104.0`).
- Plugins `.esp` y `.esl` con `TES4`, maestros `MAST`, flag `0x80` (localizado), flag `0x200` (ESL), registros comprimidos con `zlib` (`0x00040000`), grupos jerárquicos `GRUP` Tipo 7 (`DIAL -> INFO`), `PERK` (`EPFT=1` float vs `EPFT=7` string) y `GMST` (`s*` vs `f*`).
- Tablas `.STRINGS`, `.DLSTRINGS` e `.ILSTRINGS` en binario UTF-8.
- Archivos comprimidos `.BSA` versión `104` (`zlib`) y versión `105` (`LZ4 Frame` con bloque sin comprimir prefijado).
- Scripts compilados `.PEX` Big-Endian (`0xFA57C0DE`) con tabla de cadenas compartida entre identificadores e instrucciones `CALLSTATIC`.
- Archivos MCM `UTF-16 LE BOM` (`FF FE`) separados por tabulador (`\t`), `MCM Helper` `.json`, `.xml` e `*_DISTR.ini`.

---

## 2. Cómo Ejecutar la Suite de Pruebas

Desde la raíz del repositorio (`c:\Skyrim proyecto traducción`), ejecuta en PowerShell:

```powershell
.venv\Scripts\python.exe -m unittest discover -s tests -t . -v
```

Resultado esperado: **11 pruebas ejecutadas (`OK`) en ~1.6 segundos, 0 fallos, 0 llamadas de red.**

---

## 3. Desglose de Módulos de Prueba

### 3.1 `tests/test_formats.py` (archivo del workspace local) — Parsers y Reconstructores Binarios Bethesda
1. **`test_strings_dlstrings_ilstrings_roundtrip_utf8`**:
   - Verifica lectura y escritura binaria exacta de `.STRINGS` (terminado en `\x00`) y `.DLSTRINGS`/`.ILSTRINGS` (prefijado con longitud `uint32` + `\x00`) con caracteres en español (`á, é, í, ó, ú, ñ, ¡, ¿`).
2. **`test_localized_esl_plugin_with_string_tables`**:
   - Verifica resolución de IDs `uint32` contra tablas `_English` y `_Spanish`, detección de flag `0x200` (ESL) y clasificación automática `ALREADY_TRANSLATED` cuando existe traducción en `_Spanish.STRINGS`.
3. **`test_esp_inline_compressed_gmst_perk_and_dial_grup`**:
   - Verifica descompresión `zlib` (`0x00040000`) de registros `QUST`, asociación jerárquica `DIAL -> INFO` en `GRUP` Tipo 7, filtrado de `GMST` numéricos (`f*`, `i*`) frente a cadenas (`s*`), y filtrado de `PERK EPFD` cuando `EPFT != 7`.
4. **`test_pex_ast_extraction_and_safe_rebuild_preserving_shared_identifiers`**:
   - Construye un `.pex` Big-Endian donde el índice `StringTable[3]` (`"Health"`) se usa **simultáneamente** como identificador de nombre de variable y como valor literal inicial, además de una llamada técnica `Debug.SendAnimationEvent("IdleForceDefaultState")` y una llamada visible `Debug.Notification(...)`.
   - Comprueba que `SendAnimationEvent` se clasifica como `NON_TRANSLATABLE` (código técnico).
   - Reconstruye el `.pex` aplicando traducciones y verifica que `StringTable[3]` conserva intacto `"Health"` para el identificador, añadiendo la traducción al final de `StringTable` sin alterar `Object.size` ni un solo byte de código máquina.
5. **`test_bsa_v104_zlib_and_v105_lz4_readonly_extraction`**:
   - Verifica extracción en memoria de solo lectura tanto para `BSA v104` (`zlib`) como para `BSA v105` (`LZ4 Frame` nativo de Skyrim SE/AE).
6. **`test_mcm_utf16le_json_xml_and_ini_parsers`**:
   - Verifica codificación estricta `UTF-16 LE BOM` (`FF FE`) + tabulador `\t`, `MCM Helper` JSON (`config.json`), XML selectivo (`text` traducible vs `source` técnico) e INIs de `Description Framework` (`*_DISTR.ini`).

### 3.2 `tests/test_protector_glossary_tm.py` (archivo del workspace local) — Protección de Variables, QA, Glosario y Memoria TM
7. **`test_variable_protector_preserves_and_rejects_mutations`**:
   - Verifica enmascaramiento determinista `⟦VAR_i⟧` de `<Alias=Player>`, `<mag>`, `<dur>`, `<p align='left'>`, `<font size='18'>`, `[pagebreak]`, `%.1f%%`, `{0}` y `[0x00018A56]`.
   - Comprueba rechazo automático (`is_valid == False`) si falta un placeholder o si se duplica uno.
8. **`test_quality_validator_detects_glossary_violation_and_peninsular_spanish`**:
   - Verifica que una traducción correcta en `es-419` respetando glosario (`Delphine` intacto, `Whiterun -> Carrera Blanca`) y variables alcanza `ReviewStatus.APPROVED`.
   - Verifica que el uso de formas peninsulares (`vosotros`, `hablad`) o la alteración de un nombre propio protegido (`Delphine -> Delfina`) degrada automáticamente la entrada a `ReviewStatus.DUBIOUS`.
   - Verifica que oraciones narrativas con `>= 2` nombres propios no catalogados se enrutan a `ReviewStatus.REQUIRES_LORE_RESEARCH`.
9. **`test_glossary_conflict_protection_and_tm_versioning`**:
   - Verifica que la semilla de lore carga los términos canónicos de Skyrim.
   - Comprueba que intentar sobrescribir automáticamente un término aprobado con una traducción distinta lanza `GlossaryConflictError`.
   - Verifica coincidencia exacta en Memoria de Traducción, registro de historial de versiones (`tm_versions`), detección de inconsistencias contextuales (`Fire!` en `SPEL` vs `INFO`) y búsqueda difusa con `RapidFuzz`.

### 3.3 `tests/test_end_to_end.py` (archivo del workspace local) — Flujo Integral E2E, MO2 VFS, Reanudación, Actualizaciones y GUI
10. **`test_full_mo2_scan_pipeline_resume_incremental_export_and_rollback`**:
    - Verifica bloqueo `ExternalAPIDisabledError` ante cualquier intento de seleccionar un proveedor externo.
    - Verifica inspección PE de `SkyrimSE.exe 1.7.104.0` y alertas de compatibilidad de `DynamicStringDistributor.dll` + `versionlib-1-7-104-0.bin`.
    - Verifica resolución de prioridades de `modlist.txt` (orden inverso) y `plugins.txt` (el parche `02_Nolvus_Awakening_Conflict_Patch` sobrescribe `01_Awakening_Quest_Expansion` conservando la referencia al maestro original `00006000|AwakeningQuest.esp`).
    - Simula un fallo transitorio de red/motor en el primer lote (`fail_first_n_calls=1`), verifica reintento automático, pausa el trabajo tras 2 lotes (`stop_after_batches=2`) y **reanuda exactamente el mismo `job_id`** desde el checkpoint en SQLite hasta completar el 100 % de los lotes.
    - Verifica que intentar exportar dentro de `STOCK GAME` lanza `ReadOnlyViolationError`.
    - Exporta el mod independiente `Nolvus AI Spanish Translation` y verifica físicamente los archivos DSD JSON, `AwakeningQuest_SPANISH.txt` (`UTF-16 LE BOM`), `Skyrim_Spanish.STRINGS`, `NolvusQuestHelperScript.pex`, `meta.ini` y `translation_manifest.json`.
    - Simula la actualización de un mod en Nolvus (1 cadena modificada + 1 cadena nueva), ejecuta `compare_snapshots`, verifica propagación automática de traducciones válidas, ejecuta segunda exportación con backup automático y prueba el **Rollback completo**.
    - Verifica informe de cobertura real (contemplando `.swf` no soportado -> cobertura honesta `< 100 %`), exportación Excel `.xlsx` y cálculo de volumen de tokens para las 2 únicas cadenas pendientes tras la actualización.
11. **`test_desktop_gui_all_10_tabs_instantiate_cleanly`**:
    - Instancia `NolvusTranslatorApp` y verifica que las 10 pestañas obligatorias se construyen y refrescan sin errores.
