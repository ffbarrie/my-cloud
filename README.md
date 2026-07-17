# My Cloud

This repository documents my home lab cloud environment.

See the [roadmap](roadmap.md) for current focus and planned work.

## Architecture decisions

Significant design choices for My Cloud and its components are recorded under [docs/adr](docs/adr/).

- [ADR-0001: Nitrokey HSM 2 for Offline CA](docs/adr/0001-nitrokey-hsm2-offline-ca.md)
- [ADR-0002: my-cloud-pki Repository Layout](docs/adr/0002-my-cloud-pki-repository-layout.md)
- [ADR-0003: PKI Certificate Naming and Subject DN Policy](docs/adr/0003-pki-certificate-naming.md)
- [ADR-0004: EJBCA Community as Online Issuing CA](docs/adr/0004-ejbca-online-issuing-ca.md)

## Components

- PKI - https://github.com/ffbarrie/my-cloud-pki
- Identity
- Networking
- Kubernetes
- Monitoring

## License

All code, scripts, and documentation in this repository are licensed under the [Apache License 2.0](LICENSE).