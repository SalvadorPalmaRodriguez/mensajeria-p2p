# Reconciliación de entradas — PRD mensajeria-p2p

Hecha en el hilo principal (sin subagentes) el 2026-11-13, contra `prd.md` y `addendum.md`.

## brief.md

- Problema, público, solución, diferenciadores: recogidos en las secciones 1, 2 y 8 del PRD.
- Limitaciones declaradas (4): recogidas en la sección 7, ampliadas con las que nacen del PRD.
- Criterios de éxito (4): SM-1 a SM-4, sin cambios de fondo.
- Alcance dentro/fuera: sección 9.
- Preguntas abiertas del brief:
  - «Conversación normal» comprobable: resuelta (FR-15, SM-1).
  - Emparejamiento desde consola: recorrido resuelto (UJ-1, FR-7 a FR-10); mecanismo remitido a arquitectura.
  - Éxito de la v1 = «completamente funcional»: sigue abierta (cuestión 14).
  - Tests deterministas con Tor: remitida a arquitectura (cuestión 1).
  - Salas de chat: fuera de alcance (sección 9.2).
  - Nombre y canal de distribución: fuera de alcance (sección 9.2).
- Hueco cualitativo: el tono del brief («el producto declara sus límites») se conserva en la visión y en la sección 7. La visión a largo plazo (público general, GUI, otras plataformas) no se repite en el PRD: vive en el brief y en §12 del documento rector.

## addendum.md del brief

- §A comparativa: no se repite; el PRD remite al brief.
- §B boceto de comandos: nombres ilustrativos (convención de la sección 0); estados pendiente/entregado formalizados en FR-14.
- §C para arquitectura: presencia (cuestión 5), tests con Tor (cuestión 1), tres equipos (SM-2).
- §D pendientes: las tres de D-16.6 y la presencia en la sección 11.1; fase de UX en la cuestión 15.
- §E matices: IP y custodia en la sección 7; robo del equipo en UJ-4 y la sección 7. La tensión «comunidad independiente frente a licencia no open source» no se recoge en el PRD: no afecta a los requisitos de la v1.

## docs/continuidad-proyecto.md

- Decisiones aprobadas respetadas; ninguna modificada. Las técnicas se enumeran como restricciones heredadas (sección 6).
- Punto a vigilar, no contradicción: §5.1 dice que no hay revocación remota. La eliminación en cascada (FR-33) es una limpieza cooperativa entre dos equipos emparejados y el PRD la declara no garantizada (sección 7).
