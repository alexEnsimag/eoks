# EOKS architecture

EOKS is a **workload control loop**, not a monolithic agent runtime. Its purpose is to coordinate knowledge, context, reasoning and execution so automated work can become more reliable and more autonomous when evidence justifies it.

The central abstraction is **reconciliation**: establish the desired outcome and policy, observe current state and evidence, choose an appropriate action, execute it, evaluate the result, and reconcile again until acceptance criteria are satisfied or escalation is required. Plans are action proposals derived from state; they are disposable when observations invalidate their assumptions.

EOKS uses two complementary systems lenses:

- **Operating systems / computer architecture** provide a vocabulary for resource management: hierarchy, working sets, locality, caching, paging, I/O, scheduling, isolation and capacity pressure.
- **Kubernetes/control-loop architecture** provides a vocabulary for continuous reconciliation: desired state, observed state, controllers, scheduling, lifecycle, events and feedback.

These are **reference architectures, not implementation prescriptions**. The OS lens describes the resource-management problems; the Kubernetes lens describes how EOKS continuously controls the workload.

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
      RESOURCES          WORKLOAD         EXECUTION
  knowledge/evidence   working set/       workflows/runs
  providers/represent. context            models/tools/agents
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

## Architectural layers

### Resource layer

The resource layer answers **what exists, what it means, who may use it, and how it can be accessed efficiently**.

Resources include knowledge, evidence, representations, tools, models, execution capabilities and other reusable assets. They may differ in latency, capacity, freshness, authority, fidelity, retrieval cost and persistence.

The OS/computer-architecture lens is useful here: resources can have multiple representations and access tiers; locality, caching, batching, prefetching, replacement, isolation and quotas are candidate optimization mechanisms.

See [Resource model](resource-model.md) and [Knowledge representations](knowledge-representations.md). The detailed OS/architecture research is in [OS and computer-architecture lens](../research/prior-art/computer-systems-architecture.md).

### Workload layer

A **Workload** is the bounded unit whose desired outcome, policy, state, resources and execution are being controlled.

Its current information needs form a **working set**: the subset of eligible resources/evidence currently useful for making progress. Working set and context are not synonyms:

```text
resource/evidence universe
        |
 eligibility + policy
        |
 workload working set
        |
 context acquisition + compilation
        |
 model context
```

The working set is semantic; model context is one materialization of it. The working set may contain exact source, structural slices, verified facts, summaries or pointers to authoritative evidence.

A **context miss** occurs when required evidence is absent from the current working set and must be acquired. Misses, locality and context churn can become control signals.

See [Context engineering](context.md).

### Control layer

The **control plane** is the reconciliation mechanism for a workload. Its primary responsibility is the **conductor**, which compares workload requirements and policy with current state, available capabilities and evaluation evidence, then chooses what should happen next.

The conductor may derive or revise a plan, select resources and execution modality, request or reuse evidence, trigger context compilation, schedule workflow steps, execute/verify/retry/branch/re-plan/stop/escalate, and initiate maintenance or promotion.

"Scheduler", "router" and "orchestrator" are implementation functions of this responsibility, not separate EOKS architectural layers. The conductor owns workflow state and decisions, **not durable knowledge**.

Events can trigger reconciliation: test failures, dependency changes, stale resources, new commits, verification results or human input do not require a separate interrupt subsystem.

### Knowledge

The knowledge boundary contains durable project/system information and representations used to preserve or derive it. EOKS does not require one canonical representation. Graphs, indexes, timelines, observations, episodic memory and procedural knowledge are representations/resources, not automatically new ontology objects.

> **Knowledge is not a graph. A graph is one representation of knowledge or evidence.**

Knowledge maintenance is itself a reconciliation loop: authoritative sources are observed, derived representations are checked for freshness/validity, and stale or invalid representations are updated, invalidated or rebuilt.

See [Knowledge base](knowledge-base.md) and [Knowledge representations](knowledge-representations.md).

### Context

Context is the **task-specific compilation of evidence and knowledge for a reasoning step**. It is not a storage layer and is not equivalent to durable knowledge or execution state.

Computer architecture provides a useful interpretation: context is a fast, capacity-constrained materialization of the workload's working set. This supports candidate techniques such as locality-aware retrieval, demand acquisition, prefetching, admission, pinning, replacement, clustering and representation demotion/compression.

These are research interventions, not assumptions that one cache policy is universally correct. The objective is useful verified work, not maximum cache hit rate or minimum tokens in isolation.

Context compilation can retrieve, rank, transform, deduplicate, reconcile, order and budget information while preserving provenance and freshness. The resulting context should be inspectable and reproducible.

See [Context engineering](context.md) and [Context Workbench](context-workbench.md).

### Execution

Execution contains workflows and runs plus the resources that perform their steps. A **Run** is one attempt to execute a task or subtask under a particular state, policy, resource configuration and context.

Roles describe responsibilities within a workflow; they do not require a particular agent topology. Planning, execution, review and validation can be separate sessions or phases of one agent.

Execution is an actuator of the control loop. A failed or incomplete run is an observation that can cause reconciliation, not necessarily a terminal workflow failure.

Scheduling can borrow established systems techniques—priority, fairness, aging, batching, work stealing and load balancing—but these remain hypotheses to test. Deterministic tools, retrieval, specialized models, general reasoning, multi-agent workflows and humans are execution resources/modalities, not mandatory agent types.

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
2. observe current workload state, working set, resources and evidence
3. identify the relevant gap, miss, pressure or uncertainty
4. choose the minimum sufficient action/resource/modality
5. execute the action
6. observe and evaluate the result
7. update actual state/evidence and working-set state
8. reconcile again, or stop when acceptance criteria are satisfied
```

The loop does **not** imply that every step needs an LLM. The controller chooses the modality required by current evidence and risk.

A useful property is **reconstructability**: important workload state, evidence, decisions and artifacts should be durable enough that a new controller or execution attempt can resume without hidden agent memory.

Control loops can be nested. A workload loop may depend on knowledge-maintenance loops, while learning/policy improvement can be a slower loop over accumulated outcomes.

## Core distinctions

| Concept | Meaning |
|---|---|
| **Workload** | Bounded unit whose desired outcome, policy, resources, state and execution are controlled |
| **Working set** | Workload's currently useful subset of eligible resources/evidence |
| **Role** | Responsibility that must be performed in a workflow |
| **Resource** | Capability or reusable thing available to perform work or provide information |
| **Provider** | Mechanism that retrieves or derives information/evidence |
| **Representation** | Form in which knowledge/evidence is stored or expressed |
| **Loadout** | Workload-scoped resource namespace/eligibility boundary |
| **Context** | Task-specific information compiled for a reasoning step |
| **Context miss** | Required evidence absent from the current working set |
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

Models, tools, agents, memories, graphs, analyzers and working-set policies remain resources/providers/representations rather than ontology objects merely because a framework gives them names. `Decision` remains empirical: implementation traces should determine whether it needs independent lifecycle/identity or can eventually be represented as run/event state.

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

A useful systems heuristic is **deterministic-first where sufficient**: do not replace a task merely because an LLM can do it; replace probabilistic execution when the behavior is sufficiently understood, the deterministic capability is simpler to maintain, and the engineering outcome is better.

See [Deterministic execution](deterministic-execution.md) and [Continuous knowledge maintenance](continuous-knowledge-maintenance.md).

## Established systems techniques as hypotheses

OS and architecture techniques should be imported as candidate interventions, not as mandatory EOKS components.

### Context / working-set management

- locality-aware acquisition;
- demand retrieval/context misses;
- prefetching;
- cache admission;
- pinning;
- LRU/LFU/relevance-aware replacement baselines;
- evidence clustering/batching;
- representation compression/demotion;
- context-thrashing detection;
- navigation-resolution caching.

### Work scheduling

- priority;
- fairness/anti-starvation;
- aging;
- batching;
- work stealing;
- load balancing;
- asynchronous acquisition;
- deterministic-first scheduling.

### Shared resources

- scoped access/protection;
- quotas;
- versioned state;
- invalidation;
- copy-on-write derived state;
- explicit consistency boundaries.

These mechanisms should be evaluated against end-to-end outcome, evidence quality, cost, latency, reliability and recovery—not against infrastructure metrics alone.

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

## Context cache versus inference cache

Keep two concepts separate:

- **Semantic/context cache** — reusable knowledge, evidence, representations and navigation resolutions managed around an EOKS workload.
- **Inference/KV cache** — model-serving state used to avoid recomputing attention representations.

EOKS may influence inference-cache reuse indirectly through stable context structure, but the two should not be collapsed into one memory abstraction.

## Canonical document ownership

The architecture page defines boundaries; detailed behavior belongs in the following documents:

- [Resource model](resource-model.md) — resource, asset, provider, representation and loadout semantics.
- [Context engineering](context.md) — context acquisition, working sets and compilation.
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
- [OS and computer-architecture lens](../research/prior-art/computer-systems-architecture.md) — systems analogies, optimization techniques and candidate interventions.

Research and prior-art documents remain exploratory and should not create competing normative definitions.
