# Reglas y referencia técnica

Corte documental **2026-11-06**. Resume lo vigente sin sustituir los contratos del código. [Estado](ESTADO_ACTUAL.md) · [Uso](GUIA_DE_USO.md) · [Histórico y originales](HISTORICO.md).

## Reglas vigentes

**Depuración verificada:** caché leída por lotes de 400 claves y una sola instantánea SQLite, con recibos verificados y memoria de decodificación acotada. Hash canónico compartido con el evaluador legado, conservando importaciones y bytes; costes de Plan B en `mass_costs` sin ciclo proveedor/motor. El muestreo usa deque y conjuntos manteniendo el mismo orden. La posedición solo sustituye la candidata efectiva si fue recomprobada y pasa QA; el original permanece en caché. El importador compartido recibe `is_simulated=False` por defecto compatible y lo persiste en la misma transacción. Los ejecutores offline pasan `True` explícitamente. Se retiró la dependencia sin uso `charset-normalizer`; las validaciones de cada frontera y lectores históricos siguen activos.

**Modo Plan B:** `MassWorkflow` integra un `Dispatcher` y `ProjectLedger` para todas las cohortes/modelos/etapas. Usa saldos transaccionales y reconciliación al reabrir, intención antes de HTTP, recibos originales antes de aceptación, reservas inciertas sin reenvío, caché separada y TM aprobada inmutable. El contrato compacto existente conserva identidades fuera de la respuesta del modelo. El perfil explícito `mass-v1` admite Terra sin cambiar el router legado; mantiene las restricciones de herramientas, retención y credenciales.

Las muestras independientes preceden a todas las escaladas. El presupuesto determina capacidad, sin porcentaje fijo del corpus; falta cobertura se informa. Informativas no generan por sí solas revisión de riesgo. El backend pagado exige permiso por manifiesto/libro/caps/modelos/caducidad; no cambia el API Lock ni hereda autorizaciones históricas. Batch tiene ciclo simulado y reservas por línea, sin devolución automática por cancelación. La salida provisional tiene estado distinto de aprobación humana y solo cobertura DSD probada con fixtures. [Contratos, cobertura y limitaciones de Plan B](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/4060d9f3581df44732228c7cb3eb46ae8646ad3b/Fase10_Evolucion_Linguistica/PlanB_20261106/README.md).

1. **Localización propia ES-419:** no incorporar traducciones comunitarias descargadas a la TM o al glosario. La localización oficial preexistente se mantiene separada; no se reescribe masivamente el juego base.
2. **Intención y voz:** conservar historia, emoción, trato y registro. Usar ustedes; elegir tú/usted según contexto y relación. Evitar regionalismos marcados sin forzar literalidad ni inventar género o lore ausentes.
3. **Glosario:** respetar preferencias y nombres aprobados, proteger todos sus metadatos y conservar la procedencia. Las variantes son concordancias explícitas, no sinónimos libres ni nuevas aprobaciones. La revisión de 109 entradas no equivale a verificar documentalmente cada definición heredada.
4. **Integridad:** preservar variables, etiquetas, identificadores, porcentajes técnicos y controles reales/escapados. No hacer reemplazos binarios ciegos ni modificar originales de Nolvus, MO2, STOCK GAME o partidas.
5. **Separación editorial:** candidata, aceptación mecánica, propuesta reparada, referencia humana y texto exportable son evidencias distintas. Gatekeeper no aprueba significado ni alimenta automáticamente la TM con traducciones reales nuevas.
6. **Ejecución acotada:** candado de API activo por defecto, autorización explícita, presupuesto reservado antes de enviar y conciliación posterior. Una credencial disponible no autoriza gasto. No hay fallback automático a un modelo más caro ante errores de red.
7. **Operación reversible:** staging aislado, evidencia original, importación transaccional del grupo, vista previa y respaldos antes de una exportación futura. No reenviar solicitudes inciertas ni repetir candidatas válidas por defecto.

Estas reglas consolidan decisiones existentes. La autorización posterior cubrió únicamente la prueba pareada con cap US$0,35. Está cerrada por parada crítica; producción y aprobación automática nueva siguen excluidas.

## Recorrido y componentes

Detección de entorno/VFS → extracción y clasificación → contexto, glosario y protección → preparación y presupuesto → respuesta/evidencia → Gatekeeper → staging → revisión editorial → eventual exportación autorizada.

| Área | Módulos locales relevantes | Límite práctico |
|---|---|---|
| Entorno y archivos ganadores | `mo2/`, parsers `formats/` | Un formato inventariado no equivale a texto traducible |
| Protección y QA | `quality/protector.py`, `validator.py`, `gatekeeper.py` | La puntuación mecánica no mide fidelidad; “context_coherence” solo compara signos finales |
| Contexto y terminología | `lore/context_engine.py`, `editorial_policy.py`, `glossary.py` | Metadatos acotados; el piloto no envía una historia completa ni hablante identificado |
| TM | `memory/tm_engine.py` | Identidad contextual v2; coincidencias v1 o entre contextos son sugerencias, no equivalencia segura automática |
| API e intercambio | `translation/openai_provider.py`, `exchange_*`, `budget_control.py`, `model_router.py` | El proveedor solo traduce; no recibe acceso al sistema, herramientas o credenciales del workspace |
| Piloto y recuperación | `translation/pilot_four*.py`, `pilot_recovery.py` | Recuperación focalizada y cuarentena; requiere conciliación clara del estado |
| Revisión y exportación | `ui/case_review.py`, `export/` | Referencias del visor separadas de TM/DB; producción actual no autorizada |

Los identificadores configurados en código son `gpt-6-luna`, `gpt-6.1-sol` y la ruta complementaria de intercambio `gemini-4-argon`; Claude está excluido por la política existente. Esta lista describe configuración observada, no disponibilidad, tarifas o recomendación comercial verificadas hoy. No se consultaron proveedores para reorganizar documentos.

## Formatos y alcance

| Familia | Recorrido existente | Restricción |
|---|---|---|
| ESP/ESM/ESL, ESPFE | Parser estructural, registros comprimidos, salida DSD/tablas según caso | No sobrescribir plugins originales |
| STRINGS/DLSTRINGS/ILSTRINGS | Lectura y reconstrucción tipada | Respetar IDs, longitudes y codificación |
| PEX | AST y reconstrucción selectiva de literales | Filtro de llamadas y protección de identificadores; elegibilidad global aún requiere auditoría |
| PSC | Referencia contextual | No es sustituto de un parche PEX validado |
| MCM TXT / MCM Helper JSON | Claves y texto visible; TXT de salida UTF-16LE BOM | Conservar claves y estructura |
| XML, INI, JSON narrativo | Selección de campos/rutas permitidos y reconstrucción específica | No traducir indiscriminadamente archivos de configuración |
| BSA | Inventario y lectura/extracción selectiva | No reescribir el contenedor; salidas sueltas en un mod separado |
| SWF, DLL, audio y texturas | Inventario o inspección según formato | Fuera de la traducción textual automática; subtítulos se tratan aparte |

Casos binarios importantes: `PERK EPFD` solo es texto con `EPFT=7`; `GMST DATA` solo con EDID de tipo cadena (`s*`). El PEX comparte índices de identificadores y literales: la reconstrucción debe preservar los primeros. Las particularidades completas permanecen en los documentos originales archivados y el código, sin afirmar cobertura del 100 % de Nolvus.

## Compatibilidad, dependencias y evidencia

- El informe de Fase 9 registró Nolvus Awakening 6.0.21 Ultimate, MO2 2.4.4 y STOCK GAME 1.5.97.0. Son datos de aquel escaneo, no una inspección nueva. Compatibilidad DSD/SKSE/Address Library debe comprobarse contra el ejecutable efectivo antes de exportar.
- `pyproject.toml`: proyecto 0.6.0, Python >=3.11; dependencias declaradas Pydantic, RapidFuzz, lxml, charset-normalizer y XlsxWriter. No se fijan aquí versiones instaladas tomadas de auditorías antiguas.
- La política mantiene parsers propios y evita copiar código de terceros. Los análisis antiguos de licencias quedan archivados como referencias fechadas; esta reorganización no revalida sus conclusiones legales ni las condiciones actuales de terceros. Conservar avisos aplicables antes de distribuir.
- La suite funcional vigente está registrada en el estado actual. Los benchmarks de 240.684 filas son sintéticos y de SQLite; no acreditan tiempos/memoria de todos los parsers, UI, exportación o traducción del inventario real.

## Evolución offline implementada y límites

`quality/intelligent_review.py` añade severidades, cifras/ordinales, propuestas de controles terminales y gate de calibración. `lore/mod_knowledge.py` mantiene conocimiento y propuestas por mod/plugin en un sidecar, con procedencia y permisos. `translation/durable_run.py` prueba recibos, conciliación y reanudación sin reenvío; `execute_authorized_stages` coordina etapas dentro de un alcance fijo. El flujo `independent_review` / `review_workflow` / `review_experiment` conecta evaluación, posedición y recomprobación offline, con solicitudes exactas previas al modelo, recibos y reserva acotada. Una respuesta malformada no provoca reenvío ni detiene los demás casos; un transporte incierto requiere conciliación. El transporte real se ejercitó después con ocho controles sintéticos; la revisión ciega de traducciones naturales no se alcanzó y su calidad sigue sin calibrar.

Contrato de entrada 1.2, lectura compatible 1.0/1.1: hasta cuatro fragmentos de 320 caracteres, sin rutas privadas y con hash de procedencia, datos no confiables. La preparación puede sellar un snapshot contextual; la agrupación coherente es opcional y su coste debe caber en el presupuesto. No se reabren planes históricos con fingerprints modificados. Las entidades NPC son nombres de registro, no nombres propios intraducibles por defecto. El contexto específico no sobrescribe silenciosamente el glosario global aprobado.

La aprobación nueva permanece cerrada: las referencias disponibles y los juicios simulados no bastan para calibrar. Falta evaluar falsos negativos, tiempo humano, semántica y naturalidad en una muestra independiente. Véase [estado y siguiente puerta](ESTADO_ACTUAL.md).

El catálogo `context_catalog` conserva pasajes completos y roles de personaje, misión, objeto, hechizo, lugar y documento, sin convertir etiquetas genéricas en nombres intraducibles. Relaciona registro/grupo/misión dentro del mismo mod/plugin; las fuentes del autor admiten alcance de mod, versiones, permisos y hash. La búsqueda restringe el ámbito antes de BM25 ([FTS5 oficial](https://www.sqlite.org/fts5.html)); la documentación de autores se considera referencia, con compatibilidad de versión no verificada. Las denominaciones aprobadas por mod se incorporan como restricciones; los conflictos con el glosario global quedan explícitos y no lo sobrescriben. Se incorporaron extractos de los README de [MCM Helper](https://github.com/Exit-9B/MCM-Helper/blob/main/README.md) y [Alternate Perspective](https://github.com/KrisV-777/Alternate-Perspective/blob/main/README.md), con hash y licencias conservadas privadamente; no se presume que describan la versión instalada.

`long_text` divide original y candidata solo en límites protegidos compartidos, conserva vecinos narrativos y verifica la recomposición completa. No presume alineación entre frases. `durable_run` continúa los casos independientes mientras bloquea dependencias de envíos/uso inciertos; el presupuesto difiere textos completos que no caben, sin ampliar el cap. Los prompts comparan agencia, modalidad, negación, comparaciones y recursos, aceptando metáforas y paráfrasis fieles. La [evaluación de discurso en traducción literaria](https://aclanthology.org/2023.acl-long.435/) motivó conservar vecinos y consistencia; no certifica resultados propios.

`review_calibration` vincula candidata, fuente, contexto, versión y evaluador con referencias humanas. No convierte cualquier retoque en error semántico, no cuenta mocks y agrupa familias para evitar observaciones duplicadas. Reutilizar una aprobación exige igualdad exacta y nunca libera cuarentenas ni aprueba una reparación nueva.

## Consolidación de responsabilidades

`core/artifacts` comparte hash por bloques y publicación atómica entre visor/workbenches/ejecutor. `quality/review_contract` valida también el puerto legado. La selección de términos aprobados se comparte en proveedor, cola y posedición: las propuestas no son obligaciones. La ingesta usa firma por registro y una sola transacción; conserva APIs y catálogos históricos sin convertir destructivamente sus esquemas.

El proveedor registra el HTTP original antes de parsear/conciliar; un 5xx o timeout con uso desconocido retiene presupuesto y no se reintenta por defecto. La idempotencia económica sigue en `BudgetController`; el diario de reevaluación offline no representa cargos reales. `DurableRun` verifica planes y recibos terminales y serializa callbacks por diario. Las advertencias menores de puntuación no disparan solas el juez, y las observaciones con hashes válidos orientan selección sin filtrar el dictamen al prompt ciego. El ejecutor facturable aislado es `translation/paid_trial.py`; no conecta con producción. La autorización acotada posterior se detuvo por identidad inválida.

## Terminología semántica y continuidad integradas

`ModKnowledge.term_usage` amplía los términos existentes con identidad de concepto, registros/campos, uso y formas explícitas; no concede aprobación. `concept_context_for` aplica esas restricciones también a contexto, proveedor y revalidación, evitando que una vía léxica vuelva a imponer el término en otro sentido. Los catálogos históricos de solo lectura conservan su esquema; las propuestas Sol no se insertan en términos aprobados. `NameRegistry` incorpora títulos QUST/BOOK y `validation_context_for` conserva todas las referencias locales y la documentación; el prompt mantiene su límite de cuatro fragmentos.

`DurableRun(continuity=True)` exige un fingerprint nuevo y admite dependencias de etapas anteriores. Distingue `BatchValidationError` (cuarentena local con uso conocido), `ProvenNotSentError` (único reintento, cero por defecto y máximo dos) y bloqueos del plan por integridad/credenciales/autorización/uso incierto. Guarda solicitudes, recibos, hashes, intentos e incidentes y entrega estado/causa/dependencias por trabajo. Antes de enviar verifica todos los recibos y solicitudes almacenados; una interrupción posterior a recibir permite validación local sin reenvío.

`BatchWorkflow` reutiliza el contrato del proveedor, `decode_translations`, ese diario y `ReviewWorkflow(model_roles=True, continuity=True)`. Congela fuentes, referencias y términos; exige Luna para traducir y aplica validación atómica por lote. El perfil de roles usa Sol para revisar/verificar y Luna para poseditar; vincula modelo/política/binding y calcula reservas offline por etapa, sin modificar libros económicos activos. Una revisión que no cabe queda aplazada conservando la candidata. Solo hay transporte simulado; no conecta importación a producción. El formato 1.2 y la lectura de dictámenes anteriores se conservan; los planes sellados no se reinterpretan con otro fingerprint. [Pruebas y límites](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/terminology-continuity-docs-20261106/Fase10_Evolucion_Linguistica/TerminologyContinuity_20261106/README.md).

## Barrido previo y experimento acotado

`DurableRun` detecta cualquier envío/uso incierto del plan antes de despachar trabajo nuevo. `BatchWorkflow` serializa términos dataclass, restaura enumeraciones, filtra el glosario aplicable y lo comprueba en la puerta atómica; sella el backend y conserva metadatos narrativos. La revisión ampliada admite hasta 2.000 unidades solo en perfiles explícitos nuevos. `scope_term` rechaza ámbitos cuya evidencia no cabe en el contrato. Estos cambios tienen regresiones específicas; no migran datos productivos.

`heavy_trial` coordina controles Sol → traducción Luna de fuentes nuevas → revisión de muestra Sol → posedición Luna y verificación Sol → auditoría y exportación ciega. Reutiliza `PaidTrial` con un ámbito opcional sellado de modelos, total, sublímites, máximo de solicitudes y bytes. Las reservas se basan en la solicitud serializada, salida máxima, escritura de caché y margen; la conciliación por modelo usa los tokens/costes registrados. El perfil histórico de US$0,35/Luna permanece protegido por su autorización anterior. No hay reenvíos de uso incierto ni nuevos cargos durante preparación.

El plan protege fuentes/artefactos/código con hashes iniciales/finales y vigila metadatos entre llamadas, evitando hashear la DB entera en cada petición. Una alteración de plan o fallo de transporte detiene el envío; un dictamen malformado recibido con uso conocido se aísla como fallo local. Los costes proyectados son heurísticos; el cap y la reserva por llamada prevalecen y pueden aplazar etapas. La preparación puede ejecutarse solo después de autorización explícita; el transporte real aún no se ejercitó con esta selección. [Plan, mediciones y límites](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/terminology-continuity-docs-20261106/Fase10_Evolucion_Linguistica/HeavyTrial_20261106/README.md).

## Mantenimiento documental

`translation/paired_trial.py` prepara 24 fuentes únicas, cuatro por perfil, dos variantes y ocho controles. Exige partición exacta y familia explícita en lotes múltiples; usa transporte bloqueado en construcción, salida 2.048 tokens, límite 16.384 bytes y propuestas económicas aisladas. Los UID son opacos en el transporte; la referencia local conserva el origen real y no se relaja el guard de rutas. El sello vincula artefactos y fuentes; las etiquetas de controles no se envían al juez. La preparación cerrada conserva sus flags históricos; el estado ejecutado vive en un diario separado. El ejecutor real verifica recibos, presupuesto, identidad y parada persistida, sin reenvío automático.

El cierre de la prueba vincula secuencia/UID/hash a enumeraciones del esquema y añade instrucciones/esquema a la idempotencia; la validación local sigue comprobando orden y unicidad. Usa el hash legado de texto recortado y contrasta por separado el original bruto protegido. El esquema nuevo está probado offline, pendiente de confirmación API. El libro aislado distingue entrada estándar, lectura y escritura de caché; las correcciones desde recibos quedan auditadas. La parada persistida permite lectura local, pero prohíbe envíos nuevos. El límite de US$0,35 prevalece sobre el máximo de llamadas; la reserva original de preparación no cubría escritura de caché. [Resultado real y límites](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/technical-consolidation-docs-20261105/Fase10_Evolucion_Linguistica/PairedTrial_Run_20261105/README.md).

`lore/documentation_inventory.py` descubre páginas candidatas desde el ID de Nexus y textos documentales acotados, sin downloader ni escritura en mods. El inventario usa una agregación previa de cadenas por mod para evitar un join masivo. Los textos locales llevan hash; los resúmenes externos propios se distinguen del texto de autor y registran versión/procedencia. `context_catalog.retrieve` consulta documentos activos relevantes por separado y `ModKnowledge.context_for` conserva uno en el límite de cuatro fragmentos. La relevancia sigue siendo léxica y no establece equivalencias de lore ni aprobación del glosario.

`lore/consistency.py` construye un registro de nombres del último escaneo, leído con SQLite `mode=ro`. Usa `origin_plugin`, tipo y FormID para relacionar overrides/separar homónimos, y localización existente del juego base como referencia. Excluye candidatas nuevas de esa autoridad, protege variables y evita tratar palabras comunes/clases de NPC como nombres propios. Las referencias y conflictos son datos acotados, no aprobaciones. Las familias admiten formas de concordancia y preferencias pendientes; las relaciones léxicas son hipótesis.

`ConsistencyContext` combina ese registro con `ModKnowledge` para preparación del proveedor/coordinador, juez y recomprobación. `review_experiment --name-source <DB-solo-lectura>` vincula el hash del nuevo insumo al experimento; rechaza reemplazar la carpeta humana y un cambio de fuente. El diseño controlado acepta también esa fuente, sin ejecutar. Política `risk-2.3.0`, revisor `independent-review-2.3`: una discrepancia de referencia bloquea reutilización ciega/posedición insegura, conserva el estado humano y permanece visible en la cola. Esa integración no modifica el contrato 1.2 ni habilita por sí misma transporte real. El visor presenta la procedencia de los nuevos fragmentos cuando la copia los contiene.

- `README.md` solo orienta. `ESTADO_ACTUAL.md` contiene métricas/pendientes; `GUIA_DE_USO.md`, operación; este documento, reglas/técnica; `HISTORICO.md`, cronología y acceso a evidencia.
- Registrar fecha, fuente y si algo es medido, estimado, histórico o propuesto. Actualizar el estado sin repetir una matriz de métricas en todos los documentos.
- Conservar informes cerrados. Cambios de navegación se permiten, pero se registran y el original exacto queda en el archivo con hash. No reinterpretar sus resultados como aprobaciones actuales.
- Mantener los cuatro documentos activos compactos, idealmente hasta 150 líneas cada uno. Llevar el detalle cerrado a un informe fechado o al archivo; no convertir el histórico en otra enciclopedia.
- Los originales de la raíz y las fuentes documentales previas están en `archivo/originales_20261105.zip`, verificables con `archivo/MANIFIESTO.json`. Los archivos locales y públicos tienen distinto alcance; no publicar el archivo privado del workspace.
- La publicación pública contiene solo documentación agregada. La autorización de API fue específica para la prueba pareada, ahora detenida por su condición crítica. No habilita envíos fuera de ese alcance, producción, publicación de fuentes ni cambios silenciosos del glosario.

En compatibilidad de pilotos, `BatchWorkflow` conserva su `sol_selection_policy` del 2 %/14 y `heavy_trial.prepare` sus límites sellados. No son la política del modo Plan B. La regresión histórica de 80 casos ordinarios y dos consultas Sol se mantiene sin reinterpretarla como cobertura de un proyecto masivo.

La ruta real de `PaidTrial` vincula `allowed_model_id` al modelo exacto de cada solicitud y reserva dentro del bloqueo, antes del HTTP; evita heredar la restricción predeterminada a Luna al evaluar con Sol. Una regresión recorre Sol→Luna con servidor/credenciales simulados. La respuesta Luna real `incomplete` con uso cero quedó en `QUARANTINED_UNKNOWN_COST`; su reserva no se liberó ni se consideró gratuita. El HTTP original y el coste conocido de los controles se conservaron antes de la parada persistida. El sello v5 conserva los mismos payloads y caps; la preparación no debe confundirse con el diario cerrado de ejecución. [Incidencia y métricas](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/terminology-continuity-docs-20261106/Fase10_Evolucion_Linguistica/HeavyTrial_Run_20261106/README.md).

## Contratos y recuperación estabilizados

`provider_receipts` comparte identidad exacta, mensajes visibles y telemetría consistente, incluidos caché, razonamiento y tier. `translation_wire` ofrece `compact-v2`: secuencia/texto al modelo; partición, UID y SHA permanecen vinculados localmente. El intercambio 1.2 y el perfil legado siguen disponibles. `reasoning` admite únicamente esfuerzo acotado; el nuevo perfil usa `low` con salida 2.048 y sin herramientas/pro. No se presume que esto explique el recibo histórico de uso cero.

`isolate-known-v1` concilia contenido fallido con uso positivo verificado, sin aprobarlo; uso cero/anómalo, alteración de identidad económica o exceso retienen y bloquean. La intención se persiste antes del transporte. `PaidTrial` recupera recibos preservados antes/después del commit económico sin reenviar; los HTTP fallidos conciliados siguen siendo errores. `DurableRun` comprueba también el plan en memoria y estados terminales sin recibo. Los planes nuevos de revisión declaran dependencias; los antiguos no se migran.

Backends offline con `response_envelopes=True` conservan el Responses completo y calculan su coste virtual mediante ese parser común. `BatchWorkflow.import_to_staging` une revisión/posedición/verificación e importación pendiente; se revalidan originales/contextos y se impide sobrescribir trabajo humano. TM/diferencial comparan contexto completo y texto exacto. `Mo2ModPackager` integra staging hermano, bloqueo, backup y publicación reversible con historial; no garantiza atomicidad conjunta ante pérdida de energía. El Batch API legado queda solo para simulación hasta disponer de conciliación real segura. [Implementación y pruebas](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/terminology-continuity-docs-20261106/Fase10_Evolucion_Linguistica/Stabilization_20261106/README.md).
