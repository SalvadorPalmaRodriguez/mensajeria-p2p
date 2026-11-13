---
documento: continuidad-proyecto
estado: vigente
creado: 2026-11-12
ultima_actualizacion: 2026-11-13
proposito: Registro autoritativo de decisiones aprobadas, pendientes, hipótesis y estado del análisis del proyecto de mensajería P2P
autoridad: documento rector del proyecto; en caso de contradicción con cualquier otro artefacto, manda este
sustituye: versión anterior del documento de continuidad
---

# CONTINUIDAD DEL PROYECTO: APLICACIÓN DE MENSAJERÍA P2P DISTRIBUIDA

Versión del documento: 13 de noviembre de 2026 (sustituye a la versión anterior).

## 1. Tu papel

Actúa como arquitecto de software sénior especializado en:

- Redes P2P y sistemas distribuidos.
- Linux, Windows y macOS (y, en el futuro, Android e iOS).
- Protocolos IPv4, IPv6, UDP y TCP.
- WebRTC, ICE, STUN y WireGuard.
- Tor, Onion Services y Arti.
- Criptografía aplicada e identidad digital.
- Privacidad, anonimato y protección de metadatos.
- Arquitecturas hexagonales y diseño modular (Rust).

Tu misión es ayudarme a determinar la viabilidad técnica y arquitectónica de una aplicación de mensajería distribuida P2P.

No quiero confirmaciones por complacencia. Necesito análisis crítico, documentación técnica y conclusiones fundamentadas.

No debemos empezar a programar ni cerrar tecnologías por intuición.

## 2. Documento de referencia

Este documento es la referencia principal y es autosuficiente. Contiene todas las decisiones aprobadas hasta la fecha.

No vuelvas a investigar desde cero las cuestiones ya estudiadas. Continúa desde el estado actual del proyecto (apartado 16).

## 3. Reglas de trabajo

Distingue siempre entre:

- **HECHO:** respaldado por documentación técnica o comportamiento demostrado.
- **HIPÓTESIS:** posibilidad todavía no demostrada.
- **PROPUESTA:** alternativa que recomiendas estudiar.
- **DECISIÓN APROBADA:** autorizada expresamente por mí.
- **DECISIÓN PENDIENTE:** todavía no resuelta.

Reglas obligatorias:

1. No conviertas propuestas en decisiones aprobadas.
2. No modifiques decisiones aprobadas sin consultarme.
3. No inventes porcentajes de éxito, capacidades ni garantías.
4. No confundas viabilidad teórica con capacidades reales de cada sistema operativo.
5. Investiga documentación oficial, RFC, especificaciones, proyectos y código fuente.
6. Cita las fuentes que respalden las afirmaciones importantes. Distingue fuentes oficiales de no oficiales; si se contradicen, indícalo y da prioridad a la oficial.
7. Si la documentación no permite obtener una conclusión, indícalo.
8. No propongas pruebas con dispositivos físicos: actualmente no dispongo de ellos. **SÍ se permiten pruebas en Linux con máquinas virtuales y network namespaces** (ejecutadas en una VM Linux local, sin hardware adicional) — aprobado 2026-11-12. Limitación: solo simulan topologías; no verifican comportamiento de CPE/carriers reales. **DECISIÓN APROBADA (2026-11-13):** se retira de esta regla la condición «sin tráfico externo». Era una descripción del entorno de pruebas, no una decisión con motivo registrado, y es incompatible con el diseño: el descubrimiento aprobado (D-16.6) necesita siempre la red Tor, también en las pruebas.
9. Avanza progresivamente, una cuestión cada vez, y pregúntame paso a paso.
10. Antes de abordar una decisión arquitectónica nueva, explica sus implicaciones y espera mi autorización.
11. No repitas innecesariamente los análisis anteriores.
12. No supongas que una tecnología es necesaria simplemente porque otros proyectos P2P la utilizan.

## 4. Objetivo del producto

Construir una aplicación de mensajería distribuida P2P con una experiencia comparable a una aplicación convencional.

Principios fundamentales:

- La unidad de identidad es el dispositivo, no la persona.
- Cada instalación tiene su propia identidad criptográfica.
- El emparejamiento requiere presencia física.
- Cada dispositivo conoce exclusivamente sus contactos directos.
- La confianza es bilateral y no transitiva.
- No existe un directorio global de usuarios o contactos.
- No existe un grafo global de relaciones.
- Los mensajes viajan exclusivamente entre emisor y destinatario.
- No se utilizan servidores ni dispositivos intermediarios para transportar mensajes.
- No se almacenan mensajes en servidores ni dispositivos intermediarios.
- No se garantiza la entrega cuando el destinatario está desconectado o no es alcanzable.
- No queremos infraestructura central propia que gestione la red.
- La infraestructura auxiliar son los dispositivos que tengan instalada la aplicación.

**DECISIÓN APROBADA (aclaración del principio rector):** el requisito estricto de «sin servidores de terceros» se aplica al transporte de mensajes, que debe ser exclusivamente P2P. Tor sí puede utilizarse para infraestructura auxiliar (descubrimiento y señalización), nunca para transportar mensajes.

**DECISIÓN APROBADA (servicios de terceros excluidos):** quedan excluidos APNs, FCM y servidores STUN externos. Motivo del usuario: no quiere dependencias ni costes encubiertos.

**DECISIÓN APROBADA (sin cuotas):** no se pagará ninguna cuota. La forma de distribución de la aplicación queda fuera del alcance actual.

## 5. Decisiones arquitectónicas aprobadas anteriormente

### 5.1. Identidad

- Identidad criptográfica independiente por instalación.
- No existe identidad personal compartida entre dispositivos.
- Si se pierde un dispositivo, se pierde su identidad.
- Un dispositivo nuevo genera una identidad nueva.
- No hay recuperación remota.
- No hay migración automática de identidad.
- No hay transferencia automática de confianza.
- Para recuperar relaciones de confianza hay que volver a emparejar presencialmente cada contacto.

### 5.2. Emparejamiento

Debe requerir presencia física.

Se pueden estudiar QR, NFC, comparación de códigos, autenticación mutua e intercambio criptográfico seguro.

Hay que proteger frente a MITM y replay, exigir confirmación bilateral y vincular criptográficamente las identidades con la información de conectividad.

Un QR por sí solo no demuestra proximidad física.

### 5.3. Confianza

- Confianza bilateral.
- No se hereda.
- No existen contactos indirectos.
- No hay introducciones de contactos.
- No hay delegación automática.
- No existe un grafo global.

### 5.4. Disponibilidad

- Confianza y disponibilidad son independientes.
- Un contacto desconectado no pierde su confianza.
- La indisponibilidad no equivale a revocación.
- No se garantiza entrega offline.
- No es obligatorio mantener disponibilidad permanente en segundo plano.

### 5.5. Transporte de mensajes

**Los mensajes deben circular exclusivamente entre los dos dispositivos participantes.**

Quedan descartados: TURN, relays de contenido, store-and-forward, otros dispositivos como intermediarios, almacenamiento remoto de mensajes y servidores que reciban y entreguen mensajes en nombre del emisor.

Si no hay conectividad directa, la comunicación no está disponible.

**DECISIÓN APROBADA (2026-11-13, aclaración de alcance):** la prohibición de store-and-forward se refiere a intermediarios. Una cola de mensajes pendientes en el propio equipo del emisor está permitida: el mensaje no sale de ese equipo hasta que se entrega directamente al destinatario. No altera 5.4: sigue sin garantizarse la entrega.

### 5.6. IPv6

Primera implementación exclusivamente IPv6. IPv4 se estudiará posteriormente mediante un adaptador separado.

Hay que considerar firewalls, políticas de operadores, direcciones temporales, cambios de dirección, UDP entrante y restricciones de red. Tener IPv6 global no garantiza conectividad entrante.

### 5.7. WireGuard

Organización aprobada:

- Una única interfaz WireGuard por dispositivo.
- Un par de claves propio de la interfaz.
- Múltiples peers, uno por contacto.
- Direccionamiento interno individual.
- Endpoints IPv6 independientes.
- Gestión individual de altas, bajas y revocaciones.
- Eliminar un peer al revocar un contacto.
- La rotación de la clave privada de interfaz puede exigir actualizar todos los contactos.

La identidad de la aplicación debe permanecer separada de las claves WireGuard.

**DECISIÓN APROBADA (2026-11-12, reinterpretación de «interfaz»):** «una única interfaz WireGuard» se interpreta en sentido **funcional/lógico**: el modelo organizativo (un par de claves, un peer por contacto, direccionamiento interno como identificadores lógicos, gestión individual de altas/bajas/revocaciones) puede implementarse en-proceso sobre un socket UDP, sin interfaz de red del SO. Precedente de producción: Tailscale (tsnet) y NetBird ejecutan el protocolo WireGuard en espacio de usuario sin TUN. Esta reinterpretación es consecuencia necesaria de la decisión «sin root» (§14), no una expansión del modelo.

No está aprobado WireGuard como transporte definitivo. No dispone de ICE ni NAT traversal general. No utilizar servidores VPN ni relays para solucionar la conectividad.

### 5.8. WebRTC

**DECISIÓN APROBADA (2026-11-12):** el transporte de mensajería directo será **WebRTC mediante el crate `str0m`** (MIT/Apache-2.0, Sans-IO puro Rust, mantenimiento muy activo — 0.25.0 del 3-nov-2026). Razones: ICE integrado materializa la cascada de conectividad aprobada (§9); SCTP datachannels proporcionan la capa de mensajería fiable; BoringTun quedó descartado por tener 3 issues de seguridad abiertos sin parchear (#502 validación de claves efímeras, #496 redirección de endpoint por cookie forjada, #495 sesiones sordas) y mantenimiento dudoso (#407, reestructuración anunciada). Evaluadas también webrtc.rs (viable, más pesado) y el híbrido WG+ICE de Firezone (heredaba los bugs de BoringTun).

Consecuencias aprobadas:

- TURN sigue prohibido y STUN externo excluido (apartado 4): ICE operará con candidatos host + peer-reflexive sobre IPv6 (verificado viable por el informe PCP, §9).
- El modelo de 5.7 se traslada conceptualmente: un peer por contacto se implementa como certificado DTLS con fingerprint fijado al emparejamiento; la identidad de la app permanece separada del transporte.
- Riesgo aceptado y declarado: el camino P2P de str0m está menos probado que el de SFU y su ICE se mide más lento que libwebrtc; se verificará con VM/network namespaces (regla 8) antes de implementar.
- Ningún stack evaluado tiene auditoría de seguridad pública — limitación registrada.

### 5.9. Tor

Puede utilizarse para infraestructura auxiliar, descubrimiento y señalización, nunca para transportar mensajes.

Se han considerado Onion Services v3 por dispositivo y vinculación criptográfica de las direcciones Onion con la identidad de la aplicación. No está aprobada ninguna modalidad de persistencia Onion.

Decisiones nuevas sobre Tor: ver apartado 13.

### 5.10. Enola

Repositorio: https://github.com/SalvadorPalmaRodriguez/enola-cli-lite

Decisiones y hechos actualizados: ver apartado 14.

## 6. Decisión aprobada: participación voluntaria A/B

### Modalidad A: participación limitada a relaciones directas

Un dispositivo solamente participa en operaciones auxiliares relacionadas con sus propios contactos directos. No colabora con dispositivos desconocidos ni amplía su ámbito de confianza a través de otros contactos.

### Modalidad B: participación voluntaria en la infraestructura distribuida

El usuario permite que su dispositivo ayude a la red en operaciones auxiliares de otros participantes, aunque no mantenga relaciones de confianza directa con ellos (descubrimiento, localización de servicios conocidos, intercambio de información de conectividad, señalización, mantenimiento de información auxiliar distribuida).

**DECISIÓN APROBADA (2026-11-12):** la **Modalidad A es la modalidad por defecto**. Todo dispositivo/instalación comienza colaborando solo dentro de sus relaciones directas; la Modalidad B es opt-in explícito del usuario. Consecuencia: el mecanismo de descubrimiento debe funcionar con población mixta y no puede asumir que ningún dispositivo colabora más allá de sus contactos directos salvo que haya activado B.

No está aprobado DHT, Gossip, Rendezvous ni ningún otro protocolo concreto.

> **HECHO (nota de vigencia, 2026-11-13):** texto anterior a D-16.6. El protocolo de descubrimiento ya está aprobado: rendezvous derivado por pareja (familia BRP) con custodia en contactos mutuos; las DHT quedaron eliminadas (ver apartado 8).

### Restricciones comunes

- No hay servidores centrales propios ni servidores externos de señalización.
- No hay transporte ni almacenamiento de mensajes por colaboradores.
- No se establecen relaciones de confianza por colaborar.
- No se concede acceso a agendas privadas.
- No se crea un directorio global de contactos.
- No se debe poder reconstruir innecesariamente el grafo de confianza.
- La participación es voluntaria y configurable.

No está aprobado que B sea la modalidad por defecto.

## 7. Decisiones aprobadas sobre la Modalidad A

### 7.1. Autorización por solicitante

La autorización para utilizar los servicios auxiliares de un dispositivo se determina por la identidad del solicitante. Esta condición no es suficiente por sí sola (ver 7.2).

### 7.2. Limitación por relaciones directas

Escenario: A confía en B, B confía en D, A no conoce a D, B pide a A ayuda para localizar a D. **DECISIÓN APROBADA: denegar.**

Un colaborador solo puede atender solicitudes relacionadas con dispositivos que sean contactos directos suyos:

Autorizado(A,B,D) = Contacto(A,B) AND Contacto(A,D)

(Política lógica, no implementación aprobada.)

### 7.3. Consecuencias aprobadas

- No existe propagación transitiva de confianza.
- La relación B-D no concede a B permisos adicionales sobre A.
- A no puede colaborar con la localización de D si D no es contacto directo de A.
- Cada dispositivo mantiene su propio ámbito de colaboración.
- La disponibilidad de descubrimiento en A depende de sus contactos directos.
- La colaboración no implica que el dispositivo objetivo autorice automáticamente a un tercero a conocer su información de conectividad.

### 7.4. Cuestiones no resueltas

No están aprobados: el mecanismo contra la enumeración de contactos, la forma de consultar la agenda privada sin exponerla, la caducidad de los registros (valor concreto), la retirada de registros replicados, los límites de consumo de recursos, el mecanismo de descubrimiento ni los protocolos.

### 7.5. Autorización del propietario de la información de conectividad (NUEVO)

**DECISIÓN APROBADA (D-12.1).** Un colaborador solo puede revelar la información de conectividad de un dispositivo D a un solicitante si D lo ha autorizado expresamente. Que el colaborador tenga una relación directa con el solicitante y con D no basta. Esta autorización es independiente de la modalidad A/B.

Revelable(A,B,D) = Autorizado(A,B,D) AND AutorizaD(D,B)

**DECISIÓN APROBADA (D-12.2).** Solo pueden recibir autorización los contactos directos de D, o un subconjunto que D elija. Condicionada a la hipótesis H1: fuera de una relación directa B-D no existe ningún uso legítimo de la información de conectividad de D. Si H1 se refutara, D-12.2 debe revisarse.

**DECISIÓN APROBADA (D-12.3).** Se estudiará preferentemente que la autorización la imponga la criptografía (el colaborador no puede leer la información que custodia), en lugar de una política aplicada por el colaborador. No se ha elegido ningún mecanismo. Referencias de viabilidad del patrón: HPKE (RFC 9180), descubrimiento restringido de Tor (antes «client authorization»), credenciales con restricciones (macaroons).

Consecuencias aprobadas:

- Si se cumple la Modalidad A pero D no ha autorizado, la operación se deniega.
- La revocación de una autorización solo tiene efecto hacia el futuro. La caducidad de los registros es un requisito necesario para acotar la exposición (su valor sigue pendiente).
- Cada cambio de endpoint de D obliga a actualizar la información autorizada para cada destinatario.

**DECISIÓN APROBADA (D-12.4, 2026-11-12).** La comprobación Contacto(A,D) se realiza **al depositar**: A solo acepta y custodia registros de sus contactos directos; las consultas se resuelven con identificadores opacos. Concretado con las protecciones aprobadas:

- Los registros se almacenan **cifrados por destinatario autorizado** (materializa D-12.3: A no puede leer el contenido).
- Los identificadores opacos se derivarán **con clave (blinded) y rotarán** — un hash sin clave de la identidad es enumerable en dominios pequeños y no aporta opacidad real. Referencia de diseño: blinded keys de Onion Services v3 (Tor rend-spec-v3).
- Las consultas se plantean a través de **onion services** del colaborador (uso auxiliar coherente con apartado 13; el canal exacto queda ligado a la cuestión 6 de descubrimiento, aún pendiente).

**Reinterpretación aprobada de 7.2:** la «atención» de solicitudes se verifica en el depósito en lugar de cada consulta; 7.2 queda satisfecha porque A solo custodia registros de contactos directos suyos.

**Cuestión abierta asociada — resuelta 2026-11-12 (limitación declarada):** el colaborador aprende que D usa el servicio (no es información nueva: D ya es contacto de A) y puede *inferir* relaciones B–D por correlación temporal depósito→consulta y repetición de consultas al mismo registro. Esa filtración residual **no es eliminable** con el modelo actual; se acepta y documenta, no se promete anonimato frente al colaborador. **Limitación asociada:** si D deja de ser contacto de A tras depositar, el registro debe purgarse; la caducidad obligatoria ya aprobada acota pero no resuelve el caso por sí sola.

## 8. Análisis anterior sobre mecanismos de descubrimiento

Familias identificadas: Rendezvous (la implementación de libp2p tiene rasgos federados), DHT (Sybil, bootstrap, metadatos), Gossip (difusión y exposición de metadatos), Tor Onion Services (usa relays y directorios de Tor), descubrimiento oportunista.

**DECISIÓN APROBADA (D-16.6, 2026-11-13):** el protocolo de descubrimiento será un **rendezvous derivado por pareja de la familia BRP** (Bramble Rendezvous Protocol v0, Briar), con estas especificaciones:

- Cada pareja de contactos deriva offline sus endpoints (servicios onion v3) a partir del secreto DH compartido en el emparejamiento; para el canal primario no se deposita información de conectividad en terceros.
- Extensión sobre BRP v0: rotación de identificadores por época y rendezvous continuo (BRP v0 solo cubre el rendezvous inicial con ventana de 48 h y no rota identificadores). El periodo de rotación y el número de épocas concurrentes quedan como cuestión de diseño pendiente (suelo práctico documentado: horas–días; ~minutos es el límite inferior por propagación de descriptores y epoch HSDir).
- Capa complementaria de **custodia en contactos mutuos** (materializa D-12.4): registros de endpoint cifrados por destinatario custodiados en los onion services de contactos mutuos, como mecanismo de robustez. Es complementaria, no sustitutiva, del rendezvous bilateral.
- Reimplementación desde la especificación pública: Briar es GPLv3/AGPLv3, no se importa código copyleft. El crate `bramble` de crates.io queda descartado (etiquetas KDF distintas del spec → wire-incompatible, sin transporte Tor).
- Alojamiento de los onion services con C tor 0.4.9+ como proceso de usuario. La investigación confirma §13: el aviso «testing only» del doc oficial de Arti sigue sin rescindirse, así que la condición para adoptar Arti aún no se cumple.

**Eliminadas:** todas las DHT (Tox, OpenDHT/Jami, libp2p-kad, HyperDHT, Veilid) — requieren custodia o enrutado por dispositivos desconocidos y ninguna funciona sin primer contacto conocido; además, con población mayoritaria en modalidad A no se forma overlay. libp2p rendezvous — registros en claro (multiaddrs), sin autenticación ni payload opaco en el spec. Mailbox de onion estático (patrón Cwtch/Ricochet) — identificador sondeable y no rotatorio.

**Cuestiones de diseño pendientes (no aprobadas):** el periodo de rotación y las épocas concurrentes; la forma de autorización en la capa de custodia (restricted discovery de Tor — con las limitaciones declaradas de cap ~160 clientes y revocación diferida — frente a autorización pura por cifrado por destinatario, línea de D-12.3).

Informe de la investigación: `_bmad-output/planning-artifacts/research/technical-descubrimiento-p2p-2026-11-13/research.md` (45 fuentes; 21 afirmaciones verificadas, 3 sin verificar).

## 9. Investigación de conectividad ya realizada

- IPv6 global no garantiza conexión entrante. RFC 6092 describe filtrado stateful en firewalls IPv6 residenciales.
- WireGuard utiliza UDP y no incorpora ICE. PersistentKeepalive no garantiza conectividad entrante.
- ICE permite comprobar rutas directas, pero no puede obligar a un firewall a permitirlas. Sin TURN, algunas conexiones WebRTC pueden fallar.
- Sin NAT (IPv6), un equipo normalmente conoce su dirección global por su propia interfaz. Los propios contactos pueden informarse mutuamente de la dirección observada (candidatos peer-reflexive de ICE, RFC 8445).
- **PROPUESTA a estudiar:** PCP (Port Control Protocol, RFC 6887) permite a un equipo IPv6 o IPv4 pedir al router o firewall que acepte tráfico entrante. HIPÓTESIS: no se ha verificado cuántos routers domésticos lo soportan.

La conclusión provisional es que el principal obstáculo es la conectividad directa real entre dispositivos, no la posibilidad conceptual de construir una aplicación P2P. No asumir que la participación auxiliar resuelva por sí sola los problemas de conectividad.

**DECISIÓN APROBADA (2026-11-12):** el subsistema de conectividad seguirá una cascada autodetectada en runtime, según el informe `_bmad-output/planning-artifacts/research/technical-pcp-conectividad-entrante-2026-11-12/research.md` (74 fuentes, citas verificadas):

1. Escucha directa si la red no filtra el entrante (~20-25% de IPv6 residencial medido).
2. **Punching coordinado** (envío mutuo de UDP tras intercambio de endpoints) como mecanismo base — cubre la clase dominante de firewalls stateful IPv6.
3. **Pinhole por protocolo como mejora oportunista**: cliente PCP MAP-sobre-IPv6 primero, `WANIPv6FirewallControl` (UPnP IGDv2) como segunda vía; nunca como requisito — la cobertura real es minoritaria e incuantificada.
4. **Clasificador de red en runtime**: el dispositivo sondea qué clase de red tiene (abierto / stateful / pinhole-capable / bloqueado) y degrada con gracia.
5. **Limitación declarada:** los contactos tras filtrado entrante de carrier móvil son inalcanzables con este modelo — coherente con 5.4 (sin entrega garantizada) y 5.5 (sin relays); no se diseñarán parches que violen esas decisiones.
6. Keepalives por debajo de 2 minutos sobre canales establecidos (RFC 6092 REC-14).

Dependencia irreductible declarada: el punching exige conocer el endpoint del otro → el subsistema de descubrimiento debe proporcionar el canal de intercambio de endpoints (rendezvous); eso queda en la cuestión 6.

## 10. Proyectos de referencia

- Briar: identidad, confianza, Tor y comunicación directa. Su Bramble Rendezvous Protocol deriva datos de contacto de un secreto compartido por pareja. No copiar Briar Mailbox (almacena mensajes intermedios).
- Jami: ICE, conectividad P2P y descubrimiento distribuido.
- Tox: bootstrap, DHT y conexiones directas.
- Session: su transporte y almacenamiento de mensajes no cumplen nuestros requisitos.

No copiar arquitecturas completas. Extraer únicamente los mecanismos compatibles.

## 11. Arquitectura conceptual

Mantener separados estos subsistemas:

1. Identidad y confianza.
2. Descubrimiento y señalización.
3. Conectividad.
4. Mensajería.
5. Infraestructura Tor.
6. Aplicación (escritorio ahora; móvil en el futuro).

La arquitectura debe permitir cambiar los mecanismos de descubrimiento, conectividad, Tor y plataforma sin contaminar el dominio de mensajería (arquitectura hexagonal: dominio, puertos/contratos, adaptadores).

## 12. Plataformas (NUEVO)

**DECISIÓN APROBADA:**

1. La primera implementación será para **Linux**.
2. **Windows y macOS** vendrán después.
3. Los **móviles (Android e iOS)** quedan para el futuro.
4. Esas migraciones empezarán cuando la versión de Linux sea completamente funcional.

**Análisis de móviles archivado** (para el desarrollo futuro, no repetir salvo que se reabra):

- iOS sin APNs: según la nota técnica TN2277 de Apple y las respuestas de sus ingenieros en los foros oficiales, una app suspendida no atiende conexiones entrantes, el sistema puede liberar los sockets en escucha y no se entregan paquetes UDP al proceso suspendido. No hay forma general de impedir la suspensión. BGAppRefreshTask lo decide el sistema (unos 30 s, sin hora garantizada). Local Push Connectivity solo funciona en redes Wi-Fi configuradas y requiere un permiso especial de Apple. Wi-Fi Aware (iOS 26) es P2P local, solo funciona mientras la app se ejecuta y requiere iPhone 12 o posterior.
- Modelo viable documentado para iOS: alcanzable solo con la app activa e iniciando las conexiones al abrirse. No se tomó decisión.
- Android: servicios en primer plano posibles, con restricciones y coste de batería.
- Distribución en iOS: no hay vía documentada sin cuota para particulares. La distribución está fuera del alcance actual.

**Ventajas de Linux (HECHOS e hipótesis ya analizados):**

- Un proceso puede quedarse escuchando indefinidamente; no hay suspensión.
- WireGuard nativo en el kernel (pero ver el apartado 15 sobre root).
- Tor sin limitaciones de batería ni de ciclo de vida.
- Se mantiene el problema del firewall del router (RFC 6092; PCP como propuesta).

**Estimación orientativa, sin base documental** (para una persona con experiencia):

- Prototipo mínimo en Linux (emparejamiento manual, WireGuard sobre IPv6 directo, mensajes por línea de comandos, sin descubrimiento ni Tor): del orden de 4 a 8 semanas.
- Versión completa (descubrimiento, modalidades A/B, Tor, interfaz gráfica): del orden de varios meses.
- No es fiable hasta decidir WireGuard o WebRTC, el mecanismo de descubrimiento y la interfaz.

> **HECHO (nota de vigencia, 2026-11-13):** estimación basada en supuestos superados. El transporte aprobado es WebRTC con `str0m` (5.8), no WireGuard; el descubrimiento es D-16.6 (apartado 8); y la v1 definida en el brief incluye descubrimiento y Tor, con interfaz de consola (apartado 16). No se ha hecho una estimación nueva.

## 13. Tor (NUEVO)

**DECISIÓN APROBADA:** habrá un contrato (puerto) Tor común en la arquitectura.

**DECISIÓN APROBADA:** la primera implementación usará el **Tor clásico (C tor) como proceso de usuario, sin root**. Arti queda para cuando su documentación oficial lo considere apto para alojar Onion Services.

HECHOS:

- El Tor clásico funciona como usuario normal con su propio `torrc`, su directorio de datos en la carpeta del usuario y los puertos de control y SOCKS en 127.0.0.1 por encima de 1024. Los Onion Services se pueden crear con un directorio propiedad del usuario o mediante `ADD_ONION` por el puerto de control (el servicio vive hasta `DEL_ONION`, el cierre de Tor o el cierre de la conexión de control, salvo con la opción Detach).
- El Tor Project deja de dar soporte a Tor 0.4.8 y anteriores el 1 de septiembre de 2026: exigir Tor 0.4.9 o posterior.
- Arti: el cliente y la biblioteca `arti-client` son estables desde la 1.0.0. Las notas oficiales de la 1.7.0 declaran estable el descubrimiento restringido. Pero la guía oficial de Onion Services de Arti (doc/OnionService.md) sigue diciendo que el alojamiento es solo para pruebas, sin vanguards, sin protección DoS, sin prueba de trabajo ni padding. Las fuentes oficiales se contradicen en parte; un registro de cambios no oficial que afirmaba lo contrario no es fiable. Conclusión: no puede afirmarse que el alojamiento con Arti tenga hoy la seguridad de C tor.

## 14. Enola (NUEVO)

HECHOS:

- **El usuario es el autor y titular de Enola.**
- **DECISIÓN APROBADA:** Enola no se modifica por ahora.
- **Autorización del titular:** se puede reutilizar en la aplicación de mensajería lo que haga falta del repositorio (diseño, contratos y código).

HECHOS según el código (último commit revisado: 53044d3, 4 de noviembre de 2026; versión 0.12.0-alpha):

- Rust, arquitectura hexagonal (`domain`, `ports`, `adapters`, `application`, `cli`, `infrastructure`). Expone una biblioteca `enola_core` (sección `[lib]` de `Cargo.toml`). `publish = false`.
- Licencia propietaria «source-visible»: uso personal y no comercial, sin redistribución sin autorización escrita del titular.
- `src/ports/vpn.rs` (`VpnPort`): generación de claves, alta y baja individual de peers, IP interna por peer en una interfaz. Encaja conceptualmente con 5.7.
- `src/application/manage_client_auth.rs`: autorización de clientes x25519 de los Onion Services (patrón de referencia de D-12.3).
- El adaptador de WireGuard usa `wg-quick`, escribe en `/etc/wireguard/` y usa `systemctl enable`: requiere root.
- El adaptador de Tor modifica `/etc/tor/torrc` y controla el Tor del sistema con `systemctl`: requiere root. No usa Arti ni `ADD_ONION`. No se encontró comprobación de versión mínima de Tor.
- Arrastra dependencias ajenas a la mensajería (Docker/bollard, GPU/nvml, IA, CMS) sin opciones de compilación para excluirlas.
- `vpn_bridge` (WireGuard sobre Tor con socat): **sigue excluido** para mensajes (contradice 5.5).

Conclusiones ya dadas:

- El root de Enola se debe a lo que administra (servicios del sistema, Nginx, Docker, UFW, AppArmor, interfaz WireGuard del kernel), no a Tor.
- Enola no está desfasada: el Tor clásico es hoy la opción más madura para alojar Onion Services. No necesita migrar a Arti por ahora. Recomendación para cuando se modifique: exigir Tor 0.4.9 o superior.
- Alternativas para la reutilización: A) Enola entera como dependencia (no recomendada: root, dependencias pesadas, API inestable); B) separar un crate pequeño compartido (recomendada inicialmente, pero implica modificar Enola, que ahora no se hará); C) reutilizar diseño, contratos y código copiando lo necesario a la nueva aplicación, con adaptadores propios sin root.

**DECISIÓN APROBADA (2026-11-12):** vía C confirmada — reutilizar diseño, contratos y código de Enola copiándolo a la nueva aplicación, con adaptadores propios sin root. Enola no se modifica. El titular de Enola (Salvador Palma Rodríguez) autoriza expresamente la copia y adaptación.

**DECISIÓN APROBADA (2026-11-12):** la licencia de la aplicación de mensajería será la misma licencia de Enola (source-visible proprietary, ver `LICENSE` de enola-cli-lite), con Salvador Palma Rodríguez como titular y las mismas restricciones; solo cambian el nombre de la aplicación y ajustes menores.

**DECISIÓN APROBADA (2026-11-12):** la aplicación de mensajería debe funcionar sin privilegios de administrador/root. Consecuencia: quedan descartados la interfaz WireGuard de kernel, TUN, `wg-quick` y cualquier helper privilegiado; el transporte se reduce a implementaciones en proceso (p. ej. BoringTun) u otras stacks de espacio de usuario (p. ej. WebRTC). La interpretación de «interfaz» en 5.7 queda pendiente como cuestión separada.

> **HECHO (nota de vigencia, 2026-11-13):** resuelto después. «Interfaz» se interpreta en sentido lógico (5.7) y el transporte aprobado es WebRTC con `str0m`; BoringTun quedó descartado (5.8).

**DECISIÓN APROBADA (2026-11-12):** todo el software de terceros que la aplicación incorpore (dependencias, bibliotecas, stacks) debe ser open source. La licencia propia de la aplicación es la indicada en el apartado 14 (source-visible, titular Salvador Palma Rodríguez).

## 15. Root en la aplicación (NUEVO)

- Tor: no necesita root (apartado 13).
- Escuchar conexiones IPv6: no necesita root si el puerto es mayor de 1024.
- **WireGuard: única pieza pendiente.** Crear una interfaz WireGuard del kernel exige `CAP_NET_ADMIN` (en la práctica, root o configuración especial).
- **HIPÓTESIS sin verificar:** implementar el protocolo WireGuard dentro de la propia aplicación sobre un socket UDP normal (por ejemplo, con una biblioteca como boringtun), sin crear interfaz de red, evitaría root.

> **HECHO (nota de vigencia, 2026-11-13):** texto anterior a la decisión de transporte. WireGuard ya no es una pieza pendiente: el transporte aprobado es WebRTC con `str0m`, en proceso y sin root, y BoringTun quedó descartado (5.8). La hipótesis anterior ya no es necesaria para la aplicación.

## 16. PUNTO EXACTO DONDE DEBES CONTINUAR

### Primera cuestión: WireGuard sin root en Linux

> **HECHO (nota de vigencia, 2026-11-13):** cuestión resuelta el 2026-11-12: el transporte aprobado es WebRTC con `str0m` (5.8) y «interfaz» se interpreta en sentido lógico (5.7). El punto de continuación vigente es «Siguiente fase» y «Decisiones del product brief», más abajo en este apartado.

Analiza, con el formato del apartado 17:

1. Qué operaciones de WireGuard exigen privilegios en Linux (interfaz del kernel, `CAP_NET_ADMIN`, TUN) y cuáles no.
2. Si es viable usar el protocolo WireGuard dentro del proceso de la aplicación sobre un socket UDP sin crear interfaz de red (boringtun u otras implementaciones en Rust): estado, mantenimiento, licencia, auditorías y limitaciones.
3. Si eso sigue siendo «WireGuard» en el sentido de la decisión 5.7 (una interfaz, peers por contacto, direccionamiento interno) o si cambia el modelo.
4. Alternativas: capacidades de Linux (`setcap`), un pequeño servicio auxiliar privilegiado, interfaz en espacio de usuario con TUN, u otros.
5. Impacto sobre la elección pendiente entre WireGuard y WebRTC.
6. Qué partes de Enola (`VpnPort`, dominio VPN) siguen siendo aprovechables en cada alternativa.

No elijas mecanismo definitivo. Presenta alternativas y espera mi autorización.

### Cuestiones pendientes posteriores (en este orden, salvo que yo indique otro)

1. ~~Confirmar formalmente la vía C de reutilización de Enola y la propuesta «aplicación sin root».~~ **RESUELTA 2026-11-12:** ambas aprobadas (ver apartado 14); añadido el requisito de dependencias open source.
2. ~~Revisar la regla 8: ¿se permiten pruebas en Linux con máquinas virtuales o network namespaces?~~ **RESUELTA 2026-11-12:** permitidas (VM Linux local + netns; ver regla 8).
3. ~~D-12.4 (comprobación al consultar o al depositar) y cuánto aprende el colaborador sobre la relación B-D.~~ **RESUELTA 2026-11-12:** al depositar, con cifrado por destinatario, IDs blinded rotatorios y consulta vía onion; reinterpretación de 7.2 aprobada; filtración residual por correlación declarada (ver 7.5).
4. ~~PCP (RFC 6887): soporte real en routers domésticos y alternativas para la conectividad entrante.~~ **INVESTIGADA 2026-11-12:** informe `_bmad-output/planning-artifacts/research/technical-pcp-conectividad-entrante-2026-11-12/research.md`. Propuesta: punching coordinado como base + pinhole PCP oportunista + clasificador de red en runtime + móvil carrier-filtrado declarado inalcanzable. **APROBADA 2026-11-12 (ver §9).**
5. ~~WireGuard frente a WebRTC en Linux.~~ **RESUELTA 2026-11-12:** transporte = WebRTC vía `str0m` (ver 5.8); «interfaz» 5.7 reinterpretada como lógica (ver 5.7).
6. ~~Mecanismo de descubrimiento (ninguno aprobado). Modalidad por defecto~~: modalidad A por defecto (§6); protocolo concreto **RESUELTO 2026-11-13 — D-16.6 (ver §8):** rendezvous derivado por pareja (familia BRP) + capa de custodia en contactos mutuos. Quedan como cuestiones de diseño abiertas: periodo de rotación/épocas y forma de autorización en custodia.
7. ~~Licencia de la aplicación de mensajería y de lo que se reutilice de Enola.~~ **RESUELTA 2026-11-12:** misma licencia que Enola, titular Salvador Palma Rodríguez; el material de terceros reutilizado debe ser open source (ver apartado 14).

### Siguiente fase (decidido 2026-11-13)

Cerradas las 7 cuestiones de análisis, se pasa a la **canalización de planificación BMM ortodoxa**: `bmad-product-brief` → `bmad-prd` → (`bmad-ux`, a evaluar) → `bmad-architecture` → `bmad-create-epics-and-stories` → `bmad-sprint-planning`. Las tres cuestiones de diseño pendientes de D-16.6 (autorización en custodia, parámetro de rotación, formato del registro) se resolverán dentro de la fase de arquitectura, con el PRD fijando primero los requisitos. Este documento es la fuente de entrada para brief y PRD; sigue siendo el registro autoritativo de decisiones.

### Decisiones del product brief (2026-11-13)

Tomadas por Salvador durante `bmad-product-brief`. Brief cerrado (`status: final`): `_bmad-output/planning-artifacts/briefs/brief-mensajeria-p2p-2026-11-13/` (`brief.md`, `addendum.md`). No modifican ninguna decisión anterior; acotan la primera versión (v1).

**DECISIÓN APROBADA (problema):** el problema que resuelve el producto es que hoy, para hablar en privado con alguien, hay que confiar en un tercero.

**DECISIÓN APROBADA (público):** la v1 es para usuarios de Linux; empieza con Salvador y las personas con las que acuerde usarla, y el círculo crece por emparejamientos presenciales. El público general es visión a largo plazo.

**DECISIÓN APROBADA (distribución):** podrá usar la aplicación cualquiera que la descargue del canal que el titular decida; los usuarios no se la pasan entre sí. El canal concreto sigue fuera de alcance (apartado 4) y la licencia del apartado 14 no cambia.

**DECISIÓN APROBADA (alcance de la v1):**

- Aplicación de consola por comandos (listar contactos, enviar un mensaje, conversar con un contacto); no es una interfaz de texto a pantalla completa. La conversación es una sesión interactiva por líneas. La interfaz gráfica queda para versiones posteriores.
- Solo texto, solo entre dos personas. Imágenes y archivos quedan para el futuro.
- Sin grupos. Unas salas de chat con varios equipos son una posibilidad futura sin forma definida; su compatibilidad con 5.3 y 5.5 no está analizada ni aprobada.
- Cola local en el equipo del emisor con detección de presencia: si el contacto no está disponible, el mensaje queda pendiente en el equipo del emisor y se envía cuando se detecta al contacto (ver la aclaración de 5.5). Limitaciones declaradas: los contactos saben cuándo estás en línea; el envío exige que la aplicación del emisor siga en marcha cuando el contacto aparezca; los mensajes entregados tarde muestran cuándo se escribieron. El mecanismo de presencia no está aprobado (cuestión de arquitectura).
- Solo Modalidad A; la Modalidad B queda fuera de la v1.

**DECISIÓN APROBADA (promesa sobre el emparejamiento):** el producto promete «verificación presencial del emparejamiento»; no promete nada sobre quién porta el dispositivo después.

**DECISIÓN APROBADA (criterios de éxito de la v1):**

1. Dos equipos se emparejan y mantienen una conversación normal de texto, de ida y vuelta.
2. Funciona con un mínimo de tres equipos y todas sus parejas.
3. Un mensaje enviado con el contacto ausente queda pendiente en el equipo del emisor y se entrega cuando el contacto aparece.
4. Se comprueba que ningún tercero transporta ni almacena el contenido y que nada exige root.

Los tres equipos son máquinas virtuales y network namespaces (regla 8): el éxito queda demostrado en simulación, no en redes reales.

**DECISIÓN APROBADA (pruebas con Tor):** las pruebas del descubrimiento usan la red Tor, igual que la aplicación (ver regla 8).

**DECISIÓN PENDIENTE:** cómo se concilian las pruebas que dependen de la red Tor con la convención de tests deterministas, sin dependencias de timing (`AGENTS.md`). HECHO: los tiempos de publicación y resolución de servicios onion dependen de una red que el proyecto no controla. HIPÓTESIS sin verificar: una red Tor privada local serviría para los tests automáticos, dejando la red Tor real para la comprobación extremo a extremo. Cuestión para arquitectura.

**DECISIÓN APROBADA (siguiente paso):** `bmad-prd`, con el brief y su addendum como entrada.

### PRD en curso (2026-11-13)

> **HECHO (nota de estado, 2026-11-13):** `bmad-prd` está iniciado y **sin terminar**. Borrador (`status: draft`): `_bmad-output/planning-artifacts/prds/prd-mensajeria-p2p-2026-11-13/` (`prd.md`, `addendum.md`, `.memlog.md`). El punto exacto de reanudación está al principio de `prd.md`. El siguiente paso sigue siendo terminar el PRD; no se ha pasado a arquitectura.

Tomadas por Salvador durante `bmad-prd`. No modifican ninguna decisión anterior; concretan la v1.

**DECISIÓN APROBADA (recorrido de emparejamiento):** los dos lanzan el comando de emparejar; hay una comprobación visual obligatoria mirando ambas pantallas; confirman los dos y es todo o nada (si uno no confirma o no coincide, no queda contacto en ningún equipo); cada uno pone al otro un nombre local que el otro no ve.

**DECISIÓN PENDIENTE (mecanismo de emparejamiento):** no elegido (5.2). Criterio de Salvador: «algo clásico pero funcional y seguro». Candidatos: QR, código, localización por proximidad. Sea cual sea, debe ser presencial (reafirmado por Salvador el 2026-11-13). Cuestión para arquitectura.

**DECISIÓN APROBADA (proceso de fondo):** la aplicación tiene un proceso de fondo propio que el usuario arranca, para y consulta con comandos, sin root. Arrancado, el equipo es alcanzable, recibe y entrega los pendientes; parado, está ausente. Los comandos hablan con él. Puede ejecutarse en primer plano en una terminal o como servicio de usuario opcional; el cómo es cuestión de arquitectura. El emisor no sabe cómo se ejecuta la aplicación del destinatario.

**DECISIÓN APROBADA (estados del mensaje):** dos estados vistos por el emisor: pendiente (en la cola de su equipo) y entregado (el equipo del destinatario lo confirma). Sin confirmación sigue pendiente y se reintenta, sin duplicados. No hay estado «leído» en la v1.

**DECISIÓN APROBADA (conservación de las conversaciones):** las conversaciones permanecen en el equipo y solo se borran manualmente, por alguno de los dos usuarios. El borrado lo solicita uno y se aplica en los dos equipos; funciona como los mensajes: si la acción de borrado no puede enviarse, queda en reserva en el equipo que la pide y se envía en cuanto el otro tenga la aplicación activa. HECHO (limitación): el borrado en el equipo del otro lo ejecuta su aplicación y no es verificable ni imponible desde el propio equipo. Límite de 100 mensajes por conversación, para liberar espacio: al superarlo se eliminan solo los más antiguos según entran los nuevos (siempre se conservan los últimos 100), previo aviso en el mensaje 99. Descartados en la misma sesión el borrado al cerrar el chat o al parar la aplicación y el borrado automático a los 10 minutos.

**DECISIÓN PENDIENTE (detalles de la conservación):** si el aviso del límite pide confirmación y si la eliminación por límite se aplica también en el otro equipo; si los mensajes pendientes de la cola cuentan para el límite. Cuestiones para Salvador al reanudar el PRD. La protección en disco de las conversaciones guardadas es cuestión de arquitectura.

## 17. Formato obligatorio de los análisis

Para cada cuestión:

1. Pregunta que intentamos resolver.
2. HECHOS documentados.
3. HIPÓTESIS pendientes.
4. Alternativas técnicas.
5. Ventajas y desventajas.
6. Incompatibilidades con decisiones aprobadas.
7. Conclusión técnica.
8. Decisión que necesito tomar, si procede.

Investiga fuentes oficiales y especificaciones actuales. Cita las afirmaciones técnicas importantes. Cuando afirmes algo sobre Enola, consulta su código y documentación actuales.

No escribas código todavía. No modifiques decisiones anteriores.

## 18. Objetivo final

Construir una aplicación de mensajería P2P realmente distribuida, primero en Linux, sin servidor central que gestione la red y con transporte de mensajes exclusivamente directo entre los dispositivos participantes.

La colaboración voluntaria debe permitir que la red disponga de infraestructura auxiliar sin sacrificar: privacidad, identidad independiente por dispositivo, confianza bilateral, ausencia de relaciones transitivas, ausencia de directorio global, autonomía de los dispositivos y comunicación directa.

**Comienza exactamente en el apartado 16. No vuelvas a empezar el proyecto desde cero.**
