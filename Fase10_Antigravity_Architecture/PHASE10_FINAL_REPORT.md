# Informe final de Fase 10 — estrategia Antigravity-first y arquitectura dual

**Fecha del diseño:** 2026-11-03  
**Estado:** Diseño documental completado; validación operativa pendiente  
**Alcance de este informe:** decisiones, estado y secuencia. El detalle técnico vive en los documentos especializados enlazados abajo.

## 1. Objetivo

Evaluar una ruta para traducir Nolvus Awakening 6.0.21 a ES-419 utilizando primero la suscripción de Google Antigravity, sin activar APIs de pago, sin descargar modelos locales y sin poner en riesgo los archivos originales. La arquitectura API debe mantenerse como alternativa futura.

## 2. Estado

| Elemento | Estado |
|---|---|
| Auditoría de arquitectura y documentación | Diseñada; conclusiones reportadas por el trabajo local de Fase 10 |
| Modelos de Antigravity | El usuario reportó Gemini 4 Argon, Claude Sonnet 5.5 y Claude Opus 5.5 al preparar Fase 10; reconfirmar antes de probar |
| Benchmark de 300 cadenas | Diseñado; no ejecutado |
| Adaptador JSON/JSONL de Antigravity | Pendiente de inspección e implementación/verificación local |
| Gatekeeper local | Pendiente de inspección e implementación/verificación local |
| DB aislada y corpus de benchmark | Deben prepararse/verificarse localmente |
| Adaptadores API | Se describen stubs; no equivalen a integraciones operativas listas |
| Traducción masiva | No iniciada ni autorizada |

Este repositorio contiene documentación, no el código fuente ni la DB del proyecto. Por tanto, las afirmaciones sobre clases existentes, pruebas unitarias o capacidades observadas en la sesión local de Antigravity son resultados reportados y no se han reejecutado desde GitHub.

## 3. Decisión técnica

La primera opción a validar es **intercambio supervisado JSON + JSONL desde un workspace Antigravity físicamente separado, con Gatekeeper local independiente**.

La separación física es deliberada: abrir la raíz del traductor como workspace podría dejar visibles al agente la DB, el glosario, la TM y el código fuente. El agente debe recibir únicamente el paquete de intercambio con la entrada protegida y las instrucciones. No se le permite acceder a los originales ni a la base de datos.

El flujo recomendado es:
1. La aplicación local prepara el manifiesto JSON y el JSONL de entrada.
2. El usuario abre el workspace de intercambio aislado en Antigravity y solicita la traducción del lote.
3. El agente escribe un archivo JSONL completo de salida en staging.
4. El usuario o proceso local controlado ejecuta el Gatekeeper por separado.
5. El Gatekeeper valida IDs, hashes, estructura, placeholders, claves, glosario y ES-419; usa solo DB de prueba durante la evaluación.
6. Líneas parciales o inválidas quedan en cuarentena y se reintentan por UID.

No se depende de `Append=True`, de una hot-folder autónoma, de una API local no documentada ni de automatización de interfaz gráfica. JSONL permite recuperar únicamente registros completos y válidos; no ofrece garantía transaccional ni recuperación automática del 100 %.

El protocolo detallado y el contrato de archivos están en [ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md](./ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md).

## 4. Arquitectura dual

El núcleo común conserva inventario, clasificación, contexto narrativo, glosario, memoria de traducción, protección de variables, QA, revisión humana y exportación.

La capa de ejecución debe ser intercambiable:
- **Antigravity:** intercambio de archivos desde workspace separado, sin API.
- **API futura:** adaptador independiente que consuma el mismo contrato de lote. Debe implementarse y habilitarse solo con autorización expresa, nuevas tarifas verificadas y controles de gasto.
- **Inferencia local:** permanece pausada; no descargar ni instalar modelos en esta etapa.

El detalle está en [DUAL_PROVIDER_ARCHITECTURE.md](./DUAL_PROVIDER_ARCHITECTURE.md).

## 5. Benchmark y calidad

El plan propone comparar hasta tres modelos que estén disponibles en el selector el día de la prueba con las mismas 300 cadenas reales, contexto, glosario e instrucciones. Las sesiones deben ser independientes y la evaluación lingüística debe revisarse de forma ciega.

Los criterios se dividen en:
- **Estructurales:** IDs completos, sin duplicados, sin placeholders alterados ni etiquetas rotas.
- **Lingüísticos:** fidelidad, lore, consistencia terminológica y naturalidad ES-419.
- **Operativos:** primera pasada, errores, reintentos, cuota, tiempos e intervenciones humanas.

El diseño no identifica un modelo ganador. El resultado del benchmark debe decidirlo. Consulta [BENCHMARK_PLAN_ANTIGRAVITY.md](./BENCHMARK_PLAN_ANTIGRAVITY.md).

## 6. Escalabilidad y riesgos

Los umbrales de cadenas por día, sesiones o tamaño de Supertanda que aparecen en el análisis de escalabilidad son estimaciones/criterios propuestos, no cuotas verificadas de la cuenta. El benchmark de 300 cadenas no demuestra viabilidad sobre las aproximadamente 240.684 cadenas únicas recomendadas por Fase 9.

La decisión de mantener Antigravity, pasar a modo híbrido o activar API se realizará solo tras medir un piloto representativo, continuidad y recuperación. Ningún umbral activa gasto automáticamente.

Detalle: [SCALABILITY_AND_RISK_ASSESSMENT.md](./SCALABILITY_AND_RISK_ASSESSMENT.md).

## 7. Secuencia de trabajo

1. **Inspección del código local:** identificar qué contrato, exportador, Gatekeeper y DB de prueba ya existen. No asumir que faltan ni que están listos.
2. **Implementación mínima aislada:** completar solo los componentes ausentes, con autorización del usuario y sin tocar producción.
3. **Preflight + smoke test de 25 cadenas:** confirmar modelos disponibles, AI Credit Overages = Never, permisos Windows, escritura/lectura, validación y recuperación en entorno aislado.
4. **Benchmark de 300 cadenas:** solo después de revisar y aprobar el smoke test.
5. **Piloto de un mod completo:** verificar consistencia entre objetos, misiones, diálogos y textos largos.
6. **Continuidad y recuperación:** medir lotes sucesivos, fallos, recuperación y cuota sostenible.
7. **Decisión de escala:** documentar Antigravity 100 %, híbrido o API. Cualquier activación de API exige nueva cotización y autorización.

El roadmap canónico es [ROADMAP.md](../Documentacion_Base_Fases_0_a_6/ROADMAP.md). Las decisiones vigentes, restricciones y volumen de referencia están en [CURRENT_PROJECT_STATE.md](../documentacion/HISTORICO.md).

## 8. Límites y garantías

- No se han realizado traducciones, benchmark ni smoke test.
- No se han realizado llamadas API ni consumido créditos.
- No se ha modificado código fuente, DB, glosario, TM, STOCK GAME, MO2, mods originales ni partidas.
- No se han descargado ni instalado modelos o herramientas.
- La Fase 9 se cita como informe local de auditoría; las cifras son reportadas y no se han recalculado desde sus manifiestos/DB en este repositorio.
