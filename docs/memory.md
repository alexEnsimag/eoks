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

### Policy and preference memory
Policies describe what should happen and therefore require stronger validation and versioning before influencing execution. Preferences can guide behavior but should not automatically become engineering rules.

These are semantic distinctions, not necessarily separate storage systems. One implementation may represent several types with different authority and lifecycle metadata.

## Memory versus knowledge and context

Memory is information deliberately persisted for future use. It can contain experience-derived knowledge, but not every durable knowledge artifact is a memory: a reviewed ADR, source-derived graph or test result may be a reusable asset without being memory.

Context is the task-specific projection supplied to a reasoning step. Memory is therefore one possible source for future context, not the context itself.

See [Resource model](resource-model.md) for the canonical Asset/Provider/Representation/Loadout/Context vocabulary.

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

The exact number of levels is implementation-specific. Higher-level summaries can provide cheap bootstrapping while lower-level records remain available for verification. Every abstraction should retain provenance to the evidence from which it was derived.

TencentDB Agent Memory is useful prior art here: its current Chat Memory model uses L0 conversation, L1 atomic memory, L2 scenario memory and L3 core/profile memory. EOKS treats that as a design pattern, not a universal ontology. See [TencentDB Agent Memory](../research/prior-art/tencent-agent-memory.md).

## Memory lifecycle

A memory candidate should have a lifecycle:

`observe -> extract -> validate -> store -> retrieve -> use -> evaluate -> update/expire`

The difficult part is not storage. It is deciding **what deserves persistence**, how provenance is retained, and how stale or contradictory memories are handled.

For behavioral learning, extend the lifecycle with explicit promotion:

`trace -> episode -> pattern candidate -> validate -> Learning Record -> promote -> evaluate`

An observed behavior should not silently become canonical project policy. Repeated evidence, outcomes and human corrections can strengthen a candidate, while contradictory evidence can keep it scoped or prevent promotion.

## Graph memory

Graphs are promising for representing entities, dependencies, decisions and provenance, especially for software-engineering relationships such as `symbol -> caller -> dependency -> commit -> test`.

But EOKS should not require a graph. A local collection of structured files can implement the same conceptual contract. Graphs remain one representation/evidence mechanism rather than a universal memory ontology.

## Procedural and behavioral memory

A persistent knowledge base can answer **what is true about the project**. EOKS can also learn **how work gets done** by retaining structured traces of development sessions and extracting outcome-linked procedures.

A useful trajectory is:

```text
problem
  -> hypothesis
  -> evidence gathering
  -> failed attempt
  -> correction
  -> implementation
  -> verification
  -> review
  -> outcome
```

The goal is not to imitate every historical action, but to discover repeatable procedures whose usefulness is supported by outcomes.

### Session traces

A coding session should be represented as a trace rather than only a transcript:

```text
Goal -> plan -> observations/evidence -> tools/files -> hypotheses
     -> edits -> failures/corrections -> verification -> human feedback -> outcome
```

Useful events include task start/completion, plan revisions, tool calls, artifacts inspected, hypotheses, tests, failures, corrections, human intervention, acceptance/rejection and cost/latency/model information. A trace makes the **decision process and outcome** queryable. Sensitive data requires explicit filtering, retention and promotion policies.

### Observation is not learning

```text
observed behavior
      |
      v
candidate pattern
      |
      v
supporting evidence + outcomes
      |
      v
validated procedure
      |
      v
skill / workflow / policy
```

A single session is usually insufficient evidence. A behavior may reflect an unusual constraint or one-off incident. Patterns should carry provenance, scope, prerequisites, supporting sessions, outcomes and counterexamples.

EOKS should distinguish:

- **observed** — this happened;
- **repeated** — this happened several times;
- **successful** — associated with good outcomes;
- **validated** — evidence supports recommending it;
- **deprecated** — later evidence shows it should no longer be used.

### Learning Record

A useful primitive is:

```text
LearningRecord
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

A memory says what is known; a Learning Record captures **what behavior was tried in what situation and what happened**. It can produce reusable Skills, workflows, planner heuristics, tool-selection policies, verification policies or escalation rules.

### Hot path versus background learning

The executing agent can record important observations immediately, but this adds latency and overhead. A background process can analyze completed sessions, compare episodes and extract candidate patterns. The background path is therefore the preferred research direction for discovering general procedures, while high-confidence facts can still be captured during execution.

### Why transcript RAG is insufficient

Historical retrieval can answer **"Have I seen this problem before?"** Behavioral learning additionally asks **"What worked in similar situations, under what conditions, and should the system do something similar now?"**

That requires structured episodes, outcome/evaluation signals, provenance, temporal validity, promotion rules and regression evaluation. Transcripts remain evidence; they are not learned policy by themselves.

## Learning and the control loop

```text
observe -> extract -> validate -> store -> retrieve -> execute
                                           |
                                           v
evaluate -> compare outcomes -> update candidate -> controlled rollout -> evaluate
```

The learning subsystem transforms accumulated evidence into candidate improvements that can be evaluated and versioned. It should not silently rewrite canonical knowledge or policy.

A learned pattern must retain scope. A personal preference is not automatically a project rule; a project convention is not automatically a general engineering principle; and a procedure that works for one model is not necessarily effective for another.

## Skills as procedural memory made reusable

A Skill should be a governed procedural asset rather than a prompt snippet. Relevant metadata includes applicability/trigger boundaries, version, execution steps, validation rules, provenance, supporting outcomes, owner/scope and lifecycle status.

## Research boundaries

The key falsifiable question is empirical: **does learning procedural patterns from real development traces measurably improve future software-engineering outcomes enough to justify the added complexity?**

Important questions include:

- What is the minimum useful session trace?
- How many examples are needed before promotion?
- How can the system avoid learning accidental habits?
- How should contradictory procedures coexist?
- How should developer-specific behavior differ from project-specific policy?
- Can learned procedures be evaluated offline against historical sessions?
- When should a pattern become a Skill versus remain evidence?
- How do model changes affect the apparent success of a learned procedure?
- How should humans approve, correct or delete learned behavior?

## Prior-art boundary

LangMem, Mem0, Zep and related memory systems are capability references, not EOKS dependencies. Their value is in specific memory extraction, storage, retrieval or reflection mechanisms; EOKS remains broader because it connects memory with evidence providers, context compilation, execution policy, scheduling and evaluation.

## Design questions

Can memory systems expose enough provenance and evidence that the control plane can reason about **whether to trust a memory**, rather than treating retrieved text as ground truth?

How should multi-resolution memory handle source changes, contradictions and invalidation so that a durable summary does not outlive the evidence that supports it?
