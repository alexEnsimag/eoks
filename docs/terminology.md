# Terminology

The vocabulary is intentionally provisional. The definitions below are architectural working terms, not claims that the industry has standardized them.

| Term | Working meaning |
|---|---|
| Task | A bounded unit of work with an intended outcome and constraints. |
| Context | Information made available to a particular reasoning step. |
| Context engineering | Constructing task-specific model context from available evidence and knowledge. |
| Context compiler | The component that selects, ranks, transforms and assembles evidence for a reasoning step. |
| Knowledge | Durable information and relationships available to the system; not synonymous with a graph. |
| Knowledge representation | A view/IR optimized for a particular question, such as a structural graph, semantic index or timeline. |
| Canonical knowledge | Human-reviewable project intent, invariants, rationale and durable decisions; often practical as hierarchical `CLAUDE.md` + ADRs. |
| Evidence | An observation or artifact that supports or weakens a conclusion. |
| Evidence provider | A bounded component that returns evidence with provenance, scope/revision, confidence and cost characteristics. |
| Structural graph | A graph representation of code/system relationships such as calls, imports and dependencies. |
| Semantic index | A representation optimized for concept/meaning-based retrieval rather than exact structure. |
| Synthetic knowledge | Information not otherwise present in an authoritative source, such as rationale, lessons and cross-cutting invariants. |
| Memory | Information deliberately persisted for future use, including episodic and procedural memory. |
| Workflow | An explicit sequence/graph of actions, decisions and validation steps for achieving a task outcome. |
| Reasoning strategy | A reusable way of approaching a reasoning step, e.g. divergent exploration, adversarial review or hypothesis testing. |
| Model | A reasoning/generation capability with measurable characteristics. |
| Tool | A capability external to the model, often deterministic. |
| Agent | An execution loop combining reasoning, context and actions. |
| Policy | A constraint or rule governing system behavior. |
| Evaluation | Measurement of an intermediate or final outcome. |
| Confidence | A measure of evidence strength for a claim or result; should not be reduced to model self-reported confidence. |
| Control plane | The layer coordinating tasks, resources, policies and feedback. |
| Knowledge plane | The layer maintaining canonical knowledge and derived representations/evidence. |
| Context plane | The layer constructing task-specific model input. |
| Execution plane | The layer performing workflows, reasoning strategies, tool and agent actions. |

## Representation principle

> Knowledge is not a graph. A graph is one representation of knowledge.

A mature system may maintain several synchronized representations because each answers different questions. The representations should be selected according to the workload rather than forced into a single canonical structure.

## EOKS name

EOKS is retained as the project name. Its exact expansion is intentionally not treated as an architectural constraint. The project has evolved beyond a single interpretation of the acronym.
