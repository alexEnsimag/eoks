# Probabilistic decision primitives and Jev

This note records Jev / System One as recent prior art for a capability already present in the EOKS model: **Decision**. The important architectural idea is not adopting Jev, but making narrow semantic judgments available to software as typed probabilistic signals rather than inferred from generated text.

## Jev in one sentence

TypeSafe AI's Jev is a System One model that takes state plus typed questions and returns structured decisions: **Choice**, **Score**, or **Noul**. TypeSafe describes its training approach as Reinforcement Learning for Calibrated Decisions (RLCD), with calibration rather than fluent text generation as the objective.

Source: https://typesafe.ai/blog/introducing-system-one-models-and-jev

## Rosen / Foreman: the control-plane pattern

Josh Rosen's recent Jev writing adds an important systems-level framing to this prior art. His examples treat Jev as a probabilistic decision primitive embedded inside deterministic software, rather than as the owner of an agent workflow.

The recurring pattern is deterministic state/evidence -> bounded question -> probabilistic judgment -> deterministic policy/authorization -> execution -> new state/evidence.

Rosen describes applications including routing, context filtering, semantic tool gating, worker supervision and fast bounded control loops. This is directly relevant to EOKS because it sharpens the existing Decision primitive without requiring another orchestration layer.

Foreman is particularly useful software-engineering prior art: a coding worker performs implementation while a separate supervisor evaluates progress, completeness, tests, drift and verification. The important distinction is not simply that one agent watches another; it is that judgment and execution have different authority. A probabilistic assessment can contribute evidence for a control decision while deterministic policy retains authority over consequential actions.

This gives EOKS a useful three-part conceptual decomposition:

- Environment — intent, knowledge, state, capabilities, workflow and policy.
- Decision — what should happen now given relevant state and evidence.
- Execution — what actually changes the environment.

These are conceptual roles, not proposals for three new runtime primitives. Jev is one possible provider for Decision; agents, tools, workflows and humans remain execution resources/modalities.

Rosen's earlier work on execution lineage and intermediate artifacts is complementary: durable artifacts, evidence and dependencies make decisions reconstructable and allow a new controller or execution attempt to resume without relying on hidden agent memory.

This also strengthens the existing EOKS treatment of context selection. Context is not merely retrieval; selecting whether evidence is relevant or sufficient is itself a potentially probabilistic decision over the current workload state.

Sources:
- Josh Rosen, "Jev in the Wild: Early Architecture Patterns for System One Models" (2026).
- Josh Rosen, work on the Jev software-factory control plane and Foreman.
- Josh Rosen, "From Agent Loops to Deterministic Graphs: Execution Lineage for Reproducible AI-Native Work" (2026).
- Josh Rosen, "Intermediate Artifacts as First-Class Citizens" (2026).

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


## Third-pass community findings: late September–October 3, 2026

A fresh community pass shows that Jev has moved beyond isolated RAG/reranking demos into a broader **decision-model pattern**. The evidence is still early and heterogeneous, so these findings are recorded as prior art and research leads rather than architectural conclusions.

### The ecosystem is converging on a decision-model category

By September 30, an independent catalog tracked roughly 230 Jev repositories across browser agents, coding agents, search/database, model routing, guardrails, research, robotics and real-time decision use cases. This is useful as an ecosystem signal, not as evidence that the projects are mature or effective; the catalog itself says it has reviewed descriptions/READMEs rather than installed or benchmarked every project.

Source: https://jevforagents.com/repositories

A newer synthesis explicitly frames Jev alongside emerging alternatives such as Kev, CLM, GLiNER2.5-Decide, Solar Decide and Laya as **structured decision models**: models intended to classify, score, rank, route or verify rather than generate open-ended text. This supports treating Jev as evidence for a broader provider-neutral Decision abstraction in EOKS.

Source: https://medium.com/@adnanmasood/the-semantic-if-statement-structured-decision-models-as-a-new-primitive-for-ai-software-780bf71a1258

### Agent evaluation is one of the strongest concrete use cases

LangChain's independent Jev-as-a-Judge experiment evaluates fixed agent traces against human labels. In that small experiment, Jev was both inexpensive/fast and substantially less variable than the tested generative judges. The authors explicitly caution that the corpus is small and that repeatability does not establish correctness.

Source: https://www.langchain.com/blog/jev-agent-evals-langsmith

This is especially relevant to EOKS because evaluation can become an **inline control signal**, rather than only a post-run report:

```
agent state / trace
        ↓
typed semantic evaluation
        ↓
decision evidence
        ↓
continue / verify / escalate / stop
```

### Alignment and safety evaluation suggests the same primitive generalizes

RLCDAlignBench evaluates Jev on ten alignment-failure categories across 44 benchmarks and five target models. The study reports a median AUROC of 0.886 zero-shot for a generic question and a 63x lower reported cost than LLM-judge scorers. More importantly for EOKS, it explicitly separates **what is asked** from **which fields of context are supplied**, and finds that context fields matter substantially.

Source: https://arxiv.org/abs/2609.29429

This is another reason to keep **context compilation** and **semantic decision** separate: the same decision model can evaluate different propositions over different projections of state.

### Independent benchmarking is now large enough to expose boundaries

A September 29 benchmark evaluates Jev 1.13.0 over 37 datasets and 346,009 requests spanning classification, routing, NLI, reading comprehension, commonsense reasoning, moderation, legal clauses and rubric scoring. It reports strong results on many conventional tasks, but also degradation on low-resource languages, fine-grained/noisy labels and rubric-based quality judgments. It additionally finds that binary probabilities can rank well while being poorly positioned around a universal 0.5 threshold; threshold tuning materially changes results.

Source: https://arxiv.org/abs/2609.37647

This strengthens the existing EOKS conclusion that **probability is evidence, not a universal confidence scalar**. Calibration and thresholding belong to the specific question, rubric, workload distribution and model version.

### Robustness is now an explicit community research area

An independent robustness catalog collected 132 tests by September 30, focused specifically on probability/calibration behavior under wording changes, option order, distractors, repeated calls, language changes, injected text and abstention. Most were conducted shortly after release and are explicitly described as evidence to inspect rather than settled results.

Source: https://github.com/Yifan-Lan/awesome-jev-robustness

This should become an EOKS experiment category in its own right: **decision stability under context perturbation**. It is different from ordinary model accuracy.

### Production integration is starting to look like control infrastructure

AWS released Strands Decider 2B, explicitly described as a small local decision model for giving agents a fast check before they act. This is important not because AWS validates Jev specifically, but because an independent implementation from another major ecosystem reinforces the architectural pattern: use a dedicated decision model between agent reasoning and execution.

Source: https://thenewstack.io/aws-strands-decider-model/

### Model versioning becomes part of the Decision evidence contract

The emerging Jev ecosystem distinguishes pinned model builds from a rolling alias and exposes the exact model version in responses. This is directly relevant to reproducible EOKS evaluations: a decision distribution is not fully interpretable without the provider/model build, question/rubric semantics and relevant state distribution.

Source: https://jev-ai.org/docs/models/

### Updated EOKS hypothesis

The evidence now supports a sharper abstraction:

**Decision is a semantic computation over an explicit state projection.**

It can produce:
- a categorical choice;
- an ordered score;
- a proposition probability;
- an evaluation result;
- or an abstain/escalate signal.

The control loop can then combine that signal with deterministic policy, authorization, execution and outcome evidence.

The key EOKS boundary remains:

```
knowledge/resources
       ↓
context compilation
       ↓
decision state
       ↓
semantic decision provider
       ↓
decision evidence
       ↓
policy / authorization
       ↓
execution
       ↓
outcome / evaluation
```

The community evidence therefore does **not** justify adding a Jev-specific EOKS primitive. It does justify treating **semantic decision as an increasingly important provider-neutral capability of the existing Decision primitive**.

### New research questions

1. Which decision questions should be evaluated independently and in parallel, versus sequentially because later state depends on earlier decisions?
2. When should a decision model abstain or escalate rather than return its highest-probability choice?
3. How should context provenance and decision provenance be joined so an outcome can be traced back to the evidence that caused a branch?
4. Can decision-model signals reduce frontier-model/tool calls without increasing end-to-end error?
5. How stable are decision distributions under irrelevant-context, adversarial-context and option-order perturbations?
6. How should decision evidence be versioned when the model, question semantics, rubric or context compiler changes?
7. Can the same Decision interface accommodate Jev-like models, token-probability methods, conventional classifiers, deterministic validators and human judgments?



## Fourth-pass findings: mechanism and limitations

### What is actually known about the underlying mechanism

TypeSafe publicly claims three distinct ingredients: a new model architecture, a parallel sampler, and Reinforcement Learning for Calibrated Decisions (RLCD). The observable API confirms that one request can contain multiple typed questions over shared state and that questions are evaluated in parallel, returning typed distributions rather than generated text.

However, the low-level architecture is not public: parameter count, layer architecture, weights, and a complete training recipe have not been disclosed. EOKS should therefore not describe Jev as a particular classifier architecture, distillation system, transformer variant, or other guessed implementation.

The defensible explanation for the cost/latency advantage is at the interface and serving level: Jev does not autoregressively generate an output string, computes bounded decision distributions, and can evaluate multiple narrow questions in one request. This removes output-token generation/parsing and makes parallelism possible. It does **not** establish that the underlying model requires proportionally less internal compute.

Sources:
- https://typesafe.ai/blog/introducing-system-one-models-and-jev
- https://www.jevtypesafeai.com/jev/architecture
- https://api.typesafe.ai/redoc

### RLCD remains a black box

TypeSafe describes RLCD as optimizing for calibrated decisions rather than human preference or verifiable text generation, but the public material does not provide enough detail to reconstruct the training algorithm or data pipeline independently.

For EOKS, calibration should therefore be treated as an intended training objective and an empirical property to measure, not an architectural guarantee.

### A potentially important hidden state: abstention / uncertainty

Sys1Cal-v1 reports a statistical pattern in which Jev's binary Choice probabilities can be modeled substantially better by assuming an unreported third state representing uncertainty or “I don't know”. Recovering that latent mass improves the paper's soft-accuracy metric substantially.

This is a hypothesis about the observed output behavior, not proof of Jev's internal representation. It nevertheless suggests an important EOKS design point: a two-option probability distribution should not automatically be interpreted as exhaustive belief over true/false, and explicit abstention/escalation can be preferable where uncertainty matters.

Source: https://arxiv.org/abs/2609.35342

### Adversarial state is a real control boundary

JevAdvBench tests 812 typed questions and 9,744 single-edit variants against jev-1.13.0. Rewording was relatively stable, but appending an unverified opinion to the state flipped 12.1% of decisions in their test and pushed 38% of confident answers below a 0.8 review threshold.

The engineering implication is important for EOKS: **state must be treated as untrusted input**. The decision probability describes the model's judgment of the supplied state; it does not establish that the evidence itself is trustworthy.

Source: https://arxiv.org/abs/2609.31142

### Type-safe does not mean semantically safe

Typed output prevents malformed or out-of-schema answers, but it does not prevent wrong interpretations or confidently wrong decisions. TypeSafe's own jev-1.13 jaggedness documentation lists literal interpretation, arithmetic/counting, date comparison, indirection, large irrelevant state, adversarial content, contradictory criteria, and generation as known weaknesses.

It also warns against assuming mathematical relationships between separately asked questions: a Choice is a relative selection, while separate Noul questions are absolute propositions. Their probabilities should not automatically be treated as interchangeable or forced to sum to one.

Source: https://docs.typesafe.ai/model-jaggedness/jev-1.13

### Complex reasoning is a boundary

A medical benchmark published in September found Jev competitive with a frontier model on one research-abstract benchmark but substantially worse on diagnosis-heavy case benchmarks. This is useful counter-evidence to broad capability claims: strong bounded semantic judgment does not imply strong multi-step reasoning.

Source: https://arxiv.org/abs/2609.34024

### Workflow decomposition is part of the capability

TypeSafe's own workflow evaluations decompose tasks into many narrow questions plus deterministic rules, then use the resulting probabilities to branch. Their published workflows report that this structured approach outperforms asking a model to execute the same policy as one prompt.

Therefore the meaningful comparison is often **model + decision decomposition + deterministic policy**, rather than Jev versus an LLM as standalone question-answerers.

Source: https://evals.typesafe.ai/

### Refined EOKS model

The strongest formulation after this pass is:

```
untrusted resources / observations
            ↓
provenance + eligibility
            ↓
context / state projection
            ↓
typed semantic decision
            ↓
probability + decision evidence
            ↓
deterministic policy / authorization
            ↓
execution
            ↓
outcome
            ↓
calibration / evaluation
```

The important addition is the **trust boundary before Decision**. Semantic probability answers “what does this state appear to imply?” It does not answer “is this state trustworthy?” or “may this action execute?”

### Mechanism questions still open

- What architecture produces the non-autoregressive decision distributions?
- How does RLCD construct rewards and calibration targets?
- What training data and synthetic-data generation pipeline are used?
- How much of the speedup comes from architecture versus output-space restriction, parallelism and serving?
- Does the apparent latent abstention state correspond to an internal representation or only a statistical property of the output mapping?
- How does calibration transfer across question semantics, rubrics, domains and model versions?
- How does uncertainty compose when multiple correlated decisions control one trajectory?

These remain open questions rather than assumptions in EOKS.
