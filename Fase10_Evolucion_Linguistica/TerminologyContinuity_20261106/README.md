# Terminología contextual y continuidad de procesos — 2026-11-06

Se implementaron ambos desarrollos y se probaron conjuntamente **offline**. El nuevo flujo integra traducción Luna, evaluación Sol, corrección Luna y recomprobación Sol, con términos identificados por ámbito y checkpoints por lote. No habilita transporte real, aprobación automática ni importación productiva. No se cambió la condición de parada de la prueba pareada anterior.

## Cambios funcionales

| Componente privado | Problema resuelto y comportamiento |
|---|---|
| `lore/mod_knowledge.py` | Añade metadatos de uso a los términos del sidecar existente: identidad del concepto, tipo, registros, campos y formas explícitas. Un término de equipo se aplica al nombre de equipo correspondiente o a una referencia semántica explícita; no se propaga a una palabra homónima en otro sentido. Las propuestas pendientes no obligan ni se convierten en entradas del Glosario Maestro. |
| `lore/consistency.py` | Reutiliza el registro de nombres para ubicaciones, NPC, facciones, objetos y magia, y extiende referencias a títulos de misiones/libros. Contrasta conceptos sin reemplazar palabras y conserva concordancias, familias, procedencia y conflictos. La validación local examina todas las referencias recuperadas, incluso las que exceden los cuatro fragmentos del prompt. |
| `translation/independent_review.py` | Perfil nuevo Luna/Sol: Sol revisa y verifica; Luna conserva traducción y posedición. El modelo y la política se vinculan al hash del paquete. El esquema restringe el binding al valor exacto esperado. Los contratos de dictamen siguen separando error fundamentado, incertidumbre y ausencia de hallazgos; nunca conceden aprobación. |
| `translation/review_workflow.py` | Integra los roles anteriores, validación estructural/terminológica/contextual de propuestas y reservas offline por etapa/modelo. Si no cabe la revisión, conserva la candidata y registra el aplazamiento; no aumenta el presupuesto. Los perfiles nuevos son explícitos para no reinterpretar diarios antiguos. |
| `translation/durable_run.py` | Política de continuidad nueva y sellada, con incidentes por lote, dependencias, intentos acotados, estados persistentes e informe completo. Una respuesta defectuosa con uso conocido se aísla; credenciales, autorización, corrupción del diario o uso incierto bloquean nuevos envíos del plan. Se revisan todos los hashes almacenados antes de despachar trabajo. |
| `translation/batch_workflow.py` | Une contratos de traducción existentes y diario con revisión/posedición/recomprobación. Acepta solamente backends simulados con API Lock. Congela fuentes, referencias, términos y roles; rechaza fuentes duplicadas y particiones alteradas. La validación de cada lote es atómica: una cadena defectuosa retiene el lote completo, sin perder los demás. Exporta resultados y estados a una carpeta nueva. |
| `translation/paid_trial.py` | Comprueba también la cardinalidad de las fuentes originales antes de decodificar; evita aceptar parcialmente una respuesta mediante un `zip` truncado. No reabre ni ejecuta el piloto pagado. |

No se crearon un glosario paralelo, otra TM ni un segundo gestor económico. Se extendieron el sidecar, el registro de referencias y el diario existentes. La corrección de IA queda como candidata trazable con original, dictamen, contexto, modelo, propuesta y recomprobación; no alimenta automáticamente términos aprobados ni TM.

## Terminología: alcance y ejemplos comprobados

- Una ubicación conserva la referencia «Mina Panza Roja» en diálogo, HUD, misión y libro. «Despeja» y «Elimina las arañas» siguen siendo variantes válidas; «mina Redbelly» se señala como diferencia de denominación cuando existe esa referencia.
- En las fixtures con términos expresamente aprobados, `Boots` como categoría de armadura exige «Botas» y `Helmet` exige «Casco». La misma palabra como título de libro, diálogo sin identidad semántica o texto de otro mod no hereda esa obligación.
- La aparición en prosa puede vincularse mediante `concept_ids` aportados por la recuperación contextual. El sistema no fabrica esa identidad por compartir una palabra. Los nombres sin denominación de referencia quedan sin resolver; no se inventa una traducción canónica.
- Las familias existentes conservan sus formas de concordancia. La metáfora «El silencio nos abrumaba» no se modifica para uniformar palabras. Tampoco se reescriben verbos ni estilo por coincidencias léxicas.
- Las referencias del autor siguen disponibles junto con nombres/conceptos. La validación completa no se limita a los cuatro fragmentos transmitidos; una propuesta que deje una diferencia de referencia no pasa la puerta de posedición.

Las fixtures terminológicas son sintéticas y aisladas. **No se añadieron «Botas», «Casco» ni ninguna otra preferencia al glosario o sidecar aprobado del usuario.** Los términos heredados sin metadatos semánticos conservan su comportamiento por compatibilidad; falta ampliar su cobertura con referencias fiables. No se promete resolver automáticamente todos los sentidos ni todo el lore del inventario.

## Política de fallos implementada

| Incidente | Acción en el nuevo flujo |
|---|---|
| Hash/UID de salida incorrecto, estructura o respuesta defectuosa, uso conocido | Retener el lote/etapa, conservar solicitud y recibo, registrar causa y continuar los trabajos independientes. No reenviar ni importar parcialmente. |
| Dependencia de un lote no validado | Mantener pendiente con `blocked_by`; no consumir una salida retenida. |
| Interrupción local después de conservar el recibo | Revalidar el recibo al reabrir, sin repetir la solicitud ni sumar otra vez su coste simulado. |
| Envío o uso incierto, timeout, credenciales/autorización inseguras | Bloquear nuevos envíos del plan, conservar la reserva conservadora y los resultados conocidos. No inferir que un timeout significa «no enviado». |
| Corrupción de recibo/solicitud almacenados o plan alterado | Rechazar la ejecución antes de nuevos despachos; exige resolver la integridad. |
| Coste reportado superior a la reserva | Conservar el recibo/coste y bloquear el dominio económico del plan. |
| Fallo que el adaptador acredita como no enviado y sin cargo | Único caso con reintento automático: límite predeterminado cero, configurable entre cero y dos en un plan nuevo. Al agotarse, registrar el fallo y continuar lo independiente. |
| Presupuesto de revisión insuficiente | Conservar la traducción válida y aplazar su revisión; no elevar el cap ni confundirla con una aprobación. |

La distinción de identidad es deliberada: una **identidad incorrecta en una respuesta recibida** puede aislarse por lote cuando la solicitud y el uso son fiables. Una identidad o integridad incorrecta en las **fuentes, solicitudes selladas o diario compartido** impide continuar con seguridad. La prueba pagada del 2026-11-05 conserva su parada más estricta y no se reabre con esta política.

## Resultados medidos

| Comprobación | Resultado |
|---|---|
| Suite completa | **298/298**; incluye 22 regresiones nuevas y las 276 anteriores |
| Carga del diario sintético | 1.000 lotes: **940 validados, 50 en cuarentena y 10 con protocolo fallido**; 25,010 s en este equipo |
| Reanudación de esa carga | **0 intercambios adicionales**; las causas y los resultados quedaron conservados |
| Integración con respuestas programadas | Tres lotes de traducción: uno retenido por hash y dos válidos; siete intercambios simulados contando revisión, corrección y verificación; una propuesta corregida, cero aprobaciones |
| Reanudación integrada | **0 intercambios adicionales**, mismos resultados |
| Candidatas reales conservadas | Contraste de solo lectura sobre 48: 47 mecánicamente válidas y una retenida. Seis casos con avisos de referencia: cuatro diferencias y dos conflictos; no equivalen a seis errores confirmados |
| Protección | Nueve entradas congeladas con hashes idénticos, incluidas DB principal, casos activos, decisiones humanas y preparación pareada |
| API y crédito | **0 HTTP, 0 credenciales cargadas, US$0 facturados; API Lock activo** |
| Producción, TM, glosario aprobado y decisiones previas | Cero escrituras o cambios |

La carga de 1.000 lotes mide diario, aislamiento y recuperación; **no mide velocidad de traducción real ni calidad lingüística**. Los intercambios de integración son respuestas programadas que ejercitan las rutas Luna/Sol, no ejecuciones de esos modelos. El contraste de las candidatas anteriores es determinista, no un dictamen nuevo de Sol o del usuario. Las pruebas de red se bloquearon expresamente.

## Trazabilidad, operación y pendientes reales

Código privado en `codex/terminology-continuity-20261106`, commit `a1c921dc88f603e993dff9686483acd2effce90e`. La rama anterior continúa en su commit previo. Las adiciones del sidecar son compatibles con lectura de catálogos históricos; no se migró ningún catálogo productivo. El nuevo perfil de revisión es `independent-review-2.3`; los lectores conservan dictámenes anteriores, pero una preparación nueva no puede cambiar silenciosamente el fingerprint de un diario sellado.

En un procesamiento masivo futuro se preparará un ámbito nuevo, con referencias congeladas, modelos y reservas por etapa. Cada lote avanzará por sus checkpoints; sus dependencias solo usarán resultados validados. Una respuesta aislada defectuosa no descartará lotes correctos. Si falla la seguridad compartida, se conservará lo obtenido y se detendrán nuevos envíos. Al terminar, el informe distinguirá resultados válidos, protocolos fallidos, cuarentenas, dependencias y bloqueos, con sus causas.

La implementación integrada está disponible como flujo experimental offline; **no sustituye por defecto los runners históricos ni conecta un transporte pagado**. Queda verificar el adaptador real Luna/Sol y sus reservas bajo una autorización futura específica, ampliar identidades/conceptos con evidencia y medir calidad/falsas aprobaciones con adjudicación independiente. No requiere repetir las 47 traducciones válidas, modificar el glosario ni habilitar aprobación automática.

Evidencia privada: `NolvusAIT_Intelligence/20261106_TerminologyContinuity/private`: logs de la suite, hashes previos, runner offline, diario de carga, informe por lote, flujo integrado y contraste de candidatas conservadas. El informe y `METRICS.json` son agregados publicables; los diarios, código, corpus y decisiones individuales se conservan locales. La reversión se realiza volviendo al commit/rama anterior y descartando solo la selección del perfil nuevo; no exige una migración destructiva ni borrar evidencia.
