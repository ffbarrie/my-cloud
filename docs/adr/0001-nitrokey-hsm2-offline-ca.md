# ADR-0001: Nitrokey HSM 2 for Offline CA

- **Status:** Accepted
- **Date:** 2026-07-16
- **Deciders:** Fred Barrie
- **Tags:** pki, hsm, offline-ca, nitrokey

## Context

My Cloud needs a private PKI so services can trust each other with TLS certificates issued from a controlled hierarchy. The root of that trust is an offline Certificate Authority (CA): its private key must stay offline and protected against theft, accidental exposure, and single-device failure.

Software-only key storage (files on disk, even encrypted) is too fragile for a root CA in a home-lab environment that still aims for production-grade practices. A Hardware Security Module (HSM) keeps the private key inside tamper-resistant hardware and performs signing operations without exporting the key material.

Two devices are required so that loss, failure, or unavailability of a single HSM does not permanently strand the offline CA.

## Decision

Use **two Nitrokey HSM 2** devices to secure the offline root CA for My Cloud PKI.

1. The offline CA private key material is generated and held on Nitrokey HSM 2 hardware; it is not stored as a software key on disk.
2. Both devices participate in the offline CA setup so either can be used to recover or continue CA operations if the other is lost or fails.
3. The devices remain offline except during deliberate CA ceremonies (for example, issuing or renewing intermediate CA certificates).
4. Day-to-day certificate issuance for workloads uses online intermediate CAs; those intermediates are subordinate to this HSM-backed offline root.

## Consequences

### Positive

- Root CA private keys stay in hardware and are not exportable as ordinary files.
- Two devices provide redundancy for hardware failure or loss.
- Offline operation reduces exposure of the root CA to network compromise.
- Aligns the home-lab PKI with common enterprise practice: HSM-backed offline root, online intermediates.

### Negative / Trade-offs

- Ceremonies require physical access to the Nitrokey devices and careful operational procedure.
- Backup and recovery depend on both HSM devices being initialized and managed correctly; losing both devices without a recoverable backup path would strand the root.
- Nitrokey HSM 2 capacity and algorithm support constrain key types and how many objects can be stored.
- Operators must learn OpenSC / PKCS#11 tooling and Nitrokey-specific workflows.

### Neutral

- Intermediate and leaf certificate automation can evolve independently; this ADR only covers how the offline root is secured.
- Exact ceremony steps, PIN/SO-PIN policy, and device custody procedures will be documented separately as runbooks.

## Alternatives Considered

| Option | Why not chosen |
| ------ | -------------- |
| Software keys on encrypted disk | Simpler, but root key material remains extractable if the host or backup is compromised. |
| Single Nitrokey HSM 2 | Insufficient redundancy; device loss or failure could permanently lose the offline CA. |
| Cloud HSM / managed CA | Conflicts with an offline, self-hosted trust root for the home lab. |
| Smart cards without HSM-class key protection | Weaker fit for holding a long-lived offline root CA key with redundancy. |
| YubiHSM or other HSMs | Viable, but Nitrokey HSM 2 is the chosen hardware for this deployment. |

## Related

- [ADR-0002: my-cloud-pki Repository Layout](0002-my-cloud-pki-repository-layout.md)
- [My Cloud roadmap](../../roadmap.md)
- [ADR-0003: PKI Certificate Naming and Subject DN Policy](0003-pki-certificate-naming.md)
- [Offline CA ceremony runbook](https://github.com/ffbarrie/my-cloud-pki/blob/main/offline-ca/ceremony-runbook.md)
- [Bootstrap software root CA](https://github.com/ffbarrie/my-cloud-pki/blob/main/bootstrap/software-root-ca.md) — non-HSM path for labs without hardware, or while waiting for Nitrokeys; not a substitute for this ADR
- [ADR-0004: EJBCA Community as Online Issuing CA](0004-ejbca-online-issuing-ca.md)
- Follow-up docs (to be written): device custody and PIN policy
