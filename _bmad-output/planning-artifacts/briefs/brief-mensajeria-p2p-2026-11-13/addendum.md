---
title: "Addendum: Product Brief mensajeria-p2p"
status: final
created: 2026-11-13
updated: 2026-11-13
---

# Addendum — profundidad para PRD y arquitectura

## A. Comparativa con alternativas (consultado 2026-11-13)

| Producto | Hecho relevante | Fuente |
|---|---|---|
| Signal | Cliente y servidor publicados bajo AGPLv3; no es verificable qué se ejecuta en sus servidores. | https://github.com/signalapp/Signal-Server/ |
| Signal | El repositorio del servidor estuvo ~1 año sin publicar cambios (abr-2020 → abr-2021). | https://github.com/signalapp/Signal-Android/issues/11101 |
| Signal | Alta con número de teléfono capaz de recibir SMS; en beta Android, alta sin número con pago único de 2,99 USD. | https://support.signal.org/hc/en-us/articles/360007318691 · https://support.signal.org/hc/en-us/articles/11197884108826 |
| Telegram | Desde sep-2024 su política permite revelar IP y teléfono ante orden judicial válida. | https://www.theverge.com/2024/9/23/24252276/telegram-disclose-user-data-legal-requests-criminal-activity |
| Jami | Por defecto: bootstrap `bootstrap.jami.net`, TURN `turn.jami.net` activado y servidor de nombres `ns.jami.net`, operados por Savoir-faire Linux. | https://jami.net/why-is-jami-truly-distributed/ · https://jami.net/establishing-peer-to-peer-connections-with-jami/ |
| Briar | Con internet, sincroniza los mensajes a través de Tor. | https://www.briarproject.org/how-it-works/ |
| Briar | Modo mantenimiento desde 2026-07-09; citan UX difícil para añadir contactos y comunicarse offline. | https://briarproject.org/news/2026-maintenance-mode/ |
| Briar | Sin servidor central; la cuenta se guarda en el dispositivo, no en la nube (sin registro). Admite añadir contactos a distancia (enlace `briar://`) y por introducción de un contacto común. | https://briarproject.org/manual/ |
| Briar Desktop | Beta; última versión 0.6.5-beta (feb-2026); hitos posteriores vencidos sin completar. | https://briarproject.org/download-briar-desktop/ · https://code.briarproject.org/briar/briar-desktop/-/milestones |

Correcciones registradas en la sesión: Briar **no** usa registro ni servidores propios; lo que usa son relays de Tor para transportar los mensajes. Signal **no** es código cerrado; el argumento válido es la imposibilidad de verificar el servidor en ejecución.

## B. Boceto de la interfaz de consola (ilustrativo, no especificación)

```
app contacts            # listar contactos
app send salvi "hola"   # enviar un mensaje suelto
app chat salvi          # sesión de conversación por líneas (sin pantalla completa)
```

Estados ilustrativos de la cola local: `⏳ pendiente en este equipo` → `✓ entregado (escrito hace 3 h)`.

DECISIÓN (brief 2026-11-13): `chat` abre una sesión interactiva por líneas, sin pantalla completa. Los nombres de los comandos siguen siendo ilustrativos.

## C. Para arquitectura (no son decisiones)

- HIPÓTESIS: la presencia se puede inferir de la alcanzabilidad del endpoint rendezvous derivado por pareja (D-16.6), sin mecanismo nuevo. Patrón análogo en Briar (offer/request/ack al conectar): https://codeberg.org/tqt/briar/wiki/A-Quick-Overview-of-the-Protocol-Stack
- La interfaz de consola es un adaptador más (§11); Enola tiene una capa `cli`, reutilizable por la vía C de reutilización aprobada en §14; verificar contra su código antes de afirmar nada.
- DECISIÓN BRIEF (2026-11-13): las pruebas del descubrimiento usan la red Tor, igual que la aplicación; la condición «sin tráfico externo» se retiró de la regla 8 del documento rector. Queda pendiente conciliarlo con la convención de tests deterministas: los tiempos de los servicios onion dependen de una red ajena. HIPÓTESIS sin verificar: una red Tor privada local para los tests automáticos y la red real para la comprobación extremo a extremo.
- Tres equipos es el mínimo que ejercita la custodia en contactos mutuos (D-16.6).
- Salas de chat futuras (análisis, no decisión): una sala entre equipos todos emparejados entre sí es compatible con §5.3. Una sala con miembros no emparejados entre sí exigiría introducciones (prohibidas, §5.3) o que alguien reenvíe mensajes (prohibido, §5.5).

## D. Cuestiones pendientes fuera del brief

- Arquitectura: las tres cuestiones de diseño de D-16.6 (autorización en custodia, rotación del rendezvous, formato del registro) y el mecanismo de presencia.
- Proceso: registrar en el documento rector las decisiones tomadas en el brief, incluida la aclaración de §5.5 (una cola en el equipo del emisor está permitida; la prohibición de store-and-forward se refiere a intermediarios). Pendiente de autorización de Salvador.
- Proceso: evaluar al cerrar el PRD si la fase de UX (`bmad-ux`) se omite o se aligera, dado que la v1 es una consola por comandos (PROPUESTA).

## E. Matices de las limitaciones y de la comparativa

- IP: D-12.1 y D-12.2 limitan la información de conectividad a los contactos directos con independencia de la modalidad A/B. Briar, al pasar por Tor, oculta la IP también a los contactos.
- Briar es P2P y no tiene servidor central. La diferencia defendible es conexión directa por internet frente a conexión retransmitida por Tor, y contactos solo presenciales frente a contactos a distancia y por introducción.
- Robo del dispositivo: no hay revocación ni recuperación remota (§5.1); cada contacto debe revocar el dispositivo por su cuenta (§5.7).
- Custodia: la inferencia de relaciones por correlación temporal no es eliminable con el modelo actual; no se promete anonimato frente al contacto que custodia (§7.5).
- Distribución: la app la distribuye el titular por el canal que decida; los usuarios no se la pasan entre sí, lo que mantiene la licencia de §14 sin reabrirla.
- Tensión anotada: el horizonte de «comunidad independiente» convive con una licencia source-visible que no es open source (§14).
