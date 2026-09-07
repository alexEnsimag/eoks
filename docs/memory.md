# Memory

EOKS treats memory as deliberate persistence for future work, not as an ever-growing transcript. Memory is a lifecycle over retained experience and knowledge, not a single storage technology.

## Semantic types

- **Working** — information needed by the current task or reasoning chain.
- **Episodic** — what happened in a previous interaction or execution: actions, observations, failures and outcomes.
- **Semantic** — durable facts, concepts, decisions and relationships.
- **Project** — evolving codebase/project state: architecture decisions, constraints, conventions, known failures and goals.
- **Procedural** — how work is performed: debugging strategies, decomposition patterns, verification habits and successful workflows.
- **Policy** — what should happen; requires stronger validation and versioning before influencing execution.
- **Preference** — human choices that may guide behavior but should not automatically become engineering rules.

These are semantic distinctions, not necessarily separate stores or databases. Recent agent-memory research increasingly distinguishes persistent experience, generalized knowledge and reusable behavior, while classical cognitive architectures provide related working/episodic/semantic/procedural distinctions. EOKS uses these as engineering categories, not as a claim to reproduce human cognition. See [agent memory lifecycle research](../research/prior-art/agent-memory-lifecycle-2026.md).

## Evidence, experience, knowledge and capability

Memory should not collapse four different things:

```text
Evidence       what happened: observations, code, traces, tests, reviews, outcomes
Experience     structured episodes/trajectories recording what was attempted and what happened
Knowledge      generalized facts, relationships, constraints and project understanding
Capability     reusable procedures, Skills, workflows and policies describing how work can be performed
```

A graph, vector index, document, summary or database is a representation/storage choice. It does not define the semantic category of the information stored there.

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

Recent research suggests a more useful lifecycle than simply short-term versus long-term memory. A practical EOKS formulation is:

```text
observe
  -> retain evidence / episode
  -> retrieve
  -> reflect
  -> synthesize
  -> validate
  -> promote
  -> compile context
  -> execute
  -> evaluate outcome
  -> revise / invalidate / forget
```

A memory candidate therefore follows:

`observe -> extract -> validate -> store -> retrieve -> use -> evaluate -> update/expire`

The hard problem is deciding what deserves persistence and how stale, contradictory or low-quality memory is handled. Reflection is not synonymous with ordinary model reasoning or “thinking first”: in the memory literature it is trajectory refinement that can happen after or across experiences. Synthesis is the step that generalizes useful observations into reusable knowledge or capability.

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

## Reflection and synthesis

Recent research frames the evolution of agent memory as **Storage -> Reflection -> Experience**: preserve trajectories, refine them, then abstract reusable experience. This complements EOKS's existing reflection/artifact/synthesis work.

For EOKS:

- **Storage** preserves evidence and episodes.
- **Reflection** asks what happened, what explains it and what changed.
- **Synthesis** asks what generalizes across evidence or trajectories.
- **Promotion** asks what evidence justifies making the result reusable or authoritative.

This separation is important because not every reflection becomes knowledge, and not every learned procedure should become policy.

## Why transcript RAG is insufficient

Historical retrieval can answer "Have I seen this before?" Behavioral learning additionally asks "What worked in similar situations, under what conditions, and should it be reused now?" That requires structured episodes, outcome/evaluation signals, provenance, temporal validity, promotion rules and regression evaluation. Transcripts remain evidence, not learned policy by themselves.

Recent benchmarks reinforce this boundary: long-term memory should be evaluated for temporal/update reasoning, selective forgetting and actual use in task execution—not only passive fact recall. See [agent memory lifecycle research](../research/prior-art/agent-memory-lifecycle-2026.md).

## Graph memory

Graphs are promising for entities, dependencies, decisions and provenance, especially relationships such as `symbol -> caller -> dependency -> commit -> test`. Agent-memory research such as A-MEM also shows that dynamically linking memories can improve organization. But EOKS does not require a graph; structured files or other stores can implement the same conceptual contract. A graph is a representation/evidence mechanism, not a universal memory ontology.

## Learning and control

```text
observe -> extract -> validate -> store -> retrieve -> execute
                                           |
                                           v
evaluate -> compare outcomes -> update candidate -> controlled rollout -> evaluate
```

Learning is currently a **cross-cutting lifecycle**, not a separate mandatory EOKS plane. It transforms evidence into candidate improvements that can be evaluated and versioned; it must not silently rewrite canonical knowledge or policy.

A learned pattern must retain scope: a personal preference is not automatically a project rule; a project convention is not automatically a general engineering principle; and a procedure effective for one model is not necessarily effective for another.

## Evaluation

Memory evaluation should be downstream of retrieval. Relevant dimensions include:

- retrieval accuracy and relevance;
- temporal and update correctness;
- contradiction handling and abstention;
- selective forgetting/invalidation;
- cross-session and cross-trajectory generalization;
- use in actual task/action execution;
- context/token efficiency;
- downstream workload outcomes such as quality, reliability, cost and latency.

Mem2ActBench is particularly relevant because it evaluates whether memory affects tool selection and parameter grounding rather than merely whether a fact can be recalled. LongMemEval and MemoryAgentBench similarly broaden evaluation toward multi-session reasoning, updates, temporal reasoning, learning and forgetting. These are research references, not EOKS requirements.

## Research boundary

The key falsifiable question is: **does learning procedural patterns from real development traces measurably improve future software-engineering outcomes enough to justify the added complexity?**

Important questions include minimum useful traces, promotion thresholds, accidental habits, contradictory procedures, developer-vs-project scope, offline evaluation, model changes and human approval/deletion.

LangMem, Mem0, Zep and similar systems are capability references rather than EOKS dependencies. Their extraction, storage, retrieval and reflection mechanisms are useful prior art; EOKS is broader because it connects memory with evidence, context compilation, execution policy, scheduling and evaluation.
