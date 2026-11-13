---
title: 'technical research: descubrimiento-p2p'
type: 'technical'
topic: 'descubrimiento-p2p'
decision: 'Elegir el protocolo concreto de descubrimiento/rendezvous (§16 cuestión 6) compatible con D-12.4, la cascada de conectividad de §9 y la modalidad A por defecto.'
source: 'native run'
status: complete
preset: 'standard'
validation: 'normal'
claims: {verified: 21, unverified: 3, overturned: 0}
created: '2026-11-13'
updated: '2026-11-13'
---

# technical research: descubrimiento-p2p

**Decision this research serves:** elegir el protocolo concreto de descubrimiento/rendezvous para una mensajería P2P sin servidores ni relays, con depósito de registros solo en contactos directos del colaborador, registros cifrados por destinatario, IDs blinded rotatorios, consultas vía onion services, y que provea el canal de intercambio de endpoints para punching coordinado UDP.

## Resumen ejecutivo

El mecanismo que satisface todos los requisitos aprobados es un **rendezvous derivado por pareja, de la familia BRP**: los endpoints se *computan* offline a partir del secreto DH de cada pareja de contactos y se materializan como servicios onion v3 por pareja — no se deposita nada en terceros, así que «registros solo en contactos directos» se cumple de forma trivial. Las tres carencias del protocolo BRP tal cual (ventana única de 48 h, sin rotación de identificadores, un servicio por pareja) tienen arte previo resoluble: rotación por época derivada (periodo ≥ horas–días; ~minutos es el suelo) [14][15][16] y rotación anunciada in-band [16].

Hallazgos que dirigen la respuesta:

1. **Ninguna DHT encaja**. Toda DHT requiere que nodos extraños custodien/enruten (contradice D-12.4), no existe DHT WAN sin primer contacto conocido [28], Kademlia vanilla no tiene resistencia Sybil [29], y una población mayoritaria en modalidad A ni siquiera forma una DHT — el enrutamiento *es* la colaboración.
2. **libp2p rendezvous topológicamente encaja pero el protocolo no**: los registros son signed peer records en claro (multiaddrs visibles), sin autenticación ni cifrado por destinatario en el spec [19][21]; la restricción a contactos se haría fuera del protocolo [20]. Reusarlo sería protocolo propio sobre su plumbing de streams, sin ganancia.
3. **Tor es el único sustrato maduro para el canal rendezvous**: onion v3 ya implementa blinded keys rotatorios diarios [1] y restricted discovery por cliente (cap ~160) [2][4]; el coste es latencia de segundos (p50 ~1–5 s el tramo rend) y ciclos de reintento de minutos con peer recién conectado [34][35][36].

**Caveat principal:** no existe sistema desplegado que implemente exactamente D-12.4; la composición aprobada es novedosa. Los precedentes más cercanos son BRP (endpoints derivados), Briar Mailbox (custodia en dispositivo del propietario) y RetroShare Discovery (propagación entre contactos mutuos, pero en claro y transitiva — precedente de viabilidad, no modelo) [10][13][41].

## Dimensión 1 — Tor Onion Services v3 + blinded keys

**Mecánica.** El descriptor de un servicio v3 se publica en HSDirs indexado por una clave Ed25519 *blinded* que rota cada periodo (~diario); derivable solo por quien conoce la clave pública del servicio — los HSDirs no pueden enumerar ni enlazar descriptores a la dirección onion [1]. El cuerpo va cifrado bajo la clave blinded: los HSDirs ven un blob, no los introduction points [1]. El rendezvous point solo ve una cookie aleatoria de un solo uso; el intro point ve una clave efímera del servicio y puede contar/medir tiempos pero no atar peticiones a la dirección [1].

**Restricted discovery** (antes «client authorization»): la capa interna del descriptor se superencripta por clave x25519 de cada cliente autorizado; sin clave, los intro points son ilegibles [2]. Estable en C tor desde 0.3.5.1-alpha y en **Arti 1.7.0 (2025-11-03)** tras feature flag [3]. Cap recomendado: **160 clientes** por descriptor [4]. Advertencia documentada: es «un mecanismo anti-DoS, **no** un sustituto del control de acceso» — un cliente desautorizado puede seguir alcanzando el servicio hasta que expiren/roten los descriptores [4]. La segunda capa de autenticación del spec (firma por cliente en INTRODUCE1) no está implementada ni prevista en C tor ni Arti [5].

**Madurez de implementaciones.** C tor 0.4.8 quedó EOL 2026-06-01 y **bloqueado de la red ~2026-09-01**; la serie soportada es 0.4.9 — consistente con el requisito ya aprobado en §13 [6]. Arti va por **2.8.0 (2026-11-03)** con cadencia ~mensual [7]. **Contradicción verificada:** `doc/OnionService.md` en `main` sigue diciendo «suitable for testing and experimentation only… may even compromise the privacy of your other uses» (verificado 2026-11-13) — el texto es de feb-2024 y está obsoleto en detalles (vanguards-lite es default desde 1.2.2, restricted discovery es estable, no hay efímeros pero los state-dir existen), pero **ningún commit/documento oficial lo rescinde** [8][9]. La condición de §13 para adoptar Arti **sigue sin cumplirse** según la fuente oficial.

**Empotrable.** El hosting de onion services ya es in-process: `TorClient::create_onion_service` / `tor-hsservice` con `OnionServiceConfigBuilder`, `RunningOnionService`, `StreamRequest`; hay ejemplo in-tree de servicio HTTP en un proceso [39]. Caveats: conexiones cliente a .onion desactivadas por defecto (`allow_onion_addrs`); **no hay onion services efímeros** en Arti (#1186 abierto — servicios requieren state dir); APIs experimentales, MSRV Rust 1.91 [39][7].

**Gestión en runtime.** C tor: `ADD_ONION` acepta `ClientAuthV3` al crear; añadir `.auth` + `SIGNAL RELOAD` es aditivo, pero **revocar exige reiniciar el proceso tor** (o `DEL_ONION` + recrear) [38]. Arti: live reload con `watch_configuration` — cambios en `key_dirs`/static_keys se detectan y republican descriptores (con rate limit — agrupar cambios); `key_dirs` acepta formato C tor; hay RPC desde 2.7.0 [4]. Lado cliente: `ONION_CLIENT_AUTH_ADD/REMOVE/VIEW` en C tor [38]; `arti hsc key get` en Arti [4].

**Latencia y fallos.** Tramo rendezvous (service-side): ~99,8 % de builds ≤5 s (histograma de un solo operador — confianza media) [34]. Servicio recién publicado: «usualmente alcanzable en el primer intento» (Briar); timeout interno del cliente ~2 min [35]. Descriptores obsoletos: el cliente reintenta intro points y refetchea automáticamente (caché de fallo por IP ~2 min) — causa reintentos de decenas de segundos, no cuelges indefinidos [36]. El caso Quiet de «15+ min inalcanzable» es del lado servicio: peer recién conectado cuyos descriptores aún no propagaron [35][32]. Límite duro: **32 circuitos en construcción simultáneos** por proceso Tor — hay que serializar marcados a contactos (lección Cwtch) [37]. Vanguards-lite por defecto en ambos stacks; full solo recomendado para servicios >1 mes de uptime [9].

## Dimensión 2 — Briar BRP

**Spec (verificado contra BRP.md en bruto).** Cadena: `raw_secret = X25519-DH` → `static_master_key = HASH("…transport/STATIC_MASTER_KEY", raw, pub_a, pub_b)` → `rendezvous_key = KDF(smk, "…rendezvous/RENDEZVOUS_KEY", int_8(version=0))` → `stream_key = KDF(rk, "…rendezvous/KEY_MATERIAL", transport_id)` → `key_material = STREAM(sk, 64)`: 32 B semilla Ed25519 de Alice + 32 B de Bob, cada una la clave maestra de un onion v3 [10]. Cada par publica su onion derivado y marca el del otro, 1 intento/min hasta 48 h; BRP no autentica al peer — devuelve la conexión + static_master_key a la capa de aplicación [10]. Propósito declarado: rendezvous **inicial** tras el intercambio de claves, no disponibilidad continua [10].

**Modelo de adversario.** Adversario con ambas claves públicas no puede computar los endpoints (protegidos por el DH); no se deposita nada en terceros — los únicos artefactos son descriptores onion cifrados y blinded en la infraestructura Tor ya aprobada [10]. MITM solo si el canal de intercambio previo de claves es modificable — nuestro emparejamiento presencial (§5.2) ya lo cubre [10].

**Salud y reusabilidad.** Briar está en **modo mantenimiento** desde 2026-07-09 (solo fixes esenciales; el cierre de 2025 se revirtió por respuesta comunitaria); releases siguen (1.5.24, 2026-10-30) [11]. Licencia: core **GPLv3**, Desktop **AGPLv3** — la spec es pública y reimplementable, el código Java no es directamente reusable y la licencia copia solo si se importa código [12]. Bramble no es librería publicada; ningún otro mensajero lo implementa [11]. Existe `bramble` v0.1.0 (CC0, 2021) en crates.io: núcleo fiel a la spec **pero labels KDF distintas** (`…transport/…` vs `…rendezvous/…` del spec — verificado) → **wire-incompatible con Briar** y sin transporte Tor incluido; solo usable como referencia [17].

## Dimensión 3 — Rendezvous bilateral sin servidores

**Patrones vivos.** Cwtch (mailbox-on-own-onion por perfil, activo: lib v0.30.0 / UI v1.16.3; el identificador contiene el onion — estático y sondeable) [30]. Ricochet-Refresh (contacto = `ricochet-v3://{service-id}`; activo, v3.0.47, tor 0.4.9+vanguards-lite) [31]. Quiet (onion por miembro + lista vía OrbitDB sobre Tor; muy activo pero deriva hacia infraestructura opcional QSS; reportó el caso de 15 min inalcanzable) [32]. Ninguno rota identificadores — esa es la ventaja del modelo derivado [30][31][32]. Tapir (Go, MIT) es la extracción de Cwtch de primitivas onion+auth — referencia de mecánicas, mantenimiento escaso [18].

**Rotación de onions derivados — factibilidad.** Höller et al. (WTMC 2021): provisioning v3 ~5 s dominado por subida de descriptores (16 uploads iniciales); servicios dinámicos «ya viables para P2P en ciertos escenarios» [14]. Suelos de rotación: (a) propagación → periodos ≫60 s o fracción del tiempo inalcanzable; (b) epoch HSDir (1440 min default) [15][1]. Práctico: **horas–días cómodo; ~minutos el suelo; sub-minuto rompe** [15]. Arte previo: meshtor publica 3 épocas concurrentes (ayer/hoy/mañana, tolerancia skew) [15]; Zerion rota cada 5–14 días anunciando el nuevo onion in-band con old+new concurrentes [16]. Rotación derivada ≠ revocación ni forward secrecy: quien posee la semilla deriva todas las épocas [15].

**Custodia en contactos (precedentes).** Briar Mailbox = depósito en segundo dispositivo del *propietario*, HTTP+Bearer, token/inboxId/outboxId por contacto — «solo entrega entre el propietario y sus contactos, nada se expone a terceros» [13]. RetroShare Discovery: propagación de endpoints entre amigos mutuos funciona en producción («basta un amigo mutuo conectado») pero comparte en claro y transitivamente — precedente de viabilidad, no modelo [41]. Freenet darknet: registros de contacto cifrados sobre overlay F2F (2010) [42]. GNUnet: HELLOs `friend_only` nunca distribuidos a no-amigos [43]. Swirlwave: agenda poblada presencialmente (NFC) con onion por contacto [44]. SimpleX: colas en relays SMP — descartado por la regla de relays, útil solo como referencia (IDs efímeros por cola, sin identificador global) [45]. **Ninguno implementa D-12.4 exacto** — la composición es novedosa.

## Dimensión 4 — libp2p rendezvous

Spec **Working Draft r3 (2021), nunca maduró** [19]. Mecánica: `REGISTER` con namespace + signed peer record (auto-registro), `DISCOVER` con cookie de paginación, TTL default 2 h (máx 72 h) [19]. **Lo que aprende el punto**: PeerID + multiaddrs **en claro** + namespaces; `DISCOVER` sin namespace devuelve todos los registros — **sin control de acceso en el spec** [19]. rust-libp2p confirmó que la autenticación va **fuera** del protocolo (wrapper behaviour; precedente: Lattice en modo privado con allowlist) [20]. El registro es estrictamente un signed peer record — **no admite payload opaco** [21]. Es federado («single point of failure», asume puntos conocidos) pero **no depende de kad-DHT** [19]. Estado: `libp2p-rendezvous` mantenido en Rust (v0.18.0, MIT); go y js obsoletos [21]. **Evaluación:** la topología contactos-como-puntos encaja, pero cifrado-por-destinatario, IDs rotatorios y autorización habría que construirlos encima — sería protocolo propio, no adopción.

## Dimensión 5 — Familia Kademlia/DHT

**Tox**: Kademlia sobre UDP, clave DHT efímera por reinicio, módulo onion de 3 saltos para localizar amigos ocultando IP; bootstrap vía nodos bien conocidos (`tox-bootstrapd`, lista pública) aunque los clientes cachean nodos y re-bootstrap desde ellos — bootstrap = conveniencia de primer contacto, no dependencia permanente [22]. c-toxcore activo (v0.2.24, 2026-11-01), **GPLv3**; tox-rs muerto; nueva impl Rust de TokTok sin repo público localizable — sin camino Rust hoy salvo FFI [23][24].

**OpenDHT/Jami**: putEncrypted cifra valores a cert de destinatario (no editables); transporte DHT sin cifrar (issue abierta) [25]. Bootstrap `bootstrap.jami.net` por defecto, caché persistente de nodos [25][27]. Activo (v4.4.1), **MIT desde v3.7.1**; crate `opendht` oficial pero enlaza C libopendht-c, no puro Rust [25][26]. Jami almacena DeviceAnnouncement en DHT global (TTL ~10 min) — mecánicamente demuestra que un canal rendezvous puede portar ICE/endpoints para punching, pero viola «registros solo en contactos» [27]. Keet/Hyperswarm anuncia registros firmados bajo topics de 32 B en HyperDHT — la idea «topic como identificador opaco con clave» es afín, la ubicación (DHT global) no [33].

**Genérico**: no existe DHT WAN sin primer contacto [28]; Kademlia vanilla sin resistencia Sybil y eclipsable en redes dispersas [29]; una población mayoritaria no-colaboradora no forma DHT — el enrutamiento *es* la colaboración [28][29]. **Veilid** (Rust, MPL-2.0, activo): capa de datos = DHT global; no hay modo de custodia en contactos → no encaja [40].

## Hallazgos transversales

- **La pregunta «custodia» tiene dos respuestas que el modelo puede combinar**: rendezvous derivado bilateral (BRP) no deposita nada y cubre contacto directo↔directo; la custodia en contactos mutuos (D-12.4) queda como mecanismo complementario/robustez, con Briar Mailbox como precedente de protocolo y RetroShare como prueba de viabilidad a escala. No son alternativas sino capas del mismo subsistema.
- **Todos los candidatos chocan con el mismo muro de arranque**: rendezvous necesita un punto alcanzable, DHT un primer nodo — inherente a «sin servidores». El modelo derivado es el único que lo resuelve *sin* estado previo en ningún tercero (ambas partes computan el endpoint).
- **Los identificadores rotatorios vienen «gratis» dos veces**: Tor ya blinda y rota diariamente lo visible a los directorios [1]; la extensión por época derivada añade rotación de la dirección misma [15][16]. Ningún proyecto en producción hace rotación derivada de onion por pareja — meshtor es la implementación de referencia del patrón.
- **El coste Tor es latencia, no viabilidad**: segundos para peer publicado+online; reintentos de minutos para peer recién conectado — dimensiona los timeouts del punching con techo ~120 s y lógica de reintento [34][35][36].

## Evidencia contraria y contradicciones

- **Arti hosting**: el doc oficial sigue diciendo «testing only» pese a features estables desde 1.7.0 — la condición de §13 para Arti sigue abierta; C tor permanece la implementación aprobada [8][3].
- **Rendezvous como SPOF**: cada pareja depende de dos onion services propios; si el contacto está offline no hay rendezvous (igual que todo el modelo — coherente con §5.4) [19].
- **Quiet 15-min**: fallos de alcanzabilidad tras reconexión del peer — mitigable con lógica de reintento, no evitable [35][32].
- **Restricted discovery no es control de acceso**: la revocación real depende de rotación de descriptores; en C tor además exige reinicio [4][38].
- **libp2p rendezvous sin auth**: usarlo «tal cual» expondría endpoints en claro a cualquiera que marque — elimina la vía «crate estándar sin más» [19][21].

## Recomendaciones

1. **PROPUESTA: rendezvous derivado por pareja estilo BRP como mecanismo primario**, extendido con épocas (periodo ≥ horas–días) y publicación concurrente de época anterior/actual/siguiente para tolerancia de skew — precedentes meshtor/Zerion. Confianza: alta (spec verificado + arte previo de rotación) [10][15][16].
2. **PROPUESTA: capa de custodia en contactos mutuos** para el rol D-12.4, con forma de «mailbox de registros de endpoint» cifrados por destinatario accesible vía el onion del colaborador — precedente Briar Mailbox; reforzar autorización con restricted discovery donde aplique (cap 160 = cota de agenda) [13][4].
3. **Canal de punching**: el canal rendezvous (onion, TCP) porta el intercambio de endpoints UDP — el punching ocurre *después* del rendezvous, igual que BRP cede la conexión a la capa de aplicación [10][27]. Timeouts: ≥120 s por intento, reintentos en ventana de minutos, marcados a contactos serializados (cap 32 circuitos) [36][37].
4. **No adoptar libp2p rendezvous ni ninguna DHT** — incompatibilidades documentadas, no carencias subsanables sin convertirlo en protocolo propio [19][28][29].
5. **C tor 0.4.9+ como proceso de usuario** (decisión §13 confirmada por esta investigación); trackear el doc de Arti — cuando se rescinda el aviso, Arti ofrece embedding in-process y live reload superiores al reinicio-exigido de C tor [6][8][38][39].
6. **Vigilar las licencias de referencia**: Briar GPL/AGPL y Tox GPL — reimplementar desde specs, no importar código copyleft (la licencia propia es source-visible de Enola) [12][24].

## Cuestiones abiertas

- Latencia cliente-observada real de conexión onion (p50/p99) — solo hay histograma de un operador y anécdotas; un Onionprobe local o los dashboards públicos darían cifras para dimensionar timeouts. Ruta: Onionprobe en VM local o métricas publicadas [34].
- ¿Declaró Arti formalmente «producción» el hosting de onion services? El doc sigue sin rescindirse; monitorizar releases mensuales [8].
- Parámetros óptimos de rotación derivada (periodo, número de épocas concurrentes) — los suelos están acotados, el punto óptimo para agenda de mensajería no [15][16].
- Superficie RPC de Arti 2.7+ para mutar claves de restricted discovery sin reinicio — documentado parcialmente [4].
- Especificación del formato de registro custodiado (cifrado por destinatario, ID blinded, TTL) — cuestión de diseño, no de investigación.

## Apéndice de fuentes

| # | Afirmación que soporta | Editor | Fecha pub. | Accedido | Confianza |
|---|---|---|---|---|---|
| 1 | Blinded keys rotatorios diarios; HSDir no lee descriptores; cookie de rendezvous | [Tor Project spec — rend-spec-v3](https://spec.torproject.org/rend-spec/) | vigente | 2026-11-13 | alta |
| 2 | Restricted discovery: supercifrado por cliente x25519 | [Tor Project spec — hsdesc-encrypt](https://spec.torproject.org/rend-spec/hsdesc-encrypt.html) | vigente | 2026-11-13 | alta |
| 3 | Restricted discovery estable en Arti 1.7.0 | [Tor Project blog](https://blog.torproject.org/arti_1_7_0_released/) | 2025-11-03 | 2026-11-13 | alta |
| 4 | Cap 160 clientes; live reload; «no es control de acceso»; rate limit republicación | [docs.rs tor-hsservice](https://docs.rs/tor-hsservice/latest/tor_hsservice/config/restricted_discovery/index.html) | 2026 | 2026-11-13 | alta |
| 5 | Intro-auth (capa 2) no implementada en C tor ni Arti | [Onion Services Ecosystem matrix](https://onionservices.torproject.org/dev/implementations/) | act. 2026-08-19 | 2026-11-13 | alta |
| 6 | Tor 0.4.8 EOL/bloqueado; 0.4.9 serie soportada | [Tor Project blog](https://blog.torproject.org/sunsetting-tor-048/) | 2026 | 2026-11-13 | alta |
| 7 | Arti 2.8.0, cadencia mensual, MSRV 1.91 | [Tor Project blog](https://blog.torproject.org/arti_2_8_0_released/) | 2026-11-03 | 2026-11-13 | alta |
| 8 | doc/OnionService.md sigue «testing only» (verificado en main) | [Arti repo](https://gitlab.torproject.org/tpo/core/arti/-/raw/main/doc/OnionService.md) | texto 2024-02, vigente en main | 2026-11-13 | alta |
| 9 | Vanguards-lite default desde Arti 1.2.2; full para >1 mes | [Tor Project blog](https://blog.torproject.org/announcing-vanguards-for-arti/) | 2024-07-16 | 2026-11-13 | alta |
| 10 | BRP spec v0 completo (verificado raw) | [Briar Project — briar-spec](https://code.briarproject.org/briar/briar-spec/-/raw/master/protocols/BRP.md) | spec 2019, vigente | 2026-11-13 | alta |
| 11 | Briar en modo mantenimiento; releases continúan | [briarproject.org news](https://briarproject.org/news/2026-maintenance-mode/) | 2026-07-09 | 2026-11-13 | alta |
| 12 | Briar GPLv3 / Desktop AGPLv3 | [briar LICENSE.txt](https://code.briarproject.org/briar/briar/-/blob/master/LICENSE.txt) | vigente | 2026-11-13 | alta |
| 13 | Briar Mailbox: depósito por contacto en dispositivo del propietario | [briar-mailbox](https://github.com/briar/briar-mailbox) | 2023 | 2026-11-13 | alta |
| 14 | Provisioning v3 ~5 s; servicios dinámicos viables | [Höller et al., WTMC 2021](https://www.mroland.at/publications/2021-hoeller-wtmc/) | 2021-05 | 2026-11-13 | alta |
| 15 | Rotación derivada diaria + 3 épocas concurrentes | [meshtor rotate.rs](https://0rcus.vercel.app/git/r/meshtor/files/src/rotate.rs) | 2026 | 2026-11-13 | media |
| 16 | Rotación anunciada 5–14 días, old+new concurrentes | [Zerion B4 docs](https://github.com/zerionproject/Zerion/blob/0ad92a02/docs/wire/B4_ONIONWRAPPER_API.md) | 2026-05 | 2026-11-13 | media |
| 17 | Crate bramble: labels KDF distintas → wire-incompatible | [docs.rs bramble-rendezvous](https://docs.rs/bramble-rendezvous) | 2021-10-16 | 2026-11-13 | media-alta |
| 18 | Tapir: framework Go, MIT, mantenimiento escaso | [git.openprivacy.ca/cwtch.im/tapir](https://git.openprivacy.ca/cwtch.im/tapir) | últ. tag 2022-10 | 2026-11-13 | alta |
| 19 | libp2p rendezvous: draft r3, registros en claro, sin auth | [libp2p specs](https://github.com/libp2p/specs/blob/master/rendezvous/README.md) | r3 2021-07 | 2026-11-13 | alta |
| 20 | Auth fuera del protocolo (discusión mantenedor + caso Lattice) | [rust-libp2p #5638](https://github.com/libp2p/rust-libp2p/discussions/5638) | 2025–2026 | 2026-11-13 | alta |
| 21 | libp2p-rendezvous: signed peer record only, MIT, v0.18.0 | [crates.io](https://crates.io/crates/libp2p-rendezvous) | 2026-09 | 2026-11-13 | alta |
| 22 | Tox: UDP Kademlia + onion module + bootstrap nodes | [toktok.ltd/spec](https://toktok.ltd/spec) | vigente | 2026-11-13 | alta |
| 23 | c-toxcore v0.2.24 activo, GPLv3 | [TokTok/c-toxcore](https://github.com/TokTok/c-toxcore) | 2026-11-01 | 2026-11-13 | alta |
| 24 | tox-rs muerto; nueva impl Rust sin repo público | [tox-rs](https://github.com/tox-rs/tox) + [toktok.ltd](https://toktok.ltd/) | 2022-12 | 2026-11-13 | alta |
| 25 | OpenDHT: putEncrypted, MIT desde 3.7.1, transporte sin cifrar | [opendht repo](https://github.com/savoirfairelinux/opendht) | v4.4.1 2026-09 | 2026-11-13 | alta |
| 26 | Crate opendht enlaza libopendht-c (no puro Rust) | [docs.rs opendht](https://docs.rs/crate/opendht/latest) | 2026 | 2026-11-13 | alta |
| 27 | Jami: DeviceAnnouncement ~10 min en DHT; ICE en call request | [docs.jami.net](https://docs.jami.net/) | vigente | 2026-11-13 | alta |
| 28 | No existe DHT WAN sin primer contacto | [Tari University DHT report](https://delta1.github.io/tari-university/) | n/d | 2026-11-13 | alta |
| 29 | Kademlia sin resistencia Sybil; eclipse en redes dispersas | [S/Kademlia](https://giulianomega.com/) + [INRIA hal-00786438](https://hal.inria.fr/hal-00786438) | n/d | 2026-11-13 | alta |
| 30 | Cwtch: onion por perfil, activo, identificador sondeable | [cwtch.im](https://cwtch.im/cwtch.pdf) + releases | 2025-10-31 | 2026-11-13 | alta |
| 31 | Ricochet-Refresh: contacto = onion, activo | [ricochet-refresh releases](https://github.com/blueprint-freespeech/ricochet-refresh/releases) | 2026-10-21 | 2026-11-13 | alta |
| 32 | Quiet: onions por miembro + OrbitDB; caso 15-min; deriva a QSS | [TryQuiet/quiet](https://github.com/TryQuiet/quiet) | 2026-10-28 | 2026-11-13 | alta |
| 33 | Hyperswarm: topics en HyperDHT; blind-relay opcional | [holepunchto/hyperswarm](https://github.com/holepunchto/hyperswarm) | vigente | 2026-11-13 | alta |
| 34 | Tramo rend ~99,8 % ≤5 s (un operador) | [Tor forum](https://forum.torproject.org/t/tor-relays-ot-tor-metrics-for-hidden-services/13763) | n/d | 2026-11-13 | media |
| 35 | Briar: alcanzable 1er intento; Quiet: 15 min tras reconexión | [tor-dev archive](https://www.mail-archive.com/tor-dev@lists.torproject.org/msg12216.html) | ~2021 | 2026-11-13 | media |
| 36 | Reintento intro points + refetch automático; ~2 min | [rend-spec introduction-protocol](https://spec.torproject.org/rend-spec/introduction-protocol.html) | vigente | 2026-11-13 | alta |
| 37 | Cap 32 circuitos en construcción; serializar marcados | [Open Privacy — Cwtch updates](https://www.openprivacy.ca/discreet-log/34-cwtch-updates/) | ~2023 | 2026-11-13 | alta |
| 38 | ADD_ONION ClientAuthV3; revocación exige reinicio tor | [torspec control-spec + tor.1](https://archive.torproject.org/websites/lists.torproject.org/pipermail/tor-commits/2021-February/215483.html) | vigente | 2026-11-13 | alta |
| 39 | Hosting onion in-process en Arti; sin efímeros (#1186); allow_onion_addrs | [docs.rs arti-client + tor-hsservice](https://docs.rs/tor-hsservice) | 2026 | 2026-11-13 | alta |
| 40 | Veilid: DHT global, MPL-2.0, sin custodia en contactos | [veilid developer-book](https://veilid.gitlab.io/developer-book/concepts/dht.html) | 2026-11 | 2026-11-13 | alta |
| 41 | RetroShare Discovery: endpoints propagados entre amigos mutuos | [RetroShare docs](https://retroshare.readthedocs.io/en/latest/user-guide/settings/) | vigente | 2026-11-13 | alta |
| 42 | Freenet darknet: registros de contacto cifrados en overlay F2F | [Rogers & Bhatti, P2P'10](https://sites.cs.st-andrews.ac.uk/people/snb6/publications/2010/p2p2010/p2p2010-rb2010.pdf) | 2010 | 2026-11-13 | media |
| 43 | GNUnet: HELLOs friend_only no distribuidos | [docs.gnunet.org](https://docs.gnunet.org/v0.20.x/developers/peerinfo/peerinfo.html) | vigente | 2026-11-13 | media |
| 44 | Swirlwave: agenda NFC presencial con onion por contacto | [SCITEPRESS 2018](https://www.scitepress.org/PublishedPapers/2018/68301/pdf/index.html) | 2018 | 2026-11-13 | media |
| 45 | SimpleX: colas SMP en relays (referencia de diseño, descartado) | [simplex-chat docs](https://github.com/simplex-chat/simplex-chat/blob/stable/docs/SERVER.md) | vigente | 2026-11-13 | alta |

## Mapa de caducidad

Computado con `recon_kit.py staleness` (ventanas por clase del pack: versiones ≤1 mes, ecosistema/salud ≤6 meses, patrones y specs ≤2 años) sobre las 24 afirmaciones del ledger. Matiz aplicado: las afirmaciones de clase `spec`/`pattern` datadas por su publicación **no caducan por calendario sino por sustitución** — un spec de 2019 sigue vigente hasta que el proyecto lo reemplace; se re-chequean ante nueva versión del spec, no por fecha.

Re-chequeos accionables (afirmaciones volátiles que sí envejecen):

| Afirmación | Clase | Re-chequear |
|---|---|---|
| Versión Arti (2.8.0), cadencia y empotrabilidad | versión | 2026-12-13 |
| Aviso «testing only» en `doc/OnionService.md` (condición de §13 para adoptar Arti) | versión | 2026-12-13 |
| Serie soportada de C tor (0.4.9+) y restricted discovery (cap, live reload) | versión | 2026-12-13 |
| Salud de Briar (modo mantenimiento, releases) | ecosistema | 2027-05-13 |
| Salud de Cwtch / Ricochet-Refresh / Quiet / c-toxcore / OpenDHT / libp2p-rendezvous | ecosistema | 2027-05-13 |
| Latencia/caps cuantitativos (rend ~99,8 % ≤5 s; 32 circuitos; ~2 min refetch) | cuantitativa | 2027-05-13 |
| Specs (BRP v0, rend-spec-v3, libp2p-rendezvous r3) y patrones (rotación derivada, custodia en contactos) | spec/patrón | ante nueva versión del spec o supersición documentada |

Re-chequeo más próximo: **2026-12-13** (versiones y el aviso de Arti — la condición de §13 depende de él). Vía: `bmad-deep-recon` lifecycle Refresh sobre este run folder.
