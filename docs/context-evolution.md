# Context evolution

Context engineering constructs the information available to a model for a reasoning step. **Context evolution** is the lifecycle that determines how the workload's active working state changes as new evidence, decisions, failures, corrections and outcomes arrive.

> **Remember the work, not the transcript.**

The goal is not to preserve an ever-growing session history. It is to keep a compact, inspectable representation of what matters now while preserving enough provenance and causal history to explain how the current state was reached.

## Why this is distinct from context compilation

EOKS already treats context as a task-specific projection and context compilation as the operation that selects, transforms, orders, compresses and budgets information for a reasoning step. Context evolution addresses a different question:

> **What should the workload's active state be now, given what has changed since the previous state?**

```text
previous context state
        |
        | new evidence / actions / outcomes
        v
   context delta
        |
        +--> activate / reinforce
        +--> demote / decay
        +--> supersede / invalidate
        +--> synthesize / consolidate
        +--> preserve causal history
        |
        v
current context state
        |
        v
context compilation -> next reasoning step
```

Compilation is therefore primarily a **projection problem**. Evolution is a **state-transition problem**. They should remain conceptually separate even when implemented by the same component.

## What should survive a long session?

A useful long-lived working state is not a transcript. It is a compact model of the work:

- current goal and direction;
- active decisions and their rationale;
- open questions and unresolved hypotheses;
- constraints and invariants;
- high-value evidence supporting the current direction;
- important failed or rejected alternatives;
- dependencies and relevant project state;
- recent actions and outcomes that affect the next step;
- a causal spine explaining why the current state exists.

Old conversation turns can become inactive without becoming inaccessible. Historical evidence remains recoverable when verification or reconstruction requires it.

This gives EOKS a useful distinction:

```text
active relevance != historical importance
```

An item can be dormant for current reasoning while still being important because it explains a decision, records a failure, or provides provenance.

## Context state and lifecycle

Context state should be treated as a versioned artifact rather than an opaque prompt:

```text
Context v17

ACTIVE
  decision: use approach B
  question: determine whether X works
  constraint: Y cannot change

CHANGED SINCE v16
  + evidence E
  + decision B
  - hypothesis A
  ~ constraint Y clarified

CAUSAL SPINE
  B <- evidence E
  B <- rejection(A)
  B <- constraint(Y)

DORMANT
  C
  previous investigation D
```

A context transition can classify information as:

- **active** — needed for current reasoning;
- **supporting** — useful background for the current workload;
- **dormant** — not currently resident but cheaply recoverable;
- **historical** — retained primarily to explain how the current state was reached;
- **superseded** — replaced by newer information but retained for provenance;
- **invalidated** — known to be false or no longer applicable;
- **expired** — no longer trusted or useful under its retention policy.

These are lifecycle states, not necessarily storage tiers.

## Evolution signals

TTL is only one signal. Admission, retention, demotion and invalidation should consider multiple forms of evidence:

```text
freshness
+ task affinity
+ recency
+ dependency/locality
+ explicit decisions
+ repeated usefulness
+ outcome usefulness
+ contradiction
+ authority / verification
+ acquisition cost
        |
        v
context state transition
```

A TTL can reduce active eligibility without deleting the underlying evidence. New evidence or renewed task relevance can reactivate it. Conversely, an explicit contradiction or invalidation can demote information immediately even if its TTL has not elapsed.

This is why context evolution should not be reduced to cache eviction.

## Semantic boundaries and consolidation

Updating memory after every message is both noisy and expensive. EOKS should support **semantic boundaries**: points at which a coherent part of the work has changed enough to justify consolidation.

Possible boundaries include:

- a decision being made or reversed;
- a hypothesis being confirmed or rejected;
- a task/workflow stage completing;
- a tool result changing the working hypothesis;
- a verification result invalidating an assumption;
- a meaningful shift in the user's objective or constraints;
- a session compaction/restart requiring reconstruction.

Within a semantic segment, raw observations can remain episodic evidence. At a boundary, EOKS can synthesize the delta into a more durable representation.

```text
exchange -> exchange -> exchange
                    |
             semantic boundary
                    |
          consolidate meaningful delta
                    |
             update context state
```

The system should prefer incremental consolidation over rebuilding the entire knowledge base or summarizing the complete transcript repeatedly.

## Causal continuity

The central requirement for long sessions is not perfect recall. It is **causal continuity**: the system should be able to explain the important transitions that led to the current direction.

A causal spine can be represented as relationships among decisions, evidence, alternatives, constraints and outcomes:

```text
current decision
      |
      +--> supporting evidence
      +--> constraint
      +--> rejected alternative
      +--> prior decision
```

The spine should point to authoritative evidence rather than embedding every underlying detail in active context. When needed, the compiler can expand a pointer into source, history, tests or other evidence.

This supports progressive disclosure: preserve the compact reason in active state and reacquire detailed evidence only when the workload requires it.

## Relationship to memory

Memory and context evolution are related but not identical.

- **Memory** governs information retained for future work: working, episodic, semantic, project, procedural and other semantic categories.
- **Context state** describes what is currently relevant to the workload and why.
- **Compiled context** is the materialized information supplied to one reasoning step.
- **Execution state** records what the workflow has established, changed, attempted and verified.

A useful loop is:

```text
observe
  -> retain evidence / episode
  -> detect meaningful change
  -> reflect / synthesize
  -> update context state
  -> compile next context
  -> execute
  -> evaluate outcome
  -> revise / invalidate / promote
```

Not every context change becomes durable memory, and not every durable memory belongs in the active context.

## Context evolution versus compaction

Conversation compaction attempts to preserve useful information when a model-facing conversation exceeds its context budget. Context evolution is broader and can operate continuously during a workload.

```text
compaction
  = compress an existing conversation/state representation

context evolution
  = change the representation of what matters as the work changes
```

Compaction can be one trigger for reconstruction. It is not the persistence model.

## Context evolution versus caching

Caching asks whether previously acquired information or derived artifacts can be reused. Evolution asks whether the active working state should change.

```text
cache hit       -> reuse evidence
cache miss      -> acquire evidence
context delta   -> change active state
context compile -> materialize next reasoning context
```

Caching policies such as admission, pinning, demotion and TTL can support evolution, but cache efficiency is not the objective. EOKS should optimize useful verified work per unit of total reasoning cost.

## Evaluation

Context evolution should be evaluated by downstream workload outcomes, not by summary quality alone.

Useful measures include:

- context reconstruction accuracy after compaction or session restart;
- decision/rationale retention;
- stale or contradicted information remaining active;
- unnecessary historical material retained in active context;
- context churn and repeated reacquisition;
- semantic-boundary detection quality;
- causal-spine usefulness for verification and recovery;
- token, latency and acquisition cost;
- task quality, reliability and completion outcomes;
- ability to recover the required evidence when dormant information becomes relevant.

A particularly useful experiment is to compare:

```text
full transcript
vs
periodic summary
vs
evolving context state + causal spine + on-demand evidence
```

while holding model, task and budget conditions constant.

## Prior art and research hypothesis

Recent agent-memory research is converging on related operations: consolidation, updating, indexing, forgetting, retrieval and condensation; hierarchical memory architectures distinguish short-lived interaction state from more durable abstractions; reflective memory systems distinguish prospective decisions about what to retain from retrospective revision of earlier memory; and recent work explores semantic-segment consolidation to reduce the cost and noise of turn-by-turn memory construction.

Relevant EOKS prior art includes MemoryOS, MemGPT/Letta, Hindsight, Graphiti/Zep, Mem0, reflective memory management and semantic-segment consolidation. These systems demonstrate pieces of the lifecycle, but EOKS's proposed boundary remains broader: **context evolution coordinates retained knowledge/experience with task state, context compilation, execution and evaluation**.

These references are evidence and design inputs, not EOKS dependencies. The falsifiable question is whether an explicit evolving context state improves long-horizon software-engineering outcomes enough to justify its complexity.

## Architectural status

This is a current architectural hypothesis, not a claim that EOKS requires a dedicated database or runtime component called `ContextEvolution`.

The implementation may be distributed across memory management, context compilation, execution-state tracking and evaluation. The important contract is semantic: **EOKS must be able to update what is actively relevant without retaining the entire session, while preserving provenance and causal continuity for important decisions and directions.**
