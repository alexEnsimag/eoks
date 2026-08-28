# Memory

EOKS treats memory as deliberate persistence for future work, not as an ever-growing transcript.

## Semantic types

- **Working** — information needed by the current task or reasoning chain.
- **Episodic** — what happened in a previous interaction or execution: actions, observations, failures and outcomes.
- **Semantic** — durable facts, concepts, decisions and relationships.
- **Project** — evolving codebase/project state: architecture decisions, constraints, conventions, known failures and goals.
- **Procedural** — how work is performed: debugging strategies, decomposition patterns, verification habits and successful workflows.
- **Policy** — what should happen; requires stronger validation and versioning before influencing execution.
- **Preference** — human choices that may guide behavior but should not automatically become engineering rules.

These are semantic distinctions, not necessarily separate stores.

## Memory versus other resources and context

Memory is one semantic type of reusable resource. A reviewed ADR, source-derived graph or test result can be reusable without being memory. `Asset`, `Provider`, `Representation`, `Loadout` and `Context` are the generic resource/context vocabulary; see [Resource model](resource-model.md).

Context is the task-specific projection supplied to a reasoning step. Memory is therefore one possible source for future context, not the context itself.

Different resources can share governance metadata—provenance, scope, freshness, ownership/access, version and validation state—without becoming one semantic category.

## Multi-resolution memory

Memory can be represented at several resolutions:

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

The exact number of levels is implementation-specific. Higher-level summaries can provide cheap bootstrapping while lower-level evidence remains available for verification. Every abstraction should retain provenance.

TencentDB Agent Memory is useful prior art: its current Chat Memory model uses L0 conversation, L1 atomic memory, L2 scenario memory and L3 core/profile memory. EOKS treats this as a design pattern, not a universal ontology. See [TencentDB Agent Memory](../research/prior-art/tencent-agent-memory.md).

## Memory lifecycle

A memory candidate follows:

`observe -> extract -> validate -> store -> retrieve -> use -> evaluate -> update/expire`

The hard problem is deciding what deserves persistence and how stale, contradictory or low-quality memory is handled.

For behavioral learning, extend this with explicit promotion:

`trace -> episode -> pattern candidate -> validate -> Learning Record -> promote -> evaluate`

An observed behavior must not silently become canonical project policy. Repeated evidence, outcomes and human corrections can strengthen a candidate; counterexamples can keep it scoped or prevent promotion.

## Procedural / behavioral memory

A persistent knowledge base describes **what is true**. Procedural memory captures **how work gets done**. A useful development trajectory is:

```text
problem -> hypothesis -> evidence -> failed attempt -> correction
        -> implementation -> verification -> review -> outcome
```

A coding session should be represented as a trace, not only a transcript:

```text
Goal -> plan -> observations/evidence -> tools/files -> hypotheses
     -> edits -> failures/corrections -> verification -> human feedback -> outcome
```

Useful events include task start/completion, plan revisions, tool calls, artifacts inspected, hypotheses, tests, failures, corrections, human intervention, acceptance/rejection and cost/latency/model information. Sensitive data requires explicit filtering, retention and promotion policies.

Observation is not learning. Distinguish:

`observed -> repeated -> successful -> validated -> deprecated`

Patterns should retain provenance, scope, prerequisites, supporting sessions, outcomes and counterexamples. A single successful session is usually insufficient evidence for a generalized procedure.

## Learning Records and Skills

A **Learning Record** captures:

```text
situation
  action / strategy
  evidence
  outcome
  evaluation
  provenance
  confidence
  scope / validity
  status: candidate | validated | promoted | deprecated
```

A memory says what is known; a Learning Record captures what was tried in a situation and what happened. It can produce reusable Skills, workflows, planner heuristics, tool-selection policies, verification policies or escalation rules.

A **Skill** is a governed procedural asset rather than a prompt snippet. It should carry applicability/trigger boundaries, version, execution steps, validation rules, provenance, supporting outcomes, scope/visibility and lifecycle status.

The executing agent can record important observations immediately, while background processing compares completed sessions and extracts candidate patterns. This keeps general learning off the critical path where possible.

## Why transcript RAG is insufficient

Historical retrieval can answer "Have I seen this before?" Behavioral learning additionally asks "What worked in similar situations, under what conditions, and should it be reused now?" That requires structured episodes, outcome/evaluation signals, provenance, temporal validity, promotion rules and regression evaluation. Transcripts remain evidence, not learned policy by themselves.

## Graph memory

Graphs are promising for entities, dependencies, decisions and provenance, especially relationships such as `symbol -> caller -> dependency -> commit -> test`. But EOKS does not require a graph; structured files or other stores can implement the same conceptual contract. A graph is a representation/evidence mechanism, not a universal memory ontology.

## Learning and control

```text
observe -> extract -> validate -> store -> retrieve -> execute
                                           |
                                           v
evaluate -> compare outcomes -> update candidate -> controlled rollout -> evaluate
```

Learning is currently a **cross-cutting lifecycle**, not a separate mandatory EOKS plane. It transforms evidence into candidate improvements that can be evaluated and versioned; it must not silently rewrite canonical knowledge or policy.

A learned pattern must retain scope: a personal preference is not automatically a project rule; a project convention is not automatically a general engineering principle; and a procedure effective for one model is not necessarily effective for another.

## Research boundary

The key falsifiable question is: **does learning procedural patterns from real development traces measurably improve future software-engineering outcomes enough to justify the added complexity?**

Important questions include minimum useful traces, promotion thresholds, accidental habits, contradictory procedures, developer-vs-project scope, offline evaluation, model changes and human approval/deletion.

LangMem, Mem0, Zep and similar systems are capability references rather than EOKS dependencies. Their extraction, storage, retrieval and reflection mechanisms are useful prior art; EOKS is broader because it connects memory with evidence, context compilation, execution policy, scheduling and evaluation.
