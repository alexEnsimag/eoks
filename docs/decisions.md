# Architectural decisions

This is a lightweight decision record. Detailed ADRs can be split out as implementation begins.

## EOKS remains the project name

**Decision:** Keep `EOKS` as the project name without making its acronym expansion an architectural constraint.

**Reason:** The scope has expanded from context engineering to a broader AI control-plane concept. A narrow expansion would constrain future design.

## Context is a first-class resource

**Decision:** Model context separately from memory and knowledge.

**Reason:** Stored information and task-specific model input have different lifecycles and optimization goals.

## Evaluation belongs in the architecture

**Decision:** Treat evaluation and feedback as a runtime subsystem.

**Reason:** Model routing, context construction and orchestration cannot be optimized reliably without outcome evidence.

## Hybrid deterministic/probabilistic systems

**Decision:** Prefer deterministic tools for deterministic questions and use LLM reasoning where semantic synthesis is required.

**Reason:** Software-engineering tasks expose many facts that static analysis, tests and language tooling can establish more reliably than a model.

## Local-first experimentation

**Decision:** Prove EOKS abstractions locally before building hosted infrastructure.

**Reason:** The architectural question is about contracts and control loops, not whether a service is deployed. A useful implementation may initially be only files plus a CLI/runtime.

## Inspectability over magic

**Decision:** Decisions about context, models, tools and memory should be inspectable.

**Reason:** A control plane that cannot explain which evidence it selected and why is difficult to debug, evaluate and trust.
