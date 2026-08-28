# Memory and knowledge research

## Why memory became central

The initial context-engineering framing created a problem: if useful information repeatedly has to be rediscovered and reassembled, the system is missing a persistent knowledge layer.

The discussion of agent memory, especially TencentDB's Agent Memory work and GrapeRoot, pushed the idea toward a distinction between **what is available to the system** and **what is currently in the model's context**.

A useful conceptual separation is:

```text
persistent knowledge / memory
            |
            | retrieval / selection
            v
       working context
            |
            | model execution
            v
          outcome
            |
            +----> new memory / evidence / updates
```

## Context is not memory

Context is the model's current working set. Memory is persistent information that may be brought into future working sets.

This is analogous to a cache or working set, but the analogy must be used carefully: semantic retrieval, provenance and uncertainty make AI memory substantially different from CPU memory.

A useful memory system should answer:

- What was learned?
- From which source?
- When?
- Under what task or conditions?
- How reliable is it?
- Is it still valid?
- What other knowledge depends on it?
- When should it be retrieved?
- When should it be forgotten or superseded?

## Possible memory classes

We discussed several useful distinctions without settling on a canonical taxonomy:

- **working memory** — information needed for the current task;
- **episodic memory** — what happened during previous interactions/executions;
- **semantic/project knowledge** — durable facts, decisions and architecture;
- **procedural knowledge** — how a task is performed;
- **evidence** — observations supporting or contradicting a claim;
- **derived knowledge** — conclusions generated from other sources.

These categories may overlap in implementation. Their value is primarily semantic: they imply different retention, retrieval and validation policies.

## Provenance and freshness

A recurring concern was that AI memory can become harmful when old conclusions look authoritative. Memory therefore needs provenance and lifecycle management, not merely vector similarity.

A future knowledge item could carry:

```text
identity
content
source(s)
created_at
observed_at
validity / freshness
confidence
supporting evidence
contradictions
dependencies
supersedes / superseded_by
usage history
```

This enables a system to distinguish “the model said this once” from “this is a current project decision backed by evidence.”

## Graphs

Graphs repeatedly appeared as a possible representation because knowledge has relationships: a decision affects components; a component depends on APIs; an incident invalidates an assumption; a code symbol implements a requirement.

The graph is especially valuable for:

- traversing dependencies;
- finding related context;
- explaining retrieval;
- identifying impact;
- detecting stale or contradictory knowledge;
- constructing specialized context views.

But we did not conclude that EOKS should be a graph database. A graph is a useful model; storage can remain heterogeneous.

## Memory selection

The hard problem is not storing everything. It is deciding what deserves promotion into durable memory and what should remain ephemeral.

Potential signals include:

- explicit user/project decisions;
- repeated successful use;
- stable facts from authoritative sources;
- important failures and their causes;
- outcomes that change future policy;
- evidence that resolves an ambiguity.

Conversely, transient reasoning, guesses and task-local intermediate results should not automatically become durable knowledge.

## Forgetting and correction

Memory needs negative operations as first-class concepts:

- invalidate;
- supersede;
- retract;
- downgrade confidence;
- quarantine;
- expire;
- merge duplicates.

A system that only accumulates memory will eventually create a context-quality problem of its own.

## Relationship to EOKS

This led to a broader framing: **EOKS should manage a knowledge lifecycle, not simply provide an agent memory store.** Memory is one mechanism through which the control plane learns from previous workloads and improves future context composition.