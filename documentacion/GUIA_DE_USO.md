# Guía de uso y revisión local

Actualizada el **2026-11-05** a partir de los lanzadores, CLI y visor existentes. [Estado y pendientes](ESTADO_ACTUAL.md) · [Referencia técnica](REFERENCIA_TECNICA.md).

## Accesos desde la raíz del workspace

| Acceso | Qué abre |
|---|---|
| `launch_gui.bat` | Aplicación principal; puede inicializar sus carpetas de trabajo |
| `Revisar_cuarta_prueba.bat` | Visor del piloto activo R7 de 1.000 cadenas |
| `Revisar_traducciones.bat` | Casos del piloto anterior y ejemplos sintéticos |
| `Revisar_evolucion_inteligente.bat` | Copia experimental del flujo de reevaluación offline |

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

## Documentación y publicación

Edita los cuatro documentos de `documentacion/` en el workspace local. La copia `Antigravity-MD-Skyrim/documentacion/` es el espejo documental para publicar; no se actualiza automáticamente. Mantén ambas copias iguales antes de publicar, salvo sus archivos históricos, que tienen distinto alcance.

El antiguo `subir_md_a_github.bat` no prepara ni incorpora documentos nuevos: hace pull/push de commits existentes y contiene mensajes y una ruta heredados. No se ejecutó ni se modificó en esta reorganización. La publicación debe revisar el diff del repositorio documental y excluir siempre fuentes, credenciales, DB, corpus y decisiones humanas individuales.
