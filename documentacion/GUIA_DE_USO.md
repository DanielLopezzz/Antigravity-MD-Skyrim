# Guía de uso y revisión local

Actualizada el **2026-11-05** a partir de los lanzadores, CLI y visor existentes. [Estado y pendientes](ESTADO_ACTUAL.md) · [Referencia técnica](REFERENCIA_TECNICA.md).

## Accesos desde la raíz del workspace

| Acceso | Qué abre |
|---|---|
| `launch_gui.bat` | Aplicación principal; puede inicializar sus carpetas de trabajo |
| `Revisar_cuarta_prueba.bat` | Visor del piloto activo R7 de 1.000 cadenas |
| `Revisar_traducciones.bat` | Casos del piloto anterior y ejemplos sintéticos |
| `Revisar_evolucion_inteligente.bat` | Copia experimental del flujo de reevaluación offline |
| `Revisar_prueba_pareada.bat` | Resultado real separado: 47 candidatas para evaluar y una en cuarentena técnica |

Estos lanzadores pertenecen al workspace con código. El repositorio público de documentación no incorpora la aplicación. El visor de casos no llama a la API ni importa decisiones a producción o a la TM.

La aplicación principal conserva diez áreas: dashboard, escaneo, traducciones, glosario, TM, revisión, actualizaciones, exportación, informes y configuración. El visor se puede abrir también desde **6. Revisión → Revisar casos del informe**. La cola de la DB y los casos del informe tienen funciones distintas.

## Revisar la cuarta prueba

1. Abre `Revisar_cuarta_prueba.bat`. Comprueba que la cabecera indica `Phase4_Luna1000_R7_DiagnosticComplete`.
2. Usa **Recargar** para leer las candidatas y decisiones guardadas. Selecciona estado y prioridad: **Obligatoria**, **Muestra de calidad** o **Adicional**.
3. Compara original, candidata, contexto, procedencia y alertas. Que una frase no tenga alertas no demuestra que esté bien traducida.
4. Escribe tu nombre. Puedes **Aprobar caso**, **Guardar pendiente**, **Requiere investigación** o **Rechazar caso**. Investigar/rechazar requiere una nota.
5. Anota significado, naturalidad, voz, terminología, contexto o formato; distingue retoques menores y errores graves. Usa **Siguiente pendiente** para continuar.

Antes de aprobar se comprueban integridad, términos obligatorios y registro regional. La revisión humana debe juzgar también el sentido y la voz. La neutralidad ES-419 no exige calcar el inglés ni sustituir formulaciones naturales por otras rígidas.

Los casos en cuarentena se pueden evaluar lingüísticamente, pero aprobar una referencia en el visor **no arregla el hash ni libera el lote para importación**. La propuesta de formato del poema está marcada para revisión obligatoria y conserva su procedencia de IA.

Si hay texto sin guardar, el visor avisa al navegar. Si otra ventana guarda decisiones, usa **Recargar**; hay bloqueo de guardado e historial. Una decisión se vincula al original y al contexto revisados.

La copia de evolución se abre con `Revisar_evolucion_inteligente.bat`. Muestra estado del evaluador, procedencia y controles sin crear aprobaciones. Los dictámenes del simulador son inconclusos; no miden calidad lingüística. Su carpeta es `NolvusAIT_Intelligence/20261105_Functional/private/review_final`; las decisiones se guardan en su propio `private/human_reviews.json`. Cuando existe ese overlay, el ejecutor experimental rechaza regenerar la copia. Las aprobaciones previas exactas aparecen como **Referencia aprobada**, leídas de una copia inmutable separada; no se duplican en el overlay de decisiones nuevas. **Exportar evaluación** guarda referencias y exclusiones de calibración en `private/calibration_summary.json`; descarta dictámenes simulados y referencias desactualizadas. Los segmentos se recomponen solo tras comprobar su identidad, orden e integridad; si la revisión resulta inconclusa se conserva la candidata histórica completa.

El visor de evolución abre por defecto **Cola → Necesita criterio**. Las opciones **Control de calidad** y **Consulta opcional** permiten muestrear o explorar; no exigen revisar todo el corpus. **Cuarentena técnica** conserva problemas de identidad/integridad, distintos de un error lingüístico. El estado humano permanece separado: quitar una advertencia de la cola no aprueba la traducción. Las observaciones de esta sesión aparecen con su motivo y propuesta sin aplicar.

## Dónde quedan las decisiones

| Revisión | Archivo privado, relativo al workspace |
|---|---|
| Cuarta prueba activa | `NolvusAIT_PilotStaging/Phase4_Luna1000_R7_DiagnosticComplete/private/human_reviews.json` |
| Casos anteriores | `NolvusAIT_Evolution/20261104/private/human_reviews.json` |

No edites esos JSON manualmente. Conserva copias de seguridad privadas junto con las carpetas de las pruebas. Abrir otra prueba no sustituye las decisiones anteriores. Las aprobaciones del visor son referencias de evaluación; su incorporación a glosario, TM o producción requiere otra actuación trazable.

## CLI de revisión

La comparación nueva está en `NolvusAIT_Intelligence/20261105/private/final_comparison`: selecciónala con **Abrir otra prueba…** para ver las mismas candidatas con clasificación de riesgo. Las decisiones originales siguen en R7; no se copiaron a esta carpeta ni se crearon aprobaciones nuevas. Las etiquetas de riesgo informativas no equivalen a aprobación lingüística.

Desde PowerShell, situado en la raíz del workspace:

```powershell
$env:PYTHONPATH='src'
python -m nolvus_translator --review-folder 'NolvusAIT_PilotStaging/Phase4_Luna1000_R7_DiagnosticComplete'
```

Para la revisión anterior: `python -m nolvus_translator --review-cases` con el mismo `PYTHONPATH`.

## Desarrollo, configuración y exportación

- `pyproject.toml` declara Python >=3.11. El punto de entrada es `nolvus_translator.__main__:main`; no se requiere una versión exacta de Python/SQLite tomada de un informe antiguo.
- `workspace_data/config.json` es la configuración local; la DB principal es `workspace_data/nolvus_translator.db`. No se usan como destinos de una prueba de traducción nueva.
- La suite se ejecuta con `python -m unittest discover -s tests -t .` y `PYTHONPATH=src`. Crea datos temporales de prueba; no constituye una operación de solo lectura. No se ejecutó durante esta reorganización.
- Workbench offline: `python -m nolvus_translator.translation.intelligence_workbench --source <DB> --pilot <R7> --output <carpeta-nueva>`. Solo lee fuentes y crea un experimento aislado; rechaza reemplazar uno existente. `--benchmark-count` opcional mide ingesta sintética. No ejecuta traducciones ni jueces reales.
- Catálogo contextual offline: `python -m nolvus_translator.lore.knowledge_workbench --source <DB> --output <carpeta-nueva>`. Para enriquecer una copia ya creada: `--knowledge <sidecar> --documents-manifest <registro-json>`. Verifica hashes/permisos del registro; no descarga ni ejecuta código externo.
- Antes de una exportación futura: vista previa, evidencia editorial, respaldo, perfil clonado de MO2 y compatibilidad del ejecutable real. Las capacidades de exportación existentes no autorizan desplegar el piloto actual.
- No actives proveedores, ejecutes runners facturables ni deshabilites el candado por seguir esta guía. Cada prueba nueva exige alcance, presupuesto y autorización.

## Consolidación y validación preparada

La línea privada de trabajo continúa en `codex/contextual-coherence-20261105`, conservando `codex/technical-consolidation-20261105` y las ramas anteriores. Los planes históricos sellados no se reabren con un fingerprint de código diferente.

El diseño está en `NolvusAIT_Intelligence/20261105_Consolidation/private/next_validation_final/plan.json`; sus previews son privados. Preparar no envía solicitudes ni autoriza gasto. Si alguien empieza a guardar revisiones mientras se genera una copia, se conserva su archivo de casos y los recibos quedan disponibles en el diario. El visor en marcha no necesita cerrarse para este trabajo; los cambios de código se cargan al abrirlo nuevamente.

La preparación actualizada con nombres está en `NolvusAIT_Intelligence/20261105_Coherence/private/final/next_validation/plan.json`, sin ejecutar. En copias futuras, el panel de contexto muestra las denominaciones de referencia y posibles piezas relacionadas, indicando propuestas pendientes. Una discrepancia nueva de identidad puede pedir atención aunque exista una aprobación anterior; esa decisión permanece guardada. El barrido no regeneró tu carpeta activa ni te asigna otros 1.000 casos obligatorios. Los 25 de cuarentena ya tienen decisión lingüística; su retención técnica es independiente.

## Documentación y publicación

La preparación pareada vigente está en `NolvusAIT_Intelligence/20261105_PairedTrial/private/prepared_final_v4`; el plan y los previews son privados. `private/Validar_preparacion.bat` verifica archivos y fuentes sin enviar solicitudes. Los borradores anteriores se conservan, pero no son la selección final. La ejecución autorizada está cerrada por una discrepancia de identidad; `Validar_preparacion.bat` muestra el estado real por separado y no envía HTTP. Tus revisiones activas no se sustituyeron.

Para consultar el resultado nuevo, abre `Revisar_prueba_pareada.bat`; en el filtro «Casos» puedes seleccionar «Prueba pareada». «Necesita criterio» agrupa 47 candidatas pendientes de evaluación, no 47 errores; cambia la cola a «Cuarentena técnica» para la restante. La evaluación de IA natural figura como no ejecutada. Las variantes no se identifican durante la lectura y ambas muestran las mismas referencias disponibles para adjudicación. Guardar crea solo `private/live_20261105/review/private/human_reviews.json`; una aprobación no libera el recibo retenido. No es necesario repetir la revisión de los 1.000 casos. El informe parcial está en `NolvusAIT_Intelligence/20261105_PairedTrial/INFORME_RESULTADOS.md`.

El contexto ampliado está en `NolvusAIT_Intelligence/20261105_Documentation/private/knowledge.sqlite3`, separado del catálogo anterior y del visor activo. Las fuentes web se presentan como resúmenes del proyecto, con advertencia cuando su versión no coincide o no puede verificarse. Un enlace descubierto en el inventario no equivale a documentación validada. Las adjudicaciones de los 32 casos y la propuesta de El Corcel están en esa misma carpeta privada; no sustituyen tus decisiones ni crean aprobaciones.

Edita los cuatro documentos de `documentacion/` en el workspace local. La copia `Antigravity-MD-Skyrim/documentacion/` es el espejo documental para publicar; no se actualiza automáticamente. Mantén ambas copias iguales antes de publicar, salvo sus archivos históricos, que tienen distinto alcance.

El antiguo `subir_md_a_github.bat` no prepara ni incorpora documentos nuevos: hace pull/push de commits existentes y contiene mensajes y una ruta heredados. No se ejecutó ni se modificó en esta reorganización. La publicación debe revisar el diff del repositorio documental y excluir siempre fuentes, credenciales, DB, corpus y decisiones humanas individuales.
