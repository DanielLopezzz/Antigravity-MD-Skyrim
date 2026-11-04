# Contexto, memoria de traducción y calidad editorial

## Contrato y contexto

Los nuevos intercambios se emiten como versión 1.1.0; el lector conserva compatibilidad con 1.0.0. ContextEngine construye un paquete con versión propia, perfil, mod, plugin, misión, tópico, hablante y contexto de llamada cuando existen. Se rechazan claves desconocidas, rutas locales y valores excesivos. Los textos y metadatos son datos no confiables, nunca instrucciones. La agrupación deja de mezclar mods o plugins que comparten un identificador de grupo.

El proveedor recibe el paquete en su payload de prueba. Los campos iguales en todo el lote se serializan una vez como `batch_context`; cada ítem conserva las diferencias. Los registros locales mantienen el contexto completo para identidad, auditoría y reintento. El almacén aislado añade `narrative_context_json`, versión 1.1.0 y un registro de migración. Se probó la actualización sobre una copia v1; el original permaneció idéntico. La reversión compatible consiste en usar el respaldo v1 o ignorar las columnas añadidas; no se ejecutó eliminación de columnas ni migración de pilotos reales.

Los prompts comparten una política terminológica y perfiles de diálogo, misión, objeto, magia, libro, UI y script. Solo se incluyen instrucciones de los perfiles presentes. Se permiten cambios sintácticos y reformulación idiomática conservando negación, agente, condiciones, cantidades e incertidumbre. No se inventan género, relación ni hechos de lore a partir de metadatos ausentes.

## TM contextual v2

La identidad nueva combina hash del texto exacto y una firma `tm2:` de dominio, registro/subregistro, hablante, mod, plugin, misión, tópico, grupo y contexto de llamada. La serialización JSON evita colisiones por separadores. Espacios iniciales/finales distintos dejan de compartir la clave nueva.

Las filas v1 se conservan. Se pueden consultar como candidatas de contexto diferente; no se convierten en coincidencias seguras por tener el mismo texto. Las escrituras nuevas y la deduplicación usan v2. El intercambio mantiene su hash v1 por compatibilidad de artefactos; no debe confundirse ese identificador histórico con el nuevo hash exacto de TM. No hubo reescritura masiva ni migración de la TM productiva.

## Aprobación y exportabilidad

El desglose conserva los controles mecánicos existentes y añade evaluaciones de gramática, naturalidad, fidelidad y narrativa con valor inicial `not_assessed`. El antiguo `recommended_status` se mantiene como recomendación mecánica compatible; no acredita revisión humana. Una nueva respuesta real aceptada por Gatekeeper se importa como pendiente, no entra en TM y no propaga duplicados por aprobación automática.

La operación manual registra revisor y fecha, comprueba variables y terminología antes de aprobar y guarda evidencia editorial. El exportador exige esa evidencia para traducciones reales; una fila heredada marcada `approved` sin ella no se borra, pero requiere revisión antes de exportarse. Se mantienen los casos explícitos de simulación y copia de nombres protegidos sin transformación. Los pilotos existentes no fueron reetiquetados retroactivamente. Los fixtures oficiales de la prueba integral ahora realizan la aprobación explícita antes de comprobar la exportación.

## Recuperación contextual local

LocalReferenceIndex es una herramienta optativa separada de la base productiva, con raíces permitidas explícitas, rechazo de rutas productivas y marcador de formato. Indexa pasajes acotados con FTS5 y filtra por mod/misión; devuelve como máximo dos pasajes de 240 caracteres identificados como contexto sin revisar. No utiliza embeddings, red ni servicios de IA.

La conexión automática de pasajes de libros al payload queda pendiente: falta segmentación de libros largos, resolución de entidades, evaluación de relevancia y control de spoilers. La fase sí conecta los metadatos narrativos existentes; no afirma que haya construido un grafo exhaustivo del lore.

## Corpus de evaluación y ejemplos

Se prepararon 24 casos sintéticos en ocho categorías y 19 casos privados extraídos en lectura inmutable del piloto de 125 cadenas. No hay referencias nuevas revisadas por una persona. Cada caso registra original, contexto, candidato o sugerencia, criterio de aceptación y referencia humana nula. Los casos privados permanecen fuera del repositorio documental y de la TM.

Los siguientes son ejemplos escritos para esta fase. La columna propuesta no es salida de un modelo ni una traducción humana certificada.

| Categoría | Original sintético | Ejemplo literal o defectuoso | Propuesta para revisión |
|---|---|---|---|
| dialogue | You have my ear. | Tienes mi oreja. | Te escucho. |
| quest | Find out who took the key. | Encuentra afuera quién tomó la llave. | Averigua quién se llevó la llave. |
| item | Iron Sword | Hierro Espada | Espada de hierro |
| magic | Restore <mag> points of Health. | Restaura <mag> puntos de Salud. | Recupera <mag> puntos de Salud. |
| book | <p>At dawn, we set forth.</p> | <p>Al amanecer, nos pusimos adelante.</p> | <p>Al amanecer, emprendimos la marcha.</p> |
| ui | Enable notifications | Habilita notificaciones | Activar notificaciones |
| description | This effect lasts for <dur> seconds. | Este efecto dura para <dur> segundos. | Este efecto dura <dur> segundos. |
| script | HeelsFix Refresh: Invalid target | Actualizar HeelsFix: objetivo no válido | HeelsFix Refresh: objetivo no válido |

La evaluación futura usará dos revisores ciegos para fidelidad, naturalidad, voz, terminología y adecuación; las discrepancias se adjudicarán por separado. El control determinista cubre estructura y variables. Una pérdida de negación, cambio de agente o invención de lore exige revisión semántica, aunque el JSON y los placeholders sean válidos. La comparación A/B de los prompts y la medición de calidad de Luna/Sol siguen pendientes porque esta fase prohíbe llamadas reales.
