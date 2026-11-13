## Digest: Tor/Arti operational reality (r2)
_Subagent: f62a376e · Round 2 · accessed 2026-11-13_

### Claims — Arti hosting maturity

1. `doc/OnionService.md` on arti `main` *still* carries the Feb-2024 preamble — "you can run an onion service... for testing purposes", "There are many serious problems right now!", plus warnings about stability/reachability bugs, stack backtraces, and unstable on-disk key formats. The doc itself says it's "temporary" pending integration elsewhere. {source: https://git.jordan.im/arti/file/doc/OnionService.md.html (mirror of `main`; CHANGELOG shows Arti 2.8.0, mirror is current) | publisher: Tor Project (arti repo mirror) | pub_date: text dates from !1994, 2024-02-22; still present in main as of 2026-11 | accessed: 2026-11-13 | confidence: high text persists; medium on whether still authoritative | class: HECHO}

2. Resolution of the R1 contradiction — the removed security disclaimer (changelog !3464) and the persisting scary doc text coexist because `OnionService.md` was never rewritten; meanwhile the *functional* documentation moved to rustdoc (`tor-hsservice` on docs.rs documents restricted discovery, live reload, config API with no "testing only" warning) and release blogs now treat running onion services on arti as routine. The "testing only" text is a stale artifact, not current policy — but nothing retrieved this run formally rescinds it. {source: https://docs.rs/tor-hsservice/latest/tor_hsservice/config/restricted_discovery/index.html ; https://blog.torproject.org/arti_2_8_0_released/ | publisher: Tor Project | pub_date: docs.rs 2.x (2026); blog 2026-11-03 | accessed: 2026-11-13 | confidence: medium-high | class: HECHO + HIPÓTESIS}

3. Restricted discovery (f.k.a. client authorization) is a *stable* feature in arti since 1.7.0 (needs `restricted-discovery` build feature). Arti 1.8.0 added `arti hsc ctor-migrate` to import C Tor client restricted-discovery keys into Arti's keystore. {source: https://blog.torproject.org/arti_1_7_0_released/ ; https://blog.torproject.org/arti_1_8_0_released/ ; https://onionservices.torproject.org/dev/implementations/ | pub_date: 2025-11-03 / 2025-12-02 / page updated 2026-08-19 | accessed: 2026-11-13 | confidence: high | class: HECHO}

### Claims — connect latency & stale descriptors

4. Rendezvous-circuit build time (service-side, C Tor relay self-metrics): of 24,096 builds, 14,275 ≤1s, 24,060 ≤5s, 24,095 ≤10s — i.e., ~99.8% under 5s for the rend-circuit leg alone. Full client connect adds descriptor fetch + intro circuit + intro exchange. {source: https://forum.torproject.org/t/tor-relays-ot-tor-metrics-for-hidden-services/13763 | publisher: Tor forum (operator-reported `tor_hs_rend_circ_build_time` histogram) | pub_date: n/d | accessed: 2026-11-13 | confidence: medium | class: HECHO}

5. Briar measured: a *newly published* onion service is usually reachable on the first connection attempt; Tor's internal timeout for HS client connections is ~2 minutes. A Ricochet-era report: services work "within 30 seconds" of publishing. {source: https://www.mail-archive.com/tor-dev@lists.torproject.org/msg12216.html ; https://moderncrypto.org/mail-archive/messaging/2014/000849.html | confidence: medium | class: HECHO}

6. Quiet's 15+ min failures are about the service side — a peer that *just came online* hasn't finished establishing intro points / uploading descriptors to all HSDirs, so clients get descriptor-not-found. Not a client-side stale-descriptor hang. {source: https://www.mail-archive.com/tor-dev@lists.torproject.org/msg12216.html | confidence: medium-high | class: HECHO}

7. Stale-descriptor behavior is automatic at Tor layer: client tries intro points in the cached descriptor; if an INTRODUCE_ACK fails it MAY try another IP (rend-spec); when all IPs fail it triggers `hs_client_refetch_hsdesc` and refetches. Per-IP failure cache entries live ~2 min. So stale descriptors cause a retry+refetch cycle (tens of seconds), not a silent indefinite hang — but a SOCKS/API caller sees only an eventually-failed or slow connect. {source: https://spec.torproject.org/rend-spec/introduction-protocol.html ; tor-bugs #24610 | confidence: high on mechanism | class: HECHO}

8. Tor processes allow only 32 circuits under construction at once (internal DDoS throttle); Cwtch had to serialize/prioritize contact connect attempts because of it. Relevant if the app fans out to many contact-pair services simultaneously. {source: https://www.openprivacy.ca/discreet-log/34-cwtch-updates/ | pub_date: ~2023 | accessed: 2026-11-13 | confidence: high | class: HECHO}

9. *(timeout sizing, synthesis)*: realistic rendezvous connect: published+online peer → seconds (p50 ~1–5s); allow ≥120s (Tor's internal ceiling). Peer recently online/offline → retry loop spanning minutes (worst reported 15min); don't treat first failure as final. {class: PROPUESTA derived from claims 4–7}

### Claims — runtime client-auth management

10. *(C Tor, service side)*: `ADD_ONION` accepts `ClientAuthV3=<base32 ed25519 pub>` (control-spec, added Tor 0.4.6.1-alpha) — ephemeral services can be created already-restricted. For a *running* service, adding `.auth` files + `SIGNAL RELOAD` is additive-only (ticket #28184); deleting `.auth` files does NOT revoke until restart — man page: "the only real way to fully revoke an onion service v3 client authorization is by restarting the tor process" (#28275). Workaround: `DEL_ONION` + recreate. {source: tor-commits archives + tor.1 man page | confidence: high | class: HECHO}

11. *(C Tor, client side)*: `ONION_CLIENT_AUTH_ADD` registers an x25519 private key for a target onion at runtime (base64); REMOVE/VIEW also exist per control-spec. {confidence: medium-high | class: HECHO}

12. *(Arti)*: restricted discovery supports **live reload**: with `watch_configuration = true`, changes to `static_keys` and to `.auth` files in configured `key_dirs` are auto-detected and take effect without restart; each change can trigger a new descriptor publish, subject to descriptor-publishing rate limits — batch changes. `key_dirs` accepts C Tor-format `authorized_clients` directories. `arti hsc key get` manages client-side discovery keys; arti 2.7.0 added RPC for inspecting/modifying config. {source: docs.rs tor-hsservice restricted_discovery ; arti doc/hsc.md ; blog arti_2_7_0 | pub_date: 2025–2026 | confidence: high | class: HECHO}

### Claims — vanguards/guards

13. Arti has had vanguards enabled by default since 1.2.2 — *lite* mode for all onion-service circuits (client+service). *Full* mode (`vanguards.mode = "full"`) available, recommended only for services with >1 month uptime. Security bugs in 1.2.2–1.2.4 → minimum recommended 1.2.5. {source: https://blog.torproject.org/announcing-vanguards-for-arti/ | pub_date: 2024-07-16 | confidence: high | class: HECHO}

14. C Tor has vanguards-lite built in (`VanguardsLiteEnabled`); full vanguards requires the external addon plugin. For per-contact-pair rendezvous (short-lived, many services), lite on both sides is likely sufficient — full mode targets long-lived public services. {source: https://onionservices.torproject.org/dev/implementations/ (updated 2026-08-19) | confidence: high on feature status; PROPUESTA on sufficiency | class: HECHO/PROPUESTA}

### Claims — Arti embedding

15. Onion-service hosting is embeddable in-process today: `tor-hsservice` is a published crate exposing `OnionServiceConfigBuilder`, `RunningOnionService`, `StreamRequest`; `TorClient::create_onion_service` exists in arti-client; an in-tree example runs an HTTP hidden service entirely inside one process. Not daemon-only. {source: gitlab.torproject.org arti MR !1887 ; docs.rs tor-hsservice | confidence: high | class: HECHO}

16. Caveats: (a) `.onion` *client* connections are off by default in arti-client (`address_filter.allow_onion_addrs=true` needed; README still carries a pre-vanguards warning now stale). (b) Ephemeral (non-persistent) onion services are **not** supported — issue #1186 open; services need state-dir keys (fine for per-contact pairs: stable .onion wanted anyway). (c) Experimental APIs churn; MSRV now Rust 1.91 (2.5.0+). {source: arti README blame ; issue #1186 ; blog arti_2_5_0 | confidence: high | class: HECHO}

### Leads worth chasing
- Mike Perry's `vanguards` addon for C Tor — maintenance status unverified (matters only if C Tor + full vanguards; lite is built-in).
- Arti `onion-service-service` key file formats / keystore migration — check current `tor-keymgr` stability.
- Onionprobe public dashboards — per-service latency timeseries (`onion_service_latency_seconds`, `hsdir_latency_seconds`) for p50/p99.
- Descriptor republish delay after restricted-discovery change — rate-limited, concrete min interval not retrieved.
- arti 2.7.0 RPC surface — whether restricted-discovery keys can be mutated via RPC.

### Looked for but could not find
- Whether `doc/OnionService.md` was edited after Feb 2024 — still shows Feb-2024 text, but no commit formally marks onion-service hosting "production-grade"; de-facto signal is restricted discovery declared stable (1.7.0) + blogs urging upgrades.
- Tor Metrics/Onionprobe aggregate numbers for *client-observed* first-connect latency.
- Exact HSDir propagation window after first publish.
- Whether C Tor `SIGNAL RELOAD` picks up `.auth` files added for *file-configured* services.
