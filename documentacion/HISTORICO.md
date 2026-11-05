# Histórico resumido y acceso a evidencia

Organizado el **2026-11-05**. Es un índice de hechos cerrados y decisiones sustituidas, no una guía de ejecución. [Estado vigente](ESTADO_ACTUAL.md) · [Uso](GUIA_DE_USO.md) · [Reglas](REFERENCIA_TECNICA.md).

## Cronología objetiva

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

## Informes que conviene abrir solo cuando haga falta

Los enlaces siguientes apuntan a la documentación ya publicada; su estructura de fases se conserva para no perder enlaces externos.

- [Análisis profesional de revisión humana](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/main/Fase10_Evolucion_Linguistica/Pilot4_Review_Analysis_20261105/README.md).
- [Resultado completo y barrido del cuarto piloto](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/main/Fase10_Evolucion_Linguistica/Pilot4_Completed_20261104/README.md).
- [Preparación](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Fase10_Evolucion_Linguistica/Pilot4_Preparation_20261104), [primera etapa](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Fase10_Evolucion_Linguistica/Pilot4_Stage1_20261104) y [R2](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Fase10_Evolucion_Linguistica/Pilot4_R2_20261104): incidencias y evidencias anteriores.
- [Evolución lingüística offline](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Fase10_Evolucion_Linguistica/Evolution_20261104).
- [Auditoría integral de código](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Fase10_1_Code_Audit/Audit_20261104_Integral_Post_Pilot) y [especificaciones/auditorías anteriores](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Fase10_1_Code_Audit).
- [Arquitectura, preparación y pilotos 10.3](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Fase10_Antigravity_Architecture).
- [Auditoría Fase 9](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/blob/main/Fase9_Full_Audit/PHASE9_FULL_AUDIT_REPORT.md), [Fases 7–8](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Fase7_y_Fase8_Informes) y [base 0–6](https://github.com/DanielLopezzz/Antigravity-MD-Skyrim/tree/main/Documentacion_Base_Fases_0_a_6).

## Originales y trazabilidad

[Archivo de originales](archivo/originales_20261105.zip) · [Manifiesto y hashes](archivo/MANIFIESTO.json).

El ZIP del workspace conserva los 20 Markdown originales de su raíz, la hoja de estimación y las 79 fuentes Markdown/JSON previas del repositorio documental. El ZIP del espejo público contiene solo esas 79 fuentes documentales ya existentes allí: excluye originales privados del workspace. El manifiesto de cada copia indica sus miembros y SHA-256; no intercambiar esos dos ZIP.

Los originales se guardan byte por byte, sin “actualizar” sus cifras. Para consultar uno, abre el ZIP o extráelo fuera de la carpeta de documentación vigente. Las versiones antiguas pueden contener enlaces o instrucciones sustituidos; no deben volver a copiarse a la raíz.

La reorganización retira índices duplicados de la raíz, integra arquitectura/formatos/compatibilidad y políticas en la referencia, y guía de usuario/revisión en una sola guía. Las auditorías extensas siguen disponibles como evidencia, fuera del recorrido de lectura habitual. Los ajustes de enlaces de documentos históricos se registran en el manifiesto, con hash anterior y posterior.
