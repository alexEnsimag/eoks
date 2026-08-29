# EOKS ontology

This page is the vocabulary boundary for EOKS. It exists to prevent roles, resources, representations and runtime primitives from becoming competing architectural concepts.

## The core loop

EOKS is a workload control loop:

```text
workload
   |
 policy
   |
 conductor
   |
   +--> acquire evidence / knowledge
   +--> compile context
   +--> select reasoning or execution modality
   |
   v
  run
   |
   v
verify / observe
   |
   v
outcome
   |
   v
evaluation
   |
   v
control + maintenance + promotion/invalidation
   |
   `--------------------> next decision
```

The objective is not maximum agent autonomy or maximum deterministic execution. It is useful engineering work with appropriate correctness, trust, adaptability, velocity and cost.

## Five distinctions to preserve

### 1. Roles are responsibilities, not necessarily agents

A role describes work that must be done. A role can be performed by an LLM agent, deterministic code, an existing service, a workflow worker or a human.

Core working roles include:

- **Conductor** — decides what should happen next under policy and observed state.
- **Retriever** — obtains relevant evidence or knowledge.
- **Transformer** — derives or updates representations or candidate knowledge.
- **Planner** — proposes a structured plan when planning is useful.
- **Executor** — performs an operation or invokes a capability.
- **Validator** — checks whether a requirement or proposed change is sufficiently established.
- **Reviewer** — provides independent semantic judgment when required.

These are composable responsibilities, not mandatory agent types.

### 2. Resources are capabilities, not roles

A resource is something available to perform work or provide information. Examples include models, deterministic tools, agents, analyzers, databases, indexes, graphs, knowledge stores and external services.

A resource can implement one or more roles. The conductor selects resources according to capability, assurance, scope, cost, latency and policy.

### 3. Providers and representations are different

A **representation** is a form in which information exists: a structural graph, semantic index, Markdown/CLAUDE.md, ADR, timeline, runtime observation, memory record or portable knowledge bundle.

A **provider** is the mechanism that retrieves or derives evidence from one or more representations.

A graph is therefore not automatically a provider, and a provider is not itself knowledge.

### 4. Context is not knowledge or execution state

- **Knowledge** is durable information and understanding.
- **Context** is task-specific information compiled for a reasoning step.
- **Execution state** records what a workflow has observed, established, attempted, changed, verified or invalidated.
- **Evidence** is information supporting a decision or assertion.

The same underlying source may contribute evidence to context without becoming durable knowledge.

### 5. Evaluation is feedback, not another decision-maker

Evaluation measures what happened and how well it met requirements. Its results become evidence for future control decisions.

```text
run -> outcome -> evaluation -> evidence -> policy/control
```

Do not treat evaluation as an independent orchestration layer.

## Conductor

The conductor is the primary control-plane responsibility. It should reconcile workload requirements and policy against observed state and available capabilities.

Its decisions may include:

- whether more evidence is needed;
- which evidence provider or representation to query;
- what context to compile;
- whether planning is warranted;
- which model or resource to use;
- whether a deterministic capability is sufficient;
- whether to execute, verify, retry, replan, branch, stop or escalate;
- whether maintenance or knowledge promotion should occur.

The conductor should not become the knowledge store or perform every operation itself.

## Execution modality

A workflow step can be implemented by the smallest modality that satisfies its requirements:

```text
deterministic computation / tool / validator
              |
          retrieval
              |
       specialized model
              |
        general LLM
              |
       multi-agent reasoning
              |
          human judgment
```

This is a per-step choice, not a permanent classification of a resource.

The important deterministic-execution principle is:

> Use probabilistic reasoning to resolve uncertainty. Once the required behavior is sufficiently specified, prefer deterministic execution and objective verification.

The goal is not to maximize the percentage of deterministic steps. Determinism is valuable when it provides the required outcome or assurance at better overall engineering economics.

## Procedure consolidation

Continuous maintenance can discover repeated, well-understood procedures and propose simpler deterministic capabilities:

```text
repeated traces
     |
stable procedure + known dependencies
     |
validated deterministic candidate
     |
register / reuse
     |
monitor assumptions
     |
 invalidate -> reason again -> recompile
```

Repetition alone is insufficient. Promotion requires sufficient specification, evidence, safe execution semantics and a maintenance cost justified by expected reuse.

This is a maintenance lifecycle, not a new agent role or architecture plane.

## Runtime primitives

EOKS currently uses seven provisional runtime primitives:

| Primitive | Meaning |
|---|---|
| Task | Bounded work with objective, constraints and required assurance |
| Context | Information intentionally supplied to a reasoning step |
| Run | One attempt to execute a task or subtask |
| Decision | A control-plane choice about what happens next |
| Policy | Constraints and requirements governing decisions |
| Evaluation | Measurement of intermediate or final quality/assurance |
| Outcome | What actually happened, including artifacts and delayed results |

Do not add primitives merely because a concept has a name elsewhere in the architecture. Models, tools, agents, knowledge stores, analyzers and representations remain resources/providers unless implementation evidence shows that they require first-class runtime identity.

`Decision` is intentionally retained as an empirical question: implementation traces should determine whether it needs independent identity and lifecycle or can eventually be represented as part of run/event state.

## Planes

The planes are organizational boundaries, not additional ontology objects:

```text
control       -> decisions and policy
knowledge     -> durable information and representations
context       -> acquisition, selection and compilation
execution     -> workflows, runs and capabilities
evaluation    -> measurement and feedback
```

Roles cross these planes. Resources can participate in several planes. Evaluation supplies evidence back to control.

## Canonical ownership

Use these documents as the primary owners of concepts:

| Concept | Canonical document |
|---|---|
| Overall architecture / planes | [Architecture](architecture.md) |
| Vocabulary and ontology boundaries | **This document** |
| Agent roles | [Agent roles](agent-roles.md) |
| Workflow and reasoning strategies | [Agent workflows](agent-workflows.md) |
| Resources, assets, providers and loadouts | [Resource model](resource-model.md) |
| Context acquisition and compilation | [Context engineering](context.md) |
| Knowledge representations | [Knowledge representations](knowledge-representations.md) |
| Durable knowledge lifecycle | [Knowledge base](knowledge-base.md) |
| Evaluation and calibration | [Evaluation](evaluation.md) |
| Continuous maintenance | [Continuous knowledge maintenance](continuous-knowledge-maintenance.md) |
| Deterministic execution | [Deterministic execution](deterministic-execution.md) |

Research documents may contain competing hypotheses and prior art. They should not silently become normative architecture decisions.

## Design test

When proposing a new EOKS concept, ask:

1. Is it a new **runtime primitive**, or can it be represented by the existing seven?
2. Is it a **role**, or merely a resource/capability that performs a role?
3. Is it a **resource**, **provider**, **representation**, or **context artifact**?
4. Is it a workflow/policy decision rather than a new architectural layer?
5. What trace or experiment demonstrates that the distinction improves correctness, trust, velocity, cost, adaptability or observability?

Prefer consolidation until implementation or evaluation evidence demonstrates that a new abstraction is necessary.
