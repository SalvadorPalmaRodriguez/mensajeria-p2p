---
title: "PRD: mensajeria-p2p"
status: draft
created: 2026-11-13
updated: 2026-11-13
---

# PRD: mensajeria-p2p

> BORRADOR EN CURSO (Discovery, coaching por recorridos). Manda `docs/continuidad-proyecto.md`; las referencias «§» apuntan a él. Este archivo recoge solo lo ya decidido por Salvador; el registro completo de la sesión está en `.memlog.md` y la profundidad para arquitectura en `addendum.md`. Todavía no es el PRD: faltan visión, glosario, requisitos numerados, no-objetivos y métricas.

## Punto de reanudación

1. Precisar con Salvador, uno a uno, los detalles de la conservación de las conversaciones (ver «Cuestiones abiertas de la sesión»).
2. Confirmar las suposiciones 2 y 3 de UJ-2.
3. UJ-3 (escribir a un contacto ausente) y UJ-4 (revocar un contacto).
4. Glosario, requisitos funcionales y transversales, no-objetivos, métricas; después Finalize (auditoría del memlog, reconciliación con las entradas, revisión, pulido).
5. Al cerrar: actualizar §16 del documento rector y presentar el PRD a Salvador para su aprobación.

## Marco de trabajo (DECISIÓN APROBADA, Salvador 2026-11-13)

- Entradas: brief, addendum del brief y documento rector; informes de research comprobados (materia de arquitectura).
- Rigor: nivel lanzamiento.
- Modo: coaching, entrada por recorridos de usuario.

## Recorridos de usuario

### UJ-1. Salvador y Marta se emparejan en persona — DECISIÓN APROBADA

«Marta» es un nombre de relleno. Están sentados juntos, cada uno con su portátil Linux y la aplicación instalada; ninguno tiene al otro como contacto.

1. Cada uno lanza en su consola el comando de emparejar.
2. Los dos equipos se encuentran por el mecanismo que elija arquitectura (DECISIÓN PENDIENTE; candidatos y criterio en `addendum.md` §A). Sea cual sea, el emparejamiento es presencial: las dos personas juntas, cada una con su equipo (§5.2; reafirmado por Salvador el 2026-11-13).
3. Cada pantalla muestra algo que los dos comprueban juntos mirando la pantalla del otro. La comprobación visual es obligatoria sea cual sea el mecanismo.
4. Cada uno confirma en su equipo. Todo o nada: si uno no confirma, o lo mostrado no coincide, no se crea ningún contacto en ninguno de los dos equipos.
5. Cada uno pone al otro el nombre que quiera; ese nombre vive solo en su equipo y el otro no lo ve.
6. La aplicación confirma en ambas consolas que el emparejamiento se ha completado, y cada uno ve al otro al listar contactos.

### UJ-2. Salvador conversa con Marta, que está en línea — PROPUESTA en revisión

Aprobado hasta ahora: los estados del mensaje, el proceso de fondo y la conservación de las conversaciones (abajo). Pendiente: rehacer el recorrido con la conversación conservada y confirmar dos suposiciones:

- Suposición 2 (sin confirmar): «conversación normal» comprobable = varios mensajes alternos en ambos sentidos dentro de una sesión; cada uno llega una sola vez, en orden, y acaba como entregado. Sin número fijo de mensajes.
- Suposición 3 (sin confirmar): si el contacto deja de ser alcanzable a media conversación, la sesión no se cierra ni da error; lo escrito queda pendiente y la consola avisa.

### UJ-3. Escribir a un contacto ausente — por narrar

### UJ-4. Revocar un contacto — por narrar

## Decisiones de producto tomadas en esta sesión (DECISIÓN APROBADA, Salvador 2026-11-13)

### Proceso de fondo

1. La aplicación tiene un proceso de fondo propio. Arrancado: el equipo es alcanzable para sus contactos, recibe mensajes y entrega los pendientes de la cola. Parado: el equipo está ausente para todos. «La aplicación en marcha» significa «el proceso de fondo está arrancado».
2. El usuario lo arranca, lo para y consulta su estado con comandos de la aplicación, sin root.
3. Los comandos de listar contactos, enviar y conversar hablan con ese proceso; no hace falta una consola abierta para recibir.
4. Dos formas de tenerlo en marcha: en primer plano en una terminal (cualquier Linux) o como servicio de usuario opcional que arranca al iniciar sesión. El cómo es PROPUESTA para arquitectura (`addendum.md` §B).
5. Lo recibido sin tener la conversación abierta se guarda en el equipo del destinatario.
6. El emisor no sabe cómo se ejecuta la aplicación del destinatario; solo si el mensaje llega.

### Estados del mensaje

- Dos estados, vistos por el emisor: **pendiente** (está en la cola de su equipo; no ha salido hacia nadie más) y **entregado** (el equipo del destinatario ha confirmado que lo tiene; se muestra la hora de redacción).
- El paso a entregado exige confirmación del equipo del destinatario. Sin confirmación, el mensaje sigue pendiente y se reintenta; el destinatario no lo ve duplicado.
- Entregado no significa leído. No hay estado «leído» en la v1: sería un metadato nuevo que el brief no declara.

### Conservación de las conversaciones

- Las conversaciones permanecen en el equipo; al reabrir una conversación se ve lo anterior.
- Solo se borran manualmente, por alguno de los dos usuarios.
- El borrado lo solicita uno y se aplica en los dos equipos. Funciona como los mensajes: la acción de borrado se envía al otro equipo; si no se puede, queda en reserva en el equipo que la pide y se envía en cuanto el otro tenga la aplicación activa.
- Limitaciones a declarar (HECHO): la orden pendiente vive en el equipo de quien la pide y solo sale si su aplicación está en marcha cuando el otro aparece; el borrado en el equipo del otro lo ejecuta su aplicación y no se puede comprobar ni imponer desde el propio equipo.
- Límite de 100 mensajes por conversación, para liberar espacio: al superarlo se eliminan solo los más antiguos según entran los nuevos, de modo que siempre se conservan los últimos 100. Al abrir el chat se informa de la regla: al entrar el mensaje 101 desaparece el 1, al entrar el 102 desaparece el 2, y así; siempre se ven los últimos 100. Es solo informativo, sin confirmación. (Descartado: un aviso en el mensaje 99.) [ASSUMPTION: la eliminación por límite también se aplica en los dos equipos, como el borrado manual.]
- Descartado en la misma sesión: el borrado al cerrar el chat o al parar la aplicación y el borrado automático a los 10 minutos.

## Cuestiones abiertas de la sesión (no resueltas)

Sobre la conservación de las conversaciones, para Salvador (detalles sin precisar):

- (2) Límite de 100 mensajes por conversación: confirmar que la eliminación por límite se aplica en los dos equipos.
- (3) Si los mensajes pendientes de la cola cuentan para el límite y pueden eliminarse sin haberse entregado.
- (4) HECHO a declarar como limitación: quien se haga con el dispositivo puede leer las conversaciones guardadas; su protección en disco es cuestión de arquitectura.

Remitidas a arquitectura (DECISIÓN PENDIENTE; el PRD no las resuelve):

- Conciliar las pruebas que dependen de la red Tor con los tests deterministas (§16).
- Autorización en la capa de custodia (§8).
- Periodo de rotación de identificadores y número de épocas (§8).
- Formato del registro de endpoint (§16).
- Mecanismo de presencia (§16).
- Mecanismo de emparejamiento presencial (§5.2; `addendum.md` §A).
