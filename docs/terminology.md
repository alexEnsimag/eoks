# Terminology

The vocabulary is intentionally provisional. The definitions below are architectural working terms, not claims that the industry has standardized them.

| Term | Working meaning |
|---|---|
| Task | A bounded unit of work with an intended outcome, constraints and required assurance. |
| Context | Information intentionally made available to a particular reasoning step. |
| Context engineering | The broader discipline of constructing task-specific model context. |
| Context compilation | The concrete operation that selects, ranks, transforms, orders and assembles information for a reasoning step. |
| Context compiler | The component performing context compilation. |
| Context block | An inspectable unit of compiled context carrying content plus useful metadata such as provenance, revision, relevance, freshness and cost. |
| Knowledge | Durable information and relationships available to the system; not synonymous with a graph. |
| Knowledge representation | A view/IR optimized for a particular question, such as a structural graph, semantic index or timeline. |
| Canonical knowledge | Human-reviewable project intent, invariants, rationale and durable decisions; often practical as hierarchical `CLAUDE.md` + ADRs, or a portable format such as OKF. |
| Evidence | An observation or artifact that supports or weakens a conclusion. |
| Evidence provider | A bounded component that answers a question and returns evidence with provenance, scope/revision, validation/confidence and cost characteristics. |
| Structural graph | A graph representation of code/system relationships such as calls, imports and dependencies. |
| Semantic index | A representation optimized for concept/meaning-based retrieval rather than exact structure. |
| Synthetic knowledge | Information not otherwise present in an authoritative source, such as rationale, lessons and cross-cutting invariants. |
| Memory | Information deliberately persisted for future use, including episodic and procedural memory. |
| Skill | A reusable, governed procedure promoted from evidence and successful execution rather than merely repeated text. |
| Workflow | An explicit sequence/graph of actions, decisions and validation steps for achieving a task outcome. |
| Reasoning strategy | A reusable way of approaching a reasoning step, e.g. divergent exploration, adversarial review or hypothesis testing. |
| Model | A reasoning/generation capability with measurable characteristics. |
| Tool | A capability external to the model, often deterministic. |
| Resource | A capability or reusable thing EOKS can select, invoke or make available, including models, tools, agents, knowledge sources, providers and durable assets. |
| Asset | A generic lifecycle/governance abstraction for a reusable resource. Asset is not a semantic knowledge category; memory, Skills, documents, decisions and derived evidence can all be assets. |
| Representation | A form optimized for a particular query or operation, such as a graph, index, document, timeline or runtime model. A representation is not automatically canonical knowledge. |
| Loadout | The workload-scoped set of assets an agent/task is allowed and expected to use. Loadout eligibility is distinct from final context selection. |
| Agent | An execution loop combining reasoning, context and actions. |
| Run | One attempt to execute a task or subtask under a particular context, policy and resource configuration. |
| Decision | A control-plane choice about what happens next, such as retrieve, verify, retry, branch, stop or escalate. |
| Policy | A constraint or requirement governing system behavior and decisions. |
| Evaluation | Measurement of intermediate or final quality, evidence strength, assurance or task success. |
| Outcome | What actually happened, including artifacts, verification results and delayed results when they become known. |
| Confidence | A signal about uncertainty or evidence strength; should not be reduced to model self-reported confidence. |
| Assurance | The level of evidence/verification required for a task or decision before the system may proceed or stop. |
| Control plane | The layer coordinating tasks, resources, policies, decisions and feedback. |
| Knowledge plane | The layer maintaining canonical knowledge and derived representations/evidence. |
| Context plane | The layer constructing task-specific model input. |
| Execution plane | The layer performing workflows, runs, reasoning strategies, tool and agent actions. |

For the detailed relationship between Resource, Asset, Provider, Representation, Loadout and Context, see [Resource model](resource-model.md). That document is the canonical home for these boundaries; this table is the compact glossary.

## Representation principle

> **Knowledge is not a graph. A graph is one representation of knowledge.**

A mature system may maintain several synchronized representations because each answers different questions. The representations should be selected according to the workload rather than forced into a single canonical structure.

Likewise:

> **Context is not knowledge. Context is a task-specific compilation of knowledge and evidence.**

YAML, JSON or another serialization can make these structures visible, but no particular serialization is itself the EOKS semantic model.

## Resource, asset and provider

These terms intentionally form different levels:

```text
Resource
  └── broad capability/reusable thing EOKS can select or invoke
       └── Asset
            └── reusable resource governed across workloads

Provider
  └── mechanism that retrieves or derives evidence

Representation
  └── form optimized for a particular question/operation

Loadout
  └── workload-scoped eligibility/availability boundary

Context
  └── task-specific compiled projection
```

A provider can produce evidence without producing a durable asset. A representation can be derived from canonical sources without itself being canonical knowledge. A loadout can contain assets without all of them entering the final context.

## Trust and confidence

Keep these concepts separate:

```text
model confidence
    != evidence strength
    != context quality
    != outcome quality
```

EOKS should prefer multidimensional evidence and assurance signals over one universal confidence score. Policies can determine which signals are necessary for a given workload.

## EOKS name

EOKS is retained as the project name. Its exact expansion is intentionally not treated as an architectural constraint. The project has evolved beyond a single interpretation of the acronym.
