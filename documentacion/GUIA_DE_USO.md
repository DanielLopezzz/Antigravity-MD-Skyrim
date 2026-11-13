# Guía de uso y revisión local

Actualizada el **2026-11-11** a partir de los lanzadores, CLI y visor existentes. [Estado y pendientes](ESTADO_ACTUAL.md) · [Referencia técnica](REFERENCIA_TECNICA.md).

## Protocolo en Skyrim para Moon and Star

### Prueba mínima de las opciones del jugador

1. **Aislar.** Usa una instancia de prueba de MO2 con su propio `overwrite` y perfil. Si necesitas crearla, usa una instancia portable separada con rutas de `mods`, `profiles` y `overwrite` propias. Replica los archivos de configuración y las prioridades de la instalación de referencia. Activa guardados e INI específicos del perfil. Copia el `.ess` y su `.skse` asociado al directorio de guardados del perfil cuando existan ambos. Conserva una copia de referencia fuera del alcance de autoguardado y guarda `modlist.txt`, `plugins.txt`, `loadorder.txt` e INI de la copia.
2. **Control.** Inicia Skyrim mediante SKSE en el perfil aislado con el paquete desactivado. Carga un guardado anterior a cada conversación y selecciona la opción inglesa de la tabla. Anota qué responde el NPC, si el menú continúa o se cierra y el resultado de `getstage MASMainQuest` o `getstage MASSelyseDialog` antes y después. Si ya usas SKSE Menu Framework, anota el paquete de voz visible en F1 sin cambiarlo. El autor documenta el correo al viajar al aire libre o el inicio con Mandyn en Sleeping Giant Inn. [Readme de Moon and Star](https://www.nexusmods.com/skyrimspecialedition/mods/4301?tab=docs) · [Configuración de DBVO](https://www.nexusmods.com/skyrimspecialedition/mods/84329?file_id=380849).
3. **Instalar.** Cierra Skyrim. En MO2 pulsa **Instalar un mod desde un archivo** y selecciona `Moon_and_Star_ES419_Visual_Test.zip`. Instálalo como mod nuevo, al nivel que contiene `MoonAndStar_MAS.esp`. Habilítalo en el panel izquierdo con prioridad superior a Cleaned Plugin Replacer 2.0. Verifica en **Conflictos** y **Data** que gane la copia del ZIP, que `overwrite` no la sombree y que `plugins.txt` y `loadorder.txt` conserven el orden. Compara el SHA-256 de la copia y el del original con el manifiesto. Mantén DBVO y sus recursos tal como estaban.
4. **Comparar.** Inicia por SKSE, vuelve al mismo guardado de referencia y elige la opción en español. La respuesta NPC, el cierre o continuación del menú, la etapa y los objetivos deben coincidir con el control. El protagonista puede quedar en silencio o reproducir la frase inglesa correcta. Registra tres resultados separados. **Visual** comprueba caracteres, opción presente y texto. **Audio** distingue silencio, voz correcta o voz equivocada. **Función** comprueba selección, avance, cierre y estado de misión. El ZIP de dos opciones conserva los subtítulos del NPC tal como están en el original.
5. **Aislar un fallo y revertir.** Ante un bloqueo o voz equivocada, detén la prueba y conserva captura, diálogo y log. Solo en la instancia aislada, si **Data** confirma que el proveedor de `DBVO.dll` es el mod que contiene el DLL y sus mapas, desactiva esa capa y repite control y prueba con los mismos guardados. Comprueba que el DLL no esté disponible y que el orden derecho coincida. Al finalizar, cierra Skyrim, desactiva el ZIP y restaura DBVO si se desactivó en la copia. Confirma que gana Cleaned Plugin Replacer con el SHA de origen y repite desde la copia de referencia. Usa una copia nueva de ese guardado para cada pasada.

| Caso | Identidad local y cambio | Control funcional |
|---|---|---|
| Mandyn | `MASMainQuestHelp` `0000E253`. «Anything I can help with?» pasa a «¿Te puedo ayudar en algo?» | La respuesta `INFO 0000E254` conserva su continuación. Compara `MASMainQuest` y la respuesta completa con el control. |
| Selyse | `MASSelyseDialogDecline` `0000C04F`. «I'm too busy right now.» pasa a «Ahora mismo tengo mucho que hacer.» | Se espera que el encargo quede disponible para más adelante. La respuesta `INFO 0000C050` invita a volver. La aceptación se prueba con `MASSelyseDialogAccept` `0000C04D`, «Sure.». |

### Elevador y recorrido ampliado

La prueba de `Moon_and_Star_ES419_Elevator_Test.zip` queda diferida al recorrido normal. Cuando llegues al elevador averiado de Kagrenar Depths, compara desde antes de entrar a la zona. Verifica que la inicialización y activación coincidan y que el cuadro muestre «Parece que se descompuso al bajar.» y pueda cerrarse. La carpeta `Scripts` del ZIP debe ganar para `masbrokenelevator.pex`. Al desactivarla, comprueba que el script vuelva a provenir de `MoonAndStar_MAS.bsa` con el hash original del manifiesto. La prueba de empaquetado no modifica ese BSA.

En el recorrido ampliado conserva los resultados de las ocho comprobaciones iniciales del original y añade rutas alternativas, entrega de objetos, recompensas y misiones completas. Repite desde guardados anteriores a la interacción. Registra FormID o EditorID, versión del paquete, resultado antes y después, capturas y errores del log. La aceptación visual incluye texto ES-419 legible en todos los campos traducibles del alcance, con tildes, eñes y signos de apertura. La aceptación funcional incluye el comportamiento original y la reversión.

| Caso | Identidad local en `MoonAndStar_MAS.esp` | Evidencia en ambas pasadas |
|---|---|---|
| Inicio natural | `MASMainQuest` `0000696B`, Mandyn `MASMandyn` `0000E247`, carta `MASCourierLetter` `00016B6D` | Disparador, apertura y cierre de conversación, objetivos 10 y 20. Estos números identifican objetivos, las etapas se anotan según el juego. |
| Little Vivec | `MASLittleVivecLocation` `0001A581`, `LVSelyseTradehouse` `000029E4` | Coincidencia de nombres entre objetivo, conversación y entrada. |
| Toren | `MASTorenDialog` `0000799C`, `MASTorenOrder` `00008F5B` | Objetivo 10 de entregar el pedido en Riverwood Mill, recepción por el destinatario, retorno y entrega de recompensa según la línea base. |
| Bal-Ran y mapa | `MASBalRanDialog` `0000C066`, `MASMossCreekMarker` `00012ED9` | Comprueba que el objetivo 10 y el marcador que gana en el parche de Moss Creek muestran el mismo nombre. |
| Selyse | `MASSelyseDialog` `0000C01B` | Contadores de pieles, nirnroot u otros encargos. |
| Libro y recompensa | `MASKagrenarBookTranslated` `00011909`, `MASMainQuestMandynGiveTopic` `00010E11` | *Memories of Dorana*, páginas y glifos. En el recorrido correspondiente, anota inventario antes y después de entregar el objeto y recibir la recompensa. |

Para completar el piloto, coteja el texto ES-419 contra el manifiesto, incluidos los textos de menú y subtítulos. La ejecución real incluye las 194 opciones del jugador.

### Revisar el piloto real de Moon and Star

Abre `Revisar_piloto_Moon_and_Star.bat` desde la raíz. **Necesita criterio** muestra el aviso de doble negación y **Control de calidad** reúne las cuatro muestras independientes de Sol. La consulta y los cambios editoriales se guardan en el proyecto privado. El visor muestra candidatas por texto completo, incluidos libros reensamblados.

En la instancia de prueba, desactiva el ZIP de dos opciones antes de activar `Moon_and_Star_ES419_Prueba_Completa.zip`. Confirma que las cinco copias y los JSON del manifiesto ganan, y que el orden de plugins es idéntico al de la referencia. Repite Mandyn y Selyse desde el mismo guardado y verifica objetivos, diarios, subtítulos y mapa al avanzar. Cierra Skyrim y desactiva el ZIP para volver a los archivos originales. El ZIP completo es provisional y el del elevador se prueba cuando llegues a Kagrenar Depths.

## Revisar el resultado real de Amber Guard

Abre `Revisar_piloto_Amber_Guard.bat` desde la raíz. La carpeta privada vigente es `NolvusAIT_Intelligence/20261107_AmberGuard/private/full_pilot_live/run`; contiene manifiestos, plan y tres tareas contextuales pendientes. `full_pilot_live/preflight.json` registra el alcance autorizado; `classification_overlay.json`, situado en la carpeta privada de Amber Guard, justifica las cinco reclasificaciones locales. Los anteriores `full_mod_pilot_v2` y preparados intermedios se conservan como evidencia.

Consultar ese proyecto no consume API ni acredita aprobación. Contiene 133 unidades reales (132 fuentes; un libro dividido en dos). El visor original (`Revisar_piloto_Amber_Guard.bat`) conserva las 12 aprobaciones humanas de Daniel (11 padres) en `full_pilot_live/run/private/human_reviews.json`; la copia continuada en `NolvusAIT_Intelligence/20261107_AmberGuard_Continuation/private/review_continuation` conecta `approved_glossary.json`, incorpora las 11 unidades reparadas en `focused_repair_live/repaired_targets.json` (132/132 padres reensamblados en `focused_repair_live/assembled_candidates_132.json`, US$0,011996 conciliados en 9 solicitudes), sincroniza las 12 decisiones humanas (`121` Consulta opcional, `11` Aprobados sin avisos abiertos, `1` aprobado con traza de homónimo y `0` Cuarentena técnica) y genera el paquete DSD aislado en `dsd_package/Amber_Guard_ES419_Provisional` (`151` entradas: `132` base + `17` `CELL FULL` de puertas interiores + `2` `NPC_ FULL` descriptivos, con `INFO NAM1` 1-based `TRDT`, `QUST NNAM` `QOBJ` y `QUST CNAM` `original`). Los textos reensamblados y las métricas originales del piloto siguen intactos en `full_pilot_live/assembled_candidates.json`, `result.json` y `analysis.json`. Los diagnósticos previos y sus costes se consultan en [Estado](ESTADO_ACTUAL.md) e [Histórico](HISTORICO.md). No importar a la TM maestra de producción.

## Accesos desde la raíz del workspace

Tras la depuración, el visor y el staging pendiente muestran la posedición recomprobada cuando supera QA; la candidata original queda conservada en caché y en `original_candidate`. No se transforma en aprobación humana ni se promueve a TM. Reanudar mantiene los mismos recibos y no vuelve a solicitar traducciones por este cambio.

**Plan B, modo aislado:** `python -m nolvus_translator mass prepare --rows <filas.json> --project <proyecto-nuevo>` prepara manifiesto, contexto y cohortes. `mass status --project <proyecto>` abre el libro `project.sqlite3` del workbench y puede crearlo si no existe; exige los mismos caps con que se preparó. No es un lector genérico de pilotos. Para Amber Guard consulta `full_pilot_live/result.json` y `analysis.json` (o abre su `ledger.sqlite3` en solo lectura). `mass simulate --project <proyecto> --receipts <fixtures.json>` recorre el flujo integrado sin red. Los caps son virtuales en este modo; no autorizan gasto. En preparación pueden añadirse `--approved-glossary <db-o-json>`, `--name-source <db-solo-lectura>`, `--knowledge <sidecar>` y `--approved-memory <snapshot>`, todos consultados en solo lectura inmutable.

Usa el Python del entorno `.venv` desde el workspace. `--project` identifica la cartera compartida; `--run-name` permite otro mod o una revisión del manifiesto dentro de esa misma cartera, conservando gastos y caché. El valor predeterminado es `run`. La carpeta histórica `private/full_mod_pilot_v2` contiene solo preparación offline; el resultado vigente es el de Amber Guard indicado arriba.

Para revisar: `python -m nolvus_translator mass review --project <proyecto>`, o el botón **Abrir revisión Plan B** de la pestaña Revisión. Se guardan decisiones en el overlay privado del proyecto, sin escribir la TM maestra. No hay que aprobar toda la población para consultar una muestra. Los comandos existentes de pilotos históricos conservan su alcance anterior. [Procedimiento inicial de Plan B y límites](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/4060d9f3581df44732228c7cb3eb46ae8646ad3b/Fase10_Evolucion_Linguistica/PlanB_20261106/README.md#9-procedimiento-para-un-piloto-de-un-mod-completo).

| Acceso | Qué abre |
|---|---|
| `Revisar_piloto_Amber_Guard.bat` | Piloto real actual: 133 unidades y observaciones, sin llamadas API |
| `launch_gui.bat` | Aplicación principal; puede inicializar sus carpetas de trabajo |
| `Revisar_cuarta_prueba.bat` | Visor del piloto activo R7 de 1.000 cadenas |
| `Revisar_traducciones.bat` | Casos del piloto anterior y ejemplos sintéticos |
| `Revisar_evolucion_inteligente.bat` | Copia experimental del flujo de reevaluación offline |
| `Revisar_prueba_pareada.bat` | Resultado real separado: 47 candidatas para evaluar y una en cuarentena técnica |
| `Validar_prueba_pesada.bat` | Inspecciona el antecedente pesado; su sello de código es anterior a la estabilización. No llama a la API |

Estos lanzadores pertenecen al workspace con código. El repositorio público de documentación no incorpora la aplicación. El visor de casos no llama a la API ni importa decisiones a producción o a la TM.

La aplicación principal conserva diez áreas: dashboard, escaneo, traducciones, glosario, TM, revisión, actualizaciones, exportación, informes y configuración. El visor se puede abrir también desde **6. Revisión → Revisar casos del informe**. La cola de la DB y los casos del informe tienen funciones distintas.

## Revisar la cuarta prueba

1. Abre `Revisar_cuarta_prueba.bat`. Comprueba que la cabecera indica `Phase4_Luna1000_R7_DiagnosticComplete`.
2. Usa **Recargar** para leer las candidatas y decisiones guardadas. Selecciona estado y prioridad: **Obligatoria**, **Muestra de calidad** o **Adicional**.
3. Compara original, candidata, contexto, procedencia y alertas. Que una frase no tenga alertas no demuestra que esté bien traducida.
4. Escribe tu nombre. Puedes **Aprobar caso**, **Guardar pendiente**, **Requiere investigación** o **Rechazar caso**. Investigar/rechazar requiere una nota.
5. Anota significado, naturalidad, voz, terminología, contexto o formato; distingue retoques menores y errores graves. Usa **Siguiente pendiente** para continuar.

Antes de aprobar se comprueban integridad, términos obligatorios y registro regional. La revisión humana debe juzgar también el sentido y la voz. La neutralidad ES-419 no exige calcar el inglés ni sustituir formulaciones naturales por otras rígidas.

Los casos en cuarentena se pueden evaluar lingüísticamente, pero aprobar una referencia en el visor **no arregla el hash ni libera el lote para importación**. La propuesta de formato del poema está marcada para revisión obligatoria y conserva su procedencia de IA.

Si hay texto sin guardar, el visor avisa al navegar. Si otra ventana guarda decisiones, usa **Recargar**; hay bloqueo de guardado e historial. Una decisión se vincula al original y al contexto revisados.

La copia de evolución se abre con `Revisar_evolucion_inteligente.bat`. Muestra estado del evaluador, procedencia y controles sin crear aprobaciones. Los dictámenes del simulador son inconclusos; no miden calidad lingüística. Su carpeta es `NolvusAIT_Intelligence/20261105_Functional/private/review_final`; las decisiones se guardan en su propio `private/human_reviews.json`. Cuando existe ese overlay, el ejecutor experimental rechaza regenerar la copia. Las aprobaciones previas exactas aparecen como **Referencia aprobada**, leídas de una copia inmutable separada; no se duplican en el overlay de decisiones nuevas. **Exportar evaluación** guarda referencias y exclusiones de calibración en `private/calibration_summary.json`; descarta dictámenes simulados y referencias desactualizadas. Los segmentos se recomponen solo tras comprobar su identidad, orden e integridad; si la revisión resulta inconclusa se conserva la candidata histórica completa.

El visor de evolución abre por defecto **Cola → Necesita criterio**. Las opciones **Control de calidad** y **Consulta opcional** permiten muestrear o explorar; no exigen revisar todo el corpus. **Cuarentena técnica** también puede incluir incumplimientos terminológicos obligatorios: en Amber Guard hay dos por Thane. No todos indican corrupción de identidad o formato. El estado humano permanece separado: quitar una advertencia de la cola no aprueba la traducción. Las observaciones de esta sesión aparecen con su motivo y propuesta sin aplicar.

## Dónde quedan las decisiones

| Revisión | Archivo privado, relativo al workspace |
|---|---|
| Amber Guard | `NolvusAIT_Intelligence/20261107_AmberGuard/private/full_pilot_live/run/private/human_reviews.json` |
| Cuarta prueba histórica | `NolvusAIT_PilotStaging/Phase4_Luna1000_R7_DiagnosticComplete/private/human_reviews.json` |
| Casos anteriores | `NolvusAIT_Evolution/20261104/private/human_reviews.json` |

No edites esos JSON manualmente. Conserva copias de seguridad privadas junto con las carpetas de las pruebas. Abrir otra prueba no sustituye las decisiones anteriores. Las aprobaciones del visor son referencias de evaluación; su incorporación a glosario, TM o producción requiere otra actuación trazable.

## CLI de revisión

La comparación histórica del 5 de noviembre está en `NolvusAIT_Intelligence/20261105/private/final_comparison`: selecciónala con **Abrir otra prueba…** para ver las mismas candidatas con clasificación de riesgo. Las decisiones originales siguen en R7; no se copiaron a esta carpeta ni se crearon aprobaciones nuevas. Las etiquetas de riesgo informativas no equivalen a aprobación lingüística.

Desde PowerShell, situado en la raíz del workspace:

```powershell
$env:PYTHONPATH='src'
python -m nolvus_translator --review-folder 'NolvusAIT_PilotStaging/Phase4_Luna1000_R7_DiagnosticComplete'
```

Para la revisión anterior: `python -m nolvus_translator --review-cases` con el mismo `PYTHONPATH`.

## Desarrollo, configuración y exportación

- `pyproject.toml` declara Python >=3.11. El punto de entrada es `nolvus_translator.__main__:main`; no se requiere una versión exacta de Python/SQLite tomada de un informe antiguo.
- `workspace_data/config.json` es la configuración local; la DB principal es `workspace_data/nolvus_translator.db`. No se usan como destinos de una prueba de traducción nueva.
- La suite se ejecuta con `python -m unittest discover -s tests -t .` y `PYTHONPATH=src`. Crea datos temporales de prueba; no constituye una operación de solo lectura. La última suite de código verificada es 404/404.
- Workbench offline: `python -m nolvus_translator.translation.intelligence_workbench --source <DB> --pilot <R7> --output <carpeta-nueva>`. Solo lee fuentes y crea un experimento aislado; rechaza reemplazar uno existente. `--benchmark-count` opcional mide ingesta sintética. No ejecuta traducciones ni jueces reales.
- Catálogo contextual offline: `python -m nolvus_translator.lore.knowledge_workbench --source <DB> --output <carpeta-nueva>`. Para enriquecer una copia ya creada: `--knowledge <sidecar> --documents-manifest <registro-json>`. Verifica hashes/permisos del registro; no descarga ni ejecuta código externo.
- Pruebas de mods traducidos en MO2: copiar el paquete bajo `MODS\mods\<NombreMod>` y activarlo al final del panel izquierdo (para que sus archivos sueltos prevalezcan sobre BSAs en USVFS). El icono "No valid game data" (`X`) en MO2 para paquetes de solo DSD es cosmético e inocuo. En el juego, verificar interacción de puertas (`CELL FULL`), subtítulos hablados sincronizados, opciones de menú, menús MCM y etapas de misión.
- Antes de una exportación futura: vista previa, evidencia editorial, respaldo, perfil clonado de MO2 y compatibilidad del ejecutable real. Las capacidades de exportación existentes no autorizan desplegar el piloto actual.
- No actives proveedores, ejecutes runners facturables ni deshabilites el candado por seguir esta guía. Cada prueba nueva exige alcance, presupuesto y autorización.

## Antecedentes: preparación y validaciones conservadas

La línea privada de aquel corte era `codex/heavy-trial-preflight-20261106`; conserva `codex/terminology-continuity-20261106`, `codex/contextual-coherence-20261105`, `codex/technical-consolidation-20261105` y las ramas anteriores. Los planes históricos sellados no se reabren con un fingerprint de código diferente.

El diseño está en `NolvusAIT_Intelligence/20261105_Consolidation/private/next_validation_final/plan.json`; sus previews son privados. Preparar no envía solicitudes ni autoriza gasto. Si alguien empieza a guardar revisiones mientras se genera una copia, se conserva su archivo de casos y los recibos quedan disponibles en el diario. El visor en marcha no necesita cerrarse para este trabajo; los cambios de código se cargan al abrirlo nuevamente.

La preparación histórica con nombres está en `NolvusAIT_Intelligence/20261105_Coherence/private/final/next_validation/plan.json`, sin ejecutar. En copias futuras, el panel de contexto muestra las denominaciones de referencia y posibles piezas relacionadas, indicando propuestas pendientes. Una discrepancia nueva de identidad puede pedir atención aunque exista una aprobación anterior; esa decisión permanece guardada. El barrido no regeneró tu carpeta activa ni te asigna otros 1.000 casos obligatorios. Los 25 de cuarentena ya tienen decisión lingüística; su retención técnica es independiente.

## Antecedente: flujo de continuidad offline

El flujo integrado `BatchWorkflow` recibe solicitudes Luna preparadas y fuentes congeladas; crea un diario nuevo y solo admite backends simulados. Conserva traducciones válidas, aísla respuestas defectuosas por lote y dirige revisión/verificación a Sol y posedición a Luna. No activa la API ni cambia el funcionamiento de los visores históricos. Los términos comunes necesitan un ámbito semántico respaldado: una palabra compartida no basta para imponer una equivalencia. Las propuestas siguen pendientes; no actualizan TM ni glosario aprobado.

La evidencia de aquel corte está en `NolvusAIT_Intelligence/20261106_TerminologyContinuity`: `INFORME.md` y `METRICS.json` resumen resultados; `private/evidence/private/batch_report.json` enumera cada lote sintético y su causa. `private/evidence/integration/summary.json` y `private/results.json` dentro de esa integración conservan el ejemplo programado. Son pruebas técnicas; no son nuevas traducciones o decisiones humanas. Se conservan también los logs y el runner privado; este último rechaza sobrescribir una ejecución anterior.

Para una ejecución masiva futura, usa un plan nuevo con roles, referencias, dependencias y límites sellados. Reabrirlo reutiliza recibos completos. Revisa `state`, `cause`, `blocked_by` e incidentes antes de resolver pendientes; no borres el diario para forzar reenvíos. Ante consumo incierto o fallo de integridad no se continúa enviando. La política nueva no permite saltarse la parada crítica de una prueba histórica ni autoriza gasto.

## Antecedente: preparación de la prueba pesada

El plan histórico cerrado está en `NolvusAIT_Intelligence/20261106_HeavyTrial/private/prepared_final_v5`; sus previews, fuentes, términos y código están vinculados por hashes. Usa `Validar_prueba_pesada.bat` para comprobarlo offline. Los borradores anteriores y `offline_exercise` son evidencia de preparación/simulación y no deben confundirse con una ejecución real. `INFORME.md`, `SUMMARY.json` y `OFFLINE_CHECK.json` resumen alcance, costes supuestos y pruebas.

Se conserva el límite de **US$0,30**: US$0,08 Luna y US$0,22 Sol. El runner privado exige autorización nueva explícita y crea `private/live`; no reabre el piloto anterior. La ejecución posterior quedó detenida; sus resultados reales se enlazan más abajo. El plan de preparación no representa cobertura alcanzada. No se retransmiten resultados inciertos ni se eleva el cap para completar un porcentaje.

El diseño preveía hasta 28 versiones ciegas de 14 pares en `private/live/review`. La parada impidió producir esa evaluación natural; no hay un conjunto antes/después que revisar. El resultado real y su carpeta diagnóstica se describen más abajo. No necesitas volver a revisar las 1.000 cadenas antiguas.

## Documentación y publicación

La preparación pareada histórica está en `NolvusAIT_Intelligence/20261105_PairedTrial/private/prepared_final_v4`; el plan y los previews son privados. `private/Validar_preparacion.bat` verifica archivos y fuentes sin enviar solicitudes. Los borradores anteriores se conservan, pero no son la selección final. La ejecución autorizada está cerrada por una discrepancia de identidad; `Validar_preparacion.bat` muestra el estado real por separado y no envía HTTP. Tus revisiones activas no se sustituyeron.

Para consultar aquel resultado, abre `Revisar_prueba_pareada.bat`; en el filtro «Casos» puedes seleccionar «Prueba pareada». «Necesita criterio» agrupa 47 candidatas pendientes de evaluación, no 47 errores; cambia la cola a «Cuarentena técnica» para la restante. La evaluación de IA natural figura como no ejecutada. Las variantes no se identifican durante la lectura y ambas muestran las mismas referencias disponibles para adjudicación. Guardar crea solo `private/live_20261105/review/private/human_reviews.json`; una aprobación no libera el recibo retenido. No es necesario repetir la revisión de los 1.000 casos. El informe parcial está en `NolvusAIT_Intelligence/20261105_PairedTrial/INFORME_RESULTADOS.md`.

El contexto ampliado está en `NolvusAIT_Intelligence/20261105_Documentation/private/knowledge.sqlite3`, separado del catálogo anterior y del visor activo. Las fuentes web se presentan como resúmenes del proyecto, con advertencia cuando su versión no coincide o no puede verificarse. Un enlace descubierto en el inventario no equivale a documentación validada. Las adjudicaciones de los 32 casos y la propuesta de El Corcel están en esa misma carpeta privada; no sustituyen tus decisiones ni crean aprobaciones.

Edita los cuatro documentos de `documentacion/` en el workspace local. La copia `Antigravity-MD-Skyrim/documentacion/` es el espejo documental para publicar; no se actualiza automáticamente. Mantén ambas copias iguales antes de publicar, salvo sus archivos históricos, que tienen distinto alcance.

El antiguo `subir_md_a_github.bat` no prepara ni incorpora documentos nuevos: hace pull/push de commits existentes y contiene mensajes y una ruta heredados. No se ejecutó ni se modificó en esta reorganización. La publicación debe revisar el diff del repositorio documental y excluir siempre fuentes, credenciales, DB, corpus y decisiones humanas individuales.

La política 2 %/14 pertenece al ejecutor legado. Plan B calcula muestras independientes y revisiones por riesgo según reservas y presupuesto; Sol no retraduce todo el corpus. Los avisos sin evidencia no justifican una revisión adicional. No aumentes esa cobertura automáticamente ni confundas el máximo del motor offline con la política del producto.

La ejecución real de esta preparación quedó detenida. `NolvusAIT_Intelligence/20261106_HeavyTrial/RESULTADO_REAL.md` y `RUN_METRICS.json` contienen el corte real; `private/live_r2` conserva los nueve recibos y el libro económico. El primer intento local sin HTTP se conserva en `private/live`. La exportación `live_r2/review` es diagnóstica, sin revisiones naturales ni cambios de candidata: no constituye una evaluación antes/después y no es necesario revisarla. La validación muestra preparación y ejecución asociada por separado. No vuelvas a lanzar el diario cerrado ni borres su cuarentena para forzar un reenvío. [Resultado y condiciones para continuar](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/terminology-continuity-docs-20261106/Fase10_Evolucion_Linguistica/HeavyTrial_Run_20261106/README.md).

## Antecedente: perfil estabilizado y microdiagnóstico previo

Para pruebas nuevas se selecciona `compact-v2` y roles Luna/Sol explícitos. Los contratos y visores anteriores permanecen legibles. Un sello anterior puede rechazar el código actualizado: conserva ese antecedente; no borres el sello ni la cuarentena para forzar su ejecución. Los pares ciegos nuevos solo se generan si Sol obtuvo un dictamen real sobre esas candidatas.

La preparación privada está en `NolvusAIT_Intelligence/20261106_Stabilization/private/next_probe_v3/plan.json`; el procedimiento `probe_runner.py` ya pasó una simulación con replay. Prevé dos traducciones Luna y una evaluación Sol, hasta US$0,05, sin reintentos ni promoción. Requiere autorización específica nueva, hash del plan y carpeta de ejecución separada. No se ha ejecutado con API. La plantilla Sol se reconstruye sobre la candidata recibida; no se envía como traducción real. [Resultado y condiciones](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/terminology-continuity-docs-20261106/Fase10_Evolucion_Linguistica/Stabilization_20261106/README.md).

La importación del flujo integrado solo escribe pendientes en staging aislado, revalida fuente/contexto/estructura/glosario y protege decisiones humanas. Su repetición es idempotente; no escribe TM. No necesitas volver a revisar el visor histórico por estas pruebas sintéticas.
