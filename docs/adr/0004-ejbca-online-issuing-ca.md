# ADR-0004: EJBCA Community as Online Issuing CA

- **Status:** Accepted
- **Date:** 2026-07-16
- **Deciders:** Fred Barrie
- **Tags:** pki, issuing-ca, ejbca, online-ca

## Context

My Cloud needs an online intermediate / issuing CA subordinate to the
[HSM-backed offline root](0001-nitrokey-hsm2-offline-ca.md). Day-to-day leaf and
workload certificates are issued online; the offline root is used only for
ceremonies such as creating or renewing that intermediate.

The [my-cloud-pki](https://github.com/ffbarrie/my-cloud-pki) layout already
assumes a full private PKI surface: issuing CA, OCSP, EST, CRL publication,
Keycloak integration, and monitoring. The issuing CA choice must support that
surface without forcing a rewrite of the repository contract in
[ADR-0002](0002-my-cloud-pki-repository-layout.md).

The lab aims for production-grade *practices* (offline root, ceremonies,
profiles, revocation), while remaining self-hosted and operable by one person.

## Decision

Use **EJBCA Community Edition (EJBCA CE)** as the online issuing CA for My Cloud
PKI.

1. EJBCA CE runs as the always-on issuing CA service under `my-cloud-pki`
   (Compose), with persistent database storage on PostgreSQL
   ([ADR-0005](0005-postgresql-datastore.md)).
2. The issuing CA certificate is signed by the offline root (or, temporarily,
   by the [bootstrap software root](https://github.com/ffbarrie/my-cloud-pki/blob/main/bootstrap/software-root-ca.md)).
3. Certificate profiles, enrollment protocols, and revocation publishing
   (CRL / OCSP) are configured in EJBCA as the primary implementation.
   Dedicated `est/`, `ocsp/`, and `crl/` directories may hold publication
   config, reverse proxies, or later split-out services, but EJBCA is the
   system of record for issuance.
4. Identity integration (Keycloak) remains a later phase for admin / enrollment
   flows; it does not replace EJBCA as the CA.
5. Use the Community Edition Docker image (`keyfactor/ejbca-ce`) for this home-lab
   deployment. Enterprise-only features and commercial support are out of scope
   unless a future ADR revisits edition choice.
6. **Enrollment strategy (decided 2026-07-17):** Stay on EJBCA CE. Use
   **CMP and SCEP** (native CE servlets) as the near-term automated enrollment
   path. **EST remains a required capability** for the lab end-state, but it
   is **not** available in CE (`est.war` is missing; `/.well-known/est/…`
   returns 404). EST must be **built later**—either as a companion EST
   front-end that issues via EJBCA, or by revisiting EJBCA Enterprise in a
   follow-up ADR. ACME is likewise Enterprise-only and out of scope for now.
   Lab TLS profiles (`MyCloudServer` / `MyCloudServerEE`) stay shared across
   CMP/SCEP now and EST later.

## Consequences

### Positive

- One platform covers issuing CA, profiles, CRL, OCSP, and CE-native CMP/SCEP.
- Aligns with the offline-root / online-intermediate model already chosen in
  ADR-0001.
- Mature CA administration UI and certificate lifecycle operations.
- Compatible with later Keycloak-backed admin or enrollment integration.

### Negative / Trade-offs

- Operational complexity: Java application server, database, CA/profile
  configuration, backups, and upgrades.
- EJBCA CE is positioned by Keyfactor for learning, testing, and prototyping—not
  as a supported production Enterprise product. For this home lab that is an
  accepted trade-off; the goal is production-grade *practice*, not a commercial
  SLA.
- **EST must be built separately** (CE has no EST servlet). Until then,
  automated enrollment is CMP/SCEP-first; `est/` holds preparation notes and
  is not yet a working enrollment surface.
- Heavier than ACME-first tools such as Smallstep `step-ca` or HashiCorp Vault
  PKI for simple service TLS.

### Neutral

- Exact Compose image tags, ports, TLS termination, and profile templates are
  implementation details in `my-cloud-pki`. The relational database engine is
  decided in [ADR-0005](0005-postgresql-datastore.md).
- Leaf naming and SPIFFE / workload identity policy remain future work under
  ADR-0003 and follow-on ADRs.
- Building EST later (companion front-end vs Enterprise) does not change the
  decision that EJBCA is the issuing CA.

## Alternatives Considered

| Option | Why not chosen |
| ------ | -------------- |
| Smallstep `step-ca` | Excellent for ACME and lightweight lab TLS; weaker fit for the EST / OCSP / CRL / enterprise-CA layout already assumed by the repo. |
| HashiCorp Vault PKI | Strong if Vault were already the secrets platform; less natural as a classic CA with EST and external CRL/OCSP publication. |
| OpenSSL scripts only | Fine for bootstrap and offline ceremonies; insufficient for an always-on issuing CA with enrollment and revocation services. |
| Dogtag / FreeIPA CA | Viable enterprise FOSS CA, but heavier identity coupling and less aligned with the current Compose-centric PKI repo. |
| EJBCA Enterprise | Capable, but commercial licensing and support are unnecessary for this home-lab scope. |

## Related

- [ADR-0001: Nitrokey HSM 2 for Offline CA](0001-nitrokey-hsm2-offline-ca.md)
- [ADR-0002: my-cloud-pki Repository Layout](0002-my-cloud-pki-repository-layout.md)
- [ADR-0003: PKI Certificate Naming and Subject DN Policy](0003-pki-certificate-naming.md)
- [ADR-0005: PostgreSQL for Online Stateful Services](0005-postgresql-datastore.md)
- Implementation: https://github.com/ffbarrie/my-cloud-pki/tree/main/issuing-ca
- EST notes (CE limitation): https://github.com/ffbarrie/my-cloud-pki/tree/main/est
- Keyfactor docs: https://docs.keyfactor.com/ejbca/latest/tutorial-start-out-with-ejbca-docker-container
- Community vs Enterprise: https://www.ejbca.org/community-vs-enterprise/
