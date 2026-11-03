> **POLÍTICA DE GLOSARIO — FUENTE CANÓNICA:** Este archivo es la fuente documental para reglas terminológicas y tratamiento de nombres propios. La existencia de una regla aquí no demuestra que todos sus términos estén cargados o aprobados en la base de datos actual. Comprobar el Glosario Maestro real y su estado antes de preparar cada paquete de contexto.

---

# POLÍTICA DEL GLOSARIO MAESTRO Y BASE DE CONOCIMIENTO DE THE ELDER SCROLLS
**Proyecto**: NOLVUS AI TRANSLATOR — Nolvus Awakening 6.0.21  
**Implementación**: [`GlossaryTerm`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/core/models.py#L204-L220), [`GlossaryManager`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/lore/glossary.py), [`seed_data.py`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/lore/seed_data.py)  

---

## 1. Propósito del Glosario Maestro

En una lista masiva como **Nolvus Awakening (3.700 mods activos)**, cientos de mods distintos hacen referencia a las mismas ciudades, facciones, deidades, artefactos, personajes y conceptos mágicos de Tamriel. El **Glosario Maestro** garantiza que:
1. Ningún nombre propio protegido sea traducido literalmente por error (p. ej., evitar que *Delphine* pase a *Delfina* o *Winterhold* sea traducido de tres formas distintas en diferentes mods).
2. Todos los mods compartan exactamente la misma terminología aprobada en conviviencia armónica con el mundo base de Skyrim.
3. Cualquier término ya confirmado (`approval_status = 'approved'`) quede bloqueado contra modificaciones automáticas silenciosas ([`GlossaryConflictError`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/lore/glossary.py#L16-L18)).

---

## 2. Estructura Obligatoria de 10 Campos por Entrada

Cada registro de la tabla `glossary_terms` en SQLite ([`database.py`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/core/database.py#L114-L129)) y del modelo [`GlossaryTerm`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/core/models.py#L204-L220) almacena obligatoriamente los siguientes **10 atributos auditables**:

| # | Campo en Base de Datos / Modelo | Descripción Funcional | Ejemplo (`city_whiterun`) |
| :-: | :--- | :--- | :--- |
| 1 | `source_term` | Término original exacto en inglés o lengua del lore. | `Whiterun` |
| 2 | `approved_translation` | Traducción `ES-419` aprobada (o el término invariable si se conserva). | `Carrera Blanca` |
| 3 | `category` | Categoría ontológica dentro del universo de The Elder Scrolls ([`GlossaryCategory`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/core/models.py#L77-L97)). | `city` |
| 4 | `definition` | Definición concisa de la entidad dentro del mundo del juego. | `Comarca y capital central de Skyrim` |
| 5 | `context_notes` | Contexto narrativo e histórico dentro del lore de Tamriel. | `Comarca central gobernada por el Jarl Balgruuf el Grande, sede de Cuenca del Dragón y Jorrvaskr.` |
| 6 | `usage_rule` / `translation_rule` | Regla operativa de aplicación sintáctica y técnica (`preserve_untranslated`, `use_official_translation`, `translate_rank`, `contextual_translate`). | `Aplicar siempre la traducción canónica establecida en el juego base Skyrim...` (`use_official_translation`) |
| 7 | `keep_original` | Indicador booleano (`1` / `0`) que señala si el término debe conservarse exactamente sin traducir. | `False` (`0`) |
| 8 | `justification` | Justificación lingüística y de localización de la decisión adoptada. | `Término con traducción oficial consolidada en The Elder Scrolls V: Skyrim que convive directamente con los registros de Skyrim.esm y DLCs.` |
| 9 | `source_reference` | Fuente documental de referencia (Juego base oficial, UESP, Imperial Library). | `Localización oficial Skyrim / Lore UESP / Política ES-419` |
| 10 | `approval_status` | Estado de validación (`approved`, `pending_review`, `requires_lore_research`). | `approved` |

---

## 3. Categorías Ontológicas Soportadas (`GlossaryCategory`)

El sistema clasifica el conocimiento de The Elder Scrolls en **17 categorías especializadas**:
1. **`npc`**: Personajes y NPCs individuales (*Delphine*, *Paarthurnax*, *Serana*, *Inigo*, *Lucien*, *Kaidan*, *Durnehviir*).
2. **`historical_figure`**: Figuras históricas y míticas (*Tiber Septim*, *Ysgramor*).
3. **`region`**: Continentes, provincias, planos y grandes regiones (*Tamriel*, *Skyrim*, *Solstheim*, *Sovngarde*, *Páramo del Martillo*, *Roca Alta*, *Ciénaga Negra*).
4. **`city`**: Ciudades y capitales de comarca (*Carrera Blanca*, *Soledad*, *Ventalia*, *Hibernalia*, *Lucero del Alba*, *Cauce Boscoso*, *Riften*, *Markarth*, *Falkreath*, *Morthal*).
5. **`location`**: Mazmorras, fortalezas, túmulos, planos específicos y monumentos (*Alto Hrothgar*, *Garganta del Mundo*, *Túmulo de las Cataratas Lúgubres*, *Límite Sombrío*, *Cuenca del Dragón*, *Forja del Cielo*, *Recordatorio de las Almas*, *Apocrypha*, *Jorrvaskr*).
6. **`faction`**: Gremios, órdenes militares y facciones políticas (*Los Compañeros*, *Capas de la Tormenta*, *Barbas Grises*, *Los Cuchillas*, *Hermandad Oscura*, *Gremio de Ladrones*, *Colegio de Hibernalia*, *Guardia del Alba*, *Renegados*, *Thalmor*, *Colegio de Bardos*).
7. **`race`**: Razas jugables y linajes de Nirn (*Guardia Rojo*, *Khajiita*).
8. **`creature`**: Bestiario y criaturas (*Draugr*, *Falmer*, *Dwemer*, *Spriggan*, *Cauro*, *Bruja cuervo*, *Madre fuego fatuo*, *Skeever*).
9. **`religion_cult`**: Aedra, Nueve Divinos, Príncipes Daedra y entidades cósmicas (*Akatosh*, *Talos*, *Mara*, *Stendarr*, *Arkay*, *Dibella*, *Julianos*, *Kynareth*, *Zenithar*, *Azura*, *Hermaeus Mora*, *Molag Bal*, *Mehrunes Dagon*, *Sheogorath*, *Nocturnal*, *Meridia*, *Shor*, *Sithis*).
10. **`artifact`**: Artefactos daédricos, aédricos y reliquias históricas (*Pergamino Antiguo*, *Wuuthrad*, *Quebrantador del Amanecer*).
11. **`spell`**: Hechizos, gritos y poderes.
12. **`magic_system`**: Escuelas, atributos y sistemas arcanos (*Thu'um*, *Salud*, *Magia*, *Aguante*).
13. **`quest`**: Nombres de misiones y arcos narrativos.
14. **`historical_event`**: Guerras, tratados y eras históricas.
15. **`rank_title`**: Títulos nobiliarios, militares y de facción (*Sangre de Dragón*, *Dovahkiin*, *Jarl*, *Thane*, *Edecán*, *Archimago*, *Heraldo*).
16. **`item`**: Objetos, materiales e ingredientes recurrentes.
17. **`skyrim_term`**: Conceptos propios del sistema de juego (*Gema de alma*, *Muro de poder*).

---

## 4. Reglas Operativas de Decisión Terminológica

### Regla 1: Nombres Propios de Personajes y Deidades (`PRESERVE_UNTRANSLATED`)
- **Criterio**: Los nombres de pila de NPCs, figuras históricas, Aedra, Príncipes Daedra y términos en lengua dracónica/ehlnofex/dwemer (*Paarthurnax*, *Alduin*, *Miraak*, *Dovahkiin*, *Thu'um*, *Jorrvaskr*, *Sovngarde*, *Apocrypha*) **se conservan sin traducir** (`keep_original = True`).
- **Excepción con Apodos o Epítetos Descriptivos**: Cuando un nombre propio va acompañado de un epíteto o sobrenombre descriptivo consagrado en el lore base (*Ulfric Stormcloak* -> *Ulfric Capa de la Tormenta*, *Kodlak Whitemane* -> *Kodlak Melena Blanca*, *Alduin the World-Eater* -> *Alduin el Devorador de Mundos*), se conserva el nombre propio y se traduce el epíteto según la forma canónica aprobada.

### Regla 2: Topónimos y Facciones del Mundo Base (`USE_OFFICIAL_TRANSLATION`)
- **Criterio**: Dado que conservamos provisionalmente los `57.261` registros oficiales de Bethesda (`Skyrim.esm`, DLCs y Creation Club), los topónimos y facciones que ya poseen traducción oficial en el mapa base (*Whiterun* -> *Carrera Blanca*, *Solitude* -> *Soledad*, *Windhelm* -> *Ventalia*, *Winterhold* -> *Hibernalia*, *Dawnstar* -> *Lucero del Alba*, *Riverwood* -> *Cauce Boscoso*, *The Companions* -> *Los Compañeros*) **deben utilizar su nombre oficial en español** en todos los mods para evitar que un diálogo o misión de un mod pida ir a *"Whiterun"* mientras el mapa del juego muestra *"Carrera Blanca"*.
- Los topónimos que en la localización oficial de Skyrim se mantuvieron invariables (*Riften*, *Markarth*, *Falkreath*, *Morthal*, *Solstheim*, *Tamriel*, *Skyrim*) se mantienen invariables (`keep_original = True`).

### Regla 3: Topónimos y Nombres Propios Nuevos Introducidos por Mods (`PENDING_RESEARCH` / `REQUIRES_LORE_RESEARCH`)
- Cuando el clasificador [`StringClassifier`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/classification/classifier.py#L144-L169) o el validador [`TranslationQualityValidator`](file:///c:/Skyrim%20proyecto%20traducci%C3%B3n/src/nolvus_translator/quality/validator.py) detecta un topónimo o término propio de lore en registros `LCTN`, `CELL`, `WRLD`, `FACT` o `BOOK` que no figura en el Glosario Maestro (por ejemplo, *Unslaad*, *Bruma*, *Clockwork Castle*, *Wyrmstooth*), el sistema:
  1. **No inventa contexto ni traduce a ciegas**.
  2. Lo marca con estado `pending_research` / `requires_lore_research` (`718` ocurrencias / `352` términos únicos detectados en `Snapshot #1`) para revisión y alta controlada en el Glosario Maestro antes de su traducción definitiva.
