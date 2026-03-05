# RAVE Specification

| Field   | Value                              |
|---------|------------------------------------|
| Title   | RAVE — Reliability & Validation Engineering |
| Version | 0.1.0                              |
| Status  | Published                          |
| Date    | 2026-03-05                         |

---

## 1. Introduction

### 1.1 What RAVE Is

RAVE (Reliability & Validation Engineering) is a claim-centric framework for making reliability explicit, falsifiable, continuously validated, machine-referencable, and confidence-scored.

RAVE is a coordination layer between:

- **Reliability intent** — expressed as structured claims about system behaviour
- **Operational behaviour** — captured as machine-referencable evidence
- **Validation logic** — conditions that support or weaken claims
- **Incidents** — structured contradictions of claims
- **Confidence** — a dynamic score that reflects the current strength of a claim

Short term, RAVE is a reliability engineering framework. Medium term, it is a reliability governance model. Long term, it is a coordination layer across reliability, governance, and system safety.

### 1.2 The Problem RAVE Solves

Most organisations have SLOs, checklists, runbooks, security scans, SBOMs, policy engines, and incident postmortems. But they rarely:

- Model reliability claims explicitly
- Link claims to structured evidence
- Track claim confidence over time
- Detect when claims silently decay
- Treat incident reports as structured contradictions of claims

Reliability is not uptime, dashboards, or an SLO. Reliability is a statement about system behaviour under specific defined conditions. RAVE provides the structure to express, validate, and track these statements.

### 1.3 What RAVE Is Not

- **Not an SRE tool.** RAVE does not replace observability platforms, CI systems, or runbooks. It coordinates their outputs.
- **Not a governance framework.** RAVE is focused on reliability claims and evidence, not organisational policy.
- **Not a replacement for existing tooling.** RAVE does not replace SBOM tools, OPA, or any specific vendor product. It sits above heterogeneous tooling as a coordination layer.

### 1.4 Comparable Work

RAVE occupies a similar structural role to:

- **OpenTelemetry** — for telemetry
- **OpenAPI** — for APIs
- **SPDX** — for SBOMs

But for **reliability claims and validation**.

---

## 2. Scope

### 2.1 In Scope

This specification defines:

- **Claims** — structured statements about system behaviour (section 6.1)
- **Evidence** — machine-referencable signals supporting or weakening claims (section 6.2)
- **Falsifiers** — conditions that invalidate or reduce confidence in a claim (section 6.3)
- **Confidence** — a dynamic score reflecting current claim strength (section 6.4)
- **Incidents** — reliability claims whose confidence has dropped to zero (section 6.5)
- **Scopes** — first-class entities defining boundaries within which claims apply, with containment hierarchy (section 6.6)
- **Readiness** — aggregate evaluation of scope confidence for agent workflows (section 6.7)
- **File formats** — the YAML schemas for expressing claims and scopes (section 7)
- **Principles** — the foundational design principles of the framework (section 5)
- **Terminology** — shared vocabulary for RAVE concepts (section 4)

### 2.2 Out of Scope

This specification does **not** define:

- Tooling or reference implementations (see RAVEgraph)
- Integration protocols with external systems (SBOM, OPA, CI, etc.)
- APIs or service interfaces
- User interfaces or CLI commands
- Deployment or operational guidance

These may be addressed in future specifications or companion documents.

---

## 3. Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

---

## 4. Terminology

This section defines the formal vocabulary of RAVE. All terms are drawn from existing normative prose in sections 5–7 — no new semantics are introduced here. Each entry is a concise definition with a cross-reference to the authoritative section.

Terms are grouped by logical category for scannability. Within each group, terms are listed alphabetically.

---

### 4.A Core Entities

Terms that have their own data model and lifecycle (see section 6).

#### Annotation

A human-authored Value Object attached to a Claim carrying prose context, rationale, or review notes. Annotations MUST NOT affect confidence scoring.

See: section 6.1.5

#### Claim

The primary first-class entity in RAVE. A structured, falsifiable statement about expected system behaviour under defined conditions. The aggregate root through which reliability is expressed, tracked, and validated. Other core concepts (Evidence, Falsifier, Confidence, Incident) are defined in relation to Claims.

See: section 6.1

#### Evidence

A machine-referencable signal that supports or weakens a Claim. Evidence MUST be fetchable, evaluable, and timestamped. Evidence MAY be linked to zero or more Claims.

See: section 6.2

#### Falsifier

A structured, machine-evaluable condition that, when met, invalidates or reduces confidence in one or more Claims. The machine-evaluable counterpart to a Falsification Signal.

See: section 6.3

#### Incident

A structured record of a reliability contradiction — an independent entity created when one or more Claims are contradicted by operational reality. An Incident carries its own identity, timeline, scope, and remediation metadata, and feeds learnings back into the Claims it affected.

See: section 6.5

#### Scope

A first-class entity defining a boundary within which Claims apply. Scopes have a composite identity (`type` + `target`) and MAY form a containment hierarchy through optional `parent` references. Claims reference Scopes to declare what they apply to; Scopes provide the structure for roll-up views, drill-down navigation, and scope-aware dashboards.

See: section 6.6

---

### 4.B Core Properties

Derived or computed concepts that are properties of entities, not standalone entities.

#### Confidence

A derived, dynamic, non-binary score (0.0–1.0) on a Claim reflecting the current strength of that claim. Computed from evidence freshness, quality, falsifier outcomes, and validation frequency. A score of 0.0 indicates no confidence; 1.0 indicates full confidence.

See: section 6.4

#### Contradiction

The state reached when a Claim's `confidence_score` drops to 0.0, triggering the `active` → `contradicted` lifecycle transition.

See: section 6.1.6

#### Decay

The mandatory decrease in a Claim's `confidence_score` over time in the absence of fresh supporting evidence. Decay encodes the expectation that claims require ongoing evidence to remain credible.

See: sections 5.6, 6.4.4

#### Freshness

The measure of how current a piece of Evidence is relative to the present moment. Governed by the `freshness_window` field on the Evidence entity.

See: section 6.2.6

#### Staleness

The state of Evidence whose age exceeds its `freshness_window`. Stale evidence contributes reduced or zero weight to confidence scoring.

See: section 6.2.6

---

### 4.C Claim Anatomy

Sub-concepts that make up or constrain a Claim.

#### Assumption

A boundary condition declared on a Claim defining the envelope within which the claim holds. Expressed as human-readable strings. At least one Assumption is REQUIRED per Claim.

See: section 6.1.4

#### Falsification Signal

A human-readable declaration on a Claim of a condition that would invalidate it. The prose counterpart to a machine-evaluable Falsifier entity. At least one Falsification Signal is REQUIRED per Claim.

See: sections 6.1.2, 6.3

---

### 4.D Validation and Evaluation

Process concepts relating to how claims are assessed over time.

#### Continuous Validation

The practice of re-evaluating Claims on an ongoing basis as new Evidence arrives, rather than at a single point in time. Systems that implement RAVE SHOULD support Continuous Validation.

See: section 5.5

#### Evaluation Window

The time window over which a Falsifier condition is assessed (e.g. `PT1H`). Defined on the condition object within a Falsifier.

See: section 6.3.4

#### Unattached Evidence

Evidence with no linked `claim_ids`. Signals a gap in the reliability model — observable signals exist without claims to give them meaning.

See: section 6.2.8

#### Validation

The act of re-evaluating a Claim against its Evidence and Falsifiers. A successful Validation updates `last_validated` and resets the decay clock.

See: sections 5.5, 6.4.3

---

### 4.E Falsifier Sub-concepts

#### Orphaned Falsifier

A Falsifier with an empty `claim_ids` list. Defined but not yet linked to any Claim.

See: section 6.3.6

---

### 4.F Reference and Design

Implementation and design-level terms.

#### Machine-Referencable

The property of Evidence that it can be fetched and evaluated programmatically via URI, API, log query, or equivalent. Human-only artefacts (prose documents, verbal assertions) do not qualify as machine-referencable.

See: section 5.4

#### Quality Score

A numeric (0.0–1.0) measure of the inherent reliability of an Evidence source. Contributes to Confidence scoring as Q_avg.

See: sections 6.2.2, 6.4.3

#### RAVEgraph

The reference implementation of the RAVE specification. A CLI-first tool, then service. Tool-agnostic and vendor-neutral.

See: sections 1.4, 2.2

---

## 5. Principles

The following principles govern the design and use of RAVE. Each principle is stated using RFC 2119 normative language (see section 3) and accompanied by a rationale.

### 5.1 Reliability MUST be expressed as explicit claims

Reliability is not an implicit property of a system — it is a statement about expected behaviour under defined conditions. Every reliability assertion MUST be captured as a structured claim with a defined scope.

**Rationale.** Implicit reliability assumptions are invisible, untestable, and impossible to coordinate across teams. Making claims explicit is the foundation on which all other RAVE concepts depend.

### 5.2 Claims MUST be falsifiable

Every claim MUST define at least one condition under which it would be considered invalid. A claim that cannot be disproven provides no actionable signal.

**Rationale.** Non-falsifiable claims cannot be validated, scored, or contradicted. Without falsifiability, a claim is an aspiration rather than an engineering statement. Falsifiability ensures that claims remain grounded in observable system behaviour.

### 5.3 Claims MUST declare their assumptions

Every claim MUST explicitly state the conditions and boundaries under which it holds. These assumptions scope the claim and define the envelope within which it is valid.

**Rationale.** A claim without declared assumptions is unbounded — it implicitly asserts validity under all conditions, which is rarely true. Declaring assumptions makes the scope of a claim transparent and enables precise validation.

### 5.4 Evidence MUST be machine-referencable

Evidence supporting or weakening a claim MUST be addressable by a machine — via a URI, API response, log query, or equivalent reference. Human-only artefacts (prose documents, verbal assertions) do not qualify as evidence.

**Rationale.** Machine-referencable evidence enables automation, reproducibility, and continuous validation. If evidence cannot be fetched and evaluated programmatically, it cannot participate in confidence scoring or decay detection.

### 5.5 Validation SHOULD be continuous

Claim validation SHOULD be performed on an ongoing basis rather than at a single point in time. Systems that implement RAVE SHOULD re-evaluate claims as new evidence becomes available.

**Rationale.** A claim validated once provides a snapshot, not a guarantee. Systems change, environments drift, and evidence ages. Continuous validation ensures that confidence reflects the current state of the system rather than a historical assertion.

### 5.6 Confidence MUST decay over time

In the absence of fresh supporting evidence, the confidence score of a claim MUST decrease. Stale evidence does not sustain confidence indefinitely.

**Rationale.** Systems are not static — infrastructure changes, dependencies update, and configurations drift. A claim that was valid last month may no longer hold today. Confidence decay encodes the expectation that claims require ongoing evidence to remain credible.

### 5.7 Incidents SHOULD expose weak or contradicted claims

When an incident occurs, systems SHOULD identify the claims that were weakened or contradicted by the incident. Incidents provide a feedback loop from operational failures back to the reliability model.

**Rationale.** Incidents are evidence that one or more reliability claims did not hold. Linking incidents to claims closes the loop between operational reality and the reliability model, enabling teams to strengthen claims, add falsifiers, and improve coverage over time.

---

## 6. Core Concepts

### 6.0 Design Decisions

The following decisions govern the relationships between core concepts:

1. **Claims are the aggregate root.** Evidence, Falsifiers, Confidence, and Incidents are defined in relation to Claims. Claims are the primary entity through which reliability is expressed and tracked. Scopes (section 6.6) are a separate first-class entity that Claims reference — they define the boundaries within which claims apply and provide the containment hierarchy for roll-up views.

2. **Evidence has a many-to-many relationship with Claims.** A single piece of evidence MAY support or weaken multiple claims. Evidence MAY also exist unattached to any claim. Unattached evidence signals a gap in the reliability model — it indicates that observable signals exist without corresponding claims to give them meaning.

3. **Incidents are separate entities linked to Claims.** An Incident is not a state on a single Claim but an independent entity with its own identity and lifecycle. A single incident MAY affect the confidence of multiple claims simultaneously. Incidents are triggered when claim confidence drops to zero, but they carry their own metadata (timeline, scope, remediation) and feed learnings back into the claims they affected.

### 6.1 Claim

A **Claim** is the primary first-class entity in RAVE — a structured, falsifiable statement about expected system behaviour under defined conditions. Evidence, Falsifiers, Confidence, and Incidents exist in relation to Claims (design decision 6.0 #1). A Claim is the aggregate root through which reliability is expressed, tracked, and validated. Each Claim references a Scope (section 6.6) that defines the boundary of what it applies to.

#### 6.1.1 Definition

A Claim asserts that a system, component, or service behaves in a specific way under explicitly stated assumptions. Every Claim MUST be falsifiable (principle 5.2) and MUST declare the assumptions under which it holds (principle 5.3).

A Claim is **not** a metric, an SLO, or a test result. It is a first-class reliability statement that links to Evidence (section 6.2), is evaluated by Falsifiers (section 6.3), carries a Confidence score (section 6.4), and MAY be contradicted by Incidents (section 6.5).

#### 6.1.2 Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `claim_id` | `string` | REQUIRED | Unique identifier for the claim. Format is implementation-defined (e.g. UUID, URN, slug). |
| `statement` | `string` | REQUIRED | Human-readable assertion of expected behaviour. MUST be specific enough to be falsifiable. |
| `owner` | `string` or `object` | REQUIRED | Claim owner. If string, interpreted as `{ name: "<value>" }`. If object, see section 6.1.2.1. |
| `status` | `string` (enum) | REQUIRED | Lifecycle state of the claim. One of: `draft`, `active`, `contradicted`, `retired`. See section 6.1.6. |
| `scope` | `object` | REQUIRED | Reference to a Scope entity (section 6.6) defining what the claim applies to. See section 6.1.3. |
| `assumptions` | `list[string]` | REQUIRED (>=1) | Boundary conditions under which the claim holds (principle 5.3). See section 6.1.4. |
| `falsification_signals` | `list[string]` | REQUIRED (>=1) | Conditions that would invalidate the claim (principle 5.2). Each signal is a human-readable declaration; machine-evaluable structure is defined at the Falsifier level (section 6.3). |
| `confidence_score` | `number` (0.0–1.0) | REQUIRED | Current confidence in the claim, where 0.0 indicates no confidence and 1.0 indicates full confidence. Subject to decay per principle 5.6. |
| `last_validated` | `string` (ISO 8601) | REQUIRED | Timestamp of the most recent validation. Used as an input to confidence decay calculations. |
| `category` | `string` | OPTIONAL | Well-known category classifying the reliability concern. See section 6.1.8. |
| `annotations` | `list[Annotation]` | OPTIONAL | Human-authored notes attached to the claim. See section 6.1.5. |

Implementations MAY extend this schema with additional fields, but MUST NOT remove or redefine the semantics of REQUIRED fields.

##### 6.1.2.1 Owner Object

When the `owner` field is an object, it MUST conform to the following schema:

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | `string` | REQUIRED | Team or individual identifier. MUST be a non-empty string. |
| `team` | `string` | OPTIONAL | Human-readable team name. |
| `contact` | `string` | OPTIONAL | Primary contact channel (e.g. Slack channel, email address, PagerDuty service URL). Format is implementation-defined. |

**Rules:**

- For backward compatibility, a plain string is accepted and interpreted as `{ name: "<string>" }`.
- Implementations MUST accept both the string form and the object form.
- When a plain string is provided, `team` and `contact` are unset.

Evidence linking is unidirectional — Evidence entities reference Claims via their `claim_ids` field (section 6.2.2). To discover evidence supporting a Claim, implementations query Evidence entities whose `claim_ids` list includes the claim's `claim_id`.

#### 6.1.3 Scope

Every Claim MUST declare a **scope** that defines the boundary of what the claim applies to. The `scope` field is a reference to a Scope entity (section 6.6) expressed as an object with two fields:

| Field | Type | Required | Description |
|---|---|---|---|
| `type` | `string` | REQUIRED | A well-known scope type identifying the category of the target. See section 6.6.3. |
| `target` | `string` | REQUIRED | The specific entity the claim applies to. Format depends on the scope type. |

**Scoping rules:**

- A Claim MUST have exactly one scope.
- The `type` field SHOULD use a well-known scope type (section 6.6.3) where applicable.
- The `target` field MUST identify a specific entity — it MUST NOT be a wildcard or empty string.
- The `(type, target)` pair SHOULD match a Scope declared in a `.scope.yaml` file (section 7.8) or registered by the implementation. If no matching Scope is declared, implementations SHOULD auto-create it as a root scope (section 6.6.7).
- Claims with overlapping scopes are permitted. When multiple claims apply to the same target, each claim is evaluated independently. Conflict resolution between overlapping claims is implementation-defined.

#### 6.1.4 Assumptions

Every Claim MUST declare at least one assumption (principle 5.3). Assumptions are boundary conditions — they define the envelope within which the claim is expected to hold.

Assumptions are expressed as human-readable strings. They are **not** machine-evaluable predicates; their purpose is to make the scope of validity transparent to humans reviewing or operating on the claim.

**Guidelines for writing assumptions:**

- Assumptions SHOULD be specific and testable in principle, even if not machine-evaluated.
- Assumptions SHOULD NOT restate the claim itself.
- Assumptions SHOULD capture environmental, load, dependency, or configuration conditions.

**Examples:**

- `"Upstream payment gateway responds within 2s at p99"`
- `"Kafka cluster has at least 3 healthy brokers"`
- `"Traffic does not exceed 10,000 requests per second"`
- `"Database connection pool is configured with a maximum of 50 connections"`

#### 6.1.5 Annotations

An **Annotation** is a Value Object attached to a Claim. Annotations carry human-authored prose — context, rationale, review notes, or expert attestations. Annotations are **not** Evidence (principle 5.4): they MUST NOT affect confidence scoring or be used as inputs to Falsifiers.

**Annotation fields:**

| Field | Type | Required | Description |
|---|---|---|---|
| `text` | `string` | REQUIRED | Free-form human-readable content. |
| `author` | `string` | OPTIONAL | The person or system that authored the annotation. |
| `created_at` | `string` (ISO 8601) | RECOMMENDED | Timestamp of when the annotation was created. |

Annotations are append-only by convention — implementations SHOULD NOT silently modify or delete annotations.

#### 6.1.6 Lifecycle

A Claim progresses through the following lifecycle states:

| Status | Description |
|---|---|
| `draft` | Claim is being authored. Not yet subject to validation or confidence scoring. |
| `active` | Claim is live. Subject to continuous validation (principle 5.5) and confidence decay (principle 5.6). |
| `contradicted` | Confidence has dropped to zero. The claim's assertion does not hold under current conditions. MAY trigger an Incident (section 6.5). |
| `retired` | Permanently decommissioned. No further transitions are permitted. Confidence is frozen at the value it held at retirement. Retired claims are exempt from decay and further confidence computation. |

**Transitions:**

```
draft → active              Owner activates the claim for validation.
active → contradicted       Confidence drops to zero.
active → retired            Owner decommissions the claim.
contradicted → active       Claim is revised and confidence is restored above zero.
contradicted → retired      Claim is permanently decommissioned after contradiction.
draft → retired             Claim is abandoned before activation.
retired → (terminal)        No transitions out of retired.
```

**Transition rules:**

- A Claim in `draft` status MUST NOT be subject to confidence decay or continuous validation.
- A Claim MUST have at least one linked Evidence entity before transitioning to `active`. Linked evidence means at least one Evidence entity exists whose `claim_ids` list includes this claim's `claim_id`. This ensures that no active claim exists without evidence to validate it.
- A Claim MUST transition to `contradicted` when its `confidence_score` reaches 0.0.
- A Claim MAY transition from `contradicted` back to `active` only if it has been revised — i.e. its `statement`, `assumptions`, or `falsification_signals` have been updated.
- The `retired` state is **terminal**. Once retired, a Claim MUST NOT transition to any other state. Re-establishing a retired claim's assertion requires creating a new Claim.

#### 6.1.7 Examples

**Example 1 — Deployment rollback claim (active, structured owner):**

```yaml
claim_id: "claim-deploy-rollback-001"
statement: "Production deployments can be rolled back within 5 minutes"
owner:
  name: "platform-team"
  team: "Platform Engineering"
  contact: "#platform-eng"
status: "active"
category: "reliability"
scope:
  type: "pipeline"
  target: "deploy-to-prod"
assumptions:
  - "Deployment uses blue-green strategy with warm standby"
  - "Previous deployment artefact is retained for at least 24 hours"
  - "Rollback does not require database migration reversal"
falsification_signals:
  - "Rollback duration exceeds 5 minutes in any production deployment"
  - "Previous deployment artefact is unavailable at rollback time"
confidence_score: 0.82
last_validated: "2026-02-14T10:30:00Z"
annotations:
  - text: "Validated during Q1 deployment drill. One edge case identified — rollback with config change took 4m48s."
    author: "jchen"
    created_at: "2026-02-14T11:00:00Z"
```

**Example 2 — Exactly-once delivery claim (draft):**

```yaml
claim_id: "claim-kafka-exactly-once-001"
statement: "Order events are delivered exactly once to the fulfilment service"
owner: "messaging-team"
status: "draft"
category: "reliability"
scope:
  type: "component"
  target: "order-events/kafka-consumer"
assumptions:
  - "Kafka consumer uses idempotent processing with deduplication window of 7 days"
  - "Consumer group has a single active instance per partition"
  - "Broker replication factor is at least 3"
falsification_signals:
  - "Duplicate order event detected in fulfilment service within deduplication window"
  - "Order event missing from fulfilment service after consumer acknowledgement"
confidence_score: 0.0
last_validated: "2026-02-10T09:00:00Z"
annotations:
  - text: "Claim drafted based on design review. Awaiting integration test evidence before activation."
    author: "amorales"
    created_at: "2026-02-10T09:15:00Z"
```

#### 6.1.8 Well-Known Claim Categories

Claims MAY be classified using the `category` field to indicate the type of reliability concern they address. The following well-known categories are defined:

| Category | Question it answers | Example statement |
|----------|-------------------|-------------------|
| `reliability` | Does the system behave correctly under defined conditions? | "P99 latency stays below 200ms under normal load" |
| `ownership` | Is responsibility clearly defined and current? | "Service payments-api has a documented owner and escalation path" |
| `slo` | Are service level objectives defined and tracked? | "99.9% availability SLO is defined and monitored in Datadog" |
| `observability` | Is telemetry adequate to detect problems? | "All API endpoints emit structured logs, metrics, and traces" |
| `alert_quality` | Are alerts trustworthy and actionable? | "Alert signal-to-noise ratio exceeds 0.8 with MTTA under 5 minutes" |
| `change_risk` | Is this change safe to ship? | "Release v2.3.1 has acceptable risk: no DB migrations, feature-flagged" |
| `dependency` | Are dependencies healthy and documented? | "All upstream dependencies have active claims with confidence above 0.70" |

**Rules:**

- `category` is OPTIONAL. Claims without a `category` are uncategorized.
- Implementations MAY define additional categories beyond this list.
- Category MUST NOT affect confidence scoring — it is classification metadata only.
- The `category` field SHOULD use a well-known category where applicable.

### 6.2 Evidence

**Evidence** is a machine-referencable signal that supports or weakens a claim. Evidence is what makes reliability claims testable and falsifiable in practice. It provides the observable data that enables confidence scoring, continuous validation, and claim contradiction detection.

Evidence has a many-to-many relationship with Claims (design decision 6.0 #2): a single piece of evidence MAY support or weaken multiple claims, and evidence MAY exist unattached to any claim. Unattached evidence signals a gap in the reliability model — it indicates that observable signals exist without corresponding claims to give them meaning.

#### 6.2.1 Definition

Evidence is a reference to an observable signal that can be:

- **Fetched** — retrieved programmatically via URI, API, query, or file path
- **Evaluated** — interpreted by validation logic (either human or automated) to support or weaken a claim
- **Timestamped** — associated with a point in time to enable freshness and staleness determination

Evidence MUST be machine-referencable (principle 5.4). Human-only artefacts such as prose documents, verbal assertions, or un-timestamped notes do **not** qualify as evidence. However, such artefacts MAY be stored as Annotations (section 6.1.5) on a Claim.

Evidence has **two distinct aspects**:

- **Methodology** — the repeatable process, procedure, or query used to generate a signal. A methodology describes *how* evidence is obtained: the test suite configuration, the Prometheus query, the chaos experiment design, or the policy rules being evaluated. Methodology is durable — the same process runs repeatedly over time and can be reviewed for rigour and coverage independently of any single execution.

- **Result** — the point-in-time output from a specific execution of that methodology. A result describes *what* was observed: "247 tests passed", "p99 latency was 142ms", "consumers recovered in 45 seconds". Results are ephemeral and timestamped — each execution produces a new result, and results are subject to freshness and staleness (section 6.2.6).

Both aspects are captured on the same Evidence entity via the `methodology` and `result` sub-objects (sections 6.2.2.1 and 6.2.2.2). This keeps Evidence as a single concept while making the distinction between process and output explicit. The `quality_score` field reflects methodology rigour; freshness and staleness are driven by the result's `timestamp`.

Evidence is **not** a Falsifier (section 6.3). Evidence is raw data; a Falsifier interprets that data to determine if a claim's conditions are violated. Evidence is **not** Confidence (section 6.4). Evidence contributes to confidence, but confidence is a derived score, not a signal itself.

#### 6.2.2 Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `evidence_id` | `string` | REQUIRED | Unique identifier for this evidence entity. Format is implementation-defined (e.g. UUID, URN, slug). |
| `type` | `string` | REQUIRED | Evidence category. MUST be one of the well-known types defined in section 6.2.3, or an implementation-defined extension. |
| `reference` | `object` | REQUIRED | Machine-referencable location of the evidence data. See section 6.2.4. |
| `description` | `string` | REQUIRED | Human-readable summary of what this evidence captures and how it relates to claims. |
| `timestamp` | `string` (ISO 8601) | REQUIRED | The point in time this evidence was generated or captured. Used to determine freshness and staleness of the result. |
| `methodology` | `object` | RECOMMENDED | Describes the repeatable process that produces this evidence. See section 6.2.2.1. |
| `result` | `object` | OPTIONAL | Structured outcome from the most recent execution of the methodology. See section 6.2.2.2. |
| `claim_ids` | `list[string]` | OPTIONAL | List of Claim IDs this evidence is linked to. MAY be empty (unattached evidence). |
| `metadata` | `object` | OPTIONAL | Arbitrary key-value pairs providing additional context. See section 6.2.5. |
| `freshness_window` | `string` (ISO 8601 duration) | RECOMMENDED | Maximum age before this evidence is considered stale. Staleness is determined by the result's `timestamp`, not the methodology. See section 6.2.6. |
| `quality_score` | `number` (0.0–1.0) | OPTIONAL | Rigour and reliability of the evidence methodology. Higher scores indicate more rigorous processes (e.g. a chaos experiment scores higher than a unit test for resilience claims). This score reflects the quality of the *methodology*, not the outcome of any single result. |

Implementations MAY extend this schema with additional fields, but MUST NOT remove or redefine the semantics of REQUIRED fields.

##### 6.2.2.1 Methodology

The `methodology` sub-object describes the repeatable process that produces this evidence. It captures *how* evidence is obtained — independently of any single execution or result.

| Field | Type | Required | Description |
|---|---|---|---|
| `description` | `string` | REQUIRED | Human-readable description of the process, procedure, or workflow. Should explain what the methodology does and what it exercises. |
| `tooling` | `string` | OPTIONAL | The tool, platform, or system that executes the methodology (e.g. `"pytest"`, `"chaos-mesh"`, `"opa"`, `"prometheus"`). |
| `frequency` | `string` (ISO 8601 duration) | OPTIONAL | How often the methodology is executed (e.g. `"P1D"` for daily, `"PT1H"` for hourly). Omit for event-driven or ad-hoc execution. |
| `coverage` | `string` | OPTIONAL | What the methodology covers or exercises, expressed as a human-readable scope (e.g. `"All API endpoints"`, `"Kafka consumer failover path"`, `"Production traffic in us-east-1"`). |

**Methodology and quality_score:**

The `quality_score` field on the Evidence entity (section 6.2.2) reflects methodology rigour. A well-designed chaos experiment that exercises realistic failure modes scores higher than a basic smoke test, regardless of whether the last run passed or failed. When assessing `quality_score`, consider:

- **Rigour** — does the methodology exercise realistic conditions?
- **Coverage** — does the methodology cover the behaviour asserted by the linked claims?
- **Automation** — is the methodology automated and repeatable, or manual and ad-hoc?
- **Independence** — does the methodology test in an environment representative of production?

Methodology does **not** decay over time. A well-designed test suite remains valid until the system under test changes. However, methodology MAY become outdated if the system evolves and the methodology is not updated — this is a human judgement, not a freshness calculation.

##### 6.2.2.2 Result

The `result` sub-object captures the structured outcome from the most recent execution of the methodology. It describes *what* was observed at a specific point in time.

| Field | Type | Required | Description |
|---|---|---|---|
| `outcome` | `string` (enum) | REQUIRED | The outcome of the execution. One of: `pass`, `fail`, `inconclusive`. See outcome semantics below. |
| `summary` | `string` | OPTIONAL | Human-readable summary of the result (e.g. `"247 tests passed, 0 failed"`, `"p99 latency 142ms, within threshold"`). |
| `value` | `number` or `string` | OPTIONAL | The measured or observed value, if the methodology produces a quantifiable output (e.g. `142`, `"0.03%"`, `45`). |
| `unit` | `string` | OPTIONAL | Unit of the measured value (e.g. `"milliseconds"`, `"percent"`, `"seconds"`). SHOULD be present when `value` is set. |
| `run_id` | `string` | OPTIONAL | Identifier for the specific execution run (e.g. CI build number, experiment ID, query execution ID). Enables tracing back to the source system. |

**Outcome semantics:**

| Outcome | Meaning |
|---|---|
| `pass` | The methodology executed successfully and the observed result supports the linked claim(s). |
| `fail` | The methodology executed successfully but the observed result contradicts or weakens the linked claim(s). |
| `inconclusive` | The methodology could not produce a definitive result (e.g. insufficient data, transient error, environment issue). The result neither supports nor weakens the claim. |

**Result and freshness:**

The Evidence entity's `timestamp` field records when the result was produced. Freshness and staleness (section 6.2.6) are determined by this timestamp — they measure the age of the *result*, not the age of the methodology. A methodology that has not been executed recently produces stale evidence; re-executing the methodology produces a fresh result with an updated `timestamp`.

**Result immutability:**

Results are immutable once recorded (consistent with section 6.2.7). When a methodology is re-executed, a new Evidence entity SHOULD be created with a new `timestamp` and `result`, rather than overwriting the previous result. This preserves the historical record of evidence over time.

#### 6.2.3 Evidence Types

Evidence is categorized by **type** to indicate the kind of signal it represents. The following well-known evidence types are defined:

| Type | Description | Example Reference |
|---|---|---|
| `ci_log` | Output from continuous integration pipelines or build systems | Build log artifact URL, test result JSON |
| `deployment_metric` | Metrics captured during or after a deployment | Deployment success rate query, rollback duration timeseries |
| `observability_metric` | Runtime telemetry from monitoring or observability platforms | Prometheus query, Datadog dashboard URL |
| `sbom` | Software Bill of Materials or dependency manifest | SPDX JSON file, package-lock.json URI |
| `policy_output` | Results from policy-as-code evaluation (e.g. OPA, Kyverno) | OPA decision log, policy violation report JSON |
| `chaos_result` | Outcome of chaos engineering experiments | Chaos experiment report, fault injection results |
| `security_scan` | Results from security scanning tools (SAST, DAST, container scanning) | Snyk report JSON, vulnerability scan output |
| `audit_log` | Structured audit or compliance logs | Cloud provider audit log query, access log filter |
| `incident_report` | Structured incident postmortem or event timeline | Incident ID, postmortem document URI |
| `manual_test` | Results from manual testing or validation procedures | Test case execution record, sign-off form URI |
| `slo_status` | Current SLO compliance data — target vs actual performance against defined objectives | SLO dashboard export, Datadog SLO API response |
| `telemetry_coverage` | Assessment of telemetry completeness — whether logs, metrics, and traces are configured | OpenTelemetry coverage report, telemetry audit output |
| `alert_metric` | Alert hygiene and quality metrics — signal-to-noise ratio, MTTA, false positive rate | PagerDuty analytics export, alert audit report |
| `change_assessment` | Change risk analysis output — blast radius, migration complexity, rollback feasibility | PR risk score, deployment plan metadata |
| `service_catalog` | Service ownership and dependency data — who owns what, what depends on what | Backstage catalog export, service registry dump |

Implementations MAY define additional evidence types beyond this list. Custom types SHOULD be namespaced to avoid collisions (e.g. `acme.corp/custom-evidence-type`). Implementations SHOULD document the semantics and expected reference format for any custom types.

**Type selection guidance:**

- Evidence type SHOULD reflect the **nature of the signal**, not the tool that produced it. For example, a Prometheus metric is `observability_metric`, not `prometheus_output`.
- When a piece of evidence spans multiple categories, choose the type that best reflects its **primary use** in relation to the claims it supports.

#### 6.2.4 Reference

The `reference` field is a structured object that defines how to retrieve the evidence data. It MUST contain enough information for a machine to fetch or query the evidence programmatically. The reference object has the following fields:

| Field | Type | Required | Description |
|---|---|---|---|
| `uri` | `string` | REQUIRED | A URI or URL pointing to the evidence. Supported schemes: `http`, `https`, `file`, `s3`, `gs`, or implementation-defined custom schemes. |
| `format` | `string` | RECOMMENDED | MIME type or format identifier of the evidence data (e.g. `application/json`, `text/plain`, `application/x-yaml`). |
| `checksum` | `string` | OPTIONAL | Content hash (e.g. SHA-256) to verify evidence integrity. Format: `<algorithm>:<hash>`. |
| `authentication` | `object` | OPTIONAL | Hints about how to authenticate to retrieve the evidence. See section 6.2.4.1. |

**URI requirements (principle 5.4):**

- The URI MUST be resolvable by a machine. If the evidence is behind authentication or network access controls, the `authentication` field SHOULD provide sufficient hints for authorized systems to retrieve it.
- Implementations MAY support relative URIs if they define a base URI resolution mechanism.
- Dynamic queries (e.g. Prometheus PromQL, SQL queries) SHOULD be encoded in the URI using query parameters or a custom URI scheme.

**Examples of valid references:**

1. **HTTP artifact:**
   ```yaml
   reference:
     uri: "https://ci.example.com/builds/1234/logs/test-results.json"
     format: "application/json"
     checksum: "sha256:abc123..."
   ```

2. **Prometheus query:**
   ```yaml
   reference:
     uri: "promql://prometheus.example.com/api/v1/query?query=rate(http_requests_total[5m])"
     format: "application/json"
   ```

3. **Local file:**
   ```yaml
   reference:
     uri: "file:///var/rave/evidence/sbom-20260215.json"
     format: "application/json"
   ```

4. **Cloud storage:**
   ```yaml
   reference:
     uri: "s3://rave-evidence/chaos-results/exp-001.json"
     format: "application/json"
     authentication:
       method: "iam_role"
   ```

##### 6.2.4.1 Authentication

The optional `authentication` object provides hints about how to authenticate when fetching evidence. It is **not** a credential store — implementations MUST NOT include secrets directly in evidence references. Instead, `authentication` describes the mechanism required.

| Field | Type | Required | Description |
|---|---|---|---|
| `method` | `string` | REQUIRED | Authentication method required (e.g. `bearer_token`, `iam_role`, `api_key`, `mtls`, `none`). |
| `hint` | `string` | OPTIONAL | Additional context to help locate credentials (e.g. secret name, role ARN). |

**Example:**

```yaml
authentication:
  method: "bearer_token"
  hint: "CI_TOKEN secret in vault"
```

Implementations SHOULD document how they resolve authentication hints to actual credentials.

#### 6.2.5 Metadata

The `metadata` field is an optional key-value object for attaching arbitrary context to evidence. Metadata is **not** standardized — implementations and users MAY define their own metadata keys.

Common metadata examples:

- `source_system`: Name of the tool or platform that generated the evidence (e.g. `jenkins`, `github-actions`, `datadog`)
- `region`: Geographic region where the evidence was captured
- `environment`: Deployment environment (e.g. `production`, `staging`)
- `version`: Version of the tool or schema that produced the evidence

Metadata MUST NOT affect the semantics of the evidence itself. It is supplementary context only.

#### 6.2.6 Freshness and Staleness

Evidence freshness is the measure of how current the evidence *result* is relative to the present moment. Stale evidence loses its value over time and SHOULD NOT indefinitely sustain confidence in a claim (principle 5.6).

**Freshness applies to results, not methodology.** The `timestamp` field on an Evidence entity records when the result was produced (section 6.2.2.2). Freshness is determined by this timestamp — it measures the age of the most recent result, not the age of the methodology that produced it. A well-designed methodology (section 6.2.2.1) does not become stale; only its results do. To refresh stale evidence, the methodology must be re-executed to produce a new result with an updated `timestamp`.

**Freshness Window:**

The `freshness_window` field defines the maximum age after which evidence is considered stale. It is expressed as an ISO 8601 duration (e.g. `P1D` for 1 day, `PT4H` for 4 hours, `P7D` for 7 days).

When `freshness_window` is present:

- Evidence is **fresh** if `(current_time - timestamp) < freshness_window`
- Evidence is **stale** if `(current_time - timestamp) >= freshness_window`

**Rules:**

- Evidence that is **stale** SHOULD contribute reduced or zero weight to confidence scoring.
- Claims that depend solely on stale evidence SHOULD have their confidence decay applied more aggressively.
- Evidence without a `freshness_window` is assumed to have indefinite freshness. This SHOULD only be used for immutable historical evidence (e.g. an SBOM for a specific release version) or evidence types where freshness does not apply.
- The `methodology.frequency` field (section 6.2.2.1), when set, indicates how often the methodology is expected to produce new results. When `frequency` exceeds `freshness_window`, evidence will predictably go stale between executions — implementations SHOULD surface this as a configuration warning.

**Freshness window defaults (guidance):**

| Evidence Type | Recommended Freshness Window |
|---|---|
| `ci_log` | P1D–P7D (1–7 days) |
| `deployment_metric` | P1D–P3D (1–3 days) |
| `observability_metric` | PT1H–PT24H (1–24 hours) |
| `sbom` | Indefinite (if tied to a specific release) |
| `policy_output` | P1D (1 day) |
| `chaos_result` | P7D–P30D (7–30 days) |
| `security_scan` | P7D (7 days) |
| `audit_log` | PT24H–P7D (24 hours–7 days) |
| `incident_report` | Indefinite (historical record) |
| `manual_test` | P7D–P30D (7–30 days) |
| `slo_status` | PT1H–PT24H (1–24 hours) |
| `telemetry_coverage` | P1D–P7D (1–7 days) |
| `alert_metric` | P1D–P7D (1–7 days) |
| `change_assessment` | PT1H (per change, expires quickly) |
| `service_catalog` | P7D–P30D (weekly to monthly) |

Implementations MAY define their own defaults. Claim owners SHOULD explicitly set `freshness_window` when the default is inappropriate for their use case.

#### 6.2.7 Evidence Lifecycle

Evidence does **not** have an explicit lifecycle state machine like Claims. Evidence is assumed to be immutable once recorded. However, implementations MAY support:

- **Archival** — moving old evidence to cheaper storage while retaining references
- **Expiry** — automatically purging evidence after a retention period (e.g. for compliance or cost reasons)

When evidence is archived or expired:

- Its `evidence_id` SHOULD remain referencable from Claims, but the `reference.uri` MAY become unavailable.
- Implementations SHOULD mark such evidence as "unavailable" rather than silently failing when a Claim attempts to fetch it.
- Claims that depend on unavailable evidence SHOULD have their confidence reduced, as they can no longer be validated.

#### 6.2.8 Unattached Evidence

Evidence with an empty or absent `claim_ids` list is **unattached**. Unattached evidence represents a gap in the reliability model — it indicates that an observable signal exists, but no claim gives it meaning.

**Why unattached evidence matters:**

- It reveals blind spots: signals are being generated but not linked to reliability assertions.
- It indicates opportunities to create new claims or expand the scope of existing claims.
- It can be a leading indicator that a system is generating data that should inform reliability but is currently ignored.

Implementations SHOULD surface unattached evidence in dashboards, reports, or CI checks to encourage teams to either:

- Create a Claim that uses the evidence, or
- Acknowledge that the evidence is not relevant to any current reliability assertions

#### 6.2.9 Examples

**Example 1 — CI test results (fresh evidence):**

```yaml
evidence_id: "evidence-ci-test-results-001"
type: "ci_log"
reference:
  uri: "https://github.com/example/repo/actions/runs/12345/artifacts/test-results.json"
  format: "application/json"
  checksum: "sha256:d2a84f4b8b650937ec8f73cd8be2c74add5a911ba64df27458ed8229da804a26"
  authentication:
    method: "bearer_token"
    hint: "GITHUB_TOKEN"
description: "Unit and integration test results for payments-api v2.3.1 deployment"
timestamp: "2026-02-16T14:22:00Z"
methodology:
  description: "Run full unit and integration test suite against payments-api, including rollback scenario tests and API contract validation"
  tooling: "pytest"
  frequency: "P1D"
  coverage: "All API endpoints, rollback paths, and database migration compatibility"
result:
  outcome: "pass"
  summary: "247 tests passed, 0 failed, 0 skipped"
  run_id: "github-actions-run-12345"
claim_ids:
  - "claim-deploy-rollback-001"
  - "claim-payments-api-uptime-001"
metadata:
  source_system: "github-actions"
  environment: "staging"
  commit_sha: "a1b2c3d4"
freshness_window: "P7D"
quality_score: 0.95
```

**Example 2 — Prometheus metric (observability evidence):**

```yaml
evidence_id: "evidence-prom-p99-latency-002"
type: "observability_metric"
reference:
  uri: "promql://prometheus.prod.example.com/api/v1/query_range?query=histogram_quantile(0.99,rate(http_request_duration_seconds_bucket[5m]))&start=2026-02-16T10:00:00Z&end=2026-02-16T18:00:00Z"
  format: "application/json"
description: "P99 latency for payment gateway over 8-hour window"
timestamp: "2026-02-16T18:00:00Z"
methodology:
  description: "Query P99 latency from Prometheus using histogram_quantile over a rolling 8-hour window of production traffic"
  tooling: "prometheus"
  frequency: "PT1H"
  coverage: "Production traffic in us-east-1 for the payment gateway service"
result:
  outcome: "pass"
  summary: "P99 latency 142ms, within 200ms threshold"
  value: 142
  unit: "milliseconds"
claim_ids:
  - "claim-payment-gateway-latency-001"
metadata:
  source_system: "prometheus"
  region: "us-east-1"
  environment: "production"
freshness_window: "PT12H"
quality_score: 0.90
```

**Example 3 — SBOM (immutable evidence):**

```yaml
evidence_id: "evidence-sbom-payments-v2.3.1"
type: "sbom"
reference:
  uri: "https://artifacts.example.com/sboms/payments-api-v2.3.1.spdx.json"
  format: "application/json"
  checksum: "sha256:e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6"
description: "Software Bill of Materials for payments-api v2.3.1"
timestamp: "2026-02-10T08:00:00Z"
methodology:
  description: "Generate SPDX SBOM from the release build artefact using Syft, capturing all direct and transitive dependencies"
  tooling: "syft"
  coverage: "All dependencies of payments-api v2.3.1 release build"
  # No frequency — SBOM generation is event-driven (once per release)
result:
  outcome: "pass"
  summary: "SBOM generated successfully, 142 packages catalogued, 0 critical vulnerabilities"
  value: 142
  unit: "packages"
  run_id: "release-v2.3.1-sbom"
claim_ids:
  - "claim-no-critical-vulns-001"
metadata:
  source_system: "syft"
  release_version: "v2.3.1"
# No freshness_window — SBOM is tied to a specific release and does not decay
quality_score: 1.0
```

**Example 4 — Chaos experiment result (unattached evidence):**

```yaml
evidence_id: "evidence-chaos-kafka-partition-loss-003"
type: "chaos_result"
reference:
  uri: "file:///var/rave/chaos/experiments/kafka-partition-loss-2026-02-15.json"
  format: "application/json"
description: "Chaos experiment: simulated loss of 2 Kafka partitions for 5 minutes"
timestamp: "2026-02-15T16:45:00Z"
methodology:
  description: "Simulate loss of 2 out of 6 Kafka partitions for the order-events topic under sustained producer load (500 msg/s), observe consumer group recovery time and message loss"
  tooling: "chaos-mesh"
  coverage: "Kafka consumer failover path for order-events topic in staging"
  # No frequency — chaos experiments are run ad-hoc
result:
  outcome: "pass"
  summary: "Consumer group recovered in 45 seconds, 0 messages lost, 12 messages delayed >5s"
  value: 45
  unit: "seconds"
  run_id: "exp-kafka-003"
claim_ids: []  # Unattached — no claim currently references this evidence
metadata:
  source_system: "chaos-mesh"
  environment: "staging"
  experiment_id: "exp-kafka-003"
freshness_window: "P30D"
quality_score: 0.85
```

**Example 5 — Policy evaluation result:**

```yaml
evidence_id: "evidence-opa-deployment-policy-004"
type: "policy_output"
reference:
  uri: "https://policy.example.com/api/v1/evaluations/deploy-001"
  format: "application/json"
  authentication:
    method: "mtls"
    hint: "rave-service-cert"
description: "OPA evaluation result for deployment-to-production policy"
timestamp: "2026-02-16T12:30:00Z"
methodology:
  description: "Evaluate OPA policy bundle against deployment request metadata, checking approval requirements, change-freeze windows, and environment promotion rules"
  tooling: "open-policy-agent"
  frequency: "P1D"
  coverage: "All deployment-to-production requests, 3 policy rules (approval, freeze window, promotion)"
result:
  outcome: "pass"
  summary: "3/3 policy rules passed: approval present, no freeze window active, promotion chain valid"
  run_id: "deploy-001"
claim_ids:
  - "claim-deploy-requires-approval-001"
  - "claim-no-deploy-on-friday-001"
metadata:
  source_system: "open-policy-agent"
  policy_version: "v2.1.0"
  environment: "production"
freshness_window: "P1D"
quality_score: 0.92
```

### 6.3 Falsifier

A **Falsifier** is a condition that, when met, invalidates or reduces confidence in a Claim (principle 5.2). Falsifiers are what make claims genuinely falsifiable — they define observable conditions under which a claim's assertion does not hold. Without falsifiers, a claim is an aspiration rather than an engineering statement.

Falsifiers exist at two levels in RAVE:

1. **Falsification signals** — human-readable declarations attached directly to Claims (section 6.1.2, `falsification_signals` field). These are prose statements like "Rollback duration exceeds 5 minutes in any production deployment."

2. **Falsifier entities** — structured, machine-evaluable conditions that reference Evidence and define precise evaluation logic. This section defines the Falsifier entity model.

Falsification signals on Claims provide human readability and intent. Falsifier entities provide the automation layer — they connect to Evidence sources and evaluate whether the falsification condition has been met.

#### 6.3.1 Definition

A Falsifier is a structured condition that:

- **References** one or more Evidence sources (section 6.2)
- **Defines** a machine-evaluable predicate or threshold
- **Evaluates** to a boolean outcome: the condition is either met (claim is falsified) or not met (claim is upheld)
- **Impacts** the confidence score of one or more Claims (section 6.4) when the condition is met

Falsifiers are **not** the same as Evidence. Evidence is raw data; a Falsifier interprets that data to determine if a claim's conditions are violated. Falsifiers are **not** Confidence scores — they are inputs to confidence scoring.

Falsifiers have a many-to-many relationship with Claims: a single Falsifier MAY affect multiple claims, and a single Claim MAY be evaluated by multiple Falsifiers. This design reflects the reality that operational signals often have implications for multiple reliability assertions.

#### 6.3.2 Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `falsifier_id` | `string` | REQUIRED | Unique identifier for this falsifier. Format is implementation-defined (e.g. UUID, URN, slug). |
| `name` | `string` | REQUIRED | Human-readable name of the falsifier. Should concisely describe the condition. |
| `description` | `string` | REQUIRED | Detailed explanation of what this falsifier detects and why it matters. |
| `condition` | `object` | REQUIRED | Machine-evaluable predicate defining when the falsifier is triggered. See section 6.3.4. |
| `evidence_ids` | `list[string]` | REQUIRED (>=1) | References to Evidence entities used to evaluate this falsifier. |
| `claim_ids` | `list[string]` | OPTIONAL | List of Claim IDs affected by this falsifier. MAY be empty if the falsifier is defined before being linked to claims. |
| `last_evaluated` | `string` (ISO 8601) | OPTIONAL | Timestamp of the most recent evaluation of this falsifier. |
| `last_triggered` | `string` (ISO 8601) | OPTIONAL | Timestamp of when this falsifier was most recently triggered (condition met). |
| `metadata` | `object` | OPTIONAL | Arbitrary key-value pairs providing additional context. |

Implementations MAY extend this schema with additional fields, but MUST NOT remove or redefine the semantics of REQUIRED fields.

#### 6.3.3 Triggered Behaviour

Falsifiers are **binary**. A falsifier's condition is either met (the claim is disproved) or not met (the claim is upheld). There is no partial or graduated falsification.

When a falsifier is triggered (condition is met):

- The affected Claim(s) MUST transition to `contradicted` status (section 6.1.6).
- The Claim's `confidence_score` MUST be set to 0.0.
- The falsifier represents a definitive contradiction — the claim's assertion has been proven false under the current conditions.
- A Claim contradicted by a falsifier MAY transition back to `active` only after the claim has been revised (its `statement`, `assumptions`, or `falsification_signals` have been updated).

**Design guidance:**

- A falsifier SHOULD define a condition that genuinely disproves the claim — an observable threshold or state that, when reached, means the claim's assertion does not hold.
- Signals that weaken confidence without disproving the claim (e.g. latency approaching but not exceeding an SLO) are not falsifiers. They are captured by evidence results and reflected in the confidence model through evidence freshness, quality, and decay (section 6.4).
- When defining falsifier thresholds, choose values that represent genuine contradictions, not early warnings. For example, if a claim asserts "99.9% availability", the falsifier threshold should be availability < 99.9%, not availability < 99.95%.

#### 6.3.4 Condition

The `condition` field is a structured object that defines the machine-evaluable predicate for when the falsifier is triggered. It MUST contain enough information for a machine to fetch the referenced Evidence and evaluate whether the condition is met.

The condition object has the following fields:

| Field | Type | Required | Description |
|---|---|---|---|
| `type` | `string` | REQUIRED | The type of condition. Defines the evaluation semantics. See section 6.3.4.1. |
| `parameters` | `object` | REQUIRED | Type-specific parameters defining the condition. Structure depends on `type`. |
| `evaluation_window` | `string` (ISO 8601 duration) | OPTIONAL | Time window over which the condition is evaluated (e.g. `PT1H` for 1 hour, `P1D` for 1 day). If omitted, the condition is evaluated against the most recent evidence data. |

##### 6.3.4.1 Condition Types

The following well-known condition types are defined. Implementations MAY define additional types but SHOULD document their semantics.

| Type | Description | Use Case |
|---|---|---|
| `threshold` | Numeric comparison against a threshold value. | Latency > 500ms, error rate > 1%, rollback duration > 5 minutes. |
| `boolean` | Simple true/false evaluation of evidence data. | Deployment succeeded = false, health check passing = false. |
| `regex` | Pattern matching against string evidence. | Log message contains "FATAL", commit message matches banned pattern. |
| `absence` | Evidence expected but not present. | Required test evidence missing, SBOM not found for release. |
| `staleness` | Evidence exists but is stale beyond acceptable threshold. | Security scan older than 7 days, last deployment evidence > 30 days old. |
| `composite` | Logical combination of multiple sub-conditions. | (latency > 500ms AND error_rate > 1%) OR (replicas_available < 2). |

##### 6.3.4.2 Threshold Condition

The `threshold` condition type evaluates a numeric value from evidence against a comparison operator and threshold value.

**Parameters:**

| Field | Type | Required | Description |
|---|---|---|---|
| `metric` | `string` | REQUIRED | The metric path or query to extract from the evidence data. Format depends on evidence type (e.g. JSONPath, PromQL result field). |
| `operator` | `string` (enum) | REQUIRED | Comparison operator. One of: `>`, `>=`, `<`, `<=`, `==`, `!=`. |
| `threshold` | `number` | REQUIRED | The threshold value to compare against. |
| `unit` | `string` | OPTIONAL | Unit of measurement for human readability (e.g. `"seconds"`, `"percent"`, `"count"`). |

**Example:**

```yaml
condition:
  type: "threshold"
  parameters:
    metric: "$.rollback_duration_seconds"
    operator: ">"
    threshold: 300
    unit: "seconds"
  evaluation_window: "P7D"
```

This condition is met if any rollback event in the past 7 days had a duration exceeding 300 seconds.

##### 6.3.4.3 Boolean Condition

The `boolean` condition type evaluates a true/false value from evidence.

**Parameters:**

| Field | Type | Required | Description |
|---|---|---|---|
| `field` | `string` | REQUIRED | The boolean field path to extract from the evidence data. |
| `expected` | `boolean` | REQUIRED | The expected boolean value. Condition is met if the actual value does NOT match the expected value. |

**Example:**

```yaml
condition:
  type: "boolean"
  parameters:
    field: "$.deployment.success"
    expected: true
```

This condition is met if `deployment.success` is `false` (indicating deployment failure).

##### 6.3.4.4 Regex Condition

The `regex` condition type matches a string value from evidence against a regular expression pattern.

**Parameters:**

| Field | Type | Required | Description |
|---|---|---|---|
| `field` | `string` | REQUIRED | The string field path to extract from the evidence data. |
| `pattern` | `string` | REQUIRED | Regular expression pattern. Condition is met if the pattern matches. |
| `flags` | `string` | OPTIONAL | Regex flags (e.g. `"i"` for case-insensitive). Format is implementation-defined. |

**Example:**

```yaml
condition:
  type: "regex"
  parameters:
    field: "$.log_messages[*]"
    pattern: "FATAL|CRITICAL|PANIC"
    flags: "i"
```

This condition is met if any log message contains "FATAL", "CRITICAL", or "PANIC" (case-insensitive).

##### 6.3.4.5 Absence Condition

The `absence` condition type is met when expected evidence is missing or unavailable.

**Parameters:**

| Field | Type | Required | Description |
|---|---|---|---|
| `required_evidence_types` | `list[string]` | OPTIONAL | List of evidence types that must be present. If any are missing, condition is met. |
| `grace_period` | `string` (ISO 8601 duration) | OPTIONAL | How long to wait before considering evidence absent. Defaults to immediate. |

**Example:**

```yaml
condition:
  type: "absence"
  parameters:
    required_evidence_types:
      - "ci_log"
      - "security_scan"
    grace_period: "PT1H"
```

This condition is met if CI logs or security scan evidence is missing and has been expected for more than 1 hour.

##### 6.3.4.6 Staleness Condition

The `staleness` condition type is met when evidence exists but is older than acceptable.

**Parameters:**

| Field | Type | Required | Description |
|---|---|---|---|
| `max_age` | `string` (ISO 8601 duration) | REQUIRED | Maximum acceptable age of evidence. |

**Example:**

```yaml
condition:
  type: "staleness"
  parameters:
    max_age: "P7D"
```

This condition is met if the referenced evidence is older than 7 days.

##### 6.3.4.7 Composite Condition

The `composite` condition type combines multiple sub-conditions using logical operators.

**Parameters:**

| Field | Type | Required | Description |
|---|---|---|---|
| `operator` | `string` (enum) | REQUIRED | Logical operator. One of: `AND`, `OR`, `NOT`. |
| `sub_conditions` | `list[condition]` | REQUIRED | List of condition objects. Each sub-condition follows the same structure as a top-level condition. |

**Example:**

```yaml
condition:
  type: "composite"
  parameters:
    operator: "OR"
    sub_conditions:
      - type: "threshold"
        parameters:
          metric: "$.latency_p99_ms"
          operator: ">"
          threshold: 500
      - type: "threshold"
        parameters:
          metric: "$.error_rate_percent"
          operator: ">"
          threshold: 1.0
```

This condition is met if either P99 latency exceeds 500ms OR error rate exceeds 1%.

#### 6.3.5 Evaluation Semantics

Falsifiers are evaluated by fetching their referenced Evidence and applying the `condition` logic. The evaluation process is defined as follows:

1. **Fetch Evidence.** Retrieve all Evidence entities referenced by `evidence_ids`. If any required evidence is unavailable, treat this according to the falsifier's condition type (e.g. an `absence` condition would be met; a `threshold` condition may fail to evaluate).

2. **Extract Data.** Parse the evidence data according to its `format` and extract the field(s) required by the condition's `parameters`.

3. **Apply Condition Logic.** Evaluate the condition predicate using the extracted data and the parameters. The condition evaluates to a boolean: `true` (condition met, falsifier triggered) or `false` (condition not met, claim upheld).

4. **Contradict Claim.** If the condition is met, set the affected Claim(s) `confidence_score` to 0.0 and transition the Claim to `contradicted` status (section 6.3.3).

5. **Update Timestamps.** Set the falsifier's `last_evaluated` timestamp to the current time. If the condition was met, also set `last_triggered` to the current time.

6. **Record Outcome.** Implementations SHOULD log or store the evaluation outcome (triggered/not triggered) for audit and debugging purposes.

**Evaluation frequency:**

- Falsifiers SHOULD be evaluated continuously or on a recurring schedule (principle 5.5).
- The evaluation frequency is implementation-defined but SHOULD be at least as frequent as the `freshness_window` of the referenced Evidence.
- On-demand evaluation (e.g. triggered by new Evidence arriving) is also permitted.

**Edge cases:**

- If evidence is unavailable and the condition is not an `absence` type, implementations SHOULD treat this as a failure to evaluate (not a triggered condition). The falsifier remains unevaluated until evidence becomes available.
- If evidence is stale beyond its `freshness_window` (section 6.2.6), implementations MAY still evaluate the falsifier but SHOULD surface a warning that the evaluation is based on stale data.

#### 6.3.6 Relationship to Claims

Falsifiers attach to Claims via the `claim_ids` field. A Falsifier MAY reference zero or more Claims. A Claim MAY be evaluated by zero or more Falsifiers.

**Orphaned falsifiers:**

A Falsifier with an empty `claim_ids` list is considered **orphaned** — it is defined but not yet linked to any claim. Orphaned falsifiers MAY still be evaluated, but their outcomes do not affect any claim's confidence. Implementations SHOULD surface orphaned falsifiers in dashboards or reports to encourage teams to either link them to claims or remove them.

**Bidirectional linkage:**

Claims reference falsifiers indirectly via the `falsification_signals` field (human-readable prose). Falsifiers reference Claims directly via the `claim_ids` field (machine-evaluable linkage). This dual representation provides:

- **Human intent** — the `falsification_signals` on Claims declare what should invalidate the claim in natural language.
- **Automation** — the `claim_ids` on Falsifiers provide the machine-readable linkage needed for automated evaluation and confidence scoring.

Teams SHOULD ensure that every falsification signal on a Claim has a corresponding Falsifier entity, and every Falsifier entity is represented in at least one Claim's `falsification_signals` field. However, this is not strictly enforced — some falsification signals MAY remain manual (human-evaluated) without a Falsifier entity.

#### 6.3.7 Examples

**Example 1 — Deployment rollback duration exceeded:**

```yaml
falsifier_id: "falsifier-rollback-duration-001"
name: "Rollback Exceeds 5 Minutes"
description: "Detects when a production deployment rollback takes longer than 5 minutes, contradicting the claim that rollbacks complete within 5 minutes."
condition:
  type: "threshold"
  parameters:
    metric: "$.rollback_duration_seconds"
    operator: ">"
    threshold: 300
    unit: "seconds"
  evaluation_window: "P7D"
evidence_ids:
  - "evidence-deploy-log-002"
claim_ids:
  - "claim-deploy-rollback-001"
last_evaluated: "2026-02-16T18:00:00Z"
last_triggered: null
metadata:
  environment: "production"
  team: "platform-team"
```

**Example 2 — Availability below claimed SLO:**

```yaml
falsifier_id: "falsifier-availability-below-slo-002"
name: "Availability Below 99.9%"
description: "Detects when the API error rate exceeds 0.1%, proving that 99.9% availability is not being met. The claim asserts 99.9% availability; an error rate above 0.1% is a direct contradiction."
condition:
  type: "threshold"
  parameters:
    metric: "$.error_rate_percent"
    operator: ">"
    threshold: 0.1
    unit: "percent"
  evaluation_window: "PT1H"
evidence_ids:
  - "evidence-prom-error-rate-003"
claim_ids:
  - "claim-payments-api-uptime-001"
last_evaluated: "2026-02-16T18:15:00Z"
last_triggered: "2026-02-16T14:30:00Z"
metadata:
  environment: "production"
```

**Example 3 — Missing required evidence (absence):**

```yaml
falsifier_id: "falsifier-missing-security-scan-003"
name: "Security Scan Missing for Release"
description: "Detects when a release is deployed to production without a security scan, contradicting the claim that all releases undergo security scanning."
condition:
  type: "absence"
  parameters:
    required_evidence_types:
      - "security_scan"
    grace_period: "PT30M"
evidence_ids:
  - "evidence-sbom-payments-v2.3.1"  # Reference to release evidence
claim_ids:
  - "claim-no-prod-without-security-scan-001"
last_evaluated: "2026-02-16T18:30:00Z"
last_triggered: null
metadata:
  environment: "production"
  policy: "security-baseline-v2"
```

**Example 4 — SLO breach with composite condition:**

```yaml
falsifier_id: "falsifier-slo-breach-004"
name: "Service SLO Breached — Latency or Availability"
description: "Detects when the service breaches its SLO: P99 latency exceeds 500ms (the SLO threshold) or fewer than 2 replicas are available (the minimum for the availability guarantee). Either condition disproves the reliability claim."
condition:
  type: "composite"
  parameters:
    operator: "OR"
    sub_conditions:
      - type: "threshold"
        parameters:
          metric: "$.latency_p99_ms"
          operator: ">"
          threshold: 500
          unit: "milliseconds"
      - type: "threshold"
        parameters:
          metric: "$.replicas_available"
          operator: "<"
          threshold: 2
          unit: "count"
  evaluation_window: "PT30M"
evidence_ids:
  - "evidence-prom-latency-004"
  - "evidence-k8s-replica-count-005"
claim_ids:
  - "claim-payments-api-uptime-001"
last_evaluated: "2026-02-16T18:45:00Z"
last_triggered: null
metadata:
  environment: "production"
  namespace: "payments"
```

**Example 5 — Required validation evidence missing beyond retention period:**

```yaml
falsifier_id: "falsifier-validation-evidence-missing-005"
name: "Validation Evidence Missing Beyond 30 Days"
description: "Detects when required deployment validation evidence has not been produced for more than 30 days. The claim asserts the system is continuously validated; the absence of any validation evidence for this period disproves that assertion."
condition:
  type: "absence"
  parameters:
    required_evidence_types:
      - "ci_log"
    grace_period: "P30D"
evidence_ids:
  - "evidence-ci-test-results-001"
claim_ids:
  - "claim-deploy-rollback-001"
last_evaluated: "2026-02-16T19:00:00Z"
last_triggered: null
metadata:
  environment: "production"
```

### 6.4 Confidence

**Confidence** is a derived, dynamic, non-binary score on a Claim that reflects the current strength of that claim. Confidence is not a standalone entity — it is a property of a Claim (design decision 6.0 #1). It is computed from evidence freshness, evidence quality, falsifier outcomes, and validation frequency. Annotations MUST NOT affect confidence (section 6.1.5).

#### 6.4.1 Definition

Confidence is the measure of how strongly a Claim's assertion is supported by current, high-quality evidence and how recently that support has been validated. It is expressed as a single numeric score on the Claim's `confidence_score` field.

Confidence is **not** a static property. It changes over time as evidence ages, new evidence arrives, falsifiers trigger, and validation occurs. In the absence of fresh supporting evidence, confidence MUST decay (principle 5.6).

Confidence is **not** binary. A claim is not simply "valid" or "invalid" — it exists on a spectrum from fully supported (1.0) to fully contradicted (0.0). This continuous model enables early detection of weakening claims before they reach contradiction.

#### 6.4.2 Score Range and Semantics

The `confidence_score` field is a continuous value in the range **[0.0, 1.0]**.

| Value | Meaning |
|---|---|
| `0.0` | No confidence. The claim's assertion does not hold under current conditions. Triggers the `active → contradicted` transition (section 6.1.6) and MAY trigger an Incident (section 6.5). |
| `1.0` | Full confidence. All supporting evidence is fresh and high-quality, no falsifiers have been triggered, and validation is recent. |
| Between | Partial confidence. The claim has some support but may have gaps — stale evidence, aging validation, or low-quality methodology. |

**Initial confidence:**

- When a Claim transitions from `draft` to `active`, the owner sets the initial `confidence_score` (section 6.1.6). This is an assertion by the owner about the claim's starting strength.
- A newly activated Claim MAY begin with any `confidence_score` in the range (0.0, 1.0]. A score of 0.0 would immediately trigger contradiction, so implementations SHOULD reject activation with `confidence_score: 0.0`.
- Claims in `draft` status have `confidence_score: 0.0` by convention. Draft claims are exempt from decay and scoring (section 6.1.6).
- After activation, the `confidence_score` is governed entirely by the computation model (section 6.4.3, 6.4.4). There is no manual override of confidence for active claims. To improve confidence, add or refresh evidence.

#### 6.4.3 Scoring Inputs

Confidence is derived from four inputs:

1. **Evidence freshness** — driven by the `freshness_window` field on each linked Evidence entity (section 6.2.6). Fresh evidence sustains confidence; stale evidence reduces it. Evidence without a `freshness_window` is treated as indefinitely fresh (section 6.2.6). Freshness measures the age of the evidence *result* (the `timestamp` field), not the age of the methodology that produced it (section 6.2.2.1).

2. **Evidence quality** — driven by the `quality_score` field on Evidence entities (section 6.2.2). Higher quality evidence contributes more to confidence. When `quality_score` is not set, implementations SHOULD treat it as 1.0 (no quality penalty). The `quality_score` reflects the rigour of the evidence *methodology* (section 6.2.2.1), not the outcome of any single result.

3. **Contradictions** — driven by Falsifier evaluation outcomes (section 6.3). When a falsifier is triggered, the Claim's confidence is set to 0.0 and the Claim transitions to `contradicted` status (section 6.3.3). Contradiction — whether triggered by a Falsifier (section 6.3.3) or set manually by an operator — always sets `confidence_score` to 0.0. There is no partial contradiction.

4. **Validation frequency** — driven by the `last_validated` timestamp on the Claim (section 6.1.2). The longer since last validation, the more confidence decays. This reflects principle 5.5 (validation SHOULD be continuous) and principle 5.6 (confidence MUST decay over time).

When a Claim has no linked evidence (no Evidence entities reference it via `claim_ids`), `F_avg` = 0.0 and `Q_avg` = 0.0. This means the confidence formula yields 0.0 — active claims immediately need evidence to sustain any confidence.

These four inputs are the **only** permitted inputs to confidence scoring. Annotations, claim metadata, and other non-evidence signals MUST NOT affect confidence.

How these inputs combine is implementation-defined. The spec defines the **semantics** (what each input means and how it directionally affects confidence) but does not mandate a specific formula. A default reference formula is provided in section 6.4.4 as informative guidance.

#### 6.4.4 Decay Model

Confidence MUST decay over time in the absence of fresh evidence (principle 5.6). This section defines the normative decay rules and provides an informative reference formula.

**Normative rules:**

- Confidence of an `active` Claim MUST decrease when `last_validated` ages beyond an implementation-defined threshold.
- Stale evidence (per section 6.2.6) SHOULD contribute reduced or zero weight to confidence.
- Claims depending solely on stale evidence SHOULD decay more aggressively than claims with a mix of fresh and stale evidence.
- Unavailable evidence (section 6.2.7) SHOULD reduce confidence, as claims depending on it can no longer be validated.
- Claims in `draft` status are exempt from decay (section 6.1.6).
- Claims in `retired` status are exempt from decay. Their `confidence_score` is frozen at the value it held when the claim was retired.
- Confidence scores MUST remain bounded to [0.0, 1.0] at all times.
- Falsifier triggers are not part of the decay model — they immediately set confidence to 0.0 (section 6.3.3).

**Default reference formula (informative):**

The following exponential decay model is provided as a reference. Implementations MAY use this formula or any alternative that satisfies the normative rules above.

Base exponential decay:

```
C(t) = C_0 × e^(−λ × Δt)
```

Where:

- `C(t)` = confidence at time `t`
- `C_0` = confidence at `last_validated`
- `λ` (lambda) = decay rate constant, implementation-defined (e.g. `ln(2) / half_life`)
- `Δt` = `current_time − last_validated`

With modifiers for evidence:

```
C(t) = C_0 × F_avg × Q_avg × e^(−λ × Δt)
```

Where:

- `F_avg` = average freshness factor across linked evidence. For each evidence entity: F is 1.0 if fresh (within `freshness_window`), 0.0 if stale (past `freshness_window`). Evidence without a `freshness_window` contributes F = 1.0 (indefinitely fresh). Range: [0.0, 1.0].
- `Q_avg` = average `quality_score` across linked evidence. Defaults to 1.0 for evidence without a `quality_score`. Range: [0.0, 1.0].

**Recommended decay rate:** λ = 0.05 per day (equivalent to a half-life of approximately 14 days). This is a global default — implementations SHOULD allow per-claim override of λ for claims that decay faster or slower than average.

**Notes on the reference formula:**

- The formula is exponential, meaning confidence decays smoothly and never reaches exactly 0.0 through decay alone. Implementations SHOULD define a floor (e.g. confidence below 0.01 is treated as 0.0) or rely on falsifier triggers for the final transition to contradiction.
- Implementations that choose a different formula MUST still satisfy all normative rules in this section.
- When a Claim has no linked Evidence, `F_avg` and `Q_avg` are both 0.0, yielding `C(t) = 0.0` regardless of other factors. This is by design — it enforces the principle that claims without evidence have no confidence.
- **`C_0` is a rolling anchor, not a fixed value.** `C_0` is the confidence computed at the most recent recalculation event — it updates every time the formula is applied. `Δt` is elapsed time since that previous recalculation, not since an owner-driven revalidation. The formula is applied incrementally at each event that changes its inputs: new evidence arrives, evidence goes stale, or the owner formally revalidates. The `last_validated` field on a Claim records the most recent formal owner revalidation, which is one trigger for recalculation; it does not define the only moment at which `C_0` or `Δt` are updated.

#### 6.4.5 Confidence Recovery

Confidence can increase as well as decrease. Recovery occurs through the same inputs that drive decay:

- **Fresh evidence attached or refreshed.** When new evidence is linked to a Claim or existing evidence is refreshed (its `timestamp` updates within its `freshness_window`), `F_avg` increases, raising confidence at the next evaluation.
- **Owner re-validates.** When the Claim's `last_validated` is updated (e.g. through manual validation or an automated validation run), `Δt` resets to 0, restoring the base confidence before decay.
- **Higher-quality evidence.** Replacing low-quality evidence with higher-quality evidence increases `Q_avg`, raising confidence.

**Recovery from contradiction:**

A Claim in `contradicted` status (`confidence_score: 0.0`) cannot recover through scoring inputs alone. Recovery from contradiction requires the Claim to be revised — its `statement`, `assumptions`, or `falsification_signals` MUST be updated — and confidence restored above 0.0 (section 6.1.6). This prevents a contradicted claim from silently returning to active status without human review.

#### 6.4.6 Thresholds

The following interpretive bands are RECOMMENDED for implementations that surface confidence in dashboards, alerts, or reports:

| Band | Range | Interpretation |
|---|---|---|
| High | 0.70–1.00 | Claim is well-supported by recent, high-quality evidence. No action needed. |
| Medium | 0.40–0.69 | Claim has gaps — some evidence is stale or quality is low. Review RECOMMENDED. |
| Low | 0.10–0.39 | Claim is weakly supported. Action RECOMMENDED to refresh evidence or re-validate. |
| Critical | 0.00–0.09 | Claim is near or at contradiction. Immediate attention needed. |

These bands are **informative**, not normative. Implementations MAY define custom thresholds suited to their operational context.

The only **normative** threshold is `0.0`: a Claim MUST transition to `contradicted` when `confidence_score` reaches 0.0 (section 6.1.6).

#### 6.4.7 Examples

**Worked example: confidence changing over time**

The following example traces a single Claim's confidence through a series of events using the reference formula from section 6.4.4. Assume:

- Decay rate `λ = 0.05` per day (half-life ≈ 14 days)
- The Claim has two linked evidence entities, both initially fresh with `quality_score: 0.9`
- No falsifiers are triggered at the start

| Day | Event | C_0 | Δt (days) | F_avg | Q_avg | C(t) |
|---|---|---|---|---|---|---|
| 0 | Claim activated with initial confidence 0.85 | 0.85 | 0 | 1.0 | 0.9 | **0.77** |
| 3 | Evidence refreshed, claim re-validated | 0.77 | 0 | 1.0 | 0.9 | **0.69** |
| 10 | No activity — 7 days since validation | 0.69 | 7 | 1.0 | 0.9 | **0.44** |
| 12 | One evidence entity goes stale (F_avg drops to 0.5) | 0.44 | 9 | 0.5 | 0.9 | **0.13** |
| 15 | Fresh evidence provided, re-validated | 0.13 | 0 | 1.0 | 0.9 | **0.12** |
| 20 | Evidence stable, 5 days since validation | 0.12 | 5 | 1.0 | 0.9 | **0.08** |
| 21 | Falsifier triggers | — | — | — | — | **0.00** |

**Day 0:** The Claim is activated with `confidence_score: 0.85`. Applying the reference formula: `0.85 × 1.0 × 0.9 × e^(−0.05 × 0) = 0.77` (quality factor reduces the effective confidence from the owner-set value).

**Day 3:** Evidence is refreshed and the claim is re-validated. `C_0` resets to the current confidence (0.77), `Δt` resets to 0. Confidence holds at `0.77 × 1.0 × 0.9 = 0.69`.

**Day 10:** Seven days pass without validation. Decay applies: `0.69 × 1.0 × 0.9 × e^(−0.05 × 7) ≈ 0.44`.

**Day 12:** One of two evidence entities goes stale, dropping `F_avg` to 0.5. Confidence recalculates: `0.44 × 0.5 × 0.9 × e^(−0.05 × 9) ≈ 0.13`. The stale evidence halves the freshness factor.

**Day 15:** The team responds — fresh evidence is attached, the stale evidence is refreshed, and the claim is re-validated. `F_avg` returns to 1.0, `Δt` resets. However, `C_0` is now 0.13: `0.13 × 1.0 × 0.9 × e^(0) = 0.12`. Recovery from such a low score is slow without owner intervention to reset `confidence_score`.

**Day 20:** Five days pass without validation. Decay continues: `0.12 × 1.0 × 0.9 × e^(−0.05 × 5) ≈ 0.08`. Confidence is now in the Critical band.

**Day 21:** A falsifier triggers. Confidence is immediately set to 0.0. The Claim transitions to `contradicted` (section 6.1.6) and MAY trigger an Incident (section 6.5).

This example illustrates several key behaviours: gradual decay from validation aging, acceleration from stale evidence, the difficulty of recovering from very low confidence without claim revision, and the immediate and decisive effect of a falsifier.

### 6.5 Incident

An **Incident** is a structured record of a reliability contradiction — a separate entity with its own identity and lifecycle, created when one or more Claims are contradicted by operational reality. Incidents are linked to Claims but are not a state on a Claim; they are independent entities that carry their own metadata (timeline, scope, remediation, learnings) and feed learnings back into the reliability model (principle 5.7).

Incidents are separate entities linked to Claims (design decision 6.0 #3). A single incident MAY affect multiple claims simultaneously. Incidents record contradictions — they do not cause them. The confidence dropping to 0.0 causes the `active → contradicted` transition (section 6.1.6); the Incident is a downstream record of that event.

#### 6.5.1 Definition

An Incident is a structured record that:

- **Records** the contradiction of one or more Claims
- **Tracks** the investigation and remediation lifecycle from detection through resolution
- **Captures** a timeline of events related to the contradiction
- **Documents** remediation actions and their outcomes
- **Feeds back** learnings into Claims, Falsifiers, and the broader reliability model (principle 5.7)

An Incident is **not** a Claim state — Claims have their own lifecycle (section 6.1.6). An Incident is **not** a Falsifier — Falsifiers detect conditions that reduce confidence (section 6.3); Incidents record the broader operational context when contradiction occurs. An Incident is **not** Evidence — though incidents MAY reference Evidence and MAY themselves become evidence for future claims (evidence type `incident_report`, section 6.2.3).

#### 6.5.2 Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `incident_id` | `string` | REQUIRED | Unique identifier for this incident. Format is implementation-defined (e.g. UUID, URN, slug). |
| `title` | `string` | REQUIRED | Human-readable summary of the incident. |
| `description` | `string` | REQUIRED | Detailed narrative of what happened and the impact. |
| `status` | `string` (enum) | REQUIRED | Lifecycle state of the incident. One of: `detected`, `investigating`, `mitigated`, `resolved`, `closed`. See section 6.5.3. |
| `severity` | `string` (enum) | REQUIRED | Operational impact level. One of: `critical`, `major`, `minor`. See section 6.5.4. |
| `claim_ids` | `list[string]` | REQUIRED (>=1) | References to Claim entities contradicted by this incident. An Incident MUST link to at least one Claim. |
| `triggered_at` | `string` (ISO 8601) | REQUIRED | Timestamp of when the incident was detected. |
| `resolved_at` | `string` (ISO 8601) | OPTIONAL | Timestamp of when the incident was resolved. MUST be set when `status` transitions to `resolved`. |
| `timeline` | `list[TimelineEntry]` | OPTIONAL | Structured event timeline. See section 6.5.5. |
| `remediation` | `object` | OPTIONAL | Actions taken to address the incident. See section 6.5.6. |
| `learnings` | `list[string]` | OPTIONAL | Lessons learned that feed back into claims, falsifiers, or the reliability model. See section 6.5.9. |
| `evidence_ids` | `list[string]` | OPTIONAL | References to Evidence entities linked to this incident. |
| `metadata` | `object` | OPTIONAL | Arbitrary key-value pairs providing additional context. |

Implementations MAY extend this schema with additional fields, but MUST NOT remove or redefine the semantics of REQUIRED fields.

#### 6.5.3 Lifecycle

An Incident progresses through the following lifecycle states:

| Status | Description |
|---|---|
| `detected` | Incident identified. Associated Claims have transitioned to `contradicted` (section 6.1.6). |
| `investigating` | Active investigation underway. Root cause analysis in progress. |
| `mitigated` | Immediate impact contained. Root cause may not yet be resolved. |
| `resolved` | Root cause addressed. `resolved_at` MUST be set when entering this state. |
| `closed` | Learnings captured, claims updated. Terminal state. |

**Transitions:**

```
detected → investigating      Investigation begins.
investigating → mitigated     Immediate impact contained.
investigating → resolved      Root cause addressed directly (mitigation implicit).
mitigated → resolved          Root cause addressed after mitigation.
resolved → closed             Learnings captured, claims updated.
detected → closed             False positive — incident created in error.
closed → (terminal)           No transitions out of closed.
```

**Transition rules:**

- An Incident MUST begin in the `detected` state.
- An Incident MUST set `resolved_at` when transitioning to `resolved`.
- An Incident SHOULD have `learnings` populated before transitioning to `closed`.
- The `closed` state is **terminal**. Once closed, an Incident MUST NOT transition to any other state. If a related contradiction recurs, a new Incident MUST be created.
- The shortcut from `detected` to `closed` is permitted for false positives — when an incident was created but investigation determines no actual contradiction occurred.

#### 6.5.4 Severity

Incident severity reflects the **operational impact** of the contradiction:

| Severity | Description |
|---|---|
| `critical` | Fundamental failure of core claims. Broad operational impact affecting multiple systems or users. |
| `major` | Significant contradiction of one or more claims. Moderate impact on operations or users. |
| `minor` | Localised or low-impact contradiction. Limited blast radius. |

**Severity rules:**

- Severity is set at incident creation and MAY be revised during investigation as impact becomes clearer.
- Incident severity is **independent** of the falsifier that triggered it. A falsifier contradicting a narrow claim MAY result in a `minor` incident if the operational impact is limited, while a falsifier contradicting a broad claim MAY result in a `critical` incident. Severity reflects operational impact, not the falsifier's scope.
- Implementations SHOULD define severity assignment guidance suited to their operational context.

#### 6.5.5 Timeline

The `timeline` field is an OPTIONAL ordered list of **TimelineEntry** objects that record significant events during the incident lifecycle. Timelines provide a structured narrative of what happened and when.

**TimelineEntry fields:**

| Field | Type | Required | Description |
|---|---|---|---|
| `timestamp` | `string` (ISO 8601) | REQUIRED | When the event occurred. |
| `event` | `string` | REQUIRED | Human-readable description of what happened. |
| `author` | `string` | OPTIONAL | The person or system that recorded this entry. |

**Rules:**

- Timeline entries are append-only by convention — implementations SHOULD NOT modify or delete existing entries.
- Entries SHOULD be ordered chronologically by `timestamp`.
- The first entry SHOULD correspond to the incident detection event.

#### 6.5.6 Remediation

The `remediation` field is an OPTIONAL structured object that documents the actions taken to address the incident.

**Remediation fields:**

| Field | Type | Required | Description |
|---|---|---|---|
| `actions` | `list[string]` | REQUIRED | Ordered list of remediation actions taken. |
| `owner` | `string` | OPTIONAL | Team or individual responsible for remediation. |
| `verification` | `string` | OPTIONAL | Description of how the remediation was verified. |

**Rules:**

- When `remediation` is present, `actions` MUST contain at least one entry.
- Actions SHOULD be listed in the order they were performed.
- The `verification` field SHOULD describe how the team confirmed that the remediation was effective.

#### 6.5.7 Triggering Semantics

Incidents MAY be triggered when one or more Claims transition to `contradicted` status (confidence drops to 0.0, section 6.1.6). The use of MAY rather than MUST reflects the reality that not every contradiction warrants a formal incident record — some contradictions are expected (e.g. during planned maintenance) or trivial.

**Triggering rules:**

- Incidents MAY be created automatically by tooling (e.g. when a falsifier triggers) or manually by operators.
- When a single event contradicts multiple Claims simultaneously, implementations SHOULD create a single Incident with multiple `claim_ids` rather than separate incidents per claim. This avoids fragmentation and keeps the operational context unified.
- Incidents do **not** reference Falsifiers directly. The linkage from Incident to the triggering condition is indirect: Incident → Claims → Falsifiers. This keeps Incidents focused on the operational contradiction rather than the detection mechanism.

**Impact on the confidence model:**

- Incidents do not directly affect `confidence_score`. Confidence is driven by the four scoring inputs defined in section 6.4.3 (evidence freshness, evidence quality, contradictions from falsifiers, and validation frequency).
- Incidents record the **consequence** of confidence reaching 0.0 — they are downstream of the confidence model, not an input to it.
- However, the **learnings** from an incident (section 6.5.9) MAY lead to changes that indirectly affect confidence: new falsifiers, revised assumptions, updated evidence sources, or claim retirement.

#### 6.5.8 Relationship to Claims

Incidents have a **many-to-many** relationship with Claims:

- One Incident MAY affect multiple Claims (via the `claim_ids` field).
- One Claim MAY be referenced by multiple Incidents over its lifetime.

**Design decisions:**

- **Incidents record contradictions, they do not cause them.** The `active → contradicted` transition is triggered by confidence reaching 0.0 (section 6.1.6). The Incident is created in response to that transition, not as the cause of it.
- **No `incident_ids` field on Claims.** Claims remain the aggregate root (design decision 6.0 #1). The Claim schema (section 6.1.2) is not modified by the Incident model. Bidirectional traversal (from Claim to Incidents) is derived by querying Incidents by `claim_ids`.
- **Lifecycle independence.** A Claim MAY recover from `contradicted` to `active` (section 6.1.6) while the associated Incident is still in `investigating` or `mitigated` status. Conversely, an Incident MAY be `closed` while the associated Claim remains `contradicted`. The two lifecycles are independent.

#### 6.5.9 Feedback Loop

Incidents close the loop between operational failures and the reliability model (principle 5.7). When an incident is resolved, the learnings SHOULD feed back into the reliability model through one or more of the following mechanisms:

- **Claim revision** — The contradicted Claim's `statement`, `assumptions`, or `falsification_signals` are updated to reflect the lessons learned. This is the primary recovery path from `contradicted` back to `active` (section 6.1.6).
- **New falsifiers** — New Falsifier entities (section 6.3) are created to detect the conditions that led to the incident, preventing silent recurrence.
- **Updated assumptions** — Claim assumptions are tightened or expanded to more accurately reflect the conditions under which the claim holds.
- **New claims** — The incident reveals reliability gaps that warrant entirely new Claims.
- **Evidence improvements** — New evidence sources are added, freshness windows are tightened, or evidence quality is improved.
- **Claim retirement** — The contradicted Claim is retired (section 6.1.6) if the incident reveals that the claim is no longer meaningful or achievable.

Claims revised as a result of an incident SHOULD include an Annotation (section 6.1.5) referencing the incident, providing traceability from the claim revision back to the operational event that prompted it.

#### 6.5.10 Examples

**Example 1 — Falsifier triggers single claim contradiction:**

A production deployment rollback exceeds the 5-minute threshold. The falsifier triggers (section 6.3.3), the claim's confidence drops to 0.0, the claim transitions to `contradicted`, and an incident is created.

```yaml
incident_id: "incident-rollback-timeout-001"
title: "Production rollback exceeded 5-minute threshold"
description: "Deployment v2.4.1 rollback took 8 minutes due to a database migration lock. The claim that rollbacks complete within 5 minutes was contradicted."
status: "resolved"
severity: "major"
claim_ids:
  - "claim-deploy-rollback-001"
triggered_at: "2026-02-15T14:22:00Z"
resolved_at: "2026-02-15T16:45:00Z"
timeline:
  - timestamp: "2026-02-15T14:22:00Z"
    event: "Rollback initiated for v2.4.1 deployment"
    author: "deploy-bot"
  - timestamp: "2026-02-15T14:27:00Z"
    event: "Rollback still in progress — database migration lock detected"
    author: "on-call-engineer"
  - timestamp: "2026-02-15T14:30:00Z"
    event: "Rollback completed after 8 minutes. Falsifier triggered, claim contradicted."
    author: "rave-agent"
  - timestamp: "2026-02-15T15:00:00Z"
    event: "Root cause identified: migration lock held by long-running transaction"
    author: "on-call-engineer"
  - timestamp: "2026-02-15T16:45:00Z"
    event: "Fix deployed: migration timeout reduced to 2 minutes with automatic lock release"
    author: "platform-team"
remediation:
  actions:
    - "Reduced database migration timeout to 2 minutes"
    - "Added automatic lock release on rollback initiation"
    - "Updated rollback runbook to include migration lock check"
  owner: "platform-team"
  verification: "Verified via rollback drill — rollback completed in 3m12s with migration lock present"
learnings:
  - "Database migration locks can block rollbacks — add migration lock detection to rollback pre-checks"
  - "Rollback claim assumptions should explicitly state that no long-running transactions hold migration locks"
evidence_ids:
  - "evidence-deploy-log-002"
  - "evidence-rollback-timer-001"
metadata:
  environment: "production"
  deployment_version: "v2.4.1"
```

**Example 2 — Multi-claim incident:**

A Kafka broker failure contradicts two claims simultaneously — exactly-once delivery and event processing latency. A single incident is created with both `claim_ids`.

```yaml
incident_id: "incident-kafka-broker-failure-002"
title: "Kafka broker failure contradicted delivery and latency claims"
description: "Kafka broker-3 failed, reducing the cluster below the minimum replication factor. Two claims were contradicted: exactly-once delivery (duplicate events detected) and event processing latency (consumer lag exceeded threshold)."
status: "closed"
severity: "critical"
claim_ids:
  - "claim-kafka-exactly-once-001"
  - "claim-event-processing-latency-001"
triggered_at: "2026-02-12T03:15:00Z"
resolved_at: "2026-02-12T06:30:00Z"
timeline:
  - timestamp: "2026-02-12T03:15:00Z"
    event: "Kafka broker-3 became unreachable. Replication factor dropped below 3."
    author: "monitoring-agent"
  - timestamp: "2026-02-12T03:18:00Z"
    event: "Consumer lag exceeded 10,000 messages. Latency claim contradicted."
    author: "rave-agent"
  - timestamp: "2026-02-12T03:22:00Z"
    event: "Duplicate events detected in fulfilment service. Delivery claim contradicted."
    author: "rave-agent"
  - timestamp: "2026-02-12T04:00:00Z"
    event: "Broker-3 hardware failure confirmed. Replacement node provisioning initiated."
    author: "on-call-engineer"
  - timestamp: "2026-02-12T06:30:00Z"
    event: "Replacement broker online. Replication factor restored. Consumer lag cleared."
    author: "on-call-engineer"
  - timestamp: "2026-02-13T10:00:00Z"
    event: "Learnings captured. Claims revised with updated assumptions. Incident closed."
    author: "messaging-team"
remediation:
  actions:
    - "Provisioned replacement Kafka broker"
    - "Rebalanced partitions across healthy brokers"
    - "Drained and replayed consumer lag"
    - "Added alerting for replication factor below threshold"
  owner: "messaging-team"
  verification: "Verified via chaos experiment: simulated broker loss with new alerting in place"
learnings:
  - "Kafka claims should assume a minimum of 4 brokers to tolerate single-broker failure without contradiction"
  - "Add a falsifier for replication factor dropping below the minimum required for the availability guarantee"
  - "Consumer deduplication window of 7 days was insufficient during extended broker outage — consider increasing"
evidence_ids:
  - "evidence-kafka-broker-health-005"
  - "evidence-consumer-lag-006"
metadata:
  environment: "production"
  kafka_cluster: "orders-prod"
  affected_broker: "broker-3"
```

**Example 3 — Security vulnerability incident:**

A critical CVE is discovered in a dependency, contradicting the claim that no critical vulnerabilities exist in production. The incident is resolved by patching, and learnings tighten the security scan freshness window.

```yaml
incident_id: "incident-cve-critical-003"
title: "Critical CVE in payment processing dependency"
description: "CVE-2026-1234 (CVSS 9.8) discovered in the JSON parsing library used by payments-api. The claim that no critical vulnerabilities exist in production dependencies was contradicted."
status: "closed"
severity: "critical"
claim_ids:
  - "claim-no-critical-vulns-001"
triggered_at: "2026-02-10T09:00:00Z"
resolved_at: "2026-02-10T14:30:00Z"
timeline:
  - timestamp: "2026-02-10T09:00:00Z"
    event: "Security scan detected CVE-2026-1234 (CVSS 9.8) in json-parser v3.2.1"
    author: "security-scanner"
  - timestamp: "2026-02-10T09:05:00Z"
    event: "Falsifier triggered. Claim contradicted."
    author: "rave-agent"
  - timestamp: "2026-02-10T09:30:00Z"
    event: "Severity confirmed as critical. Patched version (v3.2.2) available."
    author: "security-team"
  - timestamp: "2026-02-10T12:00:00Z"
    event: "Patched dependency deployed to staging. Tests passing."
    author: "platform-team"
  - timestamp: "2026-02-10T14:30:00Z"
    event: "Patched dependency deployed to production. Security scan confirms no critical CVEs."
    author: "platform-team"
  - timestamp: "2026-02-11T10:00:00Z"
    event: "Learnings captured. Security scan freshness window tightened. Incident closed."
    author: "security-team"
remediation:
  actions:
    - "Upgraded json-parser from v3.2.1 to v3.2.2"
    - "Deployed patched version to all environments"
    - "Ran full security scan to confirm no remaining critical CVEs"
  owner: "security-team"
  verification: "Post-deployment security scan returned zero critical findings"
learnings:
  - "Security scan freshness window of P7D was too long — tighten to P1D for production dependencies"
  - "Add a falsifier for critical (CVSS >= 9.0) CVEs that contradicts the claim that no critical vulnerabilities exist"
  - "SBOM evidence should be regenerated on every deployment, not just on release"
evidence_ids:
  - "evidence-sbom-payments-v2.3.1"
  - "evidence-security-scan-007"
metadata:
  environment: "production"
  cve_id: "CVE-2026-1234"
  cvss_score: 9.8
  affected_component: "json-parser v3.2.1"
```

### 6.6 Scope

A **Scope** is a first-class entity that defines a boundary within which Claims apply. Scopes provide identity, hierarchy, and structure to the targets that claims reference — enabling roll-up views, drill-down navigation, and scope-aware dashboards.

Claims reference Scopes via the `scope` field (section 6.1.3). Evidence and Falsifiers relate to Scopes indirectly through the Claims they are linked to. Scopes exist independently of Claims — a declared Scope with no linked Claims represents a defined boundary that has not yet been covered by reliability assertions.

#### 6.6.1 Definition

A Scope identifies a specific system boundary — a service, component, team, environment, pipeline, or other organisational unit — and optionally declares its position within a containment hierarchy. Scopes answer the question "what does this claim apply to?" with a structured, referencable identity rather than an ad-hoc string.

Scopes are **not** Claims. A Scope does not assert behaviour — it defines the target about which behaviour is asserted. Scopes are **not** Evidence — they do not support or weaken claims. Scopes are **not** organizational metadata — while team and environment scopes exist, a Scope is a reliability boundary first and foremost.

#### 6.6.2 Fields

| Field | Type | Required | Description |
|---|---|---|---|
| `type` | `string` | REQUIRED | A well-known scope type identifying the category. See section 6.6.3. |
| `target` | `string` | REQUIRED | The specific entity. MUST be non-empty. Unique within a `type`. |
| `parent` | `object` | OPTIONAL | Parent scope reference: `{ type, target }`. Omit for root scopes. See section 6.6.4. |
| `owner` | `string` or `object` | OPTIONAL | Owner of this scope. Same structure as the Claim `owner` field (section 6.1.2.1). |
| `dependencies` | `list[object]` | OPTIONAL | Scopes that this scope depends on. Each entry is a `{ type, target }` reference to another declared scope. See section 6.6.5. |
| `description` | `string` | OPTIONAL | Human-readable description of this scope. |

**Identity:** A Scope is identified by the composite key `(type, target)`. Two Scopes MUST NOT share the same `type` and `target` within a given RAVE dataset. Implementations MUST enforce this uniqueness constraint.

Implementations MAY extend this schema with additional fields, but MUST NOT remove or redefine the semantics of REQUIRED fields.

> **Informative note — ownership inheritance.** If a Claim does not set an `owner`, implementations MAY inherit ownership from the Claim's scope. This is not normative; the `owner` field on Claims is REQUIRED (section 6.1.2), so inheritance applies only when implementations choose to auto-populate `owner` during authoring or ingestion workflows.

#### 6.6.3 Well-Known Scope Types

The following well-known scope types are defined:

| Type | Description | Example `target` |
|---|---|---|
| `system` | A top-level system or product | `checkout-platform` |
| `application` | A deployable application within a system | `payments-app` |
| `service` | A deployed service or microservice | `payments-api` |
| `component` | A subsystem or library within a service | `payments-api/retry-handler` |
| `endpoint` | A specific API endpoint | `POST /v2/charges` |
| `pipeline` | A CI/CD or data pipeline | `deploy-to-prod` |
| `release` | A versioned release of an application or service | `payments-app/v2.3.1` |
| `environment` | A deployment environment | `production-eu-west-1` |
| `team` | A team boundary — scopes owned collectively | `platform-team` |

Implementations MAY define additional scope types beyond this list. Custom scope types SHOULD be namespaced to avoid collisions (e.g. `acme.corp/custom-scope-type`). Implementations SHOULD document any custom scope types and their intended semantics.

**Type selection guidance:**

- Scope type SHOULD reflect the **nature of the boundary**, not the technology that implements it.
- When a target spans multiple categories, choose the type that best reflects its **primary role** in the reliability model.

#### 6.6.4 Containment Hierarchy

Scopes MAY form a **containment tree** through optional `parent` references. Containment models the "is part of" relationship — a service is part of an application, an application is part of a system.

**Tree rules (normative):**

- A Scope MAY have at most one parent.
- The `parent` field, when present, MUST be an object with `type` and `target` fields.
- The parent MUST reference a Scope that is also declared within the same RAVE dataset (i.e. in the same file or in a co-located scope file).
- The parent graph MUST be acyclic. Following `parent` references MUST terminate at a root scope (a scope with no parent). Implementations MUST reject scope declarations that introduce cycles.
- Root scopes (no `parent`) represent top-level boundaries.

**Containment is not dependency.** Containment expresses "X is part of Y" (e.g. `service:payments-api` is part of `application:payments-app`). Dependencies between scopes (e.g. "service A depends on service B") are a separate concern modelled by the `dependencies` field (section 6.6.5), not by the containment hierarchy.

**Typical containment patterns:**

```
system
  └── application
        ├── service
        │     ├── component
        │     └── endpoint
        └── pipeline
```

```
system
  └── application
        └── release
```

Not all scope types have natural containment relationships. `environment` and `team` scopes commonly exist as root scopes, but MAY be placed within a hierarchy if the organisation's model warrants it.

#### 6.6.5 Scope Dependencies

A Scope MAY declare **dependencies** on other scopes via the `dependencies` field. Dependencies model the "A depends on B" relationship — for example, a service that calls another service, or a service that requires a pipeline to have run successfully.

**Dependency rules (normative):**

- A Scope MAY have zero or more dependencies.
- Dependencies are **directed**: "A depends on B" does not imply "B depends on A."
- Each dependency entry MUST be an object with `type` (non-empty string) and `target` (non-empty string) fields.
- Dependency references MUST resolve to a declared scope — either in the same file or in a co-located scope file. Implementations MUST reject unresolvable dependency references.
- Self-dependencies are forbidden. A Scope MUST NOT list itself (its own `type` and `target` pair) in its `dependencies`.
- Circular dependencies are permitted. A depends on B, B depends on A is allowed — this is realistic in distributed systems where services have mutual call paths. Implementations SHOULD detect and warn about cycles but MUST NOT reject them.

**Dependency semantics (informative):**

- When evaluating readiness at a scope (section 6.7), implementations SHOULD also check the readiness of that scope's dependencies. A failing dependency is a signal that the scope's operational environment may be degraded.
- A scope with a failing dependency is not necessarily unready — the dependency's health is a signal, not a gate. Whether a failing dependency blocks readiness is implementation-defined.
- Dependencies are distinct from containment (section 6.6.4). A child scope inherits claims from its parent through roll-up semantics (section 6.6.6). A scope does NOT inherit claims from its dependencies. Dependency and containment are orthogonal relationships.

**Example:**

```yaml
scopes:
  - type: "service"
    target: "payments-api"
    parent:
      type: "application"
      target: "payments-app"
    dependencies:
      - type: "service"
        target: "order-service"
      - type: "pipeline"
        target: "deploy-to-prod"
```

In this example, `service:payments-api` is *part of* `application:payments-app` (containment) and *depends on* `service:order-service` and `pipeline:deploy-to-prod` (dependency). The two relationships are independent.

#### 6.6.6 Roll-Up Semantics (Informative)

When displaying confidence at a parent scope, implementations SHOULD aggregate all Claims that target the parent scope or any of its descendants. This enables questions like "how confident are we in the checkout platform?" to be answered from the claims on individual services, components, and pipelines.

**Aggregation guidance:**

- Implementations SHOULD display the count of claims at a scope and its descendants.
- Implementations SHOULD display the average confidence across those claims.
- Implementations MAY use alternative aggregation methods (minimum, weighted average, etc.) suited to their operational context.
- Implementations SHOULD make the aggregation method visible to users.
- Clicking a parent scope in a dashboard SHOULD show claims from the parent and all descendant scopes.

Roll-up semantics are **informative** — the spec does not mandate a specific aggregation formula. The goal is to provide meaningful visibility at every level of the hierarchy.

#### 6.6.7 Auto-Creation of Scopes

Claims MAY reference scopes that have not been explicitly declared in a `.scope.yaml` file (section 7.8). This supports backward compatibility and incremental adoption — teams can start writing claims immediately without first declaring a scope hierarchy.

**Auto-creation rules:**

- When a Claim references a `(type, target)` pair that does not match any declared Scope, implementations SHOULD auto-create the scope as a **root scope** (no parent, no description).
- Auto-created scopes are fully functional — they can be targeted by claims, appear in dashboards, and participate in readiness queries.
- Auto-created scopes do NOT have containment relationships. To place a scope within a hierarchy, it MUST be explicitly declared in a scope file.
- Implementations SHOULD surface auto-created scopes to encourage teams to declare them formally with descriptions and containment relationships.

#### 6.6.8 Examples

**Example 1 — A system with nested scopes:**

```yaml
# Declared in checkout-platform.scopes.yaml
scopes:
  - type: "system"
    target: "checkout-platform"
    description: "The full checkout platform including all payment and order services"

  - type: "application"
    target: "payments-app"
    parent:
      type: "system"
      target: "checkout-platform"
    description: "Payment processing application"

  - type: "service"
    target: "payments-api"
    parent:
      type: "application"
      target: "payments-app"
    owner:
      name: "backend-team"
      team: "Backend Services"
      contact: "backend-oncall@example.com"
    description: "REST API for payment processing"

  - type: "component"
    target: "payments-api/retry-handler"
    parent:
      type: "service"
      target: "payments-api"
    description: "Retry logic for upstream payment gateway calls"

  - type: "pipeline"
    target: "deploy-to-prod"
    parent:
      type: "service"
      target: "payments-api"
    description: "CI/CD pipeline for deploying payments-api to production"
```

This produces the containment tree:

```
system:checkout-platform
  └── application:payments-app
        └── service:payments-api
              ├── component:payments-api/retry-handler
              └── pipeline:deploy-to-prod
```

**Example 2 — Root scopes (no hierarchy):**

```yaml
scopes:
  - type: "environment"
    target: "production-eu-west-1"
    description: "Primary production environment in EU West"

  - type: "team"
    target: "platform-team"
    description: "Platform engineering team"
```

**Example 3 — Release scope:**

```yaml
scopes:
  - type: "application"
    target: "payments-app"

  - type: "release"
    target: "payments-app/v2.3.1"
    parent:
      type: "application"
      target: "payments-app"
    description: "Payments app release v2.3.1"
```

### 6.7 Readiness Evaluation (Informative)

This section defines the semantics of **readiness** — the aggregate evaluation of whether all reliability claims at a scope meet a minimum confidence bar. This section is informative. Implementations MAY define their own readiness semantics, but SHOULD follow this guidance.

#### 6.7.1 Definition

Readiness answers the question: "Is this scope ready?" It is the aggregate evaluation of whether all active claims at a scope are confident and none are contradicted.

Readiness is a **derived concept** — it is computed from the claims, their confidence scores, and their statuses. It is not stored as a field on any entity. Like confidence (section 6.4), readiness is computed on demand from the current state of the claims at a scope.

Readiness provides the decision point for agents and automation. Where confidence is a property of an individual claim, readiness is a property of a scope — it rolls up individual claim assessments into a single go/no-go signal.

#### 6.7.2 Readiness Conditions

A scope is **ready** when ALL of the following are true:

1. **No contradicted claims.** No active claim at this scope has status `contradicted`.
2. **All confident.** Every active claim at this scope has `confidence_score` >= the readiness threshold.

The default threshold is **0.70** — the lower bound of the "High" confidence band from section 6.4.6. This means that by default, every active claim must be well-supported by recent, high-quality evidence for the scope to be considered ready.

The threshold is configurable per evaluation. Different operational contexts MAY require stricter or more lenient thresholds.

**Evaluation rules:**

| Condition | Effect |
|---|---|
| Active claim with confidence >= threshold | Passes |
| Active claim with confidence < threshold | **Fails** — reported as a failing claim |
| Contradicted claim | **Fails** — always reported regardless of threshold |
| Draft claim | Excluded — not yet active |
| Retired claim | Excluded — no longer relevant |
| No claims at scope | Passes — empty scope is vacuously ready |

#### 6.7.3 Readiness Response Structure

The following structure is RECOMMENDED for readiness evaluation results. Implementations MAY extend or adapt this structure, but SHOULD preserve the core fields (`ready`, `scope`, `threshold`, `summary`, `failing_claims`) to ensure interoperability between tools.

```yaml
ready: false
scope:
  type: "service"
  target: "payments-api"
threshold: 0.70
evaluated_at: "2026-02-24T10:30:00Z"
summary:
  total_claims: 5
  active_confident: 3
  active_not_confident: 1
  contradicted: 1
  draft: 0
  retired: 0
failing_claims:
  - claim_id: "claim-002"
    statement: "All API endpoints emit structured logs, metrics, and traces"
    status: "contradicted"
    confidence_score: 0.0
    owner: "platform-team"
    reason: "Contradicted — requires revision and reactivation"
  - claim_id: "claim-005"
    statement: "P99 latency below 200ms"
    status: "active"
    confidence_score: 0.55
    owner: "backend-team"
    reason: "Confidence 0.55 below threshold 0.70"
```

**Field semantics:**

| Field | Description |
|---|---|
| `ready` | Boolean. `true` if the scope passes all readiness conditions (section 6.7.2); `false` otherwise. |
| `scope` | The scope being evaluated, expressed as `{ type, target }`. |
| `threshold` | The confidence threshold used for this evaluation. |
| `evaluated_at` | ISO 8601 timestamp of when the evaluation was performed. |
| `summary` | Counts of claims by category. `total_claims` is the sum of all categories. `active_confident` counts active claims meeting the threshold. `active_not_confident` counts active claims below the threshold. |
| `failing_claims` | List of claims that caused the scope to fail readiness. Empty when `ready` is `true`. Each entry includes the claim's identity, current state, owner, and a machine-readable reason for failure. |

#### 6.7.4 Descendant Inclusion

When evaluating readiness at a scope that has children in the containment hierarchy (section 6.6.4), implementations SHOULD include claims from all descendant scopes. This means asking "is the checkout-platform ready?" evaluates claims on the platform itself plus all its child services, components, pipelines, and other descendant scopes.

This behaviour follows naturally from the roll-up semantics defined in section 6.6.6. Readiness at a parent scope is a roll-up of readiness across the entire subtree.

Implementations MAY provide an option to evaluate only the direct scope without descendants. When descendant inclusion is disabled, only claims that directly target the specified scope are evaluated. This is useful for isolating the readiness of a single scope without inheriting the state of its children.

#### 6.7.5 Agent Interaction Pattern (Informative)

Readiness is designed to serve as the decision point for AI agents and automation. The expected workflow is:

1. Agent calls a readiness evaluation tool (e.g. `rave_check_readiness`) with `scope_type` and `scope_target`.
2. If `ready: true` — agent proceeds with the planned action (push, deploy, merge, etc.).
3. If `ready: false` — agent reads `failing_claims` to understand what is blocking readiness.
4. Agent uses `owner` information on each failing claim to identify responsible parties.
5. Agent attempts to address issues within its capability — run tests, refresh evidence, update claim metadata, etc.
6. Agent calls readiness evaluation again after taking action.
7. If still not ready — agent reports the blocking claims to the human operator.

This pattern positions readiness as the "definition of done" for agent-driven workflows. Rather than hard-coding deployment checks, security gates, or test requirements into CI configuration, teams express their reliability expectations as claims and let readiness evaluation enforce them dynamically.

**Example: readiness in a project's agent instructions (e.g. AGENTS.md or CLAUDE.md):**

```markdown
## Definition of Done
Before pushing, verify scope readiness using the RAVE MCP tools:
1. Call `rave_check_readiness` for the relevant scope
2. If failing claims exist, use `rave_get_claim` to understand each failure
3. Address what you can; report what you cannot to the owner
```

The spec defines the **semantics** of readiness — what "ready" means and what the evaluation produces. The transport mechanism (MCP tools, CLI commands, REST endpoints, or library calls) is implementation-defined.

#### 6.7.6 Reason Generation

For each failing claim in a readiness evaluation, the `reason` field SHOULD provide a machine-readable explanation of why the claim failed. This enables agents and tooling to programmatically categorise and respond to failures.

RECOMMENDED reason formats:

| Failure type | Reason format |
|---|---|
| Contradicted claim | `"Contradicted — requires revision and reactivation"` |
| Low confidence | `"Confidence {score} below threshold {threshold}"` |

Where `{score}` is the claim's current `confidence_score` and `{threshold}` is the readiness threshold used for the evaluation.

Implementations MAY extend the reason format with additional detail (e.g. which evidence is stale, which falsifier triggered), but SHOULD preserve the core format for interoperability.

---

## 7. File Formats

RAVE defines YAML file formats for serialising Claims (section 6.1) and Scopes (section 6.6) for storage, exchange, and version control. This section defines the file formats — sections 6.1.2 and 6.6.2 define the data models. No field semantics are redefined here.

RAVE files are human-readable, machine-parseable, and suitable for version control. Implementations MUST support both single-claim and multi-claim file formats. Implementations SHOULD support scope files.

### 7.1 YAML Version and Encoding

- RAVE files MUST be valid YAML. Implementations SHOULD target YAML 1.2.
- RAVE files MUST be UTF-8 encoded.
- RAVE files MUST contain exactly one YAML document (no `---` multi-document separators).
- YAML tags and anchors are permitted but SHOULD be avoided for portability across parsers.

### 7.2 Single-Claim Files

A single-claim file contains exactly one Claim object (section 6.1.2) as the top-level YAML mapping. The file is detected as single-claim by the presence of `claim_id` at the top level (see section 7.4).

All fields defined in section 6.1.2 apply directly at the top level — there is no wrapper key.

**Example — deployment rollback claim (active, structured owner):**

```yaml
claim_id: "claim-deploy-rollback-001"
statement: "Production deployments can be rolled back within 5 minutes"
owner:
  name: "platform-team"
  team: "Platform Engineering"
  contact: "#platform-eng"
status: "active"
category: "reliability"
scope:
  type: "pipeline"
  target: "deploy-to-prod"
assumptions:
  - "Deployment uses blue-green strategy with warm standby"
  - "Previous deployment artefact is retained for at least 24 hours"
  - "Rollback does not require database migration reversal"
falsification_signals:
  - "Rollback duration exceeds 5 minutes in any production deployment"
  - "Previous deployment artefact is unavailable at rollback time"
confidence_score: 0.82
last_validated: "2026-02-14T10:30:00Z"
annotations:
  - text: "Validated during Q1 deployment drill. One edge case identified — rollback with config change took 4m48s."
    author: "jchen"
    created_at: "2026-02-14T11:00:00Z"
```

> A standalone version of this example is available at `examples/deploy-rollback.claim.yaml`.

### 7.3 Multi-Claim Files

A multi-claim file groups multiple Claim objects under a `claims` key. The top-level mapping contains the `claims` list and MAY contain file-level metadata fields. The file is detected as multi-claim by the presence of `claims` at the top level (see section 7.4).

**Design rationale.** A single-document format with a `claims` key wrapper (rather than `---` multi-document separators) enables file-level metadata and is more reliably parsed across YAML implementations.

**Top-level fields:**

| Field | Type | Required | Description |
|---|---|---|---|
| `claims` | `list[Claim]` | REQUIRED | The list of Claim objects. Each entry follows the schema defined in section 6.1.2. |
| `version` | `string` | RECOMMENDED | RAVE spec version this file conforms to (e.g. `"0.1.0"`). |
| `description` | `string` | OPTIONAL | Human-readable purpose or grouping rationale for the claims in this file. |

Implementations MAY extend the top-level keys with additional fields, but MUST NOT redefine the semantics of `claims`.

**Example — payment service claims (two claims with file-level metadata):**

```yaml
version: "0.1.0"
description: "Reliability claims for the payment service deployment pipeline and messaging layer"
claims:
  - claim_id: "claim-deploy-rollback-001"
    statement: "Production deployments can be rolled back within 5 minutes"
    owner:
      name: "platform-team"
      team: "Platform Engineering"
      contact: "#platform-eng"
    status: "active"
    category: "reliability"
    scope:
      type: "pipeline"
      target: "deploy-to-prod"
    assumptions:
      - "Deployment uses blue-green strategy with warm standby"
      - "Previous deployment artefact is retained for at least 24 hours"
      - "Rollback does not require database migration reversal"
    falsification_signals:
      - "Rollback duration exceeds 5 minutes in any production deployment"
      - "Previous deployment artefact is unavailable at rollback time"
    confidence_score: 0.82
    last_validated: "2026-02-14T10:30:00Z"
    annotations:
      - text: "Validated during Q1 deployment drill. One edge case identified — rollback with config change took 4m48s."
        author: "jchen"
        created_at: "2026-02-14T11:00:00Z"

  - claim_id: "claim-kafka-exactly-once-001"
    statement: "Order events are delivered exactly once to the fulfilment service"
    owner: "messaging-team"
    status: "draft"
    category: "reliability"
    scope:
      type: "component"
      target: "order-events/kafka-consumer"
    assumptions:
      - "Kafka consumer uses idempotent processing with deduplication window of 7 days"
      - "Consumer group has a single active instance per partition"
      - "Broker replication factor is at least 3"
    falsification_signals:
      - "Duplicate order event detected in fulfilment service within deduplication window"
      - "Order event missing from fulfilment service after consumer acknowledgement"
    confidence_score: 0.0
    last_validated: "2026-02-10T09:00:00Z"
    annotations:
      - text: "Claim drafted based on design review. Awaiting integration test evidence before activation."
        author: "amorales"
        created_at: "2026-02-10T09:15:00Z"
```

> A standalone version of this example is available at `examples/payment-service.claims.yaml`.

### 7.4 Distinguishing File Types

Implementations MUST detect file type by inspecting the top-level keys of the YAML document:

| Top-level key present | File type |
|---|---|
| `claim_id` | Single-claim (section 7.2) |
| `claims` | Multi-claim (section 7.3) |
| `scopes` | Scope file (section 7.8) |
| Both `claim_id` and `claims` | Invalid — implementations MUST reject the file |
| None of the above | Invalid — implementations MUST reject the file |

File extension (section 7.5) is advisory only — top-level key detection is the authoritative mechanism.

### 7.5 File Naming Conventions

The following naming conventions are RECOMMENDED:

- **Single-claim files:** `<name>.claim.yaml` (or `<name>.claim.yml`)
- **Multi-claim files:** `<name>.claims.yaml` (or `<name>.claims.yml`)
- **Scope files:** `<name>.scope.yaml` (or `<name>.scope.yml`)

The file name SHOULD describe the content's scope or purpose (e.g. `deploy-rollback.claim.yaml`, `payment-service.claims.yaml`, `checkout-platform.scope.yaml`).

File extension is advisory — implementations MUST NOT rely on the extension to determine file type. The top-level key detection mechanism defined in section 7.4 is authoritative.

### 7.6 Directory Organisation

This section is non-normative guidance. Implementations MAY adopt any directory layout suited to their project structure.

**RECOMMENDED layout for a single-service project:**

```
.rave/
  scopes.yaml
  claims/
    deploy-rollback.claim.yaml
    api-availability.claim.yaml
    data-consistency.claims.yaml
```

**RECOMMENDED layout for a multi-service project:**

```
.rave/
  checkout-platform.scope.yaml
  claims/
    payments/
      deploy-rollback.claim.yaml
      api-uptime.claim.yaml
    orders/
      event-delivery.claims.yaml
      processing-latency.claim.yaml
    platform/
      infra-availability.claims.yaml
```

**Guidelines:**

- Scope files SHOULD be stored at the root of the `.rave/` directory, defining the containment hierarchy for the project.
- Claim files SHOULD be stored under a `.rave/claims/` directory.
- Multi-service projects SHOULD use subdirectories named after the service or team.
- All RAVE files SHOULD be committed to version control alongside the code they describe.

### 7.7 Validation Rules

This section consolidates the validation rules that implementations MUST or SHOULD enforce when parsing claim files. All rules are derived from sections 6.1.2, 6.1.3, 6.1.5, and 6.1.6 — no new field semantics are introduced here.

#### 7.7.1 Field Presence and Type Constraints

Implementations MUST validate field presence and types for each Claim in the file:

| Field | Validation Rule |
|---|---|
| `claim_id` | MUST be present. MUST be a non-empty string. |
| `statement` | MUST be present. MUST be a non-empty string. |
| `owner` | MUST be present. MAY be a string or an object. If string: MUST be a non-empty string, interpreted as `{ name: "<value>" }`. If object: `name` MUST be a non-empty string; `team` and `contact` are OPTIONAL strings (section 6.1.2.1). |
| `status` | MUST be present. MUST be one of: `draft`, `active`, `contradicted`, `retired` (section 6.1.6). |
| `scope` | MUST be present. MUST be a mapping with `type` and `target` keys (section 6.1.3). |
| `scope.type` | MUST be present. MUST be a non-empty string. SHOULD be a well-known scope type (section 6.6.3). |
| `scope.target` | MUST be present. MUST be a non-empty string. MUST NOT be a wildcard or empty string (section 6.1.3). |
| `assumptions` | MUST be present. MUST be a list of strings with at least one entry (section 6.1.4). |
| `falsification_signals` | MUST be present. MUST be a list of strings with at least one entry (section 6.1.2). |
| `confidence_score` | MUST be present. MUST be a number in the range [0.0, 1.0] (section 6.4.2). |
| `last_validated` | MUST be present. MUST be a valid ISO 8601 datetime string. |
| `category` | OPTIONAL. If present, MUST be a non-empty string. SHOULD be a well-known category from section 6.1.8. |
| `annotations` | OPTIONAL. When present, MUST be a list of Annotation objects (section 6.1.5). |

**Annotation validation (section 6.1.5):**

| Field | Validation Rule |
|---|---|
| `text` | MUST be present. MUST be a non-empty string. |
| `author` | OPTIONAL. When present, MUST be a string. |
| `created_at` | RECOMMENDED. When present, MUST be a valid ISO 8601 datetime string. |

#### 7.7.2 Value Constraints

- The `status` field MUST be one of the enum values defined in section 6.1.6: `draft`, `active`, `contradicted`, `retired`.
- The `confidence_score` MUST be a number in the range [0.0, 1.0] inclusive.
- The `scope.target` field MUST NOT be empty or a wildcard pattern (section 6.1.3).
- ISO 8601 datetime strings MUST include at least a date and time component. Timezone designators are RECOMMENDED.

#### 7.7.3 Multi-Claim Validation

When validating a multi-claim file (section 7.3):

- Each Claim in the `claims` list MUST be validated independently against the rules in sections 7.7.1 and 7.7.2.
- The `claim_id` field MUST be unique within a single file. Implementations MUST reject files containing duplicate `claim_id` values.
- Cross-file uniqueness of `claim_id` is RECOMMENDED but enforcement is implementation-defined.

#### 7.7.4 Error Handling

- Implementations SHOULD report all validation errors in a file, not just the first error encountered.
- Implementations MUST reject files that violate REQUIRED field rules (section 7.7.1).
- Implementations SHOULD warn (but MAY accept) files that violate RECOMMENDED or OPTIONAL field guidelines.
- Validation errors SHOULD include the field path, the rule violated, and a human-readable message.

Implementations MAY extend the validation rules with additional constraints but MUST NOT relax the REQUIRED field rules defined in section 6.1.2.

### 7.8 Scope Files

A scope file declares one or more Scope entities (section 6.6) for storage, exchange, and version control. The file is detected as a scope file by the presence of `scopes` at the top level (section 7.4).

**Top-level fields:**

| Field | Type | Required | Description |
|---|---|---|---|
| `scopes` | `list[Scope]` | REQUIRED | The list of Scope objects. Each entry follows the schema defined in section 6.6.2. |
| `version` | `string` | RECOMMENDED | RAVE spec version this file conforms to (e.g. `"0.1.0"`). |
| `description` | `string` | OPTIONAL | Human-readable description of this scope tree. |

**Example — checkout platform scope tree:**

```yaml
version: "0.1.0"
description: "Scope hierarchy for the checkout platform"
scopes:
  - type: "system"
    target: "checkout-platform"
    description: "The full checkout platform"

  - type: "application"
    target: "payments-app"
    parent:
      type: "system"
      target: "checkout-platform"
    description: "Payment processing application"

  - type: "service"
    target: "payments-api"
    parent:
      type: "application"
      target: "payments-app"
    owner:
      name: "backend-team"
      team: "Backend Services"
      contact: "backend-oncall@example.com"
    dependencies:
      - type: "service"
        target: "order-service"
      - type: "pipeline"
        target: "deploy-to-prod"
    description: "REST API for payment processing"

  - type: "service"
    target: "order-service"
    parent:
      type: "system"
      target: "checkout-platform"
    description: "Order management service"

  - type: "pipeline"
    target: "deploy-to-prod"
    parent:
      type: "service"
      target: "payments-api"
    description: "CI/CD pipeline for deploying payments-api to production"
```

> A standalone version of this example is available at `examples/checkout-platform.scope.yaml`.

#### 7.8.1 Scope File Validation Rules

Implementations MUST validate scope files against the following rules:

| Rule | Level | Description |
|---|---|---|
| `scopes` present | MUST | The `scopes` key MUST be present and MUST be a non-empty list. |
| `type` present | MUST | Each scope MUST have a `type` field that is a non-empty string. |
| `target` present | MUST | Each scope MUST have a `target` field that is a non-empty string. |
| Unique identity | MUST | The `(type, target)` pair MUST be unique within a single file. Implementations MUST reject files containing duplicates. |
| `parent` structure | MUST | When `parent` is present, it MUST be an object with `type` (non-empty string) and `target` (non-empty string) fields. |
| `owner` structure | MUST | When `owner` is present, it MUST be a non-empty string or an object. If object: `name` MUST be a non-empty string; `team` and `contact` are OPTIONAL strings. Same rules as Claim `owner` (section 6.1.2.1). |
| Parent resolution | MUST | When `parent` is present, the referenced `(type, target)` MUST resolve to a scope declared in the same file or a co-located scope file. Implementations MUST reject unresolvable parent references. |
| Acyclic | MUST | The parent graph MUST be acyclic. Implementations MUST reject scope files that introduce cycles. |
| `dependencies` structure | MUST | When `dependencies` is present, it MUST be a list of objects. Each entry MUST have `type` (non-empty string) and `target` (non-empty string) fields. |
| Dependency resolution | MUST | Each dependency reference MUST resolve to a declared scope. Implementations MUST reject unresolvable dependency references. |
| No self-dependency | MUST | A scope MUST NOT list itself in its `dependencies`. Implementations MUST reject scope entries where a dependency's `(type, target)` matches the scope's own `(type, target)`. |
| Dependency cycles | SHOULD | Implementations SHOULD detect and warn about circular dependencies but MUST NOT reject them. |
| Well-known type | SHOULD | The `type` field SHOULD be a well-known scope type (section 6.6.3). |
| Cross-file uniqueness | RECOMMENDED | The `(type, target)` pair SHOULD be unique across all scope files in a dataset. Enforcement is implementation-defined. |

---

## 8. Versioning

### 8.1 Scheme

The RAVE specification uses [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`):

- **MAJOR** — breaking changes to the spec (incompatible with prior versions)
- **MINOR** — additions that are backwards-compatible
- **PATCH** — clarifications, typo fixes, non-normative changes

### 8.2 Stability

| Version Range | Stability |
|---------------|-----------|
| `0.x`         | Draft — expect breaking changes |
| `1.0+`        | Stable — breaking changes require a new major version |

### 8.3 Status Lifecycle

Each spec version carries a status:

| Status   | Meaning |
|----------|---------|
| `Draft`  | Work in progress, open to significant change |
| `Review` | Feature-complete, open for community feedback |
| `Stable` | Ratified, changes follow SemVer rules |

---

## 9. Deferred Decisions

This section records design questions that were considered during the authoring of v0.1 and deliberately set aside. Each entry states the question, the decision reached, and the rationale. Entries here are candidates for future spec versions if the stated conditions arise.

---

### 9.1 Claim and Evidence Templates

**Question.** Should the spec define a template mechanism for Claims and Evidence — allowing a claim pattern to be defined once and instantiated per system with context-specific scope, evidence, and assumptions?

**Decision.** Not in v0.1. Deferred to a future version.

**Rationale.** For v0.1, Claims are concrete, fully-specified instances. This is sufficient for the current use case and keeps the data model simple. The duplication problem (conceptually identical claims authored separately per system, differing only in scope and evidence) is a real concern at scale, but it is not yet a pain point. A template mechanism would add significant spec complexity — instantiation semantics, inheritance rules, override behaviour — that is premature without evidence of adoption patterns that justify it.

**Revisit if:** organisations report significant duplication burden when rolling out RAVE across many systems with shared claim shapes.

**Related:** GitHub issue #21.

---

## 10. Acknowledgements & References

- [rave-overview.md](../rave-overview.md) — Original RAVE vision document
- [Semantic Versioning](https://semver.org/)
- [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt) — Key words for use in RFCs
