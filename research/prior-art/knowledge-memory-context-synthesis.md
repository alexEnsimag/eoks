# Knowledge, experience and context as EOKS synthesis

## Status

Research-layer synthesis. This note sharpens existing EOKS concepts; it does not introduce Knowledge Graph, Memory Graph or Context Graph as runtime primitives.

## Why this synthesis matters

Recent practitioner writing on knowledge graphs, memory graphs and context graphs describes three related but distinct concerns:

- durable, governed meaning and relationships;
- retained experience from previous decisions and outcomes;
- task-time context assembled for a particular decision.

The useful EOKS result is not to reproduce those labels as three subsystems. EOKS already has stronger boundaries: knowledge/evidence and their representations, memory and learning, working sets and context compilation, execution, evaluation and reconciliation.

The synthesis is:

```text
                    KNOWLEDGE
              governed / durable meaning
                         |
                         +----------------+
                         |                |
                    EXPERIENCE        LIVE STATE
                  what happened       what is true now
                         |                |
                         +--------+-------+
                                  |
                                  v
                         CONTEXT COMPILATION
                                  |
                                  v
                           CONTEXT ARTIFACT
                                  |
                                  v
                                RUN
                                  |
                               OUTCOME
                                  |
                              EVALUATION
                                  |
                                  v
                             EXPERIENCE
                                  |
                         reflection / synthesis
                                  |
                     candidate knowledge / capability
                                  |
                         validation / promotion
                                  |
                                  +-------> future context
```

The important architectural boundary is therefore **projection, not storage**: context is a task-specific projection of eligible knowledge, evidence, experience and state under policy and resource constraints.

## Four useful distinctions

### Knowledge

Knowledge is durable project/system information and representations that EOKS can preserve, derive, validate and maintain. It is not synonymous with a graph, database, document or memory store.

A relationship is useful when it contributes evidence or navigation for a workload, but derived relationships should retain provenance, freshness and a path back to authoritative evidence.

### Experience

Experience records what happened when the system observed, decided, acted and evaluated. It can include successful patterns, failed attempts, rejected alternatives, corrections, limitations and outcome evidence.

Experience should not automatically become canonical knowledge. A system that turns every observation or model-generated claim into durable truth risks contaminating the very resources used for future context.

A useful lifecycle is:

```text
observation
    -> candidate claim / diagnosis
    -> evidence
    -> decision / intervention
    -> outcome
    -> evaluation
    -> retained experience
    -> reflection / synthesis
    -> candidate knowledge or capability
    -> validation / promotion
```

This complements the existing EOKS memory lifecycle rather than replacing it.

### Context

Context is the task-specific information compiled for a reasoning step. It is a materialization of the workload's current working set, not a permanent knowledge layer.

A context artifact should ideally make visible:

- which resources/evidence were selected;
- their versions and freshness;
- the representations used;
- provenance and authority;
- policy and eligibility constraints;
- retrieval/transformation parameters;
- relevant exclusions or misses;
- the resulting representation delivered to execution.

This turns context into something that can be inspected, compared and evaluated rather than an opaque prompt string.

### Evaluation

Evaluation closes the loop. It determines whether the context and resulting action were sufficient for the workload, and whether an intervention improved the end-to-end outcome rather than merely a local proxy such as token count or retrieval score.

This preserves the distinction already established in EOKS between the empirical improvement loop and the knowledge lifecycle:

```text
empirical improvement:
execute -> trace -> evaluate -> diagnose/reflect -> refine -> execute

knowledge lifecycle:
observe -> extract -> validate -> materialize -> relate -> reuse
          -> invalidate/update/recompute
```

The two loops interact, but should not be collapsed into one undifferentiated "memory" loop.

## Decision provenance

The most important extension to context compilation is to preserve not only **what** context was selected but, where practical, **why**.

```text
Task + Policy
     |
eligible resources
     |
selection / acquisition decisions
     |
context artifact
     |
agent action
     |
outcome + evaluation
```

Useful provenance questions include:

1. What candidate evidence was available?
2. What was selected and excluded?
3. Which policy or workload requirement caused the selection?
4. Which representation was used and why?
5. What evidence justified a claim or state transition?
6. Which outcome supported or contradicted the choice?

This is especially important for reproducibility, evaluation and debugging of context interventions.

## Negative experience is valuable evidence

A durable learning system should retain useful negative experience, not only successful knowledge.

Examples include:

- an approach tried and rejected for a documented reason;
- a representation shown to be stale or misleading;
- a tool/provider that failed to satisfy an evidence requirement;
- a false relationship between engineering artifacts;
- a failed intervention whose downstream recovery work outweighed its local savings.

Negative experience should remain appropriately scoped and evidence-backed. It is not automatically a permanent prohibition. Validity, workload applicability and freshness still matter.

This gives reflection a concrete destination: reflection can produce diagnoses and candidate experience; synthesis can consolidate repeated or well-supported experience; validation controls promotion into durable knowledge or deterministic capability.

## Graphs: representation, not ontology

Knowledge Graph, Memory Graph and Context Graph are useful descriptive terms, but EOKS should not make them mandatory architectural objects.

A graph may be the right representation when a workload benefits from:

- multi-hop relationship traversal;
- identity resolution;
- temporal relationships;
- explicit provenance;
- relationship-aware impact analysis;
- reuse of expensive structural analysis.

For other workloads, documents, indexes, timelines, tables, code-property graphs, embeddings, event logs or other representations may be more appropriate.

The EOKS design test remains:

> Does the representation improve the validated workload outcome enough to justify its construction, maintenance and context-delivery cost?

## Context artifact and reproducibility

A useful research hypothesis is that a context artifact can become a first-class **evaluatable intermediate artifact** without becoming a new EOKS runtime primitive.

At minimum, experiments should consider recording:

- workload/task identity;
- policy/loadout version;
- selected resource and representation identities;
- source versions and freshness;
- provenance/evidence references;
- acquisition and transformation operations;
- context budget/materialization details;
- exclusions and misses;
- execution/run identity;
- downstream outcome and evaluation.

This enables context diffs, replay, ablation and failure analysis. It also makes it possible to distinguish a poor retrieval decision from a poor reasoning decision.

## Research questions

1. Does preserving selection rationale improve debugging and context optimization?
2. How much predictive value does negative experience provide for future coding workloads?
3. When does a relationship-aware representation outperform simpler retrieval or structural indexing?
4. What context-artifact fields are necessary for reproducible evaluation without creating excessive logging cost?
5. When should experience be promoted into durable knowledge or deterministic capability?
6. How should invalidation propagate from authoritative artifacts through derived relationships, context artifacts and learned experience?
7. Can context selection decisions themselves become reusable evidence for later workloads without overfitting to historical tasks?

## EOKS disposition

**Supported synthesis; no new runtime primitive.**

The useful architectural conclusion is to sharpen existing boundaries:

```text
knowledge = durable accepted meaning/evidence
experience = retained evidence about what happened
context = task-specific projection of eligible resources
execution = action under the selected context
outcome = what actually happened
evaluation = evidence about whether the workload succeeded
```

Graphs remain derived representations. Memory remains a lifecycle over experience and learning. Context remains compiled working-set state. The conductor/reconciliation loop remains responsible for choosing what to acquire, compile, execute, evaluate, reuse, invalidate or promote.

## Source

Sergey Vasiliev, *Knowledge Graph, Memory Graph and Context Graph* (2026), practitioner prior art. The article is useful as a conceptual synthesis, not as controlled evidence for universal graph or memory benefits.
