# Phase A tool landscape update

This addendum records the current Phase A positions after the latest tool review. It is intentionally additive so that the richer historical tool landscape is not removed or silently rewritten.

## Current Phase A positions

| Tool / mechanism | Phase A position | Rationale |
|---|---|---|
| **OpenWolf mechanisms** | **Core** | Strongest current prior art for harness-native lifecycle/state/hooks, session handoff, pre-compaction preservation, and measured token usage. Use mechanisms, not the whole product. |
| **Spotify Portal/Shunt mechanisms** | **Core** | Adds a complementary harness intervention point: intercept expensive context-producing operations and transform/delegate them before the frontier model. Use selectively rather than adopting the whole stack. |
| **Understand Anything** | **Core representation experiment** | Promoted because it is useful prior art for semantic repository/code representation and visualization. The experiment must explicitly test polyglot and multi-repository limits rather than assuming repository-local graphs solve system representation. |
| **Obsidian** | **Phase A companion** | Human-facing inspection, curation and visualization surface. Consume EOKS artifacts/Markdown; do not put it in the agent execution loop. |
| **GrapeRoot** | **Reference / prior art** | Valuable example of proactive context compilation, but the critical graph engine is proprietary and therefore weak as a first implementation substrate. |
| **Hindsight / LangMem-style memory** | **Phase B** | Targets durable/evolving memory. Defer until current-session context lifecycle and harness intervention are understood. |
| **Opik / hosted observability** | **Phase B** | Useful once experiments become complex enough to need richer tracing/evaluation. Start with minimal local measurements that EOKS fully controls. |

## Relationship changes

The important change is that these tools should not be treated as competing product categories:

```text
OpenWolf mechanisms
    lifecycle / state / measurement
             │
             ▼
       EOKS harness
             ▲
             │
Shunt mechanisms
    intercept / transform / delegate

Understand Anything
    repository/code representation
             │
             ▼
    system-representation experiment
             │
             ▼
Obsidian
    human inspection / curation
```

The harness and representation experiments should remain independently replaceable.

## New unresolved capability

The previous **Repository structure** category is too narrow for the Phase A question. EOKS needs to distinguish:

1. **repository-local representation** — files, symbols, calls, semantic relationships;
2. **cross-repository representation** — ownership and relationships across repository boundaries;
3. **system representation** — services, APIs, events, datastores and deployment/runtime relationships;
4. **end-to-end representation** — navigable paths across those layers.

Understand Anything is a Phase A provider for (1) and potentially parts of (2). It should not be assumed to solve (3) or (4).

The missing capability is tracked in [`research/system-representation-gap.md`](../research/system-representation-gap.md).

## Tool-selection rule for Phase A

Do not choose a tool because it has the richest graph or UI. Choose a provider based on the evidence it contributes to a concrete engineering question and whether that evidence can be composed with other providers.

The desired architecture is:

```text
provider(s)
    ↓
representation / evidence
    ↓
EOKS selection + context policy
    ↓
agent
```

not:

```text
tool
    ↓
canonical EOKS knowledge graph
```
