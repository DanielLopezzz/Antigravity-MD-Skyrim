# Estabilización funcional Luna/Sol — 2026-11-06

Se implementaron y verificaron cambios de comunicación, conciliación, continuidad, reutilización e importación/exportación. **332/332 pruebas offline** frente a **308/308 de la base**, repetida desde el código respaldado. Cero nuevas solicitudes facturables, créditos, aprobaciones o cambios productivos. La respuesta histórica Luna con uso cero sigue sin causa demostrada; su reserva **US$0,00249593 permanece intacta**.

Código privado: rama `codex/product-stabilization-20261106`, commit `6a153ddffde15717c93c9400349e353a230bc3a2`. Se conservaron las ramas anteriores. Este informe y `METRICS.json` son los únicos resultados nuevos destinados al repositorio documental, junto con las cuatro páginas activas.

## Problemas demostrados y cambios realizados

| Problema | Implementación y efecto comprobado |
|---|---|
| Copia de UID/hash por el modelo y controles distintos de identidad | Nuevo contrato opcional `compact-v2`: el modelo devuelve secuencia y texto. La aplicación verifica cantidad, orden, unicidad y partición sellada; reconstruye UID/hash desde el original local. El intercambio canónico 1.2 conserva su estructura y la lectura anterior. |
| Parámetro de razonamiento prohibido junto con herramientas | Se separó `reasoning.effort` de las capacidades de agente. El perfil experimental usa `low`, salida máxima 2.048, sin pro, herramientas ni razonamiento expuesto. Los payloads anteriores omiten el campo y conservan su serialización. |
| Parser con coincidencia de prefijo de modelo y extracción permisiva | `provider_receipts` comparte identidad exacta/modelo fechado, tipo de mensaje, estado y texto visible entre ejecutores. No usa razonamiento como traducción y rechaza herramientas, refusals, roles ajenos o partes desconocidas. |
| Telemetría inconsistente o caché sin tratamiento uniforme | Se comprueban tipos enteros, valores, total, particiones de caché, razonamiento incluido en salida, modelo y tarifa estándar. Uso cero de entrada en una solicitud no vacía es anómalo; no establece gratuidad. |
| Toda respuesta no completada bloqueaba incluso con consumo comprobable | Política nueva y sellada `isolate-known-v1`: contabiliza uso positivo verificable y separa el fallo de contenido. Uso incierto sigue reteniendo presupuesto y bloqueando envíos; excesos siguen siendo críticos. El perfil económico legado no se migra. |
| La intención económica de envío se registraba después de recibir | Se persiste antes de entrar al transporte. Una interrupción no permite liberar como nunca enviada una reserva potencialmente consumida. Liberación anticipada exige prueba local de cero HTTP o rechazo explícito del cliente. |
| Interrupción entre recibo, contabilidad y actualización del diario | En ámbitos nuevos, `PaidTrial` termina conciliación local desde el recibo preservado o reconoce la ya confirmada; verifica hash y coincidencia de coste. No retransmite. Los errores HTTP conciliados nunca se devuelven como éxito. |
| Plan en memoria mutable o estado terminal sin recibo | `DurableRun` comprueba el sello en memoria, metadatos del diario y existencia/integridad del recibo antes del despacho. El perfil de revisión nuevo declara dependencias entre evaluación, posedición y verificación. |
| JSON malformado escapaba de la clasificación de fallo local | El decodificador convierte el error del contrato en fallo tipado de contenido. Una carga de integración detectó esta omisión; después de corregirla continuaron los lotes independientes sin reenviar el recibo defectuoso. |
| Contexto incompleto en reutilización TM y comparación incremental | La consulta TM carga todos los campos de identidad. El diferencial compara texto exacto y contexto completo; un cambio de misión/mod no hereda aprobación y una revisión humana nueva no se sobrescribe. |
| Importación confiaba demasiado en la etiqueta de validación | Se revalidan estructura, glosario, fuente y contexto en una transacción. Rechaza duplicados y sobrescritura humana. Importar dos veces el mismo resultado es idempotente. Se eliminó la rama inalcanzable de promoción automática a TM. |
| Publicación reversible existía sin conexión al exportador | `Mo2ModPackager` prepara una carpeta hermana completa y usa bloqueo, backup y `publish_directory` junto con el historial. Una excepción de escritura/historial restaura el paquete anterior. Rollback comprueba manifiesto, versión y ámbito del backup. |
| El visor podía mostrar parejas previstas que Sol no evaluó | `heavy_trial` solo exporta comparaciones con dictamen natural efectivamente obtenido. Los duplicados diagnósticos históricos se conservaron con su advertencia; no se regeneró el visor humano. |

Se modificaron los módulos de proveedor, ensayo acotado, presupuesto, diario, flujo por lotes/revisión, contrato del juez, pipeline, diferencial y exportación. `translation_wire` y `provider_receipts` concentran contratos comunes en lugar de repetirlos en cada ejecutor. El coordinador de Batch API antiguo queda explícitamente limitado a simulación: cancelar o no encontrar una línea no demuestra cero consumo real. El transporte Responses utilizado por las pruebas no se sustituye por Batch ni aplica descuentos supuestos.

El inventario inicial cubre **95 archivos Python y 28.266 líneas** con AST, dependencias, puntos de red/escritura y excepciones; ahora hay 97 archivos fuente. La lectura de rutas críticas abarcó proveedor, ejecutores, contexto/KB, glosario/TM, protección/QA, diario, servicios, entrada/visor, importación y exportación. Las regresiones recorren también formatos, escaneo/VFS, configuración e informes. No es una prueba formal de ausencia de errores ni una validación sobre Nolvus real. `pip check` y compilación pasaron; no se añadieron dependencias.

## Comunicación, contexto y calidad

Luna traduce y propone correcciones; Sol evalúa y verifica. Se conserva selección ordinaria del 2 % redondeada hacia arriba y máximo 14 revisiones iniciales, compartido con dudas priorizadas y sujeto al presupuesto. No se envía toda la población a Sol. El perfil compacto debe seleccionarse explícitamente para planes nuevos; los sellos históricos no se reinterpretan.

El prompt compacto conserva fidelidad, agencia, negación, cantidades, ambigüedad, voz y metáforas naturales ES-419. Mantiene glosario vinculante, nombres y familias de equipo coherentes entre HUD, objetivos y diálogo, documentación con procedencia y límite de cuatro fragmentos. No inventa biografías, relaciones ni límites de UI. Las referencias locales completas siguen disponibles para revalidar. Se añadió `accuracy.mistranslation` al contrato del juez: un error de sentido general ya no necesita disfrazarse de fluidez, omisión o terminología.

En una **fixture** de 25 caracteres y el mismo contexto, el payload pasó de **7.683 a 3.646 bytes (−52,54 %)**; instrucciones: 6.050 → 2.166 bytes; esquema: 578 → 332. Es reducción de solicitud, no ahorro facturado ni mejora semántica medida. No reproduce exactamente los 8.659 bytes del incidente real ni demuestra su causa.

La documentación oficial confirma que razonamiento y texto visible comparten el límite de salida y que puede haber truncamiento sin texto visible. También describe esfuerzo por defecto `medium` en Luna/Sol. Esto justifica controlar esfuerzo y simplificar el contrato; **no explica por sí solo los contadores cero del recibo histórico**. No se elevó el límite para ocultarlo. [Razonamiento y límites](https://developers.openai.com/api/docs/guides/reasoning), [Structured Outputs](https://developers.openai.com/api/docs/guides/structured-outputs).

La investigación se adaptó al producto sin incorporar servicios o librerías nuevos:

| Fuente primaria | Aplicación y límite |
|---|---|
| [Weblate: contexto de sugerencias LLM](https://docs.weblate.org/en/weblate-2026.10/admin/machine.html) | Mantener contexto y glosario desambiguado en propuestas; no tratar una sugerencia como aprobación. Se conservó el conocimiento por mod y se reforzó su paso hasta el staging. |
| [Fluent: términos parametrizados](https://projectfluent.org/fluent/guide/terms.html) | Mantener identidad terminológica y variantes gramaticales explícitas. No imponer por palabra aislada una familia o sentido; los ámbitos semánticos existentes se conservaron. |
| [OmegaT: memorias y recuperación](https://omegat.sourceforge.io/manual-latest/en/chapter.how.to.html) | Reutilización contextual y recuperación de resultados conservados. Se corrigieron las consultas incompletas y se probó replay sin reenvío. |
| [GEMBA-MQM](https://arxiv.org/abs/2310.13988) | Hallazgos con categoría/severidad y fragmentos verificables. Se amplió el contrato de sentido; acuerdo del juez y fixtures no constituyen una referencia humana ni evidencia de mejora real. |
| [Netflix: guía de español y nombres](https://partnerhelp.netflixstudios.com/hc/en-us/articles/217349997-Spanish-Latin-America-Spain-Timed-Text-Style-Guide) | Conservar intención y nombres salvo equivalencias aprobadas. No se importaron restricciones de caracteres o temporización de subtítulos a libros/HUD. |

## Verificación y continuidad

| Comprobación | Resultado |
|---|---|
| Base respaldada, mismos tests históricos | **308/308**, cero fallos/errores y cero intentos de conexión |
| Código final | **332/332**, incluidas 24 regresiones nuevas; cero fallos/errores y cero intentos de conexión |
| Integración programada | Responses simulado → Luna → Sol detecta cantidad sembrada → Luna corrige → Sol verifica → importación pendiente; replay e importación repetida añaden cero llamadas/escrituras |
| Carga sintética | **1.000 fuentes / 40 lotes**; cuatro fallos sembrados retienen 100 fuentes; 36 lotes/900 candidatas continúan e importan como pendientes |
| Fallos sembrados en carga | Incompleto con uso conocido, JSON visible malformado, secuencia incorrecta y pérdida de marcador |
| Revisión en carga | 40 intercambios Luna y uno Sol simulados; 13 unidades de muestra aplazadas por la reserva conservadora. No significa que Sol revisó las 900 |
| Reanudación e importación repetida | Cero intercambios adicionales; cero entradas TM o aprobaciones nuevas |
| Seguridad/regresiones | Uso ausente/cero/contradictorio, exceso, herramienta/refusal, identidad, crash antes/después de conciliar, corrupción del plan, protección humana y fallo de exportación |
| Integridad anterior | **68/68 archivos protegidos idénticos**, incluidas DB/KB, decisiones humanas, preparación y ejecución incierta; 41 decisiones conservadas |

La carga duró 11,139 s en este entorno; no proyecta velocidad de modelos ni coste real. Su coste contable virtual no son créditos consumidos. Una reserva de presupuesto conservadora puede aplazar casos aunque el consumo virtual efectivo sea bajo. El antecedente de 1.000 lotes/940 validados/50 cuarentenas/10 protocolos corresponde a otra fixture y no se compara como mejora porcentual de calidad.

Los fallos de contenido conocidos se aíslan; las dependencias quedan visibles. Un envío o uso incierto, credenciales, alteración del sello o exceso económico detiene nuevos envíos del dominio. Solo la prueba de no envío permite reintento acotado; no hay reenvío automático de una respuesta consumida. La exportación conserva backups y revierte excepciones normales; una pérdida abrupta de energía entre renombres y el commit SQLite no constituye una transacción única y requiere verificación/recuperación desde backup.

## Siguiente validación, preparada sin ejecutar

Se preparó y ensayó con transporte simulado un diagnóstico independiente: **dos fuentes nuevas para Luna y una revisión Sol**, máximo **tres HTTP**, cap propuesto **US$0,05** (Luna US$0,005; Sol US$0,045). Envolvente máxima 6.144 bytes, salida 2.048, esfuerzo `low`, cero reintentos, posediciones o auditorías. La reserva conservadora total es **US$0,0469568**, usando el peor caso de escritura de caché y margen del 15 %. Los previews Luna miden 4.115/4.636 bytes; las plantillas Sol, 5.338/5.865. El binding se reconstruye sobre una candidata real válida antes de la única evaluación; la plantilla no se envía como resultado.

Las tarifas registradas siguen siendo entrada/salida por millón: Luna US$0,10/0,50 y Sol US$2/10; antes de una autorización deben verificarse nuevamente. La reserva no presume descuento. [Luna](https://developers.openai.com/api/docs/models/gpt-6-luna), [Sol](https://developers.openai.com/api/docs/models/gpt-6.1-sol).

El diagnóstico excluye la fuente incierta y no repite los ocho controles anteriores. Conservando consumo más retención previa (US$0,02646393), el máximo combinado sería US$0,07646393 dentro del cap previo US$0,30; **no modifica ese libro ni libera su reserva**. Exige autorización específica nueva. Se detiene ante salida no utilizable, uso incierto, identidad/estructura/glosario inválidos, cambio de modelo/tarifa/sello o exceso de envolvente/presupuesto. No aumenta límites para continuar.

La simulación del procedimiento completó tres intercambios, reanudó con cero adicionales y rechazó ejecución real sin autorización antes de crear archivos. Evidencia privada: `private/next_probe_v3/plan.json`, `probe_runner.py` y `probe_check.json`. La carpeta de ejecución futura debe ser nueva y separada. No se han enviado esas solicitudes.

Pendientes reales: aclarar el recibo histórico con evidencia del proveedor; confirmar por API el perfil nuevo; medir fidelidad/naturalidad y falsas alarmas sobre casos naturales adjudicados; calibrar Sol antes de reducir más intervención humana; validar escalamiento y recuperación del entorno real en un sandbox autorizado. **No se declara preparación para producción.** Producción, TM/glosario aprobados, Nolvus, MO2, mods, partidas y visores humanos no fueron modificados.
