# WCP — Worker Class Protocol

**Version:** 0.2 &ensp; · &ensp; **Status:** Published &ensp; · &ensp; **Date:** 2026-03-26

> *"The protocol belongs to whoever needs it. Build on it. Fork it. Make it yours."*

## Abstract

WCP (Worker Class Protocol) is an open standard for governing the dispatch of Python workers in AI agent systems. It defines how workers are classified, how agents request capabilities, how routing decisions are made, and what evidence must be produced.

WCP does not define a transport. WCP does not define an agent architecture. WCP defines the governance layer between a capability request and its execution — the layer that answers: *should this worker be trusted with this job, under these conditions, with this data?*

WCP is additive, not exclusive. MCP (Anthropic) defines how agents call tools. A2A (Google/Linux Foundation) defines how agents communicate with each other. ACP and AGNTCY define agent collaboration patterns. WCP governs the workers those protocols dispatch. A WCP-governed worker can be exposed as an MCP tool or called by an A2A agent — the governance layer sits beneath the transport, not beside it.

WCP is designed for the small developer, the home lab, and the enterprise. It scales from a single developer's laptop to a multi-tenant production fleet. No cloud. No vendor lock-in. Pure Python reference implementation.

---

## 1. Problem Statement

The majority of AI agent systems lack safety cards.

Every major agent protocol — MCP, A2A, ACP, AGNTCY — defines how agents communicate. None define whether a worker should be trusted to execute, under what controls, with what blast radius, and with what evidence.

The result is predictable:

**Pattern 1 — The Blast Radius Accident**
An agent calls a worker with write access. The worker fails mid-execution. Nobody knows how far the damage spread. No audit trail. No blast radius score was ever computed.

**Pattern 2 — The Silent Failure**
A worker fails. No notification. No dead-letter queue. The agent retries. The worker fails again. The system appears to be running. Nothing is happening.

**Pattern 3 — The Hallucinated Tool Call**
An agent calls a capability that doesn't exist. The system returns a generic error. The agent retries with a different tool. Neither call was governed, logged, or attributable.

**Pattern 4 — The Unqualified Worker**
An agent dispatches a worker to handle sensitive data. The worker was never certified for that data label. No policy gate exists. The data moves without authorization.

WCP is the answer to all four patterns.

---

## 2. Core Concepts

### 2.1 Capability

A **capability** is a stable, permanent identifier for something that must be possible.

```
cap.<domain>[.<subdomain>].<verb>
```

Examples:
```
cap.doc.summarize
cap.doc.pdf.extract
cap.mem.retrieve
cap.mem.embed
cap.web.fetch
cap.research.register
cap.notify.send
cap.db.write
```

**Capability IDs are permanent.** Once published in a WCP catalog, a capability ID never changes meaning and is never deleted — only tombstoned. If semantics change, a new capability is defined.

**Capabilities are verbs.** They describe what must be possible, not what implements it.

### 2.2 Worker Species

A **worker species** is a stable identifier for the type of worker that implements a capability. A worker species is a taxonomy identifier — it describes the class of worker, not its authority or trust context.

```
wrk.<domain>[.<subdomain>].<role>
```

Examples:
```
wrk.doc.summarizer
wrk.doc.pdf.extractor
wrk.mem.retriever
wrk.mem.embedder
wrk.web.fetcher
wrk.research.registrar
```

Worker species mirror their capabilities: `cap.doc.summarize` → `wrk.doc.summarizer`. The naming convention enforces the relationship.

### 2.3 Control

A **control** is a governance invariant that must be satisfied before a worker can be dispatched.

```
ctrl.<domain>.<predicate>
```

Controls are predicate statements describing a required state:

```
ctrl.obs.audit_log_append_only     — audit log is append-only
ctrl.net.egress_denied             — egress is denied by default
ctrl.identity.secrets_deny_default — secrets access is deny-by-default
ctrl.mem.provenance_required       — all memory access tracked
```

Workers declare which controls they implement. The router verifies declaration before dispatch.

### 2.4 Policy

A **policy** is a decision rule that the policy gate evaluates.

```
pol.<domain>.<rule>
```

Policies govern when escalation, step-up approval, or human review is required.

### 2.5 Profile

A **profile** is a pre-bundled posture — a named set of controls and policies for a specific environment or risk level.

```
prof.<domain>.<name>
```

Examples:
```
prof.dev.permissive       — development: relaxed controls
prof.prod.strict          — production: all controls enforced
prof.edge.isolated        — edge: no egress, local only
prof.mem.rag_strict       — RAG: strict scoped retrieval
```

The same worker fleet runs in development chaos mode and production discipline mode by swapping profiles — no code changes.

### 2.6 Router

The **router** (registry + dispatch authority) is the central dispatch component. When an agent needs a worker, it contacts the router. The router:

1. Receives a capability request
2. Matches it against routing rules
3. Verifies the worker's declared controls
4. Evaluates the policy gate
5. Computes blast radius
6. Issues a routing decision with evidence
7. Dispatches (or denies with reason)

Agents that adopt WCP get access to the governed worker fleet. The router ensures every dispatch is governed, auditable, and attributable.

---

## 3. Identifier Rules

### 3.0 Design Heritage

WCP identifier design follows established technology industry conventions. The dot-notation domain-prefix pattern is not invented — it mirrors how mature standards bodies organize stable, hierarchical identifiers:

| Standard | Pattern | Example | WCP parallel |
|----------|---------|---------|--------------|
| MITRE ATT&CK | `TA<N>` / `T<N>.<N>` | `T1566.001` = Spearphishing | `cap.doc.summarize` — stable, hierarchical |
| NIST SP 800-53 | `<Domain>-<N>` | `AC-2` = Account Management | `ctrl.identity.secrets_deny_default` |
| OpenTelemetry SemConv | `<domain>.<attribute>` | `gen_ai.request.model` | `cap.ml.infer`, `evt.os.task.routed` |
| CWE | `CWE-<N>` | `CWE-89` = SQL Injection | `ctrl.*` controls map to CWE mitigations |
| OWASP ASVS | `V<N>` levels | L1/L2/L3 verification | WCP-Basic/Standard/Full compliance levels |

WCP adopts the domain-prefix + dot-separation pattern used across these standards: short domain identifiers, human-readable without a lookup table, machine-parseable, and stable once published.

### 3.1 Namespace Ownership

WCP uses two distinct namespace families: **protocol namespaces** (reserved language primitives) and **authority namespaces** (trust attribution source).

#### Protocol Namespaces

| Prefix | Owner | Stability |
|--------|-------|-----------|
| `cap.*` | WCP catalog | Permanent once published |
| `wrk.*` | WCP catalog | Permanent once published |
| `ctrl.*` | WCP catalog | Permanent once published |
| `pol.*` | WCP catalog | Permanent once published |
| `prof.*` | WCP catalog | Permanent once published |
| `evt.*` | WCP catalog | Permanent once published |

No entity outside the WCP catalog may publish IDs in the reserved protocol namespaces.

#### Authority Namespaces

Authority namespaces identify the owner/authority behind a worker. They are used in `worker_id` for trust attribution, enrollment identity, and attestation.

| Prefix | Scope | Stability |
|--------|-------|-----------|
| `org.<name>.*` | Organizations, companies, teams, business entities | Owner's discretion |
| `x.<name>.*` | Individuals, personal namespaces, solo developers | Owner's discretion |

`org.<name>.*` and `x.<name>.*` are the only two authority namespace families defined by WCP. `x.<name>.*` denotes individual/personal authority.

### 3.2 Format Rules

```
Segments:     2 to 4 dot-separated segments
Characters:   lowercase a-z, digits 0-9, hyphens (-) only
Separator:    dot (.)
Forbidden:    underscores, uppercase, spaces, unicode
Max length:   64 characters total

VALID:
  cap.doc.summarize
  cap.doc.pdf.extract
  ctrl.obs.audit-log-append-only

INVALID:
  cap.Doc.Summarize          (uppercase)
  cap.doc.pdf_extract        (underscore)
  cap.doc.pdf.native.extract (5 segments — too deep)
```

### 3.3 Capability IDs Are Permanent

A published WCP capability ID:
- MUST never change its core meaning
- MUST never be removed — only tombstoned
- MAY be superseded by a more specific capability

Tombstone format:
```json
{
  "id": "cap.doc.ocr.basic",
  "status": "deprecated",
  "superseded_by": "cap.doc.ocr",
  "deprecated_catalog": "2.0.0",
  "sunset_catalog": "4.0.0"
}
```

### 3.4 Catalog Versioning

The **catalog** is versioned separately from capability IDs. Workers declare catalog version compatibility. Routing rules declare the minimum catalog version they require.

```
wcp-catalog@1.0.0     — initial release
wcp-catalog@1.1.0     — new capabilities added (backward compatible)
wcp-catalog@2.0.0     — breaking change in control schema (major bump)
```

Capability IDs do not contain version numbers. Versions belong to implementations and catalogs, not to capability definitions.

---

## 4. The Routing Envelope

Every WCP dispatch begins with a **RouteInput** and produces a **RouteDecision**.

### 4.1 RouteInput

```json
{
  "correlation_id": "uuid-v4",
  "tenant_id": "string",
  "env": "dev | stage | prod | edge",
  "data_label": "PUBLIC | INTERNAL | RESTRICTED",
  "tenant_risk": "low | medium | high",
  "qos_class": "P0 | P1 | P2 | P3",
  "capability_id": "cap.doc.summarize",
  "request": {},
  "policy_version": "policy.v0",
  "dry_run": false
}
```

**QoS classes:**
- P0 — critical, maximum governance, human escalation may be required
- P1 — high priority, full governance
- P2 — standard, default governance
- P3 — background, relaxed governance

### 4.2 RouteDecision

```json
{
  "decision_id": "uuid-v4",
  "timestamp": "ISO8601-UTC",
  "correlation_id": "propagated from input",
  "tenant_id": "propagated",
  "capability_id": "cap.doc.summarize",
  "matched_rule_id": "rr_abc123",
  "selected_worker_species_id": "wrk.doc.summarizer",
  "env": "dev",
  "data_label": "INTERNAL",
  "tenant_risk": "low",
  "qos_class": "P2",
  "denied": false,
  "deny_reason_if_denied": null,
  "blast_score": 3,
  "blast_gate_passed": true,
  "privilege_envelope_ok": true,
  "required_controls_effective": ["ctrl.obs.audit_log_append_only", "..."],
  "recommended_profiles_effective": [{"profile_id": "...", "score": 1.9}],
  "escalation_effective": {"policy_gate": true, "human_required_default": false},
  "telemetry_envelopes": [
    {"event_id": "evt.os.task.routed", "timestamp": "...", "correlation_id": "..."},
    {"event_id": "evt.os.worker.selected", "timestamp": "...", "correlation_id": "..."},
    {"event_id": "evt.os.policy.gated", "timestamp": "...", "correlation_id": "..."}
  ]
}
```

---

## 5. Required Behaviors

Any WCP-compliant implementation MUST:

### 5.1 Fail Closed
If no routing rule matches a capability request, the decision MUST be `denied: true`. Unknown capabilities are never executed. No exceptions.

### 5.2 Deterministic Routing
Given identical inputs, the routing decision MUST be identical. Routing rules are tested against golden snapshots. Any change to routing behavior requires a new rule version.

### 5.3 Declared Controls
A worker MUST declare which controls it implements in its registry record. The router MUST verify this declaration before dispatch. Missing required controls → `denied: true`.

### 5.4 Mandatory Telemetry
Every dispatch MUST emit three minimum telemetry events:
- `evt.os.task.routed` — routing decision made
- `evt.os.worker.selected` — worker species selected
- `evt.os.policy.gated` — policy gate evaluated

The `correlation_id` MUST be propagated through all three events and through all downstream worker calls.

**OpenTelemetry compatibility:** Workers that invoke LLMs SHOULD emit the following OpenTelemetry Generative AI semantic convention attributes within their telemetry events for out-of-the-box compatibility with existing OTel pipelines:
- `gen_ai.request.model` — the model name requested
- `gen_ai.usage.input_tokens` — tokens consumed in the prompt
- `gen_ai.usage.output_tokens` — tokens in the completion
- `gen_ai.tool.name` — the WCP capability ID that triggered the LLM call

### 5.5 Dry-Run Mode
Every WCP router MUST support `"dry_run": true`. In dry-run mode, the full routing decision is made and returned but no worker is executed. Dry-run MUST be fully functional for all capabilities.

### 5.6 Capability Discovery
A WCP-compliant router MUST respond to:
- `GET /wcp/capabilities` — all registered capabilities
- `GET /wcp/workers` — all enrolled workers
- `GET /wcp/health` — compliance status

### 5.7 Evidence Receipt
Every executed dispatch MUST return an evidence receipt containing at minimum:
- `correlation_id`
- `dispatched_at`
- `worker_id`
- `capability_id`
- `policy_decision`
- `controls_verified`
- `artifact_hash` (SHA-256 of the request payload)

#### Receipt Chain Fields (v0.3.x roadmap)

Implementations SHOULD include the following additional fields to support
hash-chain ledger and Merkle proof architecture. These are required for
FWP (Flood-em-with-Proof) courtroom-grade non-repudiation:

- `receipt_id` — UUIDv7, unique per decision event
- `prev_receipt_hash` — SHA-256 of previous receipt in stream (per tenant/session); NULL for first in stream
- `receipt_hash` — SHA-256 of canonical receipt JSON (keys sorted lexicographically, compact, excluding `receipt_hash` and `signature` fields)
- `signing_key_id` — key identifier for verifying signature
- `signature` — Ed25519 signature over `receipt_hash`

**Canonical serialization:** JSON with keys sorted lexicographically, no
trailing whitespace, compact (no spaces). Implementation MUST pin this
spec version in the `receipt_hash` computation.

**Chain breaks MUST be treated as sev1 integrity events.**

### 5.8 Human-in-Loop (REQUIRE_HUMAN)

When a policy gate returns `REQUIRE_HUMAN`, the router MUST NOT silently approve or deny. It MUST:

1. Return a `RouteDecision` with `denied: false` and `supervisor_required: true`
2. Include `supervisor_level` in the decision — one of:
   - `advisory` — human is notified, execution proceeds
   - `gatekeeper` — human must approve before execution
   - `executor` — human executes the action instead of the worker
   - `incident_commander` — human takes full control; all agent actions subordinate
3. Expose the pending decision via the router's discovery API (`GET /wcp/approvals/pending`)
4. Accept a resolution via a pluggable `HumanApprovalCallback`

**The approval mechanism is intentionally unspecified.** WCP defines the contract. Implementations wire whatever H2A channel fits their environment — webhook, email, Slack, Discord, ServiceNow, or a custom UI.

```json
// RouteDecision when supervisor is required
{
  "denied": false,
  "supervisor_required": true,
  "supervisor_level": "gatekeeper",
  "pending_approval_id": "uuid-v4",
  "approval_expires_at": "ISO8601",
  "escalation_context": {
    "capability_id": "cap.db.write",
    "blast_score": 8,
    "tenant_risk": "high",
    "data_label": "RESTRICTED",
    "policy_version": "policy.v1"
  }
}
```

**WCP capability for the approval flow itself:**
```
cap.ops.approve      — worker that facilitates human approval
wrk.ops.human-approver — the worker species that routes pending approvals to humans
```

Any WCP implementation that handles high-risk capabilities SHOULD implement `cap.ops.approve` as part of its worker fleet. The worker receives the pending approval context, routes it to the human via whatever H2A channel is configured, and submits the resolution back to the router.

**Approval endpoints** are exposed via the router's discovery API:
```
GET  /wcp/approvals/pending    — list all pending REQUIRE_HUMAN decisions
POST /wcp/approvals/resolve    — submit human decision (approve | deny | escalate)
POST /wcp/approvals/escalate   — escalate supervisor level (gatekeeper → incident_commander)
```

This makes WCP-governed approval workflows callable from any compatible client — a Discord bot, a web dashboard, a mobile app, or an enterprise ITSM integration.

### 5.9 Signatory Tenant Validation

When `require_signatory` is enabled, the router MUST deny any `RouteInput` whose `tenant_id` is not in the registered signatory list. Only registered tenants — those explicitly added to the router's allow list — can dispatch workers. An unregistered tenant is denied, regardless of how urgent the request.

**Configuration:**

```json
{
  "require_signatory": true,
  "allowed_tenants": [
    "mcp.my-app",
    "agent.my-orchestrator",
    "x.acme.deploy-bot"
  ]
}
```

**Behavior:**

| `require_signatory` | `tenant_id` in list | Result |
|---|---|---|
| `false` (default) | any | Routing proceeds normally |
| `true` | present | Routing proceeds normally |
| `true` | absent | `denied: true`, code `DENY_UNKNOWN_TENANT` |

**Default:** `false` (development mode — any tenant is accepted).

**WCP-Full compliance** requires `require_signatory: true` in production deployments.

**On trust:** Anyone can modify a binary or forge a `tenant_id`. Signatory enforcement is not about absolute prevention — it is about establishing an **auditable chain of evidence**. When a rogue dispatch occurs, the audit trail proves the tenant was either registered (and accountable) or unregistered (and unauthorized). The evidence receipt, correlation chain, and telemetry events are what make WCP attestation meaningful in post-incident review or litigation.

**Deny payload:**

```json
{
  "denied": true,
  "deny_reason_if_denied": {
    "code": "DENY_UNKNOWN_TENANT",
    "message": "Tenant is not registered as a signatory. Register via the router configuration before dispatching workers.",
    "tenant_id": "<redacted-or-full>"
  }
}
```

The router MAY redact the tenant_id in the error payload in production environments.

### 5.10 Worker Code Attestation

A worker is only as trustworthy as its code. A compromised worker — one whose code has been modified after attestation — may exfiltrate payloads, bypass controls, or behave in ways the tenant never authorized. WCP closes this attack surface at the dispatch layer.

**The threat:**

```
1. XYZ Company builds cap.doc.summarize worker
2. Attacker gains access, modifies worker code
3. Modified worker silently forwards payloads to attacker's endpoint
4. Router dispatches it normally — it sees a valid capability ID, valid tenant
5. Data exfiltrated. Evidence receipt shows nothing wrong. Attack is silent.
```

**The defense — Worker Code Attestation:**

When a worker is enrolled in the router, its code hash (SHA-256) is registered as the **known-good fingerprint**. At every dispatch, the router computes or retrieves the worker's current hash and compares it to the registered hash. A mismatch means the worker has been modified — dispatch is denied and the worker is flagged.

```
Registration:
  developer builds worker → computes SHA-256 of worker code
  → registers hash with router (and optionally with an external attestation authority)
  → router stores: { worker_species_id, registered_code_hash, attested_at }

Dispatch (when require_worker_attestation: true):
  router selects candidate worker
  → retrieves registered_code_hash from registry
  → computes current_hash from live worker (file, module, or image digest)
  → MATCH → dispatch proceeds, attestation_valid: true in evidence receipt
  → MISMATCH → DENY_WORKER_TAMPERED, worker flagged for investigation
```

**Evidence receipt when attestation is checked:**

```json
{
  "worker_species_id": "wrk.doc.summarizer",
  "worker_attestation_checked": true,
  "worker_attestation_valid": true,
  "registered_hash": "a3f9c2...",
  "current_hash": "a3f9c2..."
}
```

**Evidence receipt on tamper detection:**

```json
{
  "denied": true,
  "deny_reason_if_denied": {
    "code": "DENY_WORKER_TAMPERED",
    "message": "Worker code hash mismatch. Worker may have been modified after attestation.",
    "worker_species_id": "wrk.doc.summarizer",
    "registered_hash": "a3f9c2...",
    "current_hash": "7b2c91..."
  },
  "worker_attestation_checked": true,
  "worker_attestation_valid": false
}
```

**Hash computation methods (in order of tamper resistance):**

| Method | How | Best for |
|---|---|---|
| File hash | SHA-256 of worker Python file at dispatch | Local workers, simple deployments |
| Module hash | SHA-256 of compiled bytecode + source | Python packages |
| Container digest | OCI image digest (immutable) | Containerized workers |
| Signed manifest | Authority-issued signed attestation bundle | Multi-tenant, enterprise |

**Configuration:**

```json
{
  "require_worker_attestation": true
}
```

**Default:** `false`. WCP-Full compliance requires `require_worker_attestation: true` in production.

**External attestation authority (v0.2+):** In v0.1, attestation hashes are stored in the local router registry. In v0.2, worker hashes MUST be registered with a third-party external attestation authority, which issues a signed attestation manifest. An attestation authority MUST publish a **global ban list** — known-compromised hashes that any router operator can subscribe to, refusing dispatch of banned workers regardless of local registry state.

**On trust:** Attestation does not prevent a sufficiently privileged attacker from modifying both the worker and its registered hash simultaneously. It closes the common case: unauthorized code modification by an attacker who does not have access to the router's registry. Combined with signatory tenant validation (section 5.9) and the cryptographic evidence receipt, WCP establishes an auditable chain of custody that proves — in post-incident review or litigation — exactly what ran, when, and whether it matched what the developer originally registered.

---

## 6. Blast Radius

Before dispatching a worker, the router computes a **blast score** — the potential damage radius if the worker fails or behaves unexpectedly.

Blast score is the sum of five dimensions (0-5 each):

| Dimension | 0 | 1-2 | 3-4 | 5 |
|-----------|---|-----|-----|---|
| **Data** | No data touched | Read-only | Writes local | Writes external/shared |
| **Network** | No egress | Allowlisted egress | Broad egress | Unrestricted |
| **Financial** | None | Negligible | Noticeable | Significant |
| **Time** | Instant | <1 min | <1 hour | Long-running |
| **Reversibility** | 0 | Partially reversible | Difficult | Irreversible |

Routing rules declare maximum blast score per environment. Requests exceeding the threshold are denied or escalated for human review.

**Blast radius is additive in worker chains.** The router accounts for chain propagation:

```
Worker A blast score:      3
Worker B blast score:    + 4
                         ───
Total chain blast:       = 7
```

If the total chain blast exceeds the environment threshold, the dispatch is denied or escalated.

---

## 7. Compliance Levels

| Level | Requirements |
|-------|-------------|
| **WCP-Basic** | Capability routing, fail-closed, deterministic |
| **WCP-Standard** | + Controls enforcement, mandatory telemetry, dry-run |
| **WCP-Full** | + Blast radius, privilege envelopes, policy gate, evidence receipts, discovery API, signatory tenant validation (`require_signatory: true`), worker code attestation via third-party external attestation authority (`require_worker_attestation: true`) |

---

## 8. Companion Documents

The following documents provide additional detail on topics introduced in this specification:

- **WCP Identity, Namespace, and Taxonomy Model:**<br>
  [`WCP_IDENTITY_NAMESPACE_AND_TAXONOMY_MODEL.md`](https://github.com/workerclassprotocol/wcp/blob/main/WCP_IDENTITY_NAMESPACE_AND_TAXONOMY_MODEL.md) — defines the four-layer identity model (capability, worker species, worker identity, authority namespace), the separation between protocol namespaces and authority namespaces, and the rules for trust attribution.

- **WCP Python Worker E2E Canonical Reference:**<br>
  [`WCP_PYTHON_WORKER_E2E_CANONICAL_REFERENCE.md`](https://github.com/workerclassprotocol/wcp/blob/main/WCP_PYTHON_WORKER_E2E_CANONICAL_REFERENCE.md) — defines the canonical Python worker package layout, the 8-section file model, the two valid usage paths (full package vs. API-only), and the end-to-end execution flow for WCP-governed Python workers.

- **WCP Worker Enrollment Reference:**<br>
  [`WCP_WORKER_ENROLLMENT_REFERENCE.md`](https://github.com/workerclassprotocol/wcp/blob/main/WCP_WORKER_ENROLLMENT_REFERENCE.md) — defines the enrollment JSON schema, worker identity format, and trust attribution rules for WCP-governed workers.

---

*WCP Specification v0.2 / workerclassprotocol.dev / Published: 2026-03-26*
