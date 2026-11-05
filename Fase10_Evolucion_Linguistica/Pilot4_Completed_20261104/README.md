# Cuarta prueba: barrido, continuación y resultado completo

Corte local: 2026-11-04 (America/Bogota; artefactos UTC del 2026-11-05). Estado: ejecución de la muestra completada, revisión lingüística humana pendiente. Código y evidencias privadas permanecen locales.

## Resultado medido

- Muestra congelada de **1.000 cadenas**, 100 controles y 900 adicionales, distribuida en diálogo (200), misiones (150), objetos (150), magia (150), libros (150), interfaz (100) y scripts (100).
- **1000 candidatas recibidas**; **975 importadas mecánicamente en staging**, pendientes de revisión. **25 miembros de lotes retenidos por atomicidad**. No equivale a una tasa de calidad semántica ni de aceptación inicial.
- Se conservaron 48 propuestas válidas de R2 y se solicitaron las 952 cadenas restantes durante esta continuación, incluidas dos correcciones iniciales. Hubo 4 solicitudes pagadas de reparación focalizada posteriores; se conserva el historial de cada intento. Un intento de recuperación local de R5 falló antes de enviar HTTP y no se cuenta como llamada pagada.
- El último tramo reutilizó 825 candidatas y solicitó solo 175 nuevas, sin más reparaciones. Los fallos de QA de ese tramo se retienen completos en cuarentena, mientras los errores de transporte, integridad, parseo o conciliación detienen la ejecución.
- Un lote devolvió 25 registros, uno con SHA-256 de origen de 63 caracteres. La validación detuvo la ejecución. Tras comprobar respuesta, correspondencia de UIDs y conciliación, se adjudicó el lote completo a cuarentena, sin corregir el hash ni importar sus textos. Se guardaron progreso y resumen originales antes de continuar únicamente con las 39 entradas aún no enviadas. No fue un reintento ni una reparación pagada.
- Coste conciliado acumulado de todos los intentos de la cuarta prueba: **US$0.1176624**, dentro del límite autorizado de **US$0,17**; saldo **US$0.0523376**. Cálculo del libro local con tarifa Standard y tokens reportados; no es una certificación de la factura final del proveedor ni asume descuentos de caché.
- **109/109 tests pasaron antes del tramo final**, 37,296 s. Tras cerrar las llamadas se reforzó el esquema de SHA-256 y se ejecutó la suite de **110/110 en 54,544 s**. La simulación anterior recorrió las 1.000 fuentes con respuestas simuladas y cero HTTP; no midió calidad del modelo real.
- **494 archivos anteriores idénticos** por tamaño, mtime y SHA-256, incluidas revisiones humanas y base productiva. TM del staging: **0 filas**. Aprobaciones humanas creadas por el agente: **0**. Candado de API restaurado.

## Qué encontró y corrigió el barrido

1. Protección explícita de saltos de línea/tabulaciones reales, además de porcentajes y controles escapados; comprobación de multiplicidad y orden. Se conservan los formatos printf legítimos.
2. Reutilización desde respuestas originales verificadas por hash, con selección congelada, sin mezclar simulaciones y respuestas reales. Cada grupo completo vuelve a pasar Gatekeeper antes de la importación transaccional.
3. Corrección focalizada del alias duplicado y del control final omitido; conservación de los demás textos del grupo. Las reparaciones posteriores son acotadas y usan el mismo presupuesto global.
4. Separación de versión editorial y versión de esquema contextual. Falsa protección de `missives` como nombre del mod corregida mediante coincidencia exacta de mayúsculas en nombres inferidos; identificadores técnicos y términos aprobados mantienen sus reglas.
5. Concordancia explícita de `Renegados` para permitir `algún Renegado` sin cambiar la denominación canónica. Las respuestas correctas de missives y Renegado se revalidaron intactas.
6. Cardinalidad exacta del esquema de salida y normalización conservadora de un duplicado: solo se conserva una traducción completa inequívocamente válida frente a otra parcial. Dos traducciones completas distintas nunca se eligen automáticamente.
7. Evidencia de solicitud sin cabeceras y respuesta antes del parseo; bloqueo de reenvíos inciertos, presupuesto reconciliado y cálculo de tokens de salida por tamaño. Ningún fallo dispara escalamiento automático a otro modelo.
8. Después de completar las llamadas, el esquema de salida se alineó con el validador local: SHA-256 exige 64 caracteres hexadecimales en minúsculas mediante longitud y patrón. [OpenAI documenta estas restricciones](https://developers.openai.com/api/docs/guides/structured-outputs) para modelos estándar. La mejora se verificó offline; no se afirma su validación con otra llamada real. La fuente exacta de R7 se archivó antes del cambio; su plan histórico no se resella ni se reutiliza con otro código.

El [barrido detallado](./COMPONENT_SWEEP.md) distingue las verificaciones iniciales de los defectos descubiertos durante la ejecución real. Los tests previos redujeron riesgos; no anticiparon todos los falsos positivos de QA ni todos los errores del modelo.

## Procedencia editorial y revisión

Hay **una propuesta de formato hecha por IA**: se repuso un CRLF omitido en un poema, sin cambiar palabras ni puntuación. Se conserva tanto la respuesta fallida como la propuesta y está marcada para revisión obligatoria. No se cuenta como una respuesta API intacta ni como aprobación humana.

La auditoría final marcó 42 casos por procedencia, cifras o posibles problemas editoriales. Hay **196 casos de revisión obligatoria** y **199 de muestra de calidad sin alertas previas**. No son cantidades de errores confirmados. Diferencias entre números y cantidades escritas con palabras requieren juicio humano.

Entre los puntos para revisar están el sentido técnico de `Form` en Papyrus, nombres de equipo como `Shrouded Gloves`, el sentido de `Frostbite Venom` traducido como «Veneno de escarcha», fórmulas como `Estimado/a` y uniformidad de nombres de libros/hechizos. Son dudas que requieren contexto, no adjudicaciones terminológicas nuevas. La comprobación automática de las candidatas importadas detectó **0 fallos de placeholders**, **0 diferencias de controles reales** y **0 diferencias de controles escapados**. Estas comprobaciones no prueban fidelidad, voz o naturalidad.

Los dos casos de regresión de Elder Scroll y el nombre HeelsFix Refresh están incluidos en la muestra; su validez mecánica no sustituye la lectura humana del contexto.

## Consumo por intento

| Ejecución | Solicitudes pagadas | Tokens entrada | Tokens salida | Coste USD |
|---|---:|---:|---:|---:|
| Phase4_Luna1000 | 2 | 10,866 | 6,798 | 0.0044856 |
| Phase4_Luna1000_R2 | 2 | 10,860 | 6,623 | 0.0043975 |
| Phase4_Luna1000_R3_Continuation | 18 | 104,801 | 58,972 | 0.0399661 |
| Phase4_Luna1000_R4_Completion | 6 | 35,035 | 19,020 | 0.0130135 |
| Phase4_Luna1000_R5_Final | 10 | 67,124 | 40,469 | 0.0269469 |
| Phase4_Luna1000_R6_Complete | 3 | 16,651 | 9,252 | 0.0062911 |
| Phase4_Luna1000_R7_DiagnosticComplete | 9 | 54,242 | 34,275 | 0.0225617 |

La suma corresponde a 50 respuestas reales verificadas; las reservas están conciliadas, sin saldo reservado ni coste incierto. Se conservaron los informes fallidos originales; las continuaciones tienen identidad propia.

## Opinión y siguiente paso

La muestra completa permite evaluar el sistema de forma mucho más útil que los intentos detenidos. La reutilización, la cuarentena y los controles económicos funcionan; los falsos rechazos observados justifican mantener supervisión antes de escalar. El resultado técnico habilita la revisión humana, pero todavía no acredita calidad suficiente para traducir o exportar todo Nolvus sin revisión.

Este resultado reúne continuaciones con cambios de reglas y reparaciones documentadas. No constituye una única ejecución ciega con configuración invariable, ni permite atribuir una mejora lingüística cuantificada frente al piloto anterior. Se completó la cobertura de la misma muestra, preservando la procedencia de cada candidata.

Revisar primero los casos obligatorios y después la muestra por categoría. Registrar significado, naturalidad, voz, terminología, contexto y formato; distinguir retoques de errores graves y medir también las cadenas sin alertas. Las correcciones que impliquen una regla general deben convertirse en regresiones antes de otra prueba. No repetir toda la muestra: una futura prueba debe usar solo casos pendientes o un conjunto nuevo y conservar controles comparables.

## Cómo abrir los resultados locales

`Revisar_cuarta_prueba.bat` apunta a `NolvusAIT_PilotStaging/Phase4_Luna1000_R7_DiagnosticComplete`. Ahí se reúnen las 1.000 candidatas con su procedencia. Las decisiones del usuario se guardan en su propio `private/human_reviews.json`; las revisiones anteriores siguen intactas. La aprobación en el visor registra una referencia de evaluación y no modifica producción ni alimenta la TM.

Evidencia local: `final_metrics.json`, `protected_verification.json`, `full_tests_final.txt`, `full_tests_after_hash_fix.txt`, `full_corpus_simulation.json` y los cuerpos originales/ensamblajes de cada staging. El paquete documental contiene solo este informe, el barrido y métricas agregadas; excluye código, corpus, bases, credenciales y respuestas privadas. No se ha realizado commit ni push durante esta tarea.
