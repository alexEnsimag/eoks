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

## Memory lifecycle

A memory candidate should have a lifecycle:

`observe -> extract -> validate -> store -> retrieve -> use -> evaluate -> update/expire`

The difficult part is not storage. It is deciding **what deserves persistence**, how confidence and provenance are retained, and how stale or contradictory memories are handled.

## Memory versus context

Memory is a source for future contexts. Context is the task-specific projection of available knowledge. A memory item can be correct but still be inappropriate for a particular context.

## Graph memory

Graphs are promising for representing entities, dependencies, decisions and provenance. They are especially interesting for software-engineering workloads where relationships such as `symbol -> caller -> dependency -> commit -> test` matter.

But EOKS should not require a graph. A local collection of structured files can be a valid implementation of the same conceptual contract.

## Design question

Can memory systems expose enough provenance and confidence that the control plane can reason about **whether to trust a memory**, rather than treating retrieved text as ground truth?
