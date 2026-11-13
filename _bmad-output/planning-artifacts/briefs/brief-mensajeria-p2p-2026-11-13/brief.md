---
title: "Product Brief: mensajeria-p2p"
status: final
created: 2026-11-13
updated: 2026-11-13
---

# Product Brief: mensajeria-p2p

> Manda `docs/continuidad-proyecto.md`; las referencias «§» apuntan a él. «DECISIÓN BRIEF» marca lo decidido por Salvador el 2026-11-13 y aún no registrado allí. Detalle y fuentes en `addendum.md`.

## Resumen ejecutivo

Hoy, para hablar en privado con alguien hay que confiar en un tercero: una empresa, sus servidores, su registro y su política de datos. mensajeria-p2p (nombre provisional) es una aplicación de mensajería para Linux en la que ese tercero no existe. Dos personas se instalan la app, se emparejan en persona y a partir de ahí se escriben directamente de un equipo al otro. No hay cuenta, registro, teléfono ni cuota, y ningún mensaje pasa por una máquina ajena ni se guarda en ella.

La primera versión es deliberadamente pequeña: una aplicación de consola, texto entre dos personas, para usuarios de Linux que ya se ven en persona. Su objetivo es demostrar que dos equipos se emparejan y conversan, probado con tres equipos y todas sus parejas. El producto declara sus límites: tus contactos conocen tu IP, no hay entrega garantizada y algunos contactos no serán alcanzables.

## El problema

Los servicios de mensajería actuales exigen confiar en alguien a quien no conoces: empresas cuyos servidores ven pasar el tráfico, altas que piden un registro, software de servidor que nadie de fuera puede verificar en ejecución y políticas que permiten entregar datos a las autoridades. Tus conversaciones dependen de la continuidad y la política de ese tercero (DECISIÓN BRIEF).

Las alternativas lo resuelven solo en parte (HECHO, fuentes en el addendum §A): Signal depende de servidores centrales y pide un teléfono; Telegram puede revelar IP y teléfono ante una orden judicial; Jami depende por defecto de servidores de la empresa que lo desarrolla; Briar, el más cercano, no tiene servidor ni registro, pero transporta los mensajes por los relays de Tor y admite contactos a distancia.

## Para quién

**Primera versión.** El propio creador y las personas con las que acuerde usarla: usuarios de Linux, cómodos en una terminal, que se conocen en persona. El círculo crece de encuentro en encuentro y el valor existe desde la primera pareja; no hace falta masa crítica. Podrá usarla cualquiera que la descargue del canal que el titular decida (DECISIÓN BRIEF).

**No es para** quien necesite contactar con desconocidos, entrega garantizada o uso desde el móvil.

## La solución

1. **Emparejar.** Dos personas, juntas físicamente, emparejan sus dispositivos. Cada instalación tiene su propia identidad (DECISIÓN APROBADA, §5.1–5.2).
2. **Hablar.** Por comandos de consola: listar contactos, enviar un mensaje suelto o abrir una conversación por líneas con un contacto (DECISIÓN BRIEF).
3. **Si el otro no está.** El mensaje queda pendiente en el equipo del emisor y se entrega cuando la app detecta al contacto, mostrando cuándo se escribió (DECISIÓN BRIEF; sin garantía de entrega, §5.4).

Los mensajes viajan solo entre los dos dispositivos. La red Tor se usa únicamente para que los dispositivos se encuentren, nunca para transportar ni almacenar mensajes (DECISIÓN APROBADA, §4 y D-16.6).

## Qué lo hace diferente

- **Nadie a quien no conozcas puede escribirte.** No hay directorio, búsqueda ni contactos indirectos: tus contactos son personas con las que te has emparejado en persona (DECISIÓN APROBADA, §5.3 y §6).
- **No hay servidor del que fiarse.** Ni cuenta, ni registro, ni teléfono, ni cuota (§4).
- **El mensaje solo toca los dos dispositivos.** En Signal pasa por servidores centrales y en Briar por relays de Tor (§5.5).
- **Verificación presencial del emparejamiento.** Sabes con qué dispositivo te emparejas porque lo tienes delante. No se promete nada sobre quién lo porta después.
- **Auditable.** El código es visible (§14). Auditable no es auditado: ningún componente tiene hoy auditoría de seguridad pública (HECHO, §5.8).

## Limitaciones declaradas

- **Tus contactos conocen tu IP y saben cuándo estás en línea.** Solo ellos: es el precio de la conexión directa y de la detección de presencia.
- **No hay entrega garantizada.** Un mensaje pendiente solo sale si la app del emisor sigue en marcha cuando el contacto aparece (§5.4), y algunos contactos serán inalcanzables, en especial detrás del filtrado de los operadores móviles (§9).
- **Si pierdes el dispositivo, pierdes su identidad y sus contactos.** Si te lo roban, el portador puede hacerse pasar por ti hasta que cada contacto lo revoque (§5.1, §5.7).
- **Un contacto que custodia tu registro de conectividad puede inferir con quién te relacionas** (§7.5).

## Criterios de éxito

La v1 ha salido bien si se cumplen los cuatro (DECISIÓN BRIEF):

1. Dos equipos se emparejan y mantienen una conversación normal de texto, de ida y vuelta.
2. Funciona con un mínimo de tres equipos y todas sus parejas (A–B, A–C, B–C).
3. Un mensaje enviado con el contacto ausente queda pendiente en el equipo del emisor y se entrega cuando el contacto aparece.
4. Se comprueba que ningún tercero transporta ni almacena el contenido y que nada exige root (§5.5, §14–15).

Los «equipos» son máquinas virtuales y network namespaces (regla 8, §3), con la red Tor para el descubrimiento, que simulan topologías pero no routers ni operadores reales: el éxito queda demostrado en simulación.

No hay objetivos de adopción ni de negocio (§4, §14).

## Alcance de la v1

**Dentro:** Linux sin root (§12, §14–15); consola por comandos; emparejamiento presencial; texto entre dos personas; cola local en el equipo del emisor con detección de presencia; Modalidad A (§6).

**Fuera:** grupos; imágenes y archivos; interfaz gráfica; Modalidad B; Windows, macOS y móviles (§12); IPv4 (§5.6); canal de distribución (§4). La recuperación de identidad y las introducciones de contactos están excluidas por diseño, no solo de la v1 (§5.1, §5.3).

## Visión

Cuando la versión de Linux sea completamente funcional, el producto se extiende a Windows y macOS y, más adelante, a móviles (DECISIÓN APROBADA, §12). Por el camino: interfaz gráfica, imágenes y archivos, IPv4 mediante un adaptador separado (§5.6) y la Modalidad B como participación voluntaria (§6). Tal vez, en un futuro, salas de chat con varios equipos; hoy no existen ni tienen forma definida (DECISIÓN BRIEF).

El horizonte es una aplicación para el público general: una red que crece de encuentro en encuentro, sin compañías, servidores, registros ni cuotas, y sin que nadie la gestione.

## Preguntas abiertas

- Qué es «una conversación normal» en términos comprobables (PRD).
- Cómo se empareja en presencia desde una consola (§5.2 no elige mecanismo).
- Si el éxito de la v1 equivale a «completamente funcional» (§12).
- Cómo se hacen deterministas las pruebas que dependen de la red Tor (las pruebas del descubrimiento usan Tor, igual que la aplicación).
- Qué forma podrían tener las salas de chat sin chocar con §5.3 y §5.5.
- Nombre del producto y canal de distribución.

Las cuestiones de arquitectura y de proceso pendientes están en el addendum §D.
