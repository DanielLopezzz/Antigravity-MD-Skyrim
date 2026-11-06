# Contexto documental de mods y adjudicación — 2026-11-05

Se implementó y ejecutó una primera ampliación documental sobre el inventario completo de candidatos con texto sin traducción registrada. **API Lock activo, cero llamadas a proveedores de IA y cero créditos.** Código privado: `4b7b4e9`, rama `codex/contextual-coherence-20261105`. No se alteraron producción, Nolvus, glosario, TM ni revisiones humanas.

## Descubrimiento e integración

El descubridor lee el último escaneo y consulta únicamente `meta.ini` y documentos de texto de las carpetas de mods, sin escribir en ellas. Examina archivos README, changelog, guía, manual, documentación y créditos en la raíz y directorios documentales inmediatos. Excluye código, configuraciones arbitrarias, enlaces/junctions, archivos grandes y formatos no admitidos. No abre archivos comprimidos/BSA ni PDFs; la ausencia de un resultado no demuestra que un mod carezca de documentación.

- **1.302 mods** habilitados con registros sin traducción: universo de descubrimiento, no autorización para traducir todos sus registros.
- **1.220 páginas candidatas de Nexus** derivadas de metadatos; no son 1.220 fuentes verificadas. Un parche o ESP limpiado puede apuntar a una página distinta del mod narrativo original.
- **41 documentos locales de 36 mods** incorporados a una copia aislada del catálogo previo, con hash y procedencia. Incluyen candidatos fuera de la muestra de 1.000 cadenas, como Falskaar, Clockwork, Teldryn Serious y Moonpath to Elsweyr.
- **10 fuentes externas examinadas de siete mods**, registradas mediante resúmenes analíticos propios. No se copiaron íntegramente páginas web ni se atribuyeron esos resúmenes a sus autores.

Las fuentes externas de esta primera selección son:

| Mod | Fuentes y utilidad |
|---|---|
| Lucien | [Perfil y guía de Joseph Russell](https://josephrussellauthor.com/Lucien.php), [comentarios de misiones](https://josephrussellauthor.com/Lucien_Quests.php): evolución de voz y referencias a otras aventuras |
| Interesting NPCs | [Índice del proyecto](https://3dnpc.com/wiki/interesting-npcs/), [Zora](https://3dnpc.com/wiki/interesting-npcs/locations/whiterun/zora/), [The Children Fair](https://3dnpc.com/wiki/interesting-npcs/quests/full-quests/the-children-fair/): relaciones entre personaje, lugares y misión |
| Remiel | [Página de Maplespice](https://www.nexusmods.com/skyrimspecialedition/mods/51874?file_id=710389): ingeniera bretona y contexto dwemer |
| Katana And Megara | [Página del mod](https://www.nexusmods.com/skyrimspecialedition/mods/69622?file_id=559240): compañeros, diálogo reactivo y ausencia de presunción de que el jugador sea Sangre de Dragón |
| Mysticism | [Página de Simon Magus](https://www.nexusmods.com/skyrimspecialedition/mods/27839): modificaciones de hechizos y advertencia de versión |
| Beyond Skyrim - Bruma | [Sitio del equipo](https://beyondskyrim.org/project/cyrodiil): distinguir Bruma publicada de contenidos futuros de Cyrodiil |
| Caught Red Handed | [Página de JaySerpa](https://www.nexusmods.com/skyrimspecialedition/mods/65708?tab=description): contexto y alternativas de la misión de Svana y Haelga |

Se detectaron tres diferencias entre versión web y registrada: Remiel 1.7.6/1.7.5, Mysticism 2.5.0/2.4.2 y Caught Red Handed 1.07/1.04. Katana comparte etiqueta 3.0.5, sin certificar identidad del contenido. Las otras seis fuentes no tienen versión verificable. Estas advertencias se incluyen en el contexto recuperable; **ninguna cifra o trama instalada se reemplaza con información de otra versión**. La página de Angelic Standing Stones devolvió 403 y quedó sin importar; no se intentó eludirlo.

## Corrección funcional de recuperación

Los registros de juego ocupaban todos los primeros resultados y desplazaban a los documentos relevantes. Ahora se consulta por separado documentación activa con coincidencia léxica, dentro del mismo mod/plugin, y se reserva un lugar antes de completar el límite de cuatro fragmentos. No se incorpora un documento irrelevante solo porque exista.

En siete registros reales seleccionados, los documentos aparecían en **0/7** contextos antes y **6/7** después. Esto verifica entrega de referencias, no pertinencia semántica universal ni calidad de traducción. Mysticism no obtuvo documentación en esa muestra: la recuperación léxica bilingüe sigue siendo una limitación. El catálogo anterior permanece idéntico; la ampliación está en `private/knowledge.sqlite3`.

## Los 32 casos señalados

Se revisó el contraste de nombres de los 32 casos, sin certificar todo su contenido:

| Resultado del análisis | Casos |
|---|---:|
| Diferencias observables de denominación frente a referencias locales | 16 |
| Cambio de identidad respaldado por original y referencia | 1 |
| Alerta que no establece un error de traducción | 11 |
| Entidad, mecánica o alcance todavía insuficientemente resueltos | 4 |

Los 11 incluyen variaciones gramaticales, un verbo confundido con nombre de NPC y conflictos de fuentes aunque la candidata ya coincide con la denominación disponible. No se deben tratar automáticamente como traducciones defectuosas. Las 16 diferencias de denominación requieren unificación sustentada, no son necesariamente errores de significado.

El cambio de identidad es **The Steed → La Sombra**. Se guardó una propuesta aislada que restituye **El Corcel**, conservando el resto de la edición humana. No se sustituyó la decisión ni la candidata activa. Los cuatro pendientes incluyen alcance de Ashland, interpretación de Fortify Destruction, referente de Shrine of Talos y uso técnico de Afflicted Refugee. La documentación general de un mod no resuelve por sí sola esos referentes.

## Verificación, publicación y pendientes

**253/253 pruebas**, siete nuevas sobre 246: descubrimiento acotado, filtrado de datos privados de metadata, IDs inválidos, límites, metadatos malformados y conservación de documentación relevante sin mezclar otro mod. Se verificaron los hashes de la base productiva, los archivos JSON de la revisión activa y el catálogo anterior. Sin aprobaciones nuevas ni entrenamiento a partir de decisiones humanas.

El inventario con textos locales, resúmenes trazables, adjudicaciones por caso, propuestas y muestras de recuperación permanece privado. GitHub recibe únicamente informe, métricas agregadas y los cuatro documentos activos actualizados.

Queda verificar fuentes externas adicionales por prioridad, revisar formatos/archivos no cubiertos, ampliar relaciones documentales entre parches y mods originales y resolver los cuatro casos con evidencia insuficiente. Las páginas candidatas no se presentan como documentación ya leída. También quedan recuperación semántica bilingüe y evaluación real del juez; esta fase no autoriza gasto ni cambia la preparación facturable existente.
