# Benchmark, costes y pruebas

## Medición sintética de escala

Se ejecutó `benchmark_scale.py` sobre una DB temporal propia, con 240,684 cadenas sintéticas de ocho tipos de registro y cien mods/plugins ficticios. Las inserciones usan bloques de 2.500 y el recorrido páginas de 1.000; el proceso vuelve a abrir la DB y restaura el cursor desde un checkpoint a mitad de recorrido.

| Métrica medida | Resultado |
|---|---:|
| Filas verificadas, sin omisión/repetición | 240684 |
| Inserción e inicialización | 10.452 s |
| Recorrido por cursor e identidad contextual | 14.3 s |
| Tiempo total, incluidas comprobaciones | 27.506 s |
| Pico de memoria del proceso Windows | 39157760 bytes |
| Pico de asignaciones Python observado por tracemalloc | 4362078 bytes |
| SQLite generado | 113582080 bytes |
| Integridad SQLite | ok |
| Reanudación por checkpoint | verificada |

El plan de consulta usa `idx_strings_cursor (snapshot_id=? AND id>?)`. El benchmark se hizo con Python 3.13.7 en Windows, con tracemalloc activado. Son mediciones de este equipo y distribución sintética; no un SLA ni una prueba de todos los formatos reales. No se midió escaneo Nolvus, UI prolongada, reconstrucción de todos los formatos ni exportación completa de 240k cadenas. La memoria de un plan de exportación completo aún necesita medición.

## Eficiencia de contexto y coste

Se compararon offline dos representaciones del mismo lote sintético de 25 etiquetas UI. No se conservaron los textos del corpus real en estos artefactos públicos. El contexto repetido por ítem ocupaba 10204 bytes; compartiéndolo por lote ocupa 7415 bytes. El payload anterior sin contexto ocupaba 6704 bytes. Por tanto, deduplicar contexto reduce redundancia, pero el nuevo contexto aporta información y no es gratuito.

Las unidades heurísticas de entrada —instrucciones y payload, sin equipararlas a uso facturado— pasan de 2751 a 3121. Compartir el contexto ahorra 797 unidades frente a repetirlo. El estimador existente utiliza longitud de caracteres, placeholders y saltos de línea; no es un tokenizador BPE medido ni garantiza no subestimar. Se conserva la reserva presupuestaria y el cálculo sobre el payload final completo. No se alteraron tarifas, límites, modelos ni políticas de gasto.

El piloto histórico de 125 cadenas consumió US$0.011345, con 24.890 tokens de entrada y 17.712 de salida. Una extrapolación puramente aritmética de aquel coste a 240.684 cadenas da US$21.84447984; no es un presupuesto, una cota máxima ni una proyección actualizada validada. Los libros, contexto nuevo, razonamiento, reintentos, cuarentenas y revisión pueden cambiar esa cifra. El nuevo presupuesto requiere un piloto representativo autorizado en otra fase.

## Regresión e incidencias

Línea base al iniciar esta fase: 69 pruebas OK, 13,643 s. La suite final y su resultado se consignan en [05_ESTADO_FINAL.md](05_ESTADO_FINAL.md). Se añadieron 11 métodos de regresión que cubren contexto real del payload, compatibilidad v1, migración en copia, TM contextual, variantes y nombres, aprobación humana/exportabilidad, cursor, descompresión limitada, índice local y publicación reversible.

La primera suite tras separar aprobación editorial encontró expectativas históricas de aprobación/propagación automática. No se relajaron los controles: se cambiaron las expectativas para exigir pendientes, TM vacía y ausencia de propagación. El preflight efímero ahora comprueba 25 candidatas pendientes y cero entradas TM. La prueba integral de exportación realiza una decisión editorial explícita sobre sus referencias oficiales sintéticas. También se corrigieron conexiones SQLite sin cerrar en el índice local detectadas por las pruebas de limpieza Windows y una configuración incompleta del presupuesto del fixture nuevo.

El nombre de mod se prueba de forma positiva y negativa; el test de hash modifica bytes interiores de un archivo mayor a 16 MiB conservando tamaño y mtime. El test de migración conserva hash del original; solo abre la copia para actualizarla. Los tests siguen usando mocks y DB temporales. Ningún fallo se resolvió activando API, subiendo topes o aceptando placeholders dañados.
