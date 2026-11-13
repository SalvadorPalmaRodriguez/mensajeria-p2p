<!-- bmad:context -->
<!-- Verified 2026-11-13 against 2880e90. Managed by bmad-project-context; edits inside this block are replaced on refresh. Keep anything you want preserved outside the markers. -->

## mensajeria-p2p

Aplicación de mensajería P2P distribuida, en fase de planificación BMM (product brief cerrado; siguiente paso: PRD) — no hay código todavía. Stack previsto: Rust, arquitectura hexagonal (dominio / puertos / adaptadores). El registro autoritativo de decisiones, hipótesis y estado del proyecto es `docs/continuidad-proyecto.md`: léelo antes de trabajar en cualquier cuestión; si otro artefacto lo contradice, manda él.

## Policy

- Etiqueta siempre cada afirmación: HECHO / HIPÓTESIS / PROPUESTA / DECISIÓN APROBADA / DECISIÓN PENDIENTE (`docs/continuidad-proyecto.md` §3).
- Nunca conviertas propuestas en decisiones ni modifiques decisiones aprobadas sin autorización expresa de Salvador.
- No escribas código ni scaffolding hasta autorización expresa; la fase actual es planificación, no implementación (§16).
- Una cuestión cada vez: explica implicaciones y espera autorización antes de abrir una decisión arquitectónica nueva.
- Cita fuentes en afirmaciones técnicas importantes; si las fuentes chocan, prioriza la oficial e indícalo.
- No propongas pruebas con dispositivos físicos ni hardware adicional (regla 8, §3); SÍ permitidas VM Linux local + network namespaces. Las pruebas del descubrimiento usan la red Tor, igual que la app (DECISIÓN 2026-11-13); cómo conciliarlo con los tests deterministas es DECISIÓN PENDIENTE (§16): no la resuelvas por tu cuenta.
- Documentación y commits en español.
- La app debe funcionar sin root/admin; descartados kernel WireGuard, TUN, `wg-quick` y helpers privilegiados (DECISIÓN 2026-11-12, §14).
- Todas las dependencias de terceros deben ser open source; la licencia propia de la app es la de Enola, titular Salvador (DECISIÓN 2026-11-12, §14).
- Los mensajes viajan solo entre emisor y destinatario: nada de relays, TURN ni store-and-forward en intermediarios; una cola en el equipo del emisor sí está permitida (§5.5).

## Where things are

- Registro de decisiones y estado: `docs/continuidad-proyecto.md`
- Product brief cerrado, entrada del PRD: `_bmad-output/planning-artifacts/briefs/brief-mensajeria-p2p-2026-11-13/` (`brief.md`, `addendum.md`)
- Enola (reutilización autorizada por el titular): `../enola-cli-lite` — clon de referencia; verifica contra su código antes de afirmar nada sobre él.
- Artefactos de workflows BMAD: `_bmad-output/`; skills: `.agents/skills/` (se regeneran con `npx bmad-method install`).

## Running and verifying

- No existe build ni suite de tests — no hay código todavía; no inventes comandos.
- `npx bmad-method status` — verificar la instalación de BMAD.
- Cuando exista `Cargo.toml`: el pre-commit corre `cargo fmt --check` y `cargo clippy -- -D warnings`. Actívalo en cada clon con `git config core.hooksPath .githooks`.

## Conventions that differ from defaults

- TDD obligatorio: test primero (rojo → verde → refactor).
- Tests deterministas: sin sleeps ni dependencias de orden o timing; las condiciones de carrera se persiguen con ejecuciones repetidas, nunca se aceptan como "flaky".
- Tests de integración y e2e siempre que el componente lo permita, no solo unitarios.
- Todo archivo en `docs/` lleva cabecera YAML con `estado`, `ultima_actualizacion`, `proposito` y `autoridad`; actualízala siempre que edites el documento. Un doc `obsoleto` se conserva pero nunca es fuente de verdad.

## Known pitfalls

- No editar a mano `_bmad/config.toml` ni `_bmad/config.user.toml` — los regenera el instalador; overrides en `_bmad/custom/config.toml`.
- No editar `SKILL.md` dentro de `.agents/skills/` — se pierde al reinstalar; personalización vía `customize.toml` de cada skill.
- Las notas fechadas y de vigencia en `docs/` también llevan etiqueta (`> **HECHO (nota de vigencia, AAAA-MM-DD):** …`) — se escribieron sin ella y hubo que corregirlas.

<!-- /bmad:context -->

## Reglas propias del repositorio (no gestionadas por bmad)

- Commits sin atribución a IA: el autor de los commits es siempre Salvador. Prohibido añadir `Co-Authored-By` a cualquier bot/asistente y líneas tipo `Generated with [Devin](https://devin.ai)`. Lo hace cumplir el hook `.githooks/commit-msg` (actívalo con `git config core.hooksPath .githooks`).
- Identidad de autor: `Salvador Palma Rodríguez <salvadorpalmarodriguez@gmail.com>`. No usar el email corporativo (`@sngular.com`) en los commits.
