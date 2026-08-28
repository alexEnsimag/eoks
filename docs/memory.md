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

## Memory can be multi-resolution

Memory does not have to be a flat collection of records or embeddings. A useful hierarchy is:

```text
raw conversation / observation
        |
        v
atomic fact / event
        |
        v
scenario / project memory
        |
        v
durable pattern / profile
```

The exact number of levels is implementation-specific. The important properties are that higher-level summaries can provide cheap context bootstrapping, lower-level records remain available for verification, and every abstraction retains provenance to the evidence from which it was derived.

TencentDB Agent Memory is useful prior art here: its current Chat Memory model uses L0 conversation, L1 atomic memory, L2 scenario memory and L3 core/profile memory. EOKS should treat that as a concrete design pattern, not as a universal ontology. See [TencentDB Agent Memory](../research/prior-art/tencent-agent-memory.md).

## Memory is not the generic Asset abstraction

Memory is one semantic type of reusable resource. A Skill, document, decision, CodeGraph or test result can also be reusable, but those are not automatically memories.

EOKS uses **Asset** only as a generic lifecycle/governance abstraction across such heterogeneous resources. The canonical terminology and the asset → loadout → context boundary are defined in [Resource model](resource-model.md).

This distinction matters because different resources have different authority and lifecycle semantics:

```text
Memory       -> experience-derived information
Skill        -> reusable procedure
Decision     -> reviewed rationale
CodeGraph    -> structural representation/evidence
Test result  -> verification evidence
```

They can share metadata such as provenance, scope, freshness, ownership/access, version and validation state without becoming one semantic category.

## Memory lifecycle

A memory candidate should have a lifecycle:

`observe -> extract -> validate -> store -> retrieve -> use -> evaluate -> update/expire`

The difficult part is not storage. It is deciding **what deserves persistence**, how confidence and provenance are retained, and how stale or contradictory memories are handled.

For behavioral learning, the lifecycle should be extended with explicit promotion:

`trace -> episode -> pattern candidate -> validate -> learning record -> promote to skill/policy -> evaluate`

An observed behavior should not silently become canonical project policy. Repeated evidence, outcomes and human corrections can strengthen a candidate, while contradictory evidence can keep it scoped or prevent promotion.

## Skills as procedural memory made reusable

A useful Skill should be treated as a governed procedural asset rather than a prompt snippet. Relevant metadata can include:

- applicability/trigger boundaries;
- version;
- execution steps;
- resources;
- validation rules;
- provenance and supporting outcomes;
- owner/scope/visibility;
- lifecycle status.

This is consistent with the existing Learning Record model: a Skill should be promoted from evidence and outcomes, not merely from repetition.

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

## Design questions

Can memory systems expose enough provenance and confidence that the control plane can reason about **whether to trust a memory**, rather than treating retrieved text as ground truth?

Can EOKS distinguish a useful personal pattern from a generally valid engineering policy, and require stronger evidence before promoting one into the latter?

How should multi-resolution memory handle source changes, contradictions and invalidation so that a durable summary does not outlive the evidence that supports it?
