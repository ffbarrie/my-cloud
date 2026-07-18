# My Cloud Roadmap

This document tracks the planned and in-progress components for the My Cloud home lab environment.

## Current Focus

### PKI — In Progress

Certificate authority and TLS infrastructure for the rest of the stack.

- Repository: https://github.com/ffbarrie/my-cloud-pki
- **Enrollment (near-term):** CMP and SCEP on EJBCA Community (path 1)
- **Enrollment (later):** build EST — required end-state; not in CE
  ([ADR-0004](docs/adr/0004-ejbca-online-issuing-ca.md),
  [est notes](https://github.com/ffbarrie/my-cloud-pki/tree/main/est))

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
