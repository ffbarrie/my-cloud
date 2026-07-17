# ADR-0005: PostgreSQL for Online Stateful Services

- **Status:** Accepted
- **Date:** 2026-07-16
- **Deciders:** Fred Barrie
- **Tags:** database, postgresql, ejbca, keycloak, pki

## Context

The online issuing CA ([ADR-0004](0004-ejbca-online-issuing-ca.md)) needs a
persistent relational database. The initial `my-cloud-pki` Compose scaffold used
MariaDB because that is Keyfactor’s documented default for EJBCA CE.

The My Cloud roadmap places **Identity (Keycloak)** immediately after PKI.
Keycloak’s common and best-supported deployment path is PostgreSQL. Running
MariaDB for EJBCA and PostgreSQL for Keycloak would work, but it doubles backup,
upgrade, and operational surface for a one-person lab.

EJBCA supports PostgreSQL via JDBC; the Community container bundles the
PostgreSQL driver. No production CA data exists yet, so choosing PostgreSQL
before first boot avoids a later migration.

## Decision

Use **PostgreSQL** as the datastore for online My Cloud stateful services,
starting with EJBCA and intended for Keycloak next.

1. EJBCA CE connects to PostgreSQL (`jdbc:postgresql://...`) in
   `my-cloud-pki` Compose.
2. Each application gets its own database (or schema) on PostgreSQL—do not share
   one application schema across EJBCA and Keycloak.
3. MariaDB / MySQL remain supported by EJBCA upstream but are **not** the My
   Cloud lab standard.
4. Exact PostgreSQL major version, backup tooling, and any future shared DB
   host are implementation details; this ADR fixes the engine choice.

## Consequences

### Positive

- One database engine for PKI now and Identity next.
- Aligns with Keycloak’s usual deployment guidance.
- Avoids migrating EJBCA after SuperAdmin and issuing CA state exist.
- PostgreSQL is well supported in later Kubernetes operator / Helm ecosystems.

### Negative / Trade-offs

- Leaves Keyfactor’s MariaDB-centric EJBCA tutorials; operators must translate
  examples to PostgreSQL JDBC URLs and ops practices.
- Loses MariaDB Galera as the “default” HA story from Keyfactor docs (HA can be
  revisited later with Postgres patterns if needed).
- Slightly less copy-paste fidelity with the official EJBCA Docker Compose
  quickstart.

### Neutral

- Offline CA / HSM ceremonies are unaffected.
- In-memory H2 remains unsuitable except for throwaway experiments; this ADR
  does not change that.
- Monitoring, connection pooling, and TLS-to-database can be added later without
  revisiting the engine choice.

## Alternatives Considered

| Option | Why not chosen |
| ------ | -------------- |
| MariaDB / MySQL | Keyfactor’s preferred EJBCA path, but creates a second engine when Keycloak arrives. |
| Separate engines (MariaDB for EJBCA, Postgres for Keycloak) | Viable, but unnecessary ops cost for this lab. |
| Shared single database/schema for all apps | Couples upgrades and access control; prefer one engine, separate databases. |
| Defer the decision until after first EJBCA boot | Cheap now, expensive after CA state exists. |

## Related

- [ADR-0002: my-cloud-pki Repository Layout](0002-my-cloud-pki-repository-layout.md)
- [ADR-0004: EJBCA Community as Online Issuing CA](0004-ejbca-online-issuing-ca.md)
- Implementation: https://github.com/ffbarrie/my-cloud-pki/blob/main/compose.yaml
- Roadmap Identity phase: [roadmap.md](../../roadmap.md)
