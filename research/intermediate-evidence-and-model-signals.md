# Intermediate evidence and model-native reliability signals

## Why this matters to EOKS

EOKS increasingly treats execution as more than a request followed by a final answer. A computation can expose useful evidence while it is still running: a tool result, a failed test, an intermediate conclusion, disagreement between attempts, a probability signal from the model, or—when the runtime permits instrumentation—information encoded in internal model representations.

This note formalizes **intermediate evidence** as a research and architectural vocabulary without prescribing how any particular model or runtime must expose it.

The central distinction is:

```text
final outcome
    !=
complete description of the computation
```

An outcome can be correct even when the computation was inefficient, fragile or initially wrong. Conversely, a final outcome can look plausible while intermediate evidence reveals uncertainty, contradictory evidence or a failed validation that deserves attention.

The purpose of intermediate evidence is therefore not to replace outcome evaluation. It is to give evaluation and control access to useful signals **before the final outcome is known**.

## Core concept

**Intermediate evidence** is evidence emitted, derived, or made observable during execution before the final task outcome is established.

It can provide evidence about:

- the task or external world;
- an intermediate result;
- the current execution state;
- the quality or reliability of an intermediate result;
- the behavior or efficiency of the computation itself.

A useful taxonomy is:

```text
Intermediate evidence
├── external evidence
│   ├── retrieval results
│   ├── tool observations
│   └── deterministic validators
├── execution evidence
│   ├── state transitions
│   ├── generated artifacts
│   └── trajectory observations
├── process evidence
│   ├── intermediate conclusions
│   ├── step-level evaluations
│   ├── disagreement / inconsistency
│   └── repeated or redundant work
└── model-native evidence
    ├── token probabilities / log probabilities
    ├── entropy and probability margins
    ├── sampled semantic uncertainty
    └── internal representations / activation-derived signals
```

This is a taxonomy of possible evidence sources, **not a requirement that an EOKS implementation expose all of them**.

## Evidence produced by computation versus evidence about computation

These should remain distinct.

### Evidence produced by computation

Examples:

```text
"The API returned X."
"The repository contains file Y."
"The test failed."
"This intermediate derivation produced Z."
```

These observations describe the task, environment or result of an operation.

### Evidence about computation

Examples:

```text
"This step appears uncertain."
"Two sampled solutions disagree."
"This trajectory is unusually long."
"The current intermediate result is predicted to be erroneous."
"The computation appears to be repeating without discovering new evidence."
```

These observations describe the reliability, state or behavior of the computation itself.

The distinction is useful because a control policy may need both:

```text
computation
   -> observations
   -> evidence about observations/computation
   -> evaluation
   -> control decision
```

## Model-native signals

Language-model inference naturally produces probabilistic quantities. If the runtime exposes them, they can become one source of intermediate evidence.

### Token probabilities and log probabilities

For a generated token, the model computes a conditional distribution of the form:

```text
P(next token | context, previous tokens)
```

When token probabilities or log probabilities are exposed, useful derived signals include:

- mean or length-normalized log probability;
- probability margins between top alternatives;
- predictive entropy;
- entropy trajectories across a generation;
- unusually low probability on critical spans such as names, numbers, tool arguments or structured fields.

These are **model uncertainty signals**, not probabilities that the final answer is correct.

A fluent false statement can have high token probability because the model is estimating how likely that wording is under its learned distribution, not directly evaluating the truth of the proposition.

### Predictive entropy

For a next-token distribution:

```text
H(Y | x) = -Σ_y P(y | x) log P(y | x)
```

Lower entropy means probability mass is concentrated on fewer alternatives; higher entropy means the model is distributing probability across more alternatives.

This makes entropy a natural candidate signal for control. High uncertainty can motivate retrieval, verification, alternative sampling, branching or escalation.

The threshold and interpretation are workload-specific and require calibration.

### Semantic uncertainty

Token-level uncertainty can be misleading when different token sequences express the same meaning. Multiple generations can therefore be grouped by semantic equivalence and uncertainty can be measured over the resulting meanings rather than surface forms.

Farquhar et al.'s **semantic entropy** work provides strong evidence that semantic uncertainty can detect a useful class of hallucinations/confabulations and support selective refusal or additional grounding.

This is particularly relevant to EOKS because many control decisions concern uncertainty about a task proposition rather than uncertainty about its exact wording.

Semantic uncertainty still depends on sampling and on the quality of the semantic-equivalence procedure. It is therefore evidence, not ground truth.

### Self-consistency and agreement

Sampling multiple reasoning paths and observing agreement can provide another signal. Agreement is often useful because independent-looking solutions that converge on the same answer provide more evidence than a single generation.

However:

```text
agreement != correctness
```

Correlated reasoning errors can cause many attempts to agree on the same wrong result. Agreement becomes stronger when combined with independent evidence, deterministic validation or calibrated historical performance.

### Self-reported confidence

A model can also be asked to state how confident it is. Research has shown that verbalized confidence can sometimes be surprisingly useful and, in some settings, better calibrated than raw token probabilities.

But self-reported confidence is still another model output. It should not be treated as privileged access to an internal probability of correctness.

The appropriate question is empirical:

```text
Does this signal predict the correctness/risk relevant to this decision?
```

## Internal representations and hidden states

Transformer inference also produces intermediate numerical representations—often called hidden states or activations—between the input and final output.

The word **hidden** needs care here. It does not mean that the values are physically inaccessible. With an open or instrumentable model runtime, researchers can often capture activations at different layers. A hosted model API may expose none of them.

Three different questions must be separated:

```text
observable
    !=
interpretable
    !=
reusable
```

### Observable

An instrumented runtime may allow an activation tensor to be captured during a forward pass.

### Interpretable

A hidden representation is not normally a human-readable structure such as:

```text
confidence = 0.83
current_plan = "edit auth.go"
error_probability = 0.17
```

It is a high-dimensional numerical representation. Researchers can use probes and other analyses to test whether particular information is encoded in those representations.

### Reusable

Even if a representation contains information predictive of correctness, that does not establish that the representation is stable across prompts, model versions or runtimes, nor that it is a useful persistent computation artifact.

Therefore EOKS should **not** define a `HiddenState` resource or assume access to transformer internals.

Instead:

> Internal representations are one possible model-native source of intermediate evidence when the model/runtime permits instrumentation.

This preserves the model-agnostic EOKS boundary while leaving room for open-model implementations and research experiments.

## Probing internal representations

Recent research suggests that internal activations can contain signals predictive of model errors even when the generated answer itself is wrong.

For example, work on probing arithmetic errors reports that lightweight probes over internal activations can decode information related to predicted and correct answers and can identify erroneous reasoning in controlled arithmetic settings.

This is important evidence for the broader concept of intermediate evidence, but it does **not** establish that hidden states are universally interpretable, reliable, or better than external validation.

The EOKS-relevant abstraction is therefore the derived signal:

```text
internal representation
        |
        v
model-specific probe
        |
        v
intermediate reliability evidence
        |
        v
verify / retry / branch / continue
```

The probe is an implementation/research technique; the evidence it produces can participate in the existing evaluation/control model.

## Process and trajectory evidence

Trajectory evaluation provides another form of intermediate evidence. An execution trace can expose:

- actions and tool calls;
- arguments;
- ordering;
- retries and loops;
- intermediate state changes;
- latency and token consumption;
- recovery after errors;
- unnecessary or redundant work.

Recent agent-evaluation work demonstrates why final outcomes alone can hide important differences in these trajectories. LangChain's AgentEvals supports strict, unordered, subset and superset trajectory comparisons, while Deep Agents evaluates correctness separately from efficiency using observed/ideal step, tool-call and latency ratios.

The important EOKS conclusion is **not** that there is one correct trajectory. Multiple valid paths may exist. An ideal trajectory can be useful as an efficiency reference without becoming the definition of correctness.

Partial-credit and trajectory-aware evaluation research similarly suggests that intermediate actions can carry useful information that binary end-state scores discard.

These findings reinforce an existing EOKS principle:

```text
trajectory = evidence about computation
trajectory != universal correctness criterion
```

See [Agent trajectory evaluation](prior-art/agent-trajectory-evaluation.md) for the dedicated prior-art treatment.

## Intermediate evidence is not intermediate truth

An important failure mode would be to treat every intermediate signal as authoritative.

For example:

```text
high model confidence
    does not imply
correct answer

low model confidence
    does not imply
incorrect answer

multiple agreeing samples
    do not imply
truth

successful intermediate step
    does not imply
successful task

failed intermediate step
    does not imply
failed task
```

Later evidence can correct or override earlier evidence.

This suggests that EOKS should preserve evidence provenance and temporal position rather than simply replacing the current state with the latest confidence value.

## Calibration: turning signals into useful reliability estimates

A raw signal becomes more useful when its relationship with actual outcomes is measured.

For a signal intended to estimate correctness, candidate evaluation methods include:

- reliability diagrams;
- Expected Calibration Error (ECE);
- Brier score;
- AUROC for ranking correct versus incorrect outcomes;
- risk-coverage/rejection curves when abstention or escalation is possible;
- workload-specific post-hoc calibration such as Platt scaling or isotonic regression.

The calibration target must match the decision.

For example:

```text
signal -> probability of correctness
signal -> ranking of candidates
signal -> stop/continue decision
signal -> escalation decision
signal -> expected utility
```

These are different prediction problems and should not automatically share the same calibration.

EOKS should therefore retain the distinction:

```text
model uncertainty
      !=
evidence strength
      !=
probability of correctness
```

## Multiple evidence sources

The strongest direction is not to find a universal "best" confidence signal. It is to combine evidence sources while keeping their provenance and limitations visible.

For example:

```text
model-native signal
        +
semantic agreement
        +
retrieved evidence
        +
deterministic validator
        +
historical workload calibration
        |
        v
reliability evidence
        |
        v
control policy
```

Different sources have different failure modes. Combining genuinely independent evidence can be much stronger than multiplying or averaging correlated signals blindly.

This is consistent with EOKS's existing requirement that reliability remain decomposable and inspectable rather than being forced into one universal scalar.

## Availability hierarchy

EOKS must work across black-box and instrumentable models.

A practical hierarchy is:

```text
1. deterministic external evidence
2. provider-exposed probabilities / log probabilities
3. sampling-based semantic uncertainty
4. agreement / consistency signals
5. model-specific learned reliability estimators
6. internal activation / hidden-state signals where instrumentation exists
```

This ordering is about **availability and implementation requirements**, not a universal ranking of reliability.

For example, a well-calibrated external test can be more decisive for a coding task than any model-native confidence signal, while an internal activation probe may detect an arithmetic error earlier than a final-output validator in a research setting.

The absence of one signal should not make the control plane blind; it changes the available estimator and its cost/limitations.

## Intermediate evidence and control

The existing EOKS control pattern can be expressed as:

```text
state
  -> action / model / tool
  -> intermediate evidence
  -> evaluation
  -> policy
       |       |       |       |
      stop   verify  retrieve  branch/replan
```

Potential control signals include:

- validator success;
- sufficient evidence coverage;
- calibrated uncertainty below a workload-specific threshold;
- independent agreement;
- contradiction detection;
- no new relevant evidence from another step;
- marginal information gain below a threshold;
- expected value of another computation below its cost.

No single signal should be treated as a universal stopping rule.

## Intermediate evidence and computation reuse

Intermediate evidence also creates a bridge to EOKS's computation-reuse research.

Not every intermediate representation should be cached. A useful progression is:

```text
observable
    ↓
interpretable
    ↓
validated
    ↓
stable
    ↓
persistable
    ↓
reusable
```

A model activation normally fails several of these tests. A validated tool result or derived repository artifact may satisfy many more.

This suggests a distinction between:

```text
context reuse
    = reuse information

computation reuse
    = reuse work already performed

model runtime cache
    = reuse implementation-specific inference state
```

The last category should remain separate from EOKS's durable computed-artifact direction.

A more interesting future question is whether some computation can be transformed into a **provenance-aware, validated, reusable artifact**:

```text
computation
    ↓
intermediate evidence
    ↓
derived representation
    ↓
validation + provenance
    ↓
reusable computed artifact
```

This is a research question, not a claim that hidden states themselves are cacheable EOKS artifacts.

## Small thought experiments

### 1. Same outcome, different computation

```text
Run A: correct, 4 steps, 4 tool calls
Run B: correct, 7 steps, 6 tool calls
```

Both satisfy the task outcome. The execution evidence can nevertheless show a meaningful efficiency difference.

This motivates separate outcome and computation-efficiency evaluation.

### 2. High confidence, failed validation

```text
model confidence: high
external test: failed
```

The model-native signal is evidence, but the deterministic validator is stronger evidence for the particular property being tested.

The control policy should not average the two into an opaque scalar and lose their provenance.

### 3. Uncertain intermediate result, successful recovery

```text
step 1: uncertain intermediate result
step 2: retrieve / verify
step 3: corrected result
step 4: successful task
```

Intermediate uncertainty is not equivalent to final failure. It can instead be the signal that caused a successful verification branch.

### 4. Hidden-state error signal

```text
internal representation
        ↓
model-specific probe
        ↓
"intermediate result likely erroneous"
        ↓
verification
        ↓
correct result
```

This demonstrates how internal model evidence could participate in EOKS without requiring EOKS itself to understand transformer activations.

## Relationship to existing EOKS concepts

The concept should connect to existing EOKS material rather than create another parallel subsystem:

```text
task
  -> configuration / context
  -> computation / execution
  -> intermediate evidence
  -> outcome
  -> evaluation
  -> reliability evidence
  -> policy / reconciliation
```

Relevant existing concepts include:

- **Execution trace** — records what happened during computation.
- **Evaluation** — determines what the observed execution means for task quality.
- **Reliability evidence** — keeps evidence dimensions decomposable and calibrated.
- **Context** — supplies information used by the computation.
- **Artifacts / provenance** — support durable derived results and potential reuse.
- **Control / reconciliation** — uses evidence to decide what happens next.

Intermediate evidence is therefore best understood as a **cross-cutting representation of information available during execution**, not as a new execution engine or resource type.

## What should not be imported into EOKS

- No `HiddenState` EOKS resource.
- No requirement that models expose activations.
- No assumption that hidden representations are human-interpretable.
- No universal confidence score.
- No claim that log probabilities are probabilities of correctness.
- No claim that semantic entropy is universally superior to token-level signals.
- No assumption that agreement implies truth.
- No canonical or mandatory trajectory.
- No requirement that every execution expose every evidence category.
- No new uncertainty subsystem separate from evaluation/reliability.
- No assumption that intermediate representations are reusable computation caches.

## Research questions

- Which model-native signals predict correctness for each EOKS workload class?
- When does semantic uncertainty add measurable value over token-level signals?
- Which internal representations remain predictive across prompts, tasks and model versions?
- How much calibration data is required before a model-native signal can safely control stop/continue decisions?
- Can internal signals detect errors earlier or more cheaply than external validation?
- How should correlated evidence sources be combined without overstating confidence?
- How should step-level evidence propagate to trajectory-level risk?
- Which intermediate evidence is worth persisting?
- When does an intermediate representation become stable enough to qualify as a reusable artifact?
- Can EOKS learn which evidence provider is most valuable for a particular workload and decision?
- What is the marginal value of obtaining another intermediate signal relative to its computation cost?

## Prior art and references

### Probabilistic and semantic uncertainty

- Farquhar, Kossen, Kuhn & Gal, **Detecting hallucinations in large language models using semantic entropy**, Nature (2024): https://doi.org/10.1038/s41586-024-07421-0
- Wang et al., **Self-Consistency Improves Chain of Thought Reasoning in Language Models** (2022): https://arxiv.org/abs/2203.11171
- Kuhn et al., **Semantic Uncertainty: Linguistic Invariances for Uncertainty Estimation in Natural Language Generation** (2023): https://arxiv.org/abs/2302.09664
- Kadavath et al., **Language Models (Mostly) Know What They Know** (2022): https://arxiv.org/abs/2207.05221
- Tian et al., **Just Ask for Calibration: Strategies for Eliciting Calibrated Confidence Scores from Language Models Fine-Tuned with Human Feedback** (2023): https://arxiv.org/abs/2305.14975

### Internal representations and error detection

- **Probing for Arithmetic Errors in Language Models** (EMNLP 2025): https://aclanthology.org/2025.emnlp-main.1229/
- **Probing Hidden States for Calibrated, Alignment-Resistant Predictions in LLMs** (2026 preprint): https://arxiv.org/abs/2605.12348

The hidden-state references should be treated as research evidence about what can sometimes be extracted from instrumented models, not as evidence that EOKS can universally inspect or interpret model internals.

### Trajectory and process evaluation

See [Agent trajectory evaluation](prior-art/agent-trajectory-evaluation.md) for the dedicated EOKS prior-art note, including LangChain AgentEvals, Deep Agents efficiency metrics, trajectory-aware evaluation, stochastic evaluation and LLM-as-judge limitations.

Recent related research also includes:

- **Grounded Checklist Partial Credit for Agent Skill Trajectories** (2026): https://arxiv.org/abs/2608.27487
- **Efficient SWE Agent Benchmarking via Trajectory-Aware Evaluation** (2026): https://arxiv.org/abs/2609.01603
- **Cost-Utility Alignment in LLM Agent Trajectories** (2026): https://arxiv.org/abs/2608.26195
- **Reducing Cost of LLM Agents with Trajectory Reduction (AgentDiet)** (2025): https://arxiv.org/abs/2509.23586

These references provide convergent evidence that process information can contain useful signals about correctness, efficiency, cost and failure modes, while also reinforcing the need to distinguish process evidence from final outcomes.

## Working conclusion

The strongest EOKS-level conclusion is not that one model metric is the answer. It is that **reliability and control can use evidence available at multiple stages and levels of computation**.

The abstraction should therefore remain:

```text
intermediate evidence
    = information available during computation
      that can inform evaluation or control
```

Its sources may be deterministic, external, process-level, model-native or model-internal. Their semantics, reliability and calibration remain empirical.

This keeps EOKS abstract while preserving a path from observable execution evidence to increasingly model-native signals, and eventually to the separate question of which validated computational results are worth persisting and reusing.
