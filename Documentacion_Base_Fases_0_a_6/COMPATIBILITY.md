> **COMPATIBILIDAD — NOTA DE VIGENCIA:** Este documento fue creado como matriz de compatibilidad en una fase inicial y su título hace referencia a Skyrim 1.7.104.0. **No debe interpretarse como la versión actual de STOCK GAME.** El informe de Fase 9 del 2026-11-03 reporta `SkyrimSE.exe 1.5.97.0` para el corte auditado. La versión efectiva debe volver a leerse del ejecutable real antes de cualquier prueba o exportación. Las afirmaciones sobre SKSE, Address Library y DSD deben confirmarse contra ese runtime y las versiones instaladas.

---

# Matriz de Compatibilidad Técnica: Skyrim 1.7.104.0, Nolvus Awakening 6.0.21 y MO2

Este documento establece las reglas técnicas verificadas sobre compatibilidad en tiempo de ejecución, codificación de archivos y seguridad sobre la instalación de **Nolvus Awakening 6.0.21**.

---

## 1. Distinción Crítica: Versión del Dashboard vs. Steam vs. `STOCK GAME`

Uno de los errores más graves en herramientas de modding para listas automatizadas es confundir tres versiones distintas:

1. **Versión de Nolvus Dashboard (`6.0.21`)**: Versión del instalador/manifiesto de la modlist Nolvus Awakening. **No** representa la versión del motor de Skyrim.
2. **Instalación base de Steam (`SkyrimSE.exe` reportado en `1.7.104.0`)**: Ejecutable actualizado por Steam (Anniversary Edition build `1.7.104.0`).
3. **Instalación aislada `STOCK GAME` (`.../Nolvus/Instances/Nolvus Awakening/STOCK GAME/SkyrimSE.exe`)**:
   * Nolvus Dashboard crea una copia independiente del juego en `STOCK GAME` y ejecuta su propio *Downgrade Patcher* (históricamente fijando el ejecutable efectivo de `STOCK GAME` en **`1.5.97.0`** o **`1.6.1170.0`**, conservando los archivos de contenido AE/Creation Club).
   * Sin embargo, si el usuario tiene una variante personalizada o una actualización reciente donde `STOCK GAME\SkyrimSE.exe` permanece en **`1.7.104.0`**, las exigencias de plugins nativos SKSE cambian radicalmente.

### Verificación Automática Obligatoria (`pe_inspector.py`)
`NOLVUS AI TRANSLATOR` **nunca asume** la versión del runtime. Al analizar la instalación:
1. Lee directamente la estructura binaria Win32 PE (`VS_FIXEDFILEINFO`) de `STOCK GAME\SkyrimSE.exe`.
2. Enumera las librerías `skse64_*.dll` presentes en `STOCK GAME\`.
3. Inspecciona en el VFS de MO2 los archivos `SKSE/Plugins/versionlib-*.bin` (*Address Library for SKSE Plugins*).
4. Inspecciona `SKSE/Plugins/DynamicStringDistributor.dll` (y su versión PE / hash SHA-256).

---

## 2. Particularidades del Runtime `1.7.104.0` y Dynamic String Distributor (DSD)

### Escenario A: `STOCK GAME\SkyrimSE.exe` es `1.7.104.0`
Las builds `1.7.99` y `1.7.104.0` de Skyrim introdujeron cambios de layout en memoria que rompieron plugins SKSE compilados contra versiones previas de CommonLibSSE:
* **SKSE64**: Requiere la build específica de SKSE64 para `1.7.104.0` (`skse64_1_7_104.dll`).
* **Address Library**: Requiere **Address Library All in One v13+** (soporte de estructuras **Format 5**, incluyendo `versionlib-1-7-104-0.bin`).
* **Dynamic String Distributor (DSD)**: La versión base histórica de DSD falla al enganchar offsets en `1.7.104.0`. Requiere instalar el mod base DSD **más** el parche recompilado con CommonLibSSE-NG actualizado (**`DynamicStringDistributor-1.7.104-Patch`**) cargando después del mod base en MO2.

### Escenario B: `STOCK GAME\SkyrimSE.exe` fue degradado por Nolvus a `1.5.97.0` o `1.6.1170.0`
* **PROHIBIDO instalar DLLs de `1.7.104.0`** en un `STOCK GAME` `1.5.97.0` o `1.6.1170.0`, ya que impedirían el arranque de SKSE.
* Se debe utilizar la build estándar de DSD y el `versionlib-1-5-97-0.bin` o `versionlib-1-6-1170-0.bin` ya incluido en la instancia de Nolvus Awakening.

### Estrategia Segura de Exportación de Nuestra Herramienta
* Nuestro mod generado (`Nolvus AI Spanish Translation`) **solo contiene archivos de datos puros** (`.json` para DSD, `.STRINGS`/`.DLSTRINGS`/`.ILSTRINGS`, `.txt` UTF-16LE para MCM, y opcionalmente `.pex` verificados). **Jamás empaqueta ni sobrescribe archivos `.dll` de SKSE**, evitando cualquier riesgo de rotura binaria del runtime.
* Si el usuario aún no tiene `DynamicStringDistributor.dll` activo en su perfil de prueba de MO2, el validador de compatibilidad de la aplicación emite una alerta explícita indicando qué versión exacta de DSD necesita según el `SkyrimSE.exe` real detectado en `STOCK GAME`.

---

## 3. Interacciones Conocidas con Otros Frameworks de Nolvus Awakening

1. **Description Framework (`BOOK CNAM`)**:
   * *Comportamiento*: Las entradas DSD sobre `BOOK CNAM` suelen sobrescribir las inyecciones dinámicas de Description Framework.
   * *Mitigación*: Nuestro exportador DSD permite excluir selectivamente `BOOK CNAM` cuando detecta configuraciones activas de Description Framework (`*_DISTR.ini`) para el mismo objeto, y en su lugar inventaría los archivos `*_DISTR.ini` como recursos auxiliares traducibles.
2. **SkyPatcher**:
   * *Comportamiento*: Las reglas `.ini` de SkyPatcher que modifican nombres (`FULL`) tienen precedencia en tiempo de ejecución sobre DSD.
   * *Mitigación*: El escáner VFS detecta archivos `SKSE/Plugins/SkyPatcher/**/*.ini` para inventariar posibles sobrescrituras de nombres.
3. **Norden UI / Fuentes Scaleform (`Interface/fontconfig.txt`)**:
   * *Comportamiento*: Nolvus Awakening 6.0.21 incorpora interfaces personalizadas (como *Norden UI*). Si una fuente Flash (`.swf`) mapeada en `fontconfig.txt` carece de glifos españoles (`á, é, í, ó, ú, ñ, ü, ¿, ¡`), el juego muestra cuadrados vacíos (`[]`).
   * *Mitigación*: El inspector de compatibilidad comprueba la existencia y mapeos de `Interface/fontconfig.txt` en el VFS ganador de MO2 y verifica `sLanguage=SPANISH` en `Skyrim.ini` del perfil de MO2.

---

## 4. Requisitos Estrictos de Codificación por Formato en Skyrim SE/AE

| Formato | Ruta Relativa en `Data` (VFS MO2) | Codificación Obligatoria | Consecuencia de Codificación Incorrecta |
| :--- | :--- | :--- | :--- |
| **Tablas Localizadas** (`.STRINGS`, `.DLSTRINGS`, `.ILSTRINGS`) | `Strings/<Plugin>_Spanish.<ext>` | **UTF-8** (sin BOM; `DL`/`IL` con prefijo `uint32` longitud incluyendo `\0`) | Caracteres corruptos (*mojibake*) o CTD al abrir libros/diálogos si el prefijo `uint32` es erróneo. |
| **Traducciones MCM / SkyUI** (`.txt`) | `Interface/Translations/<Mod>_SPANISH.txt` | **UTF-16 Little Endian con BOM (`FF FE`)** y separador **TAB (`\t`)** | SkyUI/MCM Helper ignora el archivo y muestra tokens crudos `$Mod_Setting` en pantalla. |
| **Parches DSD** (`.json`) | `SKSE/Plugins/DynamicStringDistributor/<Plugin>/<archivo>.json` | **UTF-8** (sin BOM, JSON válido) | DSD rechaza el archivo JSON en `DynamicStringDistributor.log` y deja el texto en inglés. |
| **Scripts Papyrus** (`.pex`) | `Scripts/<ScriptName>.pex` | Big-Endian (`0xFA57C0DE`), cadenas `uint16 len` + **UTF-8** | Fallo de carga del script en la máquina virtual Papyrus si se altera un identificador o `Object.size`. |

---

## 5. Protocolo de Prueba en Perfil Clonado de MO2

Antes de activar cualquier paquete generado en la partida principal:
1. Abrir MO2 de Nolvus Awakening y **clonar/copiar** el perfil principal a un perfil aislado (ej. `Nolvus Awakening - Test ES`).
2. Instalar la carpeta generada `Nolvus AI Spanish Translation` al final del panel izquierdo de MO2 (máxima prioridad de archivos sueltos) únicamente en el perfil clonado.
3. Probar primero con un lote reducido de 3 a 5 mods representativos (un mod con MCM, un mod de misión/diálogos, un mod de objetos).
4. Para revertir al 100 %: basta con desmarcar la casilla de `Nolvus AI Spanish Translation` en el panel izquierdo de MO2. Ningún archivo original de Nolvus ni de `STOCK GAME` habrá sido tocado.
