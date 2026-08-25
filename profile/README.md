# QuantumShield Labs

**Post-quantum secure messaging, built in public — with the proofs and the gaps both attached.**

QSL is a messaging protocol and client suite designed post-quantum-**first**, not retrofitted:
hybrid **ML-KEM-768 + X25519** key agreement, **ML-DSA-65** signatures, a triple-ratchet-style
session layer with continuous key agreement, and a relay that forwards **opaque bytes** and is
never given the ability to parse a protocol message. No phone number, no account, no directory —
identity is a keypair you generate locally, and the server keeps no user record because there is
no user record to keep.

And one thing almost no project publishes: **our prediction ledger, misses included.** Before
each unit of work runs, its expected outcome is written down; afterwards the measured result is
scored beside it. **168 scored rows so far: 57 hits, 101 misses, 7 partial.** A project that
only reports its hits is not telling you anything.

---

## Repositories

| Repo | What it is |
|---|---|
| **[qsl-protocol](https://github.com/QuantumShieldLabs/qsl-protocol)** | The spine. Specifications, conformance vectors, formal models, and the `qsc` client core (Rust). Governance and traceability live here. |
| **[qsl-desktop](https://github.com/QuantumShieldLabs/qsl-desktop)** | Desktop client. Tauri v2 + Rust with `qsc` linked in-process; hand-written vanilla HTML/CSS/JS — **no npm, no node, no JavaScript dependencies.** |
| **[qsl-server](https://github.com/QuantumShieldLabs/qsl-server)** | Transport-only relay. Opaque ciphertext in, opaque ciphertext out. No protocol parsing, no crypto, no plaintext. Single binary, self-hostable — you are expected to run your own. |
| **[qsl-attachments](https://github.com/QuantumShieldLabs/qsl-attachments)** | Encrypted attachment plane. Opaque ciphertext parts on disk; no plaintext attachment handling on any service surface. |

---

## What is actually different here

**Formal models that actually run.** Six bounded models and four ProVerif models live in
[`formal/`](https://github.com/QuantumShieldLabs/qsl-protocol/tree/main/formal) and execute in
CI on every push to `main` — not as a one-time artifact in a paper. The ProVerif runner asserts
the expected result **per query**, and its first assertion is a sanity pair: a positive control
that must prove and a negative control that must refute, so a vacuously-accepting verifier
fails the build instead of passing it.

**Security tests that count to zero.** The handshake rejects malformed input *before* it
verifies anything — proven by handing it a counting verifier and asserting the
signature-verification count is **exactly zero**. Per-device trust is fail-closed:
`VERIFIED` is not `TRUSTED`, and verifying one device extends nothing to another.

**Post-quantum by construction.** The hybrid is load-bearing, not a label on a classical
design. A downgrade to classical-only fails closed rather than degrading quietly, and a
rejected negotiation commits no session state.

**The relay is designed to be untrusted.** It sees opaque bytes addressed to route tokens,
performs no protocol parsing, and holds no protocol state — an invariant in the relay's own
README, not an aspiration.

**Field-proven, not just CI-proven.** Real machines have completed the full journey — invite
minted in the GUI, code redeemed on a second machine, post-quantum handshake finishing over a
live relay, peer pinned by fingerprint — including the client's own background trigger
completing the exchange with no operator assistance. The marker streams are in the evidence
trail.

**Explicit delivery semantics.** `accepted_by_relay` and `peer_confirmed` are distinct states,
because conflating them is how "delivered" becomes a lie.

**An audit trail you can actually read.** **1,393** decision records with their alternatives
and why the alternatives lost, in
[`DECISIONS.md`](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/DECISIONS.md).
**239** improvement-ledger entries, findings included. **501** evidence documents, **932** test
plans. Every claim on the public evidence pages links to the exact proof — or the exact gap.

---

## Status: what works, and what does not

We would rather you find this here than discover it after cloning.

**Working today**
- The `qsc` CLI performs real end-to-end encrypted messaging against a live relay, with
  reproducible evidence for the wrong-peer, stale-peer, replay, and corrupt-delivery negatives.
- The relay runs as a hardened single binary with a deterministic error taxonomy, rate
  limiting, route caps, and idle TTL.
- The desktop client: vault lifecycle with escalating-delay unlock protection and idle
  autolock; relay configuration with a Test connection that reports what the relay actually
  answered; and the full invite flow — **create, review, revoke, and redeem**. A redeemed
  invite completes a post-quantum handshake and records the peer, automatically.

**Not working yet — stated plainly**
- **The desktop client cannot send a message.** Contacts can be established end to end, but
  there is no contact list to browse and messaging itself is not built. It is next.
- **No external security review has been completed.** Treat everything as research-grade.
- Multi-device fan-out and group messaging: designed, not built.
- No production deployment, observability, or public-ingress story.
- **Not an anonymity system.** Not metadata-free. Traffic-shape correlation is an open problem
  we name rather than hide.

The longer version of that boundary lives in the
[qsl-protocol README](https://github.com/QuantumShieldLabs/qsl-protocol#readme) and the
[release-readiness evidence map](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/docs/public/RELEASE_READINESS_EVIDENCE_MAP.md).
Those pages refresh at slice close; the repositories' own history is always the current record.

---

## Start here

- **Evaluating the protocol?** →
  [`docs/public/INDEX.md`](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/docs/public/INDEX.md)
- **Reviewing the crypto?** →
  [`docs/public/EXTERNAL_REVIEW_PACKAGE.md`](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/docs/public/EXTERNAL_REVIEW_PACKAGE.md)
  and the ProVerif models in `formal/`.
- **Want to run something?** →
  [`docs/demo/DEMO_ACCEPTANCE_CRITERIA.md`](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/docs/demo/DEMO_ACCEPTANCE_CRITERIA.md)
  for a bounded, reproducible demo path.
- **Curious how decisions get made?** →
  [`DECISIONS.md`](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/DECISIONS.md)
  and [`NEXT_ACTIONS.md`](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/NEXT_ACTIONS.md).

**Most useful contributions right now:** negative tests, reproduction notes, and claim-boundary
review — anywhere the documentation says more than the code earns.

---

## Security

**Do not file sensitive vulnerability reports in public issues.** Use the target repository's
`SECURITY.md` and GitHub's private reporting. We would rather fix it and publish the finding
together with the fix — which is what we do.

## License

Everything here is **AGPL-3.0-only** — the canonical FSF text, byte-identical, in every
repository. Build on it, audit it, self-host it; keep it free.

**QSL is under active development. Nothing here is production-ready, and we will say so until
it is.**

---

*[quantumshieldlabs.org](https://www.quantumshieldlabs.org/)*
