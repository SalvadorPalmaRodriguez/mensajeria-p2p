## Digest: Tor OSv3 + blinded keys rendezvous (r1)
_Subagent: 8f56630e · Round 1 · accessed 2026-11-13_

### Claims

**Mechanics (Q1)**

- OSv3 publishes descriptors to HSDirs indexed by a **blinded Ed25519 key that rotates every time period (~daily)**; anyone knowing the service's identity pubkey can derive the blinded keys, and it is "impossible to derive the blinded signing key lacking that knowledge" — so HSDirs cannot enumerate or link descriptors to an onion address. {https://torproject.gitlab.io/torspec/rend-spec-v3.html §1.1, §2.1 [SUBCRED], publisher: Tor Project spec, spec version: rend-spec-v3 current, accessed 2026-11-13, confidence: high, class: spec}
- The descriptor body is encrypted under the blinded signing key; decrypting requires the *unblinded* pubkey, so **HSDirs cannot read introduction points** — they learn only that "some blinded key" stores a blob. {https://spec.torproject.org/rend-spec/protocol-overview.html, accessed 2026-11-13, confidence: high, class: spec}
- Descriptor uploads are staggered per-period by a time derived from the blinded key ("thundering herd" mitigation); re-uploads are randomized in a 1–2h window and the HSDir-side descriptor lifetime is ~3h. {https://archive.torproject.org/.../tor-bugs/2017-August/131623.html #20524, accessed 2026-11-13, confidence: med, class: spec — numbers from a 2017 dev ticket; verify against current C Tor constants}
- The rendezvous point sees only a **random 20-byte single-use cookie** — "does not know for what service" and cannot identify either party; intro points see only a per-intro-point ephemeral service key and can count/timing-track requests but cannot tie them to the onion address. {https://spec.torproject.org/rend-spec/rendezvous-protocol.html + David Goulet, tor-talk 2016-12, accessed 2026-11-13, confidence: high, class: spec}

**Restricted discovery (Q2)**

- Renamed from "client authorization": the descriptor's inner layer is superencrypted **per authorized client's x25519 key** (fresh `N_hs_desc_enc` cookie + ephemeral x25519 per descriptor); without valid client keys the intro points are unreadable. {https://spec.torproject.org/rend-spec/hsdesc-encrypt.html, accessed 2026-11-13, confidence: high, class: spec}
- Implemented in C Tor since 0.3.5.1-alpha via `authorized_clients/*.auth` / `ClientOnionAuthDir`; **stabilized in Arti 1.7.0 (2025-11-03)** behind the `restricted-discovery` cargo feature and `restricted_mode` config. {https://blog.torproject.org/arti_1_7_0_released/ + https://onionservices.torproject.org/dev/implementations/, accessed 2026-11-13, confidence: high, class: version}
- Recommended cap is **`MAX_RESTRICTED_DISCOVERY_CLIENTS = 160`** (descriptor size limit `HSV3MaxDescriptorSize`; inner layer is encrypted once per client). Fits "direct contacts only" but bounds contact-list size. {https://docs.rs/tor-hsservice/latest/.../MAX_RESTRICTED_DISCOVERY_CLIENTS.html, accessed 2026-11-13, confidence: high, class: version}
- ⚠️ Arti docs explicitly call restricted discovery **"a DoS resistance mechanism, *not* a substitute for conventional access control"** — a de-authorized client may still reach the service via not-yet-expired descriptors until rotation. Revoking a contact is not immediate. {https://docs.rs/tor-hsservice/.../restricted_discovery/index.html, accessed 2026-11-13, confidence: high, class: spec}
- Spec's second auth layer — per-client ed25519 signatures inside INTRODUCE1 ("introduction-point auth") — is **draft-only: "unlikely" in C Tor, not implemented in Arti**. Only descriptor-level auth ships. {https://onionservices.torproject.org/dev/implementations/ + rend-spec-v3 [INTRO-AUTH], accessed 2026-11-13, confidence: high, class: spec}

**Mailbox/registry suitability (Q3, Q6)**

- No artificial startup delay remains: descriptor is uploaded "as soon as" intro circuits are established (post-#20082); **Briar reports a newly published HS is "usually reachable in the first connection attempt"** (they gate on one descriptor upload; client-side timeout ~2 min). {https://forum.torproject.org/t/tor-dev-.../6664, accessed 2026-11-13, confidence: med, class: pattern — practitioner report, not controlled measurement}
- **Counter-evidence:** Quiet reports newly-online services unreachable for "15 minutes or more" in some cases. Stale descriptors/dead intro points after a peer goes offline are the failure mode — directly relevant since each device's mailbox goes offline with the device. {same forum thread, confidence: med, class: pattern}
- Onion services accept **TCP streams only** — no UDP over Tor — but work fine as a low-bandwidth signaling channel for exchanging hole-punch endpoints; expect seconds-scale handshake latency (client→RP + client→IP + service→RP circuit builds), not sub-second. On the plus side it traverses NAT/CG-NAT with zero port forwarding. {inference from rend-spec rendezvous-protocol, confidence: med — no hard latency percentiles found this round, class: pattern}

**Metadata (Q4)**

- The hosting service **cannot identify which authorized contact connected**: rendezvous hides client IP, and with only descriptor-level auth shipped, all authorized clients are indistinguishable at intro/rendezvous time. {inference: implementations table shows intro-auth unimplemented + spec INTRO-AUTH, confidence: med, class: spec}
- Guard relays of the service see descriptor-upload/intro-circuit traffic → **guard-discovery (de-anonymization) risk; Arti enables Vanguards-lite by default** for all onion-service circuits since 1.2.2 (full mode recommended for uptime >1 month). {https://blog.torproject.org/announcing-vanguards-for-arti/, accessed 2026-11-13, confidence: high, class: version}
- An observer who knows the onion address can monitor HSDir fetches/correlate activity; hosting an always-on per-device onion also leaks "device is online" timing to its contacts (inherent to the design). {tor-talk 2016 thread, confidence: med, class: other}

**Project health (Q5)**

- C Tor 0.4.8 **EOL 2026-06-01 and actively blocked from the network ~2026-09-01** (not merely unsupported — directory-protocol fields removed); 0.4.9 is the supported series (first stable 2026-02-12). {https://blog.torproject.org/sunsetting-tor-048/ + https://newsletter.torproject.org/archive/2026-08-05-..., accessed 2026-11-13, confidence: high, class: version — do NOT embed 0.4.8}
- Arti is at **2.8.0 (2026-11-03)**, shipped on a ~monthly cadence (2.5.0 Jun, 2.5.1 Aug 4, 2.6.0 Sep 1, 2.7.0 Oct 1, 2.8.0 Nov 3 — all 2026); congestion control + Counter-Galois-Onion crypto became always-on in 2.6.0; active work is now on relay/dirauth side — onion-service features are shipping stable, not experimental-only. {https://blog.torproject.org/arti_2_6_0_released/ + author page + crates.io/arti 2.8.0, accessed 2026-11-13, confidence: high, class: health}
- **Contradiction on hosting maturity:** the old `doc/OnionService.md` still says "for testing purposes… many serious problems" (written Feb 2024) and crates.io feature docs at 1.7.0 warned onion-service features were "not yet as secure as C-Tor… not for security-sensitive purposes"; but the changelog records "Removed security disclaimer about onion service support" (!3464, ~1.7.0) and the ecosystem matrix lists all server features implemented. Treat as resolved-toward-stable but verify current wording. {git.jordan.im/arti OnionService.md + crates.io/arti/1.7.0 + repo.or.cz CHANGELOG, confidence: med, class: health}
- Licenses OK: **C Tor = BSD-3-Clause; Arti = MIT OR Apache-2.0**. Caveat: `equix`/`hashx` crates (PoW anti-DoS) are **LGPL-3.0-only** — matters only if you want PoW defense. {gitlab.torproject.org/tpo/core/tor LICENSE + crates.io/arti + debian/copyright, confidence: high, class: version}

### Leads worth chasing
- Tor Metrics `conntime` / Onionprobe dashboards for real percentile numbers on descriptor-fetch + rendezvous latency (needed to size hole-punch coordination timeouts).
- Whether `arti` 2.8.0 crates still carry the "not security-sensitive" warning on `onion-service-service`; locate the exact release that dropped the disclaimer.
- Whether a stale-descriptor/intro-point failure causes client-visible hangs vs. automatic refetch (Quiet's 15-min case) — critical for an intermittently-online peer mesh.
- Arti keystore/offline-blinded-key support and RPC-based config for dynamically adding/removing restricted-discovery clients at runtime (descriptor re-publish rate limits apply).

### Looked for but could not find
- Hard percentile measurements of end-to-end onion-service connect latency on the live network (only practitioner anecdotes found).
- Formal spec statement that descriptor-level auth renders authorized clients indistinguishable to the service (inferred from intro-auth being unimplemented; not stated verbatim).
- Current (2.x) Arti documentation explicitly declaring onion-service hosting production-ready vs. test-only — the old doc is stale and I could not confirm what replaced it.
