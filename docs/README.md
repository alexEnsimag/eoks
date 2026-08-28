# EOKS documentation

`docs/` is the **current architectural and design layer** of EOKS. These documents describe the working model that the project currently proposes. They are intentionally more coherent and prescriptive than the research corpus, but the architecture remains provisional unless a document is explicitly recorded as a decision.

## How the repository is organized

- **`README.md`** — project entry point and practical direction.
- **`docs/`** — current architecture, terminology, decisions, research agenda and open questions.
- **`research/`** — exploratory notes, prior art, experiments and the reasoning that led to the current model. Research may contain competing hypotheses and should not silently be treated as architecture.

A useful evidence hierarchy is:

```text
research / external prior art
        |
        v
architectural hypothesis
        |
        v
experiment / prototype
        |
        v
decision (when evidence justifies it)
```

## Current architecture

- [`architecture.md`](architecture.md) — overall planes, runtime primitives and control loop.
- [`context.md`](context.md) — context engineering and compilation.
- [`context-workbench.md`](context-workbench.md) — inspectable context blocks, budgets and human control.
- [`knowledge-base.md`](knowledge-base.md) — durable project knowledge and its lifecycle.
- [`knowledge-representations.md`](knowledge-representations.md) — multiple representations of engineering reality.
- [`memory.md`](memory.md) — memory types, lifecycle and promotion.
- [`behavioral-memory.md`](behavioral-memory.md) — learning from development behavior.
- [`agent-workflows.md`](agent-workflows.md) — workflows versus reasoning strategies.
- [`control-plane.md`](control-plane.md) — scheduling, reconciliation, policies and model selection.
- [`evaluation.md`](evaluation.md) — evaluation, reliability evidence and calibration.
- [`software-engineering.md`](software-engineering.md) — software-engineering workload and agent practices.
- [`software-analysis.md`](software-analysis.md) — invariants, dataflow and analyzer escalation.

## Project governance and research

- [`decisions.md`](decisions.md) — current architectural decisions; unresolved ideas belong in research/open questions instead.
- [`research-agenda.md`](research-agenda.md) — experiments intended to validate or falsify the architecture.
- [`open-questions.md`](open-questions.md) — unresolved questions that should not be mistaken for design commitments.
- [`prior-art.md`](prior-art.md) — consolidated prior-art landscape.
- [`terminology.md`](terminology.md) — shared working vocabulary.
- [`vision.md`](vision.md) — long-term direction.
- [`history.md`](history.md) — evolution of the project and its hypotheses.

## Document ownership rule

Avoid maintaining the same architectural claim independently in several documents. When a concept has a canonical home, other documents should summarize it and link there. Research notes may repeat a concept when necessary to explain how it was discovered, but should label it as a hypothesis or observation rather than creating a second source of truth.
