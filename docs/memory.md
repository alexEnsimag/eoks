# Memory

EOKS treats memory as a deliberate persistence layer, not simply an ever-growing conversation transcript.

## Useful distinctions

### Working memory
Information needed by the current task or reasoning chain.

### Episodic memory
What happened in a previous interaction or execution: actions, observations, failures and outcomes.

### Semantic memory
Durable facts, concepts, decisions and relationships extracted from experience or external sources.

### Project memory
The evolving state of a codebase/project: architecture decisions, constraints, conventions, known failures and current goals.

### Procedural memory
How work is performed: recurring debugging strategies, decomposition patterns, verification habits, review workflows and successful sequences of actions.

### Policy memory
What should happen. A policy is stronger than an observation and should require evidence, validation and versioning before it influences future execution.

### Preference memory
Human choices that may guide behavior but should not automatically be generalized into engineering rules.

## Memory lifecycle

A memory candidate should have a lifecycle:

`observe -> extract -> validate -> store -> retrieve -> use -> evaluate -> update/expire`

The difficult part is not storage. It is deciding **what deserves persistence**, how confidence and provenance are retained, and how stale or contradictory memories are handled.

For behavioral learning, the lifecycle should be extended with explicit promotion:

`trace -> episode -> pattern candidate -> validate -> learning record -> promote to skill/policy -> evaluate`

An observed behavior should not silently become canonical project policy. Repeated evidence, outcomes and human corrections can strengthen a candidate, while contradictory evidence can keep it scoped or prevent promotion.

## Memory versus context

Memory is a source for future contexts. Context is the task-specific projection of available knowledge. A memory item can be correct but still be inappropriate for a particular context.

This distinction becomes especially important for developer behavioral memory. Knowing that a developer used a particular workflow in one project does not mean the workflow is appropriate for every task.

## Graph memory

Graphs are promising for representing entities, dependencies, decisions and provenance. They are especially interesting for software-engineering workloads where relationships such as `symbol -> caller -> dependency -> commit -> test` matter.

But EOKS should not require a graph. A local collection of structured files can be a valid implementation of the same conceptual contract.

## Learning records

A useful primitive for behavioral memory is a **Learning Record**:

```text
situation
  -> action / strategy
  -> evidence
  -> outcome
  -> evaluation
  -> provenance
  -> confidence
  -> scope / validity
  -> status
```

This differs from ordinary memory. A memory says what is known; a learning record captures **what was tried in a situation and what happened**. Learning records can eventually produce reusable skills, verification policies, routing policies or context-selection heuristics.

## Design question

Can memory systems expose enough provenance and confidence that the control plane can reason about **whether to trust a memory**, rather than treating retrieved text as ground truth?

A second question follows: can EOKS distinguish a useful personal pattern from a generally valid engineering policy, and require stronger evidence before promoting one into the latter?