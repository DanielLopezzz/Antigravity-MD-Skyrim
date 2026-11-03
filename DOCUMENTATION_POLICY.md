# Política de mantenimiento documental

**Vigente desde:** 2026-11-03 (actualizada en la revisión integral post-Fase 10.1-B.1)

## Fuente única de verdad

1. [PROJECT_STATUS.md](./PROJECT_STATUS.md) es la fuente canónica del estado actual: fases, subsistemas, rama/commits, pruebas, cobertura, riesgos y próxima puerta.
2. [CHANGELOG.md](./CHANGELOG.md) registra cronológicamente el trabajo ya ejecutado y verificado.
3. [ROADMAP.md](./ROADMAP.md) (raíz) contiene únicamente trabajo futuro y puertas pendientes.
4. [DECISIONS.md](./DECISIONS.md) registra las decisiones técnicas y operativas vigentes, con fecha y motivo.
5. El README solo orienta y enlaza; no duplica estado, arquitectura ni políticas.
6. Los informes de auditoría conservan evidencia y metodología del corte al que pertenecen. Sus cifras no se copian a documentos vigentes sin reconciliación.

> `CURRENT_PROJECT_STATE.md` y `Documentacion_Base_Fases_0_a_6/ROADMAP.md` fueron las fuentes canónicas hasta el 2026-11-03. Se conservan con nota de supersesión para trazabilidad.

## Clasificación de documentos

- **Vigente:** decisiones actualmente aprobadas o reglas que el proyecto sigue aplicando.
- **Diseño pendiente de validación:** propuesta técnica que todavía no se probó en el entorno real.
- **Histórico:** describe un corte o fase anterior y no debe usarse como instrucción operativa actual.
- **Obsoleto/sustituido:** reemplazado por una versión posterior. Se conserva solo para trazabilidad.

## Clasificación obligatoria de estados de trabajo

`COMPLETADO` · `IMPLEMENTADO, PENDIENTE DE VALIDACIÓN` · `EN CURSO` · `PENDIENTE` · `BLOQUEADO` · `APLAZADO` · `DESCARTADO`.

## Reglas de edición

- No duplicar la misma matriz de métricas, arquitectura, política lingüística o lista de fases en varios archivos. En los demás documentos, resumir y enlazar la fuente canónica.
- No borrar informes históricos ni reemplazar resultados antiguos silenciosamente. Añadir una nota de supersesión y enlazar la versión vigente.
- Indicar fecha, estado, procedencia y si un dato es medido, reportado, estimado o pendiente de verificación.
- Antes de documentar nombres de pruebas, clases, tablas o comandos, copiarlos de una ejecución real (`python -m unittest discover -s tests -t . -v`, `git log`, lectura directa del código), nunca de memoria o resúmenes.
- No describir modelos, precios, cuotas, versiones, permisos o capacidades de herramientas como actuales sin fecha y fuente oficial.
- No presentar como implementado un diseño, stub, prueba propuesta o capacidad aún no comprobada.
- Evitar rutas locales `file:///` y rutas personales del sistema operativo en documentos nuevos. Preferir rutas relativas al repositorio.
- Toda corrección que cambie una decisión debe registrarse en `DECISIONS.md` con fecha y motivo, y el trabajo realizado en `CHANGELOG.md`.

## Política de preservación

Los informes originales de Fases 7, 8 y 9 y los entregables originales de Fase 10 se preservan como evidencia. La depuración elimina duplicación de navegación, corrige resúmenes vigentes y etiqueta la información histórica; no falsifica ni reescribe los resultados medidos en informes cerrados. El informe 10.1-B.1 se corrigió en su lugar porque contenía errores de transcripción (no resultados medidos distintos); la corrección queda documentada en su Sección 2 y en `CHANGELOG.md`.
