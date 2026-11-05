> **GUÍA DE USUARIO — DOCUMENTO HISTÓRICO:** Esta guía describe la interfaz y los flujos de la aplicación durante las fases iniciales. Sus nombres de vistas, indicadores, rutas y procedimientos no se han revalidado contra el ejecutable actual. Para el estado vigente, restricciones y próximos pasos, consulta [CURRENT_PROJECT_STATE.md](../documentacion/HISTORICO.md) y [ROADMAP.md](./ROADMAP.md). No tomes los pasos de activación/exportación de este documento como autorización para modificar MO2 o Nolvus.

---

# GUÍA DE USUARIO Y OPERACIÓN SEGURA (`USER_GUIDE.md`)

**Manual paso a paso para operar `NOLVUS AI TRANSLATOR` con Nolvus Awakening 6.0.21 y Mod Organizer 2.**

---

## 1. Arranque de la Aplicación

Ejecuta `launch_gui.bat` (archivo del workspace local) con doble clic o abre una terminal PowerShell en la carpeta del proyecto:

```powershell
.venv\Scripts\python.exe -m nolvus_translator
```

En la esquina superior derecha verás siempre el indicador de seguridad:
> **`FASE SIN API DE PAGO — MockTranslationProvider ACTIVO`**

Durante las Fases 1 a 7, el sistema funciona 100 % en local sin consumir ni contratar ninguna API externa.

---

## 2. Recorrido por las 10 Secciones de la Interfaz Gráfica

### Pestaña 1: Dashboard
- Muestra tarjetas con las métricas globales de la instantánea activa: archivos inventariados, cadenas totales, cadenas ya en español, cadenas pendientes, traducciones aprobadas, entradas en cola de revisión y tamaño del Glosario / Memoria TM.
- **Acciones rápidas**:
  - **1. Autodetectar Nolvus / MO2**: busca instalaciones estándar de Nolvus Awakening en tus unidades locales (`C:\` a `Z:\`) sin modificar nada.
  - **2. Verificar Runtime y SkyrimSE.exe**: inspecciona en solo lectura la cabecera PE de `STOCK GAME\SkyrimSE.exe`, SKSE64, `Address Library` y `DynamicStringDistributor.dll`.
  - **3. Crear Sandbox Demo Sintético (Prueba Segura)**: genera una mini-instalación sintética en `workspace_data/demo_nolvus_sandbox` para probar todo el flujo antes de conectar tu instalación real.

### Pestaña 2: Escaneo
- Permite ejecutar el **Escaneo Completo de Solo Lectura** sobre el perfil de Mod Organizer 2 seleccionado o sobre una carpeta `Data` directa.
- Muestra en tablas gemelas:
  1. **Plugins activos** en orden exacto de carga (`load_order_index`), indicando cuáles tienen flag localizado (`0x80`) o ligero (`ESL 0x200`).
  2. **Inventario del Sistema de Archivos Virtual (VFS)**: ruta virtual ganadora, mod de origen, prioridad MO2, formato detectado, si procede del interior de un archivo `.BSA` (`v104`/`v105`) y cuántos otros mods fueron sobrescritos.

### Pestaña 3: Traducciones
- Tabla paginada de alto rendimiento (200 filas por página respaldada por índices SQLite) capaz de explorar cientos de miles de cadenas sin bloquear la interfaz.
- **Filtros combinables**: búsqueda de texto libre, dominio de contenido (*Misiones, Diálogos, Libros, Objetos, Magia/Perks, MCM, Scripts*), estado de clasificación (9 estados) y estado de revisión.
- **Botones de ejecución**:
  - **Traducir Pendientes (MockTranslationProvider)**: agrupa las cadenas por contexto narrativo (`QUST`, `DIAL->INFO`, `BOOK`, `MCM`), protege variables con `⟦VAR_i⟧`, aplica el Glosario Maestro y la Memoria de Traducción, y procesa los lotes guardando checkpoints en SQLite.
  - **Reanudar Último Trabajo Incompleto**: si pausaste o cancelaste un lote, continúa exactamente desde el último lote guardado sin repetir el trabajo previo.

### Pestaña 4: Glosario
- Gestiona el **Glosario Maestro compartido entre todos los mods de Nolvus** (pre-cargado con más de 70 términos canónicos de *The Elder Scrolls V: Skyrim* para `es-419`).
- Permite crear o editar términos indicando:
  - Categoría (*Personaje/NPC, Ciudad, Región, Localización, Facción, Raza, Criatura, Rango/Título, Artefacto, Magia, Culto/Religión*).
  - Regla de traducción (*Conservar sin traducir, Usar traducción oficial, Traducir título/rango, Traducir con glosario*).
- **Protección contra conflictos**: el sistema impide que procesos automáticos sobrescriban términos ya aprobados; también permite importar y exportar el glosario en JSON o CSV.

### Pestaña 5: Memoria de Traducción (TM)
- Permite buscar coincidencias **exactas** y **similares (difusas con RapidFuzz)** indicando el porcentaje de similitud.
- Incluye el botón **"Detectar Inconsistencias en TM"**, que identifica cuándo un mismo texto en inglés ha recibido distintas traducciones según el mod o tipo de registro (`SPEL` vs `INFO`) y muestra el historial de versiones (`tm_versions`).

### Pestaña 6: Revisión y Control de Calidad
- Cola dedicada para inspeccionar cadenas marcadas como `Pendiente`, `Dudosa`, `Rechazada` o `Requiere Investigación de Lore`.
- Muestra lado a lado:
  - Texto original en inglés y metadatos de contexto (`Mod`, `Plugin`, `FormID`, `Record/Subrecord`, `Hablante`).
  - Desglose explícito de la rúbrica de calidad (`Variables intactas`, `Cumplimiento de glosario`, `Ratio de longitud`, `Coherencia contextual`, `Registro es-419`).
  - Editor de traducción con validación en tiempo real: al pulsar **"Aprobar y Guardar en TM"**, el sistema verifica que no falte ninguna variable ni etiqueta antes de promover la cadena a `APPROVED`.

### Pestaña 7: Actualizaciones (Motor Incremental)
- Permite seleccionar una **Instantánea Anterior** y una **Instantánea Nueva** tras actualizar Nolvus Awakening o añadir mods.
- Calcula en segundos:
  - Archivos nuevos, modificados (por hash SHA-256) y eliminados.
  - Cadenas nuevas, cadenas cuyo texto original cambió, cadenas cuyo contexto cambió y **traducciones previas que siguen siendo 100 % válidas**.
- Propaga automáticamente todas las traducciones válidas al nuevo snapshot para que solo tengas que traducir las cadenas nuevas o modificadas.

### Pestaña 8: Exportación MO2 (con Vista Previa, Backup y Rollback)
- **Paso 1 — Vista Previa Obligatoria (Dry-Run)**: muestra exactamente qué archivos se generarán (`SKSE/Plugins/DynamicStringDistributor/<Plugin>/nolvus_ai_es419.json`, `Strings/<Plugin>_Spanish.*STRINGS`, `Interface/Translations/<Mod>_SPANISH.txt` en `UTF-16 LE BOM` y opcionalmente `.pex` reconstruidos) sin escribir aún en disco.
- **Paso 2 — Generar Mod Independiente**: construye la carpeta del mod `Nolvus AI Spanish Translation` con su `meta.ini` y `translation_manifest.json` (con hashes SHA-256). Si ya existía una versión previa del paquete, crea primero una copia de seguridad automática en `workspace_data/backups/`.
- **Paso 3 — Rollback en 1 Clic**: selecciona cualquier exportación del historial y pulsa **"Revertir Exportación Seleccionada (Rollback)"** para eliminar el paquete generado y restaurar instantáneamente la versión anterior.

### Pestaña 9: Informes y Estimación Económica (Fase 8)
- **Informe de Cobertura Real**: desglosa la cobertura efectiva global, por dominio y por mod, descontando honestamente los formatos binarios no soportados (como `.swf` compilados) y las cadenas pendientes. Permite exportar a **Markdown (`.md`), JSON (`.json`) y Excel (`.xlsx`)**.
- **Analizador de Volumen y Costo (Preparado para Fase 8)**: calcula el número exacto de cadenas únicas pendientes (deduplicadas por TM y Glosario), caracteres, lotes y tokens de entrada/salida estimados, con una calculadora local de tarifas verificables (+15 % de margen de revisión).

### Pestaña 10: Configuración
- Configura las rutas de Nolvus Awakening, `STOCK GAME`, `Mod Organizer 2` y el perfil activo.
- Permite ajustar tamaño de lote (`batch_size`), reintentos máximos, umbral de auto-aprobación QA y activar/desactivar selectivamente cada formato de exportación (`DSD JSON`, `STRINGS`, `MCM UTF-16LE`, `PEX AST`).

---

## 3. Procedimiento Seguro para Probar el Paquete en Mod Organizer 2

Cuando desees probar el paquete generado en Mod Organizer 2:

1. **Clona tu perfil en MO2 (Obligatorio)**:
   - Abre Mod Organizer 2 en tu instancia de Nolvus Awakening.
   - Haz clic en el desplegable **Perfil -> Gestionar... -> Seleccionar `Nolvus Awakening` -> Copiar** y nómbralo **`Nolvus Awakening - Test ES419`**.
   - **Nunca realices pruebas iniciales sobre tu perfil principal de juego.**
2. **Verifica la compatibilidad de `DynamicStringDistributor.dll` en la Pestaña 1**:
   - Revisa el informe de **"Verificar Runtime y SkyrimSE.exe"**:
     - Si tu `STOCK GAME\SkyrimSE.exe` es `1.5.97.0` o `1.6.1170.0`, utiliza la versión estándar de DSD + Address Library correspondiente a ese runtime.
     - Si tu `STOCK GAME\SkyrimSE.exe` es `1.7.104.0`, asegúrate de contar con **Address Library All-in-One v13+ (`versionlib-1-7-104-0.bin`)** y el binario de DSD recompilado para `1.7.104.0` (véase [COMPATIBILITY.md](COMPATIBILITY.md)).
3. **Instala o vincula la carpeta `Nolvus AI Spanish Translation`**:
   - Copia la carpeta generada en `workspace_data/output/mo2_mods/Nolvus AI Spanish Translation` dentro de `MODS/mods/Nolvus AI Spanish Translation` (o selecciónala como destino de exportación en una subcarpeta dedicada de mods nuevos) y actívala al final de la lista izquierda de MO2 en tu perfil clonado **`Nolvus Awakening - Test ES419`**.
4. **Reversión instantánea**:
   - Desmarca la casilla de `Nolvus AI Spanish Translation` en MO2 o pulsa **"Revertir Exportación (Rollback)"** en la Pestaña 8 de la aplicación. Ningún archivo original de Nolvus habrá sido modificado.
