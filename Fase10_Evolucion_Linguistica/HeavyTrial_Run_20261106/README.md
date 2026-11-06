# Ejecución real Luna/Sol — resultado parcial, 2026-11-06

La prueba se ejecutó con autorización y cap **US$0,30**. **Sol superó los ocho controles sintéticos**, pero la primera solicitud de traducción a Luna devolvió una respuesta vacía e incompleta, con uso reportado de cero tokens. Se activó la cuarentena económica y la parada sellada. No se obtuvieron traducciones nuevas ni se alcanzó la revisión natural, posedición, verificación o auditoría adicional. No corresponde declarar éxito integral ni mejora lingüística.

## Resultados y consumo

| Medida | Resultado real |
|---|---|
| HTTP enviados / respuestas únicas | **9 / 9**; ningún reenvío automático |
| Controles de Sol | **8/8**: cuatro errores sembrados detectados; cuatro formulaciones aceptables con `no_issue` |
| Tokens de Sol reportados | Entrada **7.519**, salida **893**, sin lectura/escritura de caché reportada |
| Consumo local conciliado de Sol | **US$0,023968**; promedio de estos controles: US$0,002996 por llamada |
| Solicitudes a Luna | Una, sin respuesta completada ni candidata aceptable |
| Uso reportado por la respuesta de Luna | Cero entrada, cero salida, cero total; no se presume que esto concilie su facturación |
| Reserva retenida por Luna | **US$0,00249593**, pendiente de aclaración; no se presenta como cargo confirmado |
| Consumo conciliado más reserva retenida | US$0,02646393; son conceptos distintos, no una factura consolidada |
| Reserva activa pendiente | Cero; la reserva de Luna pasó a cuarentena |
| Candidatas anteriores conservadas / nuevas | **950 / 0** |
| Revisiones naturales Sol / correcciones / verificaciones / auditorías | **0 / 0 / 0 / 0** |
| Aprobaciones, promoción productiva, escritura TM/glosario | Cero |
| Estado de protección | API Lock activo; parada persistida; siete entradas protegidas con hashes idénticos |

El coste de Sol se calcula con la tarifa estándar registrada y los tokens reportados: `(7.519 × US$2 + 893 × US$10) / 1.000.000 = US$0,023968`. No certifica la factura del proveedor. Estos controles son textos cortos sintéticos; su promedio no permite proyectar directamente el coste de revisiones naturales más largas. [Tarifa oficial de Sol](https://developers.openai.com/api/docs/models/gpt-6.1-sol).

## Qué ocurrió

**1. Bloqueo local antes de HTTP.** El primer intento encontró que `RealHttpOpenAITransport` conservaba su restricción predeterminada a Luna, aunque el nuevo ámbito autorizaba también Sol. Se detuvo antes de cargar/enviar credenciales y antes de HTTP; la reserva se liberó y el coste fue cero. No fue una respuesta del proveedor. Se conservó ese intento en `private/live`.

Se corrigió `PaidTrial` para vincular el transporte al modelo exacto de cada solicitud y reserva dentro del bloqueo de ejecución, sin quitar la restricción de modelos. Una regresión nueva recorrió el transporte real con servidor simulado y credenciales simuladas, primero Sol y después Luna, verificando también la restauración del API Lock. Pasaron **30 pruebas focalizadas** y la suite completa **308/308**. Las solicitudes, selección, caps y máximo de llamadas permanecieron idénticos; solo se actualizó el sello del código. La continuación autorizada usó un diario separado, `private/live_r2`, conservando el intento anterior y sin duplicar un envío.

**2. Controles reales de Sol.** Las ocho solicitudes completaron su protocolo y cumplieron el criterio sellado. Esto confirma acceso al modelo, transporte, contrato y una comprobación mínima de detección de errores/falsas alarmas. No calibra su calidad sobre los mods ni autoriza aprobar automáticamente.

**3. Respuesta anómala de Luna.** La solicitud `translation_0` contenía **una cadena de 25 caracteres enmascarados**, una solicitud de 8.659 bytes y un máximo de salida de 2.048 tokens. El HTTP fue 200, con modelo `gpt-6-luna`, `status: incomplete`, `incomplete_details.reason: max_output_tokens`, salida vacía y los tres contadores de uso en cero. No hubo traducción que importar ni juzgar.

Ese recibo no demuestra que el texto fuera demasiado largo ni que aumentar el cap resuelva el problema. La combinación de truncamiento declarado y uso cero queda pendiente de aclaración. El libro económico conservó la reserva completa en `QUARANTINED_UNKNOWN_COST` y bloqueó nuevos envíos conforme a la política. No se liberó ese importe suponiendo gratuidad, no se retransmitió la solicitud y no se aumentaron límites para intentar terminar.

## Interpretación y continuidad

Se midió el coste de **ocho controles reales de Sol** y se comprobó que los bloqueos de seguridad conservaron los datos. **No se midió la mejora de ninguna traducción natural**: Sol no revisó los 14 casos previstos, Luna no generó las 100 candidatas nuevas y no hubo correcciones. Las 950 candidatas reutilizadas no recibieron aprobación nueva. Tampoco deben sumarse las fixtures de simulación al resultado real.

La exportación automática de pares de esta ejecución es solo diagnóstica: conserva las siete fuentes reutilizadas seleccionadas, con versiones idénticas y sin evaluación natural. No se presenta como comparación antes/después de Sol ni se pide al usuario revisarla como resultado de calidad. El resto de visores y las 41 decisiones anteriores permanecen intactos.

Para continuar corresponde **aclarar y conciliar el recibo incompleto**, preservar los ocho controles ya realizados y las candidatas válidas, y preparar una continuación acotada que no repita todo ni reenvíe el resultado incierto. La prueba actual está cerrada por su parada económica. No se hicieron más llamadas después de esa parada.

## Evidencia y publicación

Código funcional de cierre: rama privada `codex/heavy-trial-preflight-20261106`, commit `33f613c8752fb8302d26fc03307cdd479eccaf1d`. Preparación utilizada: `private/prepared_final_v5`, sello `8d1921daaa15ebfdc2ba1d5ea6bd53fa30fa9d5e31686b7bc0ea012a508a7e09`. Los artefactos de solicitudes son los mismos del plan anterior; los sellos previos se conservan.

La evidencia privada incluye ambos diarios, libro económico, solicitudes y respuestas con hashes, auditoría de nueve recibos y un paquete de incidente con ID de solicitud/respuesta para contrastar con el proveedor. No se publican corpus, credenciales, código, cuerpos HTTP completos ni decisiones individuales. `RUN_METRICS.json` contiene únicamente agregados de esta ejecución; los documentos de preparación y simulación conservan sus cifras originales y no se reinterpretan como resultados reales.
