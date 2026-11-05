# Reglas y referencia técnica

Corte documental **2026-11-05**. Resume lo vigente sin sustituir los contratos del código. [Estado](ESTADO_ACTUAL.md) · [Uso](GUIA_DE_USO.md) · [Histórico y originales](HISTORICO.md).

## Reglas vigentes

1. **Localización propia ES-419:** no incorporar traducciones comunitarias descargadas a la TM o al glosario. La localización oficial preexistente se mantiene separada; no se reescribe masivamente el juego base.
2. **Intención y voz:** conservar historia, emoción, trato y registro. Usar ustedes; elegir tú/usted según contexto y relación. Evitar regionalismos marcados sin forzar literalidad ni inventar género o lore ausentes.
3. **Glosario:** respetar preferencias y nombres aprobados, proteger todos sus metadatos y conservar la procedencia. Las variantes son concordancias explícitas, no sinónimos libres ni nuevas aprobaciones. La revisión de 109 entradas no equivale a verificar documentalmente cada definición heredada.
4. **Integridad:** preservar variables, etiquetas, identificadores, porcentajes técnicos y controles reales/escapados. No hacer reemplazos binarios ciegos ni modificar originales de Nolvus, MO2, STOCK GAME o partidas.
5. **Separación editorial:** candidata, aceptación mecánica, propuesta reparada, referencia humana y texto exportable son evidencias distintas. Gatekeeper no aprueba significado ni alimenta automáticamente la TM con traducciones reales nuevas.
6. **Ejecución acotada:** candado de API activo por defecto, autorización explícita, presupuesto reservado antes de enviar y conciliación posterior. Una credencial disponible no autoriza gasto. No hay fallback automático a un modelo más caro ante errores de red.
7. **Operación reversible:** staging aislado, evidencia original, importación transaccional del grupo, vista previa y respaldos antes de una exportación futura. No reenviar solicitudes inciertas ni repetir candidatas válidas por defecto.

Estas reglas consolidan decisiones existentes. La fase de evolución autorizó y verificó mejoras offline; no autoriza gasto, producción o aprobación automática nueva.

## Recorrido y componentes

Detección de entorno/VFS → extracción y clasificación → contexto, glosario y protección → preparación y presupuesto → respuesta/evidencia → Gatekeeper → staging → revisión editorial → eventual exportación autorizada.

| Área | Módulos locales relevantes | Límite práctico |
|---|---|---|
| Entorno y archivos ganadores | `mo2/`, parsers `formats/` | Un formato inventariado no equivale a texto traducible |
| Protección y QA | `quality/protector.py`, `validator.py`, `gatekeeper.py` | La puntuación mecánica no mide fidelidad; “context_coherence” solo compara signos finales |
| Contexto y terminología | `lore/context_engine.py`, `editorial_policy.py`, `glossary.py` | Metadatos acotados; el piloto no envía una historia completa ni hablante identificado |
| TM | `memory/tm_engine.py` | Identidad contextual v2; coincidencias v1 o entre contextos son sugerencias, no equivalencia segura automática |
| API e intercambio | `translation/openai_provider.py`, `exchange_*`, `budget_control.py`, `model_router.py` | El proveedor solo traduce; no recibe acceso al sistema, herramientas o credenciales del workspace |
| Piloto y recuperación | `translation/pilot_four*.py`, `pilot_recovery.py` | Recuperación focalizada y cuarentena; requiere conciliación clara del estado |
| Revisión y exportación | `ui/case_review.py`, `export/` | Referencias del visor separadas de TM/DB; producción actual no autorizada |

Los identificadores configurados en código son `gpt-6-luna`, `gpt-6.1-sol` y la ruta complementaria de intercambio `gemini-4-argon`; Claude está excluido por la política existente. Esta lista describe configuración observada, no disponibilidad, tarifas o recomendación comercial verificadas hoy. No se consultaron proveedores para reorganizar documentos.

## Formatos y alcance

| Familia | Recorrido existente | Restricción |
|---|---|---|
| ESP/ESM/ESL, ESPFE | Parser estructural, registros comprimidos, salida DSD/tablas según caso | No sobrescribir plugins originales |
| STRINGS/DLSTRINGS/ILSTRINGS | Lectura y reconstrucción tipada | Respetar IDs, longitudes y codificación |
| PEX | AST y reconstrucción selectiva de literales | Filtro de llamadas y protección de identificadores; elegibilidad global aún requiere auditoría |
| PSC | Referencia contextual | No es sustituto de un parche PEX validado |
| MCM TXT / MCM Helper JSON | Claves y texto visible; TXT de salida UTF-16LE BOM | Conservar claves y estructura |
| XML, INI, JSON narrativo | Selección de campos/rutas permitidos y reconstrucción específica | No traducir indiscriminadamente archivos de configuración |
| BSA | Inventario y lectura/extracción selectiva | No reescribir el contenedor; salidas sueltas en un mod separado |
| SWF, DLL, audio y texturas | Inventario o inspección según formato | Fuera de la traducción textual automática; subtítulos se tratan aparte |

Casos binarios importantes: `PERK EPFD` solo es texto con `EPFT=7`; `GMST DATA` solo con EDID de tipo cadena (`s*`). El PEX comparte índices de identificadores y literales: la reconstrucción debe preservar los primeros. Las particularidades completas permanecen en los documentos originales archivados y el código, sin afirmar cobertura del 100 % de Nolvus.

## Compatibilidad, dependencias y evidencia

- El informe de Fase 9 registró Nolvus Awakening 6.0.21 Ultimate, MO2 2.4.4 y STOCK GAME 1.5.97.0. Son datos de aquel escaneo, no una inspección nueva. Compatibilidad DSD/SKSE/Address Library debe comprobarse contra el ejecutable efectivo antes de exportar.
- `pyproject.toml`: proyecto 0.6.0, Python >=3.11; dependencias declaradas Pydantic, RapidFuzz, lxml, charset-normalizer y XlsxWriter. No se fijan aquí versiones instaladas tomadas de auditorías antiguas.
- La política mantiene parsers propios y evita copiar código de terceros. Los análisis antiguos de licencias quedan archivados como referencias fechadas; esta reorganización no revalida sus conclusiones legales ni las condiciones actuales de terceros. Conservar avisos aplicables antes de distribuir.
- La suite funcional vigente está registrada en el estado actual. Los benchmarks de 240.684 filas son sintéticos y de SQLite; no acreditan tiempos/memoria de todos los parsers, UI, exportación o traducción del inventario real.

## Evolución offline implementada y límites

`quality/intelligent_review.py` añade severidades, cifras/ordinales, propuestas de controles terminales y gate de calibración. `lore/mod_knowledge.py` mantiene conocimiento y propuestas por mod/plugin en un sidecar, con procedencia y permisos. `translation/durable_run.py` prueba recibos, conciliación y reanudación sin reenvío; `execute_authorized_stages` coordina etapas dentro de un alcance fijo. El flujo `independent_review` / `review_workflow` / `review_experiment` conecta evaluación, posedición y recomprobación offline, con solicitudes exactas previas al modelo, recibos y reserva acotada. Una respuesta malformada no provoca reenvío ni detiene los demás casos; un transporte incierto requiere conciliación. Solo se ejercitó con simuladores; su uso real y calidad lingüística siguen sin validar.

Contrato de entrada 1.2, lectura compatible 1.0/1.1: hasta cuatro fragmentos de 320 caracteres, sin rutas privadas y con hash de procedencia, datos no confiables. La preparación puede sellar un snapshot contextual; la agrupación coherente es opcional y su coste debe caber en el presupuesto. No se reabren planes históricos con fingerprints modificados. Las entidades NPC son nombres de registro, no nombres propios intraducibles por defecto. El contexto específico no sobrescribe silenciosamente el glosario global aprobado.

La aprobación nueva permanece cerrada: las referencias disponibles y los juicios simulados no bastan para calibrar. Falta evaluar falsos negativos, tiempo humano, semántica y naturalidad en una muestra independiente. Véase [estado y siguiente puerta](ESTADO_ACTUAL.md).

El catálogo `context_catalog` conserva pasajes completos y roles de personaje, misión, objeto, hechizo, lugar y documento, sin convertir etiquetas genéricas en nombres intraducibles. Relaciona registro/grupo/misión dentro del mismo mod/plugin; las fuentes del autor admiten alcance de mod, versiones, permisos y hash. La búsqueda restringe el ámbito antes de BM25 ([FTS5 oficial](https://www.sqlite.org/fts5.html)); la documentación de autores se considera referencia, con compatibilidad de versión no verificada. Las denominaciones aprobadas por mod se incorporan como restricciones; los conflictos con el glosario global quedan explícitos y no lo sobrescriben. Se incorporaron extractos de los README de [MCM Helper](https://github.com/Exit-9B/MCM-Helper/blob/main/README.md) y [Alternate Perspective](https://github.com/KrisV-777/Alternate-Perspective/blob/main/README.md), con hash y licencias conservadas privadamente; no se presume que describan la versión instalada.

`long_text` divide original y candidata solo en límites protegidos compartidos, conserva vecinos narrativos y verifica la recomposición completa. No presume alineación entre frases. `durable_run` continúa los casos independientes mientras bloquea dependencias de envíos/uso inciertos; el presupuesto difiere textos completos que no caben, sin ampliar el cap. Los prompts comparan agencia, modalidad, negación, comparaciones y recursos, aceptando metáforas y paráfrasis fieles. La [evaluación de discurso en traducción literaria](https://aclanthology.org/2023.acl-long.435/) motivó conservar vecinos y consistencia; no certifica resultados propios.

`review_calibration` vincula candidata, fuente, contexto, versión y evaluador con referencias humanas. No convierte cualquier retoque en error semántico, no cuenta mocks y agrupa familias para evitar observaciones duplicadas. Reutilizar una aprobación exige igualdad exacta y nunca libera cuarentenas ni aprueba una reparación nueva.

## Consolidación de responsabilidades

`core/artifacts` comparte hash por bloques y publicación atómica entre visor/workbenches/ejecutor. `quality/review_contract` valida también el puerto legado. La selección de términos aprobados se comparte en proveedor, cola y posedición: las propuestas no son obligaciones. La ingesta usa firma por registro y una sola transacción; conserva APIs y catálogos históricos sin convertir destructivamente sus esquemas.

El proveedor registra el HTTP original antes de parsear/conciliar; un 5xx o timeout con uso desconocido retiene presupuesto y no se reintenta por defecto. La idempotencia económica sigue en `BudgetController`; el diario de reevaluación offline no representa cargos reales. `DurableRun` verifica planes y recibos terminales y serializa callbacks por diario. Las advertencias menores de puntuación no disparan solas el juez, y las observaciones con hashes válidos orientan selección sin filtrar el dictamen al prompt ciego. La ejecución real del juez permanece pendiente de revisión/autorización; no está conectada a producción.

## Mantenimiento documental

`lore/consistency.py` construye un registro de nombres del último escaneo, leído con SQLite `mode=ro`. Usa `origin_plugin`, tipo y FormID para relacionar overrides/separar homónimos, y localización existente del juego base como referencia. Excluye candidatas nuevas de esa autoridad, protege variables y evita tratar palabras comunes/clases de NPC como nombres propios. Las referencias y conflictos son datos acotados, no aprobaciones. Las familias admiten formas de concordancia y preferencias pendientes; las relaciones léxicas son hipótesis.

`ConsistencyContext` combina ese registro con `ModKnowledge` para preparación del proveedor/coordinador, juez y recomprobación. `review_experiment --name-source <DB-solo-lectura>` vincula el hash del nuevo insumo al experimento; rechaza reemplazar la carpeta humana y un cambio de fuente. El diseño controlado acepta también esa fuente, sin ejecutar. Política `risk-2.3.0`, revisor `independent-review-2.2`: una discrepancia de referencia bloquea reutilización ciega/posedición insegura, conserva el estado humano y permanece visible en la cola. No modifica el contrato 1.2 ni habilita transporte real. El visor presenta la procedencia de los nuevos fragmentos cuando la copia los contiene.

- `README.md` solo orienta. `ESTADO_ACTUAL.md` contiene métricas/pendientes; `GUIA_DE_USO.md`, operación; este documento, reglas/técnica; `HISTORICO.md`, cronología y acceso a evidencia.
- Registrar fecha, fuente y si algo es medido, estimado, histórico o propuesto. Actualizar el estado sin repetir una matriz de métricas en todos los documentos.
- Conservar informes cerrados. Cambios de navegación se permiten, pero se registran y el original exacto queda en el archivo con hash. No reinterpretar sus resultados como aprobaciones actuales.
- Mantener los cuatro documentos activos compactos, idealmente hasta 150 líneas cada uno. Llevar el detalle cerrado a un informe fechado o al archivo; no convertir el histórico en otra enciclopedia.
- Los originales de la raíz y las fuentes documentales previas están en `archivo/originales_20261105.zip`, verificables con `archivo/MANIFIESTO.json`. Los archivos locales y públicos tienen distinto alcance; no publicar el archivo privado del workspace.
- La publicación pública contiene solo documentación agregada. La autorización vigente cubre implementación privada y pruebas offline; no habilita gasto, producción, publicación de fuentes ni modificación silenciosa del glosario aprobado.
