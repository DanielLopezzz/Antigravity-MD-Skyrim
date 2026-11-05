# Corrección del protector y repetición R2 de la cuarta prueba

Fecha local: 2026-11-04, America/Bogota. Estado: **corrección comprobada; R2 ejecutada y detenida por rechazo del segundo lote**.

## Corrección aplicada

Se corrigió el reconocimiento de porcentajes en `quality/protector.py`. El signo `%` posterior a un valor numérico, una etiqueta de magnitud o un marcador de valor se reserva antes de buscar especificadores `printf`, cuando el contexto corresponde a una unidad porcentual. Se protege únicamente el signo; las palabras siguientes permanecen disponibles para traducirse completas.

Esto evita interpretar fragmentos como `% le`, `% f`, `% i`, `% s` o `% d` como formatos cuando forman parte de prosa después de un porcentaje. Los formatos reales `%s`, `%d`, `%05d`, `%.1f`, `%2$s`, `%2$08.2f`, `%+8.2f`, `% f`, `% s`, `%*.*f`, `%ld` y `%%` siguen protegidos. También se verifican formatos consecutivos y formatos junto a etiquetas HTML.

Se añadieron tres pruebas de regresión, con 42 combinaciones de valor/palabra, restauración de una traducción, rechazo de pérdida del signo porcentual y conservación de formatos legítimos. Antes del cambio fallaban 43 aserciones/subcasos; después pasan. **Suite completa: 95/95 OK, 36,653 s.**

La verificación offline de las **1.000 entradas** confirmó restauración exacta del original y desaparición del patrón defectuoso en los **14 textos identificados**. Las comprobaciones no sustituyen una auditoría de todos los formatos posibles del inventario completo.

## Repetición controlada

Se preparó una nueva carpeta, `Phase4_Luna1000_R2`, sin sobrescribir el primer intento. La muestra mantiene **los mismos 1.000 identificadores y originales, en el mismo orden, y el mismo glosario**. El cambio de código queda identificado mediante la nueva huella del plan. Las instrucciones editoriales y el límite de cero reintentos permanecen intactos.

El intento anterior consumió US$0,00448560. Para mantener el límite global autorizado de US$0,17, R2 recibió un tope de **US$0,16551440**. La reserva prevista de todo R2 era US$0,16205604; se ejecutó únicamente su primera etapa y no se autorizó promoción a producción o TM.

## Resultado real de R2

| Métrica | Resultado |
|---|---:|
| Solicitudes reales / respuestas recibidas | 2 / 2 |
| Cadenas recibidas | 50 |
| Primer lote aceptado e importado | 25 |
| Segundo lote retenido por atomicidad | 25 |
| Candidatas válidas retenidas dentro del segundo lote | 24 |
| Candidata que provocó la cuarentena | 1 |
| Cadenas todavía sin enviar en R2 | 950 |
| Tokens medidos, entrada / salida | 10.860 / 6.623 |
| Coste conciliado de R2 | **US$0,00439750** |
| Coste acumulado de ambos intentos | **US$0,00888310** |
| Saldo restante del límite total autorizado | **US$0,16111690** |

**La respuesta real confirmó la corrección de porcentajes:** la descripción afectada ya se traduce sin los fragmentos ingleses que se reinsertaban antes. Esta candidata quedó retenida por la atomicidad de su lote, aunque no fue la que originó el rechazo.

El segundo lote fue rechazado porque **el modelo usó dos veces un alias de personaje que aparecía una sola vez en el original**: reemplazó una referencia pronominal posterior por otra copia del marcador. El Gatekeeper detectó la duplicación y conservó íntegros el cuerpo de respuesta y los candidatos. El contrato vigente exige multiplicidad exacta; no se relajó la validación ni se corrigió silenciosamente la respuesta para contabilizarla como aprobada.

Este fallo es distinto del anterior: el defecto de porcentajes dañaba la entrada local; aquí la entrada conserva su integridad y la respuesta del modelo incumple el contrato. Las otras 24 candidatas del lote están conservadas, pero no se importaron parcialmente. Las etapas 2 y 3 no se ejecutaron; se respetó la detención y la política de cero reintentos.

## Conciliación y límites del resumen automático

Las dos reservas de R2 figuran como **`RECONCILED`** en el libro presupuestario, con cero dinero reservado o en cuarentena. El resumen del runner marca genéricamente `billing_reconciliation_complete=False` cuando ocurre cualquier error, incluido un rechazo editorial posterior a la conciliación. En este corte se conserva ese resumen original y se documenta la discrepancia: **el coste sí está conciliado en el ledger**. Después de terminar R2 se corrigió el indicador para consultar el ledger y conservar el estado incierto cuando falta una respuesta. Una regresión adicional verifica que un rechazo de QA pueda tener coste conciliado y siga bloqueando la continuación. **Suite final: 96/96 OK, 26,860 s.** No se reescribió el resumen original ni se realizaron más solicitudes después de ese ajuste; se archivó el código utilizado en R2 y se comprobó que reproduce su huella sellada.

## Preservación y revisión

- **345 archivos preexistentes protegidos idénticos** en tamaño, mtime y SHA-256, incluidos el primer intento, los pilotos anteriores, producción y revisiones humanas.
- Las 43 referencias humanas anteriores permanecen intactas. No se crearon aprobaciones humanas de R2.
- Base de trabajo R2: 25 filas `pending` y 975 `unprocessed`; estas últimas incluyen las 25 retenidas y las 950 todavía sin enviar. La vista de evaluación sí conserva las 50 respuestas.
- TM vacía, cero promociones productivas, cero reintentos y cero llamadas a Sol.
- Candado de API restaurado. Los hashes de ambos cuerpos de respuesta coinciden con sus metadatos de transporte.

`Revisar_cuarta_prueba.bat` abre ahora **R2**. Usa **Recargar** si ya estaba abierto el visor y comprueba que la carpeta mostrada sea `Phase4_Luna1000_R2`; si sigue en la anterior, utiliza **Abrir otra prueba…**. Hay 25 casos de muestra y 25 de revisión obligatoria por cuarentena atómica; eso no significa que los 25 contengan errores individuales.

Las propuestas nuevas no son uniformemente superiores a las anteriores: hay cambios de trato, elección léxica y denominaciones que requieren juicio humano. El buen resultado del caso porcentual prueba la reparación técnica, no el éxito lingüístico de las 1.000 cadenas. **No se declara completada la prueba de carga.**

El siguiente paso operativo debe tratar el alias duplicado de forma focalizada, conservar las 24 candidatas válidas y mantener trazabilidad de cualquier reparación/reprueba; no repetir automáticamente todas las cadenas ni desactivar la aceptación atómica. La continuación debe descontar el coste acumulado del límite global.

Evidencia agregada: [verificación offline](offline_corpus_check.json), [métricas reales de R2](r2_result_metrics.json), [preservación](protected_verification.json). Las entradas, respuestas, diagnóstico por identificador y referencias permanecen en carpetas privadas locales. La documentación agregada se actualiza en el checkout local, sin commit ni push.
