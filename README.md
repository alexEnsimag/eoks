# EOKS

> An experimental operating/control layer for reliable AI workloads.

EOKS explores what an **AI operating system / control plane** could look like if context, memory, models, tools, tasks, execution, evaluation and observability were treated as coordinated resources.

The project grew out of a recurring question: **why does giving models more context not necessarily give them better understanding?** The working answer is that AI systems need infrastructure for selecting, structuring, persisting, evaluating and routing information—not simply larger prompts.

## The current thesis

EOKS is not a prompt framework, vector database, memory store, agent framework or model router. Those are potential components. The proposed abstraction is the layer that coordinates them around a workload and continuously learns from outcomes.

```text
                 EOKS CONTROL PLANE
        scheduling · policies · model/tool selection
                            |
       +--------------------+--------------------+
       |                    |                    |
    CONTEXT              MEMORY               EXECUTION
    retrieval            knowledge             tools/agents
    assembly             persistence           workflows
       |                    |                    |
       +--------------------+--------------------+
                            |
                     EVALUATION
                  evidence · quality
                            |
                     OBSERVABILITY
              decisions · cost · outcomes
```

## Documentation

- [Vision](docs/vision.md)
- [Architecture](docs/architecture.md)
- [Context engineering](docs/context.md)
- [Memory](docs/memory.md)
- [Control plane](docs/control-plane.md)
- [Evaluation](docs/evaluation.md)
- [Software engineering](docs/software-engineering.md)
- [Prior art](docs/prior-art.md)
- [Terminology](docs/terminology.md)
- [Open questions](docs/open-questions.md)

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
- [Tool notes](research/tool-notes.md)

## Status

Research / architecture / prototype stage. The architecture is intentionally provisional.

This repository is a **living specification and laboratory**: ideas should become documented hypotheses, experiments, implementations and decisions rather than a transcript of discussions.
