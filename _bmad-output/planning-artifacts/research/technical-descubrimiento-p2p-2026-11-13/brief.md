---
type: technical
mode: run
created: 2026-11-13
---

# Brief: descubrimiento-p2p

## Decisión que sirve la investigación

Elegir el protocolo concreto de descubrimiento/rendezvous (§16, cuestión 6 del documento rector) para una mensajería P2P distribuida.

## Requisitos aprobados (hard gates)

- Depósito de registros solo en contactos directos del colaborador (D-12.4).
- Registros cifrados por destinatario autorizado; IDs opacos con clave (blinded) y rotatorios.
- Consultas a través de onion services del colaborador.
- Debe proveer el canal de intercambio de endpoints para punching coordinado (§9).
- Debe funcionar con población mayoritaria en modalidad A (colaboración solo con contactos directos; B es opt-in).
- Sin servidores, sin relays, sin directorio global; los colaboradores no transportan mensajes ni leen lo que custodian.
- Sin root; toda dependencia de terceros open source.

## Candidatos / dimensiones del plan aprobado

1. Briar BRP (Bramble Rendezvous Protocol).
2. Tor Onion Services v3 + blinded keys como sustrato de rendezvous.
3. libp2p rendezvous.
4. Familia Kademlia/DHT: Tox, OpenDHT/Jami, kad-DHT, HyperDHT, Veilid.
5. Patrones de rendezvous bilateral sin servidores (mailbox-on-onion, tokens opacos, etc.).
6. Comparativa transversal: metadatos revelados, aptitud con modalidad A, aptitud como canal de punching, implementabilidad en Rust, licencia.

## Ejecución

Preset standard: 2 rondas, 3 subagentes por ronda, validación normal, red team off.
