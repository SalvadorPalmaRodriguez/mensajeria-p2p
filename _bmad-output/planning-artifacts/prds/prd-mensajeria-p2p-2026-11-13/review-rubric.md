# PRD Quality Review — mensajeria-p2p

Revisión hecha por el propio redactor en el hilo principal (sin subagentes), el 2026-11-13. No es una revisión independiente.

## Overall verdict

El PRD sostiene una tesis clara (no depender de nadie vale más que la entrega garantizada) y casi todos sus requisitos proceden de decisiones expresas de Salvador, con consecuencias comprobables. El riesgo está en lo decidido deprisa al final: el límite de 100 mensajes y la eliminación en cascada tienen huecos de comportamiento que arquitectura heredará, y los criterios de éxito no los cubren.

## Decision-readiness — adequate

Las decisiones figuran como decisiones, con su origen. Las renuncias se nombran (sección 7; FR-30 recoge que Salvador eligió perder pendientes frente a bloquear la escritura).

### Findings
- **high** Los criterios de éxito no cubren el borrado ni la eliminación (sección 10) — SM-1 a SM-4 vienen del brief; las secciones 4.7 y 4.8 quedan fuera de la definición de éxito. *Fix:* decidir un quinto criterio (cuestión 17). Anotado con `[NOTE FOR PM]`.
- **medium** Orden entre los mensajes de los dos usuarios sin definir (FR-15) — solo se fija el orden por emisor. *Fix:* cuestión 13.

## Substance over theater — strong

Sin personas decorativas ni NFR genéricos; cada NFR tiene origen o una comprobación. NFR-9 es el más blando.

### Findings
- **low** NFR-9 («los avisos dicen qué pasa y qué hacer») es difícil de comprobar de forma exhaustiva. *Fix:* concretarlo en las historias, aviso por aviso.

## Strategic coherence — strong

Las funcionalidades siguen de la tesis y las contramétricas protegen justo lo que la tesis sacrifica (SM-C1, SM-C2).

## Done-ness clarity — adequate

Los 35 FR tienen al menos una consecuencia comprobable.

### Findings
- **high** Límite de conversación «en los dos equipos» con pendientes (FR-29, FR-30) — mientras hay pendientes los dos equipos no guardan el mismo conjunto, y el PRD no dice qué significa «el más antiguo» en cada uno. *Fix:* arquitectura (`addendum.md` §C); puede volver a Salvador.
- **medium** FR-32 «solo lo justo» no es comprobable tal cual. *Fix:* arquitectura define qué se conserva; entonces se añade la consecuencia.
- **medium** Ocho `[ASSUMPTION]` sin confirmar, dos con peso (FR-25 persistencia de la cola; FR-27 borrado de la conversación entera). *Fix:* confirmación de Salvador (sección 12).

## Scope honesty — strong

No-objetivos, fuera de alcance y limitaciones explícitos. Densidad de abiertos: 9 cuestiones para arquitectura, 8 para Salvador, 8 suposiciones, 1 nota. Alta para un PRD de lanzamiento, pero ninguna bloquea el paso a arquitectura.

## Downstream usability — adequate

Glosario presente; identificadores FR-1 a FR-35, UJ-1 a UJ-4, SM-1 a SM-4 y NFR-1 a NFR-10 contiguos; referencias cruzadas comprobadas.

### Findings
- **low** «En línea» aparece en la sección 7 (heredado del brief) sin estar en el glosario; se explica en la propia frase.

## Shape fit — strong

Producto de consola con un solo tipo de usuario: cuatro recorridos cortos con protagonistas nombrados y requisitos por capacidad. Proporcionado.

## Mechanical notes

- Índice de suposiciones: 8 en línea, 8 en el índice.
- Las referencias «§» apuntan al documento rector; las internas usan «sección N».
- Los nombres de los comandos no se fijan (convención de la sección 0).
