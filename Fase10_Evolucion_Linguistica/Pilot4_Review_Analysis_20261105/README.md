# Análisis de revisión humana y evaluación profesional del piloto de 1.000 cadenas

**Fecha documental:** 2026-11-05, America/Bogota. **Estado:** análisis completado; propuestas pendientes de autorización e implementación. El corte de revisiones humanas corresponde a la inspección que originó este informe, no a un contador en tiempo real.

Se inspeccionaron código, candidatas, decisiones humanas y evidencia local en modo de solo lectura, sin HTTP ni consumo de créditos durante el análisis. No se modificaron código, configuraciones, glosario, TM, bases, decisiones ni resultados. Se verificaron 1.228 archivos controlados idénticos y el candado de API en `True`. La publicación documental posterior en GitHub es una acción separada y autorizada.

Antecedente: [barrido y resultado completo del piloto](../Pilot4_Completed_20261104/README.md). Sus métricas e informes se conservan como evidencia histórica; este análisis añade interpretación y detecta falsos positivos sin reescribirlos.

## 1. Diagnóstico del sistema de revisión humana

**196 casos obligatorios no significan 196 traducciones incorrectas.** La política de importación exige revisión editorial para toda traducción real nueva; esos 196 tienen prioridad obligatoria. El visor guarda las decisiones humanas en un archivo separado, sin promoverlas a TM o producción ni resolver una cuarentena de protocolo.

En el corte inspeccionado había **tres aprobaciones humanas válidas y sin cambios**, todas de casos obligatorios. Quedaban **997 pendientes en el visor**. La DB conservaba 834 pendientes, 141 por investigar lore y 25 sin importar; las decisiones del visor no cambian esos estados. Las 1.000 pendientes del informe anterior pertenecen al corte previo a esas tres aprobaciones.

| Conjunto | Casos | Relación |
|---|---:|---|
| Revisión obligatoria | 196 | Incluye los 42 señalados por auditoría |
| Muestra de calidad | 199 | No se solapa con los obligatorios |
| Auditoría editorial adicional | 42 | 25 del lote retenido y 17 adicionales |
| Unión de obligatorios y muestra | **395** | No son 437 |
| Prioridad adicional | 605 | No equivale a aprobación automática |

De los 42 señalados, **33 ya tenían otra alerta**; nueve adquirieron prioridad obligatoria por la auditoría. Hay **233 mensajes distribuidos entre 196 casos**, con 35 casos que acumulan varios motivos. La muestra se elige por categoría/cohorte y orden determinista de hashes: comenzó con 200 y uno pasó a obligatorio tras la auditoría. No es una selección aleatoria uniforme del inventario.

Referencias locales: `translation/pipeline.py:650`, `translation/pilot_four_execution.py:93` y `ui/case_review.py:74`. Las rutas de módulos citadas en este informe son relativas a `src/nolvus_translator` en el repositorio de código local; el código no se publica en este repositorio documental.

## 2. Clasificación de las alertas y falsos positivos

Los recuentos siguientes se solapan:

| Regla | Casos | Qué detecta | Evaluación |
|---|---:|---|---|
| Posibles nombres o lore desconocidos | 141 | Dos o más palabras inglesas con mayúscula no reconocidas por la heurística | Advertencia preventiva, no prueba de traducción incorrecta |
| Longitud atípica | 15 | Ratio moderadamente fuera del intervalo esperado | Señal débil entre idiomas |
| Longitud severa | 5 | Ratio más extremo | No demuestra desbordamiento sin medir espacio disponible |
| Exclamación final perdida | 2 | Diferencia de puntuación | Diferencia comprobada; gravedad contextual |
| Cuarentena atómica | 25 | Un registro con hash de origen incompleto | Un error de protocolo retiene todo el lote |
| Auditoría editorial adicional | 42 | Procedencia, reparaciones, cifras y dudas concretas | Mezcla trazabilidad con posibles problemas lingüísticos |

La alerta de nombres usa palabras de cuatro o más letras con mayúscula inicial; excluye heurísticamente inicios de frase, ciertas etiquetas Title Case y términos coincidentes del glosario. Cuando quedan al menos dos candidatos, reduce la familiaridad terminológica a 0,70 y recomienda investigar lore. No comprueba si esos nombres están bien traducidos o preservados. **132 casos tienen únicamente esta alerta.**

La longitud usa ratios: para originales de más de diez caracteres, considera normal 0,65–1,65; advierte dentro de 0,45–2,10 si se sale del intervalo normal y califica de severo lo restante. No usa ancho en píxeles ni un límite real del campo. El componente `context_coherence` solo compara signos finales `?` y `!`: no evalúa significado, gramática o contexto narrativo.

El resto del validador comprueba integridad de variables/etiquetas, contenido, codificación, términos obligatorios y una lista de formas regionales. El Gatekeeper bloquea fallos críticos, pero permite candidatas con advertencias de longitud, puntuación o nombres desconocidos. La ausencia de alertas no certifica naturalidad o fidelidad.

### Ejemplos reales y clasificación

| Original → candidata | Hallazgo |
|---|---|
| “A boy, up in Windhelm, name of Aventus Aretino.” → “Un chico de Ventalia, llamado Aventus Aretino.” | Alerta por Aventus/Aretino; ya aprobada por el usuario sin cambios. Nombre no registrado no equivale a error |
| “Save user settings” → “Guardar configuración del usuario” | Expansión natural; la advertencia no demuestra desbordamiento |
| “Ancient Shrouded Masked Cowl” → “Capucha antigua envuelta” | Omisión observable de la distinción explícita Masked; requiere corrección o adjudicación contextual |
| “Invalid Form Error.” → “Error: formulario no válido.” | Probable falso amigo técnico; comprobar el sentido de Form en Papyrus |
| “Shrouded Gloves” → “Guantes envueltos” | Revisar denominación: otras piezas Shrouded de la muestra usan “de la Hermandad Oscura” |
| “Frostbite Venom” → “Veneno de escarcha” | Duda de sentido y denominación; no adjudicar sin contexto del efecto |

**Dos falsos positivos numéricos confirmados:** el script de auditoría elimina marcadores de salto sin dejar separadores y convierte un número al final de una línea en texto como `20You`. Su regex deja de reconocerlo. También compara ordinales ingleses como `2nd` y `4th` con fechas españolas como `2 de…` y `4 de…` sin normalizarlos. En los dos casos señalados no se demostró alteración de cantidades. El defecto está en `NolvusAIT_Pilot4_Sweep/20261104/final_audit.py:64` del workspace local y no se corrigió durante este análisis.

Hay además **dos falsos rechazos históricos documentados**, ya corregidos antes de este análisis: proteger el nombre común missives como marca y exigir el plural Renegados ante la forma correcta “algún Renegado”. No deben sumarse a los falsos positivos actuales como si pertenecieran al mismo corte.

No existe todavía una tasa global fiable de errores o falsos positivos: solo se inspeccionó una selección lingüística y hay tres aprobaciones humanas. Las tres confirman candidatas aceptables bajo la revisión realizada; no demuestran que toda alerta de lore sea inútil. Tampoco se consideran errores confirmados los 42 señalamientos editoriales. Los 25 retenidos incluyen un fallo de protocolo; no prueban 25 fallos de traducción.

Referencias locales: `quality/validator.py:33`, `quality/gatekeeper.py:519` y `lore/editorial_policy.py:71`.

## 3. Propuesta de reevaluación inteligente

**Diseño propuesto, no implementado ni autorizado para ejecución.**

Recorrido recomendado: validación determinista → depuración de advertencias → contexto pertinente → evaluación bilingüe selectiva → clasificación por riesgo → auditoría humana de resultados.

- **Aprobación automática inicial:** solo reutilización trazable de una referencia humana aprobada, con coincidencia de original, traducción y contexto, reglas vigentes satisfechas, lote íntegro y ninguna discrepancia pendiente. No crear una aprobación humana ficticia. Las traducciones nuevas seguirían sin aprobación automática hasta calibrar y validar la estrategia.
- **Revisión humana:** ambigüedad, voz narrativa, humor, poemas, nombres o denominaciones pendientes, falta de contexto, discrepancias entre evaluadores y reparaciones sin adjudicación humana.
- **Cuarentena:** hashes/identidad incorrectos, registros ausentes, duplicados conflictivos, pérdida de placeholders/controles, codificación incompatible e incumplimientos críticos demostrados. Una sospecha semántica sin evidencia suficiente pasa a revisión, no a error confirmado.

Una segunda evaluación de Luna podría recibir original, candidata y contexto, sin el dictamen previo, y devolver hallazgos con fragmentos justificativos. No debería reescribir toda la traducción. Cambiar la configuración del mismo modelo no garantiza independencia: subsisten sesgo compartido y falsas aprobaciones coincidentes. Sol sería una opción excepcional para discrepancias con contexto suficiente, no fallback general. Ningún modelo compensa información contextual ausente.

La retraducción sería una señal auxiliar: puede reconstruir el sentido esperado, repetir el mismo error o penalizar una paráfrasis correcta. La naturalidad ES-419 debe conservar intención, voz y trato, sin exigir estructuras literales por neutralidad regional.

Antes de gastar tokens, resolver determinísticamente nombres preservados, ratios y formatos numéricos. Agrupar dudas repetidas de entidades permite investigar una denominación una vez, sin aprobar automáticamente frases diferentes. No reevaluar indiscriminadamente las 1.000 cadenas; presupuestar y acotar cualquier futuro experimento. En este análisis no se consultaron tarifas ni capacidades actuales por HTTP.

### Medición necesaria

Evaluación humana ciega, estratificada por categoría/riesgo, con familias de textos separadas entre calibración y validación. Medir precisión de alertas, advertencias sobre traducciones aceptables, errores encontrados entre los no alertados, falsas aprobaciones, tiempo humano y coste adicional por error detectado. La auditoría de supuestos aprobados es obligatoria para medir falsos negativos.

No se puede demostrar riesgo cero. Como ilustración estadística, 199 casos independientes revisados sin errores darían un límite superior aproximado del 1,5 % con confianza unilateral del 95 %. No es un resultado obtenido: esos 199 aún no constituyen una muestra humana evaluada, y sus dependencias reducen la evidencia efectiva. No usar una puntuación del modelo como probabilidad calibrada.

## 4. Evaluación profesional de la prueba de 1.000 cadenas

**Opinión técnica:** resultado técnico útil y calidad prometedora, con estabilidad y aprobación lingüística aún pendientes.

- **Hecho medido:** 975/1.000 importaciones mecánicas, 97,5 % acumulado tras continuaciones y reparaciones. No es aceptación a primera respuesta ni precisión semántica. No fue una única ejecución ciega con configuración invariable.
- **Integridad medida:** la auditoría final registró cero diferencias de placeholders y controles en las 975 importadas. Hubo fallos previos; un poema terminó en una propuesta explícita de formato de IA pendiente humana.
- **Calidad observada:** hay formulaciones fluidas y tres decisiones humanas sin cambios. No basta para estimar calidad global. La omisión de Masked y la variación Shrouded muestran riesgos que la validez mecánica no resuelve.
- **Contexto medido:** ninguna de las 200 cadenas de diálogo tiene `speaker_hint`. El runner corta por tamaño sin usar la agrupación narrativa completa de ContextEngine; en R7 hubo un lote con 18 grupos contextuales distintos. Los IDs ayudan, pero no equivalen a entregar la conversación o la historia.
- **Reparaciones medidas:** cuatro solicitudes adicionales para cinco cadenas; dos pasaron individualmente las reglas entonces vigentes. El poema siguió fallando y dos reparaciones respondían a falsos rechazos locales. Solo una solicitud completa pasó sin intervención posterior. Coste de esas cuatro reparaciones: US$0,0023423.
- **Economía medida:** 50 respuestas reales, 299.579 tokens de entrada y 175.409 de salida; US$0,1176624 acumulados. Es conciliación local, no certificación de factura ni proyección validada del inventario completo. No hay medición de tiempo humano que permita calcular el ahorro neto.
- **Estabilidad observada:** presupuesto, evidencia e integridad productiva protegidos; intervención supervisada repetida. Todavía no se demuestra operación autónoma sostenida.

La suite histórica final fue 110/110 después del refuerzo de SHA-256, verificado offline; no hubo otra llamada pagada que validara ese último cambio. No se ejecutaron tests nuevos durante este análisis.

Referencias locales: `translation/pilot_four.py:199`, `lore/context_engine.py:64`, `lore/reference_index.py:39`, `translation/openai_provider.py:1273` y `translation/pilot_recovery.py:108`.

## 5. Problemas y oportunidades detectados

1. Todas las advertencias elevan prioridad de forma similar, aunque nombres desconocidos, dudas editoriales y cuarentena requieren tratamientos distintos.
2. Los estados de reparación mezclan transporte y QA: hay `inflight` históricos con respuesta recibida y gasto conciliado. No representan solicitudes activas ni incertidumbre económica, pero dificultan la interpretación y reanudación.
3. Las 42 anotaciones editoriales provienen de un script adicional, no del validador habitual. Su procedencia y persistencia deben integrarse antes de tratarlas como clasificación estable.
4. El ratio de longitud carece de límites reales de interfaz y la puntuación contextual no examina significado. Las comprobaciones de negaciones, omisiones y adiciones no están cubiertas por una evaluación semántica general.
5. La cuarentena atómica protege consistencia, pero necesita un expediente técnico separado para no presentar un hash defectuoso como 25 errores lingüísticos.

## 6. Recomendaciones priorizadas

Todas son propuestas pendientes de autorización; su publicación no autoriza código, cambios de estado ni consumo.

| Prioridad | Recomendación | Evidencia |
|---|---|---|
| P1 | Separar bloqueos, avisos informativos, procedencia y dudas editoriales | 233 mensajes sobre 196 casos; 42 se solapan con el conjunto obligatorio |
| P1 | Corregir análisis numérico y diagnosticar antes de pagar reparaciones | Dos alertas numéricas falsas; dos falsos rechazos históricos |
| P1 | Calibrar nombres desconocidos y longitud por dominio | 132 casos solo tienen la alerta de nombres; falta espacio real medido |
| P1 | Adjudicar familias terminológicas y aportar contexto pertinente | Variación Shrouded, omisión Masked y ausencia de hablante |
| P1 | Separar envío, conciliación y resultado QA | `inflight` históricos que no significan transporte activo |
| P2 | Evaluar un segundo juez en modo experimental | Debe demostrar ahorro de trabajo sin aumentar falsas aprobaciones |

Conservar evidencia original, hashes, importación atómica, presupuesto acotado, reutilización de válidos y separación entre candidatas, referencias humanas y TM. No convertir entidades detectadas o propuestas de glosario en aprobaciones léxicas automáticas.

## 7. Dictamen de preparación para el siguiente piloto

**No se recomienda aumentar nuevamente la carga con el funcionamiento actual sin completar la calibración.** Hay base para preparar después un piloto acotado de un mod, con contexto coherente y criterios de salida definidos.

Secuencia propuesta:

1. Completar revisión humana representativa, incluyendo candidatas sin alertas.
2. Adjudicar dudas terminológicas y el expediente del lote retenido, conservando la respuesta original y sin retransmitir automáticamente el lote completo.
3. Autorizar e implementar las correcciones deterministas prioritarias.
4. Medir la clasificación propuesta sin habilitar aprobaciones nuevas automáticas.
5. Autorizar el siguiente piloto, congelar configuración y medir calidad e intervención humana.

El siguiente avance debe demostrar menos revisiones innecesarias y menos errores no detectados, además de mayor volumen. Este informe no cambia la autorización de producción, la TM ni las decisiones humanas.
