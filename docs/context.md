# Context engineering

Context engineering is the discipline of constructing the information available to a model for a specific reasoning step.

> **Knowledge is persistent; the working set is workload-specific; context is compiled for a task.**

The concrete EOKS operation is **context compilation**: selecting, transforming, ordering, compressing and budgeting information for a particular reasoning step. See [Resource model](resource-model.md) for the vocabulary boundary between resources, assets, providers, loadouts, working sets and context.

## Context is not storage

```text
engineering reality
       |
knowledge / evidence / execution-state providers
       |
task + workflow + policy
       |
loadout / eligible resources
       |
workload working set
       |
context acquisition + selection
       |
context compiler
       |
model
```

The model should normally receive the evidence it needs, not the internal graph, index or storage system itself.

## Working set, context and execution state

Three concepts must remain distinct:

**Working set** — the workload's currently useful subset of eligible resources/evidence.

**Context** — information presented to the model for the current reasoning step, materialized from the working set.

**Execution state** — durable workflow state describing what has already been observed, established, changed, attempted, verified or invalidated.

**Context acquisition** is the work required to discover or derive information for the working set: search, grep/read exploration, retrieval, graph queries, LSP, analyzers, repository indexes or sub-agents.

```text
eligible resources
       |
  working set
       |
context acquisition/compilation
       |
working context -> reason/act -> update execution state
       |                              |
       +------------------------------+
```

The working set is semantic. A working-set item can be represented as exact source, a structural slice, a verified fact, a summary or a pointer to authoritative evidence. The model context is the current materialization chosen for the reasoning step.

### Context miss

A **context miss** occurs when required evidence is absent from the current working set and must be acquired.

A miss is not necessarily a failure. It is a useful control signal:

```text
context miss
    |
what was requested?
    |
+---+----------------+
|                    |
one-off             recurring
|                    |
fetch               consider
                    prefetch / pin / promote /
                    change representation
```

Repeated misses, excessive acquisition/eviction, repeated re-expansion or low progress despite high context churn may indicate **context thrashing**: the workload spends disproportionate effort reconstructing information instead of making progress.

Thrashing is an observable workload condition, not a new runtime primitive.

## Context as a managed cache

Computer architecture provides a useful lens: model context is a fast, capacity-constrained materialization of the workload's working set. The analogy should generate hypotheses, not dictate implementation.

Candidate techniques include:

- temporal, structural, semantic and workflow locality;
- demand retrieval;
- prefetching;
- cache admission;
- pinning critical evidence;
- LRU/LFU/relevance-aware replacement baselines;
- evidence clustering and batching;
- representation compression/demotion;
- navigation-resolution caching;
- asynchronous acquisition;
- context-thrashing detection.

The objective is not maximum context utilization, minimum tokens or maximum cache hit rate in isolation. The objective is **useful verified work per unit of total reasoning cost**.

## Derived computation and reusable artifacts

Context caching and computation reuse are related but distinct:

```text
context cache
    -> reuses information already made available

incremental semantic computation
    -> reuses derived results so they do not need to be recomputed
```

A computation such as `source -> API graph -> request flow -> authentication flow` can produce a durable, provenance-aware artifact. A later workload can reuse that artifact when its dependencies remain valid, rather than reconstructing the entire derivation from source. If inputs change, dependency information can identify which derived artifacts need revalidation or recomputation.

This follows established incremental-computation ideas such as memoization, dynamic dependency tracking, self-adjusting computation and demand-driven recomputation. See [Incremental semantic computation](../research/prior-art/incremental-semantic-computation.md).

This should **not** turn the context cache into a general-purpose cache of every model state. Model-serving KV caches and other inference intermediates remain a separate implementation-level concern. The EOKS-level object is a reusable derived representation/artifact with provenance, dependencies, validity and verification evidence.

A useful conceptual flow is:

```text
source / evidence
       |
       v
computed semantic artifact
       |
       +--> dependency/provenance record
       |
       v
future context acquisition
       |
   reuse / validate / recompute
       |
       v
compiled context -> reasoning
```

Reuse can be demand-driven: a changed dependency can mark an artifact as potentially stale without immediately recomputing it. Recompute or validation can happen when a workload actually demands the artifact.

The key research question is not cache-hit rate but whether reuse reduces repeated work **without increasing stale-artifact errors or weakening evidence quality**. Relevant measures include recomputation avoided, invalidation precision, freshness, verification effort, latency, cost and end-to-end task outcome.

## Context, acquisition and exploration

Raw exploration is not automatically waste. An agent can use tools to build semantic understanding. EOKS therefore treats retrieval, graphs and other infrastructure as competing interventions rather than assuming they should replace exploration.

The systems lens sharpens this distinction:

- **cache hit** — needed evidence is already resident in the working set;
- **context miss** — needed evidence must be acquired;
- **prefetch** — evidence is acquired based on a prediction of future need;
- **admission** — acquired evidence is considered for retention in the working set;
- **eviction** — evidence is removed or demoted when it is no longer worth resident capacity;
- **pinning** — evidence judged critical is protected from ordinary eviction;
- **thrashing** — acquisition/eviction churn overwhelms useful progress.

These are conceptual labels for candidate policies, not mandatory EOKS components.

## Navigation versus knowledge

Two related but different goals are useful:

1. **Navigation** — determine where relevant authoritative evidence lives and how it is connected.
2. **Knowledge** — preserve information that does not otherwise exist in authoritative sources, such as rationale, invariants, trade-offs and lessons learned.

A code graph is often excellent navigation evidence. It does not automatically become semantic truth. Conversely, a durable invariant may need an explicit knowledge representation even when the relevant source files are easy to find.

Navigation/resolution itself can be cached. A workload should be able to request a logical evidence requirement without knowing which physical provider or storage system will satisfy it.

## Evidence providers

Graphs, semantic indexes, timelines, ADRs, static analysis, tests, runtime observations and knowledge bundles are evidence sources/representations. Providers retrieve or derive evidence from them; context acquisition and compilation select and transform the result.

```text
Task: change authentication
        |
        +--> structural provider -> affected symbols/files
        +--> repository-context provider -> routes/schema/wiki evidence
        +--> knowledge provider  -> relevant invariants/decisions
        +--> history provider   -> incidents/recent changes
        +--> verification       -> tests/static checks
        |
        v
  task working set
        |
  task-specific context
```

EOKS should prefer the cheapest provider that can reliably answer the question, while retaining provenance, scope and freshness with the resulting evidence.

## Context sources are heterogeneous

Memory, Skills, project documentation, decisions, CodeGraph/indexes, test results, runtime observations and historical records can all contribute to context. They should retain their semantic distinctions and authority rather than being flattened into one “memory” store.

These resources can share the generic governance/lifecycle abstraction described in [Resource model](resource-model.md), but **Asset is not a new semantic knowledge category**.

## Asset universe, loadout and compiled context

```text
all reusable resources
          |
 governance: access / ownership / scope / lifecycle / policy
          |
       agent/task loadout
          |
 working-set estimation
          |
 relevance / applicability / value / locality / budget
          |
    context acquisition
          |
    context compilation
          |
     compiled context
```

A **loadout** is the workload-scoped set of assets an agent/task is allowed and expected to use. It is not the final prompt. Context compilation still decides what is useful for the current reasoning step.

A resource can be relevant but inaccessible, out of scope, stale, contradictory, unverified or too expensive for the task.

## Memory hierarchy

EOKS should treat memory/resource hierarchy as a principle rather than a fixed L1/L2/L3 architecture. Different representations may trade off:

- latency;
- capacity;
- persistence;
- freshness;
- authority;
- fidelity;
- retrieval cost;
- transformation cost.

The controller should materialize the least expensive representation sufficient for the current requirement while preserving provenance and authority.

Keep the **semantic/context cache** distinct from the model-serving **KV cache**. The former contains reusable information/evidence and navigation resolutions managed around the workload; the latter is inference state used to avoid recomputation. EOKS may influence KV-cache reuse indirectly through stable context structure, but they are not the same memory abstraction.

## Locality

EOKS can test several forms of locality:

### Temporal locality

Recently useful evidence is more likely to be reused.

### Structural locality

If a workload touches one symbol/file/package, related symbols, callers, dependencies or tests may be more likely to be needed.

### Semantic locality

If reasoning concerns an invariant or decision, related evidence is more likely to be useful.

### Workflow locality

The outcome of one step can predict the evidence required by the next step.

Locality provides a principled basis for prefetching, clustering, pinning and retention decisions. It must be measured rather than assumed.

## Proactive, reactive and hybrid delivery

Context engines can differ in when they make information available.

### Proactive

```text
Task -> retrieval/packing -> working set/context -> model
```

### Reactive

```text
Task -> model -> query/context miss -> evidence -> model -> query ...
```

### Hybrid

```text
compact bootstrap
      |
      +--> memory recall
      +--> Skill retrieval/execution
      +--> Wiki drill-down
      +--> CodeGraph queries
      +--> repository-context queries
      |
      v
working set -> minimum sufficient task context
```

The OS analogy suggests a further distinction: proactive acquisition is a **prefetch hypothesis**. Its value depends on prediction accuracy and acquisition cost. EOKS should not assume proactive delivery is superior to reactive exploration.

Compare proactive, reactive and hybrid strategies by task outcome, evidence coverage, discovery work, context growth, miss rate, churn, latency and cost.

## Context quality

Context quality should consider relevance, correctness, freshness, completeness, redundancy, contradiction risk, provenance, ordering, dependency completeness, acquisition cost and token/latency cost. Add working-set health measures where possible:

- context/working-set hit and miss rate;
- repeated retrievals;
- eviction/reacquisition churn;
- pinned/resident critical evidence;
- acquisition-to-progress ratio;
- stale or contradicted resident evidence.

The goal is not maximum information but maximum useful evidence per unit of total reasoning cost.

Context should be represented conceptually as **inspectable blocks** rather than an opaque prompt. Blocks can represent constraints, knowledge, decisions, structural evidence, tests, runtime observations, history or working hypotheses.

See [Context Workbench](context-workbench.md) and [Context quality model](../research/context-quality-model.md).

## Progressive disclosure and exploration

Prefer progressive disclosure over indiscriminate repository-wide context stuffing when experiments show that it preserves understanding:

```text
compact pointer / structural answer
        |
        +--> authoritative source
        +--> related evidence
        +--> deeper graph/analyzer query
```

However, exploration itself can be useful semantic work. EOKS should measure whether an intervention reduces *avoidable* discovery without removing the investigation needed to understand unfamiliar code.

Different workflow nodes can request different context packages—for example discovery, implementation and verification. Fresh subagents should receive an explicit starting contract when evidence exists, while retaining the ability to investigate beyond it.

## Canonical project knowledge

Hierarchical `CLAUDE.md`-style files are one possible representation for concise, human-reviewed local guidance:

```text
/
  CLAUDE.md
  api/
    CLAUDE.md
  auth/
    CLAUDE.md
```

They are useful for responsibilities, boundaries, invariants, entry points, pitfalls and links to decisions. They should not become encyclopedias or duplicate source code.

**OKF is one portable representation, not the EOKS knowledge layer itself.** The canonical OKF details and interoperability rules belong in knowledge-representation research.

```text
OKF / CLAUDE.md / ADRs / other durable representations
                         |
                         v
                 knowledge providers
                         |
                         v
                  context compiler
```

## Context engines and lifecycle hooks

A context engine can prepare information around an existing coding agent, while repository evidence providers can remain queryable during execution. EOKS should define agent-neutral lifecycle semantics rather than depend on one runtime:

```text
prepare / acquire context
      |
      v
agent execution
      |
tool execution / evidence requests
      |
observe outcome
      |
update execution state / working set
      |
finalize / persist candidates
```

Adapters can map these semantics to hooks, MCP or another runtime. Session boundaries/compaction are opportunities to reconstruct working context, not necessarily the persistence mechanism itself.

## Knowledge lifecycle

Execution can produce candidate facts, decisions, rules, skills and episodes. They should not automatically become canonical knowledge:

```text
observation
    -> candidate extraction
    -> provenance + validation
    -> promote / update / invalidate
    -> future retrieval
```

This separates observation from durable truth and prevents an incorrect session summary from silently becoming future context.

## Model and repository dependence

Context policies should be evaluated across models and repository classes. An intervention may be unnecessary for a frontier model but useful for a cheaper model, or valuable on a legacy repository but redundant on a clean AI-native one.

Therefore EOKS should evaluate:

```text
model × repository × task × intervention × budget
```

rather than treating context engineering as a model-independent optimization.

## Relationship to compaction and model routing

Conversation compaction preserves useful information inside a continuing conversation. Context compilation reconstructs task-specific context from durable knowledge and authoritative evidence. Execution state records what the workflow has already established. Working-set management decides what should remain readily available across reasoning steps. These are related but different operations.

Model routing chooses **which model** to use; context compilation chooses **what information** to give it. Both should be evaluated independently before optimizing them jointly.

## Evaluation boundary

Context interventions must be evaluated on end-to-end task outcomes, not only retrieval, cache or token metrics. The canonical methodology and benchmark matrix live in [Context evaluation and benchmarking](../research/context-evaluation.md).
