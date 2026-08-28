# Recurring observations and unresolved questions

This note intentionally preserves questions that appeared repeatedly instead of forcing premature answers.

## Observations

### Bigger context windows do not remove context engineering

A larger window changes the capacity constraint, not the information-selection problem. The system still needs to decide what is relevant, current, trustworthy and worth the attention/token budget.

### Structured context keeps appearing for a reason

YAML, Markdown, JSON, graphs and other structures appeared repeatedly. The common property is explicit relationships and metadata. The exact serialization should remain replaceable.

### Context quality may deserve first-class status

The idea of “context quality” kept reappearing as a possible system metric. It may be more useful to model it as several measurable dimensions than to invent one magic score.

### Graphs and blocks serve different purposes

Graphs are powerful for relationships and discovery. Blocks are easier for human composition and budgeting. A system may need both views over the same underlying knowledge.

### Deterministic analysis is complementary to LLM reasoning

Security/dataflow and code-structure examples showed that specialized tools can establish facts that a language model should consume as evidence rather than reproduce from inference.

### Models are not interchangeable

Model upgrades can change behavior enough that “latest model” is not a sufficient selection policy. Representative workload evaluation matters.

### Observability can become a control signal

Tracing is useful, but the more ambitious possibility is to turn observations into policy inputs: retry, verify, change model, change context, escalate or learn.

## Unresolved questions

### What exactly is the EOKS resource model?

Are the first-class resources:

- tasks;
- contexts;
- memories;
- evidence;
- models;
- tools;
- agents;
- execution budgets;
- policies;
- outcomes?

Or should several of these be implementation details?

### Is EOKS a runtime, protocol, platform or methodology?

We have used all four descriptions at different points. The architecture should not settle this by terminology alone. A minimal implementation can reveal which boundaries need a real runtime.

### Does the control plane need to be centralized?

A Kubernetes-like controller is one option. A distributed set of services or libraries could provide the same behavior. Centralization should be justified by coordination requirements.

### How much autonomy is useful?

More orchestration can increase reliability, but it can also introduce latency, cost and failure modes. The scheduler must have a measurable reason to intervene.

### Can context quality be measured generically?

A metric that only works for coding tasks is still valuable, but it should not be mistaken for a universal measure of context quality.

### What is the right unit of evaluation?

Tokens, calls, tasks, workflows and projects expose different tradeoffs. EOKS likely needs hierarchical evaluation rather than one benchmark score.

### Where does memory end and project state begin?

For engineering systems, repository state, tickets, decisions and agent memories can overlap. The distinction may be policy-based rather than a hard storage boundary.

### What should EOKS not do?

The project should actively avoid becoming:

- another generic agent framework;
- a prompt-template library;
- a vector database wrapper;
- an observability dashboard with no control loop;
- a giant knowledge graph whose usefulness is never demonstrated.

The burden of proof is on the abstraction.