# Cuarta prueba — preparación offline y revisión de seguridad

Fecha local: 2026-11-04, America/Bogota. Estado: **PREPARACIÓN COMPLETADA; EJECUCIÓN REAL PENDIENTE**.

La muestra definitiva y sus solicitudes están preparadas. No se ha medido todavía su calidad lingüística ni se han utilizado créditos. La autorización recibida en este corte cubre preparación y ajustes justificados; no se ha ejecutado la API.

## Muestra y etapas

Se prepararon **1.000 textos únicos de 130 mods**, con 100 controles que coinciden en identificador y original con las traducciones importadas del piloto anterior, y 900 textos adicionales. Los candidatos antiguos se conservan para comparación, sin enviarlos como respuestas de referencia al modelo.

| Categoría | Cadenas |
|---|---:|
| Diálogos | 200 |
| Misiones | 150 |
| Objetos | 150 |
| Magia | 150 |
| Libros, incluidos títulos de control | 150 |
| Interfaz/MCM | 100 |
| Mensajes de scripts | 100 |

La selección cubre 132.260 caracteres de origen; 239 textos contienen elementos protegidos. El máximo es 2.982 caracteres. Se limita la concentración por mod y grupo de contexto, y se deduplican originales exactos. Las cadenas nuevas de libros tienen al menos 80 caracteres. Se mantienen los seis títulos breves necesarios para comparar los controles.

No es una muestra aleatoria del inventario completo: aplica cuotas, límites de longitud, diversidad y filtros conservadores. Tampoco demuestra coherencia de un mod o una historia completos.

| Etapa | Nuevas cadenas | Acumulado | Lotes | Reserva calculada, USD |
|---|---:|---:|---:|---:|
| 1 | 50 | 50 | 2 | 0,00657732 |
| 2 | 200 | 250 | 8 | 0,02755552 |
| 3 | 750 | 1.000 | 33 | 0,12765870 |
| Total | 1.000 | 1.000 | 43 | 0,16179154 |

Cada etapa requiere una invocación independiente. No se avanza automáticamente. Los lotes contienen entre 2 y 25 cadenas; los textos largos reducen el tamaño del lote. Sus topes de salida van de 4.096 a 8.064 tokens, con margen sobre la estimación de salida. El plan usa el adaptador Luna configurado en el proyecto, una solicitud simultánea, cero reintentos y ningún escalamiento a Sol. Son solicitudes síncronas de Responses; agrupar cadenas no equivale a usar el servicio Batch asíncrono.

## Presupuesto y alcance de la estimación

La reserva conservadora total es **US$0,16179154**; se propone **US$0,17 como tope local de todo el trabajo**, no por etapa. Supera el antiguo límite de US$0,05; ese presupuesto histórico y sus bases no se modificaron.

El cálculo usa las tarifas locales configuradas en `budget_control.py`: entrada 0,10 y salida 0,50 USD por millón, topes de salida y margen 1,15. Es una estimación del proyecto, no una cotización actual verificada del proveedor ni consumo medido. Antes de ejecutar, se deben reconfirmar modelo y tarifas y autorizar expresamente el importe. Si cambian los parámetros de traducción/precio, se prepara otra carpeta y otro plan.

La base económica nueva contiene **0 trabajos, 0 proyectos y 0 reservas**. No se inicializó un presupuesto facturable. Credenciales cargadas: 0. Solicitudes HTTP al proveedor: 0. El candado global sigue activo.

## Hallazgos y correcciones justificadas

1. **Literales técnicos en el inventario PEX.** Una inspección de la primera selección encontró rutas, variables y nombres de estados clasificados antiguamente como traducibles. Esa preparación se descartó antes de enviar nada y se conservó como `Phase4_Luna1000_rejected_technical_selection`. Su preflight comprobaba integridad mecánica, no idoneidad semántica del corpus. La muestra definitiva solo admite el primer argumento de texto de `Debug.Notification`, `Debug.MessageBox`, `self.SetInfoText` y `self.ShowMessage`, excluye claves `$...` y rutas. No se reclasificó ni reescaneó producción. **La clasificación general de PEX sigue requiriendo una auditoría antes de producción**; el filtro de esta prueba no corrige todo el inventario.
2. **Controles omitidos por filtros.** Se corrigió la confusión entre texto terminado en dos puntos seguido de `\n` y una unidad de Windows, y se admitieron títulos breves de libros como controles. La selección definitiva conserva los 100 controles. Incluye las dos cadenas con `Elder Scroll` y una con `HeelsFix Refresh`, procedentes de los casos que motivaron la remediación.
3. **Runner anterior fijo a cinco lotes.** Se añadió una preparación y ejecución por etapas con lotes adaptados al tamaño real, en bases independientes. Los pilotos cerrados permanecen intactos.
4. **Pérdida de evidencia al rechazar una respuesta.** El transporte nuevo conserva el cuerpo completo recibido, con redacción de credenciales, antes de analizarlo. Guarda diagnóstico y candidatos en privado incluso si el lote se rechaza. Los errores de red sin respuesta quedan registrados como intentos de resultado incierto; no se inventa una respuesta ni un coste conciliado.
5. **Reenvíos y conciliación.** Un bloqueo entre procesos y un registro persistente de lote en curso impiden continuar automáticamente tras interrupciones o fallos. Una etapa completada no se reenvía. El resumen usa el libro presupuestario, incluidos importes en cuarentena, y distingue intentos de respuestas recibidas. Una respuesta sin conciliación no se informa como coste cero definitivo.
6. **Revisión local.** El visor permite abrir otra prueba, recargar y filtrar por prioridad. El guardado detecta versiones concurrentes, usa bloqueo entre procesos y conserva historial; las decisiones nuevas se vinculan al original y su contexto. Se mantiene compatibilidad con decisiones previas. No se permite aprobar un caso todavía sin respuesta de API.
7. **Test ligado a una fecha antigua.** El test de protección productiva comparaba el mtime con un corte anterior. Se sustituyó por una comparación de tamaño, mtime y SHA-256 antes/después de intentar accesos prohibidos. Así comprueba que la operación no cambie la fuente actual.

## Integridad y verificación

- **92/92 pruebas pasan**, suite completa final: 14,736 s. Incluye simulación de tres etapas, reanudación sin duplicar envíos, respuesta malformada retenida, timeout, autorización previa a credenciales, guardado concurrente, decisiones antiguas y filtros de la interfaz.
- Preflight definitivo: `READY_OFFLINE_NOT_AUTHORIZED`, 1.000 cadenas y 43 lotes; regeneración exacta de payloads, presupuesto y contexto correcta.
- **55 archivos preexistentes protegidos idénticos** en tamaño, mtime y SHA-256. Incluyen base/configuración productivas y artefactos de pilotos anteriores. Los archivos nuevos de la cuarta prueba quedan fuera de esa comparación.
- El archivo de decisiones que está revisando el usuario se excluyó deliberadamente de la comparación: puede seguir cambiando por su trabajo. Esta preparación no lo escribió ni lo reemplazó.
- El código de traducción y los archivos estáticos quedan identificados por hashes. Si cambian después del sellado, el runner rechaza ejecutar el plan antiguo.

Evidencia local: [métricas agregadas](readiness_metrics.json), [comparación de archivos protegidos](protected_verification.json), [suite completa](TESTS.md), [preflight definitivo](preflight_result.json). `baseline_source` conserva el estado previo a estos ajustes; no es una copia destinada a publicación.

## Operación y revisión humana

Abre `Revisar_cuarta_prueba.bat`, o usa **Abrir otra prueba…** en el visor y selecciona `Phase4_Luna1000`. Hoy aparecen 1.000 casos **Pendiente de traducción**. Después de cada etapa se usa **Recargar**. El visor no ejecuta traducciones.

Tus decisiones anteriores siguen en `NolvusAIT_Evolution/20261104/private/human_reviews.json`. Las decisiones nuevas se guardarán en `NolvusAIT_PilotStaging/Phase4_Luna1000/private/human_reviews.json`. Ninguna aprobación de este visor promueve traducciones a producción o a la TM.

Antes de gastar: terminar/revisar los casos anteriores, identificar problemas recurrentes y decidir si requieren cambiar el prompt o el glosario. **No se incorporarán correcciones humanas automáticamente ni a mitad de esta misma prueba**. Si sus conclusiones exigen cambios, esta preparación es sustituible y debe regenerarse en otra carpeta con los cambios congelados. Las referencias humanas se comparan como evaluación privada; una coincidencia literal no demuestra calidad y una formulación distinta puede ser correcta.

Tras recibir resultados:

1. Revisar las 50 cadenas de la primera etapa antes de decidir avanzar. Verificar significado, naturalidad, voz, trato, contexto, nombres y variables, también en candidatos sin alertas.
2. Tras 250, revisar todas las alertas y la muestra de calidad disponible. El visor selecciona hasta 200 candidatos sin alertas por categoría y cohorte, con orden determinista por hash. La selección se recalcula al llegar nuevas etapas; se deben recargar los casos. Todas las decisiones guardadas se conservan.
3. Al terminar, cubrir toda la muestra de calidad final y todos los casos obligatorios. Registrar en notas `[significado]`, `[naturalidad]`, `[voz]`, `[terminología]`, `[contexto]` o `[formato]`, con gravedad y propuesta. Separar aprobación sin cambios, retoque menor y reescritura sustancial: aprobar una corrección humana no implica que la salida original fuera buena.
4. Producir un recuento por categoría y cohorte: cobertura recibida, aceptación mecánica, alertas, pendientes, aprobación original, correcciones, rechazos y patrones de traducción recurrentes. Comparar los 100 controles conservando ambas versiones y las preferencias humanas; no afirmar superioridad sin comparación suficiente.

**Criterios propuestos de continuidad**, todavía sujetos a evaluación humana: cero errores críticos de sentido/variables/identidad; detenerse si se repite un patrón de omisión, invención o cambio de intención; revisar antes de avanzar si más del 5 % de los evaluados requiere una corrección sustancial. Informar siempre el denominador y los pendientes. Cualquier rechazo atómico o respuesta incierta detiene el runner y requiere reconciliación; no tiene reintento automático. Un resultado técnico correcto no basta para declarar éxito lingüístico ni autoriza el corpus completo.

## Comprobación offline reproducible

Desde la raíz local del proyecto:

```powershell
$env:PYTHONPATH='src'
python -m nolvus_translator.translation.pilot_four preflight
python NolvusAIT_Pilot4_Preparation/20261104/verify_preparation.py
```

La ejecución posterior usa `pilot_four execute` con etapa, importe autorizado y confirmación de gasto explícitos; no debe invocarse durante la revisión offline. No editar `progress.json` para forzar reenvíos. Ante un lote fallido o en curso, conciliar primero las evidencias y el presupuesto y preparar una continuación identificada si procede.

El directorio activo `Phase4_Luna1000` contiene `plan.json`, `plan.sha256`, `payloads`, tres bases aisladas y `private`. Corpus, candidatos, decisiones, respuestas y bases permanecen privados. El repositorio documental solo recibe informes agregados; en este corte se actualiza su copia local sin commit ni push.
