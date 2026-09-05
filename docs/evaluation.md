# Evaluation

Evaluation is a first-class EOKS subsystem. If the system cannot measure whether a context, model, tool or orchestration decision improved an outcome, it cannot reliably optimize that decision.

The detailed benchmark methodology and community-tool survey live in [Context evaluation](../research/context-evaluation.md). The probabilistic uncertainty and control-signal discussion lives in [LLM uncertainty, semantic entropy and control](../research/llm-uncertainty-and-control.md). The recent harness, observability and telemetry evidence is synthesized in [Evaluation, reliability and model switching](../research/evaluation-and-model-switching.md) and [LLM observability and reliability signals](../research/llm-observability-and-reliability.md). This page defines the canonical evaluation concepts; the research notes contain the experimental detail.

## What should be evaluated

- task success and correctness;
- context relevance and sufficiency;
- retrieval precision/recall where retrieval is being tested;
- tool usefulness and failure rate;
- model quality by workload/task class;
- latency and token/cost efficiency;
- robustness to missing, stale or contradictory information;
- regression across model/context versions;
- calibration of reliability signals;
- usefulness of uncertainty signals for execution decisions;
- stopping and branching policy quality.

For coding-agent workloads, evaluate the **whole task**, not merely the textual answer. Tests, deterministic checks, files changed, regressions, repository exploration, tool calls, latency, tokens and cost are often more informative than prose quality alone.

## The evaluation record

A useful run should make attribution possible:

```text
task
  -> configuration
  -> context manifest
  -> model/tool decisions
  -> execution trace
  -> outcome
  -> evaluation
```

At minimum, version the task contract, model/configuration, context composition, execution environment and evaluation result. This makes controlled comparisons reproducible.

## Context metrics versus outcome metrics

Retrieval metrics such as precision, recall and relevance are **diagnostics**. They do not prove that a context intervention improved the task, because an agent can compensate for imperfect retrieval through exploration. Conversely, a retrieval change may improve the final outcome without maximizing a narrow retrieval metric.

Useful context dimensions include relevance, coverage, precision, redundancy, contradiction risk, provenance, freshness, dependency completeness, ordering and token/latency cost.

Two experimental diagnostics are especially useful:

- **Marginal context value** — the change in task quality attributable to adding a context block relative to its additional token/latency cost.
- **Context necessity** — after a run, classify selected blocks as essential, useful, irrelevant or misleading.

These are measurement concepts, not assumed universal scores.

## Controlled experiments

Change one important variable at a time where possible:

```text
baseline
   -> context intervention
   -> durable-knowledge intervention
   -> structural-evidence intervention
   -> combined configuration
```

Hold the model and task set constant when measuring context. Hold context and task set constant when comparing models. When interactions are the research question, use an explicit model × context matrix rather than mixing changes implicitly.

## Reliability and confidence

An LLM's stated confidence is evidence, not ground truth. EOKS should combine model self-assessment with external signals such as tests, static analysis, schema validation, evidence agreement, execution outcomes and human review where appropriate.

Where exposed, token log probabilities provide model-native signals from the forward pass. Derived measures include mean/length-normalized log probability, top-token margins and predictive entropy. When token probabilities are unavailable, sampled semantic entropy, semantic agreement and external evidence can provide alternative signals.

Keep three concepts separate:

```text
observability
  -> what happened?

reliability estimation
  -> how much should the result be trusted for this decision?

control
  -> what should happen next?
```

Also keep these quantities separate:

```text
model uncertainty != evidence strength != probability of correctness
```

Raw entropy, logprob-derived statistics and model self-ratings should not automatically be treated as probabilities of correctness. A reliability signal must be calibrated against actual outcomes for the relevant workload and decision.

### Intermediate evidence

**Intermediate evidence** is evidence emitted, derived, or made observable during execution before the final task outcome is established. It can describe the task or external world, an intermediate result, the current execution state, the reliability of an intermediate result, or the behavior and efficiency of the computation itself.

It is deliberately broader than trajectory data and deliberately independent of any particular model implementation:

```text
Intermediate evidence
├── external
│   ├── retrieval
│   ├── tool observations
│   └── validators
├── execution
│   ├── state transitions
│   ├── artifacts
│   └── trajectory observations
├── process
│   ├── intermediate results
│   ├── step evaluations
│   └── consistency / redundancy signals
└── model-native
    ├── log probabilities
    ├── entropy / probability margins
    ├── semantic uncertainty
    └── internal representations / activation-derived signals
```

Intermediate evidence should not be confused with intermediate truth. A high-confidence model step can be wrong; an uncertain step can later be corrected successfully; multiple agreeing samples can share a correlated error. Evidence therefore retains provenance and temporal position and remains subject to evaluation and calibration.

Model-native evidence includes signals available from the model's computation, when the provider or runtime exposes them. Internal representations (hidden states/activations) are one possible source in instrumentable model runtimes, but EOKS does not require access to them and does not define them as an EOKS resource. See [Intermediate evidence and model-native reliability signals](../research/intermediate-evidence-and-model-signals.md) for the research detail and limitations.

### Reliability is multi-dimensional evidence

Reliability should remain **decomposable and inspectable** rather than being forced into a universal confidence scalar. A useful representation can keep several evidence dimensions visible, for example:

```text
ReliabilityEvidence
├── model uncertainty
├── answer agreement
├── evidence agreement
├── evidence quality
├── execution validation
├── historical task reliability
├── evaluator results
├── provenance
└── calibration state
```

These dimensions are representations of evidence, not necessarily separate EOKS subsystems. A particular policy may derive a scalar, rank or expected-utility estimate from them, but the underlying evidence should remain available for audit, recalibration and alternative policies.

Useful calibration/evaluation families include reliability diagrams, Expected Calibration Error (ECE), Brier score, AUROC and risk-coverage/rejection curves. The metric should match the decision being controlled: correctness probability, ranking, selective acceptance, stop/continue, routing or expected utility.

## Evaluation as control evidence

Evaluation is not merely reporting. Its outputs become evidence for future EOKS decisions:

```text
run
 |
v
outcome + trace
 |
v
evaluation / calibration
 |
v
policy evidence
 |
+--> stop
+--> retrieve / verify
+--> branch / replan
+--> context-selection update
+--> model migration decision
+--> evidence-provider selection
+--> knowledge promotion / invalidation
```

A graph should prefer evidence-based termination over arbitrary iteration counts where practical. Candidate stopping signals include validator success, sufficient evidence coverage, calibrated uncertainty below a workload-specific threshold, independent agreement, lack of new relevant evidence, marginal information gain below a threshold, or expected value of another step falling below its cost.

These are policy inputs, not universal thresholds. Long-horizon workflows require trajectory-level evaluation because step-level uncertainty does not simply multiply into a correct trajectory probability; steps are often correlated and later validation can compensate for earlier errors.

## Model migration

A model upgrade is a production dependency migration. The relevant question is not whether a candidate wins a generic leaderboard, but whether it is better for the workload under the context and execution policy actually used.

Maintain a versioned golden task set and compare at least:

```text
task success
correctness / completeness
groundedness
tests / deterministic checks
serious regressions
tool calls / exploration
tokens
latency
cost
reliability calibration
```

Do not immediately collapse these into one score. A candidate that improves the average while regressing a high-value task class may not be safe to promote.

A practical migration loop is:

```text
candidate model
      |
      v
golden set
      |
      v
compare with production model
      |
      +-- critical regression --> reject / investigate
      |
      v
staged / canary evaluation
      |
      v
production traces + evaluation
      |
      v
promote or roll back
```

New production edge cases should feed back into the offline dataset.

## Evaluation layers

EOKS can separate evaluation by scope:

1. **Unit evaluation** — deterministic functions, tools and validators.
2. **Component evaluation** — retrieval, context compilation, memory selection or evidence providers.
3. **Task evaluation** — complete representative workload outcomes.
4. **Workflow evaluation** — multi-step/agent behavior, branching and termination.
5. **System evaluation** — cost, latency, reliability and operational behavior.

This decomposition helps localize regressions instead of attributing every change to the model.

## Context and model interaction

Model and context interventions can interact. A new model may compensate for weak retrieval, exploit structured context better, or react differently to a large context pack.

Use an explicit matrix when this interaction matters:

| | Baseline | Context engine | + durable knowledge | + structural evidence |
|---|---:|---:|---:|---:|
| Model A | A | B | C | D |
| Model B | E | F | G | H |

This separates model effects from context effects and exposes interaction effects. The effects should not be assumed additive.

## Recent evidence: harness and loop evaluation

Recent coding-agent work provides useful evidence for how these existing evaluation concepts should be applied, without requiring a new EOKS architectural layer.

**Agentic Harness Engineering (AHE)** treats harness evolution as a sequence of explicit, reversible component changes whose predictions are checked against subsequent task-level outcomes. Its three observability views—component, experience and decision—are useful representations of the evidence needed to evaluate an intervention. citeturn0academia0

**LoopsBench** shifts attention toward sustained, long-horizon execution: dependency-aware task graphs, regression obligations and the behavior of the complete execution loop. This reinforces the need for workflow/trajectory evaluation in addition to localized step metrics. citeturn0academia1

A broader harness survey and the *Code as Agent Harness* survey likewise treat the runtime around the model as an important contributor to behavior, while identifying evaluation beyond final task success and regression-free evolution as open problems. These are useful evidence categories for EOKS's existing configuration → execution → outcome → evaluation record. citeturn0academia2turn0academia3

The practical rule is: **evaluate the construct being claimed, preserve enough execution state to attribute changes, and keep alternative evidence representations available instead of collapsing them into one score.**

## Research questions

- Which reliability signals predict correctness for each workload class?
- How well can those signals be calibrated?
- When does semantic uncertainty add value over token-level signals?
- How should step-level uncertainty be aggregated into trajectory-level risk?
- How much evaluation is required before switching models safely?
- When is verification cheaper than the expected cost of an error?
- Can stopping policies be learned from calibrated uncertainty and marginal value?
- Can the system learn model/task/context affinity?
- Which context interventions remain valuable after a model upgrade?
- Which combinations of reliability evidence are most useful for different control decisions?
- How much execution evidence is required to attribute an intervention's effect without making evaluation prohibitively expensive?

These questions should be answered empirically through the benchmark methodology rather than becoming architecture assumptions first.
