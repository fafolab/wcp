# WCP Python Worker E2E Canonical Reference

> Boundary note:
> This document should not be treated as the primary home for PyHall-specific Python worker implementation guidance.
> PyHall implementation-facing worker guidance belongs in:
> `/mnt/fafolab/dev/2026-432_pyhall/internal/planning/PYHALL_PYTHON_WORKER_E2E_REFERENCE.md`
>
> WCP itself stays at the protocol layer:
> `Classification`, `Routing`, `Controls`, `Evidence`.

**Status:** Canonical reference for Python worker packaging and execution until superseded  
**Authority:** WCP clarification layer for PyHall v0.3.0-era Python workers  
**Prepared:** 2026-03-14  
**Prepared by:** Codex  

---

## 1) Purpose

This document is the concrete canonical reference for what a Python WCP worker package is supposed to look like end to end.

It exists because:

- loose example files are not enough
- design-review reference code is not the same as canonical implementation guidance
- PyHall is the first serious implementation of this model
- the identity clarification must now be reflected in the worker example shape

This document defines:

- the canonical package layout
- the canonical identity fields
- the canonical startup flow
- the canonical attestation flow
- the canonical Python file roles
- the difference between:
  - `PyHall app / opinionated full worker package`
  - `PyHall API-only / external runtime worker`
- which existing files are authoritative source material
- which existing files are useful but non-canonical

---

## 2) Canonical Identity Model

For Python workers, the identity rules are:

- `cap.*` = capability contract
- `wrk.*` = worker species taxonomy
- `worker_id` = actual worker instance identity
- authority namespace inside `worker_id` must be one of:
  - `org.<name>.*`
  - `x.<name>.*`

Examples:

- `worker_id = "org.xyzcompany.rss.instance-1"`
- `worker_id = "org.abcdevelopmentcorp.telegram.instance-1"`
- `worker_id = "x.joe.summarizer.instance-1"`
- `worker_id = "x.lisajohnson.notifications.instance-1"`

Examples of species identity:

- `worker_species_id = "wrk.pyhall.rss"`
- `worker_species_id = "wrk.pyhall.telegram"`
- `worker_species_id = "wrk.doc.summarizer"`

Critical rule:

- trust attribution comes from `worker_id`
- taxonomy membership comes from `worker_species_id`
- `wrk.*` is not an authority namespace

---

## 3) Two Valid Usage Paths

There are two valid ways to use PyHall governance with a Python worker.

### Path A: PyHall app / opinionated full worker package

This is the stronger, more opinionated model.

Typical characteristics:

- worker package exists on disk
- package includes `worker_logic.py`, `bootstrap.py`, `requirements.lock`, `config.schema.json`, and `manifest.json`
- desktop app can scaffold and enroll the worker
- package-level attestation runs locally
- local evidence and local worker package structure matter

This path expects the full package shape.

### Path B: PyHall API-only / external runtime worker

This is the looser, language-agnostic integration model.

Typical characteristics:

- developer may run their own router, Hall-like runtime, or API server
- developer may not use the PyHall desktop app at all
- developer may use PyHall only for:
  - namespace authority
  - enrollment
  - registry
  - attestation checks
  - policy/routing support
- local package layout can be thinner or externalized

This path does not require the same local file/package footprint as the app path.

### Important rule

PyHall app usage is not required for PyHall API usage.

PyHall can be:

- the full operator-facing stack
- or the governance/attestation layer used by an external runtime

---

## 4) Canonical Python Worker Package Layout

The canonical Python worker package layout is:

```text
worker-package/
  code/
    worker_logic.py
    bootstrap.py
  requirements.lock
  config.schema.json
  manifest.json
```

File roles:

- `code/worker_logic.py`
  - the actual worker implementation
  - identity declarations
  - policy gate
  - business logic
  - telemetry and evidence emission
- `code/bootstrap.py`
  - simple entrypoint that dispatches into worker logic
- `requirements.lock`
  - pinned dependencies for the package
- `config.schema.json`
  - package configuration contract
- `manifest.json`
  - signed attestation manifest for the full package

Canonical rule:

- attestation applies to the full worker package, not just one `.py` file

---

## 5) Canonical E2E Execution Flow

The canonical Python worker flow is:

1. package exists on disk with the standard layout
2. `manifest.json` exists for the package
3. startup attestation runs before normal work
4. worker verifies:
   - package hash
   - manifest identity
   - manifest signature
5. worker fails closed in production on attestation failure
6. worker constructs `WCPContext`
7. worker applies a fail-closed policy gate
8. worker executes business logic only if allowed
9. worker emits telemetry and evidence receipts
10. worker returns a structured result

This is the canonical end-to-end posture for the full packaged worker path.

For API-only use, some of these steps may be externalized into the caller's own runtime.

---

## 6) Canonical Python File Shape

This document adopts the **8-section model** as the primary operator-facing Python worker shape because it is simpler, easier to scaffold, and aligns to the desktop app.

Internal substructure can still exist inside a section, but the visible top-level file shape should use these 8 sections.

The canonical `worker_logic.py` structure is:

1. `Header + Identity + WCP Declarations`
2. `Core Models`
3. `Utility Functions`
4. `Package Attestation + Signature Verification`
5. `Policy Gate (fail-closed)`
6. `Observability`
7. `Business Logic`
8. `CLI + Bootstrap`

### Section-by-section notes

#### SECTION 1: Header + Identity + WCP Declarations

- required for PyHall app
- required for API-only usage
- this is where a human expects to find the top-of-file worker identity and declarations
- should include imports, identity fields, capabilities, controls, and versioning

#### SECTION 2: Core Models

- required for PyHall app
- required for API-only usage
- this defines the minimal request/response or context/result contract

#### SECTION 3: Utility Functions

- required for PyHall app
- optional/minimal for API-only usage
- can be very small
- helper functions can be externalized in API-only runtimes

#### SECTION 4: Package Attestation + Signature Verification

- required for PyHall app
- not strictly required for API-only local file shape
- attestation still matters for governed production use, but the verification step may be externalized to API/runtime services instead of living inside the local worker file

#### SECTION 5: Policy Gate (fail-closed)

- required for PyHall app
- optional in the local worker file for API-only usage if policy gating is handled by the external Hall/router/API layer
- if omitted locally, the external runtime must be the actual fail-closed enforcement point

#### SECTION 6: Observability

- required for PyHall app
- required for API-only usage in some form
- local file implementation can be thinner if evidence/telemetry is emitted by the surrounding runtime instead

#### SECTION 7: Business Logic

- required for PyHall app
- required for API-only usage
- this is the actual worker behavior

#### SECTION 8: CLI + Bootstrap

- required for PyHall app
- is not required for only API usage
- if an external server/runtime invokes the worker directly, this section can be absent or replaced by host-runtime glue

### Identity and declaration example

The canonical identity section should look like this:

```python
WORKER_ID = "org.xyzcompany.rss.instance-1"
WORKER_SPECIES_ID = "wrk.pyhall.rss"
WORKER_NAME = "RSS Intelligence Worker"
WORKER_VERSION = "0.3.0"

CAPABILITIES = [
    "cap.pyhall.rss.read",
    "cap.pyhall.rss.manage",
]

REQUIRED_CONTROLS = [
    "ctrl.obs.audit_log_append_only",
    "ctrl.integrity.package_attested",
    "ctrl.authz.fail_closed_policy_gate",
    "ctrl.trace.correlation_id_required",
]
```

Core model shape should include:

```python
@dataclass
class WorkerContext:
    correlation_id: str
    capability_id: str
    payload: dict[str, Any]
```

### Quick Reference Matrix

| Section | Required for PyHall app | Required for API-only | Can be externalized | Notes |
| --- | --- | --- | --- | --- |
| Header + Identity + WCP Declarations | Yes | Yes | No | Identity and declarations are the minimum contract anchor. |
| Core Models | Yes | Yes | Partially | Caller/runtime may define equivalent models externally, but the contract still has to exist. |
| Utility Functions | Yes | No | Yes | Helpers can live elsewhere. |
| Package Attestation + Signature Verification | Yes | No | Yes | API-only runtimes can call PyHall attestation APIs instead of verifying locally in the file. |
| Policy Gate | Yes | No | Yes | External Hall/router can enforce it. |
| Observability | Yes | Yes | Yes | Evidence and telemetry can be emitted by the host runtime. |
| Business Logic | Yes | Yes | No | This is the worker. |
| CLI + Bootstrap | Yes | No | Yes | Required for app-scaffolded packaged workers; not required for API-only invocation. |

### Full app-path example

For the PyHall app/full package path, package attestation should be explicit, for example:

```python
verifier = PackageAttestationVerifier(
    package_root=package_root,
    manifest_path=manifest_path,
    worker_id=WORKER_ID,
    worker_species_id=WORKER_SPECIES_ID,
)
ok, deny_code, attest_meta = verifier.verify()
```

### API-only example

For the API-only path, a developer may use a thinner local worker file and rely on PyHall APIs from an external runtime, for example:

```python
# SECTION 1: Header + Identity + WCP Declarations
WORKER_ID = "org.xyzcompany.rss.instance-1"
WORKER_SPECIES_ID = "wrk.pyhall.rss"
CAPABILITIES = ["cap.pyhall.rss.read"]

# SECTION 2: Core Models
class WorkerContext: ...
class WorkerResult: ...

# SECTION 7: Business Logic
def execute(context: WorkerContext) -> WorkerResult:
    ...
```

In that model, the external runtime may handle:

- attestation verification
- policy gate enforcement
- evidence emission
- API dispatch

That is valid as long as those responsibilities are actually handled somewhere real.

---

## 7) Authoritative Source Files Right Now

These are the current best authoritative source files for the canonical Python worker model.

### 6.1 Concrete worker package implementation source

Primary concrete implementation source:

- `/mnt/fafolab/dev/2026-432_pyhall/git-working/sdk/python/workers/rss_worker/code/worker_logic.py`

Why:

- full worker package shape
- startup attestation
- fail-closed policy gate
- real worker operations
- evidence / telemetry pattern
- SQLite-backed concrete business behavior

Secondary concrete implementation source:

- `/mnt/fafolab/dev/2026-432_pyhall/git-working/sdk/python/workers/telegram_worker/code/worker_logic.py`

Why:

- same overall WCP worker shape
- broader operation surface
- more complex external API interaction pattern

### 6.2 Canonical attestation implementation source

Attestation authority:

- `/mnt/fafolab/dev/2026-432_pyhall/git-working/sdk/python/pyhall/attestation.py`

Why:

- canonical package hash
- manifest build logic
- signature verification logic
- tenant namespace extraction from `worker_id`
- trust statement semantics

### 6.3 Canonical package scaffold/sign utility source

Packaging/signing companion:

- `/mnt/fafolab/dev/2026-424_fafolab/git-working/docs/directives/examples/pyhall_attest_build_sign.py`

Why:

- useful companion utility for package scaffold, sign, and verify flow
- good operational reference
- not the primary worker implementation itself

### 6.4 Useful but non-canonical reference

Useful design-review reference:

- `/mnt/fafolab/dev/2026-424_fafolab/git-working/docs/directives/examples/artifact_worker_wcp_v030_reference.py`

Why non-canonical:

- it is a strong reference artifact
- but it is explicitly a reference rewrite for design review
- it is not the best “actual shipped worker package” example

---

## 8) Important Current Mismatch

The concrete PyHall worker packages are the right structural source, but some current worker IDs are not yet aligned with the sharpened authority-namespace rule.

Example current pattern found in implementation:

- `WORKER_ID = "pyhall.rss.instance-1"`
- `WORKER_ID = "pyhall.telegram.instance-1"`

That is not the clarified canonical authority form.

Canonical authority form must be:

- `org.<name>.*`
- or `x.<name>.*`

Therefore:

- the concrete package structure is canonical
- the authority-namespace field values still need follow-on alignment

Do not treat old `pyhall.*` worker IDs as the canonical namespace rule.

---

## 9) Canonical Rule For Claude And Future Revisions

If Claude or any future agent needs to explain or implement a Python WCP worker, the order of authority should be:

1. this document
2. `WCP_IDENTITY_NAMESPACE_AND_TAXONOMY_MODEL.md`
3. `sdk/python/pyhall/attestation.py`
4. `sdk/python/workers/rss_worker/code/worker_logic.py`
5. `sdk/python/workers/telegram_worker/code/worker_logic.py`
6. helper/reference examples in `fafolab` directives

That is the correct order.

---

## 10) Bottom Line

The canonical Python WCP worker is:

- usually a full package in the PyHall app path
- allowed to be thinner in the API-only path if runtime responsibilities are externalized explicitly
- attested at package level in the full package path
- fail-closed either locally or in the surrounding runtime
- identity-bound through `worker_id`
- taxonomy-bound through `worker_species_id`
- concretely modeled today by the PyHall Python worker packages

If a future example contradicts this document, this document wins until formally superseded.
