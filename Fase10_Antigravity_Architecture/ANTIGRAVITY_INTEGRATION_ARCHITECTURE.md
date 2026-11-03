# Integración de Antigravity sin API — diseño vigente

**Fase:** 10 — Integración  
**Estado:** Diseño documental; adaptador y Gatekeeper pendientes de inspección/implementación local  
**Última revisión:** 2026-11-03

## 1. Decisión

La opción recomendada para validar Antigravity es un **intercambio supervisado de archivos JSON/JSONL desde un workspace separado, con Gatekeeper local fuera del alcance del agente**.

El principio de seguridad es el mínimo acceso: Antigravity solo necesita recibir un lote de texto protegido y devolver sus traducciones. No necesita abrir el código del traductor, la base SQLite, la memoria de traducción, el glosario maestro ni ninguna ruta de Nolvus.

**No se recomienda abrir como workspace de Antigravity la carpeta raíz de Nolvus AI Translator.** Aunque se le indique al agente que no toque determinados archivos, si estos quedan dentro de su workspace podrían seguir siendo accesibles según la configuración de permisos.

## 2. Arquitectura propuesta

### 2.1. Separación física de espacios

Se proponen dos espacios independientes:

**Espacio local del traductor** — permanece bajo control de la aplicación y del usuario:
- Código fuente y configuración.
- Base SQLite de producción.
- Glosario Maestro y Memoria de Traducción.
- DB aislada de pruebas.
- Gatekeeper, logs y auditoría.
- Acceso de solo lectura a inventarios y archivos de origen cuando se preparan los lotes.

**Workspace de intercambio Antigravity** — carpeta independiente abierta como workspace propio:
```text
C:\NolvusAIT_Exchange\
└── job_<job_id>\
    ├── 00_PROTOCOL.md
    ├── 01_INPUT\
    │   ├── batch_<id>_meta.json
    │   └── batch_<id>_input.jsonl
    ├── 02_OUTPUT\
    │   └── batch_<id>_output.jsonl
    └── 03_STATUS\
        └── agent_result.json
```

El workspace de Antigravity no contiene código fuente, DB, glosario, TM, rutas de mods, archivos BSA, STOCK GAME ni credenciales. El traslado de los archivos entre ambos espacios se realiza por el usuario o por un mecanismo local controlado. No se sincronizan carpetas de producción completas con el workspace del agente.

### 2.2. Reglas de acceso

| Elemento | Aplicación local | Agente Antigravity |
|---|---|---|
| Código fuente del traductor | Lectura/escritura controlada | No incluido en su workspace |
| DB de producción | Lectura/escritura por la aplicación | No incluida |
| Glosario y TM | Lectura/escritura por el núcleo | No incluidos |
| Texto de entrada del lote | Genera y conserva el original | Solo lectura |
| JSONL de salida | Lee y valida | Escribe en su workspace |
| DB aislada de prueba | Gatekeeper local | No incluida |
| STOCK GAME, mods, BSA, MO2 y partidas | Nunca modificar en esta fase | No incluidos |

Los permisos efectivos de Antigravity deben comprobarse en la versión Windows instalada. No se asume que el workspace otorgue automáticamente permisos uniformes ni que las políticas de otros sistemas operativos se comporten igual.

## 3. Contrato de intercambio

### 3.1. Entrada

La aplicación genera un manifiesto JSON y un JSONL UTF-8. Cada registro debe incluir, como mínimo:

- `schema_version`
- `job_id` y `batch_id`
- `seq_id` estable dentro del lote
- `string_uid` determinista
- `source_hash`
- `context_signature`
- `record_type`, `subrecord_type` y dominio
- `masked_source_text`
- Instrucciones ES-419 y términos aplicables del glosario que puedan compartirse con el modelo

Los placeholders se enmascaran antes de exportar. Por defecto, **no se entrega al modelo un mapa que revele el contenido original de cada variable protegida**. Cuando haga falta orientación gramatical, se envían pistas no sensibles (por ejemplo, “nombre propio”, “cantidad”, “duración” o “valor numérico”) sin revelar el token interno ni el valor sustituido.

El contenido original de Skyrim se trata como dato no confiable. Si una línea contiene instrucciones, URLs, comandos o texto que intenta cambiar las reglas del agente, debe traducirse como contenido del juego; nunca se ejecuta ni se sigue como instrucción operativa.

### 3.2. Salida

Antigravity escribe **un archivo de salida completo por lote** en `02_OUTPUT/`. No se presupone `write_file(..., Append=True)`, append incremental, API local ni hot-folder autónomo.

Cada línea de salida corresponde a un objeto JSON independiente y contiene:

- `seq_id`
- `string_uid`
- `source_hash`
- `translated_masked_text`
- `notes` opcional, separada del texto de traducción

No se permiten líneas de comentario, encabezados Markdown, bloques de código ni texto adicional dentro del JSONL.

### 3.3. Hashes e idempotencia

- La aplicación calcula SHA-256 del manifiesto y del archivo de entrada.
- El Gatekeeper calcula SHA-256 del archivo recibido y registra el hash de salida.
- El identificador del lote más el hash de entrada impiden confundir respuestas antiguas con lotes nuevos.
- Una misma salida no puede importarse dos veces.
- Los nombres de archivo no sustituyen la validación interna de IDs y hashes.

## 4. Gatekeeper local

El Gatekeeper se ejecuta **fuera del workspace de Antigravity**, desde el entorno local del traductor y por acción del usuario o de un proceso local controlado. El agente no ejecuta comandos para importar, validar o escribir en SQLite.

### 4.1. Validaciones obligatorias

1. JSONL UTF-8 válido, una estructura JSON por línea.
2. Esquema y versión del contrato reconocidos.
3. Conjunto exacto de `seq_id` y `string_uid` esperados, sin faltantes, extras ni duplicados.
4. Coincidencia de `source_hash`, `job_id` y `batch_id`.
5. Preservación exacta de todos los placeholders enmascarados.
6. Preservación de etiquetas, saltos y estructura técnica.
7. Las claves de traducción no modificables siguen intactas.
8. Cumplimiento del Glosario Maestro y de la política ES-419.
9. Ausencia de respuestas vacías, texto residual sin traducir y contenido añadido no solicitado.

Las comprobaciones deterministas no certifican por sí solas la fidelidad narrativa, el lore ni la naturalidad. Esas dimensiones requieren revisión lingüística y humana.

### 4.2. Ingesta de prueba y recuperación

La primera etapa utiliza una DB de prueba separada de producción.

1. El Gatekeeper parsea las líneas individualmente y clasifica cada UID.
2. Las líneas completas y válidas se guardan en un área de staging local, no se promueven automáticamente a producción.
3. Si existen errores, genera un paquete de reparación con los IDs faltantes o rechazados; el agente devuelve un archivo corregido en un nuevo staging.
4. El Gatekeeper vuelve a validar el conjunto completo.
5. Solo al superar el contrato de lote, el Gatekeeper hace una transacción en la **DB aislada de prueba** y archiva entrada, salida, hashes y reporte.

JSONL permite recuperar registros completos que se puedan parsear y validar. **No es una transacción y no garantiza recuperar una escritura interrumpida al 100 %.** La última línea parcial, IDs faltantes y registros malformados se ponen en cuarentena y se reintentan. Esta conducta debe probarse mediante una interrupción controlada.

## 5. Comparación de alternativas

| Alternativa | Ventaja | Riesgo/limitación | Decisión |
|---|---|---|---|
| Copiar y pegar texto manual | Fácil de iniciar | Difícil de auditar, escalar y reanudar; propenso a errores manuales | Solo para una comprobación mínima, no para el flujo sostenido |
| Abrir el proyecto completo como workspace compartido | Menos pasos de archivos | Expone al agente código, DB, glosario y otros recursos innecesarios; depende de permisos | No recomendada |
| JSON/JSONL en workspace separado + Gatekeeper local | Mínimo acceso, trazabilidad, validación independiente y recuperación por UID | Requiere implementar traslado/validación y probar el contrato | **Recomendada** |
| El agente escribe en SQLite o ejecuta el Gatekeeper | Menos intervención manual | Mezcla capas, amplía permisos y hace difícil aislar escrituras/fallos | Descartada |

## 6. Modelo operativo en Antigravity

- El usuario confirma el modelo disponible en el selector justo antes de la ejecución.
- El usuario abre únicamente `C:\NolvusAIT_Exchange\job_<job_id>\` como workspace.
- Se pide procesar un lote o una Supertanda acotada y escribir la respuesta en `02_OUTPUT/`.
- El agente no decide importar, validar, borrar ni mover resultados.
- El usuario cierra o deja de usar el workspace de intercambio al finalizar la tanda.
- El Gatekeeper local revisa el resultado y produce reporte de PASS/RETRY/REJECT.

Las Supertandas de 5–15 lotes son una hipótesis de operación propuesta en los documentos de escalabilidad; su tamaño debe determinarse empíricamente y puede ser menor.

## 7. Cuotas y permisos

Antes de cada corrida:

1. Confirmar en Settings que **AI Credit Overages = Never**. La configuración de cuenta del usuario no ha sido verificada desde este informe.
2. Confirmar modelo visible en el selector y registrar la fecha; la lista de Gemini 4 Argon, Claude Sonnet 5.5 y Claude Opus 5.5 es la disponibilidad reportada al diseñar la Fase 10, no una garantía actual.
3. Verificar los permisos reales en Windows para la versión instalada.
4. Registrar el uso/cuota visible antes y después de la corrida.
5. No habilitar API, facturación, compra de créditos, automatización de clics ni descargas locales.

Fuentes oficiales consultadas:
- [Antigravity Permissions](https://antigravity.google/docs/permissions)
- [Antigravity Agent Settings](https://antigravity.google/docs/agent-settings)
- [Antigravity Plans and Overages](https://antigravity.google/docs/plans)

## 8. Dependencias y secuencia

Este diseño no afirma que el adaptador ni el Gatekeeper estén implementados. Antes de ejecutar el smoke test:

1. Inspeccionar el código local actual y detectar qué componentes ya existen.
2. Con autorización expresa, completar únicamente las piezas que falten en un entorno aislado.
3. Probar serialización/deserialización, hashes, validación, cuarentena, idempotencia, recuperación y DB de prueba.
4. Ejecutar preflight y smoke test de 25 cadenas.
5. Solo tras superar y revisar el smoke test, ejecutar el benchmark de 300 cadenas.

La vía API permanece conservada como alternativa desacoplada. No se activará ni se presupuestará gasto sin nueva autorización.
