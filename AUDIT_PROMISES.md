# rust-libp2p trust assumptions and promises audit

This audit extracts user-visible assumptions/promises from the current source and documentation.

## 1) Availability (5)

1. **Connection setup can be bounded by timeout wrappers.**
   The `TransportTimeout` wrapper adds configurable timeouts to inbound and outbound connection setup, preventing indefinite hangs during dial/upgrade paths.

2. **Listener lifetime is not timed out by connection timeout wrappers.**
   The timeout layer explicitly states `listen_on` itself is not timed out, only accepted-connection setup is, preserving long-lived listening availability.

3. **Connection flood/overcommit can be constrained via configurable limits.**
   `connection-limits::Behaviour` enforces limits for pending/established inbound/outbound and per-peer/total counts.

4. **When limits are exceeded, failures are explicit and typed.**
   Denials emit identifiable errors (`ConnectionDenied` / `Exceeded`) instead of silent drops, supporting operability and fast recovery.

5. **Core stack is modular and transport/muxer/protocol composition is a first-class API.**
   Repository structure and crate exports promise users they can assemble resilient networking stacks from interchangeable components.

## 2) Integrity (5)

1. **Signed payloads can be verified against signing keys.**
   `SignedEnvelope` includes payload, type, signature, and key, and exposes verification APIs.

2. **Domain separation and payload type binding are required for safe extraction.**
   `payload_and_signing_key` requires expected payload type and domain separation string, reducing cross-protocol misuse.

3. **TLS handshake integrity is constrained to TLS 1.3 for libp2p TLS.**
   Verifier limits protocol versions to TLS 1.3 and excludes TLS 1.2 negotiation.

4. **Peer identity binding in TLS is enforced when a target peer is known.**
   Client verification checks that the certificate-derived peer ID matches intended `remote_peer_id`, aborting on mismatch.

5. **Certificate chain and signature constraints are strict.**
   libp2p TLS verifier requires exactly one presented certificate and validates parsing/signatures according to libp2p rules.

## 3) Confidentiality (5)

1. **Authenticated encryption is expected through protocol upgrades in normal secure configurations.**
   Repository docs frame authenticated encryption upgrades as standard transport composition.

2. **libp2p TLS config negotiates ALPN `libp2p` and modern TLS-only cipher/protocol settings.**
   Client/server builders pin ALPN and TLS 1.3 ciphers/protocol version sets, implying encrypted channel confidentiality.

3. **Mutual TLS client authentication is mandatory in the verifier.**
   `offer_client_auth()` returns true and client cert verification is enforced, limiting anonymous peers in TLS mode.

4. **Private network mode is an explicit opt-in capability users can assume exists.**
   The top-level crate exposes `pnet` as a feature, signaling a confidentiality boundary option for private overlays.

5. **Security disclosure flow assumes vulnerabilities are handled privately first.**
   Security policy directs private reporting rather than public issue disclosure, reducing immediate exposure of exploitable details.

