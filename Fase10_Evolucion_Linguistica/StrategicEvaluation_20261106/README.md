# Viabilidad de traducción masiva de Nolvus

Evaluación estratégica independiente · 6 de noviembre de 2026 · Nolvus Awakening 6.0.21

**Dictamen: B. EVOLUCIONAR.** Conservaría el motor de traducción y los componentes de dominio, pero rediseñaría la ejecución masiva, el presupuesto compartido y la salida editorial. La arquitectura actual **no está lista para completar de forma fiable y autónoma las 240.684 cadenas**. Tampoco hay evidencia comparativa que justifique sustituirla íntegramente por otro modelo, una red de agentes o un sistema local.

El obstáculo principal es convertir candidatas en un resultado coherente, recuperable y utilizable por una sola persona. El coste de generación parece asumible; no está demostrada la calidad ni la reducción de trabajo humano del flujo completo. Esta distinción evita justificar la continuidad por el trabajo ya invertido.

Esta evaluación produjo únicamente este informe y agregados de cálculo. La publicación incluye solo documentación y métricas agregadas; las bases, decisiones humanas y evidencias privadas permanecen locales. No modifica código, bases, glosario, TM, ramas, Nolvus o MO2; no realiza traducciones ni solicitudes a modelos. No autoriza una migración, un gasto o una publicación.

## 1. Diagnóstico de la arquitectura actual

### Lo medido

La lectura inmutable del inventario existente, sin volver a escanear Nolvus, encontró:

| Medición | Resultado | Consecuencia estratégica |
|---|---:|---|
| Registros del snapshot | 403.922 | No son traducciones únicas pendientes. |
| Ocurrencias clasificadas como traducibles | 313.731 | Incluyen repeticiones. |
| Textos exactos distintos pendientes | 235.301; 20.077.501 caracteres | Base observada para modelar el volumen. |
| TM productiva | 0 entradas | No hay ahorro inicial demostrado por TM. |
| Mediana / media de longitud | 41 / 85,33 caracteres | La media oculta una cola narrativa importante. |
| Concentración de caracteres | El 1 % de textos reúne el 32,96 % | Libros y textos largos necesitan tratamiento propio. |
| Textos mayores de 12.000 caracteres | 69; máximo 39.975 | No caben íntegros bajo el umbral de agrupación usado en la aproximación. |
| Unidades distintas al conservar ámbito contextual | 278.933 | Un texto idéntico no garantiza un significado o una traducción idénticos. |
| Diálogos con hablante identificado | 2.847 de 148.192 ocurrencias: 1,92 % | Hay una carencia concreta de contexto de voz/personaje. |

Las **240.684 cadenas** siguen siendo la referencia histórica solicitada. Difieren en 5.383 de los textos únicos de la consulta principal; el histórico incorpora otras fuentes/formatos. Esa diferencia requiere un manifiesto final por formato e identidad, no una nueva suma ni una exclusión automática. Las 48.051 fuentes distintas ya clasificadas como traducidas y los nombres protegidos están fuera de los 235.301 pendientes: descontarlos otra vez falsearía el ahorro.

Una aproximación del agrupador coherente, tomando una sola ocurrencia por texto exacto, produce **73.110 lotes para 235.232 textos**, media **3,218 cadenas**, incluidos **45.118 lotes unitarios**. No es un plan real completo: excluye los 69 textos largos y todavía podrían dividirlo los límites de tokens/bytes. Demuestra que «lotes de hasta 25» no significa «25 de media».

Evidencia reproducible: [inventario](inventory_readonly.json), [contexto y agrupación](inventory_context.json).

### Qué funciona y qué impide escalar

- **Base útil:** protección de variables, contratos de identidad, validación estructural, glosario con ámbito, referencias de nombres/familias, staging aislado, recibos persistidos y exportación reversible. Son componentes específicos de Skyrim que un cambio de proveedor no reemplaza.
- **Integración todavía experimental:** `BatchWorkflow` y `DurableRun` aceptan simulaciones; los HTTP reales usan ejecutores de pilotos separados. Las 332 pruebas registradas y la carga sintética de 1.000 fuentes respaldan comportamientos offline, no una traducción masiva real.
- **Contexto parcial:** existen registros y relaciones útiles, pero 41 documentos locales corresponden a 36 mods; las 1.220 páginas candidatas no están verificadas. Diez referencias externas de siete mods tampoco equivalen a conocimiento completo de sus versiones o historias. Un nombre NPC indexado no aporta automáticamente hablante, intención o biografía.
- **Evaluación selectiva mal dimensionada para una ejecución enorme:** la política integrada aplica `min(ceil(N × 0,02), 14)`. Un único plan de 240.684 entradas tendría como máximo 14 revisiones iniciales, no 4.814. Además, riesgo y muestra compiten por la cuota. Las reservas de un ciclo completo revisión/corrección/recomprobación pueden diferir casos aunque luego no necesiten corrección.
- **Presupuesto y concurrencia:** los pilotos mantienen libros económicos separados por ejecución y habilitan temporalmente el proveedor mediante un indicador global del proceso. Abrir muchos ejecutores en paralelo no crea un presupuesto global seguro. Primero hace falta un único despachador y un libro compartido.
- **Recuperación:** conservar un envío incierto sin repetirlo es correcto. Detener todo el trabajo por cualquier incertidumbre local limita la continuidad. Se necesitan unidades independientes, reservas retenidas y una política global de incertidumbre; no reintentos indiscriminados.
- **Complejidad del plan:** cada reclamación de trabajo en `DurableRun` vuelve a serializar y comparar el plan completo. Para un plan monolítico aparece trabajo potencialmente cuadrático. Es una inferencia del código, sin benchmark de tiempo; aconseja cohortes acotadas y registros inmutables indexados.
- **Salida editorial:** la exportación normal de resultados reales exige aprobación editorial e identificación del revisor. Las nuevas candidatas permanecen pendientes. Aprobar individualmente 240.684 entradas a 30 segundos por entrada supondría **2.006 horas**, sin investigación de lore ni pruebas dentro del juego. Ese flujo no satisface el objetivo personal.

Los ocho controles sintéticos Sol aprobados en la última ejecución no calibran el juez sobre traducciones naturales. Esa ejecución dejó cero traducciones nuevas y cero revisiones naturales; conserva US$0,00249593 de reserva incierta. La importación mecánica histórica del 97,5 % tampoco mide exactitud semántica. **Hoy no existe una tasa de falsas aprobaciones ni una mejora lingüística atribuible a Sol demostradas sobre una muestra natural representativa.**

Evidencia local: [estado vigente](../../documentacion/ESTADO_ACTUAL.md), flujo integrado (`src/nolvus_translator/translation/batch_workflow.py:29`, referencia local; código no publicado), integridad del plan (`src/nolvus_translator/translation/durable_run.py:131`, referencia local; código no publicado) y condición editorial de exportación (`src/nolvus_translator/export/mo2_packager.py:178`, referencia local; código no publicado).

## 2. Alternativas investigadas

**Modelo avanzado único.** Puede simplificar el flujo y quizá evitar correcciones, pero no elimina validación, contexto, persistencia ni control terminológico. Su calidad superior en este inventario no está medida. No lo descartaría solo por precio: ahorrar muchas horas personales puede justificar más gasto de API. Hace falta comparación ciega con el mismo contexto.

**Agentes de traducción, reflexión y mejora.** El proyecto [translation-agent](https://github.com/andrewyng/translation-agent) implementa esas tres etapas. Es una técnica plausible, no evidencia de superioridad para Nolvus. Aplicarla a cada cadena multiplica pasadas y puede introducir cambios de estilo innecesarios. La usaría únicamente para errores sustantivos identificados y con una corrección máxima. La investigación sobre [jueces LLM](https://arxiv.org/abs/2306.05685) documenta sesgos: otro modelo no equivale a una verificación independiente de la verdad. [GEMBA-MQM](https://arxiv.org/abs/2310.13988) aporta un patrón útil de evaluación por errores y fragmentos, no una garantía de aprobación automática.

**RAG y memoria.** Son complementos necesarios, no un sustituto del motor. Primero usaría identidades de registro, referencias cruzadas, búsquedas textuales y vecinos narrativos; después recuperación semántica si demuestra relevancia adicional. Las coincidencias semánticas servirían como contexto, no como traducciones exactas aprobadas. La investigación de [traducción de novelas](https://aclanthology.org/2023.acl-long.435/) evalúa problemas entre frases —entidades, terminología y correferencias— que una buena frase aislada puede ocultar.

**CAT/TMS profesional.** [Weblate](https://docs.weblate.org/en/latest/admin/machine.html) combina contexto de claves/notas, glosario y QA con traducción automática. Es útil si la interfaz actual deja de cubrir esas necesidades; incorporar ahora una segunda plataforma añadiría sincronización y mantenimiento. [Phrase](https://support.phrase.com/hc/en-us/articles/9386879702044-Translation-Memory-Quality) documenta separar memoria de trabajo y memoria maestra revisada: ese patrón sí resuelve un problema actual. La localización audiovisual aporta criterios de intención y nombres, como la [guía de Netflix](https://partnerhelp.netflixstudios.com/hc/en-us/articles/217349997-Spanish-Latin-America-Spain-Timed-Text-Style-Guide); sus límites de subtítulos no se deben imponer a libros o HUD de Skyrim.

**Servicios de traducción y otros proveedores.** Google ofrece NMT y Translation LLM con precios por caracteres; DeepL permite glosario y contexto no facturado según su [contrato de traducción](https://developers.deepl.com/api-reference/translate/request-translation). No hay comparación local de naturalidad ES-419, lore o carga humana entre ellos y Luna. La [estructura actual de planes DeepL](https://support.deepl.com/hc/en-us/articles/360021200939-DeepL-API-plans) cambió: no usaría precios antiguos de API Pro para afirmar que es más barato. Su precio aplicable y soporte exacto de la variedad deseada quedan por confirmar. Gemini es otro candidato de contraste; una tarifa menor no demuestra mejor resultado.

**Modelos locales e híbridos.** El equipo tiene unos 31,9 GiB de RAM y una Radeon RX 9070 XT, cuyo modelo anuncia [16 GB](https://www.amd.com/en/products/graphics/desktops/radeon/9000-series/amd-radeon-rx-9070xt.html). [TranslateGemma](https://huggingface.co/google/translategemma-4b-it) ofrece variantes 4B/12B/27B; sus resultados publicados no prueban la calidad en este proyecto. Un ensayo aislado de 4B sería razonable para tareas auxiliares o como candidato, sin comprar hardware. Compatibilidad, memoria efectiva, tokens/s y calidad siguen sin medirse; el soporte AMD depende del [entorno concreto](https://rocm.docs.amd.com/projects/radeon-ryzen/en/latest/docs/compatibility/compatibilityrad/windows/windows_compatibility.html). No descargaría ni desplegaría un servicio local antes de definir qué mejora debe demostrar.

**Herramientas específicas de Skyrim.** [xTranslator](https://github.com/MGuffin/xTranslator/blob/main/README.md) ya documenta diccionarios, intercambios y relaciones de diálogo/NPC. Evaluaría sus salidas y las de xEdit como referencias antes de escribir otro extractor equivalente; no está probada su integración automática con nuestro inventario. [SSE Auto Translator](https://github.com/Cutleast/SSE-Auto-Translator) automatiza obtención de traducciones existentes: sería relevante bajo otro requisito, pero contradice la política vigente de 0 % de traducciones comunitarias. [DSD](https://github.com/SkyHorizon3/SSE-Dynamic-String-Distributor) pertenece a la entrega de traducciones, no sustituye su generación o evaluación.

**Ejecución distribuida.** Temporal u otra cola duradera pueden ayudar a operaciones grandes, pero no hacen que un HTTP facturable sea exactamente una vez. Sus [reintentos](https://docs.temporal.io/encyclopedia/retry-policies) requieren límites e idempotencia. Para un proyecto personal priorizaría SQLite y un despachador local fiable antes de incorporar infraestructura distribuida.

## 3. Comparación para tomar la decisión

Las capacidades siguientes son documentadas o inferidas del diseño; **ninguna alternativa tiene un benchmark natural ES-419 equivalente en este proyecto**. «Potencial» no significa calidad demostrada.

| Opción | Naturalidad, fidelidad, términos y lore | Escala y recuperación | Trabajo humano | Desarrollo y mantenimiento | Economía orientativa |
|---|---|---|---|---|---|
| Arquitectura actual literal | Controles útiles; contexto incompleto; juez natural sin calibrar | Flujo masivo offline; pilotos fragmentados; cuota de 14 | Aprobación editorial pendiente por entrada | Complejidad acumulada entre ejecutores | No hay precio integral validado |
| **Luna + Sol selectivo, flujo rediseñado** | Contexto verificable y QA cruzada; mejora por demostrar | Una cola, presupuesto global, cohortes y recibos | Solo riesgos, muestra y dudas de contexto; política de entrega pendiente | Moderado; aprovecha dominio existente | Central: US$60–67; sensibilidad contextual: US$71–79 |
| Un modelo avanzado para toda generación | Potencial mayor fidelidad; sin contraste ES-419/lore | También requiere la misma persistencia | Podría reducirla; desconocido | Menos roles, más gasto por generación | Sol: ~US$377 solo generación a igual carga de tokens |
| Agentes de tres pasadas para todo | Pueden corregir o introducir cambios; necesitan fuentes | Más etapas, dependencias y fallos parciales | No desaparece por consenso | Alto para mantener y calibrar | Ejemplos ilustrativos: ~US$175 con tres pasadas Luna; ~US$1.730 con reflexión Sol por entrada, sin juez final adicional |
| Google NMT / Translation LLM | Terminología y adaptación posibles; lore/ES-419 sin medir aquí | Servicio masivo; recuperación debe integrarse | Posedición desconocida | Nueva integración y QA de formatos | ~US$411 / US$448 antes de QA |
| Gemini Flash-Lite | Candidato; ninguna superioridad lingüística local probada | Otro proveedor y telemetría que mantener | Desconocido | Adaptador adicional | 2.5: ~US$17; 3.1: ~US$51, solo generación a igual tokenización hipotética |
| Motor local o híbrido | Posible auxilio; calidad principal sin probar | Sin cuota remota; recuperación local necesaria | Puede aumentar si empeora calidad | Runtime, evaluación y actualizaciones adicionales | Electricidad, tiempo y preparación; no es coste cero |
| CAT/TMS comercial o abierto | Buen soporte de TM, glosarios y QA; el lore debe aportarse | Madurez operativa variable | Facilita revisión, no elimina su necesidad | Licencias o alojamiento, sincronización | Sin cotización comparable; motor adicional |

RAG, TM y reglas no aparecen como motores rivales: los incorporaría a las opciones que los necesiten. En contenido comunitario, la disponibilidad y versión de documentación del autor condicionan a todos; ningún proveedor garantiza conocer cada mod instalado.

**Velocidad:** no hay medición actual de latencia/rendimiento sostenido comparable. Con las hipótesis centrales pasaríamos de 74.794 a 30.086 solicitudes de generación, una reducción del 59,77 %; eso no implica idéntica reducción de tiempo. El cálculo temporal es `llamadas × latencia / concurrencia efectiva + esperas + revisión`; hoy faltan esos parámetros y la concurrencia real segura. No asigno días de finalización sin medirlos.

## 4. Proyección económica de 240.684 cadenas

### Base y método

Tarifas Standard verificadas: por millón de tokens, Luna **US$0,10 entrada / US$0,50 salida** y Sol **US$2 / US$10**. Fuentes: [GPT-6 Luna](https://developers.openai.com/api/docs/models/gpt-6-luna) y [GPT-6.1 Sol](https://developers.openai.com/api/docs/models/gpt-6.1-sol).

Uso el volumen de caracteres del inventario, **no el coste del piloto de 1.000**:

- `N = 240.684`; `D = N × 20.077.501 / 235.301 ≈ 20,54 millones de caracteres fuente`.
- Supuestos de tokenización: inglés 4 caracteres/token; español 3,5; expansión española 1,18; JSON de salida 12 tokens por unidad. No son conteos BPE medidos.
- Para `K = ceil(N/b)`: entrada Luna `I = D/4 + N×(M+C) + K×F`; salida facturable `O = D×1,18/3,5 + 12N + K×R`.
- `b`: unidades por solicitud; `F`: instrucciones/esquema por solicitud; `M`: metadatos por unidad; `C`: contexto por unidad; `R`: razonamiento facturable por solicitud. La salida incluye texto y razonamiento.
- Sol revisa `ceil(N×r)` unidades. `ceil(N×e)` reciben una posedición Luna y una recomprobación Sol. La primera pasada supone **0 % de aciertos TM**, conforme al estado medido.

| Hipótesis | Eficiente | Agrupación actual, central | Empaquetado propuesto, central | Estrés |
|---|---:|---:|---:|---:|
| `b / F / M / C / R` | 12 / 800 / 50 / 80 / 64 | 3,218 / 1.100 / 60 / 200 / 256 | 8 / 1.100 / 60 / 200 / 256 | 1 / 1.400 / 80 / 500 / 1.024 |
| Revisión Sol inicial / corrección | 1 % / 0,2 % | 2 % / 0,5 % | 2 % / 0,5 % | 2 % / 1 % |
| Tokens Sol por revisión, entrada / salida total | 1.000 / 200 | 1.400 / 400 | 1.400 / 400 | 2.500 / 900 |
| Tokens Luna por corrección, entrada / salida total | 1.200 / 200 | 1.600 / 300 | 1.600 / 300 | 3.000 / 1.000 |

El 2 % sostenido presupone **futuras cohortes**, no el límite actual de 14 en un único plan. La media de ocho unidades es un objetivo experimental, no una optimización implementada.

| Resultado estimado, USD | Eficiente | Agrupación actual, central | Empaquetado propuesto, central | Estrés |
|---|---:|---:|---:|---:|
| Solicitudes Luna de generación | 20.057 | 74.794 | 30.086 | 240.684 |
| Entrada / salida Luna, millones de tokens | 52,47 / 11,10 | 149,99 / 28,96 | 100,81 / 17,51 | 481,69 / 256,27 |
| Generación Luna | 10,79 | 29,48 | 18,84 | 176,31 |
| Evaluación inicial Sol | 9,63 | 32,74 | 32,74 | 67,40 |
| Posedición Luna | 0,11 | 0,37 | 0,37 | 1,93 |
| Recomprobación Sol | 1,93 | 8,19 | 8,19 | 33,70 |
| **Total sin aciertos ni escrituras de caché** | **22,46** | **70,77** | **60,13** | **279,32** |
| **Sensibilidad: toda entrada a tarifa de escritura de caché 1,25×** | **25,23** | **78,78** | **66,91** | **300,57** |

La última fila es un extremo de facturación de entrada, no el comportamiento típico esperado. Las diferencias de céntimos se deben al redondeo. Los importes excluyen impuestos, hosting, trabajo humano, preparación, ejecuciones inciertas adicionales y cambios de alcance; no son un presupuesto autorizado ni un límite garantizado.

**Lectura económica:** optimizar el empaquetado ahorraría un 36,10 % de la generación, pero solo un **15,03 % del total** central. Sol sería aproximadamente el **68 % del gasto** pese al 2 % inicial de revisión. Sus recomprobaciones suman otras 1.204 llamadas en ese escenario. Revisión inicial al 0,5 / 1 / 2 % costaría aproximadamente US$8,19 / 16,37 / 32,74; elegir una tasa exige medir defectos y falsas aprobaciones, no solo ahorrar.

**Sensibilidades que importan:**

- Conservar las distinciones contextuales observadas, extrapolando sus ratios de unidades y caracteres, elevaría el central propuesto a **US$70,79–78,81**. Algunas identidades de parches podrían consolidarse: no son 285.315 trabajos confirmados. No aplico otro descuento por duplicados sobre una población ya deduplicada por texto.
- La [caché](https://developers.openai.com/api/docs/guides/prompt-caching) requiere prefijos idénticos y al menos 1.024 tokens de entrada. No se debe presuponer su acierto ni alargar prompts para obtenerla. Con 25 % o 50 % de entrada Luna leída de caché y sin nuevas escrituras, el central total bajaría aproximadamente a US$57,87 o US$55,60; no un 90 % del proyecto.
- Un futuro 10 % de coincidencias TM aprobadas y contextualmente válidas llevaría ese mismo escenario aproximadamente a US$54,12. **Ese ahorro no existe en la primera pasada medida.**
- [Batch API](https://developers.openai.com/api/docs/guides/batch) ofrece un descuento del 50 %, hasta 50.000 solicitudes y 200 MB por archivo, con ventana de 24 horas. Aplicado solo a generación Luna, el central sería ~US$50,71 total. El coordinador actual es offline; hace falta implementar y verificar su ciclo real, resultados parciales y conciliación. No acumulo descuentos de Batch y caché. No lo pondría antes que resolver la fiabilidad por ahorrar unos US$9.
- A igual carga central de tokens, Sol generando todo costaría ~US$376,75 antes de un segundo juez. [Gemini](https://ai.google.dev/gemini-api/docs/pricing) 2.5 Flash-Lite daría ~US$17,09 y 3.1 Flash-Lite ~US$51,47 solo de generación, frente a US$18,84 de Luna. Son comparaciones de tarifa: tokenizadores, razonamiento y calidad reales pueden cambiar el resultado. El ahorro tarifario de ~US$1,75 del primero no justifica por sí solo migrar.
- Con los [precios de Google Cloud Translation](https://cloud.google.com/products/translate/pricing), NMT daría ~US$410,74 y Translation LLM ~US$447,70, antes de QA; no presupongo créditos gratuitos.
- Un motor local que emitiera diez millones de tokens tardaría 92,6 horas a 30 tokens/s o 27,8 horas a 100 tokens/s: son ejemplos, no benchmarks de esta GPU. Electricidad = kW × horas × tarifa local. La generación Luna central cuesta menos de US$19: trasladarla a GPU no elimina el gasto del juez ni garantiza ahorrar tiempo.

Detalle y supuestos verificables: [ECONOMIA_SUPUESTOS.json](ECONOMIA_SUPUESTOS.json).

## 5. Arquitectura recomendada

**Una aplicación local, un camino de ejecución real y dos modelos con responsabilidades acotadas.** No añadiría una plataforma distribuida ni un equipo de agentes para este volumen.

1. **Manifiesto versionado:** mod/plugin/registro/campo, identidad canónica, versión de fuente y destino de exportación. Reconciliar parches sin deduplicar sentidos diferentes por texto idéntico.
2. **Dos memorias separadas:** TM maestra aprobada y caché de candidatas con procedencia, versión de prompt, contexto, modelo y resultados QA. Reutilizar una candidata evita facturarla otra vez; no la convierte en referencia editorial. Coincidencias semánticas solo orientan.
3. **Contexto por necesidad:** ficha breve de mod y entidades, registros vinculados, documentación del autor con versión/licencia y vecinos de escena o párrafo. Priorizar mods con ambigüedad, diálogo y narrativa; evitar investigar indiscriminadamente todos los mods o navegar por cada cadena.
4. **Terminología por identidad y ámbito:** mismo lugar/personaje entre diálogo, objetivo, mapa y HUD; misma raíz denominativa dentro de una familia de objetos, con variantes gramaticales. Un conflicto con el glosario se retiene: no se impone una palabra a todos sus sentidos.
5. **Segmentación y empaquetado por tokens/bytes:** preservar escenas, párrafos, marcadores y continuidad. Agrupar varias unidades etiquetadas del mismo ámbito en un HTTP sin presentarlas como una conversación continua. Retener lo que no pueda segmentarse/reensamblarse con seguridad; no truncar.
6. **Luna genera; QA local revisa el 100 %:** identidades, controles, variables, números/negación cuando sea comprobable, campos y coherencia entre registros. Prompts ES-419 por tipo de texto: intención, registro, voz y naturalidad, con ejemplos de decisiones; no convertir neutralidad en literalidad o cambios de intensidad sin contexto.
7. **Sol evalúa solo riesgo y muestra acotada:** reservar una parte para muestra estratificada que los casos de riesgo no puedan agotar. Pedir errores con evidencia y fragmento, severidad y abstención; `sin incidencias` significa candidata revisada por IA. Una posedición Luna y una recomprobación Sol como máximo para errores sustantivos; las dudas de lore sin fuente se retienen. No corregir por mera preferencia estilística ni encadenar consenso.
8. **Entrega diferenciada:** candidatas, traducciones con referencia humana y material elegible para ensayo personal conservan estados distintos. Habilitar en el futuro un paquete provisional para perfil clonado, con QA y muestra aceptables, requeriría una política explícita de proyecto. **Hoy no está habilitado ni autorizado por este informe.** Si se exige revisión humana individual de todo, el objetivo de autonomía cambia y debe reconocerse.
9. **Validación de entrega:** manifiesto de archivos y versiones, comprobación de formato, carga/orden y smoke test en entorno aislado; solo después considerar el perfil de juego. DSD o exportación nativa según formato, no una promesa de compatibilidad universal con MO2.

La ejecución usaría un libro presupuestario global y cohortes acotadas —por ejemplo, 500–2.000 unidades, a dimensionar offline—. Reserva antes del envío, intención durable, recibo antes del parseo y aplicación idempotente. Un resultado con consumo conocido y contenido inválido se aísla. Un envío incierto conserva íntegra su reserva y **no se reenvía**; otros grupos solo podrían continuar si siguen cubiertos por presupuesto y por un límite global de incertidumbre. Credenciales, corrupción, tarifas o autorización inseguras detienen todo.

Para migrar posteriormente: adaptar el ejecutor actual a ese único contrato, conservar lectores de formatos y diarios antiguos, reutilizar recibos/candidatas compatibles por hashes y hacer escrituras nuevas en staging versionado. Mantener el ejecutor anterior disponible para reversión y sus evidencias en lectura. No reabrir el diario detenido ni conciliar su reserva por suposición. Esta es una propuesta, no una migración iniciada.

## 6. Riesgos y límites de la recomendación

- **Falsas aprobaciones correlacionadas:** Luna y Sol pueden compartir errores; hacen falta referencias y evaluación humana ciega para calibrar, no usar a Sol como verdad absoluta.
- **Contexto equivocado:** recuperar una página de otra versión o una entidad homónima puede empeorar una traducción. Medir pertinencia y procedencia, no solo número de referencias recuperadas.
- **Libros, voces y cronología:** incluso QA local perfecto no prueba coherencia narrativa. Revisar muestras contiguas y textos completos representativos, no únicamente frases sueltas.
- **Cola larga y formatos:** segmentos adicionales, PEX/JSON y restricciones del juego pueden elevar llamadas y exigir retenciones. No hay cifra final de unidades hasta reconciliar el manifiesto.
- **Autonomía con riesgo residual:** muestrear reduce trabajo, pero deja errores no observados. Una revisión humana del 1 % a 30 segundos serían unas 20 horas; investigar lore puede multiplicarlas. No hay promesa de traducción sin revisión o sin errores.
- **Provider y telemetría:** el perfil corregido no está confirmado mediante API. Una respuesta incompleta con uso cero no prueba ausencia de cargo. Precios, cuota y calidad deben validarse antes de aprobar el coste total.

## 7. Dictamen y cambios de rumbo concretos

**B. EVOLUCIONAR.** A sería prematuro: no basta con pulir prompts ante la separación entre flujo offline y real, la fragmentación económica y el bloqueo editorial por entrada. C tampoco está sustentado: ninguna alternativa ha demostrado mejor calidad/autonomía/coste en un contraste equivalente de Nolvus, y cambiar el motor no resuelve sus formatos o contexto.

| Decisión | Qué haría |
|---|---|
| Conservar | Adaptadores de dominio, protección, contratos, glosario/TM con ámbito, conocimiento con procedencia, staging, visor, recibos e historial. |
| Eliminar del camino operativo futuro | Runners de producción duplicados y rondas editoriales por estilo; mantener el código/evidencia histórica disponible para reversión. |
| Reemplazar | Presupuestos aislados, indicador global como base de concurrencia, plan monolítico, cuota absoluta de 14 y agrupación solo por cantidad/caracteres. |
| Dejar de desarrollar por ahora | Agentes que revisan todo, búsquedas web por cadena, fine-tuning, una enciclopedia completa de todos los mods, infraestructura distribuida y sustitución total por GPU sin benchmark. |
| Investigar antes de construir de nuevo | Intercambios y referencias NPC/diálogo de xTranslator/xEdit; herramientas profesionales solo para carencias concretas. |

Reabriría **C** si otro motor demuestra calidad y menor esfuerzo total, si Luna no supera un perfil real estable, o si la calidad exige tantas revisiones/correcciones que pierde su ventaja. Una generación más cara puede ser económicamente superior si ahorra suficiente trabajo personal. Hoy no se dispone de esa evidencia.

## 8. Hoja de ruta propuesta

| Paso | Entregable futuro | Condición de salida |
|---|---|---|
| 1. Resolver la incertidumbre básica | Adjudicación ciega de las 24 parejas naturales conservadas; conciliación del recibo incierto cuando exista evidencia | No confundir gusto, importación y fidelidad; conservar la reserva mientras no se concilie. |
| 2. Medir el cuello de botella offline | Manifiesto reconciliado, longitudes, agrupación tokenizada, contexto de hablante y comparación con salidas de herramientas existentes | Unidades y costes auditables; ninguna pérdida o mezcla narrativa. |
| 3. Consolidar ejecución | Un despachador, presupuesto compartido, cohortes e importación idempotente | Fallos inyectados y reanudación sin reenvíos; compatibilidad histórica y reversión. |
| 4. Validar perfil real y comparación mínima | Microdiagnóstico y experimento pareado, con autorización futura | Uso conciliable, calidad ciega y ahorro observados; no escalar una incidencia sin resolver. |
| 5. Calibrar QA y entrega | Muestra natural independiente por mod/tipo/longitud; política de paquete provisional aislado | Medir falsas aprobaciones, omisiones, abstenciones y minutos humanos; criterios de aceptación explícitos. |
| 6. Escalar por etapas | Cohortes crecientes y, solo si compensa, Batch | Coste por millón de caracteres, calidad, recuperación y carga humana estables; ensayo aislado de exportación antes del juego. |

No asigno fechas sin medir el paso real del proveedor, el contexto pendiente y la revisión. La escala completa no debe convertirse en el siguiente piloto.

## 9. Experimento mínimo para poner a prueba esta recomendación

**Primero, aprovechar lo que ya existe:** adjudicar a ciegas las 24 parejas naturales conservadas, ocultando qué contexto produjo cada candidata. No requiere nuevas traducciones ni gasto. Sirve para detectar si más contexto ayuda o introduce errores; no demuestra fiabilidad masiva.

**Prerrequisito pagado futuro:** el microdiagnóstico ya preparado, dos Luna y un Sol, con cap propuesto US$0,05. Verifica contrato, respuesta y telemetría del perfil corregido; no reabre el diario detenido ni mide superioridad lingüística.

**Comparación nueva propuesta, sin preparar solicitudes ni ejecutarla ahora:**

- **70 unidades naturales tras segmentación**, de varios mods, con pares de escenas, nombres de lugares, familias de objetos, UI y pasajes de libros. Separar muestra representativa de casos difíciles seleccionados; fijar fuentes/versiones y un conjunto de validación aparte.
- Brazo A: agrupación/contexto actuales compatibles con el contrato corregido. Brazo B: mismo modelo, esfuerzo, fuente y reglas editoriales, con contexto pertinente y empaquetado etiquetado propuestos. Congelar términos para no confundir cambios de glosario con arquitectura.
- Máximo **50 solicitudes Luna y dos Sol**, sin posedición. Las dos Sol evalúan una pareja de casos —una salida por brazo—; son un control exploratorio, **no calibración del juez**. La comparación de calidad principal es humana ciega sobre las 70 parejas, apoyada en referencias verificables cuando hagan falta. Dudas de lore no resueltas quedan como indeterminadas.
- Cap futuro propuesto **US$0,20**, sublímites Luna US$0,11 y Sol US$0,09; petición de hasta 6.144 bytes y salida máxima 2.048 tokens. Con el criterio conservador de planificación vigente —bytes + 512, entrada a 1,25× y margen 15 %— la reserva de 50 Luna y dos Sol sería **US$0,192096**. La cifra depende del perfil y tarifas congeladas, no garantiza que todos los textos quepan.
- Antes de autorizar, generar previews y validar que esos límites cubren realmente ambos brazos. Si no caben, diferir/reducir el experimento y declarar la pérdida de cobertura: no aumentar automáticamente salida, llamadas o presupuesto. La media de ocho elementos debe medirse, no forzarse.
- El máximo combinado de la reserva/cargo anterior US$0,02646393, el microdiagnóstico US$0,05 y esta propuesta US$0,20 sería **US$0,27646393**. Es una comprobación conservadora de compatibilidad con el antecedente US$0,30, **no una nueva autorización ni una liberación de reserva**.

**Métricas y criterio de avance:** coste y tokens por carácter fuente; llamadas por unidad; resultado por tipo/longitud; errores críticos/mayores, naturalidad y coherencia entre registros; abstenciones, casos que requieren intervención y minutos de revisión; reanudación con cero HTTP adicionales. B debería reducir al menos el 25 % del coste de generación frente a A, sin regresiones críticas observadas ni deterioro claro de fidelidad/naturalidad en la comparación ciega. Si no lo logra, revisar empaquetado/contexto antes de ampliarlo; no declarar éxito por JSON válido.

Este experimento puede refutar la propuesta y detectar una ventaja inicial; no certifica falsas aprobaciones bajas. Incluso cero errores en 70 observaciones aleatorias independientes dejarían un límite superior aproximado del 4,3 % al 95 %; escenas correlacionadas dan menos evidencia. Calibrar la revisión selectiva y decidir la entrega requiere una muestra natural mayor e independiente. Si la diferencia es incierta, el resultado correcto es inconcluso.

**Resultado de esta evaluación:** fundamentos aprovechables, ejecución y política editorial por rediseñar, gasto API probablemente secundario frente al esfuerzo humano. No se ha realizado ninguna de las operaciones futuras descritas.
