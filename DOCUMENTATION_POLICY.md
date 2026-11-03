# Política de mantenimiento documental

**Vigente desde:** 2026-11-03

## Fuente única de verdad

1. [CURRENT_PROJECT_STATE.md](./CURRENT_PROJECT_STATE.md) es la fuente canónica para cifras, entorno reportado, decisiones y estado actual.
2. [ROADMAP.md](./Documentacion_Base_Fases_0_a_6/ROADMAP.md) es la fuente canónica para hitos y puertas siguientes.
3. Cada documento técnico explica un único tema. El README solo orienta y enlaza; no duplica arquitectura ni vuelve a desarrollar políticas.
4. Los informes de auditoría conservan evidencia y metodología del corte al que pertenecen. Sus cifras no se copian a documentos vigentes sin reconciliación.

## Clasificación de documentos

- **Vigente:** decisiones actualmente aprobadas o reglas que el proyecto sigue aplicando.
- **Diseño pendiente de validación:** propuesta técnica que todavía no se probó en el entorno real.
- **Histórico:** describe un corte o fase anterior y no debe usarse como instrucción operativa actual.
- **Obsoleto/sustituido:** reemplazado por una versión posterior. Se conserva solo para trazabilidad.

## Reglas de edición

- No duplicar la misma matriz de métricas, arquitectura, política lingüística o lista de fases en varios archivos. En los demás documentos, resumir y enlazar la fuente canónica.
- No borrar informes históricos ni reemplazar resultados antiguos silenciosamente. Añadir una nota de supersesión y enlazar la versión vigente.
- Indicar fecha, estado, procedencia y si un dato es medido, reportado, estimado o pendiente de verificación.
- No describir modelos, precios, cuotas, versiones, permisos o capacidades de herramientas como actuales sin fecha y fuente oficial.
- No presentar como implementado un diseño, stub, prueba propuesta o capacidad aún no comprobada.
- Evitar rutas locales `file:///` en documentos nuevos. Preferir rutas relativas al repositorio; las rutas locales se dejan en informes históricos solo como evidencia.
- Toda corrección que cambie una decisión debe registrarse en el informe vigente o en un nuevo registro de decisiones, con fecha y motivo.

## Política de preservación

Los informes originales de Fases 7, 8 y 9 y los entregables originales de Fase 10 se preservan como evidencia. La depuración elimina duplicación de navegación, corrige resúmenes vigentes y etiqueta la información histórica; no falsifica ni reescribe los resultados medidos en informes cerrados.
