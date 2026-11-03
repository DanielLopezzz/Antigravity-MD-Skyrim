# Plan de Desarrollo por Hitos Verificables (ROADMAP)

El desarrollo de **NOLVUS AI TRANSLATOR** se organiza en 9 fases secuenciales con entregables verificables y criterios estrictos de seguridad.

---

## Estado Actual de los Hitos

| Fase | Nombre del Hito | Estado | Verificación |
| :---: | :--- | :---: | :--- |
| **FASE 0** | Investigación, Licencias, Compatibilidad `1.7.104.0` y Arquitectura | **COMPLETADA** | Matriz comparativa, auditoría copyleft y ADRs documentados. |
| **FASE 1** | Infraestructura Base, SQLite WAL/FTS5, Logs y Aplicación GUI | **COMPLETADA** | Repositorio Git, DB SQLite, configuración y GUI de 10 vistas operativa. |
| **FASE 2** | Integración con Nolvus, PE Inspector y VFS de MO2 | **COMPLETADA** | Lector PE `SkyrimSE.exe`, `ModOrganizer.ini`, `modlist.txt` inverso y `plugins.txt`. |
| **FASE 3** | Extracción Binaria Multi-Formato y Clasificador | **COMPLETADA** | Parsers `ESP/ESM/ESL`, `*STRINGS`, `PEX` AST, `BSA` v104/v105, `MCM` y pruebas. |
| **FASE 4** | Lore de Skyrim, Glosario Maestro, Contexto y Memoria (TM) | **COMPLETADA** | Base de lore TES, protección de nombres propios, agrupación y TM `RapidFuzz`. |
| **FASE 5** | Pipeline de Traducción Simulado (`MockTranslationProvider`) y QA | **COMPLETADA** | Protección `⟦VAR_i⟧`, lotes, reintentos, reanudación, rúbrica QA y revisión. |
| **FASE 6** | Exportación No Invasiva, Mod Independiente MO2 e Incrementalidad | **COMPLETADA** | Exportador DSD/`*STRINGS`/MCM/PEX, `meta.ini`, manifiesto SHA-256, diff y rollback. |
| **FASE 7** | Validación Global y Auditoría Técnica sobre Nolvus Awakening 6.0.21 | **COMPLETADA Y AUDITADA** | Escaneo real de 3.702 capas VFS, 75.298 archivos, 38.406 `.pex` en BSA auditados, política `ES-419` aplicada (`403.922` cadenas depuradas; `0` traducciones comunitarias; `57.261` oficiales Bethesda separadas). |
| **FASE 8** | Evaluación Económica Real y Comparativa Vigente de Proveedores | **COMPLETADA** | Volumen único neto auditado (`235.326` cadenas únicas / `20,06 M` caracteres / `13,09 M` tokens input / `9,52 M` tokens output) y 4 proveedores / 3 escenarios documentados en `API_COST_ESTIMATE.md`. |
| **FASE 9** | Ejecución de Localización `ES-419` por Lotes y Piloto en MO2 | **PENDIENTE DE AUTORIZACIÓN** | Esperando decisión del usuario sobre proveedor/escenario y alcance del piloto antes de activar conectores o instalar en MO2. |

---

## Detalle de las Próximas Fases con Intervención del Usuario

### FASE 7 — Validación Global (Escaneo Real de Solo Lectura)
1. Configurar en la pestaña **Configuración** la ruta real de la instancia `Nolvus Awakening` (`STOCK GAME` y carpeta `MO2`) del usuario.
2. Verificar automáticamente la versión binaria exacta de `STOCK GAME\SkyrimSE.exe` (confirmando si es `1.7.104.0`, `1.6.1170.0` o `1.5.97.0`), `SKSE64`, `Address Library` y `DynamicStringDistributor.dll`.
3. Ejecutar el escaneo completo en modo **estricto de solo lectura**, generando el inventario real por mod, formato y dominio narrativo, e identificando traducciones españolas ya existentes en el juego base/mods.
4. Probar un paquete piloto generado con `MockTranslationProvider` (o con memoria oficial) exclusivamente sobre un **perfil clonado de prueba en MO2**.

### FASE 8 — Evaluación Económica (Cero Gasto sin Autorización)
1. Ejecutar el módulo integrado `CostEstimator` sobre el volumen único neto pendiente del inventario real de la Fase 7.
2. Consultar tarifas oficiales vigentes en la fecha exacta de la Fase 8 (Gemini API Standard vs. Batch, OpenAI API Standard vs. Batch, DeepL API Free/Pro, modelos locales abiertos en RX 9070 XT 16 GB VRAM).
3. Presentar los 3 escenarios económicos (*Más económico*, *Equilibrado*, *Mayor calidad*) con márgenes de incertidumbre y esperar autorización expresa del usuario antes de implementar o activar cualquier llamada externa.
