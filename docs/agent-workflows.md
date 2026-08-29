# Agent workflows, orchestration and reasoning strategies

EOKS distinguishes **knowledge**, **workflow**, and **reasoning strategy**:

- Knowledge answers: **What do I know?**
- Workflow answers: **What should happen next?**
- Reasoning strategy answers: **How should I approach this step?**

Orchestration belongs at the **execution/control boundary**. It coordinates reliable execution, verification and escalation without becoming a second knowledge or context system.

## Workflow and orchestration

A workflow is an explicit sequence or graph of actions, decisions and validation steps. Each node requests the context it needs rather than carrying the whole project knowledge base.

```text
workflow node -> context selection -> relevant evidence
             -> reasoning strategy -> model + tools -> artifact/evidence
```

The orchestrator/conductor owns workflow state and policy. Useful state includes workload and acceptance criteria, current step, selected context/loadout, artifacts/revisions, verification evidence, failures/retries, review findings, approval/escalation state and outcome.

A useful lifecycle is:

```text
requested -> planned -> executing -> verifying -> reviewing -> completed
                                  |                    |
                                  +--> retry/repair    +--> escalate
```

The exact states are implementation details. The important property is that progress is evidence-driven and durable, not inferred from an agent's final message.

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

This is useful even when no formal MDP/POMDP planner is present. The important separation is between **state**, **context**, **reasoning**, and **execution**. See [Ronen Brafman prior art](../research/prior-art/ronen-brafman-agent-architecture.md).

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
plan
  -> start activity
  -> observe progress/outcome
  -> update state
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
conductor
    |
    +--> executor
    +--> independent reviewer
    |
    v
verification / evaluation
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
plan -> implement -> deterministic checks -> independent review
     -> behavioral validation -> accept / repair / escalate
```

Reviewers should have access to execution evidence while retaining enough independence to find executor mistakes. Deterministic evidence—tests, type checks, static analysis, deployment checks and observed behavior—should generally outrank self-reported confidence.

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

## Workflow quality and assurance

A mature workflow should expose checkpoints and evidence requirements rather than trusting agent confidence. Required assurance should be defined by policy and workload risk. Evaluation results can determine whether the workflow advances, retries, branches or escalates.

Trustworthy-agent research also suggests evaluating more than task success: policy compliance, predictability, recovery behavior, auditability and human oversight effort can matter for consequential workloads.

## Continuous learning

Completed runs produce decisions, tool traces, failures, corrections, test outcomes, review feedback and artifacts. These can produce candidate procedures or knowledge, but repeated behavior alone is insufficient for promotion. Outcome-linked evidence, scope and counterexamples matter.

See [Memory](memory.md) for the canonical learning/memory lifecycle.

## Natural-language programming

Agentic workflows suggest a higher-level programming abstraction: humans increasingly specify goals, constraints and process while the system expands them into tool calls and code changes.

```text
human intent / policy -> workflow specification -> agent execution
                     -> programming languages / tools -> software artifacts
```

This does not imply that deterministic programming languages disappear. Natural language is better viewed as a control/specification layer.

## Existing agents as execution substrates

Coding-agent products are execution substrates, not EOKS architecture. Their subagents, background sessions, isolated worktrees and multi-agent capabilities can implement workflow roles, but EOKS semantics should remain portable.

Conductor-style coding-agent plugins are useful implementation experiments for persistent task/session state, explicit workflow phases, session discovery and lightweight coordination. They should not automatically become EOKS dependencies.

> **Adopt orchestration primitives, not orchestration ceremony.**

The conductor should make state, policy, evidence and handoffs explicit. It should not create agents, summaries or coordination messages merely because a framework supports them.

## What remains outside orchestration

The conductor is not the canonical home for project knowledge, repository structure, long-term memory, context-selection algorithms, deterministic analyzers, model-specific prompts, deployment infrastructure or evaluation definitions. It references and coordinates those capabilities.

## Minimal implementation hypothesis

```text
Task
  |
  +-- Plan
  +-- Execute (coding agent)
  +-- Review (fresh context)
  +-- Verify
  +-- Outcome
```

Persist only state needed to resume and audit the run. Add parallel workers, richer routing, durable workflow engines or autonomous escalation only when observed workloads demonstrate that the simpler model is insufficient.

The high-leverage layer is deciding what should happen next, which resources and evidence are appropriate, and what evidence is sufficient to advance the workload—not creating the largest possible agent graph.
