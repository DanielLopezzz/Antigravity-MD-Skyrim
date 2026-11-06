# Preparación de prueba pareada de contexto — 2026-11-05

**Paquete preparado y validado offline; no se ejecutó una traducción ni una IA real.** El API Lock sigue activo. No cambia producción, Nolvus, TM, glosario ni las decisiones del visor. Esta preparación reemplaza como propuesta de siguiente experimento al diseño anterior de 12 casos y cuatro controles; ambos permanecen conservados.

## Diseño concreto

- **24 cadenas reales de 22 entradas de mod/origen**, cuatro por perfil: diálogo, misión, libro, objeto, magia y mensaje de script. Dos piezas ya localizadas se incluyen como controles de referencia; no se presentan como traducciones nuevas pendientes.
- **Dos variantes por cadena:** catálogo contextual anterior, frente a catálogo documental ampliado más registro de nombres. Mismo modelo/prompt/glosario actual en ambas; se mide el cambio de contexto suministrado, no el efecto aislado de cada cambio de código.
- **Ocho controles sintéticos:** cuatro errores introducidos y cuatro traducciones válidas. Sus etiquetas esperadas están en un archivo privado separado, nunca en la solicitud del juez.
- Las cuatro piezas del conjunto enano se traducen juntas en cada variante para evaluar denominación compartida y concordancia. No se mezclan como una familia cuatro objetos que solo coinciden por categoría.
- Se incluyen referencias de lugares y texto narrativo. Los casos guardados para investigación, las cuarentenas técnicas y los títulos internos de misión se excluyen de esta selección. Los mensajes de scripts proceden de llamadas registradas de mensaje/notificación, sin concluir que todo texto de script sea traducible.

El contexto cambia efectivamente en **19/24 pares**; cinco pares sin diferencia sirven como comparación frente a variación del modelo. Hay documentación de autor/resúmenes identificados en **8/24 casos** del brazo ampliado. Estos recuentos miden suministro de contexto, no relevancia o calidad acreditada. Las versiones web distintas o desconocidas siguen señaladas.

## Solicitudes y límites propuestos

| Etapa | Solicitudes máximas |
|---|---:|
| Traducción: 24 cadenas por variante, conjunto de cuatro en un lote | 42 |
| Revisión ciega de las 48 traducciones resultantes | 48 |
| Revisión de controles sintéticos | 8 |
| Posedición focalizada y recomprobación | 4 + 4 |
| **Total máximo** | **106** |

Hay 42 solicitudes exactas de traducción y ocho de controles preparadas. Las 48 revisiones naturales se deben construir **después** de recibir y validar las traducciones, vinculadas a sus hashes reales. No se fabricó una respuesta futura para simular que esas revisiones ya están selladas. El juez debe desconocer variante y veredicto previo; para la familia de equipo necesita los otros nombres traducidos como referencias vinculadas a recibos válidos.

Límite por solicitud: **16.384 bytes**, salida máxima **2.048 tokens**, una solicitud concurrente y sin reintento automático. Techo conservador agregado: 1.790.976 tokens de entrada y 217.088 de salida. La reserva estimada con la tarifa local registrada y margen del 15 % es **US$0,330787840**, bajo un **tope nuevo propuesto de US$0,35**. No es cotización vigente, cargo, reserva real ni aumento autorizado del presupuesto anterior. Debe verificarse la tarifa antes de aprobar gasto.

## Qué se comprobará

1. Fidelidad: identidad, cantidades, negación, agencia, condiciones y ausencia de adiciones.
2. Naturalidad ES-419 y voz, separadas de preferencias estilísticas menores.
3. Misma denominación de lugares y objetos entre registros; concordancia dentro del conjunto.
4. Contexto pertinente, contradictorio o insuficiente; no adoptar datos de otra versión como corrección.
5. Cuatro errores controlados detectados y ninguna falsa alarma mayor/crítica sobre los cuatro controles válidos.
6. Ningún error mayor/crítico introducido por una posedición. Cero aprobaciones automáticas.

La comparación humana debe ser ciega, con evidencia y sin tratar una aprobación anterior como verdad universal. Se registrarán tiempo de investigación y correcciones; no se atribuirá ahorro humano a contadores de cola. Esta muestra no calibra aprobación automática ni permite extrapolar tasas al inventario completo.

Se detiene ante uso incierto, desviación de presupuesto/modelo/herramientas, cambio de fuentes, fallo crítico de identidad/estructura o más del 10 % de respuestas malformadas, rechazadas o incompletas. Se conserva el recibo y no se retransmite por defecto.

## Estado operativo y trazabilidad

La carpeta vigente es `private/prepared_final_v4`. Los ensayos preliminares se conservaron y no deben ejecutarse como si fueran la propuesta final. El plan, sus artefactos y las fuentes están vinculados por SHA-256. Sello del plan: `80aa5cb3732cd532ac5365fdf7fba12e6960b7a68e678dec725e839809270d1a`.

La preparación descubrió que la etiqueta interna del inventario del juego base activaba el guard de datos locales. Se usaron identificadores opacos y una etiqueta pública de Skyrim en la solicitud; la búsqueda contextual conserva internamente el origen real. No se relajó el control de seguridad ni se cambió el texto original.

La preparación no carga credenciales y usa transporte bloqueado en etapa de construcción. `execution_authorized=false`, `execution_ready=false`, cero HTTP a proveedores y cero créditos. **Antes de ejecutar siguen pendientes la validación del transporte real del juez y de sus etapas durables, la tarifa vigente y autorización específica de API.** Preparar el paquete no habilita esos pasos. El etiquetado natural con evidencia y el enlace del juez a respuestas todavía inexistentes también están explícitos en el plan.

Los previews, las fuentes, los controles y la asociación de identificadores permanecen privados. `private/validate_preparation.py` permite verificar de nuevo el sello y los insumos sin ejecutar solicitudes. Las revisiones activas no se regeneraron ni se sustituyeron.
