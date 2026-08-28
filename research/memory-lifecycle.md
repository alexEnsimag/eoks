# Memory lifecycle

The memory discussion became less about “where do we store embeddings?” and more about the lifecycle of knowledge.

## Candidate lifecycle

```text
observe
  |
  v
candidate knowledge
  |
  +--> discard
  |
  v
validate / enrich provenance
  |
  v
promote to memory
  |
  v
retrieve for future task
  |
  v
use / evaluate
  |
  +--> reinforce
  +--> revise
  +--> supersede
  +--> invalidate
  +--> expire
```

A memory system that only implements `write` and `retrieve` is incomplete.

## Promotion

Not every model output should become memory. Promotion should require signals such as:

- explicit project decision;
- stable external fact;
- validated execution result;
- repeated successful observation;
- important failure and its verified cause;
- human confirmation.

Task-local reasoning and guesses should generally remain ephemeral.

## Retrieval

Retrieval should be task-aware. Similarity alone can return information that is semantically related but operationally wrong or stale.

Candidate retrieval signals:

- semantic relevance;
- dependency relationship;
- recency;
- authority;
- previous usefulness;
- task similarity;
- validity state.

## Contradictions

Memory should allow contradictory claims to coexist temporarily rather than silently merging them.

```text
claim A ---- supports ----> decision X
claim B ---- contradicts -> decision X
                  |
                  v
             unresolved
```

The control plane can decide whether the conflict requires more evidence, a human decision or a new task.

## Supersession

Project knowledge naturally changes. A new architecture decision should not necessarily delete the old decision; it should mark the relationship:

```text
old decision --superseded-by--> new decision
```

Historical context remains useful for understanding why a system looks the way it does.

## Memory and context budgets

Persistent memory can be arbitrarily large; working context cannot. This creates an important two-stage optimization:

```text
large knowledge space
        |
   retrieval/ranking
        |
small working set
        |
 context compilation
        |
      model
```

The hard problem is therefore not storage capacity but **working-set selection**.

## Memory as organizational learning

For recurring engineering workloads, memory could capture things that a stateless agent would rediscover repeatedly:

- project decisions;
- known failure modes;
- successful debugging paths;
- service ownership;
- architectural constraints;
- validated conventions.

This is where memory becomes part of the control loop rather than a chatbot feature.

## Evaluation

A memory system should be evaluated on both positive and negative effects:

- fewer repeated discoveries;
- lower context construction cost;
- higher task success;
- improved consistency;
- stale-memory errors;
- contradiction errors;
- retrieval failures;
- maintenance overhead.

A memory mechanism that improves one task while poisoning future tasks is not an improvement.