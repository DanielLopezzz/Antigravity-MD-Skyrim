# Barrido funcional y preparación de prueba Luna/Sol — 2026-11-06

Preparación terminada, **sin ejecutar la API ni cargar credenciales**. Se revisaron contratos, glosario/contexto, presupuesto, recuperación, selección y exportación de la evaluación. Se corrigieron problemas demostrados y se construyó un plan sellado con transporte real condicionado a una autorización nueva. API Lock activo; producción, Nolvus, TM, glosario aprobado, pilotos y decisiones humanas permanecen intactos.

## Problemas corregidos y verificación

| Hallazgo | Cambio funcional comprobado |
|---|---|
| Un envío incierto situado al final del diario podía descubrirse después de despachar otros trabajos | La política nueva comprueba todos los estados inciertos antes del primer despacho. La regresión deja `z` en envío incierto y verifica que `a`, todavía pendiente, no se envía. |
| El flujo integrado aceptaba traducciones sin comprobar todo el glosario congelado | La validación inicial aplica los términos aprobados y pertinentes, incluidos identificadores protegidos. Un incumplimiento retiene todo el lote. Un término ajeno a la fuente no se fuerza. |
| Con términos reales, la serialización podía fallar por tratar una dataclass como modelo Pydantic | Se serializan como dataclasses y se restauran sus enumeraciones. La regresión usa un glosario no vacío; el caso que antes fallaba ahora llega a su puerta terminológica. |
| Cambio de backend o pérdida de metadatos entre traducción y revisión | El plan vincula la identidad del backend y conserva misión, tema, hablante, grupo narrativo y contexto de script. Se bloquea un cambio de backend antes del envío. |
| El selector conservaba límites de 32/64 casos en el perfil nuevo | La selección ampliada permite hasta 2.000 unidades en perfiles explícitos nuevos, sin cambiar límites históricos. Se probaron 80 revisiones y reanudación sin llamadas adicionales. |
| Una referencia semántica demasiado extensa podía quedar fuera silenciosamente | La configuración del ámbito rechaza referencias que excedan su envoltura, antes de guardarlas. No se pierde la restricción sin aviso. |
| El ejecutor pagado anterior solo admitía Luna y un presupuesto fijo del piloto | Se reutiliza su transporte/diario/libro económico con un ámbito sellado opcional para Luna y Sol. Los pilotos anteriores conservan sus límites y autorización; una raíz histórica no puede adoptar el ámbito nuevo. |
| Reconciliación y atribución del coste de la segunda IA | Se admiten las tarifas registradas de caché de Sol y se entregan llamadas, tokens y costes conciliados por modelo. Se reservan importes por solicitud exacta y se aplican también sublímites por modelo. |

Se reforzó además la correspondencia exacta de modelos: un nombre que solo comparta el prefijo no pasa como respuesta del modelo solicitado. La preparación conserva solicitudes, fuentes, términos, referencias y hashes; al ejecutar se comprueba integridad completa al inicio/final y metadatos entre llamadas, evitando releer la DB completa para cada solicitud. Los fallos de protocolo de una evaluación se distinguen de una alteración del plan o de un fallo de transporte; estos últimos detienen el envío.

La implementación reutiliza los componentes existentes. El nuevo `heavy_trial` coordina la prueba acotada y su comparación; no crea una TM, glosario ni libro económico productivo alternativos. No se hicieron cambios en parsers o exportadores de plugins que esta prueba textual no utiliza.

## Alcance final y cobertura real

| Elemento | Plan preparado |
|---|---|
| Fuentes únicas en el alcance | **1.050**, procedentes de **182 mods/orígenes** |
| Candidatas reutilizadas | **950**; no se vuelven a pedir a Luna |
| Traducciones nuevas con Luna | **100**, en **93 solicitudes** coherentes y acotadas |
| Revisión natural de Sol | **14 casos**: siete reutilizados y siete nuevos; cubre siete perfiles en cada grupo |
| Controles sintéticos de Sol | Ocho: cuatro errores sembrados y cuatro formulaciones aceptables |
| Corrección y recomprobación | Hasta dos propuestas de Luna, cada una con validación local y nueva comprobación de Sol |
| Auditorías ciegas adicionales | Hasta dos consultas de Sol, sin mostrarle el dictamen anterior |
| Máximo de solicitudes | **121**; no es un compromiso de consumirlas todas |
| Perfiles de las 1.050 fuentes | 159 libros, 190 diálogos, 155 objetos, 160 magia, 161 misiones, 113 scripts y 112 interfaz |
| Aprobaciones automáticas / promoción productiva | Cero / cero |

La selección excluye las 41 decisiones humanas activas y los casos técnicamente retenidos; conserva fuentes originales exactas y candidatas históricas válidas. Las referencias se recuperan del catálogo ampliado y del registro de nombres; no se descargaron traducciones comunitarias ni se alteraron preferencias aprobadas.

**Sol no revisará las 1.050 cadenas.** La cobertura natural prevista es 14/1.050; los ocho controles sintéticos y las consultas repetidas no aumentan ese número de casos. La carga de fuentes supera ligeramente el piloto de 1.000, pero se reutiliza trabajo válido para respetar el presupuesto. Esta muestra permite observar coste y casos de mejora/regresión; no permite certificar toda la población ni calibrar aprobaciones automáticas.

## Presupuesto confirmado para dimensionar

El límite solicitado fue **US$0,30**, no US$30. Se prepara como cap nuevo y aislado, pendiente de autorización de ejecución: **US$0,08 para Luna y US$0,22 para Sol**, con tope conjunto US$0,30. No se modificaron presupuestos de pilotos anteriores.

Los previews dan un escenario orientativo de **US$0,234024 antes de correcciones y auditorías adicionales**: US$0,060624 para traducción Luna, US$0,120118 para las 14 revisiones naturales y US$0,053282 para los ocho controles de Sol. Usa una heurística de tokens de entrada y cantidades supuestas de salida, incluido razonamiento: 768 por solicitud de traducción, 512 por revisión natural y 384 por control. No es un precio garantizado, no descuenta caché y no incluye escrituras de caché ni las etapas opcionales.

La tarifa estándar consultada es US$0,10/0,50 por millón de tokens de entrada/salida de Luna y US$2/10 de Sol; se registran aparte lectura y escritura de caché. [Luna: especificaciones y tarifas](https://developers.openai.com/api/docs/models/gpt-6-luna), [Sol: especificaciones y tarifas](https://developers.openai.com/api/docs/models/gpt-6.1-sol).

Cada llamada debe reservar su cota conservadora con el máximo de salida y margen del 15 % antes de enviarse. **El cap prevalece sobre la cobertura**: si el modelo no puede reservar la siguiente solicitud, se aplaza esa etapa, se guardan los resultados y no se amplía el presupuesto. Un promedio supuesto no sustituye esa reserva; disponer del escenario de US$0,234 no garantiza completar todas las correcciones o auditorías dentro de US$0,30.

El primer diseño tenía 1.200 fuentes y 24 revisiones; su escenario orientativo ya alcanzaba US$0,416663 antes de correcciones. Se conservó como borrador privado y se redujo el alcance para respetar la preferencia económica. No se publica como plan vigente ni se confundió US$0,30 con US$30.

## Flujo y evaluación de calidad

1. Verificar fuentes, código, sello, modelos y límites; conservar el lock antes de autorizar cualquier envío.
2. Ejecutar los ocho controles de Sol. Un control fallido detiene la prueba antes de pedir traducciones nuevas.
3. Traducir solo las 100 fuentes nuevas con Luna; validar cada lote completo. Un fallo recibido con uso conocido se aísla, sin reenvío automático ni aceptación parcial. Una desviación repetida activa el umbral de parada del plan.
4. Evaluar con Sol los 14 casos fijados de antemano que tengan candidata válida. Solo se consideran para corrección hallazgos mayores/críticos fundamentados, evitando preferencias estilísticas y ambigüedades.
5. Luna propone hasta dos correcciones. Se contrastan estructura, glosario, nombres, cifras y contexto; una propuesta insegura o sin cambios no consume una recomprobación. Sol verifica las propuestas seguras; quedan como propuestas, no aprobaciones.
6. Guardar originales, propuestas, dictámenes, cobertura y coste por modelo. Exportar hasta 28 versiones (14 pares) en una copia de revisión ciega separada, junto con la correspondencia privada de variantes.

En el visor pueden repetirse los originales por las dos versiones del par. Esa repetición es deliberada para comparar antes/después sin identificar la variante. Los pares sin cambios también sirven como control; un retoque de estilo no cuenta automáticamente como mejora. La evaluación humana debe distinguir fidelidad semántica, naturalidad ES-419, voz, coherencia terminológica y regresiones. La auditoría adicional de Sol sigue siendo una consulta del mismo modelo y no constituye una referencia independiente de verdad.

## Pruebas y estado de entrega

- **307/307 pruebas** en la suite completa; nueve regresiones nuevas. La exportación final del visor se verificó además con las nueve pruebas focalizadas.
- El recorrido final de solicitudes exactas usó **117 intercambios programados**, terminó y reanudó con **cero intercambios adicionales**. Obtuvo la cobertura de 14 casos y conservó los 1.050 resultados del ejercicio. Es material de simulación que solo prueba contratos/rutas; no son nuevas traducciones ni dictámenes lingüísticos reales.
- **Cero HTTP facturable, cero credenciales cargadas y US$0 consumidos** en esta preparación. API Lock activo. Las siete entradas protegidas conservaron sus hashes, incluidas la DB principal, catálogos, casos, decisiones y estado del piloto anterior.
- Código privado: rama `codex/heavy-trial-preflight-20261106`, commit `3fdfd7016b16fad3c96121942730f20adddf0cc6`. El plan definitivo está en `private/prepared_final_v4`, sello `e51a285c30847d0cda4fe712b6192babc406feca6c38492cef6580bb2165757f`. Los borradores se conservan y no deben ejecutarse.

`Validar_prueba_pesada.bat` comprueba el plan sin API. El ejecutor privado exige autorización explícita y el flag de facturación; los resultados reales irán a `private/live`, separados de la preparación y del ejercicio simulado. El estado `execution_ready` indica que existe el ejecutor y los contratos están preparados/probados offline; no confirma permiso de cuenta para Sol, éxito futuro o garantía de completar la cobertura con el cap.

Quedan para la ejecución autorizada comprobar acceso/tarifa efectiva del proveedor, medir tokens y consumo reales por modelo y obtener las traducciones/dictámenes antes de evaluar su mejora. El informe, `SUMMARY.json` y `OFFLINE_CHECK.json` son agregados públicos; código, DB, corpus, solicitudes/respuestas completas y decisiones individuales permanecen privados.

## Política selectiva de Sol

Luna mantiene traducción y corrección. Sol actúa como segundo validador en una muestra ordinaria del 2 % y en dudas fundamentadas, compartiendo un máximo de 14 revisiones iniciales por ejecución. Se corrigió el valor por defecto del flujo integrado que podía seleccionar todas las candidatas. La regresión con 80 casos ordinarios solicita solo dos evaluaciones Sol. El plan v4 conserva exactamente los cuatro artefactos de solicitudes/datos de v3, sus 14 casos y sus caps económicos; solo cambia el sello para vincular el código actualizado. El replay v4 repite el resultado técnico de 117 intercambios programados y cero adicionales al reanudar, sin API ni calidad semántica medida.
