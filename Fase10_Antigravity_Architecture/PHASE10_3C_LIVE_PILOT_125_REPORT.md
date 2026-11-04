# FASE 10.3-C — INFORME DEL PILOTO TRANSVERSAL DE 125 CADENAS

**Fecha de ejecución (UTC):** `2026-11-04`  
**Trabajo:** `job_pilot_10_3c_luna125`  
**Modelo:** `gpt-6-luna`  
**Muestra:** 125 cadenas reales en cinco lotes de 25  
**Estado:** piloto completado con aceptación parcial; no habilita producción

## 1. Resumen ejecutivo

Se ejecutaron cinco solicitudes, una por lote, sin reintentos. Cuatro lotes superaron el Gatekeeper y se importaron en el almacén aislado (100 cadenas). El quinto recibió 25 traducciones, pero dos incumplieron el Glosario Maestro en el término `Elder Scroll` (`Pergamino Antiguo`). La política de aceptación atómica retuvo las 25 cadenas del lote: 23 quedaron como candidatas válidas y dos en cuarentena. El piloto produjo respuestas para 123 de las 125 cadenas; 100 se importaron, 23 siguen pendientes de decisión y dos requieren corrección.

**Evaluación:** éxito como prueba de operación y diagnóstico; aceptación incompleta como traducción integral de la muestra. La ejecución demuestra que el coste, el aislamiento y la revisión local funcionaron en este corte, pero no valida todavía una importación de extremo a extremo sin incidencias.

## 2. Resultado por lote

| Lote | Dominio | Recibidas | Aprobadas por Gatekeeper | Importadas | En cuarentena | Coste real (USD) |
|---|---|---:|---:|---:|---:|---:|
| `b01_dialogues_quests` | Diálogos y misiones | 25 | 25 | 25 | 0 | `$0.00216490` |
| `b02_mcm_ui` | Interfaz y MCM | 25 | 25 | 25 | 0 | `$0.00240810` |
| `b03_items_spells_perks` | Objetos, hechizos y ventajas | 25 | 25 | 25 | 0 | `$0.00206450` |
| `b04_books_notes` | Libros y notas | 25 | 25 | 25 | 0 | `$0.00301130` |
| `b05_scripts_other` | Scripts y otros | 25 | 23 | 0 | 2 | `$0.00169620` |
| **Total** |  | **125** | **123** | **100** | **2** | **`$0.01134500`** |

Las 23 candidatas del último lote no son importaciones aprobadas; deben permanecer en staging hasta resolver el lote y revisar su contexto. Los textos exactos de las dos traducciones rechazadas no quedaron preservados en el almacén reconstruido: el artefacto de respuesta del lote se regeneró desde `ExchangeStateStore` tras la cuarentena y muestra vacías esas dos traducciones. Por ello, el diagnóstico confirma el incumplimiento del glosario, pero no permite citar la formulación que lo causó.

## 3. Consumo y controles operativos

| Métrica | Resultado medido |
|---|---:|
| Solicitudes reales / reintentos | `5` / `0` |
| Tokens de entrada | `24,890` |
| Tokens de salida | `17,712` |
| Tokens totales | `42,602` |
| Coste real acumulado de este trabajo | **`US$0.01134500`** |
| Reserva máxima planificada | `US$0.01833539` |
| Límite del trabajo | `US$0.05` |

El gasto real quedó por debajo de la reserva planificada y del límite del trabajo. La reserva quedó conciliada, el candado global de API se restauró y el informe de ejecución registró que la base de producción no fue modificada. El trabajo utilizó almacenes aislados de staging, intercambio y presupuesto.

## 4. Hallazgos lingüísticos y de proceso

1. **Glosario — `Elder Scroll`:** dos cadenas fueron puestas en cuarentena con diagnóstico de incumplimiento de la forma esperada, `Pergamino Antiguo`. El término ya existe en la semilla del glosario. Como el texto exacto rechazado no se preservó en la reconstrucción del artefacto, no se puede precisar si fue una variante, un calco u otro problema. Hay que reconciliar el contexto y mejorar la retención de evidencia de cuarentena; no conviene relajar la regla.
2. **Nombre de mod:** el análisis detectó `HeelsFix Refresh` traducido como parte de `HeelsFix Refresh: Invalid Target`. El nombre debe conservarse y traducirse solo el texto descriptivo que lo acompaña.
3. **Naturalidad y longitud:** se señaló `Nightingale Armor Full Set` → `Conjunto completo de armadura de Ruiseñor` como expansión notable. Es una alerta de revisión de presentación, no un defecto automático: hay que validar uso, contexto y espacio disponible.
4. **Consistencia transversal:** el análisis posterior agrupó perfiles de registro y términos recurrentes (por ejemplo `Health`, `Magicka`, `Stamina`, `Conjure`, `Sneak` y nombres de lugares). Es un análisis heurístico útil para orientar una revisión editorial; no certifica por sí solo la naturalidad ni la armonía narrativa.

## 5. Decisiones y siguiente puerta

- Mantener la aceptación atómica del lote hasta que una revisión de dependencias demuestre que la importación parcial es segura. No convertir las 23 candidatas en aprobadas solo porque pasaron validaciones individuales.
- Reconciliar el contexto y la regla del glosario para los dos casos `Elder Scroll`; como su texto rechazado no está disponible para corrección manual, regenerar únicamente esas cadenas en staging y conservar el resultado original de cada revisión futura.
- Añadir protección determinista para nombres de mods identificados, empezando por `HeelsFix Refresh`, y verificar que el texto descriptivo sí siga traduciéndose.
- Revisar manualmente las 100 importadas y las 23 candidatas recuperables con foco en voz, sentido y contexto, no en neutralidad literal. Tratar las alertas de longitud como indicios sujetos a interfaz y legibilidad.
- Repetir una prueba dirigida de hasta 25 cadenas del dominio de scripts en staging aislado. La ampliación posterior depende de resolver las dos cuarentenas y comprobar las protecciones lingüísticas. La producción sigue pendiente de una puerta independiente.

## 6. Artefactos de evidencia

- Resumen de ejecución y conciliación: `NolvusAIT_PilotStaging/Phase10_3C_Luna125/staging_payloads/pilot_10_3c_live_execution_summary.json` (evidencia local; las bases de datos, payloads y credenciales no forman parte de este repositorio documental).
- Análisis lingüístico transversal: `NolvusAIT_PilotStaging/Phase10_3C_Luna125/staging_payloads/pilot_10_3c_cross_cutting_post_translation_analysis.md` (evidencia local).

**Resultado de fase:** `COMPLETADO CON ACEPTACIÓN PARCIAL`. **Producción:** no autorizada.
