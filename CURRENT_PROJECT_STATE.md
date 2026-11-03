# Estado actual y decisiones vigentes — Nolvus AI Translator

**Última revisión documental:** 2026-11-03  
**Fuente de estado del entorno:** informe final de Fase 9 incluido en este repositorio.  
**Estado del proyecto:** diseño de Fase 10 completado; auditoría estática 10.1-A y reconciliación documental completadas; especificación 10.1-B aprobada e **implementación aislada 10.1-B.1 completada y verificada localmente** (`23/23` pruebas en verde; puertas `B1–B7` superadas en rama `feat/phase10-1b-antigravity-exchange`). El preflight y smoke test de 25 cadenas (`10.2`) sigue pendiente de autorización.

> Este documento es la fuente de verdad para el estado vigente del proyecto. Los informes de fases anteriores conservan sus cifras y conclusiones como evidencia histórica. Cuando un informe histórico difiera de este estado, prevalecen el último corte de auditoría y las decisiones posteriores documentadas aquí.

## 1. Estado verificado en el corte de Fase 9

Los siguientes datos se transcriben del informe `Fase9_Full_Audit/PHASE9_FULL_AUDIT_REPORT.md`. Este repositorio contiene el informe, pero no la base de datos SQLite ni todos los manifiestos originales; por tanto, quien consulte este repositorio puede comprobar la consistencia documental, no repetir independientemente el escaneo de MO2.

| Elemento | Valor del corte Fase 9 | Interpretación |
|---|---:|---|
| Nolvus Awakening | 6.0.21 Ultimate | Variante y versión reportadas |
| SkyrimSE.exe de STOCK GAME | 1.5.97.0 | Versión binaria reportada por el corte |
| Mod Organizer 2 | 2.4.4.0 | Versión binaria reportada |
| Mods activos | 3.700 | Entradas activas de modlist |
| Plugins activos | 3.717 | ESM/ESL/ESP/ESPFE |
| Archivos BSA activos | 583 | Archivos ganadores del VFS |
| Elementos auditados | 1.168.337 | Archivos sueltos y miembros empaquetados; no equivale a cadenas |
| Ocurrencias traducibles del Corte C | 328.485 | Incluye duplicados y ocurrencias |
| Cadenas únicas pendientes del Corte C | 240.244 | Deduplicación según el informe |
| Subgrupo 4A JSON narrativo | ~440 únicas / ~900 ocurrencias | Adición recomendada, sujeta a allowlist |
| Volumen neto recomendado | ~240.684 únicas | 240.244 + ~440 |

La cifra de **240.684** es una estimación consolidada del informe, porque el componente JSON 4A se expresa como aproximado. No debe presentarse como un conteo exacto hasta confirmar el inventario detallado.

Otros hallazgos que afectan al flujo: 38 cadenas de 3DNPC con codificación defectuosa de origen; 26.902 cadenas con variables/placeholders; 8.667 ocurrencias de nombres propios protegidos (4.931 únicas); 201 PSC huérfanos excluidos como objetivo de traducción; 678 SWF y 211 DLL que no deben parchearse directamente; y 7.089 texturas con posible texto que quedan fuera de la traducción textual inicial.

## 2. Decisión de ejecución vigente

La ruta prioritaria es **Antigravity mediante la suscripción del usuario, sin API externa**, pero todavía no existe evidencia empírica suficiente para declarar ganador a un modelo ni para afirmar que la ruta escala a todo el volumen.

Modelos que el usuario confirmó disponibles en su selector al preparar la Fase 10:

- Gemini 4 Argon
- Claude Sonnet 5.5
- Claude Opus 5.5

La disponibilidad debe reconfirmarse el día de cada prueba. GPT-6 Luna y GPT-6 Sol no forman parte del benchmark de Antigravity. La inferencia local sobre la RX 9070 XT está pausada.

La arquitectura API se conserva como alternativa independiente. Los stubs descritos en la documentación no equivalen a adaptadores API operativos: no se deben activar, configurar ni invocar APIs hasta que el usuario lo autorice expresamente y se actualicen precios, límites y controles.

## 3. Arquitectura recomendada para validar

Se recomienda un **intercambio supervisado JSON + JSONL desde un workspace Antigravity físicamente separado del proyecto del traductor, con staging y Gatekeeper local determinista**:

1. El núcleo crea el manifiesto y el JSONL de entrada, con IDs y hash de contenido, en una carpeta de intercambio independiente, por ejemplo `C:\\NolvusAIT_Exchange\\job_<job_id>\\`.
2. Se abre únicamente esa carpeta como workspace de Antigravity. No se abre la raíz del traductor, que contiene código, DB, glosario y TM. El paquete de intercambio no contiene esos recursos ni archivos de Nolvus.
3. Antigravity lee los archivos de entrada y devuelve un único archivo JSONL completo por lote en staging. No se depende de `Append=True`, de escritura incremental ni de una API local no documentada.
4. El Gatekeeper valida sintaxis, esquema, cantidad y conjunto exacto de IDs, hashes, placeholders, etiquetas, claves no traducibles, glosario y reglas ES-419.
5. Las líneas completas y válidas pueden recuperarse individualmente; la última línea incompleta, líneas malformadas, IDs duplicados, ausentes o inesperados se ponen en cuarentena y se reintentan de forma aislada.
6. La salida validada queda como candidata. Durante smoke test y benchmark se usa exclusivamente una base de datos de prueba. La base de producción no recibe resultados hasta una autorización específica y una transacción controlada.

**JSONL no es por sí mismo una transacción ni garantiza una escritura íntegra.** Su ventaja es que facilita detectar y recuperar registros completos por línea. La tolerancia real a interrupciones debe demostrarse en la prueba de recuperación. El agente no recibe el Gatekeeper, la DB de prueba ni la DB productiva dentro de su workspace.

## 4. Orden de validación

La validación se ejecuta por puertas; no se salta a traducción global por superar un benchmark pequeño. La etapa de implementación aislada (`10.1-B.1`) ya fue completada y verificada con pruebas automatizadas locales; las etapas con modelos reales (`10.2` en adelante) permanecen pendientes de autorización.

| Etapa | Alcance | Condición de salida |
|---|---|---|
| Implementación aislada (10.1-B.1) | Exportador JSON/JSONL, bóveda local, Gatekeeper, microlotes y DB de prueba | **Completada localmente** (`23/23` pruebas OK; puertas `B1–B7` superadas) |
| Preflight | Confirmar modelos, permisos Windows, carpeta de workspace y AI Credit Overages = Never | Sin gasto automático y sin acceso a originales/DB productiva |
| Smoke test | 25 cadenas reales con el protocolo de archivos y base aislada | IDs, estructura, QA y recuperación demostrados |
| Benchmark | 300 cadenas × 3 modelos, entradas idénticas y sesiones aisladas | Comparación ciega de calidad y métricas operativas |
| Piloto | Un mod completo representativo (~1.100–1.500 cadenas) | Coherencia de términos y QA a escala de mod |
| Continuidad y recuperación | 12–15 lotes consecutivos y fallos controlados | Sin pérdida silenciosa; recuperación medida y reproducible |
| Decisión de escala | Evaluación de cuotas, ritmo sostenible, errores y carga humana | Decisión documentada: Antigravity, híbrido o API |

El benchmark de 300 cadenas no demuestra por sí solo que Antigravity pueda procesar ~240.684 cadenas.

## 5. Controles obligatorios antes de cualquier prueba de modelo

- Confirmar que la opción **AI Credit Overages** esté en **Never**. La documentación oficial indica que así no se consumen créditos automáticamente al agotar la cuota base; se espera a la renovación. Si no puede confirmarse, no ejecutar solicitudes de modelos.
- No introducir claves API, no habilitar facturación y no llamar proveedores externos.
- Registrar modelo exacto, fecha, cuota visible antes/después, avisos de límite, tiempos e intervenciones humanas.
- Probar las operaciones reales de lectura y escritura de Antigravity en el workspace. La documentación pública describe permisos y acceso al workspace, pero no establece un contrato de `write_file Append=True`; por tanto, no depender de esa opción.
- No permitir al agente modificar SQLite, glosario maestro, memoria de traducción ni archivos originales.
- No instalar herramientas, descargar modelos ni activar la inferencia local durante esta etapa.

## 6. Reglas de decisión

No se elige el modelo por reputación o por una única puntuación agregada. La selección debe respetar este orden:

1. **Puertas estructurales:** 100 % de preservación de IDs, placeholders y etiquetas en los resultados que se acepten; cero cadenas vacías, duplicadas o inesperadas.
2. **Fidelidad y lore:** sin alucinaciones, cambios de sentido ni violaciones del glosario o nombres protegidos.
3. **ES-419:** neutralidad lingüística y cero formas peninsulares prohibidas.
4. **Estabilidad:** primera pasada, truncamientos, fallos de herramientas, reintentos y recuperación.
5. **Operación:** throughput validado, cuota real, pausas, intervenciones humanas y facilidad de supervisión.

Los umbrales numéricos del plan de benchmark son criterios propuestos de aceptación, no resultados medidos. Cualquier ajuste debe justificarse antes de ver los resultados, para evitar mover el criterio a favor de un modelo.

La decisión de migrar a API solo se toma cuando haya datos de las pruebas y una nueva estimación de costes con las tarifas vigentes. Las estimaciones económicas de Fase 8 son históricas y no autorizan gasto.

## 7. Jerarquía documental

La auditoría del código 10.1-A y su reconciliación están disponibles en [PHASE10_1_CODE_AUDIT_REPORT.md](./Fase10_1_Code_Audit/Audit_20261103_113100/PHASE10_1_CODE_AUDIT_REPORT.md) y [AUDIT_10_1A_RECONCILIATION.md](./Fase10_1_Code_Audit/AUDIT_10_1A_RECONCILIATION.md). La especificación de implementación se encuentra en [PHASE10_1B_IMPLEMENTATION_SPEC.md](./Fase10_Antigravity_Architecture/PHASE10_1B_IMPLEMENTATION_SPEC.md) y el informe de su implementación y verificación en [PHASE10_1B1_IMPLEMENTATION_REPORT.md](./Fase10_Antigravity_Architecture/PHASE10_1B1_IMPLEMENTATION_REPORT.md).

| Documento | Función única |
|---|---|
| `README.md` | Entrada y navegación del repositorio |
| `CURRENT_PROJECT_STATE.md` | Estado vigente, cifras y decisiones actuales |
| `ROADMAP.md` | Fases, puertas y siguiente paso |
| `PHASE9_FULL_AUDIT_REPORT.md` | Evidencia detallada del último inventario |
| `PHASE10_FINAL_REPORT.md` | Decisiones y resumen de arquitectura |
| `BENCHMARK_PLAN_ANTIGRAVITY.md` | Diseño del benchmark y su rúbrica |
| `ANTIGRAVITY_INTEGRATION_ARCHITECTURE.md` | Protocolo de intercambio y permisos |
| `DUAL_PROVIDER_ARCHITECTURE.md` | Separación Antigravity/API |
| `TRANSLATION_EXECUTION_ENGINE.md` | Lotes, QA, reintentos y trazabilidad |
| `SCALABILITY_AND_RISK_ASSESSMENT.md` | Cuotas, escalabilidad y decisión de escala |
| `PHASE10_1B_IMPLEMENTATION_SPEC.md` | Especificación técnica de implementación 10.1-B |
| `PHASE10_1B1_IMPLEMENTATION_REPORT.md` | Evidencia de implementación y pruebas 10.1-B.1 |
| `TRANSLATION_POLICY_ES419.md` | Política lingüística canónica |
| `GLOSSARY_POLICY.md` | Política terminológica canónica |

Los documentos de Fases 0–8 y los informes de snapshots anteriores son evidencia histórica. No se deben copiar sus cifras o recomendaciones a los documentos vigentes sin contrastarlas con Fase 9 y las decisiones posteriores.
