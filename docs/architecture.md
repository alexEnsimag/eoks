# EOKS architecture

EOKS is a **workload control loop**, not a monolithic agent runtime. Its purpose is to coordinate knowledge, context, reasoning and execution so automated work can become more reliable and more autonomous when evidence justifies it.

The central abstraction is **reconciliation**: establish the desired outcome and policy, observe current state and evidence, choose an appropriate action, execute it, evaluate the result, and reconcile again until acceptance criteria are satisfied or escalation is required. Plans are action proposals derived from state; they are disposable when observations invalidate their assumptions.

```text
                         INTENT
                            |
                    desired outcome/state
                            |
                       POLICY
                            |
                            v
                 +---------------------+
                 |      CONDUCTOR      |
                 | reconciler/controller|
                 +----------+----------+
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
                   ACTUAL WORKLOAD STATE
                            |
                       RECONCILIATION
                            |
                  next decision / action
```

## Architectural boundaries

### Control

The **control plane** is the reconciliation mechanism for a workload. Its primary responsibility is the **conductor**, which compares workload requirements and policy with current state, available capabilities and evaluation evidence, then chooses what should happen next.

The conductor may derive or revise a plan, select resources and execution modality, request or reuse evidence, trigger context compilation, schedule workflow steps, execute/verify/retry/branch/re-plan/stop/escalate, and initiate maintenance or promotion.

"Scheduler", "router" and "orchestrator" are implementation functions of this responsibility, not separate EOKS architectural layers. The conductor owns workflow state and decisions, **not durable knowledge**.

A plan is an **action proposal**, not authoritative workload state. When observations invalidate assumptions, the conductor should discard or revise the plan and reconcile again from current state.

### Knowledge

The knowledge boundary contains durable project/system information and representations used to preserve or derive it. EOKS does not require one canonical representation. Graphs, indexes, timelines, observations, episodic memory and procedural knowledge are representations/resources, not automatically new ontology objects.

> **Knowledge is not a graph. A graph is one representation of knowledge or evidence.**

Knowledge maintenance is itself a reconciliation loop: authoritative sources are observed, derived representations are checked for freshness/validity, and stale or invalid representations are updated, invalidated or rebuilt.

See [Knowledge base](knowledge-base.md) and [Knowledge representations](knowledge-representations.md).

### Context

Context is the **task-specific compilation of evidence and knowledge for a reasoning step**. It is not a storage layer and is not equivalent to durable knowledge or execution state.

Context compilation can retrieve, rank, transform, deduplicate, reconcile, order and budget information while preserving provenance and freshness. The resulting context should be inspectable and reproducible.

See [Context engineering](context.md) and [Context Workbench](context-workbench.md).

### Execution

Execution contains workflows and runs plus the resources that perform their steps. A **Run** is one attempt to execute a task or subtask under a particular state, policy, resource configuration and context.

Roles describe responsibilities within a workflow; they do not require a particular agent topology. Planning, execution, review and validation can be separate sessions or phases of one agent.

Execution is an actuator of the control loop. A failed or incomplete run is an observation that can cause reconciliation, not necessarily a terminal workflow failure.

See [Agent roles](agent-roles.md), [Agent workflows](agent-workflows.md) and [Deterministic execution](deterministic-execution.md).

### Evaluation and outcomes

Execution produces observations and an **Outcome**. Evaluation measures the outcome and relevant intermediate evidence, then feeds that evidence back into reconciliation.

```text
Run -> Outcome -> Evaluation -> Evidence -> Actual state -> Reconcile
```

Evaluation is a feedback mechanism, not another orchestration layer. Reliability signals should be calibrated against actual task outcomes before they drive automatic control decisions.

See [Evaluation](evaluation.md).

## Control-loop semantics

The minimal loop is:

```text
1. establish desired outcome/state from intent and policy
2. observe current workload state and available evidence/capabilities
3. identify the relevant gap or uncertainty
4. choose the minimum sufficient action
5. execute the action
6. observe and evaluate the result
7. update actual state/evidence
8. reconcile again, or stop when acceptance criteria are satisfied
```

The loop does **not** imply that every step needs an LLM. Deterministic tools, retrieval, specialized models, general reasoning, multi-agent workflows and humans can all be resources used by a controller. The controller chooses the modality required by current evidence and risk.

A useful property is **reconstructability**: important workload state, evidence, decisions and artifacts should be durable enough that a new controller or execution attempt can resume without hidden agent memory.

Control loops can be nested. A workload loop may depend on knowledge-maintenance loops, while learning/policy improvement can be a slower loop over accumulated outcomes.

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

A resource may implement several roles. A role may be implemented by deterministic code, a tool, an agent, a model, a service or a human. These distinctions do not require new runtime primitives by themselves.

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

Models, tools, agents, memories, graphs and analyzers remain resources/providers rather than ontology objects merely because a framework gives them names. `Decision` remains empirical: implementation traces should determine whether it needs independent lifecycle/identity or can eventually be represented as run/event state.

## Resource and evidence selection

The conductor should select the **minimum sufficient capability**, not the most powerful available.

```text
workload question -> evidence requirement
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
              collect -> evaluate
                         |
                    sufficient?
                     /       \
                   yes        no
                   stop      escalate
```

This makes provider selection a control-plane decision without creating a separate routing architecture. The canonical capability model lives in [Tool capability model](tool-capability-model.md).

## Execution modality and procedure consolidation

A workflow step should use the smallest execution modality that satisfies its requirements. Possible modalities include deterministic computation/tools, retrieval, specialized models, general LLM reasoning, multi-agent reasoning and human judgment. This is a **per-step control decision**, not a hierarchy of agent types.

When probabilistic reasoning resolves uncertainty and produces a sufficiently specified procedure, continuous maintenance may promote that procedure into a deterministic capability. Promotion requires validation and explicit dependency/assumption tracking; invalidation returns the workload to reasoning rather than silently executing stale behavior.

See [Deterministic execution](deterministic-execution.md) and [Continuous knowledge maintenance](continuous-knowledge-maintenance.md).

## Existing-agent integration

EOKS should normally coordinate existing coding agents and tools rather than require a replacement runtime. Adapters, hooks, MCP and similar boundaries can connect the control loop to an existing execution environment. The EOKS abstraction is the control loop around the workload, not ownership of the underlying agent implementation.

## Design test

Before adding a new architectural concept, ask:

1. Is it genuinely a new runtime primitive, or can the seven existing primitives represent it?
2. Is it a role, resource, provider, representation, context artifact, workflow construct or policy?
3. Is it simply an implementation of the conductor's control responsibility?
4. What state does it observe, and what state can it change?
5. What makes its output verifiable, invalidatable or rebuildable?
6. Does it need independent lifecycle and provenance?
7. What experiment or run evidence demonstrates that the distinction improves quality, trust, velocity, cost, adaptability or observability?

**Prefer consolidation until implementation or evaluation evidence demonstrates that a new abstraction is necessary.**

## Cards as an open hypothesis

A **Card** may be useful as an inspectable representation of workload state, intent, evidence and verification criteria, analogous in spirit to a resource's desired specification plus observed status. It is **not currently an EOKS runtime primitive**.

Before introducing Cards into the ontology, test whether existing Task, Policy, Run, Decision, Evaluation and Outcome state already provides the necessary semantics. If Cards prove useful, initially treat them as a representation/protocol/UI over that state rather than a competing source of truth.

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
