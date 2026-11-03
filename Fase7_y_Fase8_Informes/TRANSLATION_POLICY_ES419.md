> **POLÍTICA LINGÜÍSTICA — FUENTE CANÓNICA:** Este archivo sigue siendo la fuente documental de la política ES-419. La afirmación histórica de que está activa en una versión concreta de código/base de datos no se vuelve a certificar por el solo hecho de conservar este documento. La sincronización de implementación debe comprobarse contra el código y los datos actuales antes de cada ejecución.

---

# POLÍTICA DE LOCALIZACIÓN AL ESPAÑOL LATINOAMERICANO NEUTRO (`ES-419`)
**Proyecto**: NOLVUS AI TRANSLATOR — Nolvus Awakening 6.0.21  
**Estado**: Definitiva y Activa en Código y Base de Datos (`schema_version = 1.1.0`)  

---

## 1. Principio Rector: Localización Propia desde Cero para Mods

Toda la localización generada por **NOLVUS AI TRANSLATOR** para los mods de terceros instalados en Nolvus Awakening (`3.700` mods activos) se construye **íntegramente desde cero a partir del texto original en inglés (`en` -> `es-419`)**.

### Prohibiciones Estrictas (Cero Reutilización Comunitaria)
Queda estrictamente prohibido en el motor de extracción, en la Memoria de Traducción (`translation_memory`) y en el Glosario Maestro (`glossary_terms`):
1. **Reutilizar paquetes comunitarios en español** incluidos dentro de mods de terceros (como archivos `Interface/Translations/<Mod>_spanish.txt` o tablas `<Mod>_Spanish.*STRINGS` no oficiales).
2. **Adaptar o parafrasear traducciones de español de España (`es-ES`)** realizadas por terceros para mods.
3. **Copiar cadenas traducidas de Nexus Mods**, repositorios externos o parches de traducción de otros autores.
4. **Importar memorias de traducción (TMX/CSV/JSON) de terceros**: el motor [`TranslationMemoryEngine.store_approved()`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/memory/tm_engine.py#L46-L95) bloquea por código (`ValueError`) cualquier intento de insertar registros con origen `'community'`.
5. **Procesar archivos MCM en otros idiomas extranjeros** (`_chinese.txt`, `_czech.txt`, `_french.txt`, `_german.txt`, `_italian.txt`, `_japanese.txt`, `_polish.txt`, `_russian.txt`, `_turkish.txt`, etc.): el extractor [`ProjectService.scan_and_extract()`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/services/project_service.py#L285-L291) procesa exclusivamente los archivos fuente originales `*_english.txt`.

---

## 2. Tratamiento de la Localización Oficial Preexistente de Bethesda

Se conserva provisionalmente la localización oficial de Bethesda ya instalada en `STOCK GAME (Data)` para:
- `Skyrim.esm` (`37.767` ocurrencias ganadoras)
- `Update.esm` (`488` ocurrencias ganadoras)
- `Dawnguard.esm` (`5.314` ocurrencias ganadoras)
- `HearthFires.esm` (`1.429` ocurrencias ganadoras)
- `Dragonborn.esm` (`6.546` ocurrencias ganadoras)
- Los **74 plugins oficiales de Creation Club** (`cc*.esm` y `cc*.esl`, `5.717` ocurrencias ganadoras)

### Reglas de Separación Estricta
1. **Inventario Separado**: Estas **57.261 ocurrencias** (`48.147` cadenas únicas) se registran bajo la razón explícita `"Localización oficial de Bethesda preexistente (Skyrim + DLCs + Creation Club)"` y se contabilizan en el indicador dedicado `official_bethesda_occurrences` del informe de cobertura ([`CoverageReporter`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/reports/coverage.py#L50-L135)).
2. **No Mezcla Automática en Memoria ES-419**: La localización oficial de Bethesda (redactada originalmente en español peninsular) puede consultarse como referencia contextual (`tm_origin_type = 'official_bethesda_reference'`), pero **nunca se mezcla automáticamente** como si fuera producción propia `es-419` (`tm_origin_type = 'own_es419'`) ni se descuenta silenciosamente de las cadenas pendientes de los mods.
3. **Registros Base Sobrescritos por Mods de Terceros (`37.159` ocurrencias)**: Cuando un mod de terceros en inglés (p. ej. `Unofficial Skyrim Special Edition Patch.esp`, `WACCF`, `Mysticism`, `Gourmet`) sobrescribe un registro cuyo maestro original (`origin_plugin`) es `Skyrim.esm` o un DLC, el registro ganador queda en inglés dentro del orden de carga de MO2. Esas `37.159` ocurrencias se clasifican como `translatable` para que nuestra plataforma las traduzca y distribuya limpiamente en `es-419` mediante Dynamic String Distributor (DSD), evitando la mezcla de inglés/español en partida.

---

## 3. Estándar Lingüístico Español Latinoamericano Neutro (`ES-419`)

El objetivo lingüístico es un **español latinoamericano neutro, natural, literario y atemporal**, comprensible para jugadores de cualquier país de Hispanoamérica sin sonar artificial ni robótico.

### 3.1. Gramática y Pronombres de Segunda Persona
| Caso Gramatical | Regla Obligatoria `ES-419` | Formas Prohibidas (Rechazadas por QA) |
| :--- | :--- | :--- |
| **Segunda persona del plural** | Uso exclusivo de **`ustedes`** y sus posesivos/conjugaciones (`sus`, `les`, `tienen`, `vayan`, `escuchemos`, `defiendan`). | `vosotros`, `vosotras`, `os`, `vuestro`, `vuestra`, `habéis`, `tenéis`, `sois`, `estáis`, `id`, `venid`, `decid`. |
| **Segunda persona del singular (Trato cercano)** | Uso de **`tú`** (`tienes`, `puedes`, `habla`, `escucha`) entre compañeros de armas, iguales, amigos, bandidos o familiares. | Uso de *voseo* regional (`vos tenés`, `vení`, `mirá`) o *ustedeo* forzado en contextos íntimos. |
| **Segunda persona del singular (Trato de respeto o jerarquía)** | Uso de **`usted`** (o fórmulas solemnes de cortesía) al dirigirse a Jarls, reyes, emperadores, sacerdotes, ancianos Barbas Grises, Príncipes Daedra o superiores jerárquicos cuando el tono original refleje deferencia. | Tutear de forma anacrónica o coloquial a figuras de alta autoridad cuando el inglés original emplea registro solemne. |

### 3.2. Neutralidad Léxica y Prohibición de Regionalismos
- **Evitar regionalismos peninsulares (`es-ES`)**: No emplear *coger* (usar *tomar*, *recoger* o *agarrar*), *vale* (usar *de acuerdo*, *está bien*, *entendido*), *tío/chaval*, *ordenador*, *patata* (usar *papa*), *zumo* (usar *jugo*), *echar de menos* (preferir *extrañar*).
- **Evitar modismos locales latinoamericanos**: No emplear mexicanismos (*ahorita*, *chavo*, *platicar*, *güey*), colombianismos (*parce*, *chévere*, *bacano*), argentinismos/rioplatenses (*che*, *pibe*, *quilombo*, *acá* en exceso), ni chilenismos (*po*, *cachai*, *al tiro*).
- **Preservación de la Voz y Personalidad del Personaje**:
  - Un erudito Telvanni como **Neloth** o un mago del Colegio de Hibernalia debe hablar con vocabulario culto, preciso y ligeramente arrogante.
  - Un guerrero nórdico de **Los Compañeros** o un guardia de comarca debe sonar directo, firme y marcial.
  - Un bandido, mercenario o contrabandista del **Gremio de Ladrones** debe sonar rudo y pragmático, usando léxico neutro de bajos fondos (*botín*, *soplón*, *trampa*, *degollar*) sin caer en jerga callejera moderna.
  - Los **libros, crónicas históricas, tratados mágicos y textos religiosos** deben conservar un registro literario, solemne y cuidado.

### 3.3. Reglas por Tipo de Contenido
1. **Interfaz de Usuario (UI) y Menús MCM (`mcm_txt`, `mcm_json`)**:
   - Concisión estricta para evitar desbordamientos visuales en cajas Scaleform/SkyUI (límite de alerta QA: longitud $\le 1.35\times$ respecto al original en cadenas cortas).
   - Verbos en infinitivo para acciones de configuración (*Activar*, *Desactivar*, *Restablecer*, *Mostrar*, *Ocultar*) y sustantivos precisos para parámetros (*Multiplicador de daño*, *Consumo de aguante*, *Tecla de acceso rápido*).
2. **Objetos, Armas, Armaduras, Pociones y Hechizos (`WEAP`, `ARMO`, `ALCH`, `SPEL`, `ENCH`)**:
   - Estructura nominal natural en español (`Iron Greatsword of Scorching` -> `Mandoble de hierro de abrasión`, nunca calcos sintácticos como `Hierro Mandoble`).
   - Coherencia absoluta con los atributos base de Skyrim: `Health` -> `Salud`, `Magicka` -> `Magia`, `Stamina` -> `Aguante`.
3. **Diálogos (`DIAL`, `INFO`) y Etapas de Misión (`QUST`)**:
   - Los objetivos de misión (`QUST:NNAM`) comienzan con verbo en infinitivo o imperativo neutro consistente (*Habla con...*, *Recupera el...*, *Derrota a...*).
   - Las entradas de diario en primera persona (`QUST:CNAM`) mantienen el tono reflexivo del protagonista sin asumir género gramatical marcado cuando pueda emplearse una construcción neutra (*«Logré entrar en el túmulo»* en lugar de *«Estuve cansado/a»*), salvo cuando el motor proporcione variables de género dinámicas.

---

## 4. Protección Innegociable de Sintaxis Técnica

Ninguna regla estilística puede alterar ni eliminar tokens técnicos. El módulo [`VariableProtector`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/quality/protector.py) y el validador [`TranslationQualityValidator`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/quality/validator.py) aplican rechazo automático inmediato (`passed_hard_gates = False`) si se altera cualquier:
- Variable de motor/misión: `<Alias=...>`, `<Global=...>`, `<mag>`, `<dur>`, `<area>`, `<1>`, `<2>`.
- Especificador de formato: `%s`, `%d`, `%.0f`, `%.1f%%`, `{0}`, `{1}`.
- Etiqueta Scaleform/HTML o paginación de libros: `<font ...>`, `</font>`, `<p align='...'>`, `<img ...>`, `[pagebreak]`.
- Clave de localización MCM: `$ModName_OptionKey` (la columna izquierda antes del tabulador `\t` en archivos UTF-16 LE jamás se traduce).
