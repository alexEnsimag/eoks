# Probabilistic decision primitives and Jev

This note records Jev / System One as recent prior art for a capability already present in the EOKS model: **Decision**. The important architectural idea is not adopting Jev, but making narrow semantic judgments available to software as typed probabilistic signals rather than inferred from generated text.

## Jev in one sentence

TypeSafe AI's Jev is a System One model that takes state plus typed questions and returns structured decisions: **Choice**, **Score**, or **Noul**. TypeSafe describes its training approach as Reinforcement Learning for Calibrated Decisions (RLCD), with calibration rather than fluent text generation as the objective.

Source: https://typesafe.ai/blog/introducing-system-one-models-and-jev

## Why this matters to EOKS

The existing EOKS model already treats Decision as a runtime primitive inside a larger reconciliation loop:

```
state / evidence
      ↓
    decision
      ↓
 policy / execution
      ↓
 observation / outcome
      ↓
 evaluation
      ↓
 reconcile
```

Jev makes one useful distinction explicit:

```
semantic judgment
      !=
action authorization
```

A decision model can answer:

- Which route or capability is appropriate?
- Is the evidence sufficient?
- Does this proposed action address the task?
- How risky or reversible is the action?
- Does this need human review?

The host application still owns deterministic validation, authorization, capability/egress policy, execution and evidence recording. The Jev community harness demonstrates this separation explicitly: an LLM proposes, Jev answers narrow questions, code composes the result, and the host independently decides what it may do.

Source: https://github.com/TypeSafeAI/jev-harness

## The probabilistic contract

The interesting primitive is not merely "structured output". It is that the decision exposes a distribution that software can inspect.

### Choice

A closed set of alternatives:

```text
Choice:
  search   0.72
  read     0.20
  execute  0.06
  human    0.02
```

This preserves uncertainty between alternatives instead of returning only the argmax.

### Score

An ordered rubric such as:

```text
risk:
  low → medium → high → critical
```

The result is derived from a distribution over the ordered levels and can represent an intermediate value.

### Noul

A yes/no semantic question:

```text
P("evidence is sufficient") = 0.84
```

This is particularly close to EOKS's existing uncertainty-aware control questions.

## Important distinction: probability, confidence and authority

These must remain separate:

```text
probability of an answer
        !=
distribution concentration / model confidence
        !=
probability the action is correct
        !=
permission to execute
```

For example, a Noul probability of 0.84 is not an authorization token. It can be an input to a policy such as "escalate below 0.9", but that threshold is a workload-specific engineering decision.

The Jev harness explicitly states that its default threshold is uncalibrated and that its confidence field is a distribution statistic, not a probability that the action is correct.

## Calibration is the research question

TypeSafe's central claim is that RLCD makes Jev's probabilities calibrated: broadly, an 0.8 probability should correspond to approximately 80% correctness across an appropriate population.

That claim should **not** become an EOKS assumption.

The relevant EOKS contract is:

```
signal
  → labelled outcomes
  → reliability measurement
  → workload-specific calibration
  → policy threshold
  → observed outcome
  → recalibration
```

Existing EOKS research already establishes this principle for model-native uncertainty. Jev provides a concrete, cheap and typed implementation to test it.

The launch material does not provide a public accuracy leaderboard or a full independent calibration characterization. Early independent studies are mixed across datasets and tasks. Therefore Jev should currently be treated as a strong **experimental prior-art case**, not as evidence that one universal confidence number exists.

Sources:
- https://jevtypesafeai.com/jev/benchmark
- https://github.com/jujumilk3/jev-calibration-audit
- https://github.com/SamuelSacco/jev-exploration

## Composition is another open problem

Even if individual decisions are calibrated, a workflow that combines them through thresholds, branches and repeated decisions is not automatically calibrated.

For example:

```text
p(relevant)
     ↓
retrieve
     ↓
p(evidence sufficient)
     ↓
verify
     ↓
p(action safe)
     ↓
execute
```

The resulting trajectory risk cannot safely be obtained by blindly multiplying the probabilities. Decisions can be correlated, state can change, and later verification can compensate for earlier uncertainty.

This reinforces an existing EOKS principle: retain **step-level decision evidence and trajectory-level outcomes** rather than collapsing the run into one confidence scalar.

## Architectural interpretation

Do not add "Jev" or "probability" as a new EOKS primitive.

Instead, sharpen the existing **Decision** primitive:

```text
Decision
  ├─ semantic question
  ├─ alternatives / rubric
  ├─ result
  ├─ probabilistic signal (when available)
  ├─ evidence / provenance
  ├─ policy interpretation
  └─ downstream outcome
```

The provider can be:

```text
frontier LLM
small classifier
Jev / System One
token probabilities
semantic-uncertainty estimator
deterministic validator
human
```

This keeps EOKS provider-neutral while making **probabilistic judgment a first-class kind of evidence**.

## Relationship to the harness

A useful control-loop decomposition is:

```text
                  frontier model
                       │
                 proposes / reasons
                       │
                       ▼
             ┌───────────────────┐
             │      harness      │
             │                   │
             │ deterministic     │
             │ validation        │
             │        +          │
             │ semantic decision │
             │        +          │
             │ policy            │
             └─────────┬─────────┘
                       │
                  execute / ask
                       │
                       ▼
                   outcome
                       │
                       ▼
                    evaluate
                       │
                       └──────→ next decision
```

This is consistent with the existing EOKS separation between semantic control and execution substrates. Jev is a possible decision provider inside the loop, not the harness itself and not the execution runtime.

## Research directions worth testing

1. **Decision primitive experiment** — compare free-text/JSON judgement, deterministic rules, token-derived uncertainty and Jev on the same narrow decision set.
2. **Calibration experiment** — measure reliability diagrams, Brier score, ECE and risk-coverage on an EOKS-relevant labelled workload.
3. **Threshold experiment** — test whether a calibrated decision signal can reduce expensive frontier-model calls while maintaining outcome quality.
4. **Escalation experiment** — use uncertainty to route cases between deterministic evidence, cheap semantic judgement, frontier reasoning and human review.
5. **Composition experiment** — measure how step-level probabilities relate to trajectory outcomes without assuming independence.
6. **Provider-neutral interface experiment** — define the smallest Decision evidence schema that can represent Jev, ordinary model judges and deterministic validators without making any provider canonical.
7. **Adversarial/context sensitivity experiment** — test whether irrelevant context, wording, option order or prompt injection changes the decision distribution in ways the control policy cannot detect.
8. **Version drift experiment** — pin model versions and re-run calibration after model/provider changes; thresholds should never silently migrate with a moving alias.

## Current EOKS conclusion

The evidence is sufficient to add Jev as **prior art and an experiment target**, because it concretely demonstrates a capability the EOKS model already needs: narrow semantic decisions represented as machine-readable probabilistic evidence.

It is **not** sufficient to make Jev an EOKS dependency, claim that calibrated probabilities compose across an agent trajectory, or introduce a universal confidence scalar.

The stronger architectural hypothesis is:

> **A harness/control loop can treat semantic decisions as typed evidence, optionally carrying calibrated probabilistic signals, while policy and execution remain outside the decision model.**

This extends the existing EOKS Decision + Evaluation + Policy model without adding another primitive.

## Second-pass findings: what was easy to miss

A second pass over the current TypeSafe material and the surrounding ecosystem adds several important nuances.

### The workflow, not the model alone, is the unit TypeSafe evaluates

TypeSafe's own workflow evaluation does **not** simply ask "is Jev a good classifier?" It assumes a fixed compute graph/workflow and compares models executing the same workflow. The company uses predictions from larger external models as reference probabilities rather than ordinary ground-truth labels. TypeSafe also reports that reliable production workflows tend to decompose the problem into many independent questions whose probabilities are consumed by domain-specific code.

This is particularly relevant to EOKS: the strongest Jev story is not "replace an LLM with a smaller model", but **move semantic micro-decisions into an explicit workflow/control graph**.

Source: https://typesafe.ai/blog/introducing-system-one-models-and-jev

### Parallelism is part of the primitive

Jev is not merely a smaller autoregressive model. TypeSafe describes a new architecture plus a parallel sampler, with all outputs in a request produced in parallel. This matters when evaluating the architecture: a harness with many narrow questions can exploit parallel decision evaluation rather than serially asking a general LLM one question at a time.

The API shape therefore suggests:

```
state
  ├─ question A ─┐
  ├─ question B ─┤
  ├─ question C ─┼─> parallel decision evaluation
  └─ question D ─┘
```

This should be considered alongside EOKS's existing decomposition/parallel execution work.

### High-cardinality Choice is not a trivial detail

TypeSafe currently documents a 255-option cardinality for Jev and describes a two-stage score-then-choice approach for higher-cardinality Wikiracing cases. This is useful prior art for a general EOKS pattern: **candidate generation/filtering and semantic selection may be separate control steps** rather than one enormous choice.

### Criteria and question semantics are part of the contract

Choice options should have explicit descriptions, Score levels need meaningful rubrics, and Noul needs a fully specified proposition. The question identifier itself is not sufficient semantics. The current Jev harness also pins question IDs and favorable directions and treats semantic changes as versioning events.

Therefore calibration belongs to:

```
(model, question, rubric, state distribution, version)
```

not simply to a model name.

### There is now real independent calibration evidence, and it is mixed

An independent API-only audit reports domain-dependent calibration behavior, including under-confidence on one corpus and over-confidence on others, and tests option-order invariance, distractor sensitivity, interference and cross-language behavior. It explicitly recommends computing metrics from probabilities rather than the undocumented API confidence field.

This strengthens the EOKS position that **"calibrated" must always be workload- and decision-specific and experimentally verified**.

Source: https://github.com/jujumilk3/jev-calibration-audit

### AnyJev sharpens the abstraction

Nokia Applied Research's AnyJev demonstrates that the interface can be approximated on ordinary open LLMs without fine-tuning: debias the next-token readout, expose typed decisions, then optionally calibrate with 100–500 labelled examples. Its own results show that raw logit probabilities can be badly unsuitable for thresholding even when argmax accuracy is reasonable; calibration can dramatically change the fraction of cases that can be safely automated.

This is strong evidence for separating the **Decision interface** from the **decision-model provider**.

It also exposes a useful hierarchy:

```
raw model scores
    ↓
debias / stabilize readout
    ↓
calibrate on workload
    ↓
policy threshold / abstention
    ↓
observed outcome
```

Source: https://github.com/nokia-applied-research/AnyJev

### A second System One implementation is already emerging

Contrastive-LM (CLM-8B) is an open System One model that separates state and action representations, caches them independently, and exposes a TypeSafe-compatible API. Its authors report strong verifier results after lightweight fine-tuning, while also reporting that zero-shot CLM is comparable to Jev on several fast decision tasks.

This is important because it suggests System One is becoming a **model class/interface pattern**, not just a Jev-specific product.

However, its long-horizon verifier results also show a boundary: a fast decision model can be useful for candidate selection while still being insufficient as a standalone verifier for complex long-horizon tasks.

Source: https://github.com/Contrastive-LM/CLM

### Security changes the interpretation of probabilistic approval

Recent discussion and the Jev harness itself reinforce that the decision input is not automatically trustworthy. Proposal text, evidence and rationale can contain untrusted content, and prompt injection can influence a semantic judge.

Therefore:

```
probability of proposition
        ≠
trustworthiness of proposition's inputs
        ≠
authorization to execute
```

This reinforces EOKS's existing evidence/provenance and deterministic-policy boundary.

Source: https://github.com/TypeSafeAI/jev-harness

### There is emerging real-world research, not only demos

A September 2026 arXiv study applies Jev to nearly half a million Texas crash narratives, using 27 typed probabilistic questions and auditing results against coded fields and blinded human judgments. The study reports strong classification performance and substantial improvement in calibration after recalibration on labels, while explicitly finding that calibration varies by model and must be audited.

This is useful evidence that the paradigm can support large-scale structured extraction, but it is a domain-specific study and should not be generalized to agent-control reliability.

Source: https://arxiv.org/abs/2609.24052

### The missing research question is now clearer

The interesting experiment for EOKS is no longer merely:

> "Can Jev classify an agent state?"

It is:

> **When a workflow decomposes control into many typed probabilistic questions, which decisions should be parallel, which should be sequential, which require independent evidence, and how should their uncertainty drive routing, escalation and execution?**

That is the bridge between Jev and EOKS's control-loop model.


## Context assembly is outside the Jev decision primitive

A useful clarification from examining the API shape is that Jev does **not** appear to be a context-retrieval system. The application supplies the state that the typed question evaluates. In other words:

```
knowledge / resources / evidence
              |
       context / state assembly
              |
              v
        semantic question
              |
              v
             Jev
              |
       probabilistic signal
```

The question is explicit and typed; the state is the evidence/context against which the question is evaluated. This means two different control problems should remain distinct:

1. **Context selection/compilation:** what evidence should be assembled for the decision?
2. **Semantic decision:** given that state, what proposition/category/score is supported?

This distinction is important for EOKS because context compilation is already a first-class architectural concern. A Jev-like provider can therefore sit after context compilation and before policy interpretation without owning retrieval, durable knowledge, or authorization.

The resulting boundary is:

```
Knowledge / evidence
        |
 working-set eligibility + policy
        |
 context compilation
        |
   decision state
        |
 semantic decision provider
   (Jev / LLM / classifier / human)
        |
 probabilistic or categorical evidence
        |
 policy + execution
```

This also means that a semantic decision is only as meaningful as the state supplied to it. EOKS should preserve the provenance, freshness and authority of the evidence entering a decision rather than treating the decision probability as a substitute for evidence quality.

### Implication for the EOKS research question

The interesting experiment is therefore not simply "does Jev classify agent state?" It is:

> **How should EOKS construct and verify the minimum sufficient decision state before invoking a semantic decision provider, and how should uncertainty in that decision affect subsequent control?**

This connects the existing Context/working-set research directly to Decision without merging the two primitives.

Source: https://jevmodel.org/docs/


## Third-pass findings: the state/question boundary is more nuanced

Recent TypeSafe material sharpens the model beyond "pre-written question + context":

### 1. State is broader than a prompt/context blob

TypeSafe explicitly describes System One inputs as unstructured data with an emphasis on **structured program state**, and its workflow examples join records such as alerts, assets, tickets, authorizations and maintenance state before asking questions. The important abstraction is therefore not "prompt + question" but:

```
decision state + typed question(s)
```

The state can contain the evidence and program state needed for the semantic judgment; the question defines the projection of that state being evaluated.

Source: https://typesafe.ai/blog/introducing-system-one-models-and-jev

### 2. Context selection can itself be a Jev decision

This slightly modifies the boundary in the previous section. Jev does not inherently own a memory store or retrieval system, but a Jev-like decision provider can **make decisions about context**. TypeSafe's context-compaction example evaluates each candidate memory/tool result for relevance, category and value, with the application deciding what to retain or reload.

So the architecture should not say "Jev is outside context management." The more precise statement is:

> **Context management is an EOKS responsibility; semantic decision can be one mechanism used inside it.**

That is a much cleaner abstraction boundary.

Source: https://jevmodel.org/use-cases/context-compaction/

### 3. Question semantics are part of the calibrated object

A probability is not meaningfully calibrated merely because it came from a calibrated model. Calibration depends on what proposition/category/rubric is being asked and on the population of states being evaluated. The API makes this explicit by requiring typed questions and criteria/options.

A useful EOKS formulation is therefore:

```
Decision contract =
  question semantics
  + answer type / rubric
  + state distribution
  + provider/model version
  + calibration evidence
```

This is stronger than treating a question as a reusable string prompt.

### 4. Context matters empirically

A new September 2026 RLCD study on alignment failures varied question wording separately from the fields supplied in the input. It found that **context had more effect than question wording**, while still showing that the typed-question formulation could be useful for many detection tasks. This is a useful empirical reason for EOKS to treat context selection and provenance as first-class control concerns rather than assuming the question is the dominant factor.

The result is domain-specific and does not establish general agent-control reliability, but it supports the architectural hypothesis that *what the decision model sees* deserves explicit evaluation.

Source: https://arxiv.org/abs/2609.29429

### 5. Decomposition is part of the control design

TypeSafe's workflow evaluations report that reliable real-world workflows tend to use many independent, decomposed questions whose probabilities are consumed by domain-specific code. Questions can share the same state and be evaluated together.

This suggests an important EOKS distinction:

```
one reasoning step
    ≠
one semantic decision
```

A single reconciliation step may deliberately create several typed decisions in parallel, then combine them through deterministic policy. The decomposition itself becomes part of the control design and should be observable/evaluable.

Source: https://typesafe.ai/blog/introducing-system-one-models-and-jev

### 6. AnyJev gives us a useful provider-neutral experiment

Nokia's AnyJev shows that the **interface** can be reproduced on ordinary open LLMs by reading a constrained decision distribution rather than generating text, then correcting option-position effects and calibrating with labelled data. Its published results also show that raw model probabilities can be poorly calibrated even when classification accuracy is reasonable.

This strengthens an EOKS hypothesis already present in this document:

> **The Decision primitive should describe the contract and evidence, not the internal mechanism used to produce the probability.**

Jev is one provider; an open LLM with a calibrated decision head/readout is another.

Source: https://github.com/nokia-applied-research/AnyJev

### Revised architectural picture

The more precise model is therefore:

```
                KNOWLEDGE / STATE / EVIDENCE
                           |
                    working-set control
                           |
                 context construction
                           |
              +------------+------------+
              |                         |
       deterministic rules       semantic decisions
              |                  (Jev / LLM / human)
              |                         |
              +------------+------------+
                           |
                    evaluated evidence
                           |
                     POLICY / CONTROL
                           |
                       EXECUTION
```

Semantic decisions can participate in **context construction itself** (for example, deciding which evidence is relevant), while remaining distinct from the context/knowledge stores and from the policy that authorizes consequential actions.

This is a stronger formulation of the Context → Decision boundary than treating Jev as simply "after context."
