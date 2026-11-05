# Evolución inteligente: implementación y evaluación offline

**Corte: 2026-11-05. Estado: cambios privados implementados; validación offline completada; piloto real y aprobación automática nueva no habilitados.** La publicación se propone mediante una rama y PR documental; no contiene código, corpus, DB, decisiones individuales ni credenciales.

## Resultado ejecutivo

Sobre las mismas 1.000 candidatas históricas, la nueva priorización produce **38 obligatorias frente a 196 (−80,61 %)**: 25 conservan cuarentena técnica y 13 requieren criterio editorial. Se mantienen las 199 de muestra; 763 quedan con prioridad adicional. Son **prioridades de revisión**, no traducciones nuevas aprobadas ni ahorro humano demostrado. No se modificaron las candidatas ni los resultados originales.

La comparación distingue 804 casos de riesgo bajo sin calibración, 158 con avisos informativos, 13 editoriales y 25 en cuarentena. Hay solo tres referencias humanas válidas: dos asignadas a calibración y una a validación. No permiten estimar falsos negativos ni habilitar aprobación automática. No se declara mejora medida de significado o naturalidad.

Se implementaron conocimiento contextual por mod/plugin, un contrato de evidencia acotada, evaluación selectiva simulada, reparación técnica reversible, propuestas terminológicas y coordinación durable. El código queda privado y trazado por hashes en [CHANGE_MANIFEST.json](CHANGE_MANIFEST.json). Las métricas agregadas están en [METRICS.json](METRICS.json); el estimador está documentado en [COST_COMPARISON.json](COST_COMPARISON.json).

## 1. Problemas y decisiones

| Problema observado | Decisión e implementación | Límite |
|---|---|---|
| Palabras capitalizadas implicaban investigación obligatoria | Advertencias de cobertura y ratios pasan a informativas; severidad independiente del score mecánico | No significa que las traducciones sin alertas sean correctas |
| Eliminar saltos concatenaba `20` con la palabra siguiente | Extracción numérica con separadores, ordinales y decimales normalizados | Cantidades escritas y separadores ambiguos requieren adjudicación |
| Un nombre NPC podía ser Dog o una criatura | Se confirma el papel de nombre de registro, no que sea un nombre propio intraducible | Las biografías y la condición de nombre propio necesitan evidencia adicional |
| Contexto mezclado y sin procedencia | Sidecar por mod/plugin, hermanos de grupo, entidades, documentos permitidos y términos acotados | No se reconstruye automáticamente la historia completa de cada mod |
| Consenso de jueces confundible con aprobación | Puerto de juez con errores tipados y spans verificables; sin facultad de aprobar | Solo se ejercitó con mocks; no es una evaluación LLM real |
| Reparaciones fallidas quedaban en vuelo | Estados diferenciados: no enviado, transporte incierto, protocolo, QA y completado | Los libros históricos no se reescribieron |
| Una interrupción podía perder el significado de una respuesta recibida | Diario aislado guarda cuerpo y hash antes de parsear; conserva recibos aunque falte uso fiable | Un envío incierto o conciliación incompleta exige investigación y nunca se reenvía automáticamente |
| Confirmación por cada etapa rutinaria | Coordinador de etapas con plan, alcance y cap previamente fijados, reutilización de etapas completas y detención ante desviaciones | El recorrido real sigue exigiendo autorización explícita; aquí solo se probó simulado |

Se eligió un recorrido ligero: determinismo → contexto pertinente → priorización → juez selectivo → nueva validación. Se descartó pedir dos traducciones de todo por defecto: comparte sesgos y multiplica coste sin evidencia de ahorro. Se prefirió SQLite/FTS5 frente a un servicio de embeddings: no agrega dependencias, endpoints o consumo. La retraducción queda como posible señal auxiliar, no una prueba de corrección.

## 2. Validación, reparación y calibración

`quality/intelligent_review.py` separa `quarantine`, `human_review`, `informative` y `low_risk_uncalibrated`. Comprueba integridad, texto vacío, nuevos caracteres de sustitución, glosario suministrado, cantidades, límite de campo explícito, una señal de negación y atributos distintivos. Las señales semánticas son conservadoras y limitadas; no sustituyen un análisis bilingüe completo.

El caso Masked mantiene una alerta de atributo omitido. Los avisos numéricos anteriores se reconocen como resueltos cuando coinciden los valores normalizados. Recuperación previa o resolución de un falso rechazo es procedencia, no por sí misma un error lingüístico actual. No se pierde la alerta histórica: la copia experimental conserva los motivos originales y añade `review_risk`.

La reparación automática implementada es deliberadamente estrecha: propone restituir controles terminales faltantes si la validación completa de variables vuelve a pasar. Guarda original y propuesta; no altera posiciones internas, palabras, términos ni aprueba el resultado. En las candidatas finales del piloto no había reparaciones terminales aplicables; la capacidad se verificó con casos adversariales sintéticos. No hubo correcciones lingüísticas reales nuevas.

El juez independiente recibe original, candidata y contexto como datos no confiables. Devuelve categorías inspiradas en MQM, severidad y spans que deben existir en el original/candidata. No recibe el dictamen previo como instrucción y no devuelve una aprobación. Una segunda configuración del mismo modelo no se considera independencia estadística. El puerto bloquea evaluadores vivos en esta fase; no se añadieron endpoints ni operaciones externas para evaluar o investigar.

La calibración agrupa familias narrativas completas y une originales idénticos entre familias para impedir fugas. Exige etiquetas humanas booleanas, política/evaluador identificados y validación retenida independiente. Como puerta experimental, exige al menos 300 familias independientes que habrían sido aprobadas, ninguna falsa aprobación y límite unilateral del 95 % inferior al 1 %. La fórmula no demuestra riesgo cero ni elimina la obligación de auditar por dominio. No existe un camino activo que convierta la coincidencia de dos jueces en aprobación nueva.

## 3. Conocimiento de mods y glosario

`lore/mod_knowledge.py` crea un sidecar identificado y separado. Rechaza nombres de DB productiva, rutas prohibidas y bases ajenas. Las conexiones se cierran; la consulta para preparar payloads puede abrirse en modo de solo lectura. No migra la producción.

Se indexaron **12.423 registros NPC reales** de la fuente local y las 1.000 fuentes del piloto. La evidencia de nombre apunta al UID y al contenido original, con alcance exacto de mod/plugin; no afirma historia, sexo, autoría o personalidad. Un término como Anterio puede reconocerse como nombre de registro si existe esa evidencia. Anterio se usó como fixture sintético: no se afirma haber localizado un personaje real con ese nombre.

Las fuentes admiten procedencia y condiciones de uso. `metadata_only` guarda la referencia sin incorporar el cuerpo. Documentación cuyo permiso de uso está establecido puede agregarse de forma explícita como fragmento acotado; no existe un crawler masivo ni descarga de traducciones comunitarias. Se investigaron páginas de Missives, Alternate Perspective y documentación técnica de Papyrus; no se incorporó su texto remoto al corpus. No se investigaron manualmente miles de páginas de mods.

La recuperación prioriza términos aprobados del ámbito, entidades verificadas, documentación permitida y hermanos; nunca cruza mod/plugin. Cada fragmento es de hasta 320 caracteres y se envían como máximo cuatro, con procedencia opaca hashada. Las inferencias permanecen identificadas y no sobrescriben hechos confirmados.

Se revisaron las **109 preferencias heredadas**: sin duplicados por origen normalizado ni conflictos simples de `keep_original`; las 109 referencias heredadas carecen de URL precisa. Eso exige mejorar evidencia, no reemplazar sus traducciones. No se modificó ningún término aprobado. Se crearon **dos propuestas locales de denominación**, sin aprobar, basadas en coherencia de familia y explícitamente no certificadas como denominación oficial. Las propuestas no entran al prompt como términos aprobados ni alteran el Glosario Maestro.

Un término aprobado del ámbito no invalida silenciosamente un término global vinculante: los conflictos requieren adjudicación. No se implementó inferencia libre de flexiones o sinónimos que rebaje esa protección.

## 4. Prompts, contratos y lotes

El contrato de entrada pasa a **1.2**, manteniendo lectura 1.0 y 1.1. Añade evidencia tipada, acotada y sanitizada; el prompt distingue lore original de mod, conceptos técnicos, nombres e inferencias. Conserva voz ES-419, negaciones, condiciones, cantidades e integridad, sin exigir literalidad. No se cambió el contrato de respuesta para añadir herramientas o investigación.

El adaptador acepta un proveedor opcional de contexto y estima el payload completo. Una preparación nueva puede copiar por backup SQLite un snapshot contextual al staging, sellarlo por hash y regenerarlo idéntico en preflight/envío. La agrupación coherente es opcional para conservar compatibilidad y evitar costes inadvertidos. Mantiene mod/plugin y familias narrativas; UI/objetos pueden compartir perfil. Los textos demasiado largos requieren segmentación explícita; no se recortan para hacerlos caber.

Los planes históricos no se resellan para aceptar código nuevo. Sus archivos siguen intactos; una ejecución futura debe tener identidad, fingerprint y presupuesto nuevos.

## 5. Comparación medida y costes

| Dimensión | Antes | Nuevo resultado offline |
|---|---:|---:|
| Candidatas comparadas | 1.000 | Las mismas 1.000 |
| Prioridad obligatoria | 196 | 38 |
| Cuarentena técnica | 25 | 25 |
| Muestra sin alertas prioritarias | 199 | 199 |
| Aprobaciones humanas creadas por esta fase | 0 | 0 |
| Traducciones reescritas por esta comparación | 0 | 0 |
| Referencias humanas disponibles | 3 | 2 calibración / 1 validación |
| Falsos negativos, fidelidad y naturalidad globales | Sin referencia suficiente | No medidos; no se atribuye mejora |
| Tiempo de comparación/indexación real acotada | Sin benchmark equivalente | 2,533 s en este entorno |

La reducción de prioridad **no es** un descenso demostrado del trabajo humano ni una tasa de precisión del clasificador. No se reevaluaron las 1.000 cadenas mediante una IA real. Los casos de interfaz largos pueden ser correctos; una cota real de campo, si está disponible, vuelve a elevarlos a revisión.

Se midieron dos preparaciones estimadas sobre las mismas fuentes bajo el prompt nuevo, sin enviar solicitudes:

| Estrategia | Lotes | Tokens de entrada estimados | Salida estimada | Reserva local estimada USD |
|---|---:|---:|---:|---:|
| Mezclada, sin recuperación | 43 | 321.967 | 140.552 | 0,16704072 |
| Coherente, con recuperación | 431 | 1.066.714 | 141.965 | 0,39955632 |

No es una comparación de facturas ni del prompt histórico. El muestreo disperso provoca muchos grupos pequeños: la segunda opción excede el tope previo de US$0,17. No se habilitó por defecto ni se elevó ningún límite. La próxima prueba debe escoger un ámbito denso de un mod y estimarlo antes de pedir autorización; no repetir las 1.000 para “demostrar contexto”. Hay 13 casos propuestos para evaluación selectiva, pero su coste y ahorro humano no se consideran medidos.

## 6. Pruebas, seguridad y escala

La línea base fue **110/110 en 21,541 s**. La suite final y su duración exacta se registran en `METRICS.json`; incluye nuevas regresiones de cantidades, avisos, contexto, permisos, contratos, jueces, calibración, reparación, concurrencia y reanudación.

Las pruebas cubren: no filtrar mod/plugin; no tratar NPC genérico como nombre propio; no copiar fuentes sin permiso; no aprobar propuestas sin autorización; impedir fuga entre splits; rechazar evaluadores vivos y hallazgos inventados; reparar únicamente controles recuperables; preservar respuestas con uso inválido; no duplicar llamadas/contabilidad al reanudar; reclamar una solicitud una sola vez en concurrencia; detener etapas por presupuesto y no extender autorización.

La preparación de payloads sobre fuentes reales descubrió un rechazo por espacio al truncar contexto. Se corrigió haciendo la sanitización idempotente y se añadió una regresión; se volvió a verificar la suite. Se conservan checkpoints y el primer experimento privado que detectó los falsos positivos de nombres NPC.

Benchmark adicional: **240.684 pasajes sintéticos**, ingesta SQLite en **11,884 s**, archivo **66.994.176 bytes**, integridad OK. `tracemalloc` registró 21.265 bytes de pico de asignaciones Python durante el recorrido: no mide RSS, cachés nativas de SQLite ni el pico de toda la aplicación. El acceso FTS se actualiza por rowid indexado para evitar búsquedas lineales repetidas al reinsertar. No acredita escala de todos los parsers ni calidad del inventario completo.

Se verificaron **2.186 archivos protegidos idénticos por tamaño, mtime y SHA-256**, incluidos datos productivos, revisiones y pilotos. Candado activo, cero proveedores invocados y cero créditos. Los nuevos sidecars, copias experimentales y paquetes de código quedan privados; las tarifas y caps existentes no cambiaron.

## 7. Uso local, límites y siguiente puerta

Para explorar la copia experimental, usa **Abrir otra prueba…** en el visor y elige `NolvusAIT_Intelligence/20261105/private/final_comparison`. Tiene las mismas candidatas, nueva prioridad y evidencia de riesgo; no se copiaron ni crearon decisiones humanas. La prueba original R7 sigue en su carpeta y conserva tus revisiones.

El workbench se ejecuta como módulo `nolvus_translator.translation.intelligence_workbench`, con `--source`, `--pilot`, `--output` nuevo y `--benchmark-count` opcional. No reemplaza un experimento existente. La preparación acepta `knowledge_path` y `coherent_batches`; el coordinador `execute_authorized_stages` permite etapas acotadas con una autorización. Su vía facturable no se ejercitó durante esta fase. El diario durable independiente y el juez permanecen habilitados solo para simulación; no se sustituyó a ciegas todo el runner real por infraestructura no probada.

**Dictamen:** la fase mejora clasificación, trazabilidad contextual y seguridad operativa con evidencia offline. Aún no permite aprobar masivamente nuevas traducciones ni afirmar mejora semántica. Antes del piloto real de un mod: revisar la muestra humana retenida y los no alertados, adjudicar denominaciones, calibrar con más familias, validar la integración facturable bajo autorización nueva y demostrar un presupuesto compatible con el agrupamiento seleccionado. Las ambigüedades siguen necesitando criterio editorial; no se convierten en biografías inventadas o reglas permanentes.

## Fuentes y trazabilidad

[Registro de fuentes y condiciones](SOURCES.json) · [Inventario de cambios y hashes](CHANGE_MANIFEST.json) · [Métricas](METRICS.json) · [Costes estimados](COST_COMPARISON.json) · [Integridad](PROTECTION.json).

Se consultaron [OpenAI Structured Outputs](https://developers.openai.com/api/docs/guides/structured-outputs), [evaluación](https://developers.openai.com/api/docs/guides/evaluation-best-practices), [MQM](https://www.themqm.org/mqm-pillars/typology/), [GEMBA-MQM](https://aclanthology.org/2023.wmt-1.64/), [contexto documental](https://aclanthology.org/2025.mtsummit-1.10/), [posedición en videojuegos](https://aclanthology.org/2025.ctt-1.3/) y [estimación de calidad en posedición](https://aclanthology.org/2025.mtsummit-1.38/). Son fundamentos y límites de diseño, no evidencia de resultados lingüísticos propios.
