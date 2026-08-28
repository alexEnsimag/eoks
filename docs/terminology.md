# Terminology

The vocabulary is intentionally provisional.

| Term | Working meaning |
|---|---|
| Task | A bounded unit of work with an intended outcome and constraints. |
| Context | Information made available to a particular reasoning step. |
| Context quality | How useful the supplied information is for the task, considering relevance, correctness, freshness, completeness, noise and cost. |
| Memory | Information deliberately persisted for future use. |
| Knowledge | Durable information and relationships available to the system. |
| Evidence | An observation or artifact that supports or weakens a conclusion. |
| Model | A reasoning/generation capability with measurable characteristics. |
| Tool | A capability external to the model, often deterministic. |
| Agent | An execution loop combining reasoning, context and actions. |
| Policy | A constraint or rule governing system behavior. |
| Evaluation | Measurement of an intermediate or final outcome. |
| Control plane | The layer coordinating tasks, resources, policies and feedback. |
| Context plane | The layer constructing task-specific model input. |
| Memory plane | The layer persisting and retrieving durable state/knowledge. |
| Execution plane | The layer performing tool and agent actions. |

## EOKS name

EOKS is retained as the project name. Its exact expansion is intentionally not treated as an architectural constraint. The project has evolved beyond a single interpretation of the acronym.
