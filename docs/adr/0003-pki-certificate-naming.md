# ADR-0003: PKI Certificate Naming and Subject DN Policy

- **Status:** Accepted
- **Date:** 2026-07-16
- **Deciders:** Fred Barrie
- **Tags:** pki, naming, subject-dn, root-ca, intermediate-ca

## Context

My Cloud PKI needs consistent, human-readable names for root, intermediate, and leaf
certificates. The [offline CA ceremony runbook](https://github.com/ffbarrie/my-cloud-pki/blob/main/offline-ca/ceremony-runbook.md)
requires operators to confirm the root CA subject before initialization, but the
runbook should describe *how* to perform ceremonies—not define deployment-specific
identity values.

Certificate subject names are public once certificates are published. They must be
stable for long-lived roots, distinguishable across the CA hierarchy, and easy for
forks to customize without inheriting another operator's trust anchor identity.

## Decision

Adopt a naming policy with three layers:

1. **Policy (this ADR)** — naming rules and rationale, fork-friendly.
2. **Examples (`my-cloud-pki/offline-ca/profiles/*.cnf.example`)** — committed
   templates showing the My Cloud default values.
3. **Local deployment values** — operator-specific copies such as
   `offline-ca/profiles/root-ca.cnf` and `offline-ca/.env`, not committed to git.

### Subject DN rules

| Certificate type | CN pattern | Purpose |
| ---------------- | ---------- | ------- |
| Offline root CA | `{Lab Name} Offline Root CA` | Identifies the HSM-backed offline trust anchor |
| Bootstrap root CA | `{Lab Name} Bootstrap Root CA` | Identifies a temporary or non-HSM software root |
| Issuing / intermediate CA | `{Lab Name} Issuing CA` | Identifies the online signing CA |
| Leaf / workload | Service hostname or SPIFFE ID | Identifies the workload |

Additional distinguished name fields:

| Field | Root / intermediate | Leaf |
| ----- | ------------------- | ---- |
| `O` | Lab or organization name (e.g. `My Cloud`) | Same as lab policy |
| `OU` | `PKI` for CA certificates | Optional; workload-specific if used |
| `C`, `ST`, `L` | Omit unless there is a deliberate reason | Usually omit in a private lab |

### Naming principles

1. **CN identifies the CA, not a hostname.** Root and intermediate CAs use
   descriptive names, not DNS names like `ca.internal`.
2. **Root and intermediate names must be distinct.** Operators must be able to tell
   which certificate is which in tooling output and trust stores.
3. **Include "Offline" in the HSM root CN** to reinforce [ADR-0001](0001-nitrokey-hsm2-offline-ca.md)
   offline-root posture.
4. **Include "Bootstrap" in any software-root CN.** The OpenSSL file-based root used
   while waiting for HSMs (or by labs without HSMs) must not reuse the Offline
   root name. See the [bootstrap software root runbook](https://github.com/ffbarrie/my-cloud-pki/blob/main/bootstrap/software-root-ca.md).
5. **Do not embed secrets in the DN.** PINs, tokens, and internal recovery labels
   do not belong in certificate subjects.
6. **Root CAs do not use Subject Alternative Names (SANs).** SANs are for leaf and
   some intermediate use cases, not offline or bootstrap roots.
7. **Forks must choose their own values.** Reusing `My Cloud Offline Root CA` or
   `My Cloud Bootstrap Root CA` creates a different deployment that should not be
   confused with the upstream example.

### My Cloud default values

These are the reference values for this deployment and the committed examples:

| Field | Offline root CA | Bootstrap root CA | Issuing CA |
| ----- | --------------- | ----------------- | ---------- |
| `CN` | `My Cloud Offline Root CA` | `My Cloud Bootstrap Root CA` | `My Cloud Issuing CA` |
| `O` | `My Cloud` | `My Cloud` | `My Cloud` |
| `OU` | `PKI` | `PKI` | `PKI` |

Implementation templates live in:

- Offline / HSM: [my-cloud-pki/offline-ca/profiles/](https://github.com/ffbarrie/my-cloud-pki/tree/main/offline-ca/profiles)
- Bootstrap / software: [my-cloud-pki/bootstrap/profiles/](https://github.com/ffbarrie/my-cloud-pki/tree/main/bootstrap/profiles)

## Consequences

### Positive

- Ceremony operators have a single profile source to verify before signing.
- Forks inherit policy and examples without inheriting identity.
- Certificate hierarchy is readable in `openssl x509` output and trust stores.
- Naming policy stays in `my-cloud`; implementation templates stay in `my-cloud-pki`.

### Negative / Trade-offs

- Subject DN alone does not prevent operator error; ceremonies must still verify
  profiles explicitly.
- Organizations that later formalize under a legal entity name may want to rotate
  or re-issue with updated `O`—roots are long-lived, so choose `O` carefully.

### Neutral

- Validity periods, key algorithms, and extension profiles remain in certificate
  profile files and future ADRs/runbooks.
- Leaf naming for workloads will be defined when EJBCA certificate profiles and
  enrollment automation are built ([ADR-0004](0004-ejbca-online-issuing-ca.md)).

## Alternatives Considered

| Option | Why not chosen |
| ------ | -------------- |
| DNS-style root CN (e.g. `root.pki.my.cloud`) | Reads like a service hostname; roots are identities, not endpoints. |
| Put CN only in the ceremony runbook | Runbooks are procedural; identity belongs in policy and profiles. |
| Hardcode CN in scripts | Poor fork experience; profiles and local config are clearer. |
| Commit operator-specific `root-ca.cnf` | Couples the repo to one deployment; examples plus local override are better. |

## Related

- [ADR-0001: Nitrokey HSM 2 for Offline CA](0001-nitrokey-hsm2-offline-ca.md)
- [ADR-0002: my-cloud-pki Repository Layout](0002-my-cloud-pki-repository-layout.md)
- [Offline CA ceremony runbook](https://github.com/ffbarrie/my-cloud-pki/blob/main/offline-ca/ceremony-runbook.md)
- [ADR-0004: EJBCA Community as Online Issuing CA](0004-ejbca-online-issuing-ca.md)
- [Bootstrap software root CA runbook](https://github.com/ffbarrie/my-cloud-pki/blob/main/bootstrap/software-root-ca.md)
- Offline profile examples: https://github.com/ffbarrie/my-cloud-pki/tree/main/offline-ca/profiles
- Bootstrap profile examples: https://github.com/ffbarrie/my-cloud-pki/tree/main/bootstrap/profiles
