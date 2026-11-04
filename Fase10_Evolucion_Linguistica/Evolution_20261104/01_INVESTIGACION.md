# Investigación lingüística y documental

Consulta: 2026-11-04. Se consultaron páginas concretas mediante búsqueda y lectura documental. No se descargaron corpus externos, libros completos, paquetes de traducción comunitaria ni bases de datos paralelas. El registro reproducible de entidad responsable, fecha, uso, concepto, conclusión, confianza y estado está en [source_register.json](source_register.json).

## Fuentes y condiciones

| Fuente | Tipo | Estado de consulta | Política aplicada | Enlace |
|---|---|---|---|---|
| Bethesda Legendary Edition PC ES manual | official | reviewed | All rights reserved; read-only consultation, paraphrases and brief labels only; no book corpus ingestion | [Fuente](https://assets.ctfassets.net/rporu91m20dc/OCrYkTqUakis2I6mksGCe/c886018f2c853e1e9e2ca1def5ca7435/manual_skyrim-le_pc_es.pdf) |
| Bethesda SE PS4 ES-MX mini manual | official | reviewed | All rights reserved; consultation only, no redistribution | [Fuente](https://assets.ctfassets.net/rporu91m20dc/4MYytwHTmEY8mwyGkICK6q/50fa9a639d2003d954c6cd4387a2a1dd/manual_skyrim-se_ps4_es-mx.pdf) |
| Bethesda Survival Mode | official | reviewed | Copyright; read-only consultation, own paraphrase | [Fuente](https://elderscrolls.bethesda.net/es-ES/news/5lz4Q7F4li6kwKmakkgWww/skyrim-survival-mode-coming-soon) |
| UESP copyright policy | community | unavailable | Unverified: 403 from the browsing tool; no alternate endpoint, API, scraping or import used | [Fuente](https://en.uesp.net/wiki/UESPWiki:Copyright_and_Ownership) |
| The Imperial Library | community | unavailable | Access/terms not verified; excluded from collection and content ingestion | [Fuente](https://www.imperial-library.info/about-us) |
| Creation Kit official wiki | technical | unavailable | Maintenance page only; no technical content ingestion | [Fuente](https://www.creationkit.com/) |
| xEdit upstream | technical | reviewed | MPL-2.0 shown by upstream; no source copied into this project | [Fuente](https://github.com/TES5Edit/TES5Edit) |
| JContainers upstream | technical | reviewed | MIT shown by upstream; no source copied | [Fuente](https://github.com/ryobg/JContainers) |
| SKSE official site | technical | reviewed | Site provides disclaimer; broad documentation reuse not established; consultation only | [Fuente](https://skse.silverlock.org/) |
| RAE-ASALE DPD plural | editorial | reviewed | Copyright; consult and paraphrase rules, no dictionary scraping | [Fuente](https://www.rae.es/dpd/plural) |
| OpenAI prompt engineering | technical | reviewed | Public documentation; consultation, no API calls | [Fuente](https://developers.openai.com/api/docs/guides/prompt-engineering) |
| OpenAI evaluation best practices | technical | reviewed | Public documentation; consultation, no API calls | [Fuente](https://developers.openai.com/api/docs/guides/evaluation-best-practices) |
| OpenAI GPT-6 Luna model documentation | technical | reviewed | Public documentation; consultation, no API calls | [Fuente](https://developers.openai.com/api/docs/models/gpt-6-luna) |
| OpenAI GPT-6.1 Sol model documentation | technical | reviewed | Public documentation; consultation, no API calls | [Fuente](https://developers.openai.com/api/docs/models/gpt-6.1-sol) |

La documentación de Bethesda permite contrastar denominaciones de recursos y mecánicas. El manual PC Legendary Edition se consultó en páginas PDF 4, 5 y 9 para atributos, armadura y discreción. No demuestra por sí solo una política ES-419 completa: la redacción peninsular del original se distingue de sus etiquetas terminológicas. El manual ES-MX localizado es breve y no proporciona un glosario exhaustivo. La evidencia oficial permite mantener Salud, Magia y Aguante como recursos, y proponer Índice de armadura y Discreción para usos concretos. No autoriza copiarlos como reglas universales de prosa.

UESP respondió 403 a la página de condiciones. No se intentaron otros endpoints, proxies ni APIs para eludir el bloqueo. Las condiciones de The Imperial Library no pudieron verificarse con la herramienta: quedó excluida de la recopilación. No se atribuyen a esas fuentes conclusiones de lore supuestamente verificadas. Para una segunda iteración se requiere consulta manual permitida o una fuente alternativa con condiciones claras, registrando artículo/edición y la conclusión propia, sin copiar el texto protegido.

Creation Kit redirige a una página de mantenimiento. xEdit y JContainers ofrecen referencias técnicas de sus propios autores con licencias identificadas en sus repositorios; no se incorporó código ajeno. SKSE se utilizó para identificar el nombre técnico, sin asumir una licencia general para redistribuir documentación. RAE-ASALE se consultó para reglas de plural; su función es gramatical, no decidir nombres del universo.

## Revisión de los 14 conceptos prioritarios

Todas las alternativas siguientes son decisiones de investigación, no nuevas aprobaciones del Glosario Maestro.

| Concepto | Análisis y criterio | Estado |
|---|---|---|
| Elder Scroll | Mantener la preferencia existente; plural explícito Pergaminos Antiguos. No sustituirla por una alternativa literal sin registro oficial. | in_review |
| Magicka | Magia está respaldada para el recurso; distinguir energía disponible, escuela y prosa general. | reviewed_reference |
| Health | Salud está respaldada como atributo; no añadir un plural mecánico a la etiqueta. | reviewed_reference |
| Stamina | Aguante está respaldado como recurso; no confundirlo con resistencias a daño. | reviewed_reference |
| Soul Gem | Mantener Gema de alma; registrar Gemas de alma como concordancia. Gema del alma no se introduce automáticamente. | in_review |
| Soul Cairn | No deducir que Recordatorio de las Almas es erróneo por no ser literal; verificar denominación Bethesda con edición/registro. | in_review |
| Khajiit | La terminación -a de Khajiita no demuestra género femenino. Separar raza, gentilicio y referencia individual antes de cambiarla. | in_review |
| Daedra | Distinguir sustantivo Daedra de adjetivo daédrico; forma plural y alcance por contexto requieren revisión. | proposed |
| Conjure | Conjurar o invocar dependen del efecto y de la convención del hechizo; no imponer un verbo universal. | proposed |
| Sneak | Discreción está documentada para habilidad/modo; en prosa puede ser moverse a escondidas o sigilosamente. | proposed |
| Tempered | Separar la acción de templar/mejorar del nombre de un grado de mejora. No deducir un rango universal. | proposed |
| Armor Rating | Índice de armadura está documentado como etiqueta; evitar clasificación de armadura o calificación literal. | proposed |
| Nightingale | Ruiseñor puede referir a identidad/equipo o al ave; comprobar personaje/facción antes de unificar. | proposed |
| Nocturnal | Conservar identidad del nombre; no convertirlo en el adjetivo nocturno ni inventar voz/género por contexto incompleto. | in_review |

## Rectificaciones y alcance de la investigación

La auditoría anterior planteó dudas sobre Soul Cairn y Khajiit. Una forma localizada puede ser canónica aunque no reproduzca literalmente la etimología inglesa. Por tanto, no se sustituye Recordatorio de las Almas por una propuesta de diccionario. Tampoco se deduce género femenino únicamente por la terminación de Khajiita. Ambas observaciones requieren cotejo de registros oficiales y de contexto; no constituyen errores confirmados.

El manual también motiva contrastar la entrada Word Wall: la denominación heredada Muro de poder debe compararse con el uso del manual relacionado con palabras, antes de proponer una corrección. La nota de Cicero contiene alternativas incompatibles con una única traducción aprobada; la de Serana usa un título de lore que no debe interpretarse literalmente como filiación. Las notas de voz y biografía no se envían como hechos verificados al proveedor.

La revisión completa conserva las 109 preferencias, examina categoría, alcance, metadatos y referencias por entrada y registra problemas concretos en [01B_GLOSARIO_109.md](01B_GLOSARIO_109.md). Hay evidencia externa verificada de una parte de la terminología, no de las 109 denominaciones. Faltan validación local por edición/FormID, fuentes de lore accesibles y aprobación editorial humana.
