# EOKS vision

## Thesis

EOKS explores an operating/control layer for AI engineering: a system that manages **tasks, context, memory, models, tools, execution, evaluation, and observability** as coordinated resources.

The central observation is that increasing an LLM's context window is not equivalent to giving it better context. Reliable AI systems need to decide what information is relevant, how it is represented, when it is retrieved, what should persist, which model should process it, and how the resulting work is evaluated.

## The outcome EOKS is trying to unlock

The practical goal is **trustable automated software-engineering workflows with AI agents**.

Today, agents can often perform useful work but still require substantial human verification because it is difficult to know whether they understood the system correctly, respected its constraints, and produced a complete and safe result. EOKS explores the infrastructure needed to reduce that uncertainty through better knowledge, context, planning, execution and independent assurance.

The four primary engineering outcomes are:

- **Velocity** — reach useful outcomes faster.
- **Quality** — produce correct, coherent and resilient changes with less rework.
- **Trust / autonomy** — safely delegate more engineering work to agents.
- **Cost / scale** — increase useful engineering output without proportional human effort or AI/resource cost.

See [Engineering outcomes and graduated autonomy](engineering-outcomes.md) for the measurement model and autonomy progression.

## From context engineering to an AI control plane

The project started from context engineering and evolved toward a broader systems question:

> What would Kubernetes-like infrastructure look like if the workload were AI reasoning rather than containers?

The analogy is deliberately incomplete. EOKS should not copy Kubernetes; it should borrow the useful idea of a declarative control plane that schedules work against resources and continuously observes outcomes.

## Core resources

- **Task** — the unit of work and its requirements.
- **Context** — information assembled for a particular reasoning step.
- **Memory** — information intentionally retained across steps or sessions.
- **Knowledge** — durable, structured information available to the system.
- **Model** — a reasoning/completion capability with cost, latency, capability and reliability characteristics.
- **Tool** — an external capability such as code search, static analysis, a repository API or execution environment.
- **Policy** — constraints governing selection, retrieval, execution and safety.
- **Evaluation** — evidence about whether a result was correct, useful or robust.

## What EOKS is not

EOKS is not simply:

- a prompt template;
- a vector database;
- an agent framework;
- a memory database;
- an observability dashboard;
- a model router;
- a code-search tool.

Those may all be components of an EOKS implementation.

## Working principles

1. **Context is a managed resource.**
2. **Context quality matters independently of context size.**
3. **Memory is not a transcript.**
4. **Deterministic analysis and LLM reasoning should complement each other.**
5. **Evaluation is part of the runtime, not a post-hoc activity.**
6. **Model choice should be task- and evidence-dependent.**
7. **The system should make its information and execution decisions inspectable.**
8. **Trust comes from evidence and assurance, not from agent self-confidence.**
9. **Autonomy should be graduated according to risk and assurance.**
10. **Experiments should be reproducible and falsifiable.**
11. **The architecture should remain model-agnostic.**
12. **The repository is a living specification, not a dump of conversation history.**
