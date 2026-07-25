# QuantumShield Labs

**Post-quantum secure messaging, built in public — with the proofs and the gaps both attached.**

QSL is a research-stage messaging protocol and client suite designed post-quantum-**first**, not
retrofitted. Hybrid **ML-KEM-768 + X25519** key agreement, **ML-DSA-65 + Ed25519** signatures, a
triple-ratchet-style session layer with continuous key agreement, and a relay that forwards
**opaque bytes** and is never given the ability to parse a protocol message.

Everything is AGPL-3.0 and public: specifications, conformance vectors, ProVerif models, CI
evidence, design decisions, and the reasoning behind them — **including the parts that are not
finished.** We publish the claim boundary as carefully as the claims.

---

## Repositories

| Repo | What it is |
|---|---|
| **[qsl-protocol](https://github.com/QuantumShieldLabs/qsl-protocol)** | The spine. Specifications, conformance vectors, formal models, and the `qsc` client core (Rust). Governance and traceability records live here. |
| **[qsl-desktop](https://github.com/QuantumShieldLabs/qsl-desktop)** | Desktop client. Tauri v2 + Rust with `qsc` linked in-process; static vanilla HTML/CSS/JS frontend — **no npm, no node, no JS dependencies.** |
| **[qsl-server](https://github.com/QuantumShieldLabs/qsl-server)** | Transport-only relay. Opaque ciphertext in, opaque ciphertext out. No protocol parsing, no crypto, no plaintext. Single binary, self-hostable. |
| **[qsl-attachments](https://github.com/QuantumShieldLabs/qsl-attachments)** | Encrypted attachment plane. Opaque ciphertext parts on disk; no plaintext attachment handling on any service surface. |

---

## What is actually different here

**Post-quantum by construction.** The hybrid is real and load-bearing, not a label on a
classical design. A downgrade to classical-only fails closed rather than degrading quietly.

**No phone number. No account. No directory.** Identity is a keypair you generate locally.
There is no server-side user record to subpoena, breach, or correlate, because there is no
server-side user record.

**The relay is designed to be untrusted.** It sees opaque bytes addressed to route tokens. It
performs no protocol parsing and holds no protocol state — that boundary is an invariant in the
relay's own README, not an aspiration. You are expected to run your own.

**Per-device trust, fail-closed.** `VERIFIED` is not `TRUSTED`. Verifying one device does not
implicitly extend trust to another, and the handshake rejects before it verifies on malformed
input — with tests that count verification calls to prove the ordering.

**Formal models that actually run.** ProVerif models live in
[`formal/`](https://github.com/QuantumShieldLabs/qsl-protocol/tree/main/formal) and execute in
CI on every push to `main`, not as a one-time artifact in a paper.

**Explicit delivery semantics.** `accepted_by_relay` and `peer_confirmed` are distinct states,
because conflating them is how "delivered" becomes a lie. Receipt policy is `off` / `batched` /
`immediate`, stated rather than assumed.

**An audit trail you can read.** Every design decision, its alternatives, and why the
alternatives lost are in
[`DECISIONS.md`](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/DECISIONS.md).
Findings — including the uncomfortable ones — go in the improvement ledger with measurements
attached, and get fixed in the open.

---

## Status: what works, and what does not

We would rather you find this here than discover it after cloning.

**Working today**
- The `qsc` CLI performs real end-to-end encrypted messaging against a live relay, with
  reproducible evidence for the wrong-peer, stale-peer, replay, and corrupt-delivery negatives.
- The relay runs as a hardened single binary with deterministic error taxonomy, rate limiting,
  route caps, and idle TTL.
- The desktop client's local lifecycle — vault creation, identity, unlock with escalating-delay
  protection, idle autolock, settings — and server configuration: relay address, access token and
  CA file, with a Test connection that reports what the relay actually answered.

**Not working yet — stated plainly**
- **The desktop client cannot send a message.** It can be pointed at a relay and can test that
  connection — the app opens a network connection only when you press Test connection — but
  messaging itself is not built.
- **No external security review has been completed.** Treat everything as research-grade.
- Multi-device is `primary_only` — fan-out is designed but not built.
- Group messaging is designed but not built.
- No production deployment, observability, incident-response, or public-ingress story.
- **Not an anonymity system.** Not metadata-free. Traffic-shape correlation is an open problem
  we name rather than hide.

The full, longer version of that boundary is in the
[qsl-protocol README](https://github.com/QuantumShieldLabs/qsl-protocol#what-is-not-proven-yet)
and the
[release-readiness evidence map](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/docs/public/RELEASE_READINESS_EVIDENCE_MAP.md).

---

## Start here

- **Evaluating the protocol?** →
  [`docs/public/INDEX.md`](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/docs/public/INDEX.md)
  — every claim links to the exact proof or the exact gap.
- **Reviewing the crypto?** →
  [`docs/public/EXTERNAL_REVIEW_PACKAGE.md`](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/docs/public/EXTERNAL_REVIEW_PACKAGE.md)
  and the ProVerif models in `formal/`.
- **Want to run something?** →
  [`docs/demo/DEMO_ACCEPTANCE_CRITERIA.md`](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/docs/demo/DEMO_ACCEPTANCE_CRITERIA.md)
  for a bounded, reproducible demo path.
- **Curious how decisions get made?** →
  [`DECISIONS.md`](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/DECISIONS.md)
  and [`NEXT_ACTIONS.md`](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/NEXT_ACTIONS.md).

**Most useful contributions right now:** negative tests, reproduction notes, claim-boundary
review, and anywhere the documentation says more than the code earns.

---

## Security

**Do not file sensitive vulnerability reports in public issues.** Use the target repository's
`SECURITY.md` and GitHub's private reporting. We would rather fix it and then publish the
finding together with the fix — which is what we do.

## License and posture

Source is **AGPL-3.0-only**; see `LICENSE` in each repository. Any future commercial service or
support offering is separate from these repositories and does not modify the AGPL terms on the
source published here.

**QSL is under active development. Nothing here is production-ready, and we will say so until it
is.**

---

*[quantumshieldlabs.org](https://www.quantumshieldlabs.org/)*
