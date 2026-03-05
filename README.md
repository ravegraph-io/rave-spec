
<img width="1051" height="701" alt="ChatGPT Image Mar 5, 2026, 11_39_15 AM (1)" src="https://github.com/user-attachments/assets/42cbddc0-82c9-4ff4-a3c9-2bf95ecfa878" />

# RAVE

**Reliability & Validation Engineering** — a claim-centric framework for making reliability explicit, falsifiable, and continuously validated.

## What is RAVE?

Most organisations have SLOs, runbooks, and incident postmortems — but they rarely model reliability *explicitly*. RAVE fills that gap.

RAVE defines a structured vocabulary for:

- **Claims** — explicit, falsifiable statements about system behaviour
- **Evidence** — machine-referencable signals that support or weaken claims
- **Confidence** — a dynamic score that decays without revalidation
- **Incidents** — structured records of contradicted claims
- **Scopes** — first-class boundaries within which claims apply

It sits above heterogeneous tooling (CI, observability, SBOM, OPA) as a coordination layer — comparable in role to OpenTelemetry for telemetry or OpenAPI for APIs, but for **reliability claims and validation**.

For a deeper introduction, see [`rave-overview.md`](rave-overview.md).

## Contents

| Path | Description |
|------|-------------|
| [`spec/`](spec/) | The RAVE protocol specification |
| [`examples/`](examples/) | Example RAVE YAML files |

## Specification

The RAVE spec lives in [`spec/rave-spec-v0.1.md`](spec/rave-spec-v0.1.md). It defines the data model, claim types, decay rules, and protocol semantics.

## Examples

The [`examples/`](examples/) directory contains annotated YAML files showing claims, evidence, and scopes in practice:

| File | What it illustrates |
|------|---------------------|
| `deploy-rollback.claim.yaml` | Single-claim format |
| `payment-service.claims.yaml` | Multi-claim format |
| `checkout-platform.scope.yaml` | Scope hierarchy |
| `ci-test-results.evidence.yaml` | CI evidence with methodology + result |
| `chaos-experiment.evidence.yaml` | Chaos result evidence (failed outcome) |
| `slo-status.evidence.yaml` | SLO status evidence |

## Reference Implementation

[RAVEgraph](https://github.com/mesgme/ravegraph) is the reference implementation of this specification — a CLI and service for managing RAVE claims and evidence.

## Contributing

This specification is in early draft (v0.1). Feedback, questions, and proposals are welcome via [GitHub Issues](../../issues).
