# Agent orchestration

EOKS should treat **agent orchestration as an execution/control concern**, not as a synonym for multi-agent conversation. The purpose of orchestration is to turn a workload into a reliable sequence of execution, verification and escalation steps while keeping knowledge and context reusable across runs.

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

These are **logical roles**, not necessarily three different models or three permanently running agents. A single strong agent can perform several roles sequentially; separate contexts become valuable when independence, isolation or parallelism improves the result.

## Orchestration is more than delegation

The orchestrator should own workflow state and policy rather than asking agents to coordinate themselves through prose. Important state includes:

- workload and acceptance criteria;
- current workflow step;
- selected context/loadout;
- artifacts and revisions;
- verification evidence;
- failures and retry history;
- review findings;
- approval/escalation state;
- final outcome.

A useful lifecycle is:

```text
requested
   -> planned
   -> executing
   -> verifying
   -> reviewing
   -> completed
        |
        +--> retry / repair
        +--> blocked / escalated
```

The exact states are implementation details; the important property is that **progress is evidence-driven and durable**, rather than inferred from an agent's final message.

## Context boundaries are an orchestration primitive

Each agent invocation should receive the context required for its current responsibility, not the complete history of the workload by default. This follows the EOKS context model:

```text
workflow step
     |
     v
loadout / policy
     |
     v
context compilation
     |
     v
agent + tools
     |
     v
artifact / evidence
```

The conductor therefore coordinates **when** a context is needed, while the context plane determines **what** should enter it. This avoids turning the orchestrator into a second knowledge base.

## Sequential versus parallel agents

Parallelism is useful when work can be isolated:

- independent research questions;
- competing debugging hypotheses;
- independent review dimensions;
- separate modules with clear ownership;
- cross-layer work with explicit file boundaries.

Sequential execution is preferable when steps share substantial context, edit the same files, or depend tightly on one another. Parallel agents also multiply model usage and create synchronization overhead.

For repository modifications, isolated worktrees or equivalent execution isolation are preferable when multiple agents edit concurrently.

## Separate execution from validation

An autonomous coding loop should not end when the executor says the task is complete. A robust loop is:

```text
plan
  |
implement
  |
run deterministic checks
  |
independent review
  |
behavioral validation
  |
accept / repair / escalate
```

The reviewer and validation stages should have access to evidence produced by execution, but should retain enough independence to discover executor mistakes. Deterministic evidence—tests, type checks, static analysis, deployment checks and observed behavior—should outrank self-reported confidence.

## Human involvement

The goal is not to eliminate humans from the workflow but to move them to the points where judgment and accountability matter most.

A useful progression is:

```text
early adoption
  human: goal + review + deployment decisions

mature workflow
  human: goal + ambiguous/risky decisions + production approval

highly trusted workflow
  human: objectives, policy exceptions and accountability
```

Code review is therefore **one** important human gate, but it is not the only possible gate. Production-impacting changes, ambiguous requirements, destructive operations, security-sensitive changes and low-confidence validation may require escalation. Conversely, routine implementation, testing, repair and staging validation can become increasingly autonomous when evidence supports it.

## Conductor versus agent framework

An EOKS conductor should be understood as a **workflow coordinator**, not as a framework that forces every workload into a multi-agent pattern. It may use:

- one coding-agent session;
- custom subagents for focused side tasks;
- isolated parallel sessions;
- a small agent team for genuinely collaborative work;
- deterministic tools without another LLM at all.

The conductor chooses the smallest topology that satisfies the workload's requirements.

## Claude Code as a concrete execution substrate

Claude Code provides a useful reference implementation for these concepts. Its current primitives include custom subagents, background sessions, isolated worktrees and experimental agent teams. Custom subagents are particularly useful for reusable roles such as a read-only explorer, test/validation worker or independent reviewer. Agent teams add direct teammate communication and shared task coordination, but introduce substantially more token and coordination overhead. citehttps://code.claude.com/docs/en/sub-agents

Agent teams should therefore be treated as an optional scaling mechanism, not the default architecture. Claude Code's documentation explicitly positions subagents for focused delegated work and agent teams for cases where independent workers need to communicate and coordinate. Agent teams are experimental and disabled by default. citehttps://code.claude.com/docs/en/agent-teams

This distinction maps naturally onto EOKS:

```text
EOKS workflow/control plane
          |
          +--> single Claude Code session
          +--> Claude Code subagent
          +--> isolated session/worktree
          +--> agent team
          +--> deterministic tool
```

The execution substrate is replaceable; the EOKS workflow semantics should not depend on a particular coding-agent product.

## Conductor-style plugins

Conductor-style Claude Code plugins are useful **implementation experiments** for the orchestration concept: persistent task/session state, explicit workflow phases, session discovery and lightweight coordination can be valuable primitives. They should not automatically become an EOKS architectural dependency.

For example, the community `claude-conductor` project exposes a conductor skill/commands and tracks session states such as planning, coding, reviewing, blocked and done. This is useful prior art for a thin orchestration layer, but it is an implementation of the idea rather than the EOKS abstraction itself. citehttps://github.com/code-katz/claude-conductor

The architectural rule is:

> **Adopt orchestration primitives, not orchestration ceremony.**

A conductor should make state, policy, evidence and handoffs explicit. It should not create agents, summaries or coordination messages merely because a framework supports them.

## What should remain outside the conductor

The conductor should not become the canonical home for:

- project knowledge;
- repository structure;
- long-term memory;
- context-selection algorithms;
- deterministic analyzers;
- model-specific prompts;
- deployment infrastructure;
- evaluation definitions.

Those belong to the corresponding EOKS resource, context, evaluation or execution layers. The conductor references and coordinates them.

## Minimal implementation hypothesis

A useful first prototype can be extremely small:

```text
Task
  |
  +-- Plan
  |
  +-- Execute (coding agent)
  |
  +-- Review (fresh context)
  |
  +-- Verify
  |
  +-- Outcome
```

Persist only the state needed to resume and audit the run. Add parallel workers, richer routing, durable workflow engines or autonomous escalation only when observed workloads demonstrate that the simpler model is insufficient.

This keeps orchestration aligned with the broader EOKS thesis: **the high-leverage layer is deciding what should happen next, which resources and evidence are appropriate, and what evidence is sufficient to advance the workload—not creating the largest possible agent graph.**
