# Barrido de componentes y seguimiento de la cuarta prueba

Corte local: 2026-11-04, America/Bogota. Alcance: flujo de preparación → API → validación → recuperación → importación aislada → revisión. El barrido no certifica todo el inventario ni constituye una aprobación lingüística del juego completo.

## Hallazgos y acciones

| Componente | Hallazgo o riesgo revisado | Acción y verificación |
|---|---|---|
| Protector de variables | Los porcentajes ya estaban corregidos, pero faltaban saltos de línea y tabulaciones reales. Una carta del intento anterior perdió el último LF. | Se protegen secuencias reales de CRLF/LF/CR/tab, agrupando secuencias consecutivas para evitar marcadores redundantes. Se comprueba multiplicidad y orden. La separación literal `\n` continúa protegida. |
| Pistas del contrato | Los nuevos tipos de marcador no tenían descripción gramatical específica. | Se añaden roles de unidad porcentual y secuencia de control al contrato compatible v1.1. Se mantienen tipos, hashes e identificadores existentes. |
| Instrucciones del modelo | El modelo sustituyó un pronombre por otra copia de un alias, pese a la regla general de multiplicidad. | Se añade una instrucción focalizada: contar tokens por ítem, traducir referencias pronominales sin repetir el alias y conservar el orden de controles. No se cambia el criterio editorial del resto del texto. |
| Preparación y partición | Reiniciar desde cero reenviaba propuestas válidas del intento anterior. | Selección congelada; partición disjunta y completa de textos reutilizados/solicitados. Se comparan todos los campos originales con la fuente real y se verifica la evidencia anterior por hash. |
| Reutilización | Los archivos de revisión son editables y no deben tratarse como salida original del proveedor. | Las propuestas se reconstruyen desde el cuerpo recibido y la bóveda histórica. Se comprueban marcadores originales y actuales. La admisión definitiva requiere Gatekeeper sobre el grupo completo. Se impide mezclar evidencia simulada y real. |
| Gatekeeper y atomicidad | Se conservaban 24 candidatas válidas, pero el runner detenía el trabajo sin una recuperación focalizada. | Una corrección de 1–3 casos se combina con los válidos conservados en un grupo nuevo; el grupo completo pasa otra vez las 12 puertas antes de importarse en una transacción. No se relajan reglas ni se cambia el texto del proveedor para simular una aceptación original. |
| Recuperación acotada | Repetir un lote completo incrementa gasto y cambia propuestas que ya eran válidas. | R3 permite como máximo cuatro solicitudes semánticas de reparación de hasta tres casos. Cada caso fallido solo tiene una reparación en ese flujo; una reparación que vuelve a fallar detiene el trabajo. No hay reintentos de red ni escalamiento a Sol. |
| Transporte y evidencia | Un fallo durante el envío puede dejar dudas sobre lo que se intentó transmitir. | Se conserva el cuerpo de la solicitud, sin cabeceras, antes del transporte; la respuesta completa se guarda antes de analizarla y se verifica por SHA-256. El registro de lote en curso precede al envío y bloquea reenvíos inciertos. |
| Presupuesto y contador | Los costes previos y las solicitudes de reparación debían mantenerse dentro del mismo límite global. El contador del transporte se reinicia al cambiar de proceso/etapa. | Se descuentan US$0,00888310 ya conciliados del tope total de US$0,17. R3 tiene un máximo de US$0,16111690. El libro presupuestario incluye las reparaciones; el contador de HTTP se acumula en progreso entre etapas. |
| Tamaño de solicitudes | El piso fijo de 4.096 tokens reservaba salida innecesaria en lotes cortos; los controles reales aumentan el tamaño de entrada. | Se calculan topes a partir de la estimación completa con margen 1,6 y redondeo de 64 tokens, mínimo 1.024 y máximo 8.192. Se reduce el lote si excede límites de entrada/salida. No se recortan textos para hacerlos caber. |
| Importación y TM | Una aceptación mecánica no demuestra aprobación editorial. | Las candidatas reales nuevas o reutilizadas continúan pendientes o por investigar. No se crea aprobación humana, no se alimenta la TM ni se exporta a MO2. |
| Visor y trazabilidad | El revisor debe distinguir propuestas conservadas de las que se volvieron a solicitar. | Se registra procedencia por caso y se muestra una nota para propuestas anteriores revalidadas. El acceso de la cuarta prueba apunta a la continuación activa. Las revisiones antiguas permanecen separadas. |

## Verificación previa a las llamadas reales

- **102/102 pruebas pasan**, 18,715 s en la suite completa final previa a R3. Incluyen pérdida/duplicación/orden de controles, formatos printf válidos, presupuesto, rechazo de QA, timeout, fallos acotados de recuperación, atomicidad y continuación sin reenviar los válidos.
- **Simulación de las 1.000 cadenas reales de origen**, con respuestas simuladas: tres etapas completadas hasta 50, 250 y 1.000 sin error de circuito. Cero solicitudes reales en esa simulación. Comprueba el recorrido; no mide la calidad lingüística del modelo real.
- **5 pruebas del visor** posteriores a su nota de procedencia pasan. No se modificó el código de traducción después de sellar la preparación activa.
- Nueva partición: **48 candidatas conservadas + 2 correcciones focalizadas + 950 cadenas sin enviar anteriormente = 1.000**.
- Reserva conservadora de las 952 solicitudes de traducción agrupadas en 42 lotes: **US$0,15987563**, dentro del saldo autorizado de US$0,16111690. Las reparaciones adicionales, si se necesitan, pasan por el mismo presupuesto antes de enviarse y pueden detenerse si no hay saldo.
- Los borradores de estimación se conservaron aparte y no ejecutaron solicitudes. El plan activo tiene identidad nueva y artefactos sellados; las bases y respuestas anteriores permanecen intactas.

## Límites que permanecen

## Hallazgos adicionales durante la ejecución supervisada

La validación offline inicial no anticipó todos los comportamientos reales. Se conservaron los fallos y se crearon continuaciones con identidad nueva, reutilizando únicamente grupos completos revalidados. El objetivo no fue presentar los intentos fallidos como éxitos iniciales.

| Componente | Hallazgo real y corrección |
|---|---|
| Versionado | La política editorial y el esquema del contexto necesitaban versiones independientes. Se separaron para evitar invalidar contratos al ajustar una regla lingüística. |
| Glosario y metadatos | `missives` como nombre común estaba protegido por inferencia del mod `Missives`. La protección inferida exige ahora coincidencia de mayúsculas; identificadores técnicos y glosario aprobado conservan sus reglas. Se revalidó la respuesta original sin reescribirla. |
| Concordancia | Se rechazaba «algún Renegado» por exigir el plural canónico «Renegados». Se admiten variantes explícitas del mismo lema; un sinónimo ajeno sigue rechazándose. La respuesta original se conservó intacta. |
| Cardinalidad | Una respuesta devolvió 26 registros para 25 entradas, con una versión parcial y otra completa del mismo UID. El esquema exige ahora exactamente el número solicitado. La recuperación permite elegir únicamente una versión estructuralmente válida e inequívoca, nunca entre dos versiones válidas distintas. |
| Errores recuperables | El registro de errores focalizados no incluía los errores de duplicación/identidad. Se alineó con el contrato y se probó el rechazo de códigos no admitidos antes de un envío. El intento local que detectó esa omisión no produjo HTTP ni gasto. |
| Recuperación | Se utilizaron cuatro solicitudes focalizadas pagadas en total. No se reinicia ese contador al crear una continuación. Tras agotarlo, el último tramo usa cero reparaciones. |
| Medición diagnóstica | El último tramo conserva en cuarentena un lote íntegro con fallo de QA y continúa con entradas nuevas. Los errores de red, parseo, integridad o conciliación siguen deteniendo el trabajo. Las candidatas rechazadas no se importan parcialmente. |
| Hash de salida incompleto | La validación detuvo un lote con un hash de origen de 63 caracteres. Se verificó offline que estaban los 25 UIDs esperados, la respuesta íntegra y el gasto conciliado. Se conservaron progreso y resumen originales, se adjudicó todo el lote a cuarentena y se reanudaron únicamente las 39 entradas sin enviar. No se corrigió el hash ni se volvió a solicitar ese lote. |
| Prevención posterior al piloto | El esquema remoto exigía solo cadena para SHA-256, mientras el validador local exigía 64 caracteres hexadecimales. Tras cerrar todas las llamadas se añadieron longitud exacta y patrón hexadecimal al esquema. La documentación oficial admite estas restricciones para el modelo estándar usado; no se habilitan modelos fine-tuned. La correspondencia exacta con la fuente sigue comprobándose localmente. Este cambio no altera las respuestas históricas ni se ha probado con otra llamada pagada. Se archivó la fuente exacta usada en R7 antes de modificarla. |
| Formato del poema | Una reparación real siguió omitiendo un CRLF. Se preparó una propuesta separada de IA que solo repone ese salto; se verificó igualdad de palabras y puntuación al normalizar espacios. Sigue pendiente de revisión humana y no cuenta como respuesta API intacta. |

La última suite completa, después de estos ajustes y antes del tramo final, pasa **109/109 pruebas en 37,296 s**. Incluye continuación diagnóstica con cuarentena, detención ante respuesta malformada, duplicados inequívocos y conflictivos, concordancia del glosario, atomicidad y separación de evidencia real/simulada. La simulación completa de 1.000 entradas corresponde al estado anterior a estos ajustes; no sustituye esta suite ni acredita calidad lingüística real.

Después de cerrar la API y añadir la restricción de SHA-256, la suite pasa **110/110 pruebas en 54,544 s**, sin llamadas pagadas. La regresión comprueba hashes truncados, largos, no hexadecimales, mayúsculas y espacios/saltos sobrantes. Se mantiene el control local de correspondencia con la fuente.

## Límites editoriales

La salida estructurada no garantiza significado, fluidez o nomenclatura correcta. Las alertas de nombres no registrados pueden ser informativas; requieren adjudicación y no se convierten automáticamente en errores. El contexto recuperado es acotado y no equivale a leer una historia o un mod completos. La revisión humana sigue siendo necesaria para términos técnicos, nombres de equipo, registro, género y formulaciones narrativas.

Se conservan por separado aceptación en primera respuesta, recuperación posterior y aprobación humana. Una candidata recuperada no se contabiliza como aceptación original y una corrección humana no se atribuye al modelo.

Fuentes metodológicas consultadas: [Structured Outputs](https://developers.openai.com/api/docs/guides/structured-outputs), que explica la necesidad de manejar errores pese al esquema, y [modelo configurado](https://developers.openai.com/api/docs/models/gpt-6-luna). Los resultados del barrido proceden del código y de los artefactos locales, no de esas páginas.
