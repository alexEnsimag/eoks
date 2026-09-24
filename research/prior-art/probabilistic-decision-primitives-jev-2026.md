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
