# Estado actual y próximos pasos

Verificado el **2026-11-06**. Esta es la fuente del estado vigente; [uso](GUIA_DE_USO.md), [reglas y arquitectura](REFERENCIA_TECNICA.md) e [histórico](HISTORICO.md) cubren temas distintos. Las cifras de revisión son un corte, no un contador en tiempo real.

## Dónde está el proyecto

Nolvus AI Translator localiza Nolvus Awakening al español latinoamericano neutro (ES-419). El piloto de 1.000 cadenas está procesado. La última ejecución real Luna/Sol terminó con ocho controles Sol y un recibo Luna incompleto con uso cero. La estabilización posterior está implementada y verificada offline; no hubo otra prueba facturable. **Quedan la revisión ciega natural y la calibración; traducción masiva y promoción productiva siguen sin autorización.**

| Dato comprobado | Resultado e interpretación |
|---|---|
| Muestra del cuarto piloto | 1.000 candidatas recibidas; 100 controles y 900 adicionales |
| Staging aislado | 975 importadas mecánicamente; no equivale a aprobación lingüística |
| Cuarentena | 25 retenidas por atomicidad; revisión lingüística terminada: 22 aprobadas y tres para investigación. No libera el lote |
| Prioridad obligatoria del piloto histórico | 196 casos, incluidos los 42 señalados por auditoría; ese archivo no se modificó |
| Muestra de calidad | 199 casos distintos; unión con obligatorios: 395; otros 605 con prioridad adicional |
| Revisión humana al corte actual | 41 decisiones en la copia activa de evolución: 29 aprobaciones y 12 investigaciones; el archivo humano continúa activo |
| Consumo acumulado del cuarto piloto | US$0,1176624 de US$0,17; 50 respuestas reales, sin llamadas pendientes |
| TM del staging / promoción a producción | 0 / 0; no se crearon aprobaciones humanas automáticas |
| Última suite registrada | 332/332; 24 regresiones nuevas sobre la base 308/308 repetida desde backup |
| Código local | Rama `codex/product-stabilization-20261106`, commit funcional `6a153dd`; conserva la consolidación anterior y se versiona solo localmente |

El coste histórico del cuarto piloto fue calculado con tokens reportados y tarifa registrada; no certifica la factura ni una tarifa vigente. Las 110 pruebas de cierre de aquel piloto reforzaron la forma SHA-256 offline. El consumo y los límites de la nueva ejecución se registran por separado en «Próximos pasos».

## Qué está demostrado y qué no

- Se conservaron respuestas originales, hashes, presupuesto acotado, importación atómica y reutilización de candidatas válidas. La auditoría final registró cero diferencias de placeholders y controles entre las 975 importadas.
- El resultado reúne continuaciones y reparaciones con reglas ajustadas. El 97,5 % de importación es acumulado; no mide acierto semántico ni aceptación inicial.
- En el piloto histórico, el contexto enviado es limitado: no hay indicación de hablante en las 200 cadenas de diálogo; el runner agrupa por tamaño y puede mezclar grupos narrativos.
- Hay una propuesta de formato de IA para un poema, pendiente humana. Está identificada; no es una respuesta intacta del proveedor.
- El análisis halló dos falsos positivos numéricos y 132 casos con solo alerta de nombres/lore. El nuevo validador separa severidades y normaliza cifras; conserva los informes históricos.

## Evolución inteligente verificada offline

La copia experimental reclasifica las mismas candidatas a **38 obligatorias (25 de cuarentena y 13 editoriales), 199 de muestra y 763 adicionales**, sin aprobar o reescribir ninguna. La reducción del 80,61 % es de prioridad, no de tiempo humano o errores semánticos medidos. En ese primer corte había dos referencias para calibración y una para validación; la aprobación automática nueva continúa deshabilitada.

Se implementaron contexto por mod/plugin, contrato 1.2 compatible con lectura 1.0/1.1, propuestas terminológicas separadas, reparación técnica reversible, juez simulado y ejecución por etapas con alcance fijo. Se indexaron 12.423 nombres de registro NPC; no se infieren biografías. El benchmark de 240.684 pasajes sintéticos verificó la ingesta SQLite, no la traducción completa.

La preparación coherente con recuperación de esta muestra dispersa estima US$0,39955632 frente a US$0,16704072 sin recuperación. Excede el cap anterior: la opción no se activó por defecto ni se elevó el presupuesto. Cero solicitudes facturables y 2.186 archivos anteriores verificados idénticos. [Informe y métricas](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/a232c2edc575469e3f029ff99e6eeb4774cf4273/Fase10_Evolucion_Linguistica/Intelligence_20261105/README.md).

La continuación integra revisión independiente → posedición acotada → recomprobación en un ejecutor offline con evidencia persistida. Sobre la copia de 1.000 cadenas selecciona 13 por riesgo y 8 controles; el simulador se abstiene en los 21 casos. Reanudar añade cero invocaciones. Las 25 de cuarentena permanecen aisladas; cero aprobaciones nuevas y 2.225 archivos anteriores idénticos. Falta medir el evaluador real y calibrar. [Detalle funcional y pruebas](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/4c13cb4564e046aad163135bc27629e6ce7dd64f/Fase10_Evolucion_Linguistica/Intelligence_Continuation_20261105/README.md).

La ampliación funcional incorpora un catálogo de **403.922 registros originales de 1.316 mods con textos**, relaciones por registro/grupo/misión y dos extractos documentales de autores con licencia y procedencia. No son 403.922 traducciones ni entidades únicas. Se recuperaron referencias para 987 de las 1.000 cadenas en 9,532 s; la cobertura no acredita pertinencia lingüística. Se conservan íntegros 11 textos largos en 22 segmentos, se reutilizan ocho aprobaciones humanas exactas y se aíslan transporte/uso inciertos sin bloquear otros casos.

El ejercicio final mantiene 21 consultas simuladas inconclusas y 25 casos en cuarentena, sin editar candidatas ni crear aprobaciones. La reanudación agrega cero consultas. Las ocho referencias humanas y los dictámenes simulados no permiten calibrar un juez real. API Lock activo, cero HTTP y cero créditos; la DB productiva conserva su hash. De 2.254 archivos protegidos, 2.253 siguen idénticos; el único cambio externo fue el overlay humano durante la revisión, preservado y congelado como entrada privada. Evidencia local: `NolvusAIT_Intelligence/20261105_Functional/private`.

El barrido local separa atención de aprobación: 16 casos necesitan criterio, 25 permanecen en cuarentena técnica, 207 forman un control opcional, 744 quedan para consulta y ocho conservan referencias aprobadas. De 141 avisos de nombres/lore, 132 no entran en la cola prioritaria. La lectura lingüística focalizada detectó sustituciones Elsweyr/elfa y Steed/Sombra, una variante de familia y etiquetas por comprobar; las observaciones se vinculan a hashes y no cambian candidatas ni decisiones humanas. No constituye certificación semántica de las 1.000 cadenas.

La consolidación preservó las tres ramas anteriores (mismo commit base), registró el trabajo heredado y corrigió recibos/presupuesto, integridad de planes, concurrencia, propuestas terminológicas y reingesta contextual. Las observaciones del visor llegan también a la selección de reevaluación. En aquel corte quedó preparado el experimento acotado, sin ejecutarlo; resultados, límites y comparación offline en el [informe ejecutivo único](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/30db1581792ba69d2d2fd87e7222731a45d9b4f7/Fase10_Evolucion_Linguistica/Consolidation_20261105/README.md). El inventario completo de archivos/hashes es privado.

## Estandarización y continuidad verificadas offline

El nuevo flujo integrado vincula conceptos a ámbito semántico y referencias existentes, separa Luna (traducción/posedición) de Sol (evaluación/recomprobación), congela contexto y términos por ejecución y conserva checkpoints por lote. No impone un término de equipo a otro sentido de la palabra ni convierte propuestas en glosario. La carga sintética procesó 1.000 lotes: 940 validados, 50 en cuarentena y 10 con protocolo fallido; reabrir añadió cero intercambios. La integración programada aisló un lote con hash incorrecto y completó otros dos, con una propuesta recomprobada y cero aprobaciones. No mide calidad ni velocidad real de IA. [Cambios, pruebas y límites](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/terminology-continuity-docs-20261106/Fase10_Evolucion_Linguistica/TerminologyContinuity_20261106/README.md).

Una identidad incorrecta de salida con uso conocido queda aislada por lote; corrupción del diario/fuentes, credenciales/autorización o uso incierto bloquean nuevos envíos del plan. Solo se reintenta con prueba de no envío/no cargo y límite sellado. El perfil nuevo es experimental y exclusivamente offline: no sustituye runners históricos ni reabre la parada del piloto. Cero HTTP/créditos, nueve entradas congeladas idénticas y 41 decisiones previas conservadas. El contraste determinista de las 48 candidatas antiguas señaló seis casos por referencias (cuatro diferencias y dos conflictos), no seis errores confirmados. Falta ampliar cobertura semántica y verificar/calibrar Sol con API autorizada.

## Preparación de la prueba mayor (corte previo)

El barrido previo corrigió el despacho ante un envío incierto tardío, la serialización de términos reales y su validación inicial, la pérdida de metadatos, el límite de revisión y referencias semánticas que podían descartarse por tamaño. Se añadió un ámbito pagado sellado para Luna/Sol, con atribución de coste y sublímites, sin cambiar presupuestos históricos. El ejercicio final de solicitudes exactas completó 117 intercambios programados y reanudó con cero adicionales; no mide calidad real de IA. [Hallazgos y preparación](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/terminology-continuity-docs-20261106/Fase10_Evolucion_Linguistica/HeavyTrial_20261106/README.md).

El plan cerrado comprendía **1.050 fuentes: 950 candidatas conservadas y 100 traducciones nuevas**, de 182 mods/orígenes. Sol evaluará **14 casos naturales**, más ocho controles sintéticos; hasta dos correcciones Luna con recomprobación Sol y dos auditorías adicionales. Sol no revisará toda la población. Tope propuesto para ejecutar: **US$0,30**, repartido en US$0,08 Luna y US$0,22 Sol. El escenario orientativo previo a correcciones/auditorías es US$0,234024; no garantiza cobertura completa y los caps detienen etapas que no puedan reservar la siguiente llamada. La ejecución posterior recibió autorización y quedó cerrada por cuarentena económica; véase el resultado real siguiente. `Validar_prueba_pesada.bat` no envía HTTP. El borrador de 1.200 no es vigente.

## Última ejecución real Luna/Sol

La ejecución autorizada del cap US$0,30 hizo **nueve HTTP únicos**: ocho controles Sol completados y una respuesta Luna incompleta/vacía, con motivo `max_output_tokens` y uso reportado cero. Sol superó los ocho controles y consumió **US$0,023968** según tokens/tarifa; **US$0,00249593** de reserva Luna quedaron retenidos, sin presentarlos como cargo confirmado. La parada económica bloqueó nuevos envíos. Se conservan 950 candidatas; hubo cero traducciones nuevas y cero revisiones naturales, posediciones o recomprobaciones. No se midió mejora lingüística. [Recibos, incidencia y resultado parcial](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/terminology-continuity-docs-20261106/Fase10_Evolucion_Linguistica/HeavyTrial_Run_20261106/README.md).

Antes de esos HTTP se corrigió un bloqueo local del transporte que aún restringía el modelo a Luna; ese intento no envió HTTP ni consumió. Se probó la ruta real con servidor simulado y se conservó el intento anterior. La reserva incierta requiere conciliación para resolver aquel diario; no se recomienda elevar salida ni repetir la prueba por una respuesta de uso cero. API Lock activo y siete entradas protegidas idénticas. Los controles Sol siguen siendo una comprobación mínima, no una aprobación del corpus.

## Próximos pasos

La prueba pareada **se ejecutó con API y quedó detenida por su condición crítica sellada**: 24 fuentes × dos contextos, 48 traducciones recibidas, 47 válidas y una con hash incorrecto. Los ocho controles sintéticos del juez pasaron; no se alcanzó la revisión ciega de las traducciones naturales, posedición ni recomprobación. Fueron 50 llamadas únicas, sin reenvíos, con **US$0,01524421** conciliados frente al cap autorizado de US$0,35; reserva y uso incierto: cero. El API Lock está activo y la parada persiste tras reiniciar. [Resultado, consumo y límites](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/technical-consolidation-docs-20261105/Fase10_Evolucion_Linguistica/PairedTrial_Run_20261105/README.md).

Se corrigieron offline el contrato de hash del decodificador, la vinculación exacta de identidades en el esquema, la idempotencia de la política y el cómputo de caché. Las 47 candidatas se conservan; el hash incorrecto no se reparó ni aceptó. El esquema corregido aún no tiene confirmación por API. Las referencias mejoraron tres denominaciones de lugares, pero también se observaron nombres ingleses retenidos y una frase incompleta: no hay evidencia para declarar superioridad general del contexto ni reducir más revisión humana. Visor separado: `Revisar_prueba_pareada.bat`; no modifica las 41 decisiones previas.

La ampliación documental descubrió 41 documentos locales en 36 de los 1.302 mods con texto pendiente y registró 1.220 páginas candidatas, todavía sin verificar individualmente. Se examinaron diez fuentes externas de siete mods, con versiones y resúmenes analíticos identificados; se corrigió la selección para que documentos relevantes no desaparezcan entre registros. De los 32 casos señalados: 16 diferencias de denominación, un cambio de identidad, 11 alertas que no establecen error y cuatro pendientes de contexto. La propuesta de El Corcel está aislada; tus decisiones siguen intactas. [Informe documental y límites](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/technical-consolidation-docs-20261105/Fase10_Evolucion_Linguistica/Documentation_20261105/README.md).

La coherencia entre registros ya incorpora nombres localizados con identidad/procedencia, piezas relacionadas y contraste en revisión/posedición. Sobre las mismas 1.000 cadenas recupera referencias para 483 y señala 32 posibles diferencias/conflictos, **no 32 errores confirmados**. No impone preferencias pendientes ni fabrica nombres faltantes. La reanudación añade cero invocaciones al simulador; cero HTTP, créditos y aprobaciones nuevas. Se preservaron 1.719 archivos anteriores del alcance protegido y las 41 decisiones. [Resultados y límites](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/technical-consolidation-docs-20261105/Fase10_Evolucion_Linguistica/Coherence_20261105/README.md).

| Orden | Trabajo pendiente | Condición para avanzar |
|---|---|---|
| 1 | Microdiagnóstico independiente del nuevo perfil | Autorización específica: dos Luna y un Sol, hasta US$0,05; no reabre el diario detenido |
| 2 | Aclarar el recibo económico incierto y conservar cuarentenas históricas | Evidencia del proveedor; no liberar US$0,00249593 ni reenviar su fuente |
| 3 | Adjudicar las parejas conservadas | Evaluación ciega; distinguir validación mecánica, naturalidad y fidelidad |
| 4 | Completar y calibrar el juez sobre casos naturales | Ocho controles pasados no miden falsas aprobaciones ni ahorro humano |
| 5 | Validación natural y escalamiento gradual | Perfil confirmado por API y juez calibrado; preservar candidatas y decisiones, sin repetir todo |

No se recomienda aumentar carga sin calibración y presupuesto compatible. El transporte real pasó los controles sintéticos; calidad del juez sobre traducciones naturales, calibración, escalamiento y producción siguen pendientes. La segmentación automática exige límites compartidos fiables; un párrafo demasiado largo o desalineado queda retenido sin truncarse. Los cambios de esta fase se proponen en una PR solo documental; el código permanece privado.

## Fuentes del corte

- [Ejecución completa y consumo](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/main/Fase10_Evolucion_Linguistica/Pilot4_Completed_20261104/README.md).
- [Análisis profesional: alertas, falsos positivos y recomendaciones](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/main/Fase10_Evolucion_Linguistica/Pilot4_Review_Analysis_20261105/README.md).
- Evidencia local: `NolvusAIT_Pilot4_Sweep/20261104/final_metrics.json`, `full_tests_after_hash_fix.txt` y `NolvusAIT_PilotStaging/Phase4_Luna1000_R7_DiagnosticComplete`.

El inventario de ~240.684 cadenas es una estimación histórica reconciliada; no se ha vuelto a escanear Nolvus para actualizarla. Las cifras iniciales de otras fases se consultan en el histórico, no se suman a la muestra actual.

Sol queda limitado como validador selectivo: muestreo ordinario del 2 % y hasta 14 revisiones iniciales por ejecución, incluidas dudas priorizadas. Luna mantiene traducción y posedición; no se encadenan revisiones generales de toda la población. El antecedente cerrado conserva sus 14 casos y su cap US$0,30.

## Estabilización funcional actual

Se unificaron recibos/telemetría, se añadió contrato compacto con identidades locales y razonamiento acotado, y se corrigieron intención de envío, reanudación, contexto TM/diferencial, importación protegida y exportación reversible. La carga sintética de 1.000 fuentes aisló cuatro lotes defectuosos y conservó/importó 900 candidatas pendientes; reanudar añadió cero llamadas. No mide calidad lingüística. 68/68 archivos protegidos y 41 decisiones conservados; cero HTTP facturables, créditos o aprobaciones nuevas. [Cambios implementados y límites](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/terminology-continuity-docs-20261106/Fase10_Evolucion_Linguistica/Stabilization_20261106/README.md).

Quedó preparado y simulado el microdiagnóstico, con cap propuesto US$0,05 y reserva máxima US$0,0469568 dentro del margen conservador previo. La reserva incierta anterior sigue retenida y su causa no está probada. Faltan autorización específica y validación real del perfil; no se declara producción lista.
