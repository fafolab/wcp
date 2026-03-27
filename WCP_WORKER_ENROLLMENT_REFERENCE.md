# WCP Worker Enrollment Reference

**Status:** active reference document
**Written:** 2026-03-26
**Purpose:** define the enrollment schema, worker identity format, and trust attribution rules for WCP-governed workers.

---

## Overview

Workers register in the router via a **registry record**. This document defines the enrollment JSON schema, the format of `worker_id` for identity and attribution, and the rules governing trust derivation.

---

## 1. Enrollment Schema

Workers register in the router via a registry record:

```json
{
  "worker_id": "org.example.doc-summarizer",
  "worker_species_id": "wrk.doc.summarizer",
  "capabilities": ["cap.doc.summarize"],
  "risk_tier": "low",
  "idempotency": "full",
  "determinism": "captured",
  "required_controls": ["ctrl.obs.audit_log_append_only"],
  "currently_implements": ["ctrl.obs.audit_log_append_only"],
  "allowed_environments": ["dev", "stage", "prod"],
  "privilege_envelope": {
    "secrets_access": [],
    "network_egress": "none",
    "filesystem_writes": ["/tmp/summaries/"],
    "tools": ["ollama-embed"]
  },
  "blast_radius": {
    "data": 1,
    "network": 0,
    "financial": 0,
    "time": 1,
    "reversibility": "reversible"
  },
  "owner": "org.example",
  "contact": "https://workerclassprotocol.dev/contact/",
  "artifact_hash": "sha256:...",
  "catalog_version_min": "1.0.0",
  "attestation": {
    "code_hash": "sha256:a3f9c2...",
    "hash_method": "file",
    "attested_at": "2026-02-25T00:00:00Z",
    "attested_by": "namespace-key@org.example"
  }
}
```

### Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `worker_id` | string | yes | Unique enrolled worker instance identifier (see section 2) |
| `worker_species_id` | string | yes | Worker species taxonomy identifier (`wrk.*`) |
| `capabilities` | string[] | yes | List of capability IDs this worker implements |
| `risk_tier` | enum | yes | `low`, `medium`, `high`, `critical` |
| `idempotency` | enum | yes | `full`, `partial`, `none` |
| `determinism` | enum | yes | `deterministic`, `captured`, `nondeterministic` |
| `required_controls` | string[] | yes | Controls that must be verified before dispatch |
| `currently_implements` | string[] | yes | Controls this worker currently satisfies |
| `allowed_environments` | string[] | yes | Environments where dispatch is permitted |
| `privilege_envelope` | object | yes | Declared resource access boundaries |
| `blast_radius` | object | yes | Five-dimension blast score declaration |
| `owner` | string | yes | Authority namespace of the worker's owner |
| `contact` | string | no | Contact URL or email |
| `artifact_hash` | string | yes | SHA-256 hash of the worker artifact |
| `catalog_version_min` | string | yes | Minimum WCP catalog version required |
| `attestation` | object | no | Code attestation record (required for WCP-Full) |

---

## 2. Worker Identity

A `worker_id` identifies a specific enrolled worker instance. It carries the authority namespace that determines trust attribution.

### Format

```
<authority-namespace>.<worker-name>[.<instance-suffix>]
```

### Examples

```
org.xyzcompany.doc-summarizer.prod-01
org.abcdevelopmentcorp.registry-manager.prod-02
x.joe.local-research-worker.dev-01
x.lisajohnson.mem-curator.laptop-01
```

The authority namespace is extracted from the leading segments of `worker_id`:

- `org.xyzcompany.doc-summarizer.prod-01` -- authority namespace: `org.xyzcompany`
- `x.joe.local-research-worker.dev-01` -- authority namespace: `x.joe`

---

## 3. Trust Attribution

Attestation and trust attribution MUST be based on the authority namespace carried by `worker_id`.

Trust attribution MUST NOT be derived from:

- `worker_species_id` -- this is taxonomy, not authority
- `capability_id` -- this is a contract, not ownership
- any `wrk.*` prefix -- this conveys species classification only

**Correct reasoning:**

```
This worker is trusted under org.xyzcompany because its worker_id is
org.xyzcompany.doc-summarizer.prod-01.
Its species is wrk.doc.summarizer.
It implements cap.doc.summarize.
```

**Incorrect reasoning:**

```
This worker is trusted because its species starts with wrk.doc.
```

The separation is: `worker_id` carries authority, `worker_species_id` carries taxonomy, `capability_id` carries the contract.

---

*WCP Worker Enrollment Reference v0.2*
*workerclassprotocol.dev*
*Published: 2026-03-26*
