# Plan de hitos verificables — Nolvus AI Translator

**Última consolidación documental:** 2026-11-03  
**Fuente de estado y métricas actuales:** [CURRENT_PROJECT_STATE.md](../../CURRENT_PROJECT_STATE.md)

Este roadmap reemplaza el plan inicial de nueve fases. Distingue el trabajo histórico completado del trabajo que aún requiere validación. No implica autorización para ejecutar traducciones, activar API, modificar MO2 ni tocar la instalación original.

## Estado de fases

| Fase | Alcance | Estado documental | Observación |
|---:|---|---|---|
| 0–6 | Investigación, diseño, infraestructura, extracción, clasificación, glosario/TM, QA simulado y exportación no invasiva | Reportadas como completadas en el historial del proyecto | La implementación debe verificarse contra el código actual antes de afirmar que cada componente sigue operativo. |
| 7 | Escaneo real y snapshot inicial | Completada | Sus métricas son históricas; fueron ampliadas y reconciliadas en Fase 9. |
| 8 | Estimación económica y comparativa de proveedores | Completada como análisis histórico | Los precios, modelos y escenarios no son cotización actual. Recalcular antes de cualquier gasto. |
| 9 | Auditoría integral y extracción de formatos pendientes | Completada según informe del 2026-11-03 | Ver [informe de Fase 9](../../Fase9_Full_Audit/PHASE9_FULL_AUDIT_REPORT.md). La base SQLite/manifiestos fuente no están publicados en este repositorio. |
| 10 | Evaluación de estrategia Antigravity-first y arquitectura dual | Diseño documental completado | No se ha ejecutado benchmark ni se ha implementado el adaptador/Gatekeeper. Las correcciones están registradas en el informe final. |
| 10.1 | Preflight y smoke test aislado de 25 cadenas | Pendiente | Confirmar cuota/overages, permisos Windows, herramienta real de escritura y validador sobre una DB de prueba. |
| 10.2 | Benchmark controlado de 300 cadenas × 3 modelos | Pendiente | Solo después de superar 10.1; comparar calidad y operación en sesiones aisladas. |
| 10.3 | Piloto de un mod completo (~1.100–1.500 cadenas) | Pendiente | Solo después de revisar resultados del benchmark y aprobar modelo/protocolo. |
| 10.4 | Continuidad y recuperación | Pendiente | 12–15 lotes consecutivos y prueba de interrupción, sin corrupción ni pérdida silenciosa. |
| 10.5 | Decisión de escala | Pendiente | Elegir Antigravity, híbrido o API con mediciones reales; conservar infraestructura API. |
| Producción | Traducción global y exportación de paquetes de prueba | No autorizada/iniciada | Requiere decisión expresa después de las puertas anteriores. |

## Siguiente secuencia

### 10.1 — Preflight y smoke test

1. Confirmar en el selector de Antigravity cuáles modelos están disponibles en ese momento. Los nombres confirmados históricamente por el usuario son Gemini 4 Argon, Claude Sonnet 5.5 y Claude Opus 5.5; no se presume que sigan disponibles.
2. En Settings, comprobar que **AI Credit Overages = Never**. Si no puede confirmarse, no consumir cuota de modelos hasta resolverlo.
3. Confirmar el sistema de permisos de la versión Windows instalada. No extrapolar los presets actuales de macOS/Linux a Windows.
4. Probar que el agente puede leer el paquete de entrada y escribir un archivo de salida completo en una carpeta de prueba autorizada, sin depender de `Append=True`.
5. Ejecutar el Gatekeeper local solo contra una base de datos y un corpus de prueba aislados. Validar cantidad, IDs, duplicados, placeholders, etiquetas, claves y hashes.
6. Simular salida truncada e interrupción. Recuperar solo líneas JSON completas; poner en cuarentena la línea final incompleta y reintentar únicamente sus IDs.

**Criterio de salida:** contrato reproducible, cero acceso del agente a la DB de producción, cero escrituras fuera de staging, cero traducciones aceptadas con defectos estructurales y recuperación documentada. El smoke test no traduce datos de producción ni cambia archivos de Nolvus.

### 10.2 — Benchmark

Ejecutar las 300 cadenas del corpus con cada modelo disponible, exactamente la misma entrada y contexto, sesiones separadas y resultados anonimizados para revisión ciega. No usar costes de API como métrica de esta modalidad; registrar cuota, pausas, tiempo activo, errores, reintentos y acciones humanas.

### 10.3–10.5 — Escala

Ampliar gradualmente a un mod piloto, continuidad y recuperación. La velocidad sostenible debe calcularse con cadenas aceptadas por hora/día y cuota observada, no extrapolando la prueba de 300 cadenas sin datos.

## Invariantes de seguridad

- No modificar STOCK GAME, mods originales, BSA, load order, perfiles principales MO2 ni partidas.
- No escribir desde el agente en la DB principal, glosario maestro, memoria de traducción ni carpetas de exportación productiva.
- No instalar herramientas, descargar modelos locales, activar APIs, comprar créditos ni ejecutar traducciones masivas sin autorización expresa.
- La infraestructura API se conserva como alternativa; las estimaciones de Fase 8 no autorizan activarla.
- Cada fase se detiene en su puerta de salida y entrega un informe antes de continuar.
