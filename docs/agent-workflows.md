# Agent workflows, orchestration and reasoning strategies

EOKS should distinguish **knowledge**, **workflow**, and **reasoning strategy**. They are related but not interchangeable.

- Knowledge answers: **What do I know?**
- Workflow answers: **What should happen next?**
- Reasoning strategy answers: **How should I approach this step?**

Agent orchestration belongs at the **execution/control boundary**: it turns a workload into a reliable sequence of execution, verification and escalation steps while keeping knowledge and context reusable across runs.

## Workflow is an execution layer

A workflow is an explicit sequence or graph of actions, decisions and validation steps. The workflow should not contain the entire project knowledge base. Each node requests the context it needs from the knowledge/context planes.

```text
workflow node
     |
     v
context selection
     |
     v
relevant evidence
     |
     v
reasoning strategy
     |
     v
model + tools
```

This separation allows the same knowledge to support multiple workflows and the same workflow to run over different models.

## Orchestration is more than delegation

An orchestrator/conductor owns workflow state and policy rather than asking agents to coordinate themselves through prose. Important state includes workload and acceptance criteria, current step, selected context/loadout, artifacts and revisions, verification evidence, failures/retry history, review findings, approval/escalation state and final outcome.

A useful lifecycle is:

```text
requested -> planned -> executing -> verifying -> reviewing -> completed
                                  |                    |
                                  +--> retry/repair    +--> escalate
```

The exact states are implementation details; the important property is that **progress is evidence-driven and durable**, rather than inferred from an agent's final message.

The conductor coordinates **when** a context or worker is needed. The context plane determines **what** information should enter a reasoning step. This prevents the orchestrator from becoming a second knowledge base.

## Start with the smallest useful topology

A practical software-engineering workload does not require an agent swarm. A useful initial topology is:

```text
human goal
    |
    v
orchestrator / conductor
    |
    +--> executor
    |
    +--> independent reviewer
    |
    v
verification / evaluation
    |
    +--> retry / repair
    +--> escalate
    +--> complete
```

The three roles have deliberately different responsibilities:

- **Conductor** — understands the goal, decomposes the work, chooses the next step, tracks state and decides when verification or escalation is required.
- **Executor** — changes the system, runs tools and tests, and iterates on implementation failures.
- **Reviewer** — independently challenges the implementation for correctness, regressions, security, architecture and completeness. It should not simply inherit the executor's reasoning as authoritative.

These are **logical roles**, not necessarily three different models or permanently running agents. A single strong agent can perform several roles sequentially; separate contexts become valuable when independence, isolation or parallelism improves the result.

## Conductor versus agent framework

An EOKS conductor should be understood as a **workflow coordinator**, not as a framework that forces every workload into a multi-agent pattern. It may use one coding-agent session, focused subagents, isolated parallel sessions, a small agent team, or deterministic tools without another LLM. The conductor chooses the smallest topology that satisfies the workload's requirements.

## Sequential versus parallel execution

Parallelism is useful when work can be isolated: independent research questions, competing debugging hypotheses, independent review dimensions, separate modules with clear ownership, or cross-layer work with explicit file boundaries.

Sequential execution is preferable when steps share substantial context, edit the same files, or depend tightly on one another. Parallel agents also multiply model usage and create synchronization overhead. For repository modifications, isolated worktrees or equivalent execution isolation are preferable when multiple agents edit concurrently.

## Separate execution from validation

An autonomous coding loop should not end when the executor says the task is complete:

```text
plan -> implement -> deterministic checks -> independent review -> behavioral validation -> accept / repair / escalate
```

The reviewer and validation stages should have access to evidence produced by execution, but retain enough independence to discover executor mistakes. Deterministic evidence—tests, type checks, static analysis, deployment checks and observed behavior—should outrank self-reported confidence.

## Context boundaries are an orchestration primitive

Each invocation should receive the context required for its current responsibility, not the complete history of the workload by default. A subagent can receive an explicit context contract containing the task, known facts, relevant nodes, scope and unresolved questions. It remains free to retrieve missing evidence.

```text
workflow step -> loadout/policy -> context compilation -> agent + tools -> artifact/evidence
```

If the agent discovers that the starting contract is incomplete, the additional evidence should be recorded and can feed back into the context compiler.

## Human involvement

The goal is not to eliminate humans from the workflow but to move them to points where judgment and accountability matter most. Production-impacting changes, ambiguous requirements, destructive operations, security-sensitive changes and low-confidence validation may require escalation. Routine implementation, testing, repair and staging validation can become increasingly autonomous when evidence supports it.

Code review is **one** important human gate, not the only one.

## Reasoning strategies are another reusable layer

Reasoning strategies are reusable ways of approaching a reasoning step. They are different from workflows because a strategy can be selected by many workflow nodes, and different strategies can implement the same workflow step.

Examples include divergent exploration before convergence, adversarial review, hypothesis generation and falsification, threat modeling, architecture review, performance investigation and test-first verification.

The so-called "ADHD" skills/stacking experiments are useful prior art for this general idea; the label is less important than the reusable-strategy abstraction.

## Workflows are not a replacement for knowledge

A workflow can say `understand -> design -> implement -> verify`; it does not know the architecture itself. The context/knowledge system supplies the evidence needed by each step. Conversely, a knowledge base can contain excellent architectural facts without defining how an agent should act on them.

## Workflow quality and assurance

A mature workflow should expose checkpoints and evidence requirements rather than trusting agent confidence. Required assurance should be defined by policy and workload risk. Evaluation results can determine whether the workflow advances, retries, branches or escalates.

## Continuous learning

A completed run produces decisions, tool traces, failures, corrections, test outcomes, review feedback and final artifacts. These can produce **candidate** procedures or knowledge, but repeated behavior alone is not enough to promote a rule. Outcome-linked evidence, scope and counterexamples should be considered before promotion.

See [Memory](memory.md) for the canonical learning/memory lifecycle.

## Relationship to natural-language programming

Agentic workflows suggest a higher-level programming abstraction: humans increasingly specify goals, constraints and process, while the system expands them into tool calls and code changes. This does not imply that programming languages disappear.

```text
human intent / policy
        |
workflow specification
        |
agent execution
        |
programming languages / tools
        |
software artifacts
```

Natural language is better viewed as a new control/specification layer than as an immediate replacement for deterministic programming languages.

## Claude Code as an execution substrate

Claude Code provides a useful reference implementation for these concepts. Its primitives include custom subagents, background sessions and isolated worktrees; experimental agent teams add direct teammate communication and shared task coordination. Custom subagents are useful for reusable roles such as a read-only explorer, test/validation worker or independent reviewer.

Agent teams should be treated as an **optional scaling mechanism**, not the default architecture. The execution substrate is replaceable; EOKS workflow semantics should not depend on a particular coding-agent product.

## Conductor-style implementations

Conductor-style coding-agent plugins are useful **implementation experiments** for orchestration: persistent task/session state, explicit workflow phases, session discovery and lightweight coordination can be valuable primitives. They should not automatically become EOKS dependencies.

> **Adopt orchestration primitives, not orchestration ceremony.**

A conductor should make state, policy, evidence and handoffs explicit. It should not create agents, summaries or coordination messages merely because a framework supports them.

## What should remain outside orchestration

The conductor should not become the canonical home for project knowledge, repository structure, long-term memory, context-selection algorithms, deterministic analyzers, model-specific prompts, deployment infrastructure or evaluation definitions. Those belong to the corresponding EOKS resource, context, evaluation or execution layers.

## Minimal implementation hypothesis

A useful first prototype can be extremely small:

```text
Task
  |
  +-- Plan
  +-- Execute (coding agent)
  +-- Review (fresh context)
  +-- Verify
  +-- Outcome
```

Persist only the state needed to resume and audit the run. Add parallel workers, richer routing, durable workflow engines or autonomous escalation only when observed workloads demonstrate that the simpler model is insufficient.

The high-leverage layer is deciding what should happen next, which resources and evidence are appropriate, and what evidence is sufficient to advance the workload—not creating the largest possible agent graph.
