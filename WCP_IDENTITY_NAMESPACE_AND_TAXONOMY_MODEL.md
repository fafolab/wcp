# WCP Identity, Namespace, and Taxonomy Model

**Status:** active clarification document  
**Written:** 2026-03-14 15:55 CDT  
**Purpose:** define the WCP naming model cleanly so capability, worker species, worker identity, and authority namespace are not conflated.

---

## Why this document exists

WCP has multiple identifier layers, and they answer different questions:

- what must be possible
- what class of worker implements that thing
- which specific worker is enrolled
- who stands behind that worker

If those get blurred together, routing language gets sloppy, attestation language gets wrong, and QA/QC starts documenting lies.

This document exists to stop that drift.

---

## The short version

- `cap.*` = what must be possible
- `wrk.*` = what class/species of worker does it
- `worker_id` = which actual worker instance is enrolled
- authority namespace = either `org.<name>.*` or `x.<name>.*`
- trust attribution comes from `worker_id`
- taxonomy comes from `worker_species_id`

---

## The four identity layers

## 1. Capability identity

Capability IDs answer:

> What must be possible?

Format:

```text
cap.<domain>[.<subdomain>].<verb-or-contract>
```

Examples:

```text
cap.doc.summarize
cap.mem.retrieve
cap.web.fetch
cap.notify.email.send
```

Meaning:

- capabilities are stable contracts
- capabilities are verbs or action contracts
- capabilities do not identify a specific worker

## 2. Worker species identity

Worker species IDs answer:

> What class/type of worker implements this capability?

Format:

```text
wrk.<domain>[.<subdomain>].<role>
```

Examples:

```text
wrk.doc.summarizer
wrk.doc.pdf.extractor
wrk.mem.retriever
wrk.web.fetcher
wrk.sec.artifact.verifier
```

Meaning:

- a worker species is a taxonomy identifier
- it describes the type/class of worker
- it is not a trust namespace
- it is not an owner identity

## 3. Worker instance identity

Worker IDs answer:

> Which actual worker instance is enrolled?

Format:

```text
<authority-namespace>.<worker-name>[.<instance-suffix>]
```

Examples:

```text
org.xyzcompany.doc-summarizer.prod-01
org.abcdevelopmentcorp.registry-manager.prod-02
x.joe.local-research-worker.dev-01
x.lisajohnson.mem-curator.laptop-01
```

Meaning:

- `worker_id` identifies an actual enrolled worker instance
- it is the identity used in audit records, enrollment, and trust attribution

## 4. Authority namespace

Authority namespace answers:

> Who stands behind this worker identity?

In WCP, an authority namespace is one of these:

- `org.<name>.*`
- `x.<name>.*`

There is no third authority form defined here.

That means:

- organization authority namespace = `org.<name>.*`
- individual authority namespace = `x.<name>.*`

---

## Protocol namespaces vs authority namespaces

WCP uses two different namespace families.

## Protocol namespaces

These are reserved WCP language namespaces:

- `cap.*`
- `wrk.*`
- `ctrl.*`
- `pol.*`
- `prof.*`
- `evt.*`

They are used for:

- routing contracts
- worker taxonomy
- control catalog
- policy catalog
- profile catalog
- telemetry/event semantics

They are not used for signer trust attribution.

## Authority namespaces

These identify the owner/authority behind the worker:

- `org.<name>.*`
- `x.<name>.*`

They are used for:

- `worker_id`
- namespace ownership
- worker enrollment identity
- attestation trust attribution

They are the source of authority meaning.

---

## Authority namespace definitions

## `org.<name>.*`

Use for:

- companies
- organizations
- labs
- teams
- business entities
- official org-owned worker fleets

Format:

```text
org.<name>.*
```

Examples:

```text
org.xyzcompany.*
org.abcdevelopmentcorp.*
org.pyhall.*
org.workerclassprotocol.*
```

Example worker IDs:

```text
org.xyzcompany.doc-summarizer.prod-01
org.abcdevelopmentcorp.policy-gate.us-east-01
```

## `x.<name>.*`

Use for:

- individuals
- personal namespaces
- solo developers
- named human operators
- personal worker fleets

Format:

```text
x.<name>.*
```

Examples:

```text
x.joe.*
x.lisajohnson.*
x.robkennedy.*
x.samirpatel.*
```

Example worker IDs:

```text
x.joe.local-research-worker.dev-01
x.lisajohnson.web-fetcher.laptop-01
```

Important clarification:

- `x.*` in this model means individual/personal authority
- it does **not** mean “experimental”

---

## What trust is based on

Attestation and trust attribution must be based on the authority namespace carried by `worker_id`.

Not:

- `worker_species_id`
- `capability_id`
- any `wrk.*` prefix

Correct:

```text
worker_id = org.xyzcompany.doc-summarizer.prod-01
worker_species_id = wrk.doc.summarizer
capability_id = cap.doc.summarize
```

Trust attribution:

- authority namespace = `org.xyzcompany`

Taxonomy:

- species = `wrk.doc.summarizer`

Contract:

- capability = `cap.doc.summarize`

That is the separation.

---

## What each field is for

## `capability_id`

Use for:

- routing intent
- capability discovery
- policy matching
- contract definition

Do not use for:

- ownership
- worker identity
- trust attribution

## `worker_species_id`

Use for:

- worker class/species selection
- routing outputs
- taxonomy browsing
- capability-to-implementation mapping
- execution-path documentation

Do not use for:

- authority attribution
- ownership proof
- signer identity

## `worker_id`

Use for:

- enrollment identity
- authority attribution
- audit trail identity
- instance-level tracing
- attestation trust context

Do not use for:

- replacing species taxonomy semantics that belong in `worker_species_id`

---

## Correct and incorrect reasoning

## Correct

```text
This worker is trusted under org.xyzcompany because its worker_id is
org.xyzcompany.doc-summarizer.prod-01.
Its species is wrk.doc.summarizer.
It implements cap.doc.summarize.
```

## Correct

```text
This worker is trusted under x.joe because its worker_id is
x.joe.local-research-worker.dev-01.
Its species is wrk.mem.retriever.
It implements cap.mem.retrieve.
```

## Incorrect

```text
This worker is trusted because its species starts with wrk.doc.
```

Wrong because:

- `wrk.*` is taxonomy, not authority

## Incorrect

```text
This worker belongs to xyzcompany because it implements cap.doc.summarize.
```

Wrong because:

- capability does not identify owner or signer

## Incorrect

```text
This worker's namespace is wrk.doc.summarizer.
```

Wrong because:

- that is a worker species ID, not an authority namespace

---

## Namespace registration and reservation through PyHall

If users utilize the PyHall service, they can register or reserve an authority namespace so that another user cannot use that same authoritative namespace through the PyHall service.

That means PyHall can govern authority namespace ownership for:

- `org.<name>.*`
- `x.<name>.*`

Examples:

- if `org.xyzcompany.*` is registered or reserved through PyHall, another PyHall user should not be able to enroll workers under `org.xyzcompany.*`
- if `x.lisajohnson.*` is registered or reserved through PyHall, another PyHall user should not be able to enroll workers under `x.lisajohnson.*`

Important scope note:

- this is a PyHall service-level namespace ownership and enforcement model
- it does not magically grant global internet-wide ownership outside the PyHall ecosystem
- it means that within PyHall-governed enrollment and attestation, that authority namespace is protected

---

## The current PyHall/WCP taxonomy count

The current taxonomy catalog shipped with the PyHall SDKs contains `245` total entities.

That does **not** mean “245 worker species.”

Current breakdown:

- `127` capabilities
- `48` worker species
- `33` controls
- `4` policies
- `33` profiles

So when someone says “the taxonomy has 245 things,” they mean the full WCP catalog, not only `wrk.*`.

---

## Full Python worker example

Below is a complete Python-oriented example showing the separation between authority namespace, worker identity, worker species, and capability.

```python
from pathlib import Path

from pyhall.attestation import (
    scaffold_package,
    build_manifest,
    write_manifest,
    PackageAttestationVerifier,
)

# Authority namespace: organization-owned fleet
AUTHORITY_NAMESPACE = "org.xyzcompany"

# Actual worker instance identity
WORKER_ID = "org.xyzcompany.doc-summarizer.prod-01"

# Taxonomy class/species
WORKER_SPECIES_ID = "wrk.doc.summarizer"

# Contract/capability
CAPABILITY_ID = "cap.doc.summarize"

# Build package layout
scaffold_package(Path("doc-summarizer-worker/"))

# Create manifest for this specific worker instance
manifest = build_manifest(
    package_root=Path("doc-summarizer-worker/"),
    worker_id=WORKER_ID,
    worker_species_id=WORKER_SPECIES_ID,
    worker_version="1.0.0",
    signing_secret="replace-with-real-signing-secret",
)
write_manifest(manifest, Path("doc-summarizer-worker/manifest.json"))

# Verify package at runtime
verifier = PackageAttestationVerifier(
    package_root=Path("doc-summarizer-worker/"),
    manifest_path=Path("doc-summarizer-worker/manifest.json"),
    worker_id=WORKER_ID,
    worker_species_id=WORKER_SPECIES_ID,
)

ok, deny_code, meta = verifier.verify()
if not ok:
    raise SystemExit(f"Attestation denied: {deny_code}")

print("authority namespace:", AUTHORITY_NAMESPACE)
print("worker_id:", WORKER_ID)
print("worker_species_id:", WORKER_SPECIES_ID)
print("capability_id:", CAPABILITY_ID)
print(meta["trust_statement"])
```

What this example shows:

- `org.xyzcompany` is the authority namespace
- `org.xyzcompany.doc-summarizer.prod-01` is the actual worker instance
- `wrk.doc.summarizer` is the species/class
- `cap.doc.summarize` is the capability contract

Those four things are related, but they are not interchangeable.

---

## Recommended shorthand

Use this language consistently:

- capability = what must be done
- worker species = what class of worker does it
- authority namespace = who stands behind it
- worker ID = which actual worker instance is enrolled

Best one-line explanation:

> WCP routes a capability to a worker species, but trusts and attributes that work to the authority namespace carried by the worker ID.

---

## Rules that should never drift

- `cap.*` is not ownership
- `wrk.*` is not authority
- `org.<name>.*` and `x.<name>.*` are authority namespace families
- `x.<name>.*` is for individuals, not “experimental”
- `worker_id` and `worker_species_id` are never interchangeable
- trust attribution always comes from `worker_id`
- taxonomy membership always comes from `worker_species_id`

---

## Bottom line

WCP has a clean multi-layer model:

1. capability contract
2. worker species taxonomy
3. worker identity
4. authority namespace

If those stay separated, the protocol remains coherent.

If those get blurred, routing, trust, documentation, and QA/QC all start lying to each other.
