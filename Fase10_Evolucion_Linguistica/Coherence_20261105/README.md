# Coherencia contextual entre registros — 2026-11-05

**Implementado y verificado offline.** Se incorporó contraste de nombres entre registros y referencias de familias de equipo al flujo de traducción, reevaluación y recomprobación. API Lock activo, cero HTTP a proveedores y cero créditos. No se alteraron traducciones, decisiones humanas, glosario, TM, producción ni Nolvus. El código permanece privado en `codex/contextual-coherence-20261105`, commit `d9f2aae646501f8c22e1fd7f4c2f165f0b1a0032`.

Esta fase continúa la consolidación técnica anterior. No es otra auditoría general ni una reorganización documental.

## Problema y cambio funcional

Una traducción fluida puede cambiar una identidad o divergir del nombre mostrado en el juego. El nuevo registro contextual reúne nombres de campos `FULL`, su tipo, identidad, plugin de origen y traducción existente. Se lee el último escaneo mediante SQLite en modo de solo lectura; no se usa una candidata recién generada como si fuera una denominación canónica.

- **Identidad compartida:** las modificaciones de un mismo registro pueden relacionarse por su plugin de origen. Los espacios del juego base se comparten expresamente; otros mods no se cruzan solo por tener un nombre parecido. Las dependencias adicionales requieren declararse, no se adivinan.
- **Denominación:** diálogos, objetivos y HUD pueden consultar el mismo par de nombre original/localizado. La localización existente del juego base tiene prioridad de referencia frente a variantes léxicas de overrides. El glosario aprobado conserva su autoridad; una referencia no lo sobrescribe.
- **Ambigüedad:** distintas traducciones de referencia producen un conflicto explícito. Un nombre sin traducción queda como contexto incompleto, sin inventar una equivalencia. La identidad exacta del registro permite separar títulos homónimos.
- **Conjuntos:** se recuperan posibles piezas relacionadas y pares de nombres/traducciones; las preferencias de familia admiten formas de género/número. Las propuestas permanecen propuestas y no se convierten en glosario. Un término compartido por sí solo no prueba que dos prendas pertenezcan al mismo conjunto. No se impuso «sombrío» globalmente a partir de notas todavía marcadas para investigación.
- **Integración:** las referencias llegan a la preparación del proveedor y del coordinador, al revisor independiente y a la posedición/recomprobación. Una propuesta que conserva o introduce una discrepancia de identidad se retiene. Nueva evidencia contradictoria impide reutilizar ciegamente una aprobación anterior, sin borrar aquella decisión.
- **Visor:** las copias futuras pueden mostrar nombres de referencia, propuestas de conjunto y procedencia abreviada. Una alerta nueva de identidad sigue visible aunque exista una aprobación humana. La carpeta que el usuario estaba revisando no se regeneró ni se sustituyó; las funciones nuevas se cargan al reabrir la app.

## Validación y resultados

La base preservada pasó **212/212** pruebas. La versión final pasó **246/246**, con **34 regresiones nuevas**: lugares en diálogo/misión/HUD, identidades distintas de los ejemplos iniciales, namespaces ajenos, homónimos, conflictos, ausencia de traducción, concordancia de familias, propuestas no vinculantes, atributos como `Masked`, nombres dentro de variables, preparación real del adaptador sin transporte y recuperación sin reenvíos. Se probaron metáforas naturales y palabras comunes para evitar alertas injustificadas.

El primer comando de descubrimiento de la suite omitió su raíz de paquete y falló al importar módulos; se corrigió con `-t .`. La base se verificó desde el snapshot anterior, no desde el código ya modificado. Los tiempos de suite no se interpretan como una aceleración del producto.

| Medición de la copia real | Resultado |
|---|---:|
| Cadenas históricas examinadas | 1.000 |
| Filas `FULL` leídas para construir el registro | 146.120 |
| Pares distintos de namespace/nombre indexados | 60.815 |
| Construcción del registro, una observación local | 1,566 s |
| Recuperación/contraste de las 1.000 cadenas | 3,491 s |
| Casos con alguna referencia de nombre/equipo | 483 |
| Casos con referencia de nombre traducido | 82 |
| Casos con nombre sin traducción de referencia | 431 |
| Casos con posibles piezas relacionadas | 112 |
| Casos señalados por diferencia/conflicto | 32 |

Las coberturas se solapan y no se suman. Recuperar una referencia no demuestra su pertinencia. **Los 32 señalados no son 32 errores confirmados:** hay 26 hallazgos de diferencia y siete de conflicto, con un caso compartido. El resultado tampoco es una tasa de error, de falsos positivos ni de ahorro humano.

La primera implementación experimental señaló 187 casos. Se corrigieron alertas por palabras comunes como `safe`/`dragon`, clases de NPC, nombres incluidos en variables, referencias compartidas vacías, homónimos y variación exclusiva de mayúsculas. Los ensayos intermedios señalaron 101 y 33 casos; el final, 32. Esa disminución documenta la depuración del detector, no una calibración semántica independiente. Se conservan las evidencias intermedias privadas.

El contraste genérico recupera **The Steed → El Corcel**, frente a la candidata «La Sombra», y diferencias de denominación como **Redbelly Mine** y **Honorhall Orphanage**. No depende de una lista de errores escrita específicamente para esas frases. **Rimerock Burrow** sigue sin traducción de referencia suficiente: se conserva la incertidumbre. La regla previa que detecta la pérdida de `Masked` continúa cubierta junto al control de familia.

El flujo completo se ejercitó con el catálogo contextual anterior y el nuevo registro: **21 invocaciones de simulador**, 11 textos segmentados sin truncar, cero invocaciones adicionales al reanudar, cero propuestas cambiadas y cero aprobaciones nuevas. Los dictámenes simulados son inconclusos; **no se ejecutó una IA real ni se midió una mejora de su traducción**. Los límites de alcance dejaron 21 casos diferidos; los 25 casos históricos siguieron en cuarentena técnica.

## Revisión humana y protección

Se congelaron como entrada privada **41 decisiones: 29 aprobaciones y 12 investigaciones**. En los 25 de cuarentena hay **22 aprobaciones lingüísticas y tres investigaciones**; eso no corrige el hash ni libera la importación del lote. Las decisiones no se incorporaron automáticamente como verdad del lore ni como nuevas entradas de glosario.

Se comprobaron **1.719 archivos anteriores idénticos por SHA-256**, dentro del alcance protegido definido para esta fase: producción/configuración, pilotos y experimentos anteriores de contexto/revisión. El número no se compara con otros inventarios de distinto alcance. Las 41 decisiones activas se conservaron byte por byte. No se editó el staging ni se desplegó nada al juego.

## Próxima prueba y límites reales

Se actualizó la preparación del experimento de **12 casos naturales y cuatro controles sintéticos**, con tres variantes de contexto e integración opcional del registro de nombres. Hay **40 solicitudes únicas preparadas**, más hasta cuatro posediciones y cuatro recomprobaciones: **48 como máximo**. No se ejecutó ninguna de ellas.

La reserva máxima propuesta sigue en **US$0,12718080**, bajo un tope de **US$0,15**, según la tarifa local registrada y un margen del 15 %. No es una cotización vigente ni un cargo. Los previews y el corpus son privados. Faltan etiquetas adjudicadas con evidencia, validación del adaptador real, verificación de tarifa y autorización específica antes de cualquier API facturable. La aprobación automática sigue deshabilitada.

Falta resolver las denominaciones sin referencia, ampliar fuentes/versiones fiables de los mods y medir al juez real. Las familias inferidas léxicamente son hipótesis; no sustituyen relaciones de equipo confirmadas por datos del mod. La selección admite cuatro fragmentos por solicitud: el contexto es limitado, no conocimiento exhaustivo. Sigue pendiente medir pertinencia, falsas aprobaciones, falsas alarmas y tiempo humano con una muestra independiente. Las 34 regresiones verifican mecanismos y ejemplos definidos, no certifican las 1.000 traducciones.

La evidencia detallada, el inventario con hashes y los snapshots permanecen en `private/`. GitHub recibe este informe y métricas/plan/protección agregados, junto con la actualización compacta de los cuatro documentos activos, en la PR documental existente. No se publican código fuente, bases, candidatas ni decisiones individuales.
