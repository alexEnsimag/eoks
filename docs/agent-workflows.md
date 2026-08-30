# Agent workflows, orchestration and reasoning strategies

EOKS distinguishes **knowledge**, **workflow**, and **reasoning strategy**:

- Knowledge answers: **What do I know?**
- Workflow answers: **What actions and transitions are allowed?**
- Reasoning strategy answers: **How should I approach this step?**

A fourth useful distinction is **role**: **what responsibility is being performed?** Roles include conductor, retriever, transformer, planner, executor, reviewer and validator. A role is not necessarily a separate agent. See [Agent roles](agent-roles.md) for the canonical role taxonomy.

Orchestration belongs at the **execution/control boundary**. Its semantic purpose is reconciliation: repeatedly comparing desired and observed workload state, selecting an appropriate next action, and using evidence to decide whether the workload has converged. It coordinates reliable execution, verification and escalation without becoming a second knowledge or context system.

## Control loops, workflows and plans

A **control loop** is the semantic behavior of autonomous execution:

```text
desired state + policy
        |
        v
  observe actual state
        |
        v
     reconcile
        |
     decide/action
        |
        v
     execute/verify
        |
        v
   new observations
        |
        +-----------> reconcile
```

A **workflow** is the explicit structure that constrains which actions, responsibilities and transitions are available to that loop. A **plan** is a proposed path through those possibilities. The plan is an executable hypothesis and should be discarded or recomputed when observations, dependencies or policy invalidate its assumptions.

This separation prevents the system from treating an old plan as authoritative state:

```text
intent / policy
      |
      v
desired state
      |
      v
conductor / reconciler
      |
      +--> planner -> proposed plan
      |
      +--> executor / tools -> actions
      |
      +--> retriever / providers -> evidence
      |
      +--> reviewer / validator -> evaluation
      |
      v
actual state / outcome
      |
      +-----------> reconcile
```

The conductor therefore does not need to predict the whole future. Its high-leverage responsibility is to choose the next justified action from current state and evidence.

## Workflow and orchestration

A workflow is an explicit sequence or graph of actions, decisions and validation steps. Each node requests the context it needs rather than carrying the whole project knowledge base.

```text
workflow constraint
       |
       v
role + context/loadout -> relevant evidence
       |
reasoning strategy -> model + tools -> artifact/evidence
       |
       v
observation -> evaluation -> reconciliation
```

The conductor owns the durable coordination state and policy application. Useful state includes desired outcome and acceptance criteria, current observed state, current step, selected context/loadout, assigned role, implementing agent/resource, artifacts/revisions, verification evidence, failures/retries, review findings, approval/escalation state and outcome.

A useful execution lifecycle is therefore better understood as repeated reconciliation than as a one-way state machine:

```text
requested
   |
   v
observe -> plan/propose -> execute -> verify/review
   ^                                  |
   |                                  v
   +--------- reconcile <---- outcome/evidence
                       |
             +---------+---------+
             |         |         |
           retry     re-plan   escalate
             |         |         |
             +---------+---------+
                       |
                    complete
```

The exact states are implementation details. The important property is that progress is evidence-driven and durable, not inferred from an agent's final message.

## Roles are responsibilities, not topology

A workflow assigns responsibilities to roles and selects runtime resources to implement them. One agent can perform several roles, while separate agents or sessions can implement different roles when isolation, independence, parallelism or specialization provides a concrete benefit.

```text
workflow
   |
   +--> role contract
   |       |
   |       +--> context/loadout
   |       +--> allowed resources
   |       +--> required evidence
   |
   `--> agent/resource implementation
```

The conductor should choose the smallest topology that satisfies the workload. Do not create a separate agent merely because a role has a name.

## Model-based execution and replanning

Classical planning research provides a useful architectural refinement for EOKS: an agent can operate over an explicit **workload/world model** while using an LLM as one reasoning component. The model can contain current state, relevant history, beliefs/uncertainty, temporal constraints and policies. Context is then a task-specific projection of that state and the underlying evidence; it is not the canonical state itself.

```text
resources / evidence
        |
        v
   workload state
  /      |       \
state  history  beliefs
  \      |       /
       policies
          |
   context compilation
          |
       reasoning
          |
   proposed action/plan
          |
       execution
          |
      observation
          +------> update state / verify / re-plan
```

This is useful even when no formal MDP/POMDP planner is present. The important separation is between **desired state, actual state, context, reasoning, and execution**. See [Ronen Brafman prior art](../research/prior-art/ronen-brafman-agent-architecture.md).

The planner and conductor should remain conceptually distinct: a **planner proposes an executable hypothesis**, while the **conductor decides whether and how to execute it and reconciles execution against new observations**.

A plan should be treated as an executable hypothesis rather than a guarantee. New observations, failures or changed state can invalidate later steps and should be able to trigger verification, retry, branching, escalation or re-planning.

## Capability/action models

Where useful, execution resources can expose semantics richer than a raw tool schema: inputs, preconditions, effects, duration, side effects, uncertainty, permissions and validation requirements. This creates a useful boundary:

```text
implementation -> capability/action model -> plan -> executor
```

EOKS should adopt this capability-model concept incrementally rather than require every tool to become a formal planning operator. Formal semantics are most valuable when they improve scheduling, safety, composability or verification.

## Long-running and concurrent activities

Real workloads contain actions that take time, have uncertain outcomes or can overlap. A repository scan, CI run, deployment, external API operation or review can be active while other work proceeds.

The runtime should therefore be able to represent durable activity state and observations:

```text
reconciler
  -> start activity
  -> observe progress/outcome
  -> update actual state
  -> verify
  -> continue / retry / branch / re-plan / escalate
```

Concurrency should remain dependency-aware. The goal is not maximal parallelism but correct execution with explicit state and synchronization.

## Smallest useful topology

A practical software-engineering workload does not require an agent swarm:

```text
human goal
    |
    v
conductor / reconciler
    |
    +--> planner (optional)
    +--> executor
    +--> independent reviewer
    |
    v
validator / evaluation
    |
    +--> retry / repair / escalate / complete
```

These are **logical roles**, not necessarily three models or permanently running agents. A single agent can perform several roles sequentially; separate contexts become useful when independence, isolation or parallelism improves results.

The conductor chooses the smallest topology that satisfies workload requirements. It may use one coding-agent session, focused subagents, isolated parallel sessions, an agent team or deterministic tools without another LLM.

## Sequential versus parallel execution

Parallelism is useful for isolated research questions, competing debugging hypotheses, independent review dimensions or separate modules with clear boundaries. Sequential execution is preferable when steps share substantial context, edit the same files or depend tightly on one another. Parallel workers add model and synchronization costs; repository edits should use isolated worktrees or equivalent isolation when concurrent.

## Separate execution from validation

An autonomous coding loop should not end when the executor says it is complete:

```text
reconcile -> implement -> deterministic checks -> independent review
          -> behavioral validation -> reconcile
                                      |
                              accept / repair / escalate
```

The role distinction matters here:

- **Executor** produces the change or other side effect.
- **Reviewer** independently challenges the result.
- **Validator** obtains objective or structured evidence about whether required conditions hold.

Reviewers should have access to execution evidence while retaining enough independence to find executor mistakes. Deterministic evidence—tests, type checks, static analysis, deployment checks and observed behavior—should generally outrank self-reported confidence.

Repair is normally an executor specialization or workflow mode rather than a required permanent agent. Escalation is a control/workflow transition rather than a core agent role.

## Context boundaries

Each invocation should receive the context required for its current responsibility, not the complete workload history by default. A subagent can receive an explicit context contract containing the task, known facts, relevant evidence, scope and unresolved questions, then retrieve missing evidence if necessary.

The conductor coordinates **when** a context or worker is needed; the context plane determines **what** enters the reasoning step. This prevents orchestration from becoming a second knowledge base.

## Human involvement

The goal is not to eliminate humans but to move them to points where judgment and accountability matter most. Production-impacting changes, ambiguous requirements, destructive operations, security-sensitive changes and low-confidence validation may require escalation. Routine implementation, testing, repair and staging validation can become increasingly autonomous when evidence supports it.

Code review is one important human gate, not the only one.

## Multi-agent knowledge boundaries

Multi-agent systems may have shared knowledge while retaining different observations and beliefs. EOKS should preserve scope and provenance rather than assuming that all agents have identical context.

```text
shared knowledge/state
        +
agent-local observations/beliefs
        +
communication history
        +
coordination state
```

Communication can itself be a workflow action. A locally observed fact should not silently become shared canonical knowledge merely because one agent placed it in a message or summary.

## Reasoning strategies

Reasoning strategies are reusable ways of approaching a reasoning step. A strategy can be selected by many workflow nodes, and different strategies can implement the same workflow step.

Examples include divergent exploration before convergence, adversarial review, hypothesis generation/falsification, threat modeling, architecture review, performance investigation and test-first verification. Experiments around so-called "ADHD" skills/stacking are useful prior art for the general reusable-strategy idea; the label is not an EOKS abstraction.

Roles and reasoning strategies should not be conflated. For example, adversarial review is a reasoning strategy that can implement the reviewer role; hypothesis testing can be used by a planner, reviewer or investigator.

## Workflow quality and assurance

A mature workflow should expose checkpoints and evidence requirements rather than trusting agent confidence. Required assurance should be defined by policy and workload risk. Evaluation results are observations for the control loop and can determine whether the workflow advances, retries, branches or escalates.

Trustworthy-agent research also suggests evaluating more than task success: policy compliance, predictability, recovery behavior, auditability and human oversight effort can matter for consequential workloads.

## Continuous learning

Completed runs produce decisions, tool traces, failures, corrections, test outcomes, review feedback and artifacts. These can produce candidate procedures or knowledge, but repeated behavior alone is insufficient for promotion. Outcome-linked evidence, scope and counterexamples matter.

From a control-loop perspective, learning is another reconciliation process: candidate knowledge, Skills or policies are derived from observed outcomes and should be validated and governed before becoming durable state.

See [Memory](memory.md) for the canonical learning/memory lifecycle.

## Natural-language programming

Agentic workflows suggest a higher-level programming abstraction: humans increasingly specify goals, constraints and process while the system expands them into tool calls and code changes.

```text
human intent / policy -> desired state -> workflow constraints
                     -> agent execution -> programming languages / tools
                     -> software artifacts -> observed state
```

This does not imply that deterministic programming languages disappear. Natural language is better viewed as a control/specification layer.

## Existing agents as execution substrates

Coding-agent products are execution substrates, not EOKS architecture. Their subagents, background sessions, isolated worktrees and multi-agent capabilities can implement workflow roles, but EOKS semantics should remain portable.

Conductor-style coding-agent plugins are useful implementation experiments for persistent task/session state, explicit workflow phases, session discovery and lightweight coordination. They should not automatically become EOKS dependencies.

> **Adopt reconciliation primitives, not orchestration ceremony.**

The conductor should make state, policy, evidence and handoffs explicit. It should not create agents, summaries or coordination messages merely because a framework supports them.

## What remains outside orchestration

The conductor is not the canonical home for project knowledge, repository structure, long-term memory, context-selection algorithms, deterministic analyzers, model-specific prompts, deployment infrastructure or evaluation definitions. It references and coordinates those capabilities.

## Minimal implementation hypothesis

```text
Task / desired outcome
  |
  v
Reconcile current state
  |
  +-- Plan       (planner role, optional)
  +-- Execute    (executor role)
  +-- Review     (reviewer role, fresh context)
  +-- Verify     (validator role)
  +-- Outcome / actual state
  |
  +---------------------> reconcile again if not converged
```

Persist the state and evidence needed to resume and audit the loop. Plans, contexts, caches and agent sessions should be replaceable where practical. Add parallel workers, richer routing, durable workflow engines or autonomous escalation only when observed workloads demonstrate that the simpler model is insufficient.

The high-leverage layer is deciding what should happen next, which resources and evidence are appropriate, and what evidence is sufficient to advance the workload—not creating the largest possible agent graph.
