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

Context state should be treated as a versioned artifact rather than an opaque prompt. A lifecycle transition can classify information as:

- **active** — needed for current reasoning;
- **supporting** — useful background for the current workload;
- **dormant** — not currently resident but cheaply recoverable;
- **historical** — retained primarily to explain how the current state was reached;
- **superseded** — replaced by newer information but retained for provenance;
- **invalidated** — known to be false or no longer applicable;
- **expired** — no longer trusted or useful under its retention policy.

These are lifecycle states, not necessarily storage tiers.

### State versus relationship

A lifecycle state describes an item's current standing. It does not fully explain **why** that state changed or how it relates to other evidence. Preserve sparse, explicit relationships where they carry explanatory value:

```text
E2 --supersedes--> E1
E3 --contradicts--> E2
E4 --supports-----> E2
E5 --merges-------> E2,E4
```

This distinction is important. EOKS does not need a universal knowledge graph; it needs enough relationship information to preserve causal continuity, contradiction handling and evidence lineage. Recent lifecycle-aware memory work such as MemoryLACE provides useful prior art for this pattern.

## Lossless lineage and derived context

Context evolution may create summaries, abstractions or other derived representations. Those representations should not silently become the only source of truth when the underlying evidence matters.

A useful model is:

```text
raw experience / evidence
          |
          +---- authoritative source
          |
          v
   derived representation
   (summary / abstraction / context state)
          |
          v
      active context
```

The derived representation should retain a recoverable pointer to its source when reconstruction or verification may require it. LCM (Lossless Context Management) and `lossless-claw` are useful prior art for this lineage boundary: hierarchical summaries can remain compact while original messages remain recoverable.

This does not require EOKS to adopt LCM as an implementation. The semantic requirement is **recoverability of important provenance**, not a particular summary algorithm or storage engine.

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

LCM is useful prior art for the first problem. It does not replace the second: lossless compaction preserves recoverability, while context evolution decides what is currently relevant and why.

## Context evolution versus caching

Caching asks whether previously acquired information or derived artifacts can be reused. Evolution asks whether the active working state should change.

```text
cache hit       -> reuse evidence
cache miss      -> acquire evidence
context delta   -> change active state
context compile -> materialize next reasoning context
```

Caching policies such as admission, pinning, demotion and TTL can support evolution, but cache efficiency is not the objective. EOKS should optimize useful verified work per unit of total reasoning cost.

QMD is useful prior art for sophisticated local retrieval, while OpenClaw's retirement of its QMD integration is a reminder that retrieval engines should remain replaceable rather than becoming the architectural memory boundary.

## Utility feedback

Selection should eventually learn from what actually helped, not only what looked relevant at retrieval time.

```text
candidate evidence
      |
      v
selected context
      |
      v
agent run
      |
      v
outcome
      |
      v
utility evidence
      |
      +--> selection/ranking update
      +--> lifecycle reinforcement/demotion
      +--> missing-evidence signal
```

Hindsight Memory-PRM is useful prior art for making this distinction explicit. EOKS should preserve enough run/context attribution to support experiments that distinguish retrieval relevance from actual downstream contribution.

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
- lineage/recoverability of derived representations;
- lifecycle relationship accuracy;
- context-selection utility and attribution quality;
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

A second experiment can compare retrieval ranking with utility-aware selection: measure whether evidence that appears relevant actually improves downstream engineering outcomes.

## Prior art and research hypothesis

Recent agent-memory research is converging on related operations: consolidation, updating, indexing, forgetting, retrieval and condensation. Relevant EOKS prior art now includes MemoryOS, MemGPT/Letta, Hindsight, Graphiti/Zep, Mem0, MemoryLACE, LCM/lossless-claw, OpenClaw built-in Memory and structured-memory tooling such as Obsidian/QMD. Hindsight Memory-PRM adds an important evaluation dimension: memory utility should be connected to downstream outcomes rather than inferred from retrieval alone.

These systems demonstrate pieces of the lifecycle, but EOKS's proposed boundary remains broader: **context evolution coordinates retained knowledge/experience with task state, context compilation, execution and evaluation**.

These references are evidence and design inputs, not EOKS dependencies. The falsifiable question is whether an explicit evolving context state improves long-horizon software-engineering outcomes enough to justify its complexity.

## Architectural status

This is a current architectural hypothesis, not a claim that EOKS requires a dedicated database or runtime component called `ContextEvolution`.

The implementation may be distributed across memory management, context compilation, execution-state tracking and evaluation. The important contract is semantic: **EOKS must be able to update what is actively relevant without retaining the entire session, while preserving provenance, recoverable lineage and sparse lifecycle relationships for important decisions and directions, and learning from downstream utility when evidence is selected.**
