# Prior art and adjacent systems

This is a research map, not a claim that the projects below implement EOKS. They are useful because each exposes part of the problem.

## GrapeRoot

GrapeRoot was a recurring reference point for thinking about context and agent execution. The important question was whether a system around an agent can maintain richer project state and decide what the model should see rather than repeatedly handing the model raw history.

For EOKS, the interesting contribution is the **context/memory/execution boundary**: what belongs to persistent project state versus the current model invocation.

## CodeSight

CodeSight was investigated as a code-context management approach. It is relevant to EOKS because software agents need repository understanding beyond the immediate prompt.

The comparison question is whether CodeSight is primarily a context/knowledge layer or whether its abstractions naturally extend into workload scheduling, evaluation and control.

## EKOS

EKOS entered the discussion as another attempt to frame AI/enterprise knowledge as a system rather than a collection of documents. The name is also potentially confusing because “EKOS” is used by unrelated projects.

The EOKS distinction is intentional: EOKS is broader than enterprise knowledge management and is interested in **runtime control of AI workloads**, with knowledge as one managed resource.

## TencentDB Agent Memory

The TencentDB Agent Memory work reinforced the idea that increasing the context window is not the same thing as giving an agent useful long-term memory.

The key lesson for EOKS is to treat memory as a retrieval and lifecycle problem: store useful experience, retrieve selectively, preserve provenance, and prevent stale memories from becoming authoritative context.

## XIRP / Spotify

XIRP was explored as an example of how a large engineering organization approaches AI infrastructure and contextual systems. It is useful prior art for thinking about the boundary between application-level agents and shared platform capabilities.

The EOKS question is whether common infrastructure should expose a standardized control/knowledge plane across many AI workloads.

## OKF

OKF was discussed less as a hosted service and more as a convention for organizing knowledge/context artifacts. An important observation was that a useful format does not necessarily require a centralized server: the value may be in the structure and conventions.

This led to the broader EOKS principle that **semantic structure matters more than a particular file format**.

## Graphify

Graphify was investigated as a way of extracting relationships from code. It helped expose the potential of graph context but also the limitations of relying on graph extraction alone.

Graphs are useful for discovery, traversal and explanation; EOKS should not assume that a graph by itself solves context selection.

## CodeQL

CodeQL represents a powerful deterministic query/dataflow approach. The discussion questioned whether it is too heavyweight for ordinary agent tasks.

The more useful EOKS interpretation is workload-dependent orchestration: use expensive analysis when the expected value of stronger evidence justifies its cost.

## Semgrep

Semgrep provides a lighter-weight structural/pattern-analysis alternative for many code tasks. It illustrates the same point: EOKS can select analysis tools according to task type rather than making every workload pay for the strongest possible analysis.

## Observability and continuous assurance

We also explored AI/LLM observability and continuous-assurance tooling as potential sources of signals for model behavior, evaluation and confidence.

The unresolved question is whether existing observability systems primarily record traces or whether they can become active inputs to an AI control loop.

## Herdr and cmux

Herdr and cmux sharpen the boundary between an **agent runtime/harness** and the semantic layer EOKS is intended to provide. Herdr is a server-owned terminal runtime with first-class agent state and automation primitives; cmux is an agent-oriented terminal/workspace surface with particularly strong Claude Teams integration.

The EOKS lesson is not to build another terminal multiplexer. Runtime persistence answers **where the work continues**; context evolution answers **what from that work should continue to matter**. Runtime events such as completion, blocking, handoff and validation can therefore become semantic boundaries for selective knowledge/experience extraction.

See [Herdr and cmux: agent runtime versus harness surface](prior-art/herdr-cmux.md) for the detailed comparison and architectural implications.

## What the prior art suggests

Taken together, these systems suggest a fragmented stack:

```text
knowledge / memory
       +
context management
       +
code intelligence
       +
LLM execution
       +
tool orchestration
       +
agent runtime / harness
       +
observability
       +
evaluation
```

EOKS is hypothesized to be the **coordination layer across these capabilities**, but “coordination” should not be interpreted as owning every underlying mechanism.

A more precise hypothesis is:

> **EOKS coordinates semantic resources, context, policy, assurance and learning across existing execution and harness capabilities.**
>
> **It should consume runtime primitives and events rather than reimplementing terminal/process infrastructure.**

That distinction makes the EOKS scope more testable and prevents the project from becoming an undifferentiated agent runtime.
