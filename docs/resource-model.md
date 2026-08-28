# Resource model

EOKS needs a small vocabulary for resources that persist beyond a single reasoning step. The goal is to distinguish **what a resource means** from **how it is governed or delivered**.

## Core distinction

```text
                       Resource universe
                              |
                  governance / lifecycle
                              |
                           Loadout
                              |
                   context compilation
                              |
                     Compiled context
                              |
                            Agent
```

### Asset

An **asset** is a generic lifecycle/governance abstraction for a reusable resource that may contribute to a workload. It is intentionally not a semantic category.

An asset can be a memory, Skill, document, decision, derived representation, evaluation record, or other reusable resource.

Common asset metadata can include:

- identity and kind;
- provenance;
- scope/applicability;
- freshness/revision;
- ownership and access;
- validation/verification state;
- version/lifecycle state;
- relationships;
- cost characteristics.

**Asset does not mean knowledge.** The abstraction exists so heterogeneous resources can share governance and lifecycle mechanisms without being forced into one representation.

### Semantic types

The meaning of an asset remains important:

```text
Knowledge
  ├── memory / learned fact
  ├── documentation / Wiki
  ├── decision / ADR
  └── invariant / rationale

Procedure
  ├── Skill
  └── workflow / playbook

Evidence / representation
  ├── CodeGraph
  ├── semantic / symbol index
  ├── test result
  ├── static-analysis result
  ├── runtime observation
  └── historical record
```

These categories have different authority, provenance and update semantics even when they share the generic Asset lifecycle.

### Provider

A **provider** is a mechanism that retrieves or derives evidence from a representation or source. Examples include a graph query, code search, static analyzer, test runner or memory retrieval service.

A provider is therefore an implementation mechanism; an asset is a governed resource. A provider can produce evidence without creating a durable asset.

### Representation

A **representation** is a form optimized for a particular query or operation: graph, index, document, timeline, runtime model, and so on.

A representation is not automatically canonical knowledge. Derived representations should normally point back to authoritative sources.

### Loadout

A **loadout** is the workload-scoped set of assets that an agent/task is allowed and expected to use.

Loadout selection is distinct from context selection:

```text
all assets
   ↓
access / ownership / scope / applicability
   ↓
loadout
   ↓
relevance / value / budget
   ↓
compiled context
```

This boundary is useful for security, stale-state control, agent specialization, portability and reproducibility.

### Context

**Context** is the task-specific projection presented to a model for a reasoning step. It is compiled from eligible resources and evidence; it is not the storage layer.

### Context engineering vs context compilation

**Context engineering** is the broader discipline/research area concerned with constructing useful model context.

**Context compilation** is the concrete EOKS operation that selects, transforms, orders, compresses and budgets evidence for a particular reasoning step.

Use the terms this way throughout EOKS. Avoid using "context engineering" and "context compilation" as interchangeable names for the same component.

## Canonical flow

```text
Sources / systems
  code · docs · git · runtime · humans
              |
              v
      representations/providers
              |
              v
        governed assets
              |
      access + scope + task
              |
           loadout
              |
    relevance + budget + policy
              |
      context compilation
              |
       compiled context
              |
             Run
              |
      evaluation / outcome
              |
           learning
              |
     new or updated assets
```

The final feedback loop is the distinctive EOKS concern: resources are not only retrieved; their usefulness is evaluated and that evidence can influence future asset, loadout, context and execution decisions.

## Prior-art mapping

- **TencentDB Agent Memory** is particularly useful prior art for governed reusable assets, multi-resolution memory, Skills, Wiki, CodeGraph and agent loadouts.
- **GrapeRoot** is particularly useful prior art for proactive context compilation and integration around an existing coding agent.
- **OKF** is a candidate portable knowledge representation, not the EOKS resource model itself.
- **Graphify-like tools** provide structural representations/evidence, not a universal knowledge ontology.

These systems can coexist:

```text
Tencent-like assets
        ↓
      loadout
        ↓
GrapeRoot-like context engine
        ↓
     context
        ↓
      agent
```

EOKS remains implementation-agnostic and evaluates the complete workload rather than elevating any one prior-art architecture to the canonical design.
