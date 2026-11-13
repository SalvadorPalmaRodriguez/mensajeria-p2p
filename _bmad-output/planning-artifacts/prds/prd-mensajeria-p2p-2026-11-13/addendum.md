---
title: "Addendum: PRD mensajeria-p2p"
status: draft
created: 2026-11-13
updated: 2026-11-13
---

# Addendum — profundidad para arquitectura

> Manda `docs/continuidad-proyecto.md`; las referencias «§» apuntan a él. Nada de lo recogido aquí es una decisión salvo que lleve la etiqueta DECISIÓN APROBADA.

## A. Mecanismo de emparejamiento presencial (DECISIÓN PENDIENTE, para arquitectura)

Aportado por Salvador el 2026-11-13 durante el PRD. No elige mecanismo; fija el criterio y nombra candidatos.

- Criterio de Salvador: «algo clásico pero funcional y seguro».
- Restricción reafirmada por Salvador: el emparejamiento es presencial —las dos personas juntas, cada una con su equipo Linux— sea por código, QR u otro mecanismo. Un mecanismo que permita emparejar a distancia queda excluido (§5.2).
- Candidato: QR. Objeción de Salvador: leer un QR con un ordenador es complicado.
- Candidato: código (introducido o comparado entre los dos equipos).
- Candidato: localización por proximidad del otro dispositivo con la aplicación de consola abierta, una vez lanzado el comando.
- HECHO: §5.2 exige protección frente a MITM y replay, confirmación bilateral y vinculación criptográfica de las identidades con la información de conectividad; advierte de que un QR por sí solo no demuestra proximidad física.

## B. Proceso de fondo: formas de ejecución (PROPUESTA, para arquitectura)

El PRD fija la capacidad (DECISIÓN APROBADA por Salvador el 2026-11-13: existe un proceso de fondo propio de la aplicación que el usuario arranca y para sin root). El cómo queda para arquitectura.

- Idea de Salvador: en Linux podría ser un servicio siempre activo que se pueda arrancar y parar.
- HECHO: un servicio del sistema exige root (es lo que hace Enola, §14) y choca con la decisión «sin root» (§14–15). Los servicios de usuario de systemd (`systemctl --user`) se instalan en la carpeta del usuario y se gestionan sin root (`systemd.unit(5)`, `systemctl(1)`).
- HIPÓTESIS sin verificar: mantener el servicio vivo sin sesión abierta requiere «linger» (`loginctl enable-linger`), que según la distribución puede pedir credenciales de administrador.
- HECHO: no todas las distribuciones Linux usan systemd; la ejecución en primer plano en una terminal debe bastar por sí sola.
- Consecuencia de producto aceptada por Salvador: con arranque automático, «en línea» significa «equipo encendido con sesión iniciada», no «usando la aplicación». El emisor no distingue cómo se ejecuta la aplicación del destinatario; solo si es alcanzable.

## C. Notas para arquitectura surgidas en el PRD (no son decisiones)

- **Órdenes de borrado y de eliminación.** DECISIÓN APROBADA: siguen «el mismo procedimiento que los mensajes» (cola en el equipo que las origina, envío al aparecer el contacto, confirmación). Queda para arquitectura si comparten canal, cola y confirmación con los mensajes, y cómo se ordenan respecto a ellos (un mensaje anterior a la orden de borrado no debe sobrevivirla; uno posterior no debe verse afectado, FR-28).
- **Contacto eliminado con la cascada incompleta.** DECISIÓN APROBADA: el equipo propio lo bloquea y conserva «solo lo justo» para entregarle las órdenes (FR-32). Qué es exactamente «lo justo» (material de identidad, datos de descubrimiento) es cuestión de arquitectura. HECHO: mientras lo conserve, el equipo propio sigue participando en el descubrimiento de ese equipo, y por tanto sigue siendo localizable por él.
- **Límite de conversación en los dos equipos.** DECISIÓN APROBADA: la eliminación por límite se aplica en ambos equipos (FR-29) y los pendientes cuentan (FR-30). HECHO: mientras haya pendientes, los dos equipos no guardan el mismo conjunto de mensajes; arquitectura debe definir cómo converge cada equipo a «los últimos 100» sin que uno elimine lo que el otro todavía no ha recibido.
- **Orden y reloj.** El PRD exige el orden por emisor (FR-15) y muestra la hora de redacción (FR-16). El orden entre los dos usuarios y la confianza en el reloj del emisor están abiertos (sección 11.2 del PRD, cuestión 13).
- **Espera del descubrimiento.** HECHO: segundos en régimen normal, reintentos de minutos con un servicio recién publicado, y un límite de 32 circuitos en construcción simultáneos por proceso Tor (`research/technical-descubrimiento-p2p-2026-11-13/research.md`, Dimensión 1, fuentes [34]–[37]). El PRD solo exige que la espera sea visible (NFR-8).
- **Protección en disco.** Con las conversaciones conservadas, el robo del equipo expone el historial (limitación declarada). Si se cifra en reposo y con qué secreto, dado que no hay cuenta ni recuperación (§5.1), queda para arquitectura.
- **Presencia más amplia con arranque automático.** Si el proceso de fondo arranca al iniciar sesión, los contactos observan el patrón de uso del equipo, no el de la aplicación (limitación declarada, aceptada por Salvador).
