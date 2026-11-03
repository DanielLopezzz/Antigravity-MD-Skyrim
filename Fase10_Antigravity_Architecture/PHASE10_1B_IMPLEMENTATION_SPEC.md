# Especificación técnica de implementación — Fase 10.1-B

**Proyecto:** Nolvus AI Translator  
**Base auditada:** `v0.6.0`, esquema SQLite `1.1.0`  
**Auditoría de origen:** `AUDIT-10.1A-20261103-113100`  
**Estrategia prioritaria:** Antigravity mediante intercambio supervisado de archivos, sin API externa  
**Estado:** Especificación documental; **implementación no iniciada ni autorizada**  
**Fecha:** 2026-11-03

---

## 1. Propósito y alcance

La Fase 10.1-B implementará una capa segura para preparar lotes de traducción, intercambiarlos manualmente/supervisadamente con Google Antigravity y validar localmente las respuestas.

La implementación debe reutilizar la infraestructura existente de Nolvus AI Translator y conservar la vía API futura. No debe reconstruir el núcleo, los parsers, el glosario, la memoria de traducción ni los validadores que ya existen.

### Objetivos de 10.1-B

1. Definir un contrato versionado de intercambio JSON/JSONL.
2. Preparar lotes en una carpeta física independiente del código del traductor y de sus datos.
3. Proteger los placeholders y mantener su mapa exclusivamente local.
4. Recibir resultados JSONL y verificar la integridad del lote.
5. Reutilizar el protector, el validador, el glosario y la política ES-419 actuales.
6. Aislar registros defectuosos y generar micro-lotes de reintento.
7. Prevenir la doble ingesta.
8. Operar exclusivamente contra una base de datos de pruebas durante esta fase.
9. Proporcionar una CLI local para preparar, validar y consultar el estado.
10. Mantener intactas las interfaces y el bloqueo de proveedores API de pago.

### Fuera de alcance

- Traducción masiva.
- Ejecución del benchmark de 300 cadenas.
- Activación de APIs o facturación.
- Descarga de modelos locales.
- Modificación de STOCK GAME, MO2, mods originales, BSA, plugins o partidas.
- Escritura en la base de datos productiva.
- Fusión de cambios de código a `main` sin revisión y autorización.
- Correcciones globales de parsers/exportadores de Fase 9 como parte del mismo conjunto de cambios. Esas correcciones se gestionan como un paquete separado.

---

## 2. Decisión arquitectónica principal

### 2.1. Antigravity no es un proveedor API convencional

La interfaz actual `BaseTranslationProvider.translate_batch(...)` es síncrona. Antigravity, cuando se utiliza desde el selector de modelos de su aplicación, no entrega a este proyecto un endpoint público/documentado que permita invocar de forma programática esos modelos con la cuota de suscripción.

Por ello, **no se debe implementar un falso proveedor que simule una llamada API síncrona a Antigravity**.

La recomendación es crear un coordinador de intercambio supervisado, por ejemplo `AntigravityExchangeService`, que gestione:

- Preparación de lote.
- Exportación de archivos.
- Registro del estado local.
- Recepción de la salida escrita por el agente.
- Validación mediante Gatekeeper.
- Cuarentena y reintentos.
- Entrega de resultados validados al flujo local.

La interfaz `BaseTranslationProvider` y los stubs API se conservan sin cambios funcionales en esta fase. El modo Antigravity se representa como un **modo de ejecución supervisado por archivos**, no como una llamada HTTP o API.

Si posteriormente se necesita un contrato común de ejecución, se diseñará una abstracción de nivel superior que permita operaciones síncronas y asíncronas sin forzar el intercambio de archivos dentro de `translate_batch()`.

### 2.2. Separación física de workspaces

El agente solo podrá trabajar en una carpeta independiente, por ejemplo:

```text
C:\NolvusAIT_Exchange\job_<job_id>\
├── 00_PROTOCOL.md
├── 01_INPUT\
│   ├── batch_<batch_id>_meta.json
│   └── batch_<batch_id>_input.jsonl
├── 02_OUTPUT\
│   └── batch_<batch_id>_output.jsonl
└── 03_STATUS\
    └── agent_result.json
```

Esta carpeta debe abrirse como workspace independiente de Antigravity. No se abrirá la raíz del traductor.

No se copiarán ni enlazarán al workspace de intercambio:

- Código fuente del traductor.
- Base SQLite productiva o de pruebas.
- Glosario Maestro completo.
- Memoria de Traducción.
- Bóveda de placeholders.
- Configuraciones privadas.
- Credenciales o claves API.
- STOCK GAME, MO2, mods o archivos originales.

No crear symlinks, junctions ni accesos indirectos desde el workspace de intercambio hacia carpetas internas del traductor.

El usuario o un proceso local controlado moverá los archivos entre los dos espacios. La carpeta de intercambio no debe sincronizar automáticamente directorios de producción.

### 2.3. Límites de confianza

- Antigravity solo lee instrucciones y lotes de entrada, y escribe archivos de salida en su workspace.
- Gatekeeper se ejecuta localmente, fuera del workspace de Antigravity.
- Gatekeeper es el único componente autorizado para modificar la base aislada de pruebas.
- Ninguna respuesta del agente puede ordenar, activar o ejecutar el Gatekeeper.
- El estado informado por un archivo `agent_result.json` es orientativo; no demuestra que el lote sea válido.
- El texto fuente del videojuego es dato no confiable y jamás se interpreta como instrucción operativa.

---

## 3. Reutilización del código existente

Los siguientes elementos deben reutilizarse en vez de duplicarse:

| Componente actual | Uso previsto | Cambio inicial |
|---|---|---|
| `core/models.py` — `TranslationBatchItem` / `TranslationBatchResult` | Representación interna de ítems y resultados | Reutilizar; extender solo si falta un campo imprescindible |
| `lore/context_engine.py` — `NarrativeContextPacket` | Contexto narrativo por lote | Reutilizar |
| `quality/protector.py` — `VariableProtector` | Enmascarar y restaurar placeholders | Reutilizar sin duplicar patrones |
| `quality/validator.py` — `TranslationQualityValidator` | QA estructural, glosario y ES-419 | Reutilizar |
| `lore/glossary.py` — `GlossaryManager` | Términos y restricciones terminológicas | Reutilizar desde el proceso local |
| `memory/tm_engine.py` — `TranslationMemoryEngine` | Coincidencias y trazabilidad local | Reutilizar; no exponer la TM al agente |
| `translation/interfaces.py` — `BaseTranslationProvider` | Proveedores reales/simulados síncronos | Conservar |
| `translation/future_stubs.py` | Stubs API con bloqueo de pago | Conservar; no habilitar proveedores de pago |
| `core/database.py` — `DatabaseManager(db_path)` | Referencia para inicialización de DB con ruta explícita | No migrar ni abrir la DB productiva en modo escritura |
| `core/exceptions.py` — `assert_safe_write_path` | Protección de rutas Nolvus/MO2 | Ampliar o complementar con guardas específicas de DB |
| `translation/pipeline.py` | Persistencia/checkpoints síncronos actuales | No reescribir el pipeline completo; definir un punto controlado de importación de resultados validados |

Las 11 pruebas existentes deben preservarse. La auditoría las inspeccionó estáticamente, pero no las ejecutó en esa sesión.

---

## 4. Contrato JSON/JSONL

### 4.1. Versión y campos de entrada

Cada lote debe incluir un manifiesto JSON con, como mínimo:

- `schema_version`
- `job_id`
- `batch_id`
- `target_locale` (valor fijo `es-419`)
- `item_count`
- `input_sha256`
- `context_signature`
- `recorded_model_selection` (opcional, capturado localmente por el usuario; no confiado a la respuesta del agente)
- Perfil/instrucciones ES-419 aplicables.

Cada registro JSONL de entrada debe contener:

- `schema_version`
- `job_id`
- `batch_id`
- `seq_id`
- `string_uid`
- `source_sha256`
- `context_signature`
- `content_domain`
- `record_type`
- `field_name`
- `source_masked`
- `glossary_context` estrictamente limitado a los términos relevantes que puedan compartirse.

No enviar el mapa original de placeholders ni valores internos de variables. Cuando se requiera información gramatical, usar pistas generales no sensibles (por ejemplo, nombre propio, cantidad o duración).

### 4.2. Campos de salida

Cada línea de salida JSONL debe incluir:

- `schema_version`
- `job_id`
- `batch_id`
- `seq_id`
- `string_uid`
- `source_sha256`
- `translated_masked_text`
- `translator_note` opcional, nunca mezclada con el texto traducido.

No confiar en un `model_id` declarado por el agente. El modelo seleccionado debe registrarse localmente por el usuario/sistema supervisor para que el contenido de salida no pueda falsear su procedencia.

### 4.3. Reglas de serialización

- UTF-8, un objeto JSON por línea.
- Sin encabezados Markdown, comentarios ni cercas de código.
- Rechazar líneas vacías, JSON malformado, claves desconocidas si el esquema es estricto y claves JSON duplicadas dentro de un objeto.
- Validar `job_id`, `batch_id`, `seq_id`, `string_uid` y `source_sha256` contra el manifiesto local.
- Rechazar identificadores inesperados, duplicados o faltantes.
- El orden físico de líneas no será la única fuente de identidad: `seq_id` y `string_uid` se validan y luego se ordenan localmente.
- Definir límites configurables de tamaño del archivo y de cada registro, sin hardcodear límites de cuota de modelos.

### 4.4. Integridad e idempotencia

Calcular SHA-256 de:

- Manifiesto.
- JSONL de entrada.
- JSONL de salida.

El ledger local debe registrar `job_id`, `batch_id`, hashes de entrada/salida, fecha, modelo declarado localmente, estado y resultado de validación.

La misma salida no debe poder ingerirse dos veces. La identidad del lote y el `output_sha256` deben actuar como claves de idempotencia.

---

## 5. Bóveda local de placeholders

`VariableProtector` ya existe, pero su mapa de tokens vive actualmente en memoria durante el pipeline.

Para el intercambio asíncrono, el proceso local debe poder recuperar el mapa después de cerrar/reiniciar Antigravity.

Requisitos:

1. Proteger el texto antes de generar el archivo de entrada.
2. Persistir `ProtectionEnvelope` en una bóveda local privada, asociada a `job_id`, `batch_id` y `string_uid`.
3. Guardar la bóveda fuera del workspace de intercambio.
4. No incluir el mapa en manifiestos, JSONL de entrada, JSONL de salida ni logs públicos.
5. Restaurar tokens únicamente después de validar las respuestas recibidas.
6. Rechazar omisiones, invenciones, duplicados o conteos incorrectos de placeholders.
7. Mantener el texto original y la correspondencia técnica en el espacio local.

Durante 10.1-B la bóveda y su DB deben pertenecer exclusivamente al entorno aislado de pruebas. No añadir columnas ni tablas a la DB productiva.

---

## 6. Almacenamiento aislado y protección de la base productiva

La auditoría verificó que `DatabaseManager(db_path)` acepta rutas configurables, pero `assert_safe_write_path()` no protege expresamente la ruta de la DB productiva ni sus archivos auxiliares.

### Decisión

Crear un almacenamiento independiente para estado de intercambio, por ejemplo `exchange_state_test.sqlite3`, dentro de un workspace local de pruebas separado del workspace Antigravity.

No modificar el esquema productivo `1.1.0` durante la primera implementación.

El almacenamiento aislado puede registrar las entidades:

- `exchange_jobs`
- `exchange_batches`
- `exchange_items`
- `placeholder_vault`
- `quarantine_items`
- `ingestion_ledger`
- `model_attempts`

No se deben crear todas las tablas si el diseño final demuestra que un conjunto menor cubre las necesidades; justificar la decisión antes de implementar.

### Guardas de seguridad

- Resolver rutas absolutas/canónicas antes de abrir una DB en escritura.
- Rechazar la ruta exacta de la DB productiva y sus archivos auxiliares SQLite (`-wal`, `-shm`).
- Revisar symlinks/junctions para impedir que una ruta aparentemente temporal apunte a producción.
- Permitir escritura únicamente dentro de las raíces explícitas de pruebas.
- En caso de duda sobre la identidad de la DB, fallar de forma cerrada.
- Utilizar transacciones para la ingesta validada.
- Nunca copiar/migrar la DB productiva ni ejecutar un esquema sobre ella durante 10.1-B.

Para los primeros tests se prefieren fixtures sintéticos. La extracción de muestras reales se realizará posteriormente bajo una autorización específica y en entorno local privado.

---

## 7. Gatekeeper local

Crear un validador local independiente, propuesto en:

`src/nolvus_translator/quality/gatekeeper.py`

Puede dividirse en módulos auxiliares si su tamaño lo justifica.

### 7.1. Puertas deterministas

El Gatekeeper debe comprobar:

1. Sintaxis JSONL y esquema por registro.
2. Correspondencia exacta de `job_id` y `batch_id`.
3. Igualdad exacta del conjunto esperado y recibido de `seq_id`.
4. Igualdad exacta del conjunto esperado y recibido de `string_uid`.
5. Ausencia de `seq_id` y `string_uid` duplicados.
6. Hashes de entrada y salida.
7. Coincidencia de `source_sha256`.
8. Integridad de placeholders tras restauración local.
9. Balance/preservación de etiquetas y claves no traducibles.
10. Cumplimiento del Glosario Maestro.
11. Validaciones ES-419.
12. Texto vacío, contenido extra o residuo de idioma de origen según las reglas actuales.

Reutilizar `VariableProtector.restore_and_verify()` y `TranslationQualityValidator.validate()`. No reimplementar sus regex o glosarios en un segundo validador independiente.

### 7.2. Política de aceptación

No importar parcialmente un lote a SQLite.

Si un lote tiene 25 registros y dos fallan:

- Los 23 registros válidos pueden conservarse como candidatos locales.
- Los dos inválidos se registran en cuarentena.
- El lote queda en estado `RETRY_REQUIRED`.
- No se escribe el lote en la DB de resultados de prueba hasta completar el conjunto esperado y superar todas las puertas.
- El reintento se limita a los UIDs pendientes/rechazados.
- Al integrar los resultados corregidos, validar de nuevo el conjunto completo y sus hashes.

### 7.3. Recuperación de archivos truncados

No confiar en un marcador `.done` producido por el agente como prueba de integridad.

El Gatekeeper debe:

- Parsear cada línea individualmente.
- Recuperar únicamente registros completos que superen esquema e identidad.
- Poner en cuarentena una última línea parcial/malformada.
- Determinar los UIDs ausentes y generar un paquete de reintento.
- Nunca afirmar que JSONL garantiza una recuperación del 100 %.

---

## 8. Máquina de estados del intercambio

Se propone esta secuencia de estados:

`CREATED` → `EXPORTED` → `AWAITING_AGENT` → `OUTPUT_RECEIVED` → `VALIDATING` → `VALIDATED` → `IMPORTED_TO_TEST_DB`

Estados alternativos:

- `RETRY_REQUIRED`
- `QUARANTINED`
- `FAILED`
- `CANCELLED`

Cada transición debe registrar fecha, actor, hashes y motivo. No permitir transiciones hacia atrás que sobrescriban resultados aprobados; una nueva ejecución debe registrar un intento nuevo.

---

## 9. Cuarentena, reintentos y trazabilidad

La cuarentena debe guardar únicamente:

- Job/batch UID.
- String UID.
- Seq ID.
- Código de error.
- Tipo de validación fallida.
- Hash de entrada/salida relacionado.
- Número de intento.
- Estado de resolución.

No debe duplicar más texto original del necesario.

El generador de micro-lotes de reintento debe producir un nuevo JSONL con únicamente los UIDs pendientes, pero conservar la relación con el lote original.

Límites:

- Número máximo de reintentos configurable.
- No reintentar automáticamente indefinidamente.
- No cambiar a un modelo más caro o API automáticamente.
- No alterar la instrucción/glosario después de observar resultados sin registrar una nueva versión del contexto.
- Un cambio de contexto, glosario o texto fuente debe producir un nuevo hash de solicitud.

---

## 10. CLI y límites de integración con el pipeline actual

El proyecto tiene un entrypoint `__main__.py` que actualmente abre la GUI Tkinter y no dispone de subcomandos headless.

Se propone añadir `src/nolvus_translator/cli.py` y adaptar `__main__.py` para conservar el arranque actual de la GUI cuando no se solicite un subcomando.

Comandos previstos (nombres provisionales; confirmar contrato antes de codificar):

- `exchange init-test-store`
- `exchange export-batch`
- `exchange inspect-job`
- `exchange validate-output`
- `exchange retry-quarantine`
- `exchange report`

No es necesario construir todos los comandos si la primera versión puede cubrir el flujo con menos operaciones bien delimitadas.

### Integración con TranslationPipeline

La integración debe evitar que el pipeline síncrono invoque o espere una operación manual de Antigravity dentro de `translate_batch()`.

Primera opción:

- El coordinador prepara un lote con los modelos/contexto existentes.
- La interacción con Antigravity se produce fuera del proceso del traductor.
- El Gatekeeper valida la salida.
- Un método explícito de importación recibe únicamente resultados validados y los convierte en `TranslationBatchResult`/entidades existentes.
- La actualización ocurre solo en la DB aislada durante esta fase.

Antes de añadir un método a `pipeline.py`, identificar el punto mínimo de integración que preserve la lógica existente de TM, revisión, checkpoints y trazabilidad.

---

## 11. Archivos previstos para la implementación

La siguiente lista es una hipótesis de impacto derivada de la auditoría estática. Debe confirmarse en una revisión del diff antes de implementar.

### Archivos nuevos previstos

| Archivo | Responsabilidad |
|---|---|
| `src/nolvus_translator/translation/exchange_contract.py` | Modelos Pydantic y validación de esquema JSON/JSONL |
| `src/nolvus_translator/translation/exchange_service.py` | Coordinador de jobs, exportación y recepción supervisada |
| `src/nolvus_translator/translation/exchange_store.py` | Estado local aislado, ledger, bóveda y cuarentena |
| `src/nolvus_translator/quality/gatekeeper.py` | Puertas deterministas de validación |
| `src/nolvus_translator/cli.py` | Subcomandos headless |
| `tests/test_exchange_contract.py` | Contrato, serialización e identidad |
| `tests/test_exchange_store.py` | Aislamiento, ledger, vault e idempotencia |
| `tests/test_gatekeeper.py` | Validaciones, cuarentena y reintentos |
| `tests/test_exchange_cli.py` | Comportamiento de la CLI en entorno temporal |

### Archivos existentes que podrían requerir cambios

| Archivo | Posible cambio | Restricción |
|---|---|---|
| `src/nolvus_translator/__main__.py` | Enrutamiento GUI/CLI | No romper el arranque de la GUI |
| `src/nolvus_translator/core/exceptions.py` | Excepciones/guardas de DB y rutas | Fail-closed; sin ampliar raíces de escritura de producción |
| `src/nolvus_translator/core/models.py` | Campos de lote, solo si no pueden componerse externamente | Reutilizar modelos antes de duplicarlos |
| `src/nolvus_translator/core/config.py` | Configuración explícita del modo de intercambio | No desactivar `PHASE_1_PAID_API_LOCK` ni habilitar APIs |
| `src/nolvus_translator/translation/pipeline.py` | Punto mínimo de importación de resultados validados | No reescribir el pipeline síncrono |
| `src/nolvus_translator/quality/validator.py` | Solo si falta una interfaz reutilizable | No duplicar ni debilitar validaciones |
| `pyproject.toml` | Entry point CLI si resulta necesario | No añadir dependencias sin justificación |

No se autoriza modificar todos estos archivos automáticamente. El inventario final debe listar únicamente los archivos que el diseño detallado demuestre necesarios.

---

## 12. Pruebas requeridas

Todas las pruebas nuevas deben trabajar con directorios temporales y DBs de prueba.

### Contrato y archivos

- Serialización/deserialización round-trip.
- UTF-8 y JSONL de varias líneas.
- Rechazo de JSON malformado, claves duplicadas, líneas vacías y campos inválidos.
- Detección de cambios de hash.
- Detección de batch/job equivocado.

### Identidad e integridad

- UID faltante.
- UID extra/inventado.
- UID duplicado.
- Seq ID duplicado o fuera del conjunto.
- Desalineación entre UID y Seq ID.
- Hash de texto fuente distinto.
- Salida antigua de otro job.

### Protección y QA

- Placeholder omitido.
- Placeholder inventado.
- Placeholder duplicado.
- Token residual.
- Etiqueta desequilibrada.
- Clave MCM alterada.
- Término obligatorio del glosario incumplido.
- Formas peninsulares prohibidas.
- Traducción vacía o con texto extra.

### Recuperación

- Archivo truncado en la última línea.
- Interrupción después de varios registros completos.
- Reintento de solo UIDs pendientes.
- Reconstrucción del lote completo sin doble ingesta.
- Reimportación del mismo `output_sha256` bloqueada.

### Seguridad del entorno

- Intento de abrir la DB productiva en escritura durante smoke/benchmark rechazado.
- Rechazo de rutas que escapan del workspace autorizado.
- Rechazo de symlink/junction que apunta fuera del espacio permitido.
- El agente no recibe rutas ni contenidos de DB, código, glosario completo o vault.
- No existe invocación de API en modo Antigravity.
- El texto del juego se mantiene dentro del campo de datos y no se ejecuta como instrucción.

### Compatibilidad

- Las 11 pruebas existentes deben ejecutarse y pasar en un entorno temporal cuando se autorice la fase de pruebas.
- No eliminar ni relajar tests existentes para hacer pasar nuevas pruebas.
- Registrar por separado resultados históricos y ejecuciones nuevas.

---

## 13. Puertas de aceptación

No avanzar a la siguiente puerta hasta cerrar la anterior.

| Puerta | Resultado exigido |
|---|---|
| B1 — Contrato | Esquema versionado, serialización determinista, hashes y round-trip correctos |
| B2 — Aislamiento | Workspace de Antigravity físicamente separado; DB productiva bloqueada |
| B3 — Protección | Mapa de tokens local; placeholders/etiquetas restaurados y validados |
| B4 — Gatekeeper | Identidad exacta, duplicados, hashes, glosario y ES-419 validados |
| B5 — Recuperación | Cuarentena, reintentos limitados e idempotencia demostrados |
| B6 — CLI | Comandos locales funcionales; GUI existente sigue arrancando igual |
| B7 — Regresión | 11 pruebas existentes y todas las pruebas nuevas pasan en entorno temporal |

Al terminar B1–B7, detenerse para revisión del usuario. No ejecutar aún los modelos reales.

### Puerta posterior: smoke test

Solo después de revisar y autorizar:

- Confirmar los modelos realmente disponibles en Antigravity.
- Comprobar manualmente `AI Credit Overages = Never`.
- Utilizar un lote de 25 cadenas en el workspace separado.
- Registrar el modelo seleccionado, tiempos, cuota observada e intervenciones.
- Usar únicamente DB aislada.
- Revisar el informe antes de preparar el benchmark de 300 cadenas.

---

## 14. Deuda técnica de cobertura de Fase 9 — paquete separado

La auditoría identificó hallazgos que no deben mezclarse sin control con la implementación del intercambio Antigravity:

1. **PEX dentro de BSA:** revisar/eliminar la condición que omite archivos cuando `len(vfs_out.files) > 500`, con un algoritmo seguro y pruebas sobre un VFS sintético grande.
2. **JSON_OTHER:** conectar el parser existente con `ProjectService.scan_and_extract()`, usando una allowlist explícita para el Subgrupo 4A narrativo y excluyendo el Subgrupo 4B técnico/presets.
3. **Reconstrucción/exportación:** añadir soporte probado solo para los formatos autorizados, incluyendo MCM JSON, INI de localización, XML y JSON_OTHER narrativo.
4. **3DNPC:** crear una normalización dirigida y reversible para los 38 casos de codificación reportados; no aplicar sustituciones globales a todos los archivos.

Estas correcciones deben tener pruebas y una revisión independiente. Son necesarias antes de declarar cobertura completa del volumen de Fase 9 y antes de una traducción productiva global, pero no deben retrasar el diseño del contrato de intercambio ni mezclarse en el primer PR del Gatekeeper.

---

## 15. Riesgos y decisiones no negociables

- No abrir la raíz del traductor como workspace de Antigravity.
- No exponer al agente el código fuente, la DB, el glosario completo ni la bóveda de placeholders.
- No confiar en `.done` o en metadatos escritos por el agente como prueba de integridad.
- No aceptar lote parcial en SQLite.
- No permitir reimportaciones.
- No modificar producción en pruebas.
- No desbloquear APIs de pago.
- No cambiar automáticamente a un modelo más caro.
- No activar facturación ni créditos adicionales.
- No descargar modelos locales.
- No fusionar ramas ni publicar código fuente en el repositorio documental público.

## 16. Condición de finalización de 10.1-B

La Fase 10.1-B termina cuando el contrato, exportación, recepción, Gatekeeper, bóveda, cuarentena, reintentos, idempotencia, DB aislada y CLI superan todas las pruebas de aceptación.

La implementación se publicará en una rama de código independiente y será revisada mediante Pull Request. No se fusionará automáticamente.

**La finalización de 10.1-B no autoriza por sí sola el smoke test, benchmark de 300 cadenas ni traducción masiva. Cada una requiere su propia revisión y autorización.**
