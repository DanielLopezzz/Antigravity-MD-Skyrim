# Auditoría de Licencias, Herramientas Existentes y Componentes de Terceros

Este documento registra la investigación legal y técnica realizada durante la **Fase 0** del proyecto **NOLVUS AI TRANSLATOR**, evaluando las herramientas existentes en el ecosistema de localización de *The Elder Scrolls V: Skyrim* y las dependencias de software utilizadas.

---

## 1. Matriz Comparativa de Herramientas Investigadas

| Herramienta | Repositorio / Autor | Lenguaje / Stack | Formatos Soportados | Licencia | Riesgo Copyleft / Restricción | Decisión Arquitectónica |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **NIM Translator** (*Lexicon / Phoenix*) | `YD525/NIM-Translator` | C# / .NET (WPF/WinForms) | `ESP`, `ESM`, `ESL`, `STRINGS`, `PEX`, `MCM` (`TXT`/`XML`) | **LGPL-3.0** | **Medio**: Enlazar sus ensamblados obliga a cumplir LGPL-3.0 (permitir relinkeo y distribuir avisos/código modificado de la librería). | **No enlazar ni copiar código**. Se adoptan conceptualmente (clean-room) sus patrones de agrupación contextual, preprocesado protector de etiquetas y caché aislada por modelo. |
| **xTranslator** | `MGuffin/xTranslator` | Delphi / Object Pascal | `ESP`, `ESM`, `ESL`, `*STRINGS`, `PEX`, `MCM`, `XML`, `BSA/BA2` | **MPL 1.1** + componentes **GPL / MIT / MPL** (`SynEdit`, `xEdit` derivados) | **Medio-Alto**: Mezcla MPL 1.1 con unidades adaptadas de ecosistema GPL (`xEdit`). Código monolítico acoplado a GUI VCL. | **No copiar código**. Se implementan parsers binarios nativos clean-room desde especificación pública (UESP) y se soporta carga opcional de tablas oficiales `*_English` / `*_Spanish.*STRINGS` como semilla terminológica. |
| **Dynamic String Distributor (DSD)** | `SkyHorizon3/SSE-Dynamic-String-Distributor` | C++ (Plugin SKSE64 + CommonLibSSE-NG) | Consume archivos `.json` en `Data/SKSE/Plugins/DynamicStringDistributor/<Plugin>/` | **GPL-3.0** (código C++ y `.dll`) | **Nulo por interoperabilidad de datos**: Generar archivos `.json` independientes consumidos por DSD en tiempo de ejecución NO constituye obra derivada del binario GPL-3.0. | **Adoptar como formato de salida primario no invasivo** para textos de plugins (`ESP`/`ESM`/`ESL`), sin empaquetar ni redistribuir el binario `DynamicStringDistributor.dll`. |
| **SSE Auto Translator** | `Cutleast/SSE-Auto-Translator` | Python / C++ helpers | Escaneo de modlist, descarga de Nexus Mods, flujos `DSD` y edición básica | **CC BY-NC-ND 4.0** (*Atribución - NoComercial - SinObraDerivada*) | **CRÍTICO / PROHIBIDO**: La cláusula **NoDerivatives (ND)** prohíbe estrictamente reutilizar, adaptar o redistribuir código derivado. | **CERO reutilización de código**. Analizado únicamente como referencia funcional externa para identificar limitaciones del enfoque basado en descargas. |
| **Mutagen** | `Mutagen-Modding/Mutagen` | C# / .NET | `ESP`, `ESM`, `ESL` (lectura/escritura tipada) | **GPL-3.0** | **Alto**: Copyleft fuerte; cualquier aplicación que enlace `Mutagen.Bethesda.Skyrim` debe licenciarse bajo GPL-3.0. | **Descartado**. Además de la infección GPL-3.0, el entorno del usuario no dispone de SDK `.NET` instalado. |
| **Champollion** | `Orvid/Champollion` (`li1nx`) | C++ CLI | Descompilación `PEX -> PSC / ASM` | **LGPL-3.0** | **Bajo si es proceso CLI externo opcional; Medio si se empaqueta**. | **No empaquetado**. Implementamos nuestro propio parser/reconstructor AST binario clean-room para `.pex` basado en especificación pública. |

---

## 2. Análisis Detallado de Cada Herramienta Evaluada

### 2.1 NIM Translator (`YD525/NIM-Translator`)
* **Funciones aprovechables (a nivel conceptual)**:
  * Agrupación secuencial de textos conversacionales en lugar de traducir líneas sueltas sin contexto.
  * Motor heurístico de preprocesamiento para enmascarar secuencias de escape (`\n` en MCM), etiquetas y variables de motor antes de invocar un LLM.
  * Separación entre memoria histórica aprobada y caché de lote por modelo.
* **Limitaciones frente a Nolvus Awakening 6.0.21**:
  * No resuelve de forma integral el Virtual File System (VFS) completo de Mod Organizer 2 (`modlist.txt` invertido + `plugins.txt` + resolución de FormID ganador entre más de 2.000 mods y parches `Synthesis`/conflict resolution).
  * No genera paquetes de distribución no invasivos mediante `Dynamic String Distributor (DSD)` con verificación específica del runtime `STOCK GAME` (`1.5.97` vs `1.6.1170` vs `1.7.104.0`).

### 2.2 xTranslator (`MGuffin/xTranslator`)
* **Funciones aprovechables (a nivel conceptual)**:
  * Modelo de seguridad para scripts `.pex`: bloqueo estricto de identificadores internos de la tabla de cadenas para impedir corrupción de nombres de funciones, estados, propiedades y tipos Papyrus.
  * Construcción automática de diccionarios bilingües emparejando los IDs de `Skyrim_English.*STRINGS` con `Skyrim_Spanish.*STRINGS` (y DLCs `Dawnguard`, `HearthFires`, `Dragonborn`, `cc*`).
* **Limitaciones**:
  * Herramienta orientada a edición manual mod por mod; inviable para orquestar la localización incremental, clasificación contextual y estimación de costos de una lista masiva como Nolvus Awakening.

### 2.3 Dynamic String Distributor (`SkyHorizon3/SSE-Dynamic-String-Distributor`)
* **Funciones aprovechables**:
  * Permite inyectar traducciones de registros de plugins (`FULL`, `DESC`, `CNAM`, `NNAM`, `NAM1`, `RNAM`, `ITXT`, `DNAM`, `EPFD`, `EPF2`, `RDMP`, `MNAM`, `FNAM`, `SHRT`, `TNAM`, `DATA`) en memoria al iniciar el juego usando archivos JSON en `SKSE/Plugins/DynamicStringDistributor/<PluginName>/`.
  * **Ventaja crítica para Nolvus**: Evita modificar un solo byte de los `.esp`/`.esm`/`.esl` originales de Nolvus Awakening (respetando hashes, parches, FormIDs y orden de carga).
* **Limitaciones y riesgos**:
  * Requiere `SKSE64` + `Address Library` compatibles con el ejecutable exacto de `STOCK GAME\SkyrimSE.exe` (y el parche `DynamicStringDistributor-1.7.104-Patch` si el runtime efectivo es `1.7.104.0`).
  * Conflictos conocidos de precedencia con **Description Framework** (`BOOK CNAM`) y **SkyPatcher** (alteración de `FULL`).
  * No cubre archivos MCM (`Interface/Translations/*.txt`), scripts `.pex` ni archivos auxiliares de interfaz (que nuestra herramienta exporta complementariamente como archivos sueltos en el mod independiente de MO2).

### 2.4 SSE Auto Translator (`Cutleast/SSE-Auto-Translator`)
* **Evaluación**: Su propósito primordial es localizar traducciones humanas previas en Nexus Mods y automatizar su descarga/conversión a DSD.
* **Restricción legal estricta**: Licenciado bajo **CC BY-NC-ND 4.0**. Queda prohibido copiar cualquier fragmento de código, expresión regular o plantilla de dicho repositorio.

---

## 3. Dependencias Reales de NOLVUS AI TRANSLATOR (100 % Permisivas / Zero Copyleft)

Para garantizar máxima libertad de uso, mantenimiento y empaquetado sin obligaciones GPL ni LGPL, **NOLVUS AI TRANSLATOR** utiliza exclusivamente bibliotecas con licencias permisivas verificadas:

| Biblioteca / Componente | Versión Verificada | Licencia | Uso en el Proyecto | Obligaciones de Distribución |
| :--- | :--- | :--- | :--- | :--- |
| **Python Standard Library** (`sqlite3`, `tkinter`, `zlib`, `struct`, `ctypes`, `unittest`) | `3.13.7` | **PSF-2.0** (SQLite: Dominio Público; Tcl/Tk: BSD-style) | Motor base, base de datos local WAL + FTS5, GUI nativa oscura, inspección PE Win32, compresión zlib y suite de pruebas. | Conservar aviso de copyright PSF en distribuciones binarias de Python. **Sin copyleft**. |
| **Pydantic** | `2.13.4` | **MIT** | Validación estricta de modelos de configuración, esquemas DSD, glosario y manifiestos MO2. | Conservar aviso MIT. **Sin copyleft**. |
| **RapidFuzz** | `3.14.5` | **MIT** *(Nota: a diferencia de `fuzzywuzzy` que es GPLv2, RapidFuzz es MIT)* | Búsqueda difusa de alta velocidad en Memoria de Traducción y detección de similitud terminológica. | Conservar aviso MIT. **Sin copyleft**. |
| **lxml** | `6.1.1` | **BSD-3-Clause** | Parsing seguro y estructurado de archivos XML (MCM XML, FOMOD `ModuleConfig.xml`, diccionarios XML). | Conservar aviso BSD-3. **Sin copyleft**. |
| **charset-normalizer** | `3.4.7` | **MIT** | Detección determinista de codificación (`UTF-8`, `UTF-16LE BOM`, `Windows-1252`) en recursos de mods. | Conservar aviso MIT. **Sin copyleft**. |
| **XlsxWriter** | `3.2.9` | **BSD-2-Clause** | Exportación opcional de informes de cobertura y auditoría a libros Excel (`.xlsx`). | Conservar aviso BSD-2. **Sin copyleft**. |
| **Parsers Bethesda (`formats/*`) y Codec LZ4 (`core/lz4_codec.py`)** | Propios (`0.6.0`) | **MIT** (Implementación Clean-Room propia) | Lectura/reconstrucción binaria de `ESP/ESM/ESL`, `*STRINGS`, `PEX` AST, `BSA` v104/v105 y `MCM`. | Código 100 % propio basado en especificaciones públicas de formato (UESP). |

> **Nota sobre futura migración opcional de GUI a `PySide6`**: La capa `ui/` está desacoplada mediante `ProjectService`. Si en el futuro el usuario desea añadir un frontend alternativo en `PySide6` (Qt 6), deberá tenerse presente que `PySide6` está bajo **LGPL-3.0**, lo cual exige enlace dinámico (distribución en carpeta `--onedir` y no ejecutable monolítico bloqueado) e inclusión del texto LGPL-3.0. En la implementación actual **no se incorpora ninguna dependencia GPL ni LGPL**.
