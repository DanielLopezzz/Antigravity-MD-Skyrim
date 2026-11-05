# Consolidación técnica y optimización — 2026-11-05

**Implementado y verificado offline.** API Lock activo, cero HTTP a proveedores, cero créditos, cero traducciones nuevas y ninguna modificación productiva. El siguiente experimento está preparado, sin ejecutar. La revisión humana del visor puede continuar.

## Línea de trabajo preservada

Las ramas locales `codex/linguistic-evolution`, `codex/independent-review-flow-20261105` y `codex/contextual-recovery-20261105` apuntaban todas a `066ae03`. Sus nombres no representaban tres evoluciones comprometidas independientes: compartían los cambios sin commit del mismo directorio.

Se creó **`codex/technical-consolidation-20261105`**, sin merges ni resets. `12f1125` registra el código/pruebas/lanzadores heredados; **`b68c7dd`** registra la consolidación funcional. Las tres referencias anteriores permanecen intactas. Las versiones previas también tienen snapshots privados y los informes cerrados no se alteraron. El código no se publicó.

## Qué se limpió, integró y corrigió

- **Artefactos:** hash en bloques y publicación atómica con temporales únicos, flush y fsync. El visor, el ejecutor experimental y los workbenches comparten estas primitivas; se retiraron implementaciones duplicadas y sus imports asociados.
- **Contratos:** el puerto de revisión antiguo y el flujo estructurado usan los mismos modelos y comprobación de fragmentos. Un nombre de categoría inventado ya no pasa por el puerto antiguo. Se conservó su interfaz pública compatible.
- **Glosario:** selección compartida de términos aprobados en proveedor, cola y posedición. Una propuesta global no se convierte en obligación; tampoco desplaza un término aprobado. No se modificó el glosario ni la TM productivos.
- **Contexto incremental:** una firma por registro evita reescribir pasajes/FTS sin cambios. Una transacción abarca el flujo, incluidos los registros que solo se consultan. La reingesta de un documento idéntico no reescribe sus fragmentos; un nombre de NPC sustituido no reaparece como evidencia vigente.
- **Diario durable:** se verifican conjunto, etapa, payload y reserva contra el plan sellado, incluso al reanudar. También se comprueba la integridad de recibos terminales. Un bloqueo por diario evita ejecutar dos veces el callback de validación concurrente.
- **Proveedor y presupuesto:** el adaptador conserva el HTTP original en el almacén aislado antes de interpretar o conciliar. Los errores de servidor/timeout o respuestas con uso reportado se concilian o quedan en cuarentena económica; no se supone coste cero ni se reintenta automáticamente un 5xx incierto. Los rechazos de cliente sin uso mantienen su tratamiento acotado. La idempotencia del libro mayor sigue bloqueando cargos duplicados.
- **Cola y evaluador:** una advertencia de puntuación menor permanece como control informativo y no dispara sola una evaluación. Las observaciones de esta sesión pueden alimentar la selección del flujo, con hashes de fuente/candidata/contexto; su dictamen no se copia al prompt independiente.
- **Revisión concurrente:** si alguien inicia una revisión entre preparación y publicación, se conservan casos y decisiones del visor. El bloqueo humano solo se toma al publicar, nunca mientras se espera al evaluador.

Se hizo un barrido estático de **89 módulos** y se revisaron manualmente los límites críticos modificados. Se conservaron APIs históricas como `LocalReferenceIndex`; sustituir su esquema sin necesidad habría puesto en riesgo compatibilidad. No se eliminaron archivos o interfaces solo por parecer antiguos. El detalle privado contiene inventario, hashes y límites de cobertura.

## Pruebas y comparación

Suite inicial: **193/193**, 48,601 s. Suite final: **212/212**, 72,155 s; **19 regresiones nuevas**. Incluye integridad del plan/recibo, validación concurrente única, fallos de publicación, ingesta idempotente, términos propuestos, respuesta persistida antes de fallar la conciliación, 5xx sin reenvío, integración de observaciones y preservación de revisión humana concurrente. También pasaron las regresiones existentes de TM, glosario, presupuesto, formatos y exportación aislada.

| Medición offline | Antes | Después |
|---|---:|---:|
| Ingesta inicial de 240.684 registros sintéticos | 59,827 s | 70,422 s |
| Reingesta de los mismos registros sin cambios | 113,464 s | 6,926 s |
| Registros contextuales reescritos en esa reingesta | Todos | 0 |
| Recuperación de 100 consultas de esa muestra | 9,951 s | 7,102 s |
| Pico de working set del proceso de ingesta | 29.114.368 bytes | 28.762.112 bytes |
| Pico del proceso al hashear un archivo sintético de 64 MiB | 85.471.232 bytes | 21.487.616 bytes |

Los dos catálogos terminaron con `integrity_check=ok`, el mismo número de registros y **los mismos 100 resultados de recuperación, comprobados por SHA-256**. El hash del archivo sintético también coincide. Se midió working set real de Windows, no únicamente asignaciones Python.

La carga inicial **no se aceleró**: calcular/persistir firmas añade trabajo y espacio. El beneficio principal está en no reconstruir contenido ya indexado y en limitar memoria de hashing. El hash en bloques tardó 0,067 s frente a 0,057 s: su mejora es memoria, no velocidad. Los tiempos varían en este entorno compartido; los ensayos preliminares de reingesta dieron 53,536 s y 5,679 s. No se extrapolan estos números al rendimiento de toda la app, la traducción del inventario ni a calidad lingüística. La suite final tampoco demuestra una reducción de tiempo de pruebas.

Se verificaron **2.311 archivos anteriores idénticos** al corte de protección, incluidos pilotos, revisiones, producción y el catálogo anterior. Los ensayos usaron bases nuevas y datos sintéticos. Las revisiones humanas posteriores al corte siguen siendo actividad del usuario y no se revierten.

## Próxima validación: diseño preparado, no ejecutado

El plan privado contiene **12 casos naturales**, con seis perfiles y al menos seis mods, más **cuatro controles sintéticos** de cantidades, negación, entidad y adición. Compara ausencia de contexto, contexto recuperado y distractor del mismo ámbito. No se da por demostrada la pertinencia de lo recuperado.

Hay 48 comparaciones lógicas; los brazos idénticos reutilizan un recibo. Se prepararon **40 solicitudes únicas de revisión**, más un máximo de cuatro posediciones y cuatro recomprobaciones: **48 solicitudes máximas**. Las solicitudes y el corpus están únicamente en el archivo privado.

- Entrada heurística de las revisiones preparadas: **46.181 tokens**; límite conservador total: **614.400 de entrada y 98.304 de salida**, incluyendo el margen de solicitudes de reparación. Hasta 12.288 bytes por solicitud y 2.048 tokens de salida por paso.
- Reserva máxima estimada con tarifa local registrada y margen del 15 %: **US$0,12718080**, bajo un tope propuesto de **US$0,15**. No es una cotización vigente ni una reserva/cargo real; verificar tarifa antes de autorizar gasto.
- Antes de ejecutar: etiquetas humanas independientes, revisión del adaptador de ejecución real y autorización específica de API. La aprobación automática continúa deshabilitada, cualquiera que sea el resultado de esta muestra pequeña.
- Aceptación exploratoria: al menos 10/12 conjuntos de contexto natural útiles, ninguna evidencia fuera del ámbito, detección de los cuatro errores controlados, cero errores mayores/críticos introducidos al corregir y medición pareada de alarmas falsas/correcciones/tiempo humano. Resultados naturales y sintéticos se informarán separados.
- Parada: desviación de presupuesto/modelo/herramientas, uso incierto, fallo crítico de identidad/estructura o más del 10 % de respuestas malformadas, rechazadas o incompletas. Conservar recibos y no retransmitir por defecto.

Una invocación ciega independiente del traductor no garantiza independencia estadística si usa la misma familia de modelo. El [contrato estructurado oficial](https://developers.openai.com/api/docs/guides/structured-outputs) orienta la validación de forma y el tratamiento de respuestas excepcionales; no demuestra fidelidad semántica.

## Pendientes reales

Faltan etiquetas humanas representativas, evaluación real de relevancia y del juez, calibración de falsas aprobaciones/negativos y validación del adaptador facturable antes de conectarlo a una ejecución autorizada. Continúan retenidos los 25 casos históricos con problema de identidad; no se corrigieron sus recibos inventando hashes. Los párrafos sin alineación fiable siguen retenidos. No se ha medido mejora semántica, naturalidad ni ahorro humano real en esta fase.

GitHub recibe únicamente este informe, métricas agregadas, plan resumido y evidencia agregada de cambios, mediante una rama/PR documental independiente. El inventario privado completo está en `private/change_manifest.json`; datos, previews, bases y snapshots permanecen locales.
