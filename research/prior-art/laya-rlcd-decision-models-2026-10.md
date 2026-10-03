# Laya and RLCD: learned decision models as control primitives

Laya is useful prior art for the existing EOKS **Decision** primitive. It is not primarily interesting as another agent framework or model router. The important mechanism is a small encoder-based model that evaluates a caller-supplied decision schema against explicit state and returns a distribution over the alternatives.

This note focuses on the implementation and on what Laya adds to the existing Jev/System One research.

## Executive synthesis

Laya makes a useful architectural pattern concrete:

> **state + typed question + bounded alternatives -> probabilistic semantic decision**

The model does not autoregressively generate an answer. The runtime constructs a sequence containing the question, option descriptions and state, and the model scores the option marker positions.

That creates a reusable decision primitive:

```text
DECIDE(
  state,
  question,
  options / rubric
) -> distribution + selected result
```

This is closely related to Jev, but the open implementation makes several mechanisms unusually inspectable:

- ModernBERT-style bidirectional encoding rather than autoregressive generation;
- runtime-defined Choice, Score and Noul decision schemas;
- explicit option/rubric text as part of the model input;
- batched parallel evaluation of multiple questions;
- shortlisting before high-cardinality choice;
- hooks that can observe, modify or short-circuit decisions;
- calibration and option-order evaluation as first-class engineering concerns;
- RLCD fine-tuning that directly rewards probabilistic decision quality.

The strongest EOKS implication is **not "add Laya"**. It is that learned semantic decisions can be treated as **providers of typed evidence to a deterministic control loop**, while context construction, policy, authorization, execution and evaluation remain separate.

## What the code actually does

The core sequence construction in Laya is approximately:

```text
[CLS]
<question type> instructions
[SEP]
[MASK] option A
[MASK] option B
[MASK] option C
...
[SEP]
state
[SEP]
```

Each option receives a marker position. The encoder processes the complete sequence and the decision head scores the marker representations.

The implementation supports three question types:

- **Choice** — choose one alternative from a supplied set.
- **Score** — evaluate an ordered rubric and derive an ordinal result from the distribution.
- **Noul** — a binary semantic proposition, with configurable false/true labels.

The important boundary is that the answer set is **not a fixed classifier vocabulary**. The caller supplies the option labels and their descriptions at inference time.

For example:

```text
question:
  "What should happen next?"

options:
  retrieve: "more repository evidence is needed"
  execute: "the current evidence is sufficient"
  human: "the uncertainty requires user input"
```

The same checkpoint can evaluate a different decision question with a different option set without introducing a new output head.

This is better described as a **schema-conditioned decision model** than as a conventional classifier.

## Why this is different from JSON generation

An LLM can be prompted to emit:

```json
{"action":"retrieve","confidence":0.72}
```

but that does not by itself give a calibrated decision primitive.

Laya's output distribution is produced directly by a dedicated decision architecture. The application can inspect the complete distribution, not just parse the model's generated text.

This makes a useful conceptual distinction:

```text
generative model
  -> proposes structured text

decision model
  -> evaluates a supplied decision space
```

The two can coexist in the same harness.

## High-cardinality decisions reveal an architectural boundary

Laya's option descriptions share a fixed question/head token budget. The implementation progressively constrains option representations when the budget becomes tight.

This matters in practice. The repository's own Banking77 experiment reports roughly 0.425 accuracy with 77 choices, while smaller-choice tasks perform much better. The implementation therefore includes a shortlist path for large candidate sets.

The resulting pattern is:

```text
large candidate space
        |
        v
candidate shortlist
        |
        v
bounded semantic choice
        |
        v
policy / execution
```

This is useful EOKS prior art even if Laya itself is not adopted. Candidate generation/filtering and semantic selection do not necessarily have to be one operation.

It also establishes a practical limitation: these models are naturally suited to **bounded control decisions**, not arbitrary planning over a huge action space.

## Option order is a real learned-model concern

Because the options occupy different positions in the sequence, option order can influence scores.

Laya therefore has explicit robustness checks for permutation effects and supports evaluating reordered options. Its benchmark documents non-zero order sensitivity for some suites and discusses permutation averaging as a mitigation.

This reinforces a broader EOKS principle:

> A machine-readable probability distribution is still an empirical model signal; the interface being typed does not make the underlying judgment invariant.

For a decision used in control, useful evidence can therefore include:

```text
decision
+ distribution
+ question/rubric version
+ model version
+ calibration evidence
+ robustness evidence
+ provenance
```

## The confidence/calibration boundary is especially important

Laya's own benchmark documentation reports substantial over-confidence in the shipped checkpoints and large improvements after temperature fitting.

That is a useful corrective to an overly simple RLCD narrative:

> RLCD does not make every shipped probability universally calibrated.

Calibration is workload-dependent. It depends on the model, question semantics, rubric, option cardinality, state distribution and deployment environment.

The appropriate EOKS contract is therefore:

```text
decision signal
    |
    v
labelled outcomes
    |
    v
reliability measurement
    |
    v
workload-specific calibration
    |
    v
policy threshold
    |
    v
observed outcome
    |
    +----> recalibration / model evaluation
```

This is consistent with the existing EOKS distinction:

```text
probability of an answer
    !=
confidence / distribution concentration
    !=
probability an action is correct
    !=
authorization to execute
```

An explicit `act_probability` field is especially worth treating cautiously: the public implementation/issues document saturation problems in the act head, while the option distribution remains the more directly useful signal. This is concrete evidence that **"the model says it can act" must not become an authorization primitive**.

## RLCD is a learning mechanism, not merely an inference trick

The Laya fine-tuning implementation makes the RLCD idea more concrete.

At a high level, training:

1. obtains decision logits;
2. introduces exploration/noise;
3. turns the resulting logits into a probability distribution;
4. scores the distribution with proper scoring rules;
5. derives a learning signal from those scores;
6. combines this with supervised target information.

The key idea is that the objective evaluates the **quality of the probability distribution**, rather than only whether the argmax label is correct.

This differs from ordinary classification training and from preference-based RLHF.

The useful general abstraction is:

```text
experience / labelled decision
        |
        v
target distribution or outcome
        |
        v
proper scoring rule
        |
        v
learn a decision policy
```

That makes RLCD relevant to EOKS's learning research because agent trajectories can potentially provide the decision/outcome pairs needed to train specialized control policies.

## Connection to EOKS session learning

A possible learning loop is:

```text
frontier agent / human
        |
        | decisions + eventual outcomes
        v
engineering trajectory
        |
        v
decision dataset
        |
        v
small learned decision provider
        |
        +-----------------------------+
        |                             |
    confident                       uncertain
        |                             |
        v                             v
    cheap control                 escalate to
    / routing                     frontier model
                                      / human
```

Candidate questions could include:

- Should more repository evidence be retrieved?
- Is the current evidence sufficient?
- Should a test run happen now?
- Is this change ready for review?
- Should the task be escalated?
- Which of these bounded capabilities is relevant?
- Does this result satisfy the task rubric?

The important point is not that every such decision should become learned. Rules, deterministic analyzers, frontier models and humans remain valid providers.

The learning hypothesis is narrower:

> Repeated engineering experience may allow stable, high-volume semantic micro-decisions to move from an expensive general model into a cheaper learned decision provider, provided the resulting policy remains empirically evaluated.

## Hooks make Laya look more like a decision runtime

The implementation exposes prediction lifecycle hooks including:

- prediction start/end;
- routing;
- model load/eviction;
- errors.

A start hook can also short-circuit inference with a cached result.

That gives the runtime shape:

```text
request
   |
   v
decision hook
   |\
   | \-- cached / policy result -> return
   v
model inference
   |
   v
decision
   |
   v
post-decision hook
   |
   v
host policy / execution
```

This is particularly relevant to EOKS because hooks can be understood as **lifecycle mechanisms around a decision provider**, not as the learning mechanism itself.

That preserves a useful separation:

```text
hooks       -> lifecycle / interception
decision    -> semantic judgment
policy      -> authority / constraints
execution   -> side effects
learning    -> improvement from outcomes
```

## Relationship to Jev

Laya and Jev should not be collapsed into one product category without preserving their differences.

| Dimension | Jev / System One | Laya |
|---|---|---|
| Core idea | typed probabilistic semantic decisions | typed probabilistic semantic decisions |
| Model architecture | proprietary System One | open encoder-based implementation built around ModernBERT |
| Decision types | Choice / Score / Noul | Choice / Score / Noul |
| Runtime-defined options | yes | yes |
| Probability distribution | central | central |
| Calibration | central claim/research area | explicit benchmark + calibration tooling |
| Open weights/code | limited/proprietary model | open implementation/weights |
| Large choice sets | stronger documented cardinality; also uses staged selection | fixed head budget makes shortlisting important |
| Hooks/runtime integration | ecosystem/harness integrations | explicit prediction/router hooks |
| EOKS interpretation | Decision provider | Decision provider |

The useful EOKS abstraction is therefore **provider-neutral Decision**, not Jev or Laya.

## What Laya adds to the existing Jev synthesis

The Jev research already established that:

- semantic judgment can be represented as typed probabilistic evidence;
- deterministic policy should retain authority;
- calibration must be workload-specific;
- decision composition is not automatically calibrated;
- question/rubric/state/version are part of the contract.

Laya adds implementation-level evidence for several of these ideas:

1. **The decision primitive can be implemented with a non-autoregressive encoder.**
2. **The decision space can be supplied dynamically at runtime.**
3. **Multiple narrow decisions can be evaluated efficiently in batches.**
4. **Large action spaces naturally motivate shortlist → decision composition.**
5. **Hooks can surround the decision provider without becoming part of the model.**
6. **Option-order robustness must be tested explicitly.**
7. **Calibration is an empirical deployment property, not something guaranteed by the interface.**
8. **A learned decision provider can be fine-tuned directly from decision outcomes using proper scoring rules.**

## What this does *not* justify

The current evidence does not justify:

- replacing frontier reasoning with Laya generally;
- assuming RLCD produces universally calibrated probabilities;
- using model confidence as authorization;
- treating a decision distribution as an explanation;
- assuming independent decision probabilities compose into trajectory risk;
- introducing a Laya-specific EOKS primitive;
- making learned decision models mandatory for agent control.

The strongest current claim is narrower:

> **Learned semantic decision models are a plausible provider for high-volume, bounded control decisions inside an EOKS loop, where their outputs are treated as empirical evidence and evaluated against workload-specific outcomes.**

## Research questions

The most useful next experiments are now clearer:

1. **Decision-provider comparison** — rules vs frontier LLM vs small classifier vs Jev/Laya on the same EOKS decision set.
2. **Trajectory learning** — train a decision provider from real engineering-session decisions plus eventual outcomes.
3. **Selective escalation** — measure frontier-model calls saved at fixed outcome-quality/assurance levels.
4. **Robustness** — test option-order, irrelevant-context, wording, prompt-injection and state-perturbation sensitivity.
5. **Calibration drift** — measure calibration across repositories, task types, model versions and changing policies.
6. **Candidate selection** — compare direct high-cardinality choice with shortlist → bounded decision.
7. **Decision composition** — evaluate trajectory-level outcomes rather than multiplying step probabilities.
8. **Provider-neutral contract** — define the smallest EOKS Decision evidence schema that can represent Laya, Jev, ordinary model judges and deterministic validators.

## EOKS conclusion

Laya strengthens the existing **Decision + Evaluation + Policy** model without requiring a new EOKS primitive.

The most important conceptual addition is:

> **A learned decision model can be a reusable control-plane resource that evaluates an explicit state projection against a runtime-defined decision schema and returns probabilistic evidence.**

That leaves the responsibilities cleanly separated:

```text
context compilation
        |
        v
decision provider
        |
        v
probabilistic evidence
        |
        v
deterministic policy / assurance
        |
        v
execution
        |
        v
outcome / evaluation
        |
        +------> learning / recalibration
```

This is a useful bridge between the Jev work and EOKS's broader learning research: **learning does not have to mean making the agent itself more generative. It can mean progressively compiling repeated semantic control decisions into cheaper, evaluated decision resources.**

## Sources

- Laya repository: https://github.com/NandhaKishorM/laya
- Laya model implementation: https://github.com/NandhaKishorM/laya/blob/main/laya/common.py
- Laya hooks: https://github.com/NandhaKishorM/laya/blob/main/laya/hooks.py
- Laya benchmark report: https://github.com/NandhaKishorM/laya/blob/main/BENCHMARKS.md
- Laya model weights: https://huggingface.co/convaiinnovations/laya
- Laya RLCD overview: https://laya.studio/learn/rlcd-reinforcement-learning-calibrated-decisions
- Jev / System One prior art: see [probabilistic decision primitives and Jev](probabilistic-decision-primitives-jev-2026.md)
