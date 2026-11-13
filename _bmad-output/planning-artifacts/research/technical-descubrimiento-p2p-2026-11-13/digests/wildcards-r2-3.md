## Digest: wildcards + loose ends (r2)
_Subagent: fc5521f8 · Round 2 · accessed 2026-11-13_

### Claims — Veilid

- Veilid is an actively maintained Rust framework (crate `veilid-core`, also Python/Flutter/WASM bindings) for E2EE app-to-app communication; license MPL-2.0. Current version v0.5.8, released ~1 week before access, with commits/MRs within days. {source: https://gitlab.com/veilid/veilid + https://pypi.org/project/veilid/ + https://veilid.com/download/ | publisher: Veilid team | pub_date: repo activity Nov 2026 | accessed: 2026-11-13 | confidence: high | class: health}
- Veilid's data layer is a **global DHT**: record keys placed on node identities XOR-nearest to the key, replicated to nearest ~4 neighbors. Nodes may opt out of DHT storage, but a published record still lands on arbitrary XOR-close nodes — no mode where records live only at the owner's contacts. {source: https://veilid.gitlab.io/developer-book/concepts/dht.html + https://veilid.com/documentation/rpc/ | accessed: 2026-11-13 | confidence: high | class: spec}
- Veilid provides private routing (safety routes + private routes) and `app_message`/`app_call` up to 32,768 bytes, plus local encrypted TableStore. Route blobs are themselves exchanged via the DHT, so even the direct-messaging path depends on the global DHT. {source: docs.rs/veilid-core + veilid.com/documentation/private-routing/ | confidence: high | class: spec}
- **Verdict: NON-FIT** for the approved custody model (D-12.4). Record placement is global-DHT, not contact-scoped; would violate "records live only at contacts" and Modalidad A. MPL-2.0 compatible, but no code path offers per-contact custody. {synthesis | confidence: high | class: assessment}

### Claims — TokTok new Rust impl

- TokTok officially states "A new implementation in Rust is in progress. It evolves together with the [Haskell] model and is tested against the specification"; roadmap lists "Write a Rust implementation of the Tox protocol" under not-yet-complete. {source: https://toktok.ltd/ + roadmap.html | confidence: high | class: health}
- Only new public Rust repo found: `TokTok/rs-toxcore-c` (created 2026-01-16, active): **FFI bindings over c-toxcore** plus Merkle-DAG history-sync suite, TUI client, bots — **not** a standalone Rust toxcore. {source: github.com/TokTok/rs-toxcore-c | confidence: high | class: health}
- No dedicated public repo for the new Rust toxcore located; possibly inside Bazel toktok-stack monorepo. Not usable today; all Tox code GPL-3.0 — copyleft friction with source-visible proprietary license. {confidence: medium | class: health}

### Claims — libp2p-rendezvous payload

- **Strictly a libp2p signed peer record — no arbitrary opaque bytes.** Protobuf `Register` = `{ns, signed_peer_record: bytes, ttl}`; rust-libp2p produces those bytes from `PeerRecord` (peer_id, seq, multiaddrs) via `into_signed_envelope()`. The `Registration` struct is `{namespace, record: PeerRecord, ttl}`; server validates the record (NotAuthorized on mismatch). An application cannot register arbitrary blobs through the standard behaviour — only advertise its own signed addresses under a namespace. {source: docs.rs/crate/libp2p-rendezvous source rendezvous.pb.rs + codec.rs + Registration | accessed: 2026-11-13 | confidence: high | class: spec}

### Claims — custody-at-contacts patterns

- **RetroShare "Discovery" service is the closest deployed precedent.** In "Private: Discovery only" mode (no DHT), endpoint updates propagate through mutual friends: "if you change IP... you only need one mutual friend to be connected for all other mutual friends to be able to find you." Caveats: shares friend certs + IPs with friends-of-friends by default (transitive, cleartext); it is *forwarding between mutual contacts*, not encrypted per-recipient custody. Proves mutual-contact endpoint propagation works at scale; differs from D-12.4 in confidentiality/authorization granularity. {source: retroshare.readthedocs.io user-guide/settings + medium community writeup | confidence: high | class: pattern}
- **Freenet darknet (F2F) mode**: DHT over friend-to-friend links only; "users can publish their encrypted contact details under updatable keys for their friends to retrieve anonymously." Deployed precedent for encrypted contact records over a friend overlay — placement key-nearest within F2F topology, not strictly owner's-direct-contacts. {source: Rogers & Bhatti P2P'10 paper | confidence: medium (2010; current impl unverified) | class: pattern}
- **GNUnet friend-only HELLOs**: PEERINFO distinguishes public vs `friend_only` HELLOs; friend-only address records never distributed to non-friends. Friend_only flag documented in v0.20.x. {source: docs.gnunet.org + handbook commit | confidence: medium | class: pattern}
- **Swirlwave** (research middleware): contact list populated out-of-band (NFC/face-to-face); entries contain peer ID, **onion address**, services, public key — contact-scoped reachability over onion services, conceptually close to the approved model. {source: SCITEPRESS 2018 | confidence: medium | class: pattern}
- **SimpleX confirmed relay-based** — messages through SMP server queues; violates no-relays. Design reference only: pairwise per-queue ephemeral IDs, no global identifiers, contacts stored solely on devices. {source: simplex-chat docs SERVER.md + simplex-messaging.md | confidence: high | class: spec}
- Briar Mailbox = store-and-forward *message* custody at owner's own second device (excluded for messages per continuity doc). Historical F2F systems Galet/Cryptic6/Alliance did friends-of-friends address exchange without DHT. {confidence: medium | class: pattern}
- **Confirmed absence**: no deployed system found matching D-12.4 exactly (owner-contact-scoped custody + per-recipient encryption + rotating blinded keys). Closest: RetroShare (mutual-friend propagation, cleartext/transitive) and Freenet darknet (encrypted records on friend overlay, key-routed not contact-scoped). The approved design appears novel as a composition. {synthesis | confidence: medium | class: assessment}

### Claims — missed wildcards

- Wildcard scan surfaced: GNUnet F2F/friend-only HELLOs, Freenet darknet USK records, Swirlwave onion contact lists, historical Galet/Cryptic6/Octopod/CSpace/Alliance/OneSwarm. No additional actively maintained, open-source, serverless, contact-scoped discovery system beyond those and Round-1 coverage found. {confidence: medium | class: HECHO}

### Looked for but could not find
- A dedicated public repository for TokTok's announced new Rust toxcore.
- Any deployed system implementing D-12.4's exact pattern.
- A Veilid mode scoping DHT record custody to contacts — does not exist.

**Actionable note:** RetroShare's Discovery is the strongest existence proof that mutual-contact endpoint propagation works in production, but its default transitive/cleartext semantics are precisely what D-12.3/D-12.4 reject — cite as feasibility precedent, not model to copy. libp2p-rendezvous cannot carry opaque encrypted records (signed peer record only), closing that loose end against naive reuse.
