---
title: 'technical research: PCP y conectividad entrante IPv6'
type: 'technical'
topic: 'PCP y conectividad entrante IPv6'
decision: 'Decidir la vía para conectividad entrante P2P en redes domésticas IPv6 sin servidores propios ni de terceros (alimenta §16.4, la elección WireGuard/WebRTC y el subsistema de conectividad)'
source: 'deep-recon run'
status: complete
preset: 'standard'
validation: 'normal'
created: '2026-11-12'
updated: '2026-11-12'
claims: '11 verified, 0 disputed, 0 overturned (memlog); afirmaciones adicionales en digests con confianza indicada'
earliest_recheck: '2025-12-01 (re-verificar bloque measured/source-code al consumir)'
---

# technical research: PCP y conectividad entrante IPv6

**Decision this research serves:** Decidir la vía para conectividad entrante P2P en redes domésticas IPv6 sin servidores propios ni de terceros (alimenta §16.4, la elección WireGuard/WebRTC y el subsistema de conectividad).

## Resumen ejecutivo

**Respuesta a la decisión:** la conectividad entrante P2P sobre IPv6 residencial es viable sin ningún servidor, pero **ningún mecanismo único la cubre** — el diseño correcto es una cascada de tres niveles que se autodetecta en tiempo de ejecución:

1. **~20-25% de las redes IPv6 residenciales no filtran el entrante** (tres estudios medidos independientes: CCS 2026, AsiaCCS 2024, ACM CSUR 2023). Ahí basta escuchar — nada que hacer [59][60][61][62].
2. **La mayoría filtra con firewall stateful tipo RFC 6092** (deny-inbound por defecto en OpenWrt, AVM, Xfinity), donde el envío mutuo coordinado de UDP abre el camino de retorno — probado en producción (EasyTier, Tailscale, PCAPs), más fácil que en IPv4 por no haber traducción de puertos. Requiere solo un canal para intercambiar endpoints, que la app ya tiene entre contactos [55][56][57][69][70][71].
3. **Donde falla lo anterior quedan los pinholes por protocolo** — PCP MAP-sobre-IPv6 (Fritz!Box + ecosistema miniupnpd; producto P2P real que ya lo hace: NetBird, mergeado abr-2026, verificado directamente) y `WANIPv6FirewallControl` como segunda vía. Cobertura minoritaria; NAT-PMP es IPv4-only e irrelevante aquí; UPnP-IGD está desactivado por defecto en gran parte del parque por razones de seguridad documentadas [32][36][45][48][49][54].
4. **Lo irremediable:** el filtrado de entrante en la red del operador móvil (T-Mobile, EE, carriers alemanes, reconocido en NANOG) y políticas CPE que bloquean categorías P2P enteras. Sin relays permitidos (decisión 5.5), esos contactos son simplemente inalcanzables — coherente con la decisión 5.4 (sin entrega garantizada) [64][65][66][67].

**Mayor caveat:** la dependencia irreductible no es el traversal sino el **rendezvous** — conocer la dirección:puerto del otro exige canal de señalización (el subsistema de descubrimiento del proyecto; Tor/onion encaja como canal auxiliar ya aprobado). Y ninguna medición pública cuantifica la cobertura de PCP en el parque de CPE — su contribución real es incuantificada, por eso se propone como mejora oportunista, nunca como mecanismo base.

## Síntesis entre dimensiones

- **La segmentación de CPE es la arquitectura**: «abierto / stateful / pinhole-capable / bloqueado» no es un detalle sino la clasificación que el subsistema de conectividad debe computar por contacto/red — combinar D3 (qué fracción de redes cae en cada clase) con D1/D2 (qué protocolo mejora cada clase) y D4a/D4b (qué mecanismo gratuito cubre la clase dominante).
- **El connectivity check ES el punch — gratis para ambos candidatos de transporte**: ICE-sin-STUN lo formaliza (checks = transacciones STUN peer-to-peer, host candidates como GUAs, prflx para direcciones aprendidas) [1][3], pero el handshake mutuo de WireGuard logra el mismo efecto de apertura de estado. Esto **reduce aún más el diferencial WireGuard-vs-WebRTC**: ICE aporta maquinaria sistemática de sondeo/selección de candidatos; WG lo haría a mano.
- **El patrón Briar, adaptado dentro de nuestras restricciones**: Briar delega toda la alcanzabilidad a onion services por contacto [23]. Nuestra decisión aprobada prohíbe Tor para transporte de mensajes, **pero no para señalización** — un onion por contacto como canal de rendezvous garantizaría el intercambio de endpoints incluso cuando el UDP directo aún no funciona: el papel exacto que el descubrimiento debe resolver.
- **Precedente de diseño validado**: NetBird (PR #5219, verificado) y Bitcoin Core (PCP propio post-UPnP) muestran la forma de cliente correcta: PCP primero → fallback → degradado grácil → nunca exigir UPnP [16][17][45].
- **Jami es el contraejemplo de restricciones**: funciona con ICE + UPnP + TURN *activado por defecto* contra su propio coturn — la vía que nuestro modelo prohíbe; ilustra por qué otros P2P "resuelven" el entrante con infraestructura que nosotros no aceptamos [21].

## Recomendaciones

Cada una ligada a la decisión y al artefacto que la consume:

1. **Mecanismo base: envío mutuo coordinado tras intercambio de endpoints** (sin cooperación del router). Cubre la clase dominante de CPE. Confianza: alta — especificación RFC 6092/5128 + tres evidencias de producción/test [55][69][70][71][72].
2. **Intento oportunista de pinhole: cliente PCP MAP-sobre-IPv6 primero, WANIPv6FirewallControl segundo.** Solo si el probe anterior no basta o para escucha pasiva. Confianza: alta en que funciona donde está; incuantificada su cobertura real (vacío de medición pública) [32][36][45][49].
3. **Clasificador de red en runtime** — sondear (no asumir) «abierto / stateful / pinhole / bloqueado», con degradado grácil. Plantilla de diseño: netbird portforward y bitcoin `-natpmp` [45][17].
4. **Keepalives <2 min** sobre el canal establecido (RFC 6092 REC-14: el estado UDP no debe expirar antes de 2 min ocioso) [55].
5. **Redes móviles: limitación declarada, no bug.** Documentar en la spec que contactos tras carrier-filtering son inalcanzables bajo 5.5; no diseñar parches (relay) que violen decisiones [64][67].
6. **Alimenta:** la elección WireGuard-vs-WebRTC (§16.5 — el punching es gratis en ambos; no decidir por conectividad), el contrato del subsistema de conectividad (§11.3) y el requisito de canal de rendezvous del subsistema de descubrimiento (§11.2 + rol auxiliar de Tor, §13).

## Preguntas abiertas

- **Cobertura real de PCP en el parque de CPE** — no existe medición pública; respondería un sondeo propio o datos de Censys/Shodan sobre UDP:5351 (no realizado en este run).
- **¿El filtrado carrier móvil es stateful-por-flujo o ACL estática?** — determina si el punching celular puede funcionar; no hay documentación primaria de carriers; solo se respondería con sondeo empírico (ver regla 8 del proyecto sobre pruebas permitidas).
- **Anomalía IPv6 de libp2p** (punching medido peor en IPv6 que en IPv4) — sin causa raíz publicada; si es artefacto de descubrimiento de direcciones, no nos aplica (sin NAT, la GUA es la propia dirección de interfaz).
- **Estado actual de Freebox** (`Inbound Pinhole Allowed: 0`, dato de ~2021) y de ASUSWRT stock (UPnP reportado roto en firmware reciente).
- **Comparación empírica punching-por-WG-handshake vs ICE** — no existe medición; nuestro propio test con VMs/network namespaces lo resolvería una vez aprobada la regla 8.

## Mapa de obsolescencia

Ventanas aplicadas (preset technical): `spec` 24m, `measured`/`source-code`/`vendor-doc`/`measurement`/`forum` 6m, `issue-tracker` 3m. Computado con `recon_kit.py staleness` a 2026-11-12.

| Claim | Clase | Publicado | Re-chequeo | ¿Vencido? |
|---|---|---|---|---|
| E. RFC 6092 deny-inbound + 5-tuple stateful | spec | 2011-01 | — (documento congelado) | n/a¹ |
| J. W6FC AddPinhole; NAT-PMP IPv4-only | spec | 2010-12 | — (congelado) | n/a¹ |
| A. ICE checks = STUN peer-to-peer | spec | 2018-07 | — (congelado) | n/a¹ |
| F. ~20-25% redes IPv6 sin filtrado | measured | 2026-01 | 2026-07 (vencido; próx. al consumir) | sí |
| G. Punching funciona en stateful IPv6 | measured | 2025-04 | 2025-10 (vencido) | sí |
| B. DCUtR ~70% agregado | measured | 2025-10 | 2026-04 (vencido) | sí |
| L. UPnP off por defecto en CPEs ISP | measured | 2025-06 | 2025-12 | sí (próximo) |
| H. Carriers móviles filtran entrante | measured | 2026-11 | 2027-05 | no |
| I. PCP IPv6 pinholes Fritz!Box/miniupnpd | source-code | 2026-11 | 2027-05 | no |
| K. NetBird PCP mergeado | source-code | 2026-04 | 2026-10 (vencido; recién) | sí |
| C. Bitcoin v29 sin UPnP | source-code | 2025-04 | 2025-10 (vencido) | sí |
| D. miniupnpd PCP off por defecto | source-code | 2022-01 | vencido hace tiempo | sí |
| M. Jami TURN default-on | source-code | 2026-05 | 2026-11 | sí (próximo) |
| N. Briar BRP onion por contacto | spec | 2024-01 | 2026-01 (vencido) | sí |

¹ Las clases `spec` con documentos inmutables (RFC, specs UPnP publicadas) se marcan fuera del cómputo: su contenido no cambia; solo se revisa si una implementación nueva los contradice.

**Interpretación honesta:** las afirmaciones *load-bearing* (E, F, G, H, I, A) descansan en especificaciones congeladas o en medición reciente convergente (tres estudios + docs de vendor + test reproducido); que varias marquen «vencido» por ventana de 6 meses significa *re-verificar al consumir el informe*, no que el contenido sea dudoso. **Fecha de re-chequeo más próxima significativa: 2025-12-01** (L) — y en la práctica, re-verificar todo el bloque `source-code`/`measured` si la decisión se consume después de 2027-05.

## D4b. ICE sin STUN: qué puede hacer realmente

Los connectivity checks de ICE son transacciones STUN Binding intercambiadas **entre los dos peers**; el servidor STUN solo existe para descubrir candidatos server-reflexive, no para ejecutar ICE [1]. Con cero servidores STUN configurados, un agente ICE recolecta únicamente candidatos host — comportamiento documentado de los stacks WebRTC (`{iceServers: []}` es legal) [2].

Los candidatos peer-reflexive (prflx) permiten a ICE aprender la dirección externa que el par observa **sin servidor STUN**: cuando llega un Binding request desde una dirección no listada, el receptor sintetiza un candidato prflx y lo prueba como otro cualquiera [1]. Limitación documentada: un check tiene que llegar entrante para que el prflx se forme — no crea alcanzabilidad que no existe; RFC 8863 (ICE PAC) existe precisamente porque los agentes podían declarar fallo antes de que llegaran los prflx [3].

En el escenario de la decisión — IPv6 tras firewall stateful — ICE con solo candidatos host es funcional de verdad: los candidatos host son GUAs reales (sin traducción → no hace falta srflx) y el check saliente de cada lado crea estado conntrack que admite el check entrante del par. **El connectivity check ES el hole punch.** Un firewall stateful por 5-tupla se comporta como un NAT port-restricted-cone sin el problema de reescritura de puertos — más fácil que el traversal de NAT IPv4 [4][5]. Evidencia de despliegue corroborante: el daemon de Jami omite explícitamente STUN para transportes IPv6 ("no NAT", commit de mayo 2026) [6], y hay un caso reportado de Jami completamente serverless sobre IPv6 (Yggdrasil) funcionando hasta una regresión a finales de 2025 [7].

Aviso solo-navegador: desde Chrome 76 los candidatos host se exponen a JS como nombres mDNS `.local`; si los pares no resuelven mDNS se degradan a prflx. Los stacks ICE no-navegador (pion, webrtc-rs, etc.) usan IPs literales y no se ven afectados [8].

ICE-TCP (RFC 6544) no es vía para entrantes entre extremos arbitrarios: los navegadores solo implementan candidatos TCP *activos*; los pasivos y simultaneous-open nunca llegaron a ningún navegador, y un par activo↔pasivo sigue requiriendo que un lado tenga listener alcanzable [9][10]. (Contradice parcialmente la medición DCUtR — ver D5.)

## D5. Qué hacen realmente las apps P2P en producción

**libtorrent / qBittorrent.** Implementa los tres protocolos de mapeo: UPnP IGD, NAT-PMP y PCP — intenta PCP primero con fallback a NAT-PMP, por socket/subred [11][12]. qBittorrent activa "UPnP/NAT-PMP" por defecto [13]. En IPv6: el mapeo UPnP IGD no hace nada, y `WANIPv6FirewallControl AddPinhole` de IGD2 **no está implementado** (issue abierta desde 2019) [14]; PCP sí puede abrir pinholes de firewall IPv6 y libtorrent lo soporta si el router habla PCP [12]. Además implementa BEP 55 `ut_holepunch` (desde 2015): un par del swarm conectado a ambos retransmite mensajes de rendezvous para que dos pares inalcanzables se marquen simultáneamente por uTP [15]. No funciona en torrents privados (PEX/DHT desactivados) [15].

**Bitcoin Core.** UPnP venía activado por defecto, se desactivó tras vulnerabilidades de miniupnpc y se **eliminó por completo en v29.0** (abril 2025). Su sustituto es una implementación propia de **PCP con fallback NAT-PMP**, opt-in vía `-natpmp`, que soporta explícitamente **pinholing IPv6** (traversal de firewall, no mapeo). Activarlo por defecto sigue en debate (issue #31663) [16][17][18]. Realidad documentada: el mecanismo entrante dominante es **Tor onion** — ~64-66% de los nodos alcanzables son `.onion` [19]; un crawl de julio 2026 encontró 99.1% de direcciones IPv6 gossiped inalcanzables (gran parte churn obsoleto, pero ilustra lo raro que es que una dirección clearnet autoanunciada sea marcable) [20].

**Jami.** ICE vía pjproject parcheado (incluye ICE-over-TCP). Candidatos anunciados por OpenDHT: direcciones de interfaz + "reflexive" vía UPnP + TURN — y **TURN viene activado por defecto** apuntando a coturn de jami.net [21][22]. IPv6 se trata como directamente alcanzable ("skip STUN") [6]. No se encontró uso de PCP ni tratamiento específico de IPv6 filtrado más allá de candidatos host + UPnP (ausencia de evidencia → confianza media) [21].

**Briar.** No requiere alcanzabilidad IP entrante en absoluto: por el Bramble Rendezvous Protocol, cada par de contactos deriva un secreto compartido y cada peer publica un **onion service Tor v3** dedicado (clave de identidad derivada de la stream key) conectando hacia el onion del par. Los onion services no necesitan port forwarding ni pinholes — Briar **delega enteramente la alcanzabilidad entrante a Tor** [23][24]. Sincronización local vía Bluetooth/Wi-Fi LAN.

**Tox.** UDP-first: la DHT (solo UDP) localiza peers y sirve de descubrimiento de dirección (los peers aprenden su IP/puerto externo de respuestas DHT — un STUN informal). Hole punching UDP conducido por la DHT; para NAT simétrico, routed ping + adivinación de puertos adyacentes. La caída de respaldo son **TCP relays** entre amigos (voluntarios con puertos TCP reenviados — "NAT-PMP y uPNP pueden ayudar"), porque "hole punching con TCP no es tan fiable" [25][26].

**Mediciones de éxito.** El hole punching DCUtR de libp2p: **70% ± 7.1%** de éxito agregado (paper IMC'26, 4.4M intentos en 85k redes); la campaña honeypot de ProbeLab midió 82.9% en NAT port-restricted-cone, ~40% en NAT simétrico, y ~36% de los peers NAT'd ni siquiera obtienen la reserva de relay necesaria para intentar el punch. TCP y QUIC fueron estadísticamente indistinguibles (~70% ambos) con marcado simultáneo sincronizado por RTT [27][28]. En IPv6 el punching de libp2p midió *peor* que en IPv4 — atribuido a que los peers carecen de dirección IPv6 pública observada que intercambiar (artefacto de descubrimiento de direcciones, no necesariamente comportamiento de firewall); issue aún abierta [29].

**Disponibilidad de protocolos pinhole en el lado router.** miniupnpd (el daemon UPnP/NAT-PMP/PCP de facto en CPE) distribuye con `enable_pcp_pmp` en **no** por defecto; OpenWrt se niega a incluirlo por defecto citando superficie de ataque [30][31]. UPnP IGD tiene mala reputación de seguridad documentada — exactamente la razón por la que Bitcoin lo retiró [16].

### Contradicciones registradas en esta dimensión

- **Fiabilidad del TCP hole punching**: los docs de toxcore afirman que "no es tan fiable" (justifica UDP+TCP relays), pero la medición IMC'26 de DCUtR encontró TCP y QUIC indistinguibles (~70%) con apertura simultánea sincronizada. Ambas citadas; la medición es más reciente y empírica pero específica de protocolo [25][27].
- **UPnP "peligroso" vs "necesario"**: qBittorrent mantiene el default-on pese a argumentos de riesgo (issue #7715 expone el debate); Bitcoin resolvió la misma tensión retirándolo [13][16].

### No encontrado en esta dimensión

- Tasas de éxito publicadas de BEP 55 ni de disponibilidad de PCP en poblaciones de CPE residencial.
- Documentación de Jami sobre IPv6 *filtrado por firewall* específicamente.
- Medición standalone (no-navegador) del éxito de ICE con solo candidatos host sobre IPv6 filtrado — la evidencia es inferencial.

## D1. Soporte real de PCP (RFC 6887)

El protocolo está diseñado para el caso que nos importa: PCP MAP controla tanto mapeos NAT44 como **firewall IPv6** — el mismo opcode produce un pinhole entrante según la familia de direcciones [32]. Cliente UDP 5350, servidor 5351.

**Dónde funciona de verdad (verificado por terceros):**

- **AVM Fritz!Box** — el soporte insignia. Servidor PCP en UDP 5351 por IPv4 e IPv6 (sondeado en hardware real); un MAP request sobre IPv6 crea un pinhole de firewall a la GUA del cliente [33][34]. Dos particularidades de interop documentadas: rechaza pinholes para direcciones IPv6 *temporales* [33][35] y devuelve NOT_AUTHORIZED si el toggle por dispositivo «Permit independent port sharing» está apagado [35]. AVM lo documenta oficialmente como vía de «port sharing» junto a UPnP IGD [34].
- **Ecosistema miniupnpd** (OpenWrt, pfSense/OPNsense, Turris, Asuswrt-Merlin, Gargoyle, EdgeRouter): `pcpserver.c` implementa el caso IPv6-firewall del opcode MAP [36]; el opcode PEER-firewall está stubeado (UNSUPP_OPCODE) — solo pinholes vía MAP [36]. Mantenimiento activo: changelog con fixes de pinholes IPv6 hasta el 2-nov-2026 [37]. En OpenWrt, pinholes IPv6 verificados funcionando (`upnpc -6 -A`); persistencia de leases v6 aún en PR abierta [38]. Tailscale verificó mapeos PCP sobre pfSense/OPNsense [39].

**Dónde NO hay:** MikroTik (solo NAT-PMP desde 7.13; sin pinholing IPv6 de ningún tipo) [40], UniFi (toggle presente pero roto; Ubiquiti recomienda mantenerlo desactivado) [41], TP-Link/Netgear (sin protocolo; la «solución» Netgear es modo Pass Through que lo abre todo) [42][43], CPEs de operador (Comcast/Vodafone: UPnP desactivado por defecto o ausente; Comcast lo *reverte* tras reboot según reportes) [44]. No hay evidencia de PCP en Netgear, TP-Link, D-Link, Linksys, Nest Wifi ni eero.

**Precedente en cliente P2P real (verificado directamente):** NetBird mergeó soporte PCP completo en abril 2026 — dual-stack, pinhole IPv6, monitorización de epoch del router y health checks ANNOUNCE [45]. Tailscale sondea PCP→NAT-PMP→UPnP [46]; libtorrent lo intenta primero (ver D5); Bitcoin Core implementó PCP propio tras retirar UPnP (ver D5). Transmission, en contraste, sigue sin pinholes IPv6 (issue abierta desde 2019) [47].

**Vacío de datos relevante:** no existe medición sistemática de despliegue PCP en la población de CPE — la cobertura es *minoritaria* fuera del ecosistema Fritz!Box/miniupnpd.

## D2. NAT-PMP y UPnP-IGD

**NAT-PMP es IPv4-only por diseño** — mapea NAT44 y devuelve la IPv4 externa; no puede expresar pinholes IPv6 [48]. Descartado para nuestro escenario salvo como fallback IPv4 futuro.

**UPnP IGDv2 `WANIPv6FirewallControl`** es el único mecanismo UPnP de pinholes IPv6 (AddPinhole/DeletePinhole/GetFirewallStatus; spec 2010) [49]. Implementaciones reales: miniupnpd (desde ~2012), linuxigd de Orange, Fritz!Box (AddPinhole vía TR-064/UPnP verificado) [49][50]. Quirks: miniupnpd exige puerto interno ≥1024 sin autenticación y tampoco acepta direcciones temporales ni NAT66 [51]. Freebox implementa IGDv2 pero **deniega pinholes IPv6** (`Inbound Pinhole Allowed: 0`) — su firewall es todo-o-nada, así que los usuarios lo desactivan entero [52]. Medición JANOG49 (2021): cero routers domésticos japoneses con UPnP IPv6 [53].

**Postura de seguridad:** la razón de los defaults. Rapid7 (2013): ~80M de dispositivos UPnP expuestos a WAN; Akamai UPnProxy (2018): 4.8M expuestos, 65k+ comprometidos con inyecciones NAT [54]. Por eso Comcast, Vodafone, ASUSWRT reciente y MikroTik lo traen desactivado y Ubiquiti/ASUS recomiendan mantenerlo off [44][41]. Consecuencia: donde UPnP existe suele estar apagado; donde está encendido es un riesgo que nuestro diseño no debe exigir.

## D3. Realidad del filtrado entrante IPv6

**El modelo de referencia es deny-inbound + retorno stateful** (RFC 6092, Informational — recomendación, no mandato): el tráfico no se reenvía al interior salvo que lo solicite un endpoint interior (retorno de flujo saliente o excepción configurada) [55]. UDP: el estado 5-tupla no debe expirar antes de 2 min ocioso (5 min por defecto) — «salir abre el retorno» y «keepalive cada <2 min» son comportamiento especificado [55]. Defaults de vendors confirman el modelo: OpenWrt (wan6 input REJECT), AVM (SPI, todo entrante cerrado salvo respuestas directas), Xfinity (nivel mínimo bloquea no-solicitado) [56][57][58].

**Pero una minoría sustancial de IPv6 residencial NO filtra nada** — el contrapeso clave: 2/10 gateways probados sin filtrado (ACM CSUR 2023) [59]; 5/25 routers funcionales sin firewall IPv6, con CVEs asignados (AsiaCCS 2024) [60]; **25% de las redes IPv6 exploradas sin filtrado alguno, frente a 5.6% en IPv4** (CCS 2026) [61]; 14.9M de hosts *internos* residenciales respondiendo desde 4.412 ASes en el mayor escaneo IPv6 publicado (arXiv, sep-2025) [62]. En esos segmentos no hace falta mecanismo alguno — el entrante simplemente funciona.

**Los ISPs de fijo generalmente NO filtran en la red — filtra el CPE** [63]. **En móvil/celular la situación se invierte: el filtrado es comúnmente *en la red del carrier***, por encima de cualquier CPE: T-Mobile US (todo el entrante IPv6 filtrado, verificado por usuarios con router propio sin firewall), EE UK (respuesta oficial: bloqueo «en el core network»), operadores móviles alemanes (entrante bloqueado en casi todos los tarifas privadas, salvo APN empresarial de Telekom; reporte de nov-2026), y reconocimiento genérico en NANOG («muchos operadores no permiten tráfico iniciado desde internet hacia sus abonados») [64][65][66][67]. Enterprise/Wi-Fi pública: filtrado opt-in institucional existe (Northwestern, LMU/eduroam) [68].

## D4a. Traversal coordinado sin infraestructura

**Funciona en firewalls stateful tipo RFC 6092 — probado, no teorizado.** El envío UDP saliente crea un registro 5-tupla; el retorno desde ese endpoint queda admitido. Como IPv6 no traduce puertos, un firewall stateful equivale a un NAT port-restricted-cone *sin* el problema de adivinar el puerto — más fácil que el traversal de NAT IPv4 [55][69]. Evidencia: EasyTier issue #778 (dos peers tras firewalls IPv6 con entrante prohibido: envíos mutuos 7777→7777 establecieron sesión) [70]; PCAPs de apps VoIP/vídeo P2P sobre IPv6 mostrando direcciones directas de peer tras firewalls stateful [69]; Tailscale trata endpoints IPv6 como «directo en casi todas las situaciones» en producción [71]. Base específica: RFC 5128 (hole punching; ambos extremos deben enviar) [72].

**Matiz de timing:** «simultáneo» no exige simultaneidad estricta — cada lado solo necesita haber enviado una vez para que los paquetes del otro casen con el estado; los primeros se pierden, los reintentos entran. Lo que sí falla es la iniciación *unidireccional* hacia un peer que filtra [72].

**Mediciones:** la mayor campaña pública (libp2p/ProbeLab): ~70% de éxito agregado sobre NATs IPv4; en IPv6 midió *peor* que en IPv4 — causa no resuelta (artefacto de descubrimiento de direcciones, no necesariamente comportamiento de firewall; issue abierta) [27][29]. Histórico BitTorrent IPv4: >80% entre pares punzables [73].

**Dónde NO funciona:** filtrado stateless/ACL en el carrier que bloquea todo el entrante al prefijo del abonado independientemente del estado de flujo (el caso móvil de D3 — ahí ningún envío saliente crea retorno útil para el escucha); políticas CPE paranoides que bloquean categorías P2P enteras (Xfinity «Medium/High»); y trivialmente, iniciación solo-desde-un-lado [63][64][74].

**El quid del rendezvous:** el mecanismo de traversal no necesita servidor — el endpoint externo de cada peer es su propia GUA:puerto (no hay nada que descubrir, no hace falta STUN). Lo irreductible es **conocer la dirección:puerto del otro** — sin canal de señalización compartido, dos peers nunca pueden nombrarse. Con *cualquier* intercambio fuera de banda (el canal de la propia app entre contactos, QR, pegado manual), el UDP directo funciona donde ambos firewalls son stateful-por-saliente — la mayoría de IPv6 residencial según la evidencia — y donde el carrier no descarta el tráfico móvil-terminado [55][71].

## Fuentes

| # | Fuente (publisher, fecha, acceso 2026-11-12) |
|---|---|

| [1] | RFC 8445 — IETF, 2018 — https://www.rfc-editor.org/rfc/rfc8445.html |
| [2] | W3C webrtc-pc issue #1949 / StackOverflow 55563736 — 2018/2019 — https://github.com/w3c/webrtc-pc/issues/1949 |
| [3] | RFC 8863 (ICE PAC) — IETF, 2021 — https://www.rfc-editor.org/rfc/rfc8863.html |
| [4] | "How NAT traversal works: firewalls" — APNIC/Tailscale blog, 2022-04 — https://blog.apnic.net/2022/04/12/how-nat-traversal-works-figuring-out-firewalls/ |
| [5] | ipSpace "Response: P2P apps over IPv6" — 2025-04 (PCAP-verificado) — https://blog.ipspace.net/2025/04/response-peer-to-peer-apps-ipv6/ |
| [6] | jami-daemon commit 2117847 "skip STUN for IPv6 transports" — 2026-05-16 — https://github.com/savoirfairelinux/jami-daemon/commit/2117847382929b5d42e00e72e864f658a7f5a726 |
| [7] | Jami forum: uso serverless sobre Yggdrasil — 2025-11-19 — forum.jami.net |
| [8] | Chromium PSA / release notes M73 mDNS candidates — Google, 2019 — https://groups.google.com/g/discuss-webrtc/c/6stQXi72BEU |
| [9] | "Active ICE-TCP: punch firewalls directly" — Mozilla Hacks, 2017 — https://blog.mozilla.org/webrtc/active-ice-tcp-punch-firewalls-directly/ |
| [10] | pion/ice issue #562 (TCP activo añadido 2023) — https://github.com/pion/ice/issues/562 |
| [11] | libtorrent manual-ref / features-ref — https://libtorrent.org/features-ref.html |
| [12] | libtorrent natpmp.cpp + PR #3076 (PCP-first) — https://github.com/arvidn/libtorrent/pull/3076 |
| [13] | qBittorrent issue #7715 (default-on UPnP/NAT-PMP) — https://github.com/qbittorrent/qBittorrent/issues/7715 |
| [14] | libtorrent issue #4111 (WANIPv6FirewallControl no implementado) — https://github.com/arvidn/libtorrent/issues/4111 |
| [15] | BEP 55 ut_holepunch — BitTorrent.org — https://www.bittorrent.org/beps/bep_0055.html |
| [16] | Bitcoin Core release notes v29.0 — 2025-04 — https://bitcoincore.org/en/releases/29.0/ |
| [17] | bitcoin PR #30043 (PCP + IPv6 pinholing) — https://github.com/bitcoin/bitcoin/pull/30043 |
| [18] | bitcoin issue #31663 (debate default-on) — https://github.com/bitcoin/bitcoin/issues/31663 |
| [19] | bitnodes snapshot .onion — 2026-03 — https://bitnodes.io/nodes/?q=.onion |
| [20] | bnoc.xyz crawl de direcciones inalcanzables — 2026-07 — https://bnoc.xyz/t/unreachable-addresses-in-bitcoin-getaddr-responses/155 |
| [21] | Jami docs: calls / peer-to-peer connections — https://docs.jami.net/developer/jami-concepts/calls.html |
| [22] | Jami forum dev-update — 2026 — https://forum.jami.net/t/dev-update-6-major-bug-fixes/2626 |
| [23] | Bramble Rendezvous Protocol spec — briar-spec — https://code.briarproject.org/briar/briar-spec/-/blame/master/protocols/BRP.md |
| [24] | Briar "How it works" / FAQ — https://www.briarproject.org/how-it-works/ |
| [25] | toxcore README + docs/TCP_Network.txt + Symmetric-NAT-Transversal.md — https://github.com/irungentoo/toxcore/ |
| [26] | hackage toxcore DHT docs — https://hackage.haskell.org/package/toxcore/docs/src/Network.Tox.DHT.html |
| [27] | "Measuring DCUtR hole punching" — IMC'26, arXiv 2510.27500 — 2025-10 — https://arxiv.org/abs/2510.27500 |
| [28] | ProbeLab DCUtR final report + NAT hole punching measurements — https://github.com/probe-lab/network-measurements/blob/main/results/rfm15-nat-hole-punching.md |
| [29] | libp2p go-libp2p issue #2068 (IPv6 punching peor) — 2023-02, abierta — https://github.com/libp2p/go-libp2p/issues/2068 |
| [30] | miniupnpd.conf (`enable_pcp_pmp=no` por defecto) — https://github.com/miniupnp/miniupnp/blob/master/miniupnpd/miniupnpd.conf |
| [31] | OpenWrt-devel: rechazo de UPnP por defecto — 2022-01 — https://lists.openwrt.org/pipermail/openwrt-devel/2022-January/037753.html |
| [32] | RFC 6887 (PCP) — IETF, 2013 — https://www.rfc-editor.org/rfc/rfc6887.html |
| [33] | Radicle commit (sondeo hardware real: Fritz!Box PCP v6, rechazo de direcciones temporales) — ~2025 — https://radicle.thalheim.io/ |
| [34] | AVM KB: port sharing automático vía UPnP IGD o PCP — docs actuales — https://uk.fritz.com/service/knowledge-base/dok/FRITZ-Box-4040/894_Setting-up-automatic-port-sharing-in-the-FRITZ-Box/ |
| [35] | tailscale issue #13145 (Fritz!Box devuelve NOT_AUTHORIZED con toggle apagado) — 2024-08 — https://github.com/tailscale/tailscale/issues/13145 |
| [36] | miniupnpd pcpserver.c (MAP IPv6-firewall; PEER stubeado) — master — https://github.com/miniupnp/miniupnp/blob/master/miniupnpd/pcpserver.c |
| [37] | miniupnpd Changelog.txt (fixes pinholes IPv6 hasta 2026-11-02) — https://github.com/miniupnp/miniupnp/blob/master/miniupnpd/Changelog.txt |
| [38] | OpenWrt packages: miniupnpd Makefile --igd2 --ipv6 + PR #28765 — 2025 — https://github.com/openwrt/packages/blob/master/net/miniupnpd/Makefile |
| [39] | Netgate pfSense UPnP docs + tailscale PR #2577 + opnsense#8070 — 2021-2024 — https://docs.netgate.com/pfsense/en/latest/services/upnp.html |
| [40] | MikroTik NAT-PMP docs + forum ipv6-pin-holing — 2024 — https://help.mikrotik.com/docs/spaces/ROS/pages/220233819/NAT-PMP |
| [41] | Ubiquiti community + help (UniFi UPnP roto; recomiendan desactivado) — 2022-2025 — https://help.ui.com/hc/en-us/articles/12648697125783 |
| [42] | TP-Link community (sin WANIPv6FirewallControl/PCP) — https://community.tp-link.com/en/home/forum/topic/168302 |
| [43] | Netgear community (firewall IPv6 no configurable; Pass Through) — ~2023 — https://community.netgear.com/t5/Nighthawk-with-WiFi-6-AX-and/IPv6-Firewall-and-router-support/td-p/2275026 |
| [44] | Vodafone UPnP + Xfinity forum + Edovia (UPnP off/deshabilitado en CPEs de ISP) — ~2025 — https://www.vodafone.de/featured/article/upnp-aktivieren-vodafone-179510 |
| [45] | NetBird PR #5219 "Add PCP support" — mergeado 2026-04 (verificado directo) — https://github.com/netbirdio/netbird/pull/5219 |
| [46] | Tailscale portmapper (sonda PCP→NAT-PMP→UPnP) — https://deepwiki.com/SagerNet/tailscale/7.2-port-mapping |
| [47] | Transmission issue #993 (sin pinholes IPv6) — abierta desde 2019 — https://github.com/transmission/transmission/issues/993 |
| [48] | RFC 6886 (NAT-PMP) — IETF, 2013 — https://www.rfc-editor.org/info/rfc6886/ |
| [49] | UPnP WANIPv6FirewallControl v1 spec — UPnP Forum, 2010-12 — https://upnp.org/specs/gw/UPnP-gw-WANIPv6FirewallControl-v1-Service.pdf |
| [50] | Orange igd2-for-linux + ip-phone-forum (Fritz!Box AddPinhole vía TR-064) — https://github.com/Orange-OpenSource/igd2-for-linux |
| [51] | miniupnp issues #692/#731/#786 (quirks: puerto ≥1024, temp addrs, sin NAT66) — 2024-2025 — https://github.com/miniupnp/miniupnp/issues/731 |
| [52] | Freebox FS#37515 (Inbound Pinhole Allowed: 0) — ~2021 — https://dev.freebox.fr/bugs/task/37515 |
| [53] | JANOG49 LT8 (cero routers domésticos JP con UPnP IPv6) — 2021-12 — https://www.janog.gr.jp/meeting/janog49/lt8/ |
| [54] | Rapid7 UPnP 2013 + Akamai UPnProxy 2018 — estudios medidos — https://www.rapid7.com/blog/post/2013/01/29/security-flaws-in-universal-plug-and-play-unplug-dont-play/ |
| [55] | RFC 6092 (filtrado CPE IPv6 residencial) — IETF, 2011-01 — https://www.rfc-editor.org/rfc/rfc6092.html |
| [56] | OpenWrt firewall.config (wan6 input REJECT por defecto) — https://github.com/openwrt/openwrt/blob/master/package/network/config/firewall/files/firewall.config |
| [57] | AVM KB: firewall de fábrica, entrante cerrado salvo respuestas — https://uk.fritz.com/service/knowledge-base/dok/FRITZ-Box-7560/57_Security-functions-firewall-of-the-FRITZ-Box/ |
| [58] | Xfinity gateway niveles de seguridad (secundario) — https://routersnetwork.net/blog/setup-firewall-xfinity-gateway/ |
| [59] | "IPv6 in the Home" survey, ACM Computing Surveys — Olson/Wampler/Keller, 2023 — https://eric-keller.github.io/papers/2023/ipv6-NAT-csur2023.pdf |
| [60] | AsiaCCS 2024 "routers default settings" + CVEs — https://xavier2dc.fr/papers/routers-default-settings-asiaccs2024.pdf |
| [61] | Pearce et al., "IPv6 end hosts" CCS 2026 (25% redes IPv6 sin filtrado) — https://pearce.prof/papers/ipv6_endhosts_ccs_2026.pdf |
| [62] | "Where Have All the Firewalls Gone?" — arXiv 2509.04792, 2025-09 — https://arxiv.org/html/2509.04792 |
| [63] | Comcast Business forums (no carrier-filter; filtra el CPE) — ~2019-2021 — https://forums.businesshelp.comcast.com/ |
| [64] | T-Mobile community + GL.iNet forum (IPv6 entrante filtrado en red carrier) — 2021-2024 — https://forum.gl-inet.com/t/wireguard-and-ddns-server-on-t-mobile-5g-home-with-gl-x3000/41403 |
| [65] | ISPreview thread citando respuesta oficial EE UK — 2020-12 — https://www.ispreview.co.uk/talk/threads/ipv6-wan-settings-on-pfsense-and-mikrotik-passthrough-configuration.38596/ |
| [66] | O2 community DE (móviles filtran entrante salvo APN empresarial) — 2026-11-04 — https://hilfe.o2online.de/mobilfunkdienste-telefonie-internet-messaging-mehr-22/ |
| [67] | NANOG list (operadores no permiten iniciado-desde-internet) — 2025-02 — https://seclists.org/nanog/2025/Feb/22 |
| [68] | Northwestern "Deny Unsolicited Inbound" + LMU eduroam — https://services.northwestern.edu/TDClient/30/Portal/Requests/ServiceDet?ID=198 |
| [69] | ipSpace "Response: P2P IPv6" (PCAPs, 5-tuple equivale port-restricted-cone) — 2025-04 — https://blog.ipspace.net/2025/04/response-peer-to-peer-apps-ipv6/ |
| [70] | EasyTier issue #778 (test punching IPv6 stateful verificado) — ~2025 — https://github.com/EasyTier/EasyTier/issues/778 |
| [71] | Tailscale device-connectivity docs + NAT traversal blog — https://tailscale.com/docs/reference/device-connectivity |
| [72] | RFC 5128 (hole punching, simultaneous open) — IETF, 2008-03 — https://www.rfc-editor.org/info/rfc5128/ |
| [73] | Halkes & Pouwelse, BitTorrent hole-punch IPv4 (>80%) — IFIP Networking 2011 — https://opendl.ifip-tc6.org/db/conf/networking/networking2011-2/HalkesP11.pdf |
| [74] | StackOverflow 43686011 (fallo punching en celular IPv6) — ~2017 — https://stackoverflow.com/questions/43686011/ |