# My Cloud Roadmap

This document tracks the planned and in-progress components for the My Cloud home lab environment.

## Current Focus

### PKI — In Progress

Certificate authority and TLS infrastructure for the rest of the stack.

- Repository: https://github.com/ffbarrie/my-cloud-pki
- **Enrollment:** companion **EST** MVP live (`/cacerts`, `/simpleenroll` on port 8444);
  `/simplereenroll` deferred v1.1. **CMP** native on CE. **SCEP** native CE
  **CA/Client mode** under `scep/` (RA mode is Enterprise-only).
  ([ADR-0004](docs/adr/0004-ejbca-online-issuing-ca.md),
  [est](https://github.com/ffbarrie/my-cloud-pki/tree/main/est),
  [scep](https://github.com/ffbarrie/my-cloud-pki/tree/main/scep))

## Up Next

These components will follow once PKI is in place.

| Component   | Status  | Description                                      |
| ----------- | ------- | ------------------------------------------------ |
| Identity    | Planned | Authentication and authorization for services      |
| Networking  | Planned | Network design, DNS, ingress, and connectivity   |
| Kubernetes  | Planned | Cluster platform for running workloads           |
| Monitoring  | Planned | Observability, metrics, logs, and alerting       |

## Order of Operations

1. **PKI** — establish trust and TLS across the environment
2. **Identity** — integrate auth with PKI-issued credentials
3. **Networking** — connect services with secure, reliable networking
4. **Kubernetes** — deploy and orchestrate workloads
5. **Monitoring** — observe health and performance across the stack
