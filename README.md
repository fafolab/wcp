# WCP — Worker Class Protocol

**WCP** is an open standard for governing the dispatch of AI agent workers.

Every major agent protocol — MCP, A2A, ACP, AGNTCY — defines how agents communicate. None define whether a worker should be trusted to execute, under what controls, with what blast radius, and with what evidence. WCP fills that gap.

WCP is additive, not exclusive. MCP handles tool transport. A2A handles agent-to-agent communication. WCP governs the workers those protocols dispatch — it sits beneath the transport as the governance layer that answers: *should this worker be trusted with this job, under these conditions, with this data?*

---

## Specification

The protocol specification is in [`WCP_SPEC.md`](./WCP_SPEC.md).

**Version:** 0.1 — Published
**Status:** Open for implementation and community contribution

---

## Reference Implementation

| Language | Package | Status |
|----------|---------|--------|
| Python | `pip install pyhall` | Reference implementation — WCP-Full |
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

## Contributing

Contributions to the spec happen through GitHub Issues and pull requests. See [`CONTRIBUTING.md`](./CONTRIBUTING.md).

---

## License

MIT
