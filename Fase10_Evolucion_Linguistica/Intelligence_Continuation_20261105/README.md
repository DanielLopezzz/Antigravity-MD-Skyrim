# Continuación funcional: reevaluación independiente, posedición y reanudación

**Corte: 2026-11-05.** Implementación privada y ejecución offline; API Lock activo, cero solicitudes HTTP y US$0 consumidos. Este informe continúa la evolución anterior sin repetir la auditoría ni reorganizar documentación.

## Estado confirmado del encargo

| Pregunta | Respuesta comprobada |
|---|---|
| ¿Se comenzó la evolución técnica? | Sí. La fase anterior modificó 14 archivos de implementación/pruebas y pasó 136 pruebas; esta continuación añade el flujo completo de reevaluación offline. |
| ¿Hubo implementación funcional? | Sí: módulos ejecutables, persistencia SQLite, contratos estrictos, integración contextual y visor. El repositorio documental no contiene ese código. |
| ¿Dónde está el código? | Rama local `codex/independent-review-flow-20261105`, base `066ae03`, con cambios sin commit; conserva el trabajo previo de `codex/linguistic-evolution`. Hay snapshot privado antes/después. |
| ¿Dónde está la documentación publicada? | Rama `codex/intelligent-translation-20261105`, PR #6, separada de la rama local de código; sin fusionar a main. |
| ¿Hay bloqueo o falta autorización? | Ninguno para lo offline implementado. Las llamadas pagadas, producción y aprobación automática nueva permanecen fuera del alcance vigente. |

## Cambios de esta continuación

| Componente privado | Comportamiento implementado |
|---|---|
| `translation/independent_review.py` | Preparación de solicitudes Responses sin envío: revisión bilingüe, posedición focalizada y recomprobación. Esquemas estrictos, evidencia textual, categorías/severidades y vinculación SHA-256 al original, candidata, contexto, glosario, operación y hallazgos. |
| `translation/review_workflow.py` | Selección por riesgo más muestra de control; hasta tres pasos por caso. Edita solo tras hallazgos mayores/críticos fundamentados. Descarta propuestas que pierdan variables, glosario, nombres confirmados o mantengan discrepancias de hechos detectables. |
| `translation/durable_run.py` | Guarda cuerpo exacto de solicitud antes de invocar el simulador y respuesta antes de interpretarla. Distingue fallo de protocolo, QA, transporte incierto y uso sin conciliar. Reanuda sin retransmitir; cambios de alcance/contexto/cap invalidan el diario previo. |
| `translation/review_experiment.py` | Entrada CLI que verifica originales congelados, lee conocimiento en modo readonly y genera una carpeta nueva revisable. Rechaza carpetas desconocidas, entradas cambiadas o experimentos cuya revisión humana ya comenzó. Solo admite el simulador que se abstiene. |
| `ui/case_review.py` y lanzador | El visor muestra estado, procedencia, muestra de control y condición experimental. `Revisar_evolucion_inteligente.bat` abre la copia nueva; sus decisiones quedan en su propio overlay privado. |
| `tests/test_review_workflow.py` | 25 pruebas nuevas de flujo y casos adversariales; ninguna requiere red ni credenciales reales. |

Los prompts conservan voz y naturalidad ES-419 sin forzar literalidad. Incorporan perfil de diálogo, misión, libro, objeto, magia, interfaz o script, datos narrativos disponibles y hasta cuatro referencias del mismo mod/plugin. La recomprobación ve la candidata propuesta sin el dictamen ni los hallazgos previos. Esto limita el anclaje, pero no demuestra independencia estadística ni elimina sesgos compartidos.

## Pruebas y resultados

- **161/161 pruebas**, 70,254 segundos en la suite final; antes de esta continuación: 136/136. Incluye edición válida, pérdida de variables/nombres/glosario, cantidades erróneas, respuesta malformada, evidencia inventada, fallo de recomprobación, interrupción local, timeout y reanudación.
- **1.000 candidatas históricas leídas y copiadas**: 13 seleccionadas por riesgo y 8 sin alertas prioritarias como control. Las 25 de cuarentena no pasan al modelo; 954 quedan sin reevaluación en este ejercicio.
- **21 invocaciones al simulador**, todas inconclusas por diseño. Los otros 42 pasos previstos se omiten sin modelo. Los 63 estados `validated` del diario validan el flujo/protocolo local, no la calidad de 63 traducciones.
- **Reanudación: cero invocaciones adicionales**. Cero candidatas históricas reescritas, cero correcciones inventadas sobre el corpus y cero aprobaciones nuevas.
- La copia tiene 46 prioridades obligatorias: las 38 de la clasificación previa más 8 controles. La clasificación anterior de 196 a 38 sigue siendo reducción de prioridad, no evidencia de mejora semántica ni ahorro humano.
- Reserva offline para los 63 pasos posibles: **US$0,16692480**, bajo el cap de US$0,17. Usa la tarifa ya registrada, un límite de 12.288 bytes por solicitud completa, 512 tokens de encuadre, salida de 2.048 tokens y margen del 15 %. Es una reserva de simulación, no un cargo ni cotización vigente. Un texto que no cabe se retiene; no se trunca ni se amplía presupuesto automáticamente.
- **2.225 archivos protegidos idénticos**, incluidos producción, pilotos, decisiones humanas e informe previo. API Lock sigue activo. Los dos ejercicios intermedios de desarrollo se conservaron aparte, igualmente simulados.

El simulador devuelve `uncertain`: comprueba selección, contratos, persistencia, omisiones de pasos y reanudación. **No mide la capacidad de una segunda IA, fidelidad, naturalidad, falsos negativos ni ahorro humano.** La posedición válida se demostró mediante ejemplos sintéticos programados en pruebas, no mediante juicios fabricados sobre traducciones reales.

## Uso local y evidencia

Abre `Revisar_evolucion_inteligente.bat` desde el workspace privado. Carpeta: `NolvusAIT_Intelligence/20261105_Continuation/private/experiment`. El piloto R7 y sus decisiones conservan su acceso anterior. Aprobar en esta copia guarda una referencia humana aislada; no publica ni importa nada al juego.

Evidencia privada: `review.sqlite3`, cuerpos de solicitud, recibos con hashes, casos, manifiesto de entradas congeladas, `resume_stats.json`, `tests_release.log` y snapshots de código. En GitHub solo se publican este informe, métricas agregadas, protección e inventario de nombres/hashes; nunca código, DB, corpus, decisiones individuales o credenciales.

## Pendientes y siguiente paso

1. Medir el evaluador real en una muestra acotada con referencias humanas independientes, cuando exista autorización específica para consumir API. Actualmente no hay transporte real habilitado en el flujo nuevo.
2. Calibrar falsas aprobaciones y falsos negativos, incluidos casos sin alertas; mantener familias narrativas separadas. La coincidencia de dos evaluaciones no permite aprobar automáticamente. Las tres referencias heredadas siguen siendo insuficientes.
3. Ampliar cobertura contextual de mods con documentación autorizada y adjudicar propuestas terminológicas sin sobrescribir el glosario aprobado. Los 12.423 nombres estructurales previos no equivalen a biografías o conocimiento exhaustivo de todos los mods.
4. Validar tiempos humanos, coste real y naturalidad antes de escalar. Para textos que excedan los límites, falta segmentación explícita con contexto y recomposición verificada. GPT-6.1 Sol, promoción productiva y traducción masiva siguen deshabilitados.

Referencias reutilizadas: [Structured Outputs](https://developers.openai.com/api/docs/guides/structured-outputs) para el contrato y [evaluación específica del dominio](https://developers.openai.com/api/docs/guides/evaluation-best-practices) para diseñar la calibración. El esquema estricto no certifica el significado.
