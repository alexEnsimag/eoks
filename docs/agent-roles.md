# Agent roles

EOKS uses **roles** to describe responsibilities within a workflow. A role is not an agent, model, tool or resource: it is a responsibility that can be implemented by one or more runtime participants.

This distinction keeps the semantic model small while allowing different execution topologies. A single agent can perform several roles sequentially, or separate agents/sessions can implement different roles when isolation, independence, parallelism or specialization provides a concrete benefit.

## Core distinction

```text
Workflow
   |
   +--> assigns responsibilities -> Role
   |
   +--> selects capabilities -----> Resource
   |
   `--> executes through ---------> Agent / Tool / Provider
```

A useful mental model is:

- **Role** — what responsibility must be performed?
- **Agent** — what runtime execution loop performs it?
- **Resource** — what capability or reusable thing can it use?
- **Reasoning strategy** — how should the reasoning step approach the responsibility?
- **Workflow** — how are responsibilities and actions connected over time?
- **Conductor** — how is the workflow coordinated and reconciled against observed state?

Roles therefore cut across the EOKS planes rather than introducing another architectural plane.

## Core roles

### Conductor

The **conductor** coordinates a workload and owns workflow state, policy application, handoffs and reconciliation. It decides what should happen next based on the current state, available resources, policy and evaluation evidence.

Typical responsibilities include:

- selecting or requesting a plan;
- assigning workflow steps to roles/resources;
- coordinating context and loadouts without becoming a knowledge base;
- scheduling sequential or parallel work;
- tracking artifacts, revisions, failures and verification evidence;
- deciding whether to continue, retry, branch, repair, re-plan or escalate.

"Orchestrator" is a useful implementation synonym, but **conductor** is the preferred EOKS role name.

The conductor is a control/execution responsibility, not necessarily a dedicated model or service.

### Retriever

The **retriever** obtains relevant information or evidence from available sources and providers.

Examples include code search, graph queries, memory retrieval, documentation lookup, historical evidence and external knowledge access.

Retrieval is not synonymous with context compilation: a context compiler may invoke one or more retrievers and then rank, transform, order and budget their results.

```text
Retriever -> evidence
```

### Transformer

The **transformer** changes information from one useful representation into another while preserving relevant provenance and semantics.

Examples include summarization, extraction, normalization, compression, dependency-slice construction and conversion of raw observations into structured evidence.

Transformation can occur in the context plane, knowledge lifecycle or execution workflow. It is a role, not a separate EOKS plane.

```text
information -> Transformer -> derived information/evidence
```

### Planner

The **planner** proposes a sequence or graph of actions for achieving a workload objective under known state, evidence and policy.

A plan is an **executable hypothesis**, not a guarantee. New observations, failures, changed state or policy can invalidate its assumptions and trigger verification, branching or re-planning.

The planner is therefore distinct from the conductor:

```text
Conductor -> requests / selects planning
Planner   -> proposes a plan
Conductor -> decides how to execute and reconcile it
```

A dedicated planner is optional. Planning may be performed by the same agent that executes the work when that is sufficient.

### Executor

The **executor** performs actions and produces artifacts or side effects.

Examples include editing code, invoking APIs, running commands, executing tests or making other permitted changes.

Execution should remain subject to policy and verification requirements rather than being treated as successful because an agent reports completion.

### Reviewer

The **reviewer** independently challenges an artifact, decision or proposed result.

The reviewer asks questions such as:

- Is the result correct?
- Is it complete with respect to the task?
- Does it respect requirements and architectural intent?
- What assumptions or failure modes did the executor miss?

Independence is a property of the workflow/context boundary, not necessarily of model identity. A fresh session or sufficiently isolated context can provide useful independence even when the same underlying model is used.

### Validator

The **validator** obtains objective or structured evidence about whether an assertion, invariant or expected outcome holds.

Examples include tests, type checking, static analysis, deployment checks, policy checks and observed runtime behavior.

Validator and reviewer are deliberately distinct:

```text
Reviewer  -> independent judgment/challenge
Validator -> evidence about whether conditions hold
```

Deterministic validation evidence should generally outrank self-reported agent confidence when the two disagree.

## Secondary roles and workflow actions

Not every useful name should become a first-class core role.

### Repairer

A **repairer** responds to review or validation findings by changing the work. In most workflows this is best treated as a specialization or mode of the executor rather than a permanently separate runtime role:

```text
validation failure -> repair -> executor -> validation
```

The semantic distinction can be useful in workflow traces, but EOKS should not require a dedicated repair agent.

### Escalation

**Escalation** is a control/workflow transition rather than a core agent role. A conductor can escalate when required assurance cannot be achieved, ambiguity remains, policy requires human involvement, or the current execution topology cannot safely proceed.

```text
insufficient assurance
        |
        v
    escalate
        |
 human / stronger authority / different workflow
```

A system may implement escalation through a human gate, another agent, a stronger model or a different workflow. The semantic primitive is the decision to escalate, not a permanent "escalator" agent.

## Role composition

Roles are responsibilities, not a prescribed swarm topology.

The smallest useful software-engineering topology is commonly:

```text
human goal
    |
    v
conductor
    |
    +--> planner (optional)
    +--> executor
    +--> reviewer
    |
    v
validator / evaluation
    |
    +--> continue
    +--> repair / retry
    +--> re-plan
    +--> escalate
```

A simple task can collapse this into one agent/session. A more demanding task can isolate the reviewer, run independent retrieval or planning workers in parallel, or use deterministic validators without another LLM.

The conductor should choose the **smallest topology that satisfies workload requirements**. More agents are not inherently better; they add model, context and synchronization costs and can duplicate repository discovery.

## Role, resource and plane boundaries

Roles do not replace the existing EOKS planes.

```text
                       CONTROL PLANE
                         CONDUCTOR
                            |
              +-------------+-------------+
              |             |             |
          RETRIEVER      PLANNER       EXECUTOR
              |             |             |
        context/evidence   plan        artifacts
              |             |             |
              +-------------+-------------+
                            |
                   REVIEWER / VALIDATOR
                            |
                    evaluation/outcome
```

The same role may use resources from multiple planes. For example, a reviewer may use a structural graph provider, canonical project knowledge, tests and a model. A retriever may be implemented by a deterministic search service or an agentic tool loop.

This also preserves the existing resource model:

```text
Role
  |
  +--> requires capabilities
              |
              v
          Resources
       /      |       \
    model   tool    provider
              |
              v
        evidence/artifact
```

A role should therefore not be encoded as a new resource type merely because a framework calls its runtime implementation an "agent".

## Role contracts

A workflow node assigning a role should define enough of a **role contract** to make execution and evaluation observable. Depending on the workload, this can include:

- responsibility and objective;
- input/context requirements;
- allowed resources and tools;
- scope and permissions;
- expected outputs/artifacts;
- required validation/evidence;
- independence requirements;
- policy constraints;
- escalation conditions.

The contract should remain workload-specific. EOKS should not prescribe a universal prompt for each role.

## Relationship to reasoning strategies

A role says **what responsibility is being performed**; a reasoning strategy says **how to approach its reasoning step**.

For example, the reviewer role can use:

- adversarial review;
- hypothesis generation/falsification;
- threat modeling;
- architecture review;
- performance investigation.

Likewise, a planner can use divergent exploration before convergence, while an executor can use test-first verification. The same strategy can be used by different roles, and the same role can use different strategies.

## Role traces and evaluation

Runs should make role participation observable. A run can record which role was assigned, which agent/resource implemented it, what context/loadout it received, what it produced and how the result was evaluated.

This enables EOKS to investigate questions such as:

- Does an independent reviewer improve defect detection?
- When does a planner improve outcomes enough to justify its cost?
- Does specialized retrieval reduce repository rediscovery?
- Which validation sources provide sufficient assurance for a given workload?
- When does role separation improve trust versus simply increasing latency and cost?

These are empirical questions. The role taxonomy is an organizational model, not a claim that every workload benefits from every role.

## Design principle

> **Model responsibilities as roles; model runtime implementations as agents/resources; connect them through workflows and evaluate the resulting outcomes.**

This keeps EOKS compositional and avoids turning an agent framework's particular topology into the EOKS architecture.