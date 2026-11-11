# Histórico resumido y acceso a evidencia

Actualizado el **2026-11-11**. Es un índice de hechos cerrados y decisiones sustituidas, no una guía de ejecución. [Estado vigente](ESTADO_ACTUAL.md) · [Uso](GUIA_DE_USO.md) · [Reglas](REFERENCIA_TECNICA.md).

## Cobertura visual y línea base inicial de Moon and Star, 2026-11-11

El usuario completó ocho comprobaciones del mod original. El criterio visual acepta silencio o voz inglesa correcta del protagonista. Se comprobó la carga de DBVO 2.0.1.5, se incorporaron tres marcadores de mapa al inventario y se probó offline la exportación de 800 campos, incluidas 194 opciones, con rollback. Se construyeron dos ZIP aislados, uno para Mandyn y Selyse y otro para el mensaje del elevador. Suite 412/412, validación de los ZIP en Skyrim pendiente. [Guía de la prueba](GUIA_DE_USO.md#protocolo-en-skyrim-para-moon-and-star).

## Preparación offline de Moon and Star, 2026-11-11

Se cerró el inventario de plugins del piloto y se verificó la exportación por copia del ganador con cambios solo de texto. El ensayo de la PR #16 procesó 605 campos y revirtió dos publicaciones de prueba. Se verificaron dos ejemplos de mapeo DBVO offline. En ese corte quedaron retenidas las 194 opciones y un mensaje de script. [PR #16](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/pull/16).

## Compatibilidad y seguridad funcional, 2026-11-11

Se inspeccionó el perfil instalado y el código de DSD. El BSA de Amber Guard contiene 27 LIP asociados a WAV y dos INFO ganadores referencian fragmentos ausentes del alcance inventariado. Se corrigió el diagnóstico del 9 de noviembre en la referencia canónica. El escáner y la exportación incorporaron índices binarios, ganador completo, tablas conservadoras y puertas de perfil e integridad. Suite offline 407/407. Se seleccionó Moon and Star para la siguiente línea base funcional, pendiente de ejecutar. [Evidencia y matriz](REFERENCIA_TECNICA.md#compatibilidad-y-exportación-auditadas-el-11-de-noviembre).

## Continuación técnica, reparación focalizada y paquete DSD de Amber Guard, 2026-11-07 / 2026-11-09

Conexión inmutable del glosario maestro aprobado (`ApprovedGlossaryView`, 109 entradas), aplicación por identidad/sentido (`1.0.3`), consistencia entre registros y candidatas hermanas sin aprobar (`coherence-1.2`), adaptación contextual ante inglés defectuoso con contexto (`risk-2.4.0`), registro de 12 decisiones humanas aprobadas por Daniel (`Equipo para la aventura`, `Torre del Este` / `Llave de la Torre del Este`, fechas cardinales), ejecución autorizada de la reparación focalizada de 11 unidades (`amber_guard_1_09_2_focused_repair_v1`, conservando 121/132 candidatas; 9 solicitudes reales Luna/Sol, US$0,011996 conciliados frente al cap US$0,093406, primer ciclo natural completo `Sol → posedición Luna → recomprobación Sol` en `0006DBE1`, replay con cero HTTP) y corrección del exportador DSD tras la primera prueba en juego (`INFO NAM1` 1-based `TRDT` evitando colisión con `INFO RNAM` en `index=0`, `QUST NNAM` con índice `QOBJ`, `QUST CNAM` con `original`, más 17 `CELL FULL` de puertas interiores y 2 `NPC_ FULL` sobrescritos por parches de MO2; `151` entradas en `Amber_Guard_ES419_Provisional`, suite 404/404, candado activo y 180/180 archivos protegidos idénticos). Detalle vigente en ESTADO_ACTUAL.

## Revisión documental, 2026-11-07

Se corrigieron referencias a última ejecución, preparaciones presentadas como vigentes, límites heredados de Sol, dependencias, rutas del piloto, lectura de su libro económico y enlaces de la entrega inicial. Se actualizaron README y los cuatro documentos, conservando resultados e informes cerrados. Sin cambios funcionales ni llamadas API; no se repitió la suite de código por esta edición documental.

## Piloto real de Amber Guard, 2026-11-07

Autorizado tras la propuesta US$0,120590: 132 fuentes/133 unidades, 17 Luna y dos Sol, US$0,009556 conciliados. Reensambladas 132 candidatas; cero aprobaciones, posediciones, reenvíos o reservas pendientes nuevas. Hallazgos posteriores: omisión de T, nombres inconsistentes, ortografía/formato y ambigüedad del original; la preparación no conectó el glosario maestro. Sol no señaló errores en sus dos muestras, sin certificar el conjunto. Métricas, limitaciones y pasos siguientes en ESTADO_ACTUAL.

Se corrigió compatibilidad del visor con metadatos y prioridad Plan B; suites completas 396/396 y 397/397, carga offline de 133 casos y reutilización de recibos con cero HTTP. Observaciones separadas de candidatas y decisiones; 100 archivos protegidos idénticos. No se instaló en Nolvus/MO2 ni se modificó producción. Código privado; publicación solo documental.

## Confirmación del contrato estricto, 2026-11-07

Con autorización posterior: un Sol, US$0,001198 conciliados, 10→20 detectado como `accuracy/major`, evidencia aceptada y cero aprobaciones editoriales. Cero reservas pendientes nuevas, 88 archivos protegidos idénticos y candado restaurado. Reanudación offline con cero HTTP y hash del recibo intacto. En aquel corte no hubo cambios funcionales adicionales: suite 395/395. La calibración y el piloto completo aún estaban pendientes; el piloto se ejecutó después, según el apartado superior; detalle económico acumulado en ESTADO_ACTUAL.

## Confirmación acotada de Sol, 2026-11-07

Con autorización posterior se envió una solicitud nueva, sin reabrir diarios: US$0,000918 conciliados. Sol aceptó JSON y señaló 10→20, pero el enum traducido `fidelidad` provocó rechazo de contrato. Se conservó el recibo, no hubo reenvío ni aprobación, y se añadió esquema estricto para evaluación. Suite 395/395; el nuevo esquema se verificó solo offline. Los 79 archivos protegidos, incluidos los del diagnóstico previo, permanecieron idénticos.

## Diagnóstico y preparación versionada, 2026-11-07

Se ejecutó únicamente el alcance autorizado de dos Luna y un Sol, cap US$0,05. Dos respuestas válidas de Luna; Sol HTTP 400 sin evaluación ni uso verificable. Se conservaron recibo/reserva y se corrigió el formato offline, sin cuarto envío. No demuestra mejora lingüística. El detalle económico vigente está en ESTADO_ACTUAL.

Se completaron las cinco clasificaciones pendientes en un overlay nuevo, se incorporó evidencia de hablantes desde el plugin y se identificó la diferencia entre la versión instalada y la documentación actual. Se conservaron preparaciones anteriores y tres dudas narrativas reales. Cinco regresiones nuevas, suite completa 394/394. Código privado `40335ec`; publicación solo de los cuatro documentos canónicos.

## Cronología objetiva

**Depuración de Plan B, 2026-11-06:** 385/385, 387/387 y 389/389 pruebas completas después de cada bloque. Lectura coherente y agrupada de caché, posedición conectada al staging/visor, procedencia atómica, utilidades compartidas y muestreo equivalente más eficiente. Se conservaron lectores/ejecutores históricos y datos aprobados; cero solicitudes reales a IA. Las mediciones de planificación de la entrega inicial son su corte histórico, no un benchmark nuevo de esta depuración.

**Plan B, 2026-11-06:** implementación privada de ejecución por cohortes y libro compartido, revisión escalonada y entrega provisional aislada, posterior al dictamen estratégico B. Pruebas offline y preparación de un mod completo; sin nuevas solicitudes a modelos, promociones ni cambios en la reserva histórica. Los ejecutores anteriores se conservaron para compatibilidad. [Entrega y evidencias agregadas](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/4060d9f3581df44732228c7cb3eb46ae8646ad3b/Fase10_Evolucion_Linguistica/PlanB_20261106/README.md).

| Corte | Qué quedó registrado | Qué no debe inferirse |
|---|---|---|
| Fases 0–6 | Aplicación de escritorio, parsers, glosario/TM, QA y exportación | No certifica calidad lingüística ni toda la compatibilidad de un despliegue real |
| Fases 7–8 | Escaneo inicial y estimaciones económicas sucesivamente corregidas | Sus modelos, precios, tiempos y volúmenes no son parámetros actuales |
| Fase 9, 2026-11-03 | 3.700 mods activos, 3.717 plugins, 583 BSA y 1.168.337 elementos; Corte C de 328.485 ocurrencias y 240.244 únicas pendientes, más ~440 JSON narrativas propuestas | ~240.684 es estimación reconciliada; elementos inventariados no equivalen a cadenas |
| Fase 10.0–10.1 | Diseño Antigravity e intercambio supervisado; contrato, bóveda y Gatekeeper implementados y auditados | Las propuestas de hot-folder, autonomía y cuotas no son garantías operativas |
| Fase 10.2–10.3-A | Cambio a API-First, presupuesto local y preparación aislada de 25 cadenas | “Cero llamadas” describe aquella preparación, no el proyecto actual |
| Pilotos 10.3-B/10.3-C | Piloto de 25 con incidencias; prueba posterior de 125: 100 importadas, 23 candidatas retenidas y dos en cuarentena; coste 10.3-C US$0,011345 | No sumar esas muestras a las 1.000 actuales ni contar importación como aprobación humana |
| Evolución offline, 2026-11-04 | Contexto/contrato v1.1, TM contextual, variantes, aprobación editorial y benchmark sintético; suite entonces 80/80 | Propuestas de glosario y benchmark no acreditan mejora lingüística real |
| Cuarto piloto, 2026-11-04 | Primera etapa, R2 y continuaciones supervisadas hasta 1.000 respuestas; correcciones de porcentajes, controles, falsos rechazos y duplicados | Los intentos fallidos se conservaron; el resultado no es una sola ejecución ciega |
| Análisis, 2026-11-05 | Intersecciones de alertas, dos falsos positivos numéricos y propuesta de reevaluación | Análisis y publicación no autorizan implementar ni consumir créditos |
| Reorganización, 2026-11-05 | Un README y cuatro documentos activos; originales comprimidos con hashes; enlaces corregidos | No se alteraron app, glosario, TM, decisiones humanas, pilotos ni producción; la publicación documental es una acción separada |
| Evolución inteligente, 2026-11-05 | Validación por riesgo, conocimiento por mod, contrato 1.2, simulación de juez y coordinación durable; 38 prioridades frente a 196 en copia offline | No mide mejora semántica ni ahorro humano; no hubo aprobación nueva o API facturable. [Informe](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/a232c2edc575469e3f029ff99e6eeb4774cf4273/Fase10_Evolucion_Linguistica/Intelligence_20261105/README.md) |
| Ampliación funcional, 2026-11-05 | Catálogo completo de registros, fuentes de autores, recuperación por relevancia, revisión por segmentos, reutilización de referencias humanas y aislamiento de fallos | Implementación privada y pruebas offline; no acredita calidad de un juez real ni habilita aprobación nueva. [Estado y evidencia vigente](ESTADO_ACTUAL.md) |
| Consolidación técnica, 2026-11-05 | Herencia versionada en una línea privada, contratos/artefactos compartidos, recibos antes de conciliación, contexto incremental y validación preparada | Benchmarks de ingesta/memoria no acreditan calidad lingüística. [Informe ejecutivo](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/30db1581792ba69d2d2fd87e7222731a45d9b4f7/Fase10_Evolucion_Linguistica/Consolidation_20261105/README.md) |

**Prueba pareada real, 2026-11-05, posterior a su preparación:** 50 llamadas, ocho controles favorables y 48 candidatas; 47 válidas mecánicamente y una retenida por identidad. Parada sellada antes del juez natural; US$0,01524421 conciliados, lock activo, 276/276 pruebas offline. Corrección exacta del esquema aún sin confirmación API. No modifica pilotos ni decisiones anteriores. [Resultado parcial y correcciones](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/technical-consolidation-docs-20261105/Fase10_Evolucion_Linguistica/PairedTrial_Run_20261105/README.md).

**Terminología contextual y continuidad, 2026-11-06:** extensión del sidecar/registro/diario existentes y flujo integrado offline Luna/Sol; ámbitos semánticos, validación completa, aislamiento de lotes y bloqueos de seguridad. Suite 298/298; carga sintética y reanudación sin reenvío, cero créditos o aprobaciones nuevas. No sustituye runners históricos ni reabre el piloto detenido. [Informe y evidencia agregada](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/terminology-continuity-docs-20261106/Fase10_Evolucion_Linguistica/TerminologyContinuity_20261106/README.md).

**Barrido y preparación acotada Luna/Sol, 2026-11-06:** correcciones demostradas, suite 307/307 y replay offline sin intercambios adicionales. Plan de aquel corte: 1.050 fuentes, 100 traducciones nuevas y 14 revisiones naturales Sol, cap US$0,30; conserva borradores anteriores, pilotos y decisiones. No hubo API ni mejora lingüística real medida. [Informe y alcance económico](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/terminology-continuity-docs-20261106/Fase10_Evolucion_Linguistica/HeavyTrial_20261106/README.md).

## Informes que conviene abrir solo cuando haga falta

Tras la consolidación, **coherencia contextual del 2026-11-05**: registro de nombres con identidad y localización existente, referencias de conjuntos e integración en preparación/revisión/recomprobación/visor. Suite 246/246; barrido de las 1.000 candidatas sin reescribirlas ni aprobarlas. [Informe de continuación y límites](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/technical-consolidation-docs-20261105/Fase10_Evolucion_Linguistica/Coherence_20261105/README.md). La revisión humana cerró los 25 casos de cuarentena lingüísticamente, sin resolver su retención técnica. No se ejecutó una IA real.

Los enlaces siguientes apuntan a la documentación ya publicada; su estructura de fases se conserva para no perder enlaces externos.

- [Integración MO2, Nolvus y reglas de exportación DSD](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/amber-guard-continuation-docs-20261107/Fase10_Evolucion_Linguistica/MO2_Nolvus_Integration_20261109/README.md).
- [Análisis profesional de revisión humana](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/main/Fase10_Evolucion_Linguistica/Pilot4_Review_Analysis_20261105/README.md).
- [Resultado completo y barrido del cuarto piloto](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/main/Fase10_Evolucion_Linguistica/Pilot4_Completed_20261104/README.md).
- [Preparación](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Fase10_Evolucion_Linguistica/Pilot4_Preparation_20261104), [primera etapa](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Fase10_Evolucion_Linguistica/Pilot4_Stage1_20261104) y [R2](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Fase10_Evolucion_Linguistica/Pilot4_R2_20261104): incidencias y evidencias anteriores.
- [Evolución lingüística offline](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Fase10_Evolucion_Linguistica/Evolution_20261104).
- [Auditoría integral de código](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Fase10_1_Code_Audit/Audit_20261104_Integral_Post_Pilot) y [especificaciones/auditorías anteriores](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Fase10_1_Code_Audit).
- [Arquitectura, preparación y pilotos 10.3](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Fase10_Antigravity_Architecture).
- [Auditoría Fase 9](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/main/Fase9_Full_Audit/PHASE9_FULL_AUDIT_REPORT.md), [Fases 7–8](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Fase7_y_Fase8_Informes) y [base 0–6](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Documentacion_Base_Fases_0_a_6).

## Originales y trazabilidad

**Preparación pareada, 2026-11-05, posterior a ampliación documental:** 24 cadenas reales y ocho controles, contexto anterior/ampliado, conjunto enano agrupado y fuentes selladas; suite 262/262. Solo construcción y validación, sin traducción real ni API. En esa preparación el tope US$0,35 estaba propuesto; la ejecución autorizada posterior y su parada se registran arriba. [Informe](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/technical-consolidation-docs-20261105/Fase10_Evolucion_Linguistica/PairedTrial_20261105/README.md).

**Ampliación documental, 2026-11-05, posterior a coherencia:** descubrimiento sobre 1.302 mods, integración aislada de documentos locales y resúmenes externos versionados, adjudicación de los 32 avisos y corrección de la recuperación de fuentes relevantes. Suite 253/253, cero API/créditos. [Informe y alcance real](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/technical-consolidation-docs-20261105/Fase10_Evolucion_Linguistica/Documentation_20261105/README.md). No se presenta el conjunto de enlaces candidatos como investigación ya completada.

[Archivo de originales](archivo/originales_20261105.zip) · [Manifiesto y hashes](archivo/MANIFIESTO.json).

El ZIP del workspace conserva los 20 Markdown originales de su raíz, la hoja de estimación y las 79 fuentes Markdown/JSON previas del repositorio documental. El ZIP del espejo público contiene solo esas 79 fuentes documentales ya existentes allí: excluye originales privados del workspace. El manifiesto de cada copia indica sus miembros y SHA-256; no intercambiar esos dos ZIP.

Los originales se guardan byte por byte, sin “actualizar” sus cifras. Para consultar uno, abre el ZIP o extráelo fuera de la carpeta de documentación vigente. Las versiones antiguas pueden contener enlaces o instrucciones sustituidos; no deben volver a copiarse a la raíz.

La reorganización retira índices duplicados de la raíz, integra arquitectura/formatos/compatibilidad y políticas en la referencia, y guía de usuario/revisión en una sola guía. Las auditorías extensas siguen disponibles como evidencia, fuera del recorrido de lectura habitual. Los ajustes de enlaces de documentos históricos se registran en el manifiesto, con hash anterior y posterior.

**Aclaración de roles, 2026-11-06:** Luna conserva la traducción/corrección; Sol solo valida muestras o dudas fundamentadas. Se corrigió un valor por defecto que podía seleccionar todas las candidatas; suite 307/307 y sello v4 con las mismas solicitudes, cobertura y presupuesto. Cero API/créditos.

**Ejecución real Luna/Sol, 2026-11-06:** ocho controles Sol favorables, US$0,023968 conciliados y una respuesta Luna vacía/incompleta con reserva US$0,00249593 en cuarentena. Nueve HTTP únicos; cero traducciones nuevas/revisiones naturales y ninguna mejora lingüística medida. Se corrigió previamente un rechazo local del modelo sin HTTP ni cargo; suite 308/308. [Resultado parcial](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/terminology-continuity-docs-20261106/Fase10_Evolucion_Linguistica/HeavyTrial_Run_20261106/README.md).

**Estabilización funcional, 2026-11-06:** contratos/recibos comunes, razonamiento explícito, aislamiento económico de contenido con uso verificado, intención de envío durable, reanudación, contexto TM/incremental, staging pendiente y exportación reversible. Base respaldada 308/308; cierre 332/332 y carga sintética con replay sin reenvíos. Cero API/créditos/aprobaciones; 68/68 archivos protegidos conservados. Reserva incierta intacta y microdiagnóstico US$0,05 preparado sin autorización de ejecución. [Resultado implementado](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/codex/terminology-continuity-docs-20261106/Fase10_Evolucion_Linguistica/Stabilization_20261106/README.md).
