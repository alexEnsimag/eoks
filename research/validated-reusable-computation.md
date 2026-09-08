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

## Evidence from evaluation and optimization tooling

Recent practitioner tooling helps separate several ideas that are easy to collapse into "reflection."

**Opik** is primarily an observability and evaluation substrate for LLM applications and agents: traces capture executions, evaluations turn traces or dataset runs into evidence, and failures can become regression cases. Its optimizer layer then uses datasets, metrics and traces to compare candidate prompts, tools or workflows. This is concrete prior art for the empirical side of the lifecycle, but it should not be treated as evidence that EOKS needs an Opik-like platform or an observability primitive. Opik's own documentation presents GEPA and HRPO as interchangeable optimization algorithms with different strengths, reinforcing that optimization strategy is a workload-dependent choice rather than a universal EOKS stage.

**GEPA** is better understood as an empirical search/optimization method than as a generic reflection architecture. Reflection is part of its proposal mechanism, but the broader pattern is candidate generation → evaluation → selection → further search. It has also been integrated into other systems, including Opik, DSPy, MLflow and Pydantic AI, which is stronger evidence for the underlying optimization pattern than for any one framework. Its current limitations are equally informative: community reports include budget/accounting mismatches, repeated failed proposals, and concerns about generalization or overfitting. These are reasons to treat optimization evidence as experimental evidence rather than as correctness by itself.

**HRPO** is one particular diagnosis/refinement strategy. Its hierarchical failure analysis is useful prior art for decomposing observation into diagnosis, synthesis and intervention, but it should not be promoted to an EOKS primitive. The fact that Opik exposes HRPO alongside GEPA and other optimizers is evidence that multiple strategies can occupy the same empirical refinement slot.

A useful abstraction is therefore:

```text
execution
   ↓
trace / observation
   ↓
evaluation
   ↓
evidence
   ↓
proposal / diagnosis / search
   ↓
candidate
   ↓
re-evaluation
   ↓
accept / reject / continue
```

The EOKS question begins where this empirical loop is not sufficient by itself:

```text
accepted result
      ↓
what should become durable?
      ↓
knowledge / constraint / test /
representation / validated computation / policy
      ↓
what dependencies make it valid?
      ↓
when should it be invalidated?
```

This distinction prevents three different concepts from being conflated:

- **observability:** what happened during an execution;
- **evaluation/optimization:** what evidence says about a candidate and what change to try next;
- **durable knowledge/reuse:** what should survive the execution and under which future conditions it remains valid.

The community evidence is therefore useful not because these tools collectively "are EOKS," but because they provide increasingly concrete implementations of the empirical half of a lifecycle whose durable-knowledge and dependency-aware-reuse half remains an open EOKS research question.

### Control economics is part of the evidence

Optimization and evaluation also expose a cost dimension that should remain explicit. Reflection calls, metric evaluations, tool calls and candidate trials all consume budget. Recent work on cost-aware evolutionary optimization reports large search-cost reductions by separating cheap high-volume evaluation from stronger models used for reflection/variation, while current GEPA/Opik documentation explicitly tracks reflection calls and distinguishes optimizer-internal scores from fresh evaluation scores.

This strengthens an existing EOKS hypothesis:

> **The value of another evaluation, diagnosis or refinement step must be compared with its additional cost and its expected effect on accepted outcome quality.**

It does not imply that a particular optimizer or cost strategy is universally best.

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
- Opik documentation — https://www.comet.com/docs/opik/
- Opik Agent Optimizer documentation — https://www.comet.com/docs/opik/development/optimization-runs/overview
- GEPA — https://github.com/gepa-ai/gepa
