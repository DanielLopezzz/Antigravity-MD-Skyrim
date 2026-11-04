# Estado final y siguiente paso

Fecha del corte: 2026-11-04. Rama de código: `codex/linguistic-evolution`. Commit inicial/final: `066ae03b014f0aab1bbbcdbb4db16658e45b0327`; no se creó commit de código. La rama se creó conservando el árbol ya modificado. Hay copia de referencia de fuentes/pruebas, patch previo y hashes de archivos protegidos en los artefactos locales de la fase. No se usaron reset, stash, clean ni eliminación del trabajo anterior.

La revisión comparó 26 archivos fuente/pruebas con esa referencia inicial. No se eliminaron archivos. Se añadieron cuatro módulos propios y once métodos de regresión; la suite pasó de 69 a 80 pruebas. La ejecución final dio `Ran 80 tests in 22.496s — OK`. Los reportes y metadatos se agregaron a un paquete documental nuevo, conservando los informes históricos. El repositorio documental permanece en `main`, base `29d58f26b740bc1e34ed6487d032392dedd00ec8`, con cambios locales sin commit ni publicación de esta fase.

## Aislamiento comprobado

`PHASE_1_PAID_API_LOCK=True`. Cero llamadas a proveedores de IA, cero créditos y ninguna activación de Sol, Batch ni traducción masiva. La investigación web fue lectura documental. No se cargaron ni copiaron credenciales.

Los 57 archivos protegidos registrados al inicio conservan exactamente contenido SHA-256, tamaño y mtime: DB oficial, configuración y artefactos de staging incluidos. Se leyeron dos conjuntos de evidencia histórica y se seleccionaron casos del piloto en modo SQLite `ro&immutable=1`; no se instanciaron runners contra los pilotos. La migración v1.1 se probó solo sobre una copia sintética. El índice contextual y el benchmark utilizan DB separadas. No se modificaron juego, mods, MO2, orden de carga ni partidas.

Los resultados históricos se conservan: primer intento incompleto de 25, segundo intento 25/25 en staging y piloto transversal con 100 importadas, 23 candidatas y 2 en cuarentena. No hubo promociones a producción ni reetiquetado retroactivo. Los temporales de las pruebas fallidas no se limpiaron indiscriminadamente.

## Criterios de aceptación

| Criterio solicitado | Evidencia / resultado |
|---|---|
| Conservar comportamiento previo válido | 69 pruebas de línea base y 80 finales OK; se mantienen parsers, presupuestos, recuperación y exportación de fixtures aprobados. El cambio deliberado de aprobación mecánica está documentado. |
| Contexto llega al payload | Prueba de ContextEngine → contrato → JSON de solicitud; campos comunes por lote y específicos por ítem; rutas privadas excluidas. |
| TM distingue contextos | Pruebas de mod/misión distintos, separadores y whitespace; lectura legacy como sugerencia, sin reutilización segura automática. |
| Variantes válidas sin inconsistencias | Frases completas permitidas, límites Unicode, casos negativos y género/número explícitos; sin sufijo universal. |
| Nombre de mod conservado y mensaje traducible | Casos positivos, traducción incorrecta rechazada y conflicto de subcadenas resuelto. |
| Calidad técnica separada de lingüística | Dimensiones lingüísticas no evaluadas por defecto; candidatos pendientes; revisor/fecha y barrera de exportación. |
| Resultados de pilotos conservados | Hashes, tamaños y mtime de los 57 archivos iguales al inicio; no runners ni migraciones sobre staging real. |
| Datos representativos de prueba | 240.684 filas sintéticas en SQLite y 24 casos sintéticos/19 privados para revisión. La distribución real completa aún no se reprodujo. |
| Aislamiento y reversibilidad | Prueba de migración en copia, rollback de directorio preparado y DB local con raíces explícitas. Publicación transaccional activa y caída eléctrica pendientes. |
| Cero facturación y cambios productivos | Candado activo, pruebas con mocks, cero transporte real y verificación final de archivos protegidos. |

## Trabajo pendiente, sin declararlo resuelto

1. Completar fuentes individuales de lore para las preferencias heredadas. UESP/Imperial Library requieren consulta permitida o alternativa accesible; no hubo extracción automatizada de su contenido.
2. Revisar por personas los 24 ejemplos sintéticos y 19 casos privados, incorporando referencias solo con nombre de revisor y adjudicación de discrepancias. Después preparar un protocolo A/B ciego para una fase futura autorizada.
3. Integrar recuperación contextual de libros con segmentación, entidades, relevancia y límites medidos. Hoy existe un índice local optativo y contexto de metadatos conectado, no conocimiento exhaustivo del universo.
4. Integrar la publicación reversible preparada al exportador, con bloqueo, diario de recuperación, backups y pruebas de fallos Windows. El reemplazo activo todavía escribe directamente en el directorio final. No se afirma atomicidad ante pérdida de energía.
5. Ampliar fixtures de parsers y límites por formato, validar cabeceras/checksums LZ4 exhaustivamente, medir distribución real de tamaños y probar UI/exportación a escala. La paginación nueva no sustituye todavía todas las consultas de la GUI.
6. Preparar un sistema general de migraciones para la DB principal. La fase incorpora una extensión aditiva del almacén de intercambio y claves TM versionadas, pero no una migración global de producción.

El siguiente trabajo recomendado es la revisión editorial del corpus y de las entradas de mayor riesgo, seguida de pruebas offline de recuperación contextual y exportación. Esta fase no autoriza otra traducción facturable. Una prueba futura debe conservar staging histórico, usar una copia nueva y tener autorización y presupuesto explícitos.
