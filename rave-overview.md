# RAVE — Reliability & Validation Engineering

## 1. Core Idea

RAVE (Reliability & Validation Engineering) is a claim-centric framework for making reliability:

- Explicit
- Falsifiable
- Continuously validated
- Machine-referencable
- Confidence-scored

It is **not** another SRE tool.
It is **not** another governance framework.

It is a coordination layer between:

- Reliability intent (claims)
- Operational behaviour (evidence)
- Validation logic
- Incidents
- Confidence over time

---

## 2. Foundational Reframe

### Reliability Is a Claim

Reliability is not uptime.
Reliability is not dashboards.
Reliability is not an SLO.

Reliability is a statement about system behaviour under defined conditions.

Example:

> "Deployments are reversible within 10 minutes under normal load."

This is a *reliability claim*.

Claims must be:

- Explicit
- Scoped
- Falsifiable
- Backed by evidence

---

## 3. The Gap RAVE Addresses

Most organisations have:

- SLOs
- Checklists
- Runbooks
- Security scans
- SBOMs
- Policy engines
- Incident postmortems

But they rarely:

- Model reliability claims explicitly
- Link claims to structured evidence
- Track claim confidence over time
- Detect when claims silently decay
- Treat incidents as structured contradictions

RAVE coordinates these elements.

---

## 4. RAVE Principles v0.1

1. Reliability must be expressed as explicit claims.
2. Claims must be falsifiable.
3. Claims must declare their assumptions.
4. Evidence must be machine-referencable.
5. Validation must be continuous.
6. Confidence decays over time.
7. Incidents expose weak or contradicted claims.

---

## 5. Core Concepts

### Claim
A structured statement about system behaviour.

Fields:
- claim_id
- statement
- owner — string or structured object (`name`, optional `team`, optional `contact`). A plain string is shorthand for `{ name: "<value>" }`.
- status
- category (optional — well-known category classifying the reliability concern)
- scope
- assumptions
- falsification_signals
- confidence_score
- last_validated

---

### Scope
A first-class entity defining a boundary within which claims apply.

Fields:
- type
- target
- parent (optional)
- dependencies (optional)
- description (optional)

Scopes have a composite identity (`type` + `target`) and form a containment
hierarchy through optional `parent` references ("is part of" relationships).

Well-known scope types:
- system
- application
- service
- component
- endpoint
- pipeline
- release
- environment
- team

Claims reference scopes by `{type, target}`. Scopes that are referenced by
claims but not explicitly declared are auto-created as root scopes.

Scopes are stored in `.scope.yaml` files.

---

### Evidence
Machine-referencable signals supporting or weakening a claim.

Fields:
- evidence_id
- type
- reference
- description
- timestamp
- methodology
- result
- claim_ids
- metadata
- freshness_window
- quality_score

Evidence has two distinct aspects:
- **Methodology** — the repeatable process that produces the signal (e.g. the test suite, the Prometheus query, the chaos experiment design)
- **Result** — the point-in-time output from executing that methodology (e.g. "247 tests passed", "p99 latency 142ms")

Methodology fields:
- description
- tooling
- frequency
- coverage

Result fields:
- outcome (pass | fail | inconclusive)
- summary
- value
- unit
- run_id

Quality reflects methodology rigour; freshness reflects result recency.

Evidence types:
- ci_log
- deployment_metric
- observability_metric
- sbom
- policy_output
- chaos_result
- security_scan
- audit_log
- incident_report
- manual_test
- slo_status
- telemetry_coverage
- alert_metric
- change_assessment
- service_catalog

---

### Falsifier
A condition that, when met, disproves a claim. Falsifiers are binary:
the condition is either met (claim contradicted, confidence → 0.0) or
not met (claim upheld).

Example:
- rollback_duration > 5 minutes disproves "rollbacks complete within 5 minutes"

---

### Confidence
Not binary.
A dynamic score influenced by:
- Evidence freshness (result recency)
- Evidence quality (methodology rigour)
- Contradictions
- Validation frequency

Confidence decays over time if not revalidated.

---

### Readiness
An aggregate evaluation of whether all claims at a scope meet the reliability bar.

A scope is ready when:
- No claims are contradicted
- All active claims have confidence >= threshold (default 0.70)

Readiness is the "definition of done" that AI agents check before proceeding.
It is a derived concept — computed from claims, not stored as a field.

---

## 6. Incidents

An incident is not just "something went wrong."

An incident is:

> A structured record created when one or more reliability claims
> are contradicted — their confidence drops to zero.

RAVE treats incidents as separate entities linked to the contradicted claims,
carrying their own timeline, scope, remediation, and learnings.

---

## 7. What RAVE Is

Short term:
- A claim-centric reliability engineering framework.

Medium term:
- A reliability governance model.

Long term:
- A coordination layer across reliability, governance, and system safety.

---

## 8. RAVEgraph

RAVEgraph is the reference implementation of the RAVE specification.

It is:

- A CLI + library first
- Then a service with API
- Tool-agnostic
- Vendor-neutral

It does not replace:
- SBOM tools
- OPA
- CI systems
- Observability platforms

It coordinates their outputs.

---

## 9. Long-Term Vision

RAVE aims to become:

- A formal open specification
- A reference implementation (RAVEgraph)
- A shared vocabulary for reliability claims
- A coordination layer above heterogeneous tooling

Comparable ambition (structurally):
- OpenTelemetry (for telemetry)
- OpenAPI (for APIs)
- SPDX (for SBOMs)

But for:
Reliability Claims and Validation.

---
