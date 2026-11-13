---
title: "PRD: mensajeria-p2p"
status: draft
created: 2026-11-13
updated: 2026-11-13
---

# PRD: mensajeria-p2p

*Nombre provisional. Borrador completo, pendiente de la aprobación de Salvador.*

## 0. Propósito del documento

Este PRD fija qué debe hacer la primera versión (v1) de mensajeria-p2p. Está escrito para Salvador, que lo aprueba, y para las fases siguientes de la planificación: arquitectura, épicas e historias. Parte del product brief cerrado y de su addendum (`_bmad-output/planning-artifacts/briefs/brief-mensajeria-p2p-2026-11-13/`) y no los repite.

Manda `docs/continuidad-proyecto.md` (el documento rector): si este PRD lo contradice, vale el documento rector. Las referencias «§» apuntan siempre a él; las partes de este PRD se citan como «sección N». Las elecciones técnicas ya aprobadas allí (transporte, descubrimiento, Tor, conectividad) no se repiten aquí como requisitos; el PRD describe capacidades y la sección 6 enumera las restricciones que heredan.

Convenciones:

- Cada requisito lleva una línea **Origen** con su etiqueta: DECISIÓN APROBADA y dónde consta (un apartado «§» del documento rector, o «PRD 2026-11-13» para lo decidido por Salvador durante este PRD y registrado en §16).
- Lo que el redactor ha inferido sin confirmación de Salvador va marcado `[ASSUMPTION: …]` y está indexado en la sección 12. No es una decisión.
- Lo que el PRD no resuelve va etiquetado DECISIÓN PENDIENTE y está en la sección 11.
- Los términos del glosario (sección 3) se usan siempre con el mismo significado.
- Los nombres de los comandos son ilustrativos (brief, addendum §B). El PRD describe lo que hace cada comando, no cómo se llama.

## 1. Visión

Hoy, para hablar en privado con alguien hay que confiar en un tercero: una empresa, sus servidores, su registro y su política de datos (DECISIÓN APROBADA, §16). mensajeria-p2p es una aplicación de mensajería para Linux en la que ese tercero no existe. Dos personas se instalan la aplicación, se emparejan en persona y a partir de ahí se escriben directamente de un equipo al otro. No hay cuenta, registro, teléfono ni cuota, y ningún mensaje pasa por una máquina ajena ni se guarda en ella (§4, §5.5).

La v1 es deliberadamente pequeña: una aplicación de consola por comandos, texto entre dos personas, para usuarios de Linux que ya se ven en persona. Su objetivo es demostrar que dos equipos se emparejan y conversan, probado con tres equipos y todas sus parejas (§16).

La apuesta del producto es que, para ese público, vale más no depender de nadie que tener la entrega garantizada. Por eso el producto declara sus límites en lugar de ocultarlos (sección 7): tus contactos conocen tu IP y saben cuándo estás en línea, no hay entrega garantizada y algunos contactos no serán alcanzables.

## 2. Usuario objetivo

### 2.1 Trabajos por hacer

- Hablar por escrito con alguien que conozco en persona sin que ninguna empresa ni servidor vea, transporte o guarde la conversación.
- Saber que quien me escribe es un equipo con el que me emparejé teniéndolo delante.
- No recibir nada de desconocidos: sin directorio, sin búsqueda y sin contactos indirectos (§5.3, §6).
- Dejar un mensaje a alguien que ahora no está y saber si le ha llegado.
- Poder borrar una conversación, o romper con un contacto, sin dejar restos.
- No pagar, no registrarme y no necesitar permisos de administrador (§4, §14).

### 2.2 Para quién no es la v1

Quien necesite contactar con desconocidos, entrega garantizada, uso desde el móvil, grupos o envío de archivos (brief, «Para quién»).

El público de la v1 es Salvador y las personas con las que acuerde usarla: usuarios de Linux, cómodos en una terminal, que se conocen en persona (DECISIÓN APROBADA, §16).

### 2.3 Recorridos de usuario

Los cuatro recorridos están aprobados por Salvador (PRD 2026-11-13). «Marta» es un nombre de relleno para el segundo usuario.

**UJ-1. Salvador y Marta se emparejan en persona.**
Están sentados juntos, cada uno con su equipo y la aplicación instalada; ninguno tiene al otro como contacto.

1. Cada uno lanza en su consola el comando de emparejar.
2. Los dos equipos se encuentran por el mecanismo que elija arquitectura (DECISIÓN PENDIENTE, sección 11.1).
3. Cada pantalla muestra algo que los dos comprueban juntos mirando la pantalla del otro.
4. Cada uno confirma en su equipo. Si uno no confirma, o lo mostrado no coincide, no se crea ningún contacto en ninguno de los dos equipos.
5. Cada uno pone al otro el nombre local que quiera.
6. La aplicación confirma en ambas consolas que el emparejamiento se ha completado, y cada uno ve al otro al listar contactos.

**UJ-2. Salvador conversa con Marta, que está en línea.**
Días después, cada uno en su casa, los dos con el proceso de fondo arrancado.

1. Salvador teclea el comando de conversar con Marta.
2. La consola informa de la regla de los últimos 100 mensajes, muestra lo anterior de la conversación y dice si Marta es alcanzable ahora.
3. Salvador escribe una línea. Aparece con su hora de redacción y como pendiente; cuando el equipo de Marta confirma, pasa a entregado.
4. Marta, que tiene abierto el chat con Salvador, ve la línea con el nombre local que ella le puso y la hora de redacción, y contesta. A Salvador le aparece la respuesta sin teclear nada.
5. Siguen de ida y vuelta.
6. Salvador sale del chat. El proceso de fondo sigue arrancado y la conversación queda guardada.

Caso límite: si Marta deja de ser alcanzable a media conversación, el chat no se cierra ni da error. La consola avisa, y lo que Salvador escriba queda pendiente hasta que ella vuelva.

**UJ-3. Salvador escribe a Marta, que está ausente.**

1. Marta tiene el equipo apagado. Salvador le envía un mensaje, con el comando de enviar o desde el chat.
2. La consola le dice que Marta no es alcanzable y que el mensaje queda pendiente en su equipo.
3. Salvador cierra la consola. Su proceso de fondo sigue arrancado.
4. Horas después Marta enciende su equipo. El equipo de Salvador la detecta y le entrega el mensaje sin que él haga nada.
5. Cuando Marta abre el chat, ve el mensaje con la hora en que Salvador lo escribió, no la hora en que llegó.
6. Cuando Salvador vuelve a mirar, el mensaje aparece como entregado.

Caso límite: si Salvador intenta enviar con su propio proceso de fondo parado, la aplicación no acepta el mensaje y le dice que tiene que arrancarlo primero.

**UJ-4. Salvador elimina a Marta de sus contactos.**
A Marta le han robado el equipo y se lo dice a Salvador en persona.

1. Salvador teclea el comando de eliminar a Marta de sus contactos.
2. La aplicación le pide confirmación, porque no se puede deshacer.
3. Al confirmar, el equipo de Salvador deja de aceptar y de mostrar nada del equipo de Marta, aunque ese equipo esté ausente.
4. La eliminación sigue una cascada: primero se borran los mensajes en los dos equipos y después se elimina el contacto en los dos equipos. Lo que no pueda enviarse al equipo de Marta queda pendiente y sale cuando ese equipo aparezca.
5. Salvador puede consultar en qué punto está la eliminación: pendiente de que el otro equipo aparezca, o completada.
6. Para volver a hablar con Marta hay que emparejarse otra vez en persona, con su equipo nuevo.

## 3. Glosario

- **Equipo** — Un ordenador Linux con una instalación de la aplicación. Es la unidad de identidad: la aplicación identifica equipos, no personas (§4).
- **Usuario** — La persona que maneja un equipo.
- **Identidad** — El material criptográfico propio de una instalación. Una por equipo; no se recupera ni se migra (§5.1).
- **Emparejamiento** — El acto presencial en el que dos usuarios, juntos y cada uno con su equipo, establecen la confianza mutua entre sus dos equipos (§5.2).
- **Contacto** — Un equipo con el que el propio equipo está emparejado. La relación es bilateral y no transitiva (§5.3).
- **Nombre local** — El nombre que un usuario pone a un contacto. Vive solo en su equipo; el contacto no lo ve.
- **Proceso de fondo** — El proceso propio de la aplicación que mantiene al equipo en la red. «La aplicación en marcha» significa que el proceso de fondo está arrancado.
- **Alcanzable** — Estado de un contacto cuyo equipo puede recibir ahora mismo desde el propio equipo.
- **Aparecer** — Pasar de ausente a alcanzable.
- **Ausente** — Estado de un contacto que no es alcanzable, sea cual sea el motivo: equipo apagado, proceso de fondo parado o red que impide la conexión directa.
- **Mensaje** — Un texto que un usuario envía a un contacto.
- **Conversación** — El conjunto de mensajes intercambiados con un contacto, guardado en cada uno de los dos equipos. Una por contacto.
- **Chat** — La sesión interactiva por líneas en la que un usuario ve una conversación y escribe en ella.
- **Hora de redacción** — El momento en que el emisor escribió el mensaje.
- **Pendiente** — Estado de un mensaje, o de una orden, que está en la cola del equipo que lo originó y todavía no ha sido confirmado por el equipo del contacto.
- **Entregado** — Estado de un mensaje cuya confirmación de entrega ha llegado al emisor. No significa leído.
- **Confirmación de entrega** — El aviso con el que el equipo del destinatario comunica al del emisor que tiene guardado un mensaje o que ha aplicado una orden.
- **Cola** — El lugar, dentro del equipo que los originó, donde esperan los mensajes y las órdenes pendientes. Nunca está fuera de ese equipo (§5.5).
- **Orden de borrado** — La petición que un equipo envía al del contacto para que borre la conversación entre ambos.
- **Orden de eliminación** — La petición que un equipo envía al del contacto para que deje de tenerlo como contacto.
- **Límite de conversación** — El máximo de 100 mensajes que se conservan por conversación.
- **Tercero** — Cualquier máquina que no sea uno de los dos equipos de una conversación.

## 4. Funcionalidades

### 4.1 Proceso de fondo

**Descripción.** La aplicación tiene un proceso de fondo propio. Mientras está arrancado, el equipo es alcanzable para sus contactos, recibe mensajes y entrega lo que tenga pendiente en la cola. Parado, el equipo está ausente para todos sus contactos. El usuario decide cuándo arrancarlo y pararlo. Un contacto no puede saber de qué forma se ejecuta la aplicación en otro equipo; solo si ese equipo es alcanzable. Sostiene UJ-2, UJ-3 y UJ-4.

#### FR-1: Arrancar el proceso de fondo

El usuario puede arrancar el proceso de fondo con un comando de la aplicación, sin root.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13; §14–15).

**Consecuencias comprobables:**
- Un usuario sin privilegios arranca el proceso de fondo y la aplicación lo confirma.
- Con el proceso de fondo arrancado en dos equipos emparejados, cada uno llega a ver al otro como alcanzable.
- Arrancarlo cuando ya está arrancado no crea un segundo proceso; la aplicación informa de que ya lo está.

#### FR-2: Parar el proceso de fondo

El usuario puede parar el proceso de fondo con un comando de la aplicación, sin root.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- Tras pararlo, los contactos del equipo pasan a verlo como ausente.
- Los mensajes y las órdenes pendientes siguen en la cola después de parar y volver a arrancar (ver FR-25).

#### FR-3: Consultar si el proceso de fondo está arrancado

El usuario puede consultar con un comando si el proceso de fondo está arrancado o parado.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- El comando devuelve «arrancado» o «parado» de forma inequívoca, también cuando lo ejecuta un script.

#### FR-4: Dos formas de tener el proceso de fondo en marcha

El usuario puede tener el proceso de fondo en primer plano en una terminal, o como servicio de usuario que arranca al iniciar sesión. La segunda forma es opcional y solo existe donde el sistema la permita.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13). El cómo es PROPUESTA para arquitectura (`addendum.md` §B).

**Consecuencias comprobables:**
- En primer plano, la aplicación funciona completa en un Linux sin gestor de servicios de usuario.
- Ninguna de las dos formas exige root ni instala nada fuera de la carpeta del usuario.
- La forma de ejecución no cambia lo que los contactos pueden observar del equipo.

#### FR-5: Los comandos dependen del proceso de fondo

Los comandos de listar contactos, enviar, conversar, borrar y eliminar hablan con el proceso de fondo. No hace falta tener una consola abierta para recibir.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- Con el proceso de fondo arrancado y ninguna consola abierta, un mensaje entrante se recibe y se guarda (ver FR-17).
- Con el proceso de fondo parado, el comando de enviar no acepta el mensaje y dice que hay que arrancarlo primero (ver FR-24).
- `[ASSUMPTION: con el proceso de fondo parado, los demás comandos tampoco operan y dan el mismo aviso.]`

### 4.2 Emparejamiento presencial

**Descripción.** Dos usuarios que están juntos, cada uno con su equipo, emparejan esos dos equipos. El producto promete «verificación presencial del emparejamiento»: sabes con qué equipo te emparejas porque lo tienes delante. No promete nada sobre quién maneja ese equipo después (DECISIÓN APROBADA, §16). El mecanismo concreto es DECISIÓN PENDIENTE (sección 11.1); los requisitos siguientes valen para cualquiera que se elija. Realiza UJ-1.

#### FR-6: Identidad propia de cada instalación

Cada instalación tiene su propia identidad, creada en el equipo. No se recupera, no se migra y no se comparte entre equipos.

**Origen:** DECISIÓN APROBADA (§5.1).

**Consecuencias comprobables:**
- Dos instalaciones distintas tienen identidades distintas.
- La aplicación no ofrece ningún comando para exportar, importar ni recuperar una identidad.
- Una instalación nueva en el mismo equipo no hereda los contactos de la anterior.

#### FR-7: Emparejar dos equipos presentes

Dos usuarios pueden emparejar sus equipos lanzando cada uno el comando de emparejar. El emparejamiento solo puede completarse con los dos usuarios juntos, cada uno con su equipo.

**Origen:** DECISIÓN APROBADA (§5.2; reafirmado en PRD 2026-11-13).

**Consecuencias comprobables:**
- El emparejamiento exige una acción en cada uno de los dos equipos; no puede completarse desde uno solo.
- La aplicación no ofrece ninguna forma de añadir un contacto a distancia: ni enlace, ni invitación, ni introducción por otro contacto (§5.3).

**Fuera de alcance:** elegir el mecanismo (código, QR, proximidad u otro).

#### FR-8: Comprobación visual y confirmación de los dos

Durante el emparejamiento, cada equipo muestra algo que los dos usuarios comprueban juntos mirando la pantalla del otro. Cada usuario confirma después en su equipo.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13; §5.2 exige confirmación bilateral y protección frente a MITM y replay).

**Consecuencias comprobables:**
- El emparejamiento no avanza hasta que los dos usuarios han confirmado.
- Si un tercero se interpone entre los dos equipos durante el emparejamiento, lo que muestran las dos pantallas no coincide.
- Repetir más tarde lo intercambiado en un emparejamiento anterior no produce un emparejamiento.

#### FR-9: Todo o nada

Si uno de los dos usuarios no confirma, o lo mostrado no coincide, no se crea ningún contacto en ninguno de los dos equipos.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- Tras un emparejamiento cancelado, rechazado o interrumpido, ninguno de los dos equipos lista al otro como contacto.
- Tras un emparejamiento completado, los dos equipos se listan mutuamente como contacto, y la aplicación lo confirma en ambas consolas.

#### FR-10: Nombre local

Cada usuario pone al contacto recién emparejado el nombre local que quiera.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- El nombre local que Salvador pone a Marta no llega al equipo de Marta.
- Los mensajes de un contacto se muestran con su nombre local.
- `[ASSUMPTION: dentro de un equipo, dos contactos no pueden tener el mismo nombre local.]`

### 4.3 Contactos

**Descripción.** El usuario ve sus contactos y nada más: no hay directorio, búsqueda ni contactos indirectos. Solo un contacto puede comunicarse con el equipo. Sostiene UJ-1 a UJ-4.

#### FR-11: Listar contactos

El usuario puede listar sus contactos con un comando.

**Origen:** DECISIÓN APROBADA (§16, alcance de la v1).

**Consecuencias comprobables:**
- La lista contiene exactamente los contactos emparejados y no eliminados, cada uno con su nombre local.
- Un contacto en proceso de eliminación no aparece como contacto (ver FR-32).
- `[ASSUMPTION: la lista indica, para cada contacto, si es alcanzable ahora.]`
- `[ASSUMPTION: la lista indica qué contactos tienen mensajes recibidos que el usuario todavía no ha visto. Es un indicador local; no se comunica al contacto.]`

#### FR-12: Solo los contactos pueden comunicarse con el equipo

El equipo solo acepta mensajes y órdenes de sus contactos.

**Origen:** DECISIÓN APROBADA (§5.3, §6; brief, «Qué lo hace diferente»).

**Consecuencias comprobables:**
- Un equipo que no es contacto no consigue que el equipo acepte ni muestre nada suyo.
- Que A sea contacto de B y B de C no da a A ninguna capacidad sobre C.
- La aplicación no ofrece búsqueda de usuarios ni listado de equipos que no sean contactos.

### 4.4 Mensajes y estados

**Descripción.** El usuario envía texto a un contacto y sabe si ha llegado. Hay dos estados, vistos por el emisor: pendiente y entregado. No hay estado «leído» en la v1, porque sería un dato nuevo que los contactos aprenderían del usuario (DECISIÓN APROBADA, PRD 2026-11-13). Realiza UJ-2 y UJ-3.

#### FR-13: Enviar un mensaje suelto

El usuario puede enviar un mensaje de texto a un contacto con un comando, sin abrir un chat.

**Origen:** DECISIÓN APROBADA (§16, alcance de la v1).

**Consecuencias comprobables:**
- El comando termina informando del estado del mensaje: entregado, o pendiente porque el contacto está ausente.
- El mensaje aparece en la conversación con ese contacto.

**Fuera de alcance:** imágenes, archivos y cualquier contenido que no sea texto; mensajes a más de un contacto.

#### FR-14: Estados pendiente y entregado

Todo mensaje enviado está en uno de dos estados: pendiente o entregado. Pasa a entregado solo cuando llega la confirmación de entrega del equipo del destinatario.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- Un mensaje cuyo envío no dio error pero cuya confirmación de entrega no ha llegado sigue pendiente.
- Un mensaje pendiente se reintenta hasta que llega su confirmación de entrega, se borra o lo elimina el límite de conversación.
- El emisor puede ver el estado de cada mensaje suyo en la conversación.
- La aplicación no muestra ni transmite ningún indicador de lectura.

#### FR-15: Sin pérdidas, sin repetidos y en orden

Entre dos equipos que son alcanzables entre sí, cada mensaje llega una sola vez, y los mensajes de un mismo emisor se muestran en el orden en que los escribió.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13, definición de «conversación normal»).

**Consecuencias comprobables:**
- Un mensaje reintentado tras perderse su confirmación de entrega aparece una sola vez en el equipo del destinatario.
- Una serie de mensajes alternos en ambos sentidos acaba con todos entregados, ninguno repetido y el orden de cada emisor respetado, en los dos equipos.

#### FR-16: Hora de redacción

Cada mensaje se muestra con su hora de redacción, tanto al emisor como al destinatario.

**Origen:** DECISIÓN APROBADA (§16; PRD 2026-11-13).

**Consecuencias comprobables:**
- Un mensaje entregado horas después de escribirse se muestra al destinatario con la hora en que se escribió, no con la de llegada.
- `[ASSUMPTION: la hora de redacción es la del reloj del equipo emisor.]`

#### FR-17: Recibir sin tener el chat abierto

El equipo recibe y guarda los mensajes de sus contactos mientras el proceso de fondo está arrancado, tenga o no el usuario un chat abierto.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- Un mensaje recibido sin chat abierto está en la conversación cuando el usuario la abre.
- El emisor lo ve entregado desde que el equipo del destinatario lo guarda, aunque el destinatario no lo haya visto.

### 4.5 Chat

**Descripción.** El chat es una sesión interactiva por líneas con un contacto, sin pantalla completa (DECISIÓN APROBADA, §16). El usuario ve la conversación guardada, escribe líneas y ve llegar las del contacto. Realiza UJ-2.

#### FR-18: Abrir un chat

El usuario puede abrir un chat con un contacto con un comando.

**Origen:** DECISIÓN APROBADA (§16; PRD 2026-11-13).

**Consecuencias comprobables:**
- Al abrir, la consola informa de la regla del límite de conversación: se conservan los últimos 100 mensajes; al entrar el 101 desaparece el 1, y así sucesivamente.
- Al abrir, la consola muestra los mensajes guardados de la conversación, cada uno con su hora de redacción y, los propios, con su estado.
- Al abrir, la consola dice si el contacto es alcanzable ahora.
- La aplicación no pide confirmación de ese aviso.

#### FR-19: Conversar en directo

Con el chat abierto, cada línea que el usuario escribe se envía como un mensaje, y los mensajes del contacto aparecen sin que el usuario teclee nada.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- Una línea escrita aparece como pendiente y pasa a entregado al llegar la confirmación de entrega, sin que el usuario haga nada.
- Un mensaje del contacto aparece en el chat abierto con el nombre local del contacto y su hora de redacción.
- Si el descubrimiento del contacto todavía está en curso, la consola lo indica en lugar de quedarse sin respuesta (ver NFR-8).

#### FR-20: El chat sobrevive a la desconexión

Si el contacto deja de ser alcanzable con el chat abierto, el chat sigue abierto.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- La consola avisa de que el contacto ha dejado de ser alcanzable; el chat no se cierra ni termina con error.
- Las líneas escritas después quedan pendientes y se entregan cuando el contacto vuelve a ser alcanzable.
- La consola avisa cuando el contacto vuelve a ser alcanzable.

#### FR-21: Salir del chat

El usuario puede salir del chat con una orden de salida.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- Salir no para el proceso de fondo ni borra la conversación.
- Los mensajes que lleguen después se reciben según FR-17.

### 4.6 Cola y entrega al aparecer el contacto

**Descripción.** Si el contacto está ausente, el mensaje queda pendiente en la cola del equipo del emisor y se entrega cuando el contacto vuelve a ser alcanzable. La cola nunca sale del equipo del emisor: no es almacenamiento en un tercero (DECISIÓN APROBADA, §5.5). No hay garantía de entrega (§5.4). Realiza UJ-3.

#### FR-22: Dejar pendiente un mensaje para un contacto ausente

Cuando el usuario envía un mensaje a un contacto ausente, el mensaje queda pendiente en la cola de su equipo.

**Origen:** DECISIÓN APROBADA (§5.5, §16).

**Consecuencias comprobables:**
- La consola dice que el contacto no es alcanzable y que el mensaje queda pendiente en este equipo.
- Mientras está pendiente, el contenido del mensaje no sale del equipo del emisor hacia ninguna otra máquina.

#### FR-23: Entrega automática cuando el contacto aparece

Cuando un contacto con mensajes pendientes vuelve a ser alcanzable, el equipo del emisor se los entrega sin intervención del usuario.

**Origen:** DECISIÓN APROBADA (§16, criterio de éxito 3).

**Consecuencias comprobables:**
- Con el proceso de fondo del emisor arrancado y ninguna consola abierta, los mensajes pendientes pasan a entregado después de que el contacto aparezca.
- Los mensajes pendientes de un mismo emisor se entregan en el orden en que se escribieron.
- Si el proceso de fondo del emisor está parado cuando el contacto aparece, no se entrega nada hasta que vuelva a arrancarse y los dos coincidan.

**Fuera de alcance:** cómo detecta el equipo que un contacto ha aparecido. El mecanismo de presencia es DECISIÓN PENDIENTE (sección 11.1).

#### FR-24: No aceptar envíos con el proceso de fondo parado

Si el proceso de fondo del propio equipo está parado, la aplicación no acepta mensajes nuevos.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- El comando de enviar termina con un aviso que dice que hay que arrancar el proceso de fondo, y el mensaje no se guarda como pendiente.

#### FR-25: La cola persiste

`[ASSUMPTION: los mensajes y las órdenes pendientes, y las conversaciones, se conservan al parar y volver a arrancar el proceso de fondo y al reiniciar el equipo.]`

**Origen:** inferido de la DECISIÓN APROBADA «las conversaciones permanecen en el equipo» (PRD 2026-11-13); la persistencia de la cola entre reinicios no se ha confirmado expresamente.

**Consecuencias comprobables:**
- Un mensaje pendiente antes de parar el proceso de fondo sigue pendiente al volver a arrancarlo y se entrega según FR-23.

### 4.7 Conservación y borrado de conversaciones

**Descripción.** Las conversaciones permanecen en los dos equipos. Solo desaparecen de dos maneras: porque uno de los dos usuarios las borra, o porque el límite de conversación va eliminando los mensajes más antiguos. Las dos se aplican en los dos equipos. Salvador descartó durante el PRD el borrado al cerrar el chat y el borrado automático por tiempo. Sostiene UJ-2 y UJ-4.

#### FR-26: Las conversaciones se conservan

La conversación con cada contacto se guarda en el equipo y sigue ahí al cerrar el chat y al parar el proceso de fondo.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- Al reabrir un chat, el usuario ve los mensajes anteriores de esa conversación, dentro del límite de conversación.
- Ningún mensaje se elimina por el paso del tiempo.

#### FR-27: Borrar una conversación en los dos equipos

Cualquiera de los dos usuarios puede borrar la conversación. El borrado lo pide uno y se aplica en los dos equipos.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- Tras el borrado, la conversación no aparece en el equipo de quien lo pidió.
- Si el contacto es alcanzable, la conversación deja de aparecer también en su equipo, y el equipo que lo pidió recibe la confirmación.
- El contacto sigue siendo contacto: borrar una conversación no lo elimina.
- `[ASSUMPTION: el borrado manual actúa sobre la conversación entera, incluidos los mensajes pendientes; no hay borrado de mensajes sueltos.]`

#### FR-28: La orden de borrado espera en la cola

Si el contacto está ausente, la orden de borrado queda pendiente en la cola del equipo que la pidió y se envía cuando el contacto vuelve a ser alcanzable, igual que un mensaje.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- Con el contacto ausente, la conversación se borra de inmediato en el equipo que lo pidió y la orden de borrado queda pendiente.
- Cuando el contacto aparece, con el proceso de fondo de quien lo pidió arrancado, la conversación se borra en el equipo del contacto.
- Los mensajes escritos después de pedir el borrado no se ven afectados por esa orden.

#### FR-29: Límite de conversación

Cada conversación conserva como máximo 100 mensajes. Al entrar uno nuevo por encima del límite, se elimina el más antiguo. La eliminación por límite se aplica en los dos equipos, igual que el borrado manual.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- Una conversación nunca muestra más de 100 mensajes en ninguno de los dos equipos.
- Al entrar el mensaje 101 deja de mostrarse el 1; al entrar el 102, el 2.
- El límite es por conversación: llegar a 100 con un contacto no afecta a las conversaciones con otros.
- La aplicación no muestra ningún aviso al llegar al límite, salvo el caso de FR-30; la regla se explica al abrir el chat (FR-18).
- `[ASSUMPTION: cuentan para el límite los mensajes de los dos usuarios, enviados y recibidos.]`

#### FR-30: Los mensajes pendientes cuentan para el límite

Los mensajes pendientes cuentan para el límite de conversación. Si la conversación llega al límite con mensajes pendientes, al entrar uno nuevo se elimina el más antiguo aunque sea un mensaje pendiente.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13). Salvador eligió esta opción frente a la de impedir escribir más.

**Consecuencias comprobables:**
- Cuando hay mensajes pendientes y la conversación llega al mensaje 99, la aplicación recuerda al usuario que está a un mensaje del límite.
- Sin mensajes pendientes, ese recordatorio no aparece.
- Un mensaje pendiente eliminado por el límite no se entrega nunca al destinatario.

### 4.8 Eliminación de un contacto

**Descripción.** El usuario puede dejar de confiar en un contacto, por ejemplo tras el robo de su equipo. La eliminación protege desde el primer momento y después limpia los dos equipos en cascada, para no dejar restos: primero los mensajes, después el contacto. Es una limpieza cooperativa entre los dos equipos; no es una revocación remota garantizada (§5.1; sección 7). Realiza UJ-4.

#### FR-31: Eliminar un contacto, con confirmación

El usuario puede eliminar un contacto con un comando. La aplicación pide confirmación antes de empezar, porque no se puede deshacer.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13; §5.7).

**Consecuencias comprobables:**
- Sin la confirmación del usuario, no cambia nada.
- La eliminación puede pedirse con el contacto ausente.

#### FR-32: Bloqueo inmediato

Desde que el usuario confirma la eliminación, su equipo deja de aceptar y de mostrar nada del equipo eliminado, y deja de mostrarlo como contacto. Solo conserva lo justo para entregarle las órdenes pendientes.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- Un mensaje enviado por el equipo eliminado después de la confirmación no se guarda ni se muestra.
- El usuario no puede enviar mensajes al equipo eliminado.
- El bloqueo vale aunque el equipo eliminado esté ausente y la cascada no haya empezado en él.

#### FR-33: Cascada en los dos equipos

La eliminación sigue este orden: primero se borran los mensajes en los dos equipos; después se elimina el contacto en los dos equipos. Cada paso se envía al otro equipo con el mismo procedimiento que los mensajes: si está ausente, la orden queda pendiente en la cola y sale cuando aparezca.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- En el equipo que pide la eliminación, la conversación se borra al confirmar, y lo último que se descarta es lo que ese equipo conserva del contacto para entregarle las órdenes (FR-32).
- En el otro equipo, la orden de borrado se aplica antes que la orden de eliminación.
- Completada la cascada, ninguno de los dos equipos conserva la conversación ni lista al otro como contacto, y el equipo que la pidió ya no conserva nada del eliminado.

#### FR-34: Saber en qué punto está la eliminación

El usuario puede consultar en qué punto está una eliminación que ha pedido.

**Origen:** DECISIÓN APROBADA (PRD 2026-11-13).

**Consecuencias comprobables:**
- La consulta distingue al menos dos situaciones: órdenes pendientes de que el otro equipo aparezca, y cascada completada en el otro equipo.
- Una eliminación completada deja de aparecer como eliminación en curso.

#### FR-35: Volver a hablar exige emparejarse de nuevo

Tras una eliminación, los dos equipos solo pueden volver a ser contactos mediante un nuevo emparejamiento presencial.

**Origen:** DECISIÓN APROBADA (§5.1, §5.3).

**Consecuencias comprobables:**
- La aplicación no ofrece ningún comando para deshacer una eliminación.

## 5. Requisitos transversales

- **NFR-1. Sin root.** Toda la funcionalidad de la v1 funciona ejecutada por un usuario sin privilegios de administrador y sin capacidades adicionales. *Comprobable:* la batería completa de pruebas pasa con un usuario sin privilegios. **Origen:** DECISIÓN APROBADA (§14–15).
- **NFR-2. El contenido solo toca los dos equipos.** Ningún tercero transporta ni almacena el contenido de un mensaje, cifrado o no. *Comprobable:* en la topología de pruebas, el tráfico que lleva mensajes tiene como origen y destino únicamente los dos equipos de la conversación. El método de comprobación lo define arquitectura. **Origen:** DECISIÓN APROBADA (§5.5; §16, criterio de éxito 4).
- **NFR-3. Sin conectividad directa no hay comunicación.** Si dos equipos no pueden conectarse directamente, los mensajes quedan pendientes; la aplicación no recurre a ningún camino alternativo. *Comprobable:* con la conexión directa bloqueada entre dos equipos, ningún mensaje se entrega y todos quedan pendientes. **Origen:** DECISIÓN APROBADA (§5.5, §9).
- **NFR-4. La red Tor solo sirve para encontrarse.** Se usa para el descubrimiento y la señalización, nunca para transportar ni almacenar mensajes. **Origen:** DECISIÓN APROBADA (§4, §5.9).
- **NFR-5. Sin servicios de terceros ni infraestructura central.** La aplicación no usa cuentas, registro, servidores propios, APNs, FCM ni servidores STUN externos. **Origen:** DECISIÓN APROBADA (§4).
- **NFR-6. Solo se acepta lo que viene de un contacto autenticado.** Todo mensaje y toda orden se aceptan solo si proceden de un equipo que se autentica con la identidad fijada en el emparejamiento. *Comprobable:* un equipo que presenta una identidad distinta de la emparejada no consigue entregar nada. **Origen:** DECISIÓN APROBADA (§5.2, §5.8).
- **NFR-7. Participación limitada a los contactos directos.** La v1 opera solo en Modalidad A: un equipo solo colabora en operaciones auxiliares relacionadas con sus propios contactos. **Origen:** DECISIÓN APROBADA (§6, §16).
- **NFR-8. La espera se ve.** Cuando una operación depende de encontrar al contacto, la consola indica que la búsqueda está en curso. HECHO: el descubrimiento por la red Tor tarda segundos en régimen normal y puede exigir reintentos de minutos con un servicio recién publicado (informe de descubrimiento, fuentes [34]–[36]). El PRD no fija tiempos máximos: no hay base para ellos (regla 3, §3).
- **NFR-9. Los avisos dicen qué pasa y qué hacer.** Todo aviso de la aplicación nombra la situación con los términos del glosario (alcanzable, ausente, pendiente, entregado) y, si el usuario puede hacer algo, dice qué.
- **NFR-10. Código visible y dependencias abiertas.** El código de la aplicación es visible bajo la licencia de §14, y todo el software de terceros incorporado es open source. **Origen:** DECISIÓN APROBADA (§14).

## 6. Restricciones heredadas del documento rector

Son DECISIONES APROBADAS que este PRD respeta y no reabre. Arquitectura las toma del documento rector, no de aquí.

- Linux primero; Windows, macOS y móviles después (§12).
- Rust y arquitectura hexagonal: dominio, puertos y adaptadores (§11).
- Transporte directo con WebRTC mediante `str0m`; sin TURN ni STUN externo (§5.8).
- IPv6 primero; IPv4 más adelante, con un adaptador separado (§5.6).
- Descubrimiento por rendezvous derivado por pareja, con custodia en contactos mutuos (D-16.6, §8).
- Tor clásico 0.4.9 o posterior como proceso de usuario (§13).
- Conectividad en cascada autodetectada (§9).
- Reutilización de Enola por la «vía C» (§14).
- Pruebas solo con máquinas virtuales y network namespaces; las pruebas del descubrimiento usan la red Tor (regla 8, §3; §16).
- TDD y tests deterministas (`AGENTS.md`).

## 7. Limitaciones declaradas

El producto las declara al usuario; no son defectos por corregir en la v1. Las cuatro primeras vienen del brief; las demás nacen de decisiones tomadas en este PRD (HECHO en todos los casos).

- **Tus contactos conocen tu IP y saben cuándo estás en línea.** Solo ellos. Si el proceso de fondo arranca al iniciar sesión, «en línea» significa «equipo encendido con la sesión iniciada», no «usando la aplicación».
- **No hay entrega garantizada.** Un mensaje pendiente solo sale si el proceso de fondo del emisor está arrancado cuando el contacto aparece (§5.4). Algunos contactos serán inalcanzables, en especial detrás del filtrado de los operadores móviles (§9).
- **Si pierdes el equipo, pierdes su identidad y sus contactos** (§5.1).
- **Un contacto que custodia tu registro de conectividad puede inferir con quién te relacionas** (§7.5).
- **Quien se haga con tu equipo puede leer las conversaciones guardadas** y hacerse pasar por ti hasta que cada contacto lo elimine. La protección de lo guardado en disco es cuestión de arquitectura (sección 11.1).
- **El borrado en el equipo del otro no se puede garantizar.** Lo ejecuta su aplicación: tu equipo no puede imponerlo, ni impedir que el otro haya copiado la conversación antes.
- **Una orden de borrado o de eliminación puede no completarse nunca.** Solo sale si tu proceso de fondo está arrancado cuando el otro equipo aparece, y el otro equipo puede no volver a aparecer.
- **Un mensaje pendiente puede perderse por el límite de conversación.** Si escribes más de 100 mensajes a un contacto ausente, los más antiguos se eliminan sin haberse entregado (FR-30).
- **Encontrar a un contacto puede tardar** de segundos a algunos minutos (NFR-8).
- **Auditable no es auditado.** Ningún componente tiene hoy una auditoría de seguridad pública (§5.8).

## 8. Lo que el producto no es

- No es un servicio: no hay empresa, servidor, cuenta ni cuota detrás (§4).
- No es una red para conocer gente: no hay directorio, búsqueda, invitaciones a distancia ni introducciones (§5.3).
- No es mensajería con entrega garantizada ni con almacenamiento en la nube (§5.4, §5.5).
- No es una herramienta de anonimato: los mensajes no viajan por Tor y los contactos ven la IP del usuario (§5.9).
- No recupera identidades ni transfiere la confianza de un equipo a otro; es así por diseño, no solo en la v1 (§5.1).
- No recurrirá a relays, TURN ni intermediarios para alcanzar a contactos que no son alcanzables directamente (§5.5, §9).

## 9. Alcance de la v1

### 9.1 Dentro

- Linux, sin root, por consola de comandos.
- Proceso de fondo que el usuario arranca, para y consulta (sección 4.1).
- Emparejamiento presencial (sección 4.2) y lista de contactos (sección 4.3).
- Mensajes de texto entre dos equipos, con estados pendiente y entregado (sección 4.4).
- Chat interactivo por líneas (sección 4.5).
- Cola en el equipo del emisor y entrega al aparecer el contacto (sección 4.6).
- Conversaciones conservadas, con borrado en los dos equipos y límite de 100 mensajes (sección 4.7).
- Eliminación de un contacto en cascada (sección 4.8).
- Modalidad A.

### 9.2 Fuera

- Grupos y salas de chat. Son una posibilidad futura sin forma definida; su compatibilidad con §5.3 y §5.5 no está analizada (§16).
- Imágenes, archivos y cualquier contenido que no sea texto.
- Estado «leído» (descartado para la v1, PRD 2026-11-13).
- Interfaz gráfica e interfaz de texto a pantalla completa.
- Modalidad B (§6).
- Windows, macOS y móviles (§12).
- IPv4 (§5.6).
- Canal de distribución y nombre definitivo del producto (§4).
- Pruebas con dispositivos físicos y en redes de operadores reales (regla 8, §3).

## 10. Criterios de éxito

La v1 ha salido bien si se cumplen los cuatro (DECISIÓN APROBADA, §16).

- **SM-1.** Dos equipos se emparejan y mantienen una conversación normal: varios mensajes alternos en ambos sentidos dentro de un chat, sin pérdidas ni repetidos y en orden. Valida FR-7 a FR-10 y FR-13 a FR-21.
- **SM-2.** Lo anterior funciona con un mínimo de tres equipos y todas sus parejas (A–B, A–C, B–C). Valida además FR-11 y FR-12. Tres es el mínimo que ejercita la custodia en contactos mutuos (brief, addendum §C).
- **SM-3.** Un mensaje enviado con el contacto ausente queda pendiente en el equipo del emisor y se entrega cuando el contacto aparece. Valida FR-22 y FR-23.
- **SM-4.** Se comprueba que ningún tercero transporta ni almacena el contenido y que nada exige root. Valida NFR-1 y NFR-2.

Los equipos son máquinas virtuales y network namespaces, con la red Tor para el descubrimiento. El éxito queda demostrado en simulación, no en redes reales (§16).

`[NOTE FOR PM]` Los cuatro criterios vienen del brief y no cubren lo decidido después en este PRD: el borrado de conversaciones (sección 4.7) y la eliminación de un contacto (sección 4.8) quedan fuera de la definición de éxito. Esos requisitos siguen siendo comprobables por sus consecuencias, pero Salvador debe decidir si añade un quinto criterio (cuestión 17 de la sección 11.2).

**Lo que no se debe optimizar:**

- **SM-C1. Proporción de contactos alcanzables.** No se mejora añadiendo intermediarios: un contacto inalcanzable de forma directa sigue siendo inalcanzable (§5.5, §9). Contrapesa SM-1 y SM-2.
- **SM-C2. Rapidez o seguridad de la entrega.** No se mejora guardando mensajes fuera del equipo del emisor ni manteniendo al usuario en línea sin que lo decida (§5.4). Contrapesa SM-3.
- **SM-C3. Adopción.** No hay objetivos de adopción ni de negocio (brief, «Criterios de éxito»).

## 11. Cuestiones abiertas

### 11.1 Remitidas a arquitectura

Son DECISIONES PENDIENTES. Este PRD fija los requisitos que deben cumplir y no las resuelve.

1. **Tests deterministas frente a la red Tor.** Cómo se concilian las pruebas del descubrimiento, que usan la red Tor, con la convención de tests deterministas. HIPÓTESIS sin verificar: una red Tor privada local para los tests automáticos (§16).
2. **Autorización en la capa de custodia.** Restricted discovery de Tor frente a cifrado por destinatario (§8).
3. **Rotación de identificadores.** Periodo de rotación y número de épocas concurrentes (§8).
4. **Formato del registro de endpoint** (§16).
5. **Mecanismo de presencia.** Cómo sabe un equipo que un contacto es alcanzable (§16). Lo necesitan FR-18, FR-20 y FR-23.
6. **Mecanismo de emparejamiento.** Debe cumplir FR-7 a FR-9 y §5.2. Candidatos y criterio de Salvador en `addendum.md` §A.
7. **Forma de ejecución del proceso de fondo** (`addendum.md` §B).
8. **Protección en disco** de la identidad, las conversaciones y la cola.
9. **Órdenes de borrado y de eliminación.** Cómo se envían, se confirman y se ordenan respecto a los mensajes; qué conserva exactamente un equipo sobre un contacto eliminado mientras la cascada no se completa (`addendum.md` §C).

### 11.2 Para Salvador

Ninguna impide pasar a arquitectura; conviene cerrarlas antes de las historias.

10. Qué ve el usuario cuando su contacto borra la conversación o lo elimina: si la aplicación se lo dice, o si la conversación y el contacto simplemente desaparecen.
11. Si una orden pendiente hacia un equipo que no vuelve a aparecer caduca alguna vez, o se conserva indefinidamente.
12. Si se puede emparejar de nuevo con un equipo mientras una eliminación anterior sigue sin completarse en él.
13. Cómo se ordenan en pantalla los mensajes de los dos usuarios cuando uno llega mucho después de escribirse: por hora de redacción o por orden de llegada.
14. Si el éxito de la v1 equivale a «completamente funcional» a efectos de §12 (empezar Windows y macOS). Viene del brief.
15. Si la fase de UX (`bmad-ux`) se omite o se aligera, dado que la v1 es una consola por comandos (PROPUESTA del addendum del brief, §D).
16. Las suposiciones de la sección 12.
17. Si se añade un criterio de éxito que cubra el borrado de conversaciones y la eliminación de un contacto (sección 10).

## 12. Índice de suposiciones

Todo lo marcado `[ASSUMPTION]` en el documento, para que Salvador lo confirme o lo corrija:

- FR-5 — Con el proceso de fondo parado, los demás comandos tampoco operan y dan el mismo aviso que el de enviar.
- FR-10 — Dentro de un equipo, dos contactos no pueden tener el mismo nombre local.
- FR-11 — La lista de contactos indica si cada contacto es alcanzable ahora.
- FR-11 — La lista de contactos indica qué contactos tienen mensajes recibidos sin ver; el indicador es local.
- FR-16 — La hora de redacción es la del reloj del equipo emisor.
- FR-25 — La cola y las conversaciones se conservan al parar y volver a arrancar el proceso de fondo y al reiniciar el equipo.
- FR-27 — El borrado manual actúa sobre la conversación entera, incluidos los mensajes pendientes; no hay borrado de mensajes sueltos.
- FR-29 — Cuentan para el límite los mensajes de los dos usuarios, enviados y recibidos.
