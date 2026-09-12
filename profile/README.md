[README.md] (https://github.com/user-attachments/files/32135540/README.md)

# QuantumShield Labs

**Secure messaging designed to stay secure after quantum computers arrive.**

We build and publish the **QSL Protocol** in the open: canonical specifications, conformance
vectors, reference implementations, and the relay infrastructure they run against.

> [!WARNING]
> **QSL is research-stage software. It has not been independently audited, and it is not ready
> to protect anyone who is actually at risk.** Please read [Status and honest limits](#status-and-honest-limits)
> before you use any of this.


---

## Why this exists

Encryption that is unbreakable today is not automatically unbreakable tomorrow.

An adversary with enough storage can record encrypted traffic now and simply wait. When a
cryptographically relevant quantum computer arrives, the classical key exchange protecting
those recordings falls, and the conversations open — retroactively, all at once. This is
called **harvest now, decrypt later**, and it is the reason post-quantum work cannot wait for
quantum computers to exist.

Messages that need to stay private for a decade need post-quantum protection *this* decade.

QSL is a messenger built for people whose conversations have long tails: lawyers, clinicians,
journalists, organizers, and anyone whose notes today could be evidence later.

---

## What makes it different

**No phone number.** Identity is a keypair you generate. Nothing is tied to a SIM, a carrier,
or a number that can be ported away from you.

**Self-hostable relays.** The server moves sealed bytes between mailboxes. You can run your
own, and the protocol does not assume the relay is trustworthy.

**Hybrid by construction, not as an option.** Every single message body is encrypted under a
key derived from *both* a classical and a post-quantum secret. Breaking one is not enough.

**Specified before implemented.** The protocol is written down as canonical documents with
numbered normative requirements, then implemented against conformance vectors. The spec is
public, so the design can be argued with rather than trusted.

---

## For cryptographers and reviewers

QSL Suite-2 (`protocol_version = 0x0500`, `suite_id = 0x0002`) is a **True Triple Ratchet**:
three ratchets composed so that every message key is hybrid.

| Component | Construction |
|---|---|
| **Classical DH ratchet** | X25519, advancing the root at epoch boundaries |
| **Classical symmetric ratchet** | `CK_ec -> ec_mk`, per message |
| **PQ symmetric ratchet** | `CK_pq -> pq_mk`, per message |
| **Hybrid combiner** | `mk = KMAC32(ec_mk, "QSP5.0/HYBRID", pq_mk \|\| 0x01)` |

**Primitives.** ML-KEM-768 (key encapsulation) - ML-DSA-65 (signatures) - X25519 -
AES-256-GCM (12-byte nonce, 16-byte tag) - SHA-512 - KMAC-256 for all key derivation, with
domain-separated labels.

**The interesting problem, and our answer.** A per-message PQ KEM operation is expensive in
bandwidth. Dropping PQ between key exchanges means most messages are not actually hybrid. QSL
resolves this with **SCKA** — Sparse Continuous Key Agreement. The PQ ratchet is seeded at
session establishment and *reseeded* by sparse ML-KEM events carried in message prefix fields,
so the PQ chain keeps advancing without paying KEM bandwidth on every message. Reseed events
are monotonic, fail-closed, and crash-safe. The design is specified in
[DOC-CAN-004](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/docs/canonical).

**Design commitments.**

- **Fail-closed everywhere.** Unknown version or suite is rejected with no silent fallback.
  Negotiation is downgrade-resistant and transcript-bound. TLS never offers a "connect anyway."
- **Transactional commit.** No ratchet state is persisted unless header *and* body decrypt
  succeed and every bound check passes. A rejected message mutates nothing.
- **Contributory DH.** All-zero X25519 outputs are rejected (RFC 7748 §6.1) on both the
  sending and receiving side, before the root absorbs them.
- **Explicit reject codes.** Failures map to stable, documented reason codes rather than a
  generic error, so conformance vectors can pin exact behaviour.
- **No server-held trust.** The relay stores route tokens only as digests and holds no session
  keys. Contact aliases are local-only; no display name is ever taken from the network.

**Where to start reading:**

1. [`docs/canonical/DOC-CAN-003`](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/docs/canonical) — the Suite-2 ratchet, normative
2. [`docs/canonical/DOC-CAN-004`](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/docs/canonical) — SCKA
3. [`inputs/suite2/vectors/`](https://github.com/QuantumShieldLabs/qsl-protocol) — conformance vectors
4. [`tools/refimpl/`](https://github.com/QuantumShieldLabs/qsl-protocol) — the reference implementation the client links against

Fourteen conformance vector categories are defined (KDFs, establishment, negotiation and
downgrade, parsing, out-of-order and replay, boundaries, PQ reseed, crash recovery, transcript
binding, cross-implementation interop). Coverage is real but **not complete**, and we would
rather you knew that than discovered it.

---

## Repositories

| Repository | What it is |
|---|---|
| [**qsl-protocol**](https://github.com/QuantumShieldLabs/qsl-protocol) | Canonical specifications, conformance vectors, and the research-stage reference implementation. Start here. |
| [**qsl-desktop**](https://github.com/QuantumShieldLabs/qsl-desktop) | The desktop client. Tauri + Rust with the `qsc` core in-process. Pre-release. |
| [**qsl-server**](https://github.com/QuantumShieldLabs/qsl-server) | The relay. Mailbox transport for sealed frames, plus the server-mediated invite plane. |
| [**qsl-attachments**](https://github.com/QuantumShieldLabs/qsl-attachments) | Encrypted attachment plane and runtime. No plaintext attachment handling. |
| [**.github**](https://github.com/QuantumShieldLabs/.github) | This profile and community health files. |

All code repositories are **AGPL-3.0**.

### What the relay does and does not see

This deserves precision, because "the server can't see anything" is a claim people make too
loosely.

**On the messaging plane, the relay is opaque transport.** It accepts sealed frames, holds them
in a mailbox addressed by a route token, and hands them back. It holds no session keys, performs
no ratchet operations, and cannot read message content.

**The invite plane is deliberately server-mediated.** Establishing a new contact uses relay
endpoints that create, redeem, and revoke invite slots. The relay verifies a presented
capability against a stored hash and burns one-shot tickets, which is what makes an invite
genuinely single-use and revocable. That is protocol participation, and we would rather name it
than describe the relay as semantics-free and have a reviewer find the invite routes.

**What a relay operator can still observe.** Traffic metadata: which mailboxes are active, how
much, how large, and when. Message *content* is end-to-end encrypted and the relay cannot read
it. Metadata resistance is an open work item, not a solved one.

---

## Status and honest limits

QSL is under active development and every artifact here should be treated as **research-grade
until a formal external security review is published.**

- **Not independently audited.** No third party has reviewed this cryptography. Our own
  reasoning is not a substitute, and we do not present it as one.
- **Pre-release.** Interfaces, wire formats, and on-disk state are still changing.
- **The canonical specifications are marked DRAFT**, and they mean it.
- **v1 targets Linux.** macOS and Windows come later.
- **The command-line client is a laboratory instrument, not the product.** It exposes
  affordances the shipped application deliberately does not.
- **Open defects exist and are tracked in the open.** A protocol can be correctly specified and
  still have implementation bugs that lose messages. We find them, file them, and fix them in
  public. Check the repository's issues and traceability records for current state.

**Do not use QSL to protect anyone whose safety depends on it.** Not yet. When that changes, it
will change because an external audit says so — not because we feel ready.

---

## Security reporting

Please do **not** file security-sensitive reports in public issues.

Use **GitHub private vulnerability reporting** on the affected repository, or follow the
process in that repository's `SECURITY.md`. If private reporting is unavailable, open a minimal
public issue with **no exploit details**, stating that you can share specifics privately.

Reports about the specifications themselves are as welcome as reports about the code. A flaw in
DOC-CAN-003 is worth more to us than a flaw in one implementation of it.

---

## Contributing

Issues and contributions are welcome. See `CONTRIBUTING.md` in the target repository for scope
and workflow.

Particularly valuable:

- **Adversarial review of the canonical specs.** Tell us where the normative text is ambiguous,
  circular, or silent on a case that occurs in practice.
- **Independent implementations.** Cross-implementation interop is a defined conformance
  category, and a second implementation is the strongest test a spec can get.
- **Conformance vector gaps.** If you can name a behaviour the categories cannot reach, that is
  a finding in itself.

---

## Licensing posture

The public repositories are governed by the licenses shipped in them (AGPL-3.0 for all code).
Any future commercial services or support offerings are separate from those licenses and should
not be read back into the public source terms.

---

<sub>QuantumShield Labs - United States - <https://www.quantumshieldlabs.org/></sub>
