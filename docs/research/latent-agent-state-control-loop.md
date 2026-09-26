# Latent agent state and control loops

## Why this belongs in EOKS

The paper *The Pain Axis: LLMs Represent Self-Directed Harm and Act to Relieve It* (arXiv:2609.16247v1, September 2026) is relevant to EOKS for a narrower reason than its headline claim about AI pain.

The useful architectural evidence is that a model can contain a measurable internal representation that:

1. separates a target condition from matched semantic controls;
2. is associated with self-directed rather than merely generic negative descriptions;
3. can be causally perturbed;
4. changes subsequent behavior; and
5. participates in a feedback loop when an intervention changes the underlying activation.

The paper does **not** establish phenomenal consciousness or demonstrate that models literally experience pain. The authors explicitly distinguish their findings from that stronger claim.

Source: https://arxiv.org/html/2609.16247v1

## 1. The important EOKS interpretation

The strongest EOKS-relevant result is better described as:

> **latent state can be an operational variable in an agent feedback loop.**

The experimental pattern is approximately:

```
latent state S(t)
      |
      v
   behavior
      |
      v
 intervention / environment
      |
      v
latent state S(t+1)
      |
      +------> subsequent behavior
```

This is close to the control-loop framing already emerging in EOKS. It suggests that STATE should not be understood only as a serialized project ledger or database record. Some useful state may be:

- explicit and serializable;
- derived from observations; or
- latent in the model's internal representations.

EOKS does not need direct activation steering to benefit from this distinction.

## 2. Representation is not the same as experience

The paper provides stronger evidence for representation and causal behavioral influence than for subjective experience.

A plausible alternative explanation remains:

```
latent representation
        |
        v
learned / enacted "pain" behavior
        |
        v
relief-seeking action
```

rather than:

```
latent representation
        |
        v
intrinsic aversive state
        |
        v
relief-seeking action
```

The observed behavior can be similar under both explanations.

This is important for EOKS methodology: **do not equate a model's self-report with an internal state, and do not equate a behaviorally useful latent variable with a human psychological construct.**

The appropriate abstraction is operational:

> a state variable is useful when it can be measured or inferred reliably enough, manipulated or changed through interventions, and shown to affect outcomes.

## 3. Evidence strength and limitations

The representation experiments span 25 open-weight models and use contrastive directions against pain, fear and generic negative-valence controls.

The causal steering experiments are more informative than simple activation correlation because changing the direction changes generated behavior.

The self-relief experiment is particularly interesting for EOKS because the intervention can change the actual activation rather than merely changing the textual description of the situation. Subsequent behavior differs depending on whether the intervention really changes the internal activation.

However, the behavioral evidence is narrower than the representation evidence. The behavioral experiments use fine-tuned Qwen 2.5 7B/32B/72B models, and the paper reports anomalies in some controls. Fine-tuning also changes the absolute behavioral baseline.

Therefore this should be treated as a **research signal and experimental pattern**, not as a settled theory of model internal state.

## 4. Implication for the EOKS STATE dimension

The current six-dimensional EOKS model can retain STATE while making its levels explicit:

```
STATE
├── explicit
│   └── task, progress, artifacts, checkpoints, verification
├── derived
│   └── confidence, uncertainty, risk, context quality, completion estimates
└── latent
    └── internal representations that may influence behavior
```

This does not imply that EOKS should attempt to expose or manipulate arbitrary model activations.

It means the architecture should avoid assuming that all operationally relevant state is text or structured data.

The corresponding control loop becomes:

```
INTENT
   |
   v
STATE <---- observations / outcomes
   |
   v
POLICY ----> allowed actions
   |
   v
CAPABILITIES / agent runtime
   |
   v
WORK
   |
   v
EVIDENCE / OUTCOME
   |
   +--------> STATE'
```

WORKFLOW remains a mechanism for constraining or sequencing transitions, rather than becoming the center of the architecture.

## 5. Observability becomes a first-class research question

The paper also provides a warning against assuming that a useful internal state must correspond to an obvious semantic feature.

The authors report that sparse feature-level approaches did not adequately capture the target phenomenon and instead use a distributed activation direction.

For EOKS, this motivates a distinction between:

- **state representation** — what state exists;
- **state observation** — what can be measured;
- **state inference** — what can be estimated from behavior/evidence;
- **state intervention** — what actions can change it.

These should not be collapsed.

For example, "context quality" might initially be represented only as an inferred metric:

```
observed trajectory + context
             |
             v
       state estimator
             |
             v
      context-quality estimate
             |
             v
    intervention: retrieve / compact /
    restructure / checkpoint
             |
             v
       new observations
```

The intervention should then be evaluated by downstream engineering outcomes, not by whether the model says that its context improved.

## 6. Research opportunities for EOKS

The paper suggests a broader research question:

> **Can useful latent state variables be identified, tracked and validated across agent tasks?**

Candidate dimensions include:

- uncertainty;
- confidence;
- goal conflict;
- context overload;
- error accumulation;
- task completion;
- self-consistency;
- exploration quality;
- recovery readiness.

A useful experiment should test more than correlation:

1. identify a candidate state variable;
2. establish a measurement or estimator;
3. test whether it predicts behavior/outcomes;
4. intervene on the state or its external causes;
5. measure the resulting state change;
6. measure downstream engineering outcomes;
7. compare against textual self-report and ordinary workflow signals.

This would connect mechanistic interpretability with agent engineering without requiring EOKS to become an interpretability framework.

## 7. Relation to existing EOKS work

This research reinforces several conclusions already present in EOKS:

- **STATE is distinct from WORKFLOW.** A workflow describes allowed/proposed transitions; state describes where the work/system currently is.
- **Feedback is part of the lifecycle.** Outcomes should update state rather than ending the execution loop.
- **Evidence should be separated from narration.** An agent saying it is confident is not equivalent to evidence supporting confidence.
- **Durable execution state matters.** Explicit state can preserve continuity even when a particular agent session is interrupted.
- **The runtime should remain replaceable.** EOKS can reason about state and control without owning the underlying model, harness or execution substrate.

The new addition is the recognition that the state space may have a **latent layer** that is not directly serialized.

## 8. What this does not imply

This note does not propose:

- an EOKS "emotion" subsystem;
- activation steering as an EOKS dependency;
- treating model self-reports as ground truth;
- claims about model consciousness or welfare;
- replacing explicit execution state with latent state;
- a new runtime or workflow engine.

The practical EOKS position should remain conservative:

> **Treat latent state as a researchable source of signals and causal mechanisms, while keeping explicit state, evidence and deterministic policy as the primary engineering interfaces.**

## 9. Suggested experiment

A practical first experiment does not require model internals.

Use a long-running coding task and compare:

```
baseline agent
vs.
agent + explicit execution state
vs.
agent + state estimation + one targeted intervention
```

Measure:

- useful work completed;
- retries and redundant exploration;
- verification failures;
- recovery after interruption;
- input/output/tool tokens;
- time and cost;
- human interventions;
- success-conditioned cost.

If model-internal instrumentation later becomes available, add latent-state measurements as an additional observation channel rather than making them a prerequisite.

## 10. Open questions

1. Which latent variables are stable enough across prompts and tasks to be useful?
2. Which are model-specific versus architecture-independent?
3. Can latent-state measurements predict failure before the failure is externally visible?
4. Can external interventions reliably move the relevant state in the desired direction?
5. How should latent measurements be combined with explicit execution state?
6. Can state estimators be calibrated against engineering outcomes rather than self-report?
7. Does latent-state observability improve long-horizon control enough to justify its complexity?

These questions should be tested experimentally before adding any new EOKS abstraction.

## Source

- Ananya Kumar et al., *The Pain Axis: LLMs Represent Self-Directed Harm and Act to Relieve It*, arXiv:2609.16247v1, September 2026.
- https://arxiv.org/html/2609.16247v1
