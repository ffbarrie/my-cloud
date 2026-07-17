# ADR-0002: my-cloud-pki Repository Layout

- **Status:** Accepted
- **Date:** 2026-07-16
- **Deciders:** Fred Barrie
- **Tags:** pki, repository-layout, offline-ca, docker-compose

## Context

My Cloud PKI spans offline ceremonies (HSM-backed root CA), online services (issuing CA, OCSP, EST, CRL distribution), and supporting pieces (identity integration, monitoring, backups). Implementation lives in a dedicated repository ([my-cloud-pki](https://github.com/ffbarrie/my-cloud-pki)), while architecture and ADRs stay in [my-cloud](https://github.com/ffbarrie/my-cloud).

The offline CA will not run continuously in Docker, but its documentation, bootstrap scripts, ceremony notes, and HSM initialization instructions still need a canonical home. Splitting offline material into a separate repo would fragment the PKI source of truth.

## Decision

Structure `my-cloud-pki` as a single canonical PKI repository with this layout:

```text
my-cloud-pki
├── compose.yaml
├── .env.example
├── README.md
├── docs/
├── bootstrap/
├── offline-ca/
├── issuing-ca/
├── ocsp/
├── est/
├── crl/
├── keycloak/
├── monitoring/
├── scripts/
└── backups/
```

### Roles of top-level areas

| Path | Role |
| ---- | ---- |
| `compose.yaml` / `.env.example` | Bring up online PKI services for the lab |
| `docs/` | PKI-specific runbooks, ceremony notes, and operational docs |
| `bootstrap/` | One-time or rarely run setup helpers |
| `offline-ca/` | Offline root CA material: HSM init, ceremony scripts, docs (not a long-running Compose service) |
| `issuing-ca/` | Online intermediate / issuing CA configuration and service assets |
| `ocsp/` | OCSP responder |
| `est/` | Enrollment over Secure Transport |
| `crl/` | CRL publication / distribution |
| `keycloak/` | Identity integration needed by PKI enrollment or admin flows |
| `monitoring/` | Health and observability for online PKI components |
| `scripts/` | Shared operational utilities |
| `backups/` | Backup procedures and non-secret backup tooling/config |

Architecture decisions that affect My Cloud as a whole (including PKI) continue to be recorded in `my-cloud` under `docs/adr/`.

## Consequences

### Positive

- One repository is the source of truth for the entire PKI stack.
- Offline CA practices stay next to the online services they anchor.
- Compose remains focused on continuously running components.
- Directory names map cleanly to PKI concerns and future ADRs/runbooks.

### Negative / Trade-offs

- The repo mixes always-on services with ceremony-only assets; contributors must treat `offline-ca/` as offline/operational, not as a default Compose service.
- Secrets and HSM-related material must never be committed; layout alone does not enforce that discipline.
- Some directories will stay sparse until later phases of the roadmap.

### Neutral

- Exact service images, ports, and Compose profiles will be defined as components are implemented.
- Component-level READMEs inside each directory can grow without changing this top-level contract.

## Alternatives Considered

| Option | Why not chosen |
| ------ | -------------- |
| Separate `my-cloud-offline-ca` repository | Splits the PKI source of truth and makes ceremonies harder to discover. |
| Flat scripts-only PKI repo without service directories | Poor fit once issuing CA, OCSP, EST, and CRL are online. |
| Put all PKI docs only in `my-cloud` | `my-cloud` stays architecture-centric; implementation and runbooks belong with the code. |
| Run offline CA as a permanent Compose service | Conflicts with [ADR-0001](0001-nitrokey-hsm2-offline-ca.md) offline root posture. |

## Related

- [ADR-0001: Nitrokey HSM 2 for Offline CA](0001-nitrokey-hsm2-offline-ca.md)
- [My Cloud roadmap](../../roadmap.md)
- [ADR-0003: PKI Certificate Naming and Subject DN Policy](0003-pki-certificate-naming.md)
- Implementation repository: https://github.com/ffbarrie/my-cloud-pki
