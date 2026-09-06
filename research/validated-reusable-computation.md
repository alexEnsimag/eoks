# Validated and reusable computation

This note synthesizes several bodies of work that appear at first to be separate: reflection and self-refinement, formal program refinement, process and outcome evaluation, provenance, incremental computation, dependency tracking and reusable computation.

The goal is not to introduce these as EOKS primitives. The goal is to ask whether they illuminate a common systems problem:

> **How can an AI system preserve, validate, explain and selectively reuse the results of computation as the underlying state changes?**

This is a research synthesis, not a normative architecture decision.

## Executive synthesis

The practitioner "reflection harness" pattern from the SDD literature is one concrete implementation of a broader pattern:

```text
produce representation
        ↓
 evaluate / verify
        ↓
 revise or accept
        ↓
 materialize useful state
        ↓
 reuse when dependencies remain valid
        ↓
 detect change
        ↓
 invalidate / recompute affected work
```

Several established traditions illuminate different parts of this loop:

| Tradition | Main contribution | EOKS-relevant question |
| --- | --- | --- |
| Self-refinement / reflection | iterative critique and revision | When does another evaluation/revision step improve the result enough to justify its cost? |
| Formal refinement | correctness-preserving transformation between representations | What does it mean for a new representation to preserve constraints of an earlier one? |
| Process supervision | evaluation of intermediate reasoning/work | When is intermediate evaluation better than outcome-only evaluation? |
| External/tool verification | evidence from execution or specialized tools | Which claims should be established by evidence outside the generating model? |
| Provenance | lineage of entities, activities and derivations | Can a result's origin and supporting evidence be reconstructed? |
| Incremental computation | dependency-aware partial recomputation | Which previously computed results remain valid after change? |
| Build/cache systems | practical reuse under dependency and reproducibility constraints | When can computation be safely reused rather than rerun? |

The synthesis suggests that **reflection is a strategy, while validated reusable computation is the broader systems problem**.

## 1. Representation is not computation

A durable artifact, a derived representation and a cached computation should not be conflated.

```text
representation
  = a durable description of state/knowledge

computation
  = a process transforming inputs into outputs

evidence
  = information used to assess a claim or state

provenance
  = information describing how a result was produced and derived
```

For example:

```text
repository → architecture graph
requirements → design representation
source + analyzer → dataflow result
prompt + model + context → generated proposal
```

These outputs may all be persistent, but persistence alone does not make them interchangeable.

A Markdown proposal may be valuable durable knowledge. A generated graph may be a reusable representation. A verified analysis may be reusable evidence. A completed model/tool computation may potentially be reusable computation.

The distinction matters because different reuse and invalidation rules apply to each.

## 2. Reflection is one evaluation strategy

Self-Refine demonstrates iterative generation, feedback and refinement using the same model. Reflexion adds environmental feedback and persistent verbal experience. CRITIC demonstrates critique grounded in external tools.

These approaches establish a useful spectrum:

```text
model self-judgment
       ↓
independent/model critique
       ↓
deterministic or executable verification
       ↓
human or external authoritative evidence
```

The stronger the external grounding, the less the system depends on the generating model being a reliable judge of its own output. But stronger verification may cost more time, compute or engineering effort.

Therefore EOKS should not treat "reflection" as a correctness mechanism by itself. It is an **evaluation/refinement strategy whose value must be measured for the workload**.

### Important evidence boundary

Practitioner claims that a reflection workflow makes a weaker model perform roughly like a stronger model are useful hypotheses, not model benchmarks. Improvements can arise from extra tokens, extra attempts, workflow constraints, evaluator differences or selection effects.

Controlled comparisons should separate:

- model capability;
- additional computation/tokens;
- evaluator quality;
- workflow structure;
- external verification;
- number of attempts.

## 3. Refinement is stronger than freezing

Classical refinement calculus provides a more formal interpretation of staged development than the SDD notion of freezing documents.

A freeze establishes structural immutability:

```text
representation A
      ↓
     freeze
      ↓
representation A'
```

It does not establish that a later representation is semantically correct relative to the earlier one.

Formal refinement instead asks for a relation between representations that preserves specified properties:

```text
abstract specification
        ↓ refinement
more concrete representation
        ↓ refinement
implementation
```

This suggests a useful EOKS distinction:

> **An accepted representation is not necessarily a refined representation.**

Where a correctness-preserving relation can be established, it is stronger evidence than immutability. Where no such relation is available, EOKS should not imply one merely because an artifact was reviewed or frozen.

LLM-assisted program-refinement research is relevant prior art because it explores combining formal refinement with LLM-generated implementations and verification. This is a concrete example of probabilistic generation operating inside a stronger formal assurance boundary.

## 4. Process evaluation versus outcome evaluation

Research on verifiers and process supervision shows that intermediate evaluation can improve performance on some workloads. But theoretical work also cautions against assuming that process supervision is universally superior to outcome supervision.

This is important for EOKS.

The system should be able to choose among:

```text
compute → outcome → evaluate
```

and

```text
compute A → evaluate A
        ↓
compute B → evaluate B
        ↓
outcome → evaluate
```

The second can prevent error propagation, but it also introduces additional cost and potentially noisy intermediate judgments.

Therefore:

> **Evaluation boundaries should be workload-specific control decisions, not universal stages.**

This connects directly to EOKS's existing control-loop idea that evaluation produces evidence used to choose whether to continue, verify, repair, replan, escalate or stop.

## 5. Provenance turns artifacts into lineage

W3C PROV provides a useful formal vocabulary for describing entities, activities, agents, usage, generation and derivation.

Applied to an AI workload:

```text
source state
     │
     │ used by
     ▼
computation / activity
     │
     │ generates
     ▼
derived representation
     │
     │ evaluated using
     ▼
validation evidence
```

A provenance-aware system can retain questions such as:

- Which inputs produced this representation?
- Which computation produced it?
- Which model, tool or agent was involved?
- Which evidence supported acceptance?
- Which source version was used?
- What downstream representations depend on it?
- What changes could make it stale?

This is stronger than storing an artifact's timestamp or filename.

Agent-provenance research extends this direction to prompts, responses, decisions, observations and context. This is particularly relevant to EOKS because the useful unit of lineage may cross the boundaries between knowledge, context, tools and model execution.

## 6. Reuse is a dependency problem

Incremental and self-adjusting computation provide a deeper model for the "computed cache" hypothesis.

A naive cache is:

```text
inputs → result
```

A dependency-aware cache is closer to:

```text
inputs
  ↓
computation graph
  ↓
result + dependencies
```

When an input changes:

```text
change
  ↓
identify affected dependencies
  ↓
invalidate affected results
  ↓
reuse unaffected results
  ↓
recompute affected results
```

This is the formal territory of self-adjusting computation and the practical territory of incremental build systems.

It is much closer to the earlier EOKS hypothesis of saving a computed representation or entire computation than simply persisting an SDD document.

## 7. Reproducibility and cache validity

Build systems add an important practical constraint: reuse must not silently change correctness.

Systems such as Bazel use input/action identity, dependency tracking and output validity to decide whether work can be skipped. Reproducible-build practices similarly emphasize that the same source, environment and instructions should produce verifiable artifacts.

The EOKS analogue is not necessarily byte-for-byte reproducibility. AI workloads can be probabilistic. Instead, the system may need to record enough execution identity and dependency information to determine whether a previous result remains applicable.

Potential dependency dimensions include:

```text
source state
context contents
context compiler/version
model/version
model configuration
prompt/instructions
tools and tool versions
policies
external environment
retrieved evidence
```

Not every workload needs all of these. The important research question is which dependencies are **semantically relevant to validity**.

## 8. Invalidation may be as important as caching

Once an intermediate result becomes reusable, stale state becomes a first-class failure mode.

For example:

```text
repository
   ↓
architecture representation
   ↓
dataflow analysis
   ↓
security conclusion
```

If the repository changes, EOKS needs to determine whether:

- the architecture representation is still valid;
- the dataflow result is still valid;
- the security conclusion still follows;
- only a subgraph needs recomputation;
- previous evidence remains applicable.

This creates a useful lifecycle:

```text
produced
   ↓
evaluated
   ↓
accepted
   ↓
reusable
   ↓
stale / invalidated
   ↓
recomputed or retired
```

A system that aggressively caches without strong invalidation can be less reliable than one that recomputes everything.

## 9. A possible EOKS synthesis

The concepts can be combined without making any of them EOKS primitives:

```text
                         WORKLOAD
                            │
                            ▼
                       computation
                            │
                            ▼
                  derived representation
                            │
              ┌─────────────┼─────────────┐
              │             │             │
          evaluation     provenance    dependencies
              │             │             │
              └─────────────┼─────────────┘
                            ▼
                       accepted state
                            │
                    materialize / reuse
                            │
                            ▼
                    subsequent execution
                            │
                      observe change
                            │
                            ▼
                  invalidate / recompute
                            │
                            ▼
                        reconcile
```

The EOKS control plane surrounds this lifecycle:

```text
                    ┌─────────────────────────┐
                    │      CONTROL POLICY     │
                    │                         │
                    │ choose / evaluate /     │
                    │ accept / retry / branch │
                    │ escalate / reuse        │
                    └────────────┬────────────┘
                                 │
                                 ▼
       authoritative state → computation → representation
                                  │               │
                                  │               ├─ provenance
                                  │               ├─ dependencies
                                  │               └─ evidence
                                  │
                                  ▼
                              evaluation
                                  │
                           ┌──────┴──────┐
                           │             │
                        accepted       revise
                           │             │
                           ▼             └────→ computation
                        materialize
                           │
                     ┌─────┴─────┐
                     │           │
                   reuse      invalidation
                     │           │
                     └─────┬─────┘
                           ▼
                         control
```

This is deliberately a synthesis, not a proposed API or object model.

## 10. Relationship to existing EOKS concepts

This synthesis extends rather than replaces the current EOKS model.

### Workload

The workload remains the unit of coordination. The new question is what useful state and computation a workload can leave behind and safely reuse.

### Context

A context package is a representation compiled from persistent resources. Its provenance and dependencies can help determine whether it remains comparable or reusable across executions.

### Evidence

Evidence is not just a final score. It can establish properties of intermediate representations and determine whether a state is acceptable for a particular control decision.

### Control loop

The control loop can decide whether another computation is justified, whether verification is required, whether an existing result can be reused, or whether stale state must be invalidated.

### Memory and learning

Memory can preserve useful representations and experience, but promotion should depend on validation, scope, provenance and freshness. A remembered result is not automatically authoritative.

### Model/tool routing

Different verification and computation strategies can be selected according to risk, cost and expected value. A deterministic analyzer may be preferable to another model call when it can establish the required property.

## 11. What should *not* be inferred

This synthesis does **not** imply that EOKS should:

- make reflection a first-class primitive;
- require formal verification for every step;
- persist every intermediate model state;
- cache every model computation;
- require a graph database;
- require W3C PROV as its storage format;
- treat every artifact as executable state;
- assume intermediate evaluation is always better;
- assume cached computation is cheaper than recomputation;
- assume provenance makes a result correct.

These are implementation and empirical questions.

## 12. Research questions

The most useful next experiments are likely to be:

### Materialization value

When does the expected future reuse of an intermediate representation exceed the cost of producing and maintaining it?

### Validation value

What types of evidence provide enough additional reliability to justify an evaluation boundary?

### Reuse validity

Which input and execution dimensions must match before a previous result can be reused?

### Invalidation

Can a dependency graph identify stale results with acceptable false-negative risk?

### Partial recomputation

How much work can be reused when only part of the authoritative state changes?

### Computation reuse

Can meaningful portions of an AI workflow be reused at the computation level rather than merely at the document/context level?

### Control economics

Can EOKS estimate the expected value of another computation or verification step against token, compute, latency and risk costs?

### Evaluation placement

For a given workload, when does intermediate evaluation outperform outcome-only evaluation after accounting for its additional cost?

## 13. Candidate experimental vocabulary

For experiments, rather than architecture, it may be useful to measure:

```text
representation reuse rate
computation reuse rate
invalidation precision / recall
recomputation avoided
validation cost
validation effectiveness
stale-result rate
provenance completeness
outcome quality
cost per accepted outcome
latency per accepted outcome
```

These are candidate metrics, not established EOKS metrics. They should be validated against concrete workloads before entering the canonical model.

## Sources and prior art

- Madaan et al., **Self-Refine: Iterative Refinement with Self-Feedback**, NeurIPS 2023 — https://arxiv.org/abs/2303.17651
- Shinn et al., **Reflexion: Language Agents with Verbal Reinforcement Learning**, NeurIPS 2023 — https://arxiv.org/abs/2303.11366
- Gou et al., **CRITIC: Large Language Models Can Self-Correct with Tool-Interactive Critiquing**, ICLR 2024 — https://arxiv.org/abs/2305.11738
- Findings of EMNLP 2024, **Multi-step Problem Solving Through a Verifier: An Empirical Analysis on Model-induced Process Supervision** — https://aclanthology.org/2024.findings-emnlp.429/
- Jia et al., **Do We Need to Verify Step by Step? Rethinking Process Supervision from a Theoretical Perspective**, ICML 2025 — https://proceedings.mlr.press/v267/jia25f.html
- Morgan, **The Refinement Calculus**, South African Computer Journal — https://ir.unisa.ac.za/bitstream/handle/10500/24169/1995_SACJ_13_Morgan.pdf
- **Towards Large Language Model Aided Program Refinement** — https://arxiv.org/abs/2406.18616
- W3C, **PROV-DM / PROV Primer** — https://www.w3.org/TR/prov-primer/
- Acar et al., work on **Self-Adjusting Computation** and dynamic dependence graphs — https://doi.org/10.1145/2076021.2048101
- Bazel documentation on **hermeticity and incremental builds** — https://bazel.build/versions/8.6.0/basics/hermeticity
- Reproducible Builds, **Definition** — https://reproducible-builds.org/docs/definition/
- Recent agent provenance work, **PROV-AGENT** — https://arxiv.org/abs/2508.02866

## Current EOKS position

The strongest current hypothesis is not that EOKS needs a "reflection layer" or a "cache layer".

It is that reliable AI workloads may benefit from **durable representations whose derivation, supporting evidence, dependencies and validity can be tracked, so that accepted work can be reused and selectively recomputed as the underlying state changes**.

This hypothesis connects reflection, refinement, provenance, evaluation and incremental computation without requiring EOKS to adopt any one of them as its architecture.

The hypothesis remains open and should be tested experimentally.