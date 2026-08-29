# EOKS documentation

`docs/` is the **current architectural and design layer** of EOKS. These documents describe the working model that the project currently proposes. They are more coherent and prescriptive than the research corpus, but the architecture remains provisional unless a document is explicitly recorded as a decision.

## Current architecture

- [`architecture.md`](architecture.md) — canonical architecture, control loop, boundaries and seven provisional runtime primitives.
- [`resource-model.md`](resource-model.md) — resources, assets, providers, representations and loadouts.
- [`context.md`](context.md) — context engineering and compilation.
- [`context-workbench.md`](context-workbench.md) — inspectable context blocks, budgets and human control.
- [`knowledge-base.md`](knowledge-base.md) — durable project knowledge and lifecycle.
- [`knowledge-representations.md`](knowledge-representations.md) — multiple representations of engineering reality.
- [`memory.md`](memory.md) — memory and behavioral learning.
- [`agent-roles.md`](agent-roles.md) — workflow responsibilities.
- [`agent-workflows.md`](agent-workflows.md) — workflows and reasoning strategies.
- [`evaluation.md`](evaluation.md) — evaluation, reliability evidence and calibration.
- [`tool-capability-model.md`](tool-capability-model.md) — provider capabilities, evidence requirements and selection semantics.
- [`continuous-knowledge-maintenance.md`](continuous-knowledge-maintenance.md) — incremental maintenance, promotion and invalidation.
- [`deterministic-execution.md`](deterministic-execution.md) — deterministic execution as one modality.
- [`software-engineering.md`](software-engineering.md) — software-engineering workloads and agent practices.
- [`software-analysis.md`](software-analysis.md) — invariants, dataflow and analyzer escalation.

The architecture page is deliberately short. Detailed behavior belongs in the document that owns the concept rather than being duplicated across architecture and specialist pages.

## Governance and research

- [`decisions.md`](decisions.md) — current architectural decisions.
- [`research-agenda.md`](research-agenda.md) — experiments intended to validate or falsify the architecture.
- [`open-questions.md`](open-questions.md) — unresolved questions.
- [`prior-art.md`](prior-art.md) — consolidated prior-art landscape.
- [`terminology.md`](terminology.md) — shared working vocabulary.
- [`vision.md`](vision.md) — long-term direction.
- [`history.md`](history.md) — evolution of the project and its hypotheses.

## Document ownership rule

Avoid maintaining the same architectural claim independently in several documents. When a concept has a canonical home, other documents should summarize it and link there. Research notes may repeat a concept when necessary to explain how it was discovered, but should label it as a hypothesis or observation rather than creating a second source of truth.

In particular:

- **Architecture, control loop, planes and runtime primitives** belong in `architecture.md`.
- **Resource/Asset/Provider/Representation/Loadout definitions** belong in `resource-model.md`; the glossary in `terminology.md` stays compact.
- **Context selection and compilation** belong in `context.md`; the Workbench focuses on inspectable interaction/prototyping rather than redefining context engineering.
- **Knowledge and memory lifecycle** belong in `knowledge-base.md` and `memory.md`.
- **Workflow, roles and reasoning strategies** belong in `agent-roles.md` and `agent-workflows.md`. Scheduler, router and orchestrator are implementation terms under the conductor/control responsibility, not competing architecture documents.
- **Evaluation, reliability and calibration** belong in `evaluation.md`.
- **Provider capabilities, evidence requirements and selection semantics** belong together in `tool-capability-model.md`; there is no separate normative tool-selection model.
- **Incremental maintenance, promotion and invalidation** belong in `continuous-knowledge-maintenance.md`; deterministic execution is a modality/deep dive, not a separate control architecture.
- **Individual tools/projects** belong primarily in the landscape/prior-art documents or research notes, not in the core architecture unless they establish a reusable capability boundary.

When a new concept appears, first ask whether it is already a role, resource, provider, representation, context artifact, workflow construct, policy or one of the seven runtime primitives. Prefer moving/merging material into an existing owner over adding another document.
