# WCP — Worker Class Protocol

**WCP** is an open standard for governing the dispatch of AI agent workers.

Every major agent protocol — MCP, A2A, ACP, AGNTCY — defines how agents communicate. None define whether a worker should be trusted to execute, under what controls, with what blast radius, and with what evidence. WCP fills that gap.

WCP is additive, not exclusive. MCP handles tool transport. A2A handles agent-to-agent communication. WCP governs the workers those protocols dispatch — it sits beneath the transport as the governance layer that answers: *should this worker be trusted with this job, under these conditions, with this data?*

---

## Specification

The protocol specification is in [`WCP_SPEC.md`](./WCP_SPEC.md).

**Version:** 0.2 — Published
**Status:** Open for implementation and community contribution

---

## Reference Implementation

| Language | Package | Status |
|----------|---------|--------|
| Python | `pip install pyhall-wcp` | Reference implementation — WCP-Full |
| TypeScript | `npm install @pyhall/core` | Full port — WCP-Full |
| Go | `go get github.com/pyhall/pyhall-go@latest` | Scaffold / interfaces |

**PyHall** is the reference implementation: [github.com/pyhall](https://github.com/pyhall) · [pyhall.dev](https://pyhall.dev)

---

## Where WCP Fits

```
Agent reasoning (Claude, GPT, local LLM)
         ↓
WCP Hall  (capability request → governed routing → dispatch)
         ↓
Transport (MCP stdio, HTTP, A2A, direct subprocess)
         ↓
Worker execution
```

WCP answers: *should this worker execute?*
MCP answers: *how does the agent call the tool?*
A2A answers: *how do agents communicate with each other?*

---

## Compliance Levels

| Level | Requirements |
|-------|-------------|
| **WCP-Basic** | Capability routing, fail-closed, deterministic |
| **WCP-Standard** | + Controls enforcement, mandatory telemetry, dry-run |
| **WCP-Full** | + Blast radius, privilege envelopes, policy gate, evidence receipts, discovery API, signatory tenant validation, worker attestation |

---

## Identity and Namespace Model

WCP defines a four-layer identity model that separates capability contracts, worker taxonomy, worker instance identity, and authority namespaces:

| Layer | Prefix | Answers |
|-------|--------|---------|
| **Capability** | `cap.*` | What must be possible? |
| **Worker Species** | `wrk.*` | What class of worker does it? |
| **Worker Identity** | `<authority>.<name>[.<suffix>]` | Which actual worker instance is enrolled? |
| **Authority Namespace** | `org.<name>.*` / `x.<name>.*` | Who stands behind this worker? |

Worker IDs carry the authority namespace that determines trust attribution:

```
org.xyzcompany.doc-summarizer.prod-01   — organization-owned worker
x.joe.local-research-worker.dev-01      — individual-owned worker
```

Two authority namespace families are defined:

- `org.<name>.*` — organizations, companies, teams
- `x.<name>.*` — individuals, personal namespaces

Trust attribution comes from the authority namespace in `worker_id`, never from `worker_species_id` or `capability_id`. Taxonomy membership comes from `worker_species_id`. These are not interchangeable.

The full WCP taxonomy catalog contains 245 entities: 127 capabilities, 48 worker species, 33 controls, 4 policies, and 33 profiles.

For the complete identity model, see [`WCP_IDENTITY_NAMESPACE_AND_TAXONOMY_MODEL.md`](./WCP_IDENTITY_NAMESPACE_AND_TAXONOMY_MODEL.md).

For the canonical Python worker package layout and E2E reference, see [`WCP_PYTHON_WORKER_E2E_CANONICAL_REFERENCE.md`](./WCP_PYTHON_WORKER_E2E_CANONICAL_REFERENCE.md).

---

## Contributing

Contributions to the spec happen through GitHub Issues and pull requests. See [`CONTRIBUTING.md`](./CONTRIBUTING.md).

---

## License

MIT
