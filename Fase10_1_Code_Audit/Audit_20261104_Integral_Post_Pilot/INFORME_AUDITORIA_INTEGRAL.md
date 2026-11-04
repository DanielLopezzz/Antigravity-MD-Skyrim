# Auditoría integral — NOLVUS AI TRANSLATOR

**Fecha:** 2026-11-04<br>
**Objetivo declarado:** Nolvus Awakening 6.0.21, localización ES-419<br>
**Alcance operativo:** análisis estático, pruebas automatizadas y correcciones locales reversibles. Sin escaneo de una instalación real, solicitudes a proveedores, traducciones facturables ni escrituras en bases de staging o producción.

## Resumen ejecutivo

El proyecto ya tiene una base técnica considerable para inventariar formatos de Skyrim, preparar intercambios offline y controlar futuras solicitudes facturables. Sus decisiones más sólidas son el aislamiento físico de los libros de staging, la reserva previa del presupuesto, la transacción SQLite `BEGIN IMMEDIATE`, la cuarentena de costes desconocidos, la protección de placeholders y un Gatekeeper que valida identidad, hashes, estructura e idempotencia.

Mi valoración independiente es que el sistema está preparado para **staging offline y pilotos pequeños supervisados**, pero aún no para traducir automáticamente las 240.684 cadenas y exportarlas como aprobadas. El obstáculo principal ya no es la construcción de la API: es que la validación lingüística no prueba significado, la información narrativa disponible no llega completa al payload de OpenAI y la memoria de traducción permite agrupar algunos contextos distintos. Además, la capacidad publicada para cientos de miles de cadenas no está demostrada por las pruebas actuales.

No encontré un P0 demostrado en esta revisión. Corregí cuatro riesgos de escritura/integridad y un defecto de lectura binaria. También amplié la protección del glosario para que cualquier cambio a un término aprobado requiera autorización explícita; la GUI conserva los metadatos que no expone en su formulario.

La revisión catalogó y parseó con AST los 65 módulos Python propios, recorrió los subsistemas solicitados y revisó los flujos críticos de escaneo, extracción, traducción, presupuesto, cuarentena, staging y exportación. El AST no detectó errores de sintaxis; detectó 47 funciones de 100 líneas o más y 29 manejadores `except Exception` o equivalentes. El recuento sirve como indicador de concentración de responsabilidades, no como prueba de defecto.

## A. Informe general de auditoría

### Fortalezas

- La arquitectura separa `core`, `mo2`, `formats`, `classification`, `lore`, `memory`, `translation`, `quality`, `export`, `reports`, `updates`, `services` y `ui`.
- El VFS aplica prioridad de archivos sueltos sobre recursos de BSA y evita seguir symlinks al recorrer capas.
- Los parsers extraen varios formatos de Skyrim y los tests cubren fixtures ESP/ESM/ESL, STRINGS, DLSTRINGS, ILSTRINGS, PEX, BSA 104/105, MCM, XML e INI.
- El flujo API valida payloads de traducción, deshabilita herramientas, mantiene `store=False`, redacta credenciales y bloquea fallbacks automáticos a un modelo más caro ante fallos de transporte.
- `BudgetController` reserva antes del envío, serializa reservas con SQLite, rechaza claves de idempotencia duplicadas, concilia uso reportado y detiene trabajos cuando el coste no puede determinarse.
- `ExchangeStateStore` y `LocalGatekeeper` conservan identidad y hashes del lote, detectan duplicados/omisiones y registran cuarentena. El trabajo local previo también conserva la respuesta rechazada para revisión posterior.
- El empaquetador genera un mod independiente para MO2 y guarda manifiesto, backup e historial de exportación.

### Debilidades estructurales

- Las responsabilidades se concentran en `ProjectService.scan_and_extract`, `TranslationPipeline.run_or_resume`, `Mo2ModPackager.build_export_plan`, `LocalGatekeeper.evaluate_output_bytes`, `BudgetController.reconcile_reservation`, `GPT6LunaAdapter.prepare_batch_request/execute_batch` y los scripts de piloto 125. No los refactoricé: hacen trabajo funcional y moverlos ahora elevaría el riesgo de regresión.
- `scan_and_extract` mantiene en memoria los archivos resueltos, cadenas extraídas, clasificaciones y listas de inserción antes de persistir el snapshot en una sola transacción. Puede funcionar bien en los pilotos actuales, pero falta medir RAM y duración con el inventario real de 240.684 cadenas.
- El test de escala cubre 2.500 filas sintéticas, exporta y procesa lotes de 250 y exige menos de 15 segundos; no valida 240.684 filas ni el uso máximo de memoria (`tests/test_phase9_coverage_and_scale.py`).
- La GUI usa un ejecutor de fondo y una cola para devolver resultados al hilo de Tk, lo cual evita el bloqueo directo de la interfaz. No hay una prueba visual prolongada del flujo de escaneo en una instalación real.
- Hay módulos y funciones muy extensos y numerosos manejadores amplios de excepción. En parsers es útil convertir datos corruptos en errores controlados; en servicios grandes puede ocultar fallos de programación si se registran como una extracción fallida y el proceso continúa.

## B. Informe de código

### Cambios implementados en esta auditoría

| Archivo | Problema y cambio | Comportamiento anterior → nuevo | Prueba | Riesgo residual |
|---|---|---|---|---|
| `src/nolvus_translator/core/config.py` | Las rutas configurables de workspace, DB, salida, backups y logs se creaban antes de comprobar si caían dentro de una raíz protegida. Ahora se validan todas antes de crear directorios; también se valida la ruta del archivo de configuración antes de guardarlo. | Podía crear/abrir rutas dentro de STOCK GAME, Nolvus o MO2 si estaban configuradas así → la operación falla antes de crear carpetas o escribir. | `test_workspace_path_under_protected_root_is_rejected_before_mkdir`; `test_config_save_under_protected_root_is_rejected`. | Solo protege las raíces que el usuario configuró. No confina rutas arbitrarias a un único directorio de workspace; conviene añadir una política explícita de rutas permitidas.
| `src/nolvus_translator/lore/glossary.py` | Se evita que un `term_id` ajeno reemplace otro término y que una edición de un término aprobado cambie silenciosamente traducción, categoría, reglas, notas o estado. La comprobación corre dentro de la misma transacción y el `UPDATE` limita la fila por término de origen. | Una colisión podía reemplazar otro término y un metadato podía cambiar sin confirmación → la operación enumera los campos en conflicto y exige autorización explícita; una colisión no altera el registro existente. | `test_glossary_term_id_collision_does_not_overwrite_existing_term`; `test_approved_glossary_metadata_change_requires_explicit_override`; suite de glosario existente. | No se cambió el esquema ni se auditaron/reescribieron filas de producción.
| `src/nolvus_translator/ui/app.py` | Al guardar un término existente, la GUI reconstruía una entrada parcial y podía borrar metadatos no editables. Ahora conserva ID, definición, regla de uso, justificación, referencia y estado existentes. | Guardar desde el formulario podía sustituir campos ocultos por valores vacíos o genéricos → se conservan los campos que el formulario no permite editar; los cambios a campos visibles de términos aprobados requieren confirmación. | Suite de regresión del glosario; inspección del flujo de guardado. | El formulario no precarga automáticamente todos los campos al seleccionar una fila; los cambios intencionales de campos visibles requieren autorización explícita.
| `src/nolvus_translator/export/mo2_packager.py` | El empaquetador comprobaba raíces protegidas, pero no que cada ruta planificada permaneciera dentro de la carpeta del mod. Ahora rechaza rutas absolutas, `..`, ADS de Windows, symlinks/junctions, nombres reservados y colisiones que solo difieran en mayúsculas; valida antes de respaldar o reemplazar la salida anterior. Acepta `./` benigno, usado por el parser de INI. | Un plan corrupto podía escapar de la carpeta objetivo o sobrescribir un archivo del manifiesto → solo se escriben rutas relativas validadas y únicas. | `test_export_paths_reject_escape_reserved_names_and_case_collisions`; prueba integral existente de exportación/rollback. | El reemplazo de una exportación todavía no es atómico ante caída o error de E/S; el backup existe, pero una escritura parcial puede dejar una carpeta incompleta.
| `src/nolvus_translator/formats/strings_table.py` | El parser de tablas localizadas recortaba silenciosamente bloques y entradas DL/ILSTRINGS declarados con más bytes de los disponibles, y convertía offsets fuera del bloque en texto vacío. Ahora rechaza esas estructuras como `BinaryIntegrityError`. | Una tabla truncada podía producir texto incompleto aparentemente válido → se marca como archivo corrupto y no se trata como traducción normal. | `test_string_tables_reject_declared_sizes_beyond_available_bytes`; round-trip válido de las tres tablas. | No se añadieron límites de tamaño máximo a descompresión BSA/ESP; ver recomendaciones.
| `tests/test_audit_safety_regressions.py`, `tests/test_formats.py` | Añadí regresiones para rutas protegidas, colisión de ID, metadatos aprobados, escape del empaquetador y longitudes/offsets inválidos. | No existían pruebas para estos casos → ahora quedan cubiertos localmente. | Suite completa. | No se probaron junctions reales del empaquetador ni fallos de disco durante reemplazo.

No eliminé código ni alteré contratos públicos, modelos o tarifas. No cambié configuraciones por defecto, estados o traducciones aprobados, términos existentes ni bases de datos; el cambio al glosario añade una validación antes del guardado.

### Riesgos de código confirmados o potenciales

1. **Defecto mitigado:** las rutas de salida configurables no se validaban antes de crear carpetas. Corregido arriba.
2. **Defecto mitigado:** el `term_id` de un glosario podía apropiarse de otro registro. Corregido arriba.
3. **Defecto mitigado:** guardar un término aprobado con la misma traducción podía alterar silenciosamente metadatos y reglas. Ahora todos los campos protegidos se comparan y los cambios exigen autorización explícita; la GUI conserva campos que no edita.
4. **Defecto mitigado:** escape de ruta en archivos de salida planificados. Corregido arriba.
5. **Defecto mitigado:** texto truncado aceptado en tablas localizadas. Corregido arriba.
6. **Riesgo pendiente:** `Mo2VfsScanner._fast_file_sha256` devuelve para archivos mayores a 16 MB un hash de tamaño, fecha de modificación y primeros 64 KiB, no el SHA-256 de todo el contenido. `IncrementalUpdateEngine` compara ese valor como si fuera huella del archivo. Una modificación interior preservando tamaño y mtime podría pasar inadvertida; cambiar la semántica requiere versionar huellas para no declarar todos los snapshots anteriores como modificados.
7. **Riesgo pendiente:** `BsaArchiveReader.extract_file` y el parser ESP usan `zlib.decompress` sin límite de salida; el parser ESP lee el tamaño descomprimido declarado pero no lo compara con el resultado. Un archivo local corrupto o preparado para inflar puede consumir mucha memoria o entregar un registro parcial. No fijé un límite arbitrario sin medir archivos reales y decidir cómo reportar entradas legítimas grandes.
8. **Riesgo pendiente:** la ejecución de exportación copia el mod anterior a backup, lo borra y luego escribe en la carpeta final archivo por archivo. El rollback ayuda, pero no evita una salida parcial si el proceso cae a mitad de escritura. Recomiendo preparar y validar en carpeta temporal y hacer un reemplazo controlado.
9. **Riesgo pendiente:** `query_strings_paginated` busca con `LIKE '%texto%'`, que no aprovecha un índice convencional, y usa `OFFSET`, cuyo coste crece en páginas profundas. El docstring afirma menos de 5 ms hasta 500.000 cadenas, pero no encontré una prueba que sustente esa cifra.
10. **Riesgo pendiente:** las migraciones principales agregan columnas y establecen `schema_version=1.1.0`, pero no existe historial versionado de migraciones con mecanismo reversible probado sobre copias. No ejecuté ninguna migración.

## C. Informe lingüístico ES-419

Las reglas base reflejan decisiones acertadas: usar «ustedes», escoger entre tú/usted según relación narrativa, evitar voseo y regionalismos marcados, y priorizar un tono de fantasía. El problema es que `TranslationQualityValidator` no puede demostrar naturalidad ni fidelidad semántica. Su `context_coherence` se basa sobre todo en conservar signos `?` y `!`; la longitud y una lista de formas peninsulares son señales, no evaluación del significado. La lista cerrada tampoco cubre toda variante regional o error gramatical.

El Gatekeeper separa fallos estructurales de algunas infracciones lingüísticas, pero el clasificador de fallos no exige que `recommended_status` sea apto ni rechaza de forma general `REQUIRES_LORE_RESEARCH` o una baja puntuación de familiaridad terminológica. En consecuencia, una respuesta puede superar la validación técnica sin que eso demuestre que la historia, el hablante, el tiempo verbal o el sentido sean correctos. El estado de candidato validado debe seguir separado de aprobación editorial humana.

### Términos que requieren revisión humana y fuente precisa

La semilla contiene **109 términos**, con IDs y términos de origen únicos. Todos se marcan como `approved`, pero la referencia de muchos apunta genéricamente a “Localización oficial Skyrim / Lore UESP / Política ES-419”, sin archivo, registro, edición regional ni decisión editorial por entrada. No cambié ninguna traducción. Estas alternativas son preguntas de revisión, no aprobaciones:

| Entrada actual | Alternativas a contrastar | Motivo |
|---|---|---|
| `Elder Scroll` → `Pergamino Antiguo` | `Pergamino Antiguo` frente a `Pergamino de los Ancianos` | La primera es literal; la segunda puede tratar el nombre del artefacto como entidad propia. Debe prevalecer el uso oficial verificable y el contexto del mod.
| `Magicka` → `Magia` | `Magia` frente a conservar `Magicka` en el atributo/interfaz | La traducción puede confundirse con la magia genérica del mundo; requiere verificar la etiqueta oficial y su uso en prosa.
| `Soul Gem` → `Gema de alma` | `Gema de alma` frente a `Gema del alma` | Diferencia de naturalidad y posible terminología oficial; conviene contrastarla en objetos, interfaz y encantamiento.
| `Soul Cairn` → `Recordatorio de las Almas` | La entrada actual frente a una alternativa literal como `Cementerio de las Almas` | La equivalencia semántica de “Recordatorio” necesita una fuente explícita antes de considerarla canónica.
| `Khajiit` → `Khajiita` | Mantener `Khajiit` como nombre de raza y resolver género según personaje/contexto | El femenino puede ser apropiado para una persona concreta, pero no necesariamente para la raza o un nombre genérico.

`HeelsFix Refresh`, `Skyrim` y `Nocturnal` están presentes como formas que se conservan. `Alternate Perspective` y `JContainers`, mencionados en la especificación, no aparecen en la semilla actual. No los incorporé como aprobados: ampliar la semilla se aplica automáticamente en futuras inicializaciones y debe pasar por decisión terminológica con alcance/contexto.

El matcher de glosario permite en general solo el término exacto más `s` o `es` al final de toda la frase. Esto no modela concordancia plural en cada palabra de una frase («Pergamino Antiguo» → «Pergaminos Antiguos») y puede producir falsos rechazos o aceptar flexiones impropias. La solución durable es registrar variantes permitidas y restricciones por categoría/uso, no relajar la regla con un sufijo general.

## D. Informe del Glosario Maestro

- Términos de semilla revisados estructuralmente: **109**; IDs únicos: **109**; fuentes únicas ignorando mayúsculas: **109**.
- Se revisó su esquema, categoría, reglas de conservación/traducción, metadatos de definición/uso y comportamiento de `ensure_seeded`/`upsert_term`/importación.
- No añadí ni aprobé términos durante esta auditoría. La entrada `HeelsFix Refresh` ya estaba en el árbol local previo a esta auditoría y forma parte del recuento.
- El guardado existente de un término por `source_term` conserva el `term_id` existente. Ahora bloquea cambios a cualquier campo de un término aprobado sin autorización explícita y muestra qué campos difieren. La GUI conserva además los metadatos que no expone en el formulario.
- Pendiente: citar una fuente comprobable por término, separar `draft/proposed/approved`, registrar variantes y género/número, y permitir alcance global o específico por mod sin aplicar reglas contextuales a todo el corpus.

## E. Informe de infraestructura

### Escáner, parsers y exportación

- El escáner resuelve prioridad VFS y orden de plugins; ignora recursos no textuales y conserva procedencia/conflictos. No escaneé la instalación real ni validé la interpretación con un perfil MO2 productivo.
- Los parsers cubren formatos binarios y de texto relevantes mediante fixtures sintéticos. La corrección en `strings_table.py` ahora convierte bloques/entradas truncados en error estructural.
- BSA y ESP aún carecen de un límite de memoria de descompresión; PEX y otros parsers tampoco tienen límites de tamaño de entrada unificados. Se necesitan presupuestos de bytes, validación de longitudes y tests de datos hostiles.
- La exportación crea un mod independiente y conserva backups/manifiestos; las rutas de salida ahora se validan. La actualización no es una operación de intercambio atómica y no se ha probado exhaustivamente con una instalación real de MO2.

### SQLite, datos y escala

- La DB principal de `workspace_data` mide **431.386.624 bytes**. Su tamaño, `mtime_ns` y los metadatos de WAL/SHM coinciden exactamente con los observados antes de modificar código.
- La DB usa WAL, claves únicas por snapshot/UID e índices para filtros, hashes y grupos contextuales. La conexión general habilita claves foráneas; la política `synchronous=NORMAL` privilegia rendimiento y no ofrece la máxima durabilidad ante corte eléctrico.
- El almacenamiento de resultados por snapshot es transaccional. La migración necesita un ledger de versiones con backup, ensayo de ida/vuelta y ejecución solo sobre copias.
- Hay una prueba de 2.500 filas sintéticas, de las cuales 2.000 son únicas; procesa dos lotes de 250 y comprueba menos de 15 s. Eso es evidencia de una ruta pequeña, no de escala completa.
- La búsqueda de texto y la paginación profunda son candidatas a FTS5/keyset pagination. Requieren medir primero patrón de consulta, índices y carga de UI sobre una DB de prueba de 240.684 filas.

### API, costes, presupuesto y staging

- `PHASE_1_PAID_API_LOCK` permanece en `True`. Las pruebas usan transportes mock; la prueba de seguridad exige que el transporte HTTP real registre cero solicitudes cuando el candado está activo.
- El proveedor bloquea llamadas reales antes de enviar, y el coordinador Batch se bloquea bajo la política actual. No ejecuté `GPT-6 Luna`, `GPT-6.1 Sol`, Gemini ni Batch API.
- La reserva presupuestaria utiliza SQLite con `BEGIN IMMEDIATE`, límites por proyecto/trabajo, caps de salida, idempotencia y conciliación. Respuestas incompletas con uso reportado registran el coste real; uso desconocido se pone en cuarentena. La recuperación libera reservas no enviadas y pone en cuarentena las que quedaron en vuelo.
- Los importes se representan con `float` y se redondean a 8 decimales; esto limita la deriva práctica en la configuración actual, aunque unidades enteras o `Decimal` darían una semántica contable más clara. La migración no es necesaria para esta revisión y se debe diseñar aparte.
- Las tarifas están codificadas como constantes sin fecha de vigencia ni URL/captura de la fuente en el código revisado. No las validé en la web ni ante proveedores porque la fase prohíbe solicitudes externas; no deben tomarse como tarifas vigentes sin una verificación futura autorizada.
- Los runners de pilotos y reportes preparan/copían datos aislados. Esta auditoría no lanzó los runners ni importó, promovió o reinició datos de los pilotos.

## F. Registro de cambios

| Archivo | Cambio | Justificación | Comportamiento previo | Comportamiento nuevo | Pruebas | Riesgo |
|---|---|---|---|---|---|---|
| `core/config.py` | Validación de rutas protegidas antes de mkdir/save | Evitar que una ruta de trabajo o el archivo config escriba dentro de raíces configuradas como originales | Creación sin comprobación | Fallo cerrado antes de crear/escribir | 2 regresiones | Solo protege raíces configuradas |
| `lore/glossary.py` | Impedir reutilizar un `term_id` ajeno y exigir confirmación para editar cualquier campo aprobado | Evitar pérdida de términos y cambios silenciosos de reglas/metadatos | Colisión podía reemplazar; solo se protegía la traducción | Colisión rechazada; diferencias de cualquier campo aprobado requieren permiso | 2 regresiones más suite de glosario | Sin migración; no se escribió la DB oficial |
| `ui/app.py` | Conservar metadatos no editables del término al guardar desde GUI | Evitar borrar definiciones/reglas/referencias que el formulario no muestra | La GUI reconstruía una entrada parcial con valores genéricos | Reutiliza los metadatos existentes del término | Suite de regresión de glosario; revisión del flujo | El formulario no precarga valores al seleccionar fila |
| `export/mo2_packager.py` | Validar nombres y rutas objetivo planificadas antes de backup/reemplazo | Impedir escape de ruta, symlink, colisión case-insensitive y sobrescritura de manifiestos | Solo comprobaba raíces protegidas | Solo acepta rutas relativas únicas dentro del mod | 1 regresión más e2e de exportación | No hace el reemplazo final atómico |
| `formats/strings_table.py` | Rechazar bloque/offset/longitud truncados | Evitar traducir texto parcial como si fuera válido | Recortaba bytes o devolvía cadena vacía | Eleva `BinaryIntegrityError` | 1 test con 3 casos inválidos más round-trip | No fija máximos de descompresión |
| `tests/test_audit_safety_regressions.py` | 5 pruebas nuevas | Fijar garantías de rutas y glosario | Sin regresiones explícitas para estos casos | Casos cubiertos | Suite completa | Ninguno fuera de sus límites |
| `tests/test_formats.py` | Casos corruptos de tablas localizadas | Probar lectura fail-closed | Sin estos casos | Tres formas de truncamiento/offset se rechazan | Suite completa | Fixtures sintéticos solamente |

No se borraron archivos. Los cambios previos que ya estaban en el árbol antes de iniciar esta auditoría se conservaron y no se atribuyen a este registro; aparecen en el estado de Git al final.

## G. Informe de pruebas

- Línea base del árbol local previo a esta auditoría: **63 pruebas, OK**. La primera invocación había fallado por permisos del directorio temporal de AppData; al dirigir `TEMP`/`TMP` a `.test_tmp` pasó la línea base.
- Después de los cambios, la primera suite completa encontró un caso válido `./archivo.ini` rechazado por la nueva validación del exportador. Normalicé `.` como segmento inocuo manteniendo el rechazo de `..`, rutas absolutas y colisiones; añadí una prueba positiva para ese formato.
- Pruebas enfocadas posteriores al cambio final de glosario: **8 pruebas, OK** (`test_audit_safety_regressions` y `test_protector_glossary_tm`). Antes de ese último ajuste, las pruebas enfocadas de exportación y parsers también pasaron: **13 pruebas, OK**.
- Suite completa final tras todos los cambios: **69 pruebas, OK** en 11,752 s.
- `git diff --check`: sin errores de whitespace; Git solo avisó sobre la normalización LF/CRLF de Windows.
- Solicitudes reales externas: **0**. El test de candado comprueba contador HTTP real igual a 0; no se consumieron créditos.
- Bases: todas las pruebas usan DB temporales. La DB principal mantuvo `431386624` bytes y `mtime_ns=1793683151811389300`; sus archivos WAL/SHM también conservaron tamaño y mtime. No se abrió una conexión de escritura a la DB principal ni se ejecutó una migración.

## H. Recomendaciones de arquitectura y mejoras pendientes

### P1 — No convertir una validación mecánica en aprobación editorial

- **Problema/evidencia:** las métricas de contexto son longitud y puntuación; el Gatekeeper no impone en general la recomendación `REQUIRES_LORE_RESEARCH` o una familiaridad terminológica insuficiente.
- **Solución:** separar `estructuralmente válido`, `terminología verificada`, `revisión humana aprobada` y `exportable`. La salida del modelo debería quedar pendiente hasta aprobación humana en contenido narrativo, nombres propios y textos con incertidumbre.
- **Alternativa:** revisión humana completa de todas las cadenas desde el primer lote; más coste editorial pero menor riesgo de aprobación automática.
- **Beneficio/riesgo:** reduce errores semánticos que pasan controles de placeholders; aumenta el tiempo de revisión y exige UI/estados adicionales.
- **Esfuerzo/confianza:** 3–5 días iniciales, luego calibración; confianza alta.
- **Pruebas:** corpus gold de 50–100 cadenas por dominio, medición de falsos aceptados/rechazados, y test de que ninguna cadena sin aprobación humana llega al exportable.

### P1 — Llevar contexto útil al contrato de traducción

- **Problema/evidencia:** `ContextEngine` agrupa filas y construye un resumen, pero el payload de `openai_provider` se arma desde `ExchangeInputRecord`, que solo lleva dominio, tipo de registro/campo, texto enmascarado, glosario y placeholders. No envía mod, plugin, misión, tópico, hablante ni textos hermanos. `to_prompt_summary()` tampoco aparece conectado a la solicitud API.
- **Solución:** diseñar `EXCHANGE_SCHEMA_VERSION` siguiente con contexto opcional y acotado, separando identificadores narrativos de rutas locales. Usar resumen por lote cuando aporte continuidad y metadatos por cadena cuando cambie el sentido; medir su coste en tokens.
- **Alternativa:** mantener contexto corto por lote y pasar datos adicionales solo a las categorías marcadas como ambiguas.
- **Beneficio/riesgo:** mejora interpretación de diálogo, misiones y nombres; cambia el contrato de JSONL y requiere soporte de versiones antiguas.
- **Esfuerzo/confianza:** 1–2 semanas incluyendo compatibilidad; confianza alta.
- **Pruebas:** fixtures de versión anterior/nueva, control de privacidad, coste de tokens y prueba A/B ciega con traductores.

### P1 — Versionar identidad de TM antes de ampliar su reutilización

- **Problema/evidencia:** `compute_source_hash` usa `text.strip()` y `compute_context_signature` solo considera dominio/tipo/subtipo/hablante. Ignora mod/plugin/quest/topic/context group/call context; `UNIQUE(source_hash, context_signature)` puede actualizar una traducción que representa otro uso.
- **Solución:** definir `source_hash_v2` sobre texto exacto y contexto versionado con los campos pertinentes. Leer entradas v1 como candidatas de baja confianza, nunca migrarlas o aplicarlas automáticamente si el nuevo contexto no es inequívoco.
- **Alternativa:** no cambiar clave pero permitir reutilización solo con coincidencia exacta del texto y más campos comparados en runtime.
- **Beneficio/riesgo:** reduce propagación de una traducción contextual a otro mod/diálogo; puede reducir cobertura de TM y necesita migración/dual-read.
- **Esfuerzo/confianza:** 3–6 días y revisión de datos; confianza alta.
- **Pruebas:** cadenas iguales con contextos distintos, diferencias de whitespace, coexistencia v1/v2, historial e idempotencia sobre una copia.

### P1 — Limitar consumo de memoria de parsers

- **Problema/evidencia:** BSA y ESP descomprimen sin máximo de salida; el ESP no compara el tamaño resultante con el tamaño declarado. Las tablas localizadas ahora sí rechazan truncamiento, pero no existe una política global de bytes por archivo.
- **Solución:** límites configurables por familia de formato, streaming/bounded decompression cuando exista, comparación de longitudes y estado `EXTRACTION_FAILED` con razón explícita.
- **Alternativa:** dejar que el proceso del SO limite RAM; no es control suficiente ante un mod malformado.
- **Beneficio/riesgo:** evita inflado de memoria y pérdida silenciosa; un umbral bajo podría rechazar un archivo legítimo grande.
- **Esfuerzo/confianza:** 2–4 días; confianza media-alta hasta medir corpus.
- **Pruebas:** fixtures con tamaño declarado falso, salida comprimida por encima del límite, truncamiento, archivo legítimo grande y memoria máxima observada.

### P1 — Demostrar escala objetivo sin tocar datos oficiales

- **Problema/evidencia:** 2.500 filas sintéticas no validan 240.684 cadenas, cientos de miles de archivos, uso de memoria ni tiempo de índices/GUI.
- **Solución:** benchmark sintético reproducible en DB temporal con 240.684 strings y distribución por formato/mod; registrar RAM pico, tiempo de escaneo/extracción, páginas UI, consultas, batch preparation y cancelación/reanudación.
- **Alternativa:** benchmark parcial de 25k/50k por etapa; más rápido pero no decide readiness total.
- **Beneficio/riesgo:** permite dimensionar lote y almacenamiento con evidencia; los sintéticos no reproducen toda la distribución real.
- **Esfuerzo/confianza:** 2–3 días; confianza alta en el diagnóstico.
- **Pruebas:** pasos repetibles, baseline por etapa y assertions de memoria/tiempo en agente Windows comparable al equipo objetivo.

### P2 — Aclarar huellas e integridad de archivos

- **Problema/evidencia:** `_fast_file_sha256` no genera el SHA-256 completo para archivos grandes, aunque el campo se llama `sha256`.
- **Solución:** versionar el tipo de fingerprint (rápido vs criptográfico), usar SHA-256 real donde se prometa integridad y conservar fingerprints anteriores como históricos.
- **Alternativa:** calcular hash completo solo para archivos de texto que se parsean y usar fingerprint etiquetado para BSA no inspeccionadas.
- **Beneficio/riesgo:** evita falsas garantías y mejora detección incremental; el hash completo aumenta el tiempo de escaneo de BSA/ESM grandes.
- **Esfuerzo/confianza:** 1–3 días; confianza alta.
- **Pruebas:** modificar bytes interiores manteniendo tamaño/mtime en fixture y validar el comportamiento de diff.

### P2 — Mejorar consultas de corpus y reemplazo de exportación

- **Problema/evidencia:** `LIKE '%...%'` no aprovecha índices normales, `OFFSET` escala peor en páginas profundas y el exportador escribe directamente en la carpeta final luego de retirar la previa.
- **Solución:** primero medir; luego considerar FTS5 para texto y paginación por cursor. Para exportación, crear staging local, verificar manifiesto/hashes, mover la salida anterior a backup y publicar el directorio completo con reemplazo de Windows probado.
- **Alternativa:** conservar la arquitectura actual con límites de páginas y exportación bajo supervisión; menos cambios ahora.
- **Beneficio/riesgo:** búsqueda rápida y salida consistente ante interrupciones; requiere pruebas Windows de renombrado/locks y posiblemente índices/migración.
- **Esfuerzo/confianza:** 1–2 semanas; confianza media.
- **Pruebas:** corpus 240k, planes `EXPLAIN QUERY PLAN`, exportación con inyección de fallo y rollback/backup.

### P2 — Cerrar la política de glosario con evidencia

- **Problema/evidencia:** 109 términos se marcan aprobados con referencias genéricas, faltan dos nombres de ejemplo y la forma plural está limitada a sufijo final.
- **Solución:** revisión editorial por término con fuente exacta y alcance, variantes permitidas/prohibidas, género/número y regla contextual. Mantener propuestas separadas de aprobaciones y sincronizar con staging/candidatos solo mediante operación controlada.
- **Alternativa:** mantener semilla conservadora y remitir los faltantes a revisión humana por lote.
- **Beneficio/riesgo:** coherencia entre mods y menos falsos rechazos; trabajo editorial y futura migración si se añaden campos.
- **Esfuerzo/confianza:** 1–2 semanas para las entradas principales, continuo para corpus; confianza alta.
- **Pruebas:** términos en diálogos, interfaces, libros, plurales y nombres de mod; no sobrescribir traducciones aprobadas.

### P3 — Simplificar módulos de alta concentración y completar trazabilidad de tarifas

- **Problema/evidencia:** múltiples funciones superan 200 líneas; tarifas/modelos se guardan como constantes sin fecha ni fuente auditable.
- **Solución:** después de estabilizar contratos, extraer funciones puras pequeñas de preparación/serialización/reconciliación y documentar cada tarifa con fecha, origen y hash de la captura local validada.
- **Alternativa:** no refactorizar mientras el conjunto de tests y contratos siga creciendo.
- **Beneficio/riesgo:** facilita mantenimiento y auditoría económica; puede añadir capas y errores de integración si se hace antes de mejorar pruebas.
- **Esfuerzo/confianza:** refactor 1–2 semanas por áreas; confianza media.
- **Pruebas:** pruebas unitarias de contratos, equivalencia de fixtures y comparación exacta de payloads/ledger.

## I. Estado final

- **Rama raíz:** `feat/phase10-2-openai-api-first`.
- **Commit inicial/final:** `066ae03b014f0aab1bbbcdbb4db16658e45b0327` / el mismo; no se creó commit. No abrí rama porque el árbol ya tenía cambios sin confirmar.
- **Archivos de código cambiados por esta auditoría:** `core/config.py`, `lore/glossary.py`, `ui/app.py`, `export/mo2_packager.py`, `formats/strings_table.py`.
- **Pruebas cambiadas/agregadas por esta auditoría:** `tests/test_formats.py` y nuevo `tests/test_audit_safety_regressions.py`.
- **Archivos eliminados:** ninguno.
- **Archivos/documentos agregados:** este informe y el inventario de módulos. Se dejaron intactos los archivos locales modificados/no rastreados que ya existían.
- **Estado Git:** sucio al inicio y final; contiene trabajo previo, documentos y pilotos locales además de estos cambios. Los temporales de prueba permanecen en `.test_tmp`; no eliminé ni limpié esa carpeta. No se ejecutó `reset`, `clean`, commit ni push.
- **Candado API:** `PHASE_1_PAID_API_LOCK=True`.
- **Coste/solicitudes externas:** cero llamadas; cero créditos consumidos.
- **DB principal:** 431.386.624 bytes; el tamaño, `mtime_ns` y metadatos de WAL/SHM permanecen idénticos a la observación inicial de la auditoría.
- **DB de staging:** no se ejecutaron runners contra `NolvusAIT_PilotStaging`; las pruebas nuevas usan carpetas temporales. No se reinició ni promovió ningún resultado.
- **Resultados previos:** no se modificaron 100 cadenas importadas, 23 candidatas retenidas, 2 en cuarentena ni la política de cero promociones indicada en el estado publicado de Fase 10.3-C.
- **Repositorio de documentación anidado:** permanece local en `main`, commit `d7260e32c16f6b4c33e1bd53cd644e0e973a3bef`; no hubo publicación durante esta auditoría.
- **Próximo paso recomendado:** implementar primero el estado de aprobación humana y el contexto narrativo versionado; después corregir la identidad de TM y medir parsers/consultas con un benchmark de 240.684 filas. Mantener la fase facturable detenida hasta validar tarifas, contenido y el candado mediante una autorización explícita futura.
