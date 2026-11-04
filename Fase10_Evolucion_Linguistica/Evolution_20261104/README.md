# Evolución integral y excelencia lingüística — corte 2026-11-04

Estado: implementación local compatible y validación offline; evaluación lingüística humana y comparación de modelos pendientes. La fase facturable permanece detenida.

La fase añade contexto narrativo acotado al intercambio y a la solicitud de prueba, perfiles editoriales compartidos, variantes terminológicas explícitas, preservación de identidades técnicas, TM contextual v2, separación entre aceptación mecánica y aprobación editorial, descompresión limitada y paginación por cursor. Conserva los resultados anteriores y prepara una primitiva de publicación reversible de directorios. La integración de esa primitiva en el exportador sigue pendiente.

La mejora comprobada es funcional: las entradas contextualizadas se serializan, la TM distingue usos, se rechazan variantes incorrectas, el guardado humano comprueba integridad y una candidata mecánica no se convierte en aprobación editorial ni se exporta como tal. No se ha medido una mejora de traducción de un modelo: no hubo llamadas reales y no existen referencias humanas nuevas aprobadas.

## Evidencia y entregables

- [Investigación lingüística, derechos y términos prioritarios](01_INVESTIGACION.md).
- [Revisión por entrada de los 109 términos](01B_GLOSARIO_109.md); las aprobaciones heredadas no se alteraron.
- [Contexto, TM, calidad y corpus de evaluación](02_CONTEXTO_TM_CALIDAD.md).
- [Benchmark, eficiencia y regresión](03_BENCHMARK_COSTES_PRUEBAS.md).
- [Registro de cambios por archivo](04_CAMBIOS.md).
- [Estado final, aceptación y trabajo pendiente](05_ESTADO_FINAL.md).

Los JSON de este paquete contienen informes y metadatos. Los originales y candidatos del piloto seleccionados para evaluación permanecen exclusivamente en el archivo privado local `private/pilot_linguistic_cases.json`, fuera del repositorio documental. Los ejemplos públicos de calidad son textos sintéticos escritos para esta fase.
