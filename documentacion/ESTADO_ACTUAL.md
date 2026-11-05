# Estado actual y próximos pasos

Verificado el **2026-11-05**. Esta es la fuente del estado vigente; [uso](GUIA_DE_USO.md), [reglas y arquitectura](REFERENCIA_TECNICA.md) e [histórico](HISTORICO.md) cubren temas distintos. Las cifras de revisión son un corte, no un contador en tiempo real.

## Dónde está el proyecto

Nolvus AI Translator localiza Nolvus Awakening al español latinoamericano neutro (ES-419). El piloto de 1.000 cadenas está procesado; la evolución inteligente está implementada y probada offline en copias separadas. **Quedan calibración y validación real autorizada; la traducción masiva y la promoción productiva no están autorizadas.**

| Dato comprobado | Resultado e interpretación |
|---|---|
| Muestra del cuarto piloto | 1.000 candidatas recibidas; 100 controles y 900 adicionales |
| Staging aislado | 975 importadas mecánicamente; no equivale a aprobación lingüística |
| Cuarentena | 25 retenidas por atomicidad: un registro devolvió un hash de origen de 63 caracteres |
| Prioridad obligatoria del piloto histórico | 196 casos, incluidos los 42 señalados por auditoría; ese archivo no se modificó |
| Muestra de calidad | 199 casos distintos; unión con obligatorios: 395; otros 605 con prioridad adicional |
| Revisión humana al corte | 3 aprobadas sin cambios y 997 pendientes en el visor; sus decisiones no modifican la DB |
| Consumo acumulado del cuarto piloto | US$0,1176624 de US$0,17; 50 respuestas reales, sin llamadas pendientes |
| TM del staging / promoción a producción | 0 / 0; no se crearon aprobaciones humanas automáticas |
| Última suite registrada | 136/136 en la evolución offline; antes: 110/110. Registro exacto en el informe de evolución |
| Código local | Rama `codex/linguistic-evolution`, base `066ae03`, con cambios locales sin commit |

Coste calculado por el libro local con tokens reportados y tarifa registrada: no certifica la factura ni una tarifa vigente del proveedor. Las últimas 110 pruebas incluyen el refuerzo del esquema SHA-256, comprobado offline después de cerrar la API. Ese último ajuste no se ha validado con otra llamada pagada.

## Qué está demostrado y qué no

- Se conservaron respuestas originales, hashes, presupuesto acotado, importación atómica y reutilización de candidatas válidas. La auditoría final registró cero diferencias de placeholders y controles entre las 975 importadas.
- El resultado reúne continuaciones y reparaciones con reglas ajustadas. El 97,5 % de importación es acumulado; no mide acierto semántico ni aceptación inicial.
- El contexto enviado es limitado: no hay indicación de hablante en las 200 cadenas de diálogo; el runner agrupa por tamaño y puede mezclar grupos narrativos.
- Hay una propuesta de formato de IA para un poema, pendiente humana. Está identificada; no es una respuesta intacta del proveedor.
- El análisis halló dos falsos positivos numéricos y 132 casos con solo alerta de nombres/lore. El nuevo validador separa severidades y normaliza cifras; conserva los informes históricos.

## Evolución inteligente verificada offline

La copia experimental reclasifica las mismas candidatas a **38 obligatorias (25 de cuarentena y 13 editoriales), 199 de muestra y 763 adicionales**, sin aprobar o reescribir ninguna. La reducción del 80,61 % es de prioridad, no de tiempo humano o errores semánticos medidos. Hay solo dos referencias para calibración y una para validación: aprobación automática nueva deshabilitada.

Se implementaron contexto por mod/plugin, contrato 1.2 compatible con lectura 1.0/1.1, propuestas terminológicas separadas, reparación técnica reversible, juez simulado y ejecución por etapas con alcance fijo. Se indexaron 12.423 nombres de registro NPC; no se infieren biografías. El benchmark de 240.684 pasajes sintéticos verificó la ingesta SQLite, no la traducción completa.

La preparación coherente con recuperación de esta muestra dispersa estima US$0,39955632 frente a US$0,16704072 sin recuperación. Excede el cap anterior: la opción no se activó por defecto ni se elevó el presupuesto. Cero solicitudes facturables y 2.186 archivos anteriores verificados idénticos. [Informe y métricas](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/a232c2edc575469e3f029ff99e6eeb4774cf4273/Fase10_Evolucion_Linguistica/Intelligence_20261105/README.md).

## Próximos pasos

| Orden | Trabajo pendiente | Condición para avanzar |
|---|---|---|
| 1 | Revisión humana representativa, incluidos casos sin alertas | Separar errores de significado de retoques y advertencias |
| 2 | Adjudicar familias terminológicas y el lote retenido | Conservar evidencia; no retransmitir todo el lote por defecto |
| 3 | Revisar los cambios offline ya implementados | Validar su comportamiento en el ámbito real, sin modificar el histórico |
| 4 | Calibrar y evaluar un juez independiente autorizado | Medir falsas aprobaciones y ahorro de trabajo; sin autoaprobación nueva |
| 5 | Preparar un piloto acotado de un mod | Configuración congelada, contexto coherente, presupuesto y autorización explícitos |

No se recomienda aumentar carga sin calibración y presupuesto compatible. El puerto del juez está probado con mocks; su ejecución real, el escalamiento, la aprobación automática nueva y la producción siguen pendientes. Los cambios de esta fase se proponen en una PR solo documental; el código permanece privado.

## Fuentes del corte

- [Ejecución completa y consumo](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/main/Fase10_Evolucion_Linguistica/Pilot4_Completed_20261104/README.md).
- [Análisis profesional: alertas, falsos positivos y recomendaciones](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/main/Fase10_Evolucion_Linguistica/Pilot4_Review_Analysis_20261105/README.md).
- Evidencia local: `NolvusAIT_Pilot4_Sweep/20261104/final_metrics.json`, `full_tests_after_hash_fix.txt` y `NolvusAIT_PilotStaging/Phase4_Luna1000_R7_DiagnosticComplete`.

El inventario de ~240.684 cadenas es una estimación histórica reconciliada; no se ha vuelto a escanear Nolvus para actualizarla. Las cifras iniciales de otras fases se consultan en el histórico, no se suman a la muestra actual.
