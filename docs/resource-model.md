# Resource model

EOKS needs a small vocabulary for resources that persist beyond a single reasoning step. The goal is to distinguish **what a resource means** from **how it is governed, represented or delivered**.

The resource model is the lower-level complement to the EOKS control loop. Operating-system and computer-architecture concepts suggest useful questions about resource hierarchy, locality, access cost, capacity, isolation and lifecycle, but do not impose a literal OS implementation.

## Core distinction

```text
                       Resource universe
                              |
                  governance / lifecycle
                              |
                     workload loadout
                              |
                   workload working set
                              |
                   context compilation
                              |
                     Compiled context
                              |
                            Run
```

The important distinction is:

- **Loadout** defines the workload-scoped resource namespace and eligibility boundary.
- **Working set** is the subset of eligible resources/evidence currently useful for progress.
- **Context** is a task-specific materialization of that working set for a reasoning step.

## Asset

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

## Semantic types

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
  ├── structural graph / repository map
  ├── semantic / symbol index
  ├── test result
  ├── static-analysis result
  ├── runtime observation
  └── historical record
```

These categories have different authority, provenance and update semantics even when they share the generic Asset lifecycle.

A **structural graph is a representation, not an EOKS primitive**. Graph-based tools such as Graphify, CodeGraph and GitNexus are alternative providers for this representation/evidence family. They may also perform higher-level analysis or context delivery, but those capabilities should remain conceptually separate from the underlying representation.

## Provider

A **provider** is a mechanism that retrieves or derives evidence from a representation or source. Examples include a graph query, code search, static analyzer, test runner or memory retrieval service.

A provider is therefore an implementation mechanism; an asset is a governed resource. A provider can produce evidence without creating a durable asset.

Provider resolution is intentionally hidden behind a logical resource/evidence requirement where possible:

```text
logical requirement
       |
 provider resolution
       |
 representation/source
       |
 evidence
```

This is analogous to virtual resource addressing: the workload should not need to know where information is physically stored or which provider implements the request. Resolution itself can be cached when repeated navigation is expensive.

## Representation

A **representation** is a form optimized for a particular query or operation: graph, index, document, timeline, runtime model, and so on.

A representation is not automatically canonical knowledge. Derived representations should normally point back to authoritative sources.

For software engineering, a structural graph is one particularly useful representation because it makes relationships such as imports, calls, inheritance, dependencies, flows and impact explicit. It is **not required by EOKS**, and should be selected only when the workload benefits from graph-shaped evidence.

The distinction is:

```text
source / canonical knowledge
            |
            v
     representation
   (graph, index, wiki...)
            |
            v
       evidence
            |
            v
   working set / context compilation
```

See [Engineering knowledge as a multi-representation system](knowledge-representations.md) for the canonical discussion of representation families and their trade-offs.

## Loadout

A **loadout** is the workload-scoped set of assets/resources that an agent/task is allowed and expected to use.

The OS/resource-protection analogy suggests treating this as a **resource namespace and eligibility boundary**, not merely as a prompt package. It can express readable, writable, executable, derived, restricted or approval-gated resources where policy requires it.

Loadout selection is distinct from working-set/context selection:

```text
all resources
   ↓
access / ownership / scope / applicability / policy
   ↓
loadout
   ↓
working-set estimation
   ↓
relevance / value / budget / locality
   ↓
compiled context
```

This boundary is useful for security, stale-state control, agent specialization, portability and reproducibility.

## Working set

A **working set** is the workload's currently useful subset of eligible resources and evidence.

It is not the same as context. A working-set item can be represented as exact source, a structural slice, a verified fact, a summary or a pointer to authoritative evidence. The model context is the current materialization chosen for a reasoning step.

The working set should be allowed to change as the workload changes phase:

```text
understand -> architecture + invariants + relevant code
implement  -> changed symbols + interfaces + tests
verify     -> changed artifacts + failures + invariants + checks
```

This creates a useful control signal. A **context miss** occurs when required evidence is absent from the current working set and must be acquired. Repeated misses, excessive churn or repeated eviction/retrieval cycles can indicate working-set pressure or context thrashing.

## Context

**Context** is the task-specific projection presented to a model for a reasoning step. It is compiled from eligible resources and evidence; it is not the storage layer.

Computer architecture provides a useful lens: context is a capacity-constrained, fast materialization of a workload's working set. Candidate techniques include locality-aware retrieval, demand acquisition, prefetching, cache admission, pinning, replacement, clustering and representation compression/demotion.

These techniques are **research interventions**, not assumptions that a particular cache policy is optimal. The objective is useful verified work per unit of total reasoning cost.

## Context engineering vs context compilation

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
     working-set estimation
              |
    relevance + locality + policy
              |
      context acquisition
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

The final feedback loop is the distinctive EOKS concern: resources are not only retrieved; their usefulness is evaluated and that evidence can influence future asset, loadout, working-set, context and execution decisions.

## Memory hierarchy and representations

EOKS should treat resource hierarchy as a **principle**, not a fixed L1/L2/L3 architecture. Different representations can trade off:

- latency;
- capacity;
- persistence;
- freshness;
- authority;
- fidelity;
- retrieval cost;
- transformation cost.

The controller/context system should materialize the least expensive representation that is sufficient for the current requirement, while preserving provenance and authority.

This is distinct from the model-serving **KV cache**, which is inference state. EOKS's semantic/context cache may influence KV-cache reuse indirectly but should not be collapsed into the same abstraction.

## Locality and optimization hypotheses

EOKS can test several kinds of locality:

- **temporal** — recently useful evidence is likely to be reused;
- **structural** — related symbols/files/dependencies are likely to be accessed together;
- **semantic** — related invariants, decisions and evidence are likely to co-occur;
- **workflow** — one step's result predicts the next step's evidence needs.

Candidate optimization techniques include:

- demand retrieval/context misses;
- prefetching;
- cache admission;
- pinning critical evidence;
- LRU/LFU/relevance-aware replacement baselines;
- evidence clustering and batching;
- representation compression/demotion;
- navigation-resolution caching;
- asynchronous acquisition;
- context-thrashing detection.

These should be compared using end-to-end workload outcomes rather than token savings or cache statistics alone.

## Shared state and lifecycle

Multiple agents/workflow phases may share authoritative evidence while keeping derived or tentative state local:

```text
canonical resource
      |
  shared read
      |
 local derived state
      |
 candidate -> validate -> promote/update/invalidate
```

This is a copy-on-write-like design principle. It supports provenance and avoids uncontrolled shared mutable memory without introducing a mandatory multi-agent memory implementation.

## Prior-art mapping

- **TencentDB Agent Memory** is particularly useful prior art for governed reusable assets, multi-resolution memory, Skills, Wiki, CodeGraph and agent loadouts.
- **GrapeRoot** is particularly useful prior art for proactive context compilation and integration around an existing coding agent.
- **OKF** is a candidate portable knowledge representation, not the EOKS resource model itself.
- **Graphify, CodeGraph and GitNexus** are representative structural-graph/evidence providers. They overlap at the graph substrate while differing in graph construction, resolution, precomputed analysis, provenance and agent-facing delivery.
- Recent systems research independently explores multi-agent memory as a computer-architecture problem and LLM context as a demand-paged hierarchy. See [OS and computer-architecture lens](../research/prior-art/computer-systems-architecture.md).

These systems can coexist:

```text
Tencent-like assets
        ↓
      loadout
        ↓
 working set / context engine
        ↑
 structural evidence providers
 Graphify / CodeGraph / GitNexus
        ↓
     context
        ↓
      agent
```

EOKS remains implementation-agnostic and evaluates the complete workload rather than elevating any one prior-art architecture to the canonical design.
