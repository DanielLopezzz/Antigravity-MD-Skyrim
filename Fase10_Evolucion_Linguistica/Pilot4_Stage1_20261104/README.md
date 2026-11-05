# Cuarta prueba — resultado de la primera etapa

Fecha local: 2026-11-04, America/Bogota. Ejecución: 2026-11-05 01:43:39–01:44:34 UTC.

**Estado: primera etapa ejecutada; continuación detenida por un defecto local de enmascaramiento.** Las etapas 2 y 3 no se ejecutaron. La detención posterior es una decisión de revisión editorial; el runner completó sus dos lotes y devolvió `AWAITING_STAGE_REVIEW`.

## Alcance autorizado y ajustes humanos

El usuario autorizó iniciar la prueba preparada tras adjudicar los cuatro puntos de revisión. Se restauró la tercera persona en el mensaje de conocimiento de un hechizo; se recuperaron los dos puntos y saltos de línea en un mensaje de script; se conservó su elección de tiempo presente en el ejemplo narrativo y se aprobó el caso con el marcador de magnitud.

Las **43 referencias previas están aprobadas**. Se guardaron las tres actualizaciones mediante el almacén de revisiones, conservando notas e historial y una copia privada previa a los cambios. No se modificó el tiempo presente elegido por el usuario. Estas referencias siguen separadas de la nueva prueba y no se inyectaron como respuestas en las solicitudes.

El alcance económico utilizado fue un tope local de **US$0,17 para todo el trabajo**, con una primera etapa de 50 textos, cero reintentos, sin escalamiento a Sol, sin promoción a producción ni a la TM.

## Resultado medido

| Métrica | Resultado |
|---|---:|
| Solicitudes reales | 2 |
| Respuestas recibidas | 2 |
| Cadenas recibidas y aceptadas mecánicamente | 50/50 |
| Cadenas todavía sin enviar | 950 |
| Tokens de entrada medidos | 10.866 |
| Tokens de salida medidos | 6.798 |
| Coste conciliado | **US$0,00448560** |
| Saldo local disponible del trabajo | US$0,16551440 |
| Reservas abiertas / importes económicos en cuarentena | 0 / 0 |
| Traducciones promovidas a producción o TM | 0 |

Ambas reservas económicas quedaron `RECONCILED`. El estado de trabajo aislado es 43 candidatas `pending`, 7 `requires_lore_research` y 950 `unprocessed`. Las siete alertas automáticas se refieren a posibles nombres/términos sin registrar; no son siete errores de traducción confirmados.

## Hallazgo que impide continuar

La revisión de los 50 resultados detectó fragmentos ingleses extraños dentro de una descripción con porcentajes. Se reprodujo la causa completamente offline, comparando original, entrada enmascarada, tabla de tokens y respuesta original:

- La expresión regular de formatos `printf` acepta espacios y modificadores de longitud.
- En este contexto, capturó **`% le`** y **`% f`**, que eran partes de palabras ordinarias después de porcentajes.
- El modelo recibió palabras partidas y devolvió todos los marcadores intactos. Al restaurarlos, se reinsertaron los fragmentos ingleses.
- El Gatekeeper aceptó la estructura porque las variables coinciden con el sobre que recibió. Esto confirma que integridad de marcadores y fidelidad de la entrada son comprobaciones distintas.

El diagnóstico encontró **15 falsos tokens en 14 cadenas de las 1.000 preparadas**: una ya enviada y otras 13 todavía sin enviar. Por esa razón se decidió no gastar las etapas siguientes con el mismo defecto. No se atribuye este caso a una mala traducción del modelo: la entrada ya estaba dañada por el protector local.

Los detalles con textos e identificadores están solo en el archivo privado `stage_1_input_masking_diagnosis.json`, dentro de la carpeta de la prueba. Las respuestas recibidas, payloads y candidatos originales se conservaron íntegros; no se reemplazó la salida defectuosa por una corrección manual.

## Observaciones editoriales adicionales

La lectura de los 50 textos muestra muchas formulaciones claras y funcionales, pero no basta para declarar éxito lingüístico del conjunto ni superioridad frente al piloto anterior. Se identificaron además puntos para adjudicar:

- Mezcla de destinatario plural y singular en un mensaje corto.
- Cambio de trato formal/informal en un diálogo.
- Variación de la denominación de un hechizo frente a la referencia humana anterior.
- Dos construcciones diferentes para el nombre de la misma forja.

Son seis casos anotados por IA en total, incluido el defecto de porcentajes. Se añadieron a los siete casos ya señalados por el validador: **13 casos de revisión obligatoria en el visor**. El resto de los 50 conserva prioridad de muestra de calidad. Las anotaciones distinguen su autoría por IA y **no constituyen aprobación humana ni corrección automática**. Las diferencias de estilo o denominación requieren contexto; no se contabilizan automáticamente como fallos.

## Preservación y acceso

- Candado de API restaurado: `True`.
- **55 archivos protegidos anteriores idénticos** en tamaño, mtime y SHA-256; producción, configuración y pilotos anteriores intactos.
- TM de la nueva prueba vacía. No se exportó nada al juego.
- No se cambió código de traducción durante la ejecución. La suite anterior de 92 pruebas no cubría este falso positivo; no se afirma que esas pruebas certifiquen la corrección de este caso.
- Solo se modificaron las tres revisiones expresamente adjudicadas por el usuario, los artefactos propios de la prueba y los informes locales.

Abre `Revisar_cuarta_prueba.bat` y pulsa **Recargar**. Selecciona **Pendiente** para revisar los 50 candidatos, o **Prioridad de revisión → Obligatoria** para los 13 señalados. Los otros 950 siguen como **Pendiente de traducción**. La primera etapa todavía no tiene aprobaciones humanas nuevas.

## Siguiente paso requerido

Corregir la detección de porcentajes sin romper formatos `printf` legítimos; añadir regresiones offline con el caso observado y variantes; verificar que la entrada ya no contenga palabras partidas; y preparar una continuación identificada por separado, manteniendo congeladas las evidencias de esta primera etapa. No editar el plan antiguo ni reenviar sus lotes como si fueran solicitudes nuevas de la misma etapa.

El presupuesto debe seguir descontando el consumo ya realizado de US$0,00448560. La continuación debe considerar los resultados de la revisión humana de esta etapa, distinguir fallos locales de calidad del modelo y mantener el límite autorizado. El permiso de ejecución no habilita promoción a producción ni aprobación editorial automática.

Evidencia agregada: [métricas de la etapa](stage_1_metrics.json). La carpeta privada de ejecución contiene `stage_1_summary.json`, `progress.json`, evidencias completas por lote, snapshot de referencias humanas y `EDITORIAL_CONTINUATION_HOLD.json`. Este último registra la recomendación de detención editorial; no se presenta como un nuevo candado automático implementado en el runner.
