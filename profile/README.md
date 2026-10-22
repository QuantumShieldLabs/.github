# QuantumShield Labs

**Open research into post-quantum messaging.**

QSL brings together protocol specifications, conformance vectors, Rust implementations and a desktop prototype. The work focuses on hybrid message keys, explicit state transitions and evidence that others can inspect and challenge.

**Research stage. Not independently audited or production-ready. Do not use QSL for safety-critical communications.**

## Explore the project

| Repository | Start here if you want to… |
|---|---|
| [qsl-protocol](https://github.com/QuantumShieldLabs/qsl-protocol) | Read the specifications, inspect vectors and review the client core. |
| [qsl-desktop](https://github.com/QuantumShieldLabs/qsl-desktop) | Explore the Tauri desktop prototype and its current feature limits. |
| [qsl-server](https://github.com/QuantumShieldLabs/qsl-server) | Understand the relay's mailbox, invitation and operational boundaries. |
| [qsl-attachments](https://github.com/QuantumShieldLabs/qsl-attachments) | Inspect the separate encrypted-object service and its storage contract. |

## What we are investigating

- **Hybrid message keys:** combine classical and post-quantum secret material, with sparse key-agreement events and per-message chain advancement.
- **Explicit failure behavior:** specify rejection, replay, persistence and recovery rather than leaving them to application convention.
- **Locally generated identities:** explore messaging without requiring a phone number.
- **Metadata minimization:** treat routing, timing, size and retention as separate security questions. QSL does not currently establish anonymity or resistance to global traffic analysis.

The [public evidence guide](https://github.com/QuantumShieldLabs/qsl-protocol/blob/main/docs/public/INDEX.md) connects claims to their evidence and remaining limits. The [canonical specifications](https://github.com/QuantumShieldLabs/qsl-protocol/tree/main/docs/canonical) define the protocol; they remain drafts.

## Review and contribute

Useful contributions include reproducible failures, specification ambiguities, conformance gaps, independent implementations and clearer documentation. Start with the affected repository's contribution guide.

For a security-sensitive finding, follow its `SECURITY.md` and use private reporting. Keep exploit details out of public issues.

Each repository's license controls its source. The four code repositories identify their license as `AGPL-3.0-only`.
