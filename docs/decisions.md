# Architectural decisions

This is a lightweight decision record. Detailed ADRs can be split out as implementation begins.

## EOKS remains the project name

**Decision:** Keep `EOKS` as the project name without making its acronym expansion an architectural constraint.

**Reason:** The scope has expanded from context engineering to a broader AI control-plane concept. A narrow expansion would constrain future design.

## Context is a first-class resource

**Decision:** Model context separately from memory and knowledge.

**Reason:** Stored information and task-specific model input have different lifecycles and optimization goals.

## Knowledge is multi-representational

**Decision:** Do not make a graph, vector store or any other single representation synonymous with knowledge.

**Reason:** Structural graphs, semantic indexes, canonical documents, timelines, runtime observations and other representations answer different questions. EOKS should select representations as evidence providers rather than force everything into one canonical structure.

## Evidence providers are bounded capabilities

**Decision:** Treat repository graphs, static analyzers, tests, runtime observations, semantic retrieval and similar systems as evidence providers that can be selected according to task requirements.

**Reason:** Different questions require different assurance. The control plane should prefer the cheapest reliable provider that can answer the question instead of indiscriminately running every analyzer.

## Resource vocabulary is not the runtime ontology

**Decision:** Keep `Resource`, `Asset`, `Provider`, `Representation` and `Loadout` as vocabulary for describing capabilities, reusable state and eligibility boundaries. Do not promote them to EOKS runtime primitives unless implementation evidence demonstrates that they need stable lifecycle or API semantics.

**Reason:** These terms are useful for discussing heterogeneous systems, but a large resource ontology would add complexity before the first end-to-end control loop establishes which entities must actually persist and interact.

## Existing agents remain replaceable execution resources

**Decision:** Prefer integrating with existing coding-agent runtimes through adapters, launchers, hooks, MCP or similar boundaries before building a replacement agent runtime.

**Reason:** The EOKS research question is coordination and feedback, not whether EOKS can reproduce an agent's internal reasoning loop. GrapeRoot is useful prior art for this sidecar/integration pattern.

## Evaluation belongs in the architecture

**Decision:** Treat evaluation and feedback as a runtime subsystem.

**Reason:** Model routing, context construction and orchestration cannot be optimized reliably without outcome evidence.

## Reliability is not model confidence

**Decision:** Keep observability, model uncertainty, evidence strength, context quality and outcome quality as distinguishable signals. Calibrate any derived reliability signal against actual workload outcomes before using it for automatic control.

**Reason:** Traces and self-reported confidence are useful evidence, but neither is ground truth. A control policy needs workload- and decision-specific evidence.

## Learning is a lifecycle, not another agent runtime

**Decision:** Treat learning/reflection as a cross-cutting process that consumes traces and outcomes and proposes versioned knowledge, skills or policies. Do not make a separate autonomous learning plane a required architectural component yet.

**Reason:** Learning currently describes a transformation of evidence into improved resources; real traces should determine whether it needs independent runtime infrastructure.

## Minimal semantic model first

**Decision:** Start with the seven working runtime primitives `Task`, `Context`, `Run`, `Decision`, `Policy`, `Evaluation` and `Outcome`. Treat models, tools, agents, knowledge stores and analyzers as resources/providers until implementation evidence justifies promoting more concepts to first-class primitives.

**Reason:** A large ontology would encode assumptions before real run traces show which objects and relationships are actually necessary.

## Hybrid deterministic/probabilistic systems

**Decision:** Prefer deterministic tools for deterministic questions and use LLM reasoning where semantic synthesis is required.

**Reason:** Software-engineering tasks expose many facts that static analysis, tests and language tooling can establish more reliably than a model.

## Local-first experimentation

**Decision:** Prove EOKS abstractions locally before building hosted infrastructure.

**Reason:** The architectural question is about contracts and control loops, not whether a service is deployed. A useful implementation may initially be only files plus a CLI/runtime.

## Inspectability over magic

**Decision:** Decisions about context, models, tools and memory should be inspectable.

**Reason:** A control plane that cannot explain which evidence it selected and why is difficult to debug, evaluate and trust.

## Controlled knowledge promotion

**Decision:** Observations and generated candidates must not silently become canonical project knowledge or policy. Promotion should retain provenance, scope/freshness and validation evidence and remain reversible where practical.

**Reason:** Automatic extraction is valuable, but stale or incorrect generated knowledge can contaminate every future context that retrieves it.
