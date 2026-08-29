# EOKS documentation

`docs/` is the **current architectural and design layer** of EOKS. These documents describe the working model that the project currently proposes. They are more coherent and prescriptive than the research corpus, but the architecture remains provisional unless a document is explicitly recorded as a decision.

## How the repository is organized

- **`README.md`** — project entry point and practical direction.
- **`docs/`** — current architecture, terminology, decisions, research agenda and open questions.
- **`research/`** — exploratory reasoning, prior art and experiments. Research may contain competing hypotheses and should not silently be treated as architecture.

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
- [`resource-model.md`](resource-model.md) — canonical vocabulary for resources, assets, providers, representations, loadouts and context.
- [`context.md`](context.md) — context engineering and context compilation.
- [`context-workbench.md`](context-workbench.md) — inspectable context blocks, budgets and human control.
- [`knowledge-base.md`](knowledge-base.md) — durable project knowledge and its lifecycle.
- [`knowledge-representations.md`](knowledge-representations.md) — multiple representations of engineering reality.
- [`memory.md`](memory.md) — semantic, episodic and procedural memory plus behavioral learning.
- [`agent-workflows.md`](agent-workflows.md) — workflows, orchestration and reusable reasoning strategies.
- [`control-plane.md`](control-plane.md) — scheduling, reconciliation, policies and model selection.
- [`evaluation.md`](evaluation.md) — evaluation, reliability evidence and calibration.
- [`software-engineering.md`](software-engineering.md) — software-engineering workloads and agent practices, including enforceable architecture.
- [`software-analysis.md`](software-analysis.md) — invariants, dataflow and analyzer escalation.
- [`tool-capability-model.md`](tool-capability-model.md) — canonical capability/selection model for evidence providers.
- [`tool-selection.md`](tool-selection.md) — evidence requirements and task-specific provider selection.
- [`tool-landscape.md`](tool-landscape.md) — current visual map, tool families, experiment shortlist and known evidence gaps.

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

In particular:

- **Resource/Asset/Provider/Representation/Loadout/Context definitions** belong in `resource-model.md` and the compact glossary in `terminology.md`.
- **Context selection and compilation** belong in `context.md`; the Workbench document focuses on the inspectable interaction/prototype rather than redefining context engineering.
- **Memory and learning** belong in `memory.md`; session-learning research may remain under `research/`.
- **Workflow, orchestration and reasoning strategies** belong in `agent-workflows.md`.
- **Evidence, reliability and model migration evaluation** belong in `evaluation.md`.
- **Tool capability and selection semantics** belong in `tool-capability-model.md` and `tool-selection.md`; the visual landscape is a current snapshot derived from them, not a competing source of truth.
- **Enforceable architecture** is currently a research-backed extension of the software-engineering model; keep the detailed prior-art survey under `research/prior-art/enforceable-architecture.md` until implementation evidence justifies a dedicated canonical subsystem.
- **Individual tools/projects** belong primarily in the landscape/prior-art documents or research notes, not in the core architecture unless they establish a reusable capability boundary.
