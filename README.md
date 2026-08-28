# EOKS

> An experimental operating/control layer for reliable AI workloads.

EOKS explores what an **AI operating system / control plane** could look like if context, memory, models, tools, tasks, execution, evaluation and observability were treated as coordinated resources.

The project grew out of a recurring question: **why does giving models more context not necessarily give them better understanding?** The working answer is that AI systems need infrastructure for selecting, structuring, persisting, evaluating and routing information—not simply larger prompts.

## The current thesis

EOKS is not a prompt framework, vector database, memory store, agent framework or model router. Those are potential components. The proposed abstraction is the layer that coordinates them around a workload and continuously learns from outcomes.

A recent refinement is important: **knowledge is not a graph, and context engineering is not the knowledge layer**. EOKS should maintain multiple representations of engineering reality and compile task-specific context from them.

```text
                         EOKS CONTROL PLANE
              scheduling · policies · resource selection
                                  |
        +-------------------------+-------------------------+
        |                         |                         |
  KNOWLEDGE PLANE          CONTEXT PLANE            EXECUTION PLANE
 canonical knowledge       selection · assembly      workflows · agents
 graphs · semantic         ranking · compression     reasoning strategies
 history · runtime         progressive disclosure    tools · artifacts
 evidence
        |                         |                         |
        +-------------------------+-------------------------+
                                  |
                         EVALUATION / FEEDBACK
                  evidence · quality · confidence · outcomes
                                  |
                           OBSERVABILITY
                decisions · cost · latency · provenance
```

## Documentation

- [Vision](docs/vision.md)
- [Architecture](docs/architecture.md)
- [Context engineering](docs/context.md)
- [Context Workbench architecture](docs/context-workbench.md)
- [Knowledge base](docs/knowledge-base.md)
- [Knowledge as a multi-representation system](docs/knowledge-representations.md)
- [Agent workflows and reasoning strategies](docs/agent-workflows.md)
- [Memory](docs/memory.md)
- [Behavioral memory and learning](docs/behavioral-memory.md)
- [Control plane](docs/control-plane.md)
- [Evaluation](docs/evaluation.md)
- [Software engineering](docs/software-engineering.md)
- [Software analysis, dataflow and invariants](docs/software-analysis.md)
- [Prior art](docs/prior-art.md)
- [Terminology](docs/terminology.md)
- [Open questions](docs/open-questions.md)

## Practical coding-agent direction

The current practical hypothesis is intentionally conservative:

1. Use hierarchical `CLAUDE.md` files as human-reviewable, canonical package/project knowledge.
2. Keep cross-cutting architectural rationale in a small number of Markdown/ADR documents.
3. Derive structural information automatically with deterministic analysis and optional graphs.
4. Use context compilation to select only task-relevant evidence.
5. Treat context as an inspectable, budgeted artifact; the proposed Context Workbench provides blocks, provenance, selection rationale, context diffs and a graph view without requiring manual curation for every task.
6. Treat agent workflows and reasoning strategies as execution-layer resources.
7. Learn continuously through candidate extraction and controlled promotion rather than silently rewriting canonical knowledge.
8. Update representations incrementally; do not rebuild the entire knowledge base after every change.
9. For software engineering, prefer the cheapest reliable deterministic evidence source that answers the question: types before custom analysis, lightweight rules before deep dataflow, and deeper analyzers only when the invariant requires them.

This gives EOKS a path from a simple Git repository to richer knowledge infrastructure without requiring a graph database or a new canonical format on day one.

## Research

- [Agent code understanding and architecture tooling](research/agent-code-understanding-and-architecture.md) — how repository knowledge graphs, deterministic analysis, architecture governance and AI coding tools can map onto EOKS.
- [Context engineering](research/context-engineering.md)
- [Context quality model](research/context-quality-model.md)
- [Context workbench](research/context-workbench.md)
- [Evaluation and model switching](research/evaluation-and-model-switching.md)
- [Control loop](research/control-loop.md)
- [Behavioral memory and learning how developers work](docs/behavioral-memory.md) — how session traces can become validated procedural knowledge, skills and policies.
- [Learning from development sessions](research/session-learning.md)
- [Agent memory and continuous-learning prior art](research/prior-art/agent-memory.md)
- [Claude Code learning stack](research/claude-learning-okf-hindsight.md) — decomposition of `CLAUDE.md`, Skills, Hooks, memory engines, OKF and optional graph/retrieval systems.
- [Hindsight and OKF](research/prior-art/hindsight-and-okf.md) — why persistent learning engines and portable knowledge formats are complementary layers.
- [Tool notes](research/tool-notes.md)

## Status

Research / architecture / prototype stage. The architecture is intentionally provisional.

This repository is a **living specification and laboratory**: ideas should become documented hypotheses, experiments, implementations and decisions rather than a transcript of discussions.
