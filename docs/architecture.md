# EOKS architecture

EOKS is a **workload control loop**, not a monolithic agent runtime. Its purpose is to coordinate knowledge, context, reasoning and execution so automated work can become more reliable and more autonomous when evidence justifies it.

```text
                         WORKLOAD
                            |
                          POLICY
                            |
                         CONDUCTOR
                            |
          +-----------------+-----------------+
          |                 |                 |
      KNOWLEDGE          CONTEXT          EXECUTION
    representations     compilation       workflows/runs
    + evidence          + budgets         models/tools/agents
          |                 |                 |
          +-----------------+-----------------+
                            |
                           RUN
                            |
                    OBSERVE / VERIFY
                            |
                         OUTCOME
                            |
                        EVALUATION
                            |
                   FEEDBACK / MAINTENANCE
                            |
                     next decision
```

## Architectural boundaries

### Control

The **control plane** decides what should happen next. Its primary responsibility is the **conductor**, which reconciles workload requirements and policy against current workflow state, available capabilities and evaluation evidence.

The conductor may request or skip planning, select resources and execution modality, request or reuse evidence, trigger context compilation, schedule workflow steps, execute/verify/retry/branch/re-plan/stop/escalate, and initiate maintenance or promotion.

"Scheduler", "router" and "orchestrator" describe implementation functions of this responsibility; they are not separate EOKS architectural layers. The conductor owns workflow state and decisions, **not durable knowledge**.

### Knowledge

The knowledge boundary contains durable project/system information and the representations used to preserve or derive it. EOKS does not require one canonical representation. Examples include human-maintained Markdown/ADR knowledge, portable structured knowledge, structural graphs, semantic indexes, timelines, runtime observations, episodic memory and procedural knowledge.

> **Knowledge is not a graph. A graph is one representation of knowledge or evidence.**

Knowledge maintenance is incremental and provenance-aware. Derived representations can be updated or invalidated from source changes rather than rebuilding everything after every event.

See [Knowledge base](knowledge-base.md) and [Knowledge representations](knowledge-representations.md).

### Context

Context is the **task-specific compilation of evidence and knowledge for a reasoning step**. It is not a storage layer and is not equivalent to durable knowledge or execution state.

Context compilation can retrieve, rank, transform, deduplicate, reconcile, order and budget information while preserving provenance and freshness. The resulting context should be inspectable and reproducible.

The Context Workbench is an observability/control surface for this boundary, not a second context architecture.

See [Context engineering](context.md) and [Context Workbench](context-workbench.md).

### Execution

Execution contains workflows and runs plus the resources that perform their steps. A **Run** is one attempt to execute a task or subtask under a particular state, policy, resource configuration and context.

Roles describe responsibilities within a workflow; they do not require a particular agent topology. Planning, execution, review and validation can be separate sessions or phases of one agent, depending on workload requirements.

See [Agent roles](agent-roles.md), [Agent workflows](agent-workflows.md) and [Deterministic execution](deterministic-execution.md).

### Evaluation and outcomes

Execution produces observations and an **Outcome**. Evaluation measures the outcome and relevant intermediate evidence, then feeds that evidence back into control.

```text
Run -> Outcome -> Evaluation -> Evidence -> Control decision
```

Evaluation is therefore a feedback mechanism, not another orchestration layer. Reliability signals should be calibrated against actual task outcomes before they drive automatic control decisions.

See [Evaluation](evaluation.md).

## Core distinctions

| Concept | Meaning |
|---|---|
| **Role** | Responsibility that must be performed in a workflow |
| **Resource** | Capability or reusable thing available to perform work or provide information |
| **Provider** | Mechanism that retrieves or derives information/evidence |
| **Representation** | Form in which knowledge/evidence is stored or expressed |
| **Context** | Task-specific information compiled for a reasoning step |
| **Execution state** | What the workflow has observed, attempted, changed, verified or invalidated |
| **Evidence** | Information supporting an assertion or decision |
| **Reasoning strategy** | How a reasoning step approaches its problem |
| **Workflow** | Temporal/dependency structure connecting responsibilities and actions |
| **Policy** | Constraints and requirements governing control decisions |

A resource may implement several roles. A role may be implemented by deterministic code, a tool, an agent, a model, a service or a human. A representation may be queried by multiple providers. These distinctions do not require new runtime primitives by themselves.

## Seven provisional runtime primitives

EOKS currently keeps seven runtime primitives:

| Primitive | Meaning |
|---|---|
| Task | Bounded work with objective, constraints and required assurance |
| Context | Information intentionally supplied to a reasoning step |
| Run | One attempt to execute a task or subtask |
| Decision | A control-plane choice about what happens next |
| Policy | Constraints and requirements governing decisions |
| Evaluation | Measurement of intermediate or final quality/assurance |
| Outcome | What actually happened, including artifacts and delayed results |

Models, tools, agents, memories, graphs and analyzers remain resources/providers rather than ontology objects merely because a framework gives them names.

`Decision` remains an empirical question: implementation traces should determine whether it needs independent lifecycle/identity or can eventually be represented as run/event state.

## Resource and evidence selection

The conductor should select the **minimum sufficient capability**, not the most powerful available one.

```text
workload question
      |
evidence requirement
      |
existing evidence sufficient?
   |             |
  yes            no
   |             |
continue      candidate providers
                  |
          capability / policy filter
                  |
        reliability + cost + latency
                  |
           minimum sufficient
                  |
              collect
                  |
              evaluate
                  |
          sufficient? -> escalate
```

This makes provider selection a control-plane decision without creating a separate routing architecture. The canonical capability model and evidence-selection semantics live in [Tool capability model](tool-capability-model.md).

## Execution modality and procedure consolidation

A workflow step should use the smallest execution modality that satisfies its requirements. Possible modalities include deterministic computation/tools, retrieval, specialized models, general LLM reasoning, multi-agent reasoning and human judgment. This is a **per-step control decision**, not a hierarchy of agent types.

When probabilistic reasoning resolves uncertainty and produces a sufficiently specified procedure, continuous maintenance may promote that procedure into a deterministic capability. Promotion requires validation and explicit dependency/assumption tracking; invalidation returns the workload to reasoning rather than silently executing stale behavior.

See [Deterministic execution](deterministic-execution.md) and [Continuous knowledge maintenance](continuous-knowledge-maintenance.md).

## Existing-agent integration

EOKS should normally coordinate existing coding agents and tools rather than require a replacement runtime:

```text
                    EOKS
                 /       \
       prepare/context   observe/evaluate
             |                |
             v                |
        existing agent ------+
```

Adapters, hooks, MCP and similar boundaries can connect the control loop to an existing execution environment. The EOKS abstraction is the control loop around the workload, not ownership of the underlying agent implementation.

## Design test

Before adding a new architectural concept, ask:

1. Is it genuinely a new runtime primitive, or can the seven existing primitives represent it?
2. Is it a role, resource, provider, representation, context artifact, workflow construct or policy?
3. Is it simply an implementation of the conductor's control responsibility?
4. Does it need independent lifecycle and provenance?
5. What experiment or run evidence demonstrates that the distinction improves quality, trust, velocity, cost, adaptability or observability?

**Prefer consolidation until implementation or evaluation evidence demonstrates that a new abstraction is necessary.**

## Canonical document ownership

The architecture page defines boundaries; detailed behavior belongs in the following documents:

- [Resource model](resource-model.md) — resource, asset, provider, representation and loadout semantics.
- [Context engineering](context.md) — context acquisition and compilation.
- [Knowledge base](knowledge-base.md) — durable knowledge lifecycle.
- [Knowledge representations](knowledge-representations.md) — representation families.
- [Memory](memory.md) — memory and behavioral learning.
- [Agent roles](agent-roles.md) — workflow responsibilities.
- [Agent workflows](agent-workflows.md) — workflow structure and reasoning strategies.
- [Evaluation](evaluation.md) — evaluation, reliability evidence and calibration.
- [Tool capability model](tool-capability-model.md) — provider capabilities, evidence requirements and selection.
- [Continuous knowledge maintenance](continuous-knowledge-maintenance.md) — incremental maintenance and procedure consolidation.
- [Deterministic execution](deterministic-execution.md) — deterministic execution as a modality.
- [Software analysis](software-analysis.md) — deterministic software evidence and analyzer escalation.

Research and prior-art documents remain exploratory and should not create competing normative definitions.
