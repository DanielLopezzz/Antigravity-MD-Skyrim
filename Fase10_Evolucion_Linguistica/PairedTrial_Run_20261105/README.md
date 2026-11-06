# Prueba pareada real — resultado parcial, 2026-11-05

La API se ejecutó con autorización explícita y un tope de **US$0,35**. Se recibieron las 48 traducciones previstas; **47 pasaron la validación mecánica y una quedó retenida por un hash de origen incorrecto**. Se cumplió la parada crítica del plan sellado. La prueba no completó el juez ciego sobre traducciones naturales ni la posedición; no constituye un éxito integral ni una medición de mejora semántica.

## Ejecución y consumo

| Medida | Resultado |
|---|---|
| Diseño | 24 fuentes reales × dos contextos; cuatro por cada uno de seis perfiles; 22 orígenes |
| Solicitudes de traducción | 42; las cuatro piezas de la familia enana viajaron juntas en cada variante |
| Controles del juez | Ocho consultas reales: cuatro errores sembrados detectados y cuatro aceptables con `no_issue` |
| Solicitudes HTTP únicas | 50; 50 ID de respuesta distintos; cero reenvíos automáticos |
| Traducciones recibidas / válidas / retenidas | 48 / 47 / 1; validación mecánica, no aprobación editorial |
| Revisión natural / posedición / recomprobación | 0 / 0 / 0: no se alcanzaron esas etapas |
| Tokens reportados | Entrada: 88.206; salida: 19.442, incluidos los tokens internos reportados |
| Desglose de entrada | 46.372 de lectura de caché, 35.043 de escritura y 6.791 restantes |
| Consumo conciliado local | **US$0,01524421**; reserva pendiente: 0; presupuesto incierto: 0 |
| Estado de protección | API Lock activo; parada persistida; cinco entradas congeladas conservan sus hashes |
| Pruebas offline de cierre | **276/276**, más la repetición focalizada de las 14 pruebas del ejecutor |

El cálculo usa los tokens reportados y las tarifas estándar verificadas el 2026-11-05 para `gpt-6-luna`: entrada US$0,10, lectura de caché US$0,01, escritura US$0,125 y salida US$0,50 por millón. No certifica la factura del proveedor. El recálculo decimal independiente da US$0,01524422: la diferencia de US$0,00000001 corresponde al redondeo por llamada del libro existente. [Modelo y tarifas oficiales](https://developers.openai.com/api/docs/models/gpt-6-luna), [precios oficiales](https://developers.openai.com/api/docs/pricing).

La reserva por nueva solicitud se corrigió para contemplar escritura de caché y margen del 15 %, manteniendo el cap de US$0,35. La estimación cerrada de preparación no se sobrescribió: su cota global no contemplaba escritura de caché. Con la reserva corregida, 106 solicitudes máximas costarían US$0,3822784 en el peor caso reservado; no se garantiza ejecutar ese máximo dentro del cap. El control económico detiene el trabajo que no quepa; el gasto real de esta ejecución quedó ampliamente por debajo.

## Incidencias y correcciones funcionales

1. **Interrupción local a las 29 solicitudes.** El decodificador nuevo calculaba el hash sobre texto bruto; el contrato heredado lo calcula sobre `text.strip()`. Un libro con saltos finales activó esa incompatibilidad. Se corrigió el decodificador y se mantuvo la comprobación exacta de la envoltura del texto bruto y sus controles. Se reanudó leyendo los 29 recibos conservados, sin volver a enviarlos.
2. **Parada crítica real al completar las traducciones.** La respuesta `t41`, variante ampliada de un mensaje de Unslaad, conservó UID y secuencia pero devolvió un hash diferente, también de 64 caracteres. No fue un error de longitud ni del decodificador. El recibo se conserva intacto en cuanto a texto/identidad; no se corrigió su hash ni se aceptó su candidata. La retención afecta a una solicitud de una cadena. Su uso se concilió porque el proveedor sí ejecutó la llamada.
3. **Prevención verificada offline.** El proveedor ahora restringe secuencia, UID y hash a sus valores exactos mediante enumeraciones; los lotes múltiples permiten solo las tuplas de identidad suministradas. La validación local sigue exigiendo cardinalidad, orden y correspondencia. La idempotencia incorpora instrucciones y esquema para distinguir una política corregida. La compatibilidad del nuevo esquema con una llamada real sigue pendiente. [Contrato de Structured Outputs](https://developers.openai.com/api/docs/guides/structured-outputs).
4. **Persistencia y conciliación.** El ejecutor guarda solicitud y recibo antes de interpretar, verifica hashes al reutilizar, no reenvía estados inciertos y restaura el lock en `finally`. La parada crítica queda en disco: permite leer recibos, pero bloquea nuevos envíos incluso tras reiniciar. Se agregó conciliación de caché con ajustes idempotentes auditados; ningún libro económico de pilotos históricos fue reescrito.
5. **Visor aislado.** Se exportaron 48 candidatas con orden e identificadores ciegos; 47 en «Necesita criterio» y una en «Cuarentena técnica». La cola indica evaluación pendiente, no 47 errores. La asociación de variantes permanece en un archivo privado separado. El visor muestra documentación de referencia y el estado de reevaluación no ejecutada.

La suite cubre rechazo de identidades/controles alterados, lectura sin reenvío, recibos manipulados, parada persistida, timeout, rechazo HTTP, uso desconocido, modelo/tarifa incompatibles y conciliación idempotente. El primer recorrido completo tras cambiar el esquema detectó una prueba antigua que suponía un único esquema genérico; se adaptó a las variantes vinculadas y el recorrido final pasó completo.

## Lectura lingüística preliminar de las parejas

Es una lectura del analista, sin etiquetas humanas nuevas ni juez ciego natural. Cada caso tiene una sola generación por variante; una diferencia no demuestra por sí sola causalidad del contexto.

- **Coherencia de conjunto:** ambas variantes produjeron «Guanteletes enanos», «Casco enano», «Botas enanas» y «Armadura enana». La concordancia varía, pero comparten la denominación del conjunto. Aquí no hay ventaja observada de una variante.
- **Referencias de lugares:** la variante ampliada usó «Lago Pino Claro», «Mina Panza Roja» y «Fortaleza de Piedra Baja», coincidiendo con las denominaciones localizadas recuperadas. La anterior dejó dos nombres parcialmente ingleses y usó «Fortaleza de Piedra Inferior». Es una señal favorable de consistencia con referencias; falta contrastar el recorrido completo diálogo/objetivo/HUD.
- **Naturalidad y mecánicas:** varias frases son fluidas en ambas variantes; cifras y porcentajes de las cuatro descripciones mecánicas se conservaron. Esto no prueba equivalencia semántica completa ni permite medir falsas aprobaciones.
- **Regresiones o pendientes:** la variante ampliada dejó `Brittleshin Pass`, `Northwatch Keep` y `Cathedral of St. Martin` en inglés. Deben vincularse a sus registros localizados antes de imponer una forma. Una carta usa «Querido/a»; conviene contrastar el criterio de estilo sin inventar género. En otro libro, «me veré obligado a volver a trazar» resulta incompleto frente a «volver a trazar mis planes» de la otra variante; el `redraw` original es ambiguo y exige contexto. Estas observaciones están vinculadas a las candidatas en el visor, sin editarlas ni aprobarlas.

El contexto ampliado ayuda en denominaciones respaldadas, pero **no garantiza una traducción superior en todos los casos**. Los ocho controles favorables son una comprobación mínima del evaluador; no calibran su precisión sobre mods reales ni demuestran reducción segura de revisión humana. Se conserva cerrada la aprobación automática.

## Evidencia, alcance y continuación

El sello de preparación sigue siendo `80aa5cb3732cd532ac5365fdf7fba12e6960b7a68e678dec725e839809270d1a`. Se conservaron los previews y sus estimaciones históricas; el estado real se registra por separado. Código privado de cierre: rama `codex/contextual-coherence-20261105`, commit `3e4e9740fd03962425f6731b55061e17cfdc0cf0`. Incluye correcciones posteriores a las solicitudes selladas, verificadas offline.

Evidencia local bajo `NolvusAIT_Intelligence/20261105_PairedTrial/private/live_20261105`: diario y libro económico, solicitudes/respuestas con hashes, resultados, inventario conciliado, parejas para análisis, correspondencia privada de variantes y visor separado. Algunas copias de recibos tienen redacciones preventivas en campos opacos; se conservan el hash del cuerpo HTTP recibido y el hash de la copia guardada. No se publica corpus, respuestas completas, razonamiento opaco, credenciales ni código fuente.

Abre **`Revisar_prueba_pareada.bat`** en la raíz local. Guardar crea únicamente el overlay privado de esta muestra. Una aprobación lingüística de la retenida no arregla la identidad ni habilita su importación. La copia activa de 1.000 casos y sus 41 decisiones anteriores permanecen sin cambios; tampoco se escribieron TM, glosario, producción o Nolvus.

Queda preparada offline solo la solicitud afectada con el esquema corregido. La ejecución actual está cerrada por su condición sellada «Any critical identity or markup failure». Una continuación requiere un alcance nuevo y acotado: reutilizar las 47 candidatas válidas, resolver la solicitud retenida y completar revisión ciega y, cuando proceda, posedición/recomprobación. No corresponde repetir las 48 traducciones ni aumentar la carga todavía. No se enviaron solicitudes después de la parada crítica.
