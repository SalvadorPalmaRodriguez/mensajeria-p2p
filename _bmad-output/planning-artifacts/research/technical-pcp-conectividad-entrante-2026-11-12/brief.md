# Deep-research prompt — PCP y conectividad entrante IPv6 (tool-neutral)

Copia y pega el texto entre las líneas ─── en tu herramienta de deep research.

---

Investiga a fondo la viabilidad real de recibir conexiones entrantes P2P en redes domésticas IPv6 sin servidores propios ni de terceros, para decidir el mecanismo de conectividad de una aplicación de mensajería P2P. Contexto que condiciona QUÉ preguntar (no qué concluir): transporte estrictamente directo entre dos dispositivos, prohibidos TURN/relays/STUN externos/store-and-forward, primera implementación IPv6-only en Linux de escritorio, usuarios domésticos con routers CPE de operador o retail.

Responde estas cinco líneas de investigación, cada una con hallazgos citados:

1. **Soporte real de PCP (RFC 6887)** en routers domésticos: ¿qué CPEs de operador y qué routers retail (OpenWrt, Fritz!Box, Ubiquiti, ASUS, Netgear, etc.) lo implementan HOY? Distingue soporte PCP para mapeos NAT44 del caso que nos importa: control del firewall IPv6 para abrir pinholes entrantes (RFC 6887 §6 / comportamiento "MAP" sobre IPv6). Documenta problemas de interoperabilidad conocidos, implementaciones rotas o parciales, y si algún ISP lo deshabilita deliberadamente. Incluye datos de despliegue medidos si existen (sondeos de Internet, estudios).

2. **NAT-PMP y UPnP-IGD como alternativas**: soporte real en CPE domésticos, postura de seguridad documentada (vulnerabilidades UPnP, exposición WAN, deshabilitado por defecto en qué vendors), y la cuestión crítica: ¿pueden estos protocolos abrir pinholes de firewall IPv6 o solo sirven para NAT44? (UPnP IGDv2/WANIPv6FirewallControl: implementaciones reales, no teoría de especificación.)

3. **Realidad del filtrado entrante IPv6**: comportamiento por defecto de los CPE conforme a RFC 6092 y práctica real (¿deny-inbound por defecto? ¿excepciones?), políticas de operadores domésticos sobre tráfico entrante IPv6, y el caso de redes móviles/celulares (¿los carriers bloquean UDP/IPv6 entrante en sus CGN/firewalls?).

4. **Atravesado coordinado sin infraestructura**: viabilidad documentada de hole punching sobre firewalls IPv6 stateful (¿el firewall stateful permite retorno de flujos iniciados desde dentro? ¿sirve simultaneous open?), ICE sin STUN (host candidates + peer-reflexive solamente), y qué hacen realmente los proyectos P2P de referencia ante el problema de entrantes: Briar (Tor/Onion), Jami (ICE + ¿qué para IPv6 filtrado?), Tox, qBittorrent/libtorrent (UPnP/NAT-PMP/PCP support real), Bitcoin Core (¿qué traversal usa?).

5. **Realidad de despliegue en apps existentes**: qué mecanismos de aceptación de entrantes usan aplicaciones P2P en producción hoy (orden de preferencia implementado), documentación sobre tasas de éxito o de configuración manual necesaria, y lecciones de retrospectivas/post-mortems — no anuncios de lanzamiento.

Requisitos de evidencia (no negociables):

- Toda afirmación con URL de fuente y fecha de publicación; fecha de acceso si es relevante.
- Prioriza fuentes primarias: RFC, documentación oficial de vendors/proyectos, código fuente, issues de trackers, papers medidos. Los agregadores y motores de respuesta no cuentan como fuente: persigue y cita sus citas.
- Las afirmaciones sobre soporte de un vendor/protocolo y las que afirman que algo "no funciona" requieren dos fuentes independientes.
- Reporta evidencia contraria explícitamente; si la documentación no permite concluir algo, dilo — un vacío de datos es un hallazgo.
- Frescura: versiones/compatibilidad ≤1 mes, señales de ecosistema ≤6 meses, paisaje general ≤12 meses cuando sea posible; marca lo más antiguo como histórico.

Formato de salida: un informe organizado por las cinco líneas (hallazgos en prosa con citas inline), seguido de una tabla de fuentes [n] con URL, publicador y fecha, y una sección final de "gaps y contradicciones".

---
