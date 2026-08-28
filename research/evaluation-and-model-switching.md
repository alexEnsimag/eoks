# Evaluation, reliability and model switching

This note owns the **reliability and model-migration** questions. The broader benchmark design—especially context interventions, OKF, GrapeRoot, Graphify and community evaluation harnesses—is in [Context evaluation and benchmarking](context-evaluation.md).

## Operational reliability

A model's stated confidence is not automatically calibrated evidence of correctness. EOKS should instead reason about **operational reliability**: evidence that helps decide whether an outcome is trustworthy enough for the workload and whether more verification is justified.

Useful evidence can include:

- model self-assessment;
- agreement between independent attempts;
- deterministic validators and tests;
- static analysis;
- tool execution outcomes;
- source provenance and evidence quality;
- consistency with known project decisions;
- successful execution;
- historical performance on similar tasks;
- human review where appropriate.

No single signal should be assumed sufficient.

## Reliability is a control signal

The useful output is not necessarily a confidence number shown to a user. It is evidence that can change execution policy:

```text
execution
   -> evidence
   -> evaluation
   -> reliability / risk estimate
   -> policy
      |       |
      |       +--> accept
      +----------> verify / retrieve / retry / escalate / change model
```

This makes evaluation part of orchestration rather than a dashboard attached after execution.

## Calibration

Raw entropy, logprob-derived statistics and model self-ratings should not automatically be interpreted as probabilities of correctness. A reliability signal must be calibrated against actual outcomes for the relevant workload and decision.

Potential evaluation families include calibration error and proper scoring rules for probabilistic decisions, but the appropriate metric depends on the control decision: binary correctness, ranking, expected utility, stop/continue selection or model routing can require different treatment.

The principle is:

> **Reliability is workload- and decision-dependent; calibrate it against outcomes rather than assuming a raw model score is trustworthy.**

## Evaluation layers

EOKS can separate evaluation by scope:

1. **Unit evaluation** — deterministic functions, tools and validators.
2. **Component evaluation** — retrieval, context compilation, memory selection or evidence providers.
3. **Task evaluation** — complete representative workload outcomes.
4. **Workflow evaluation** — multi-step/agent behavior.
5. **System evaluation** — cost, latency, reliability and operational behavior.

This decomposition helps localize regressions instead of attributing every change to the model.

## Model switching

Replacing a model is a workload-level dependency change. Two models with similar benchmark scores can differ in instruction following, coding behavior, tool use, context sensitivity, failure modes, latency and cost.

The relevant question is:

> **Is the candidate model better for the workload under the context and execution policy actually used?**

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
```

Do not immediately collapse these dimensions into one score. A candidate that improves the average while regressing an important task class may not be safe to promote.

## Model × context interaction

Model and context interventions can interact. A new model may compensate for weak retrieval, exploit structured context better, or react differently to a large context pack.

Use an explicit matrix when this interaction matters:

| | Baseline | Context engine | + durable knowledge | + structural evidence |
|---|---:|---:|---:|---:|
| Model A | A | B | C | D |
| Model B | E | F | G | H |

This separates model effects from context effects and exposes interaction effects. The effects should not be assumed additive.

## Canary and regression workflow

```text
candidate model
      |
      v
versioned golden set
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

Production edge cases should feed back into the offline dataset. This turns model selection into continuous assurance rather than a one-time benchmark exercise.

## Model/task affinity

A single global ranking can hide useful specialization. EOKS can maintain evidence about which model performs reliably for which workload classes and under which context policies. A future router can use that evidence together with cost, latency, context requirements and verification availability.

This should be learned from representative outcomes rather than assumed from model branding or generic benchmark rankings.

## Research questions

- Which reliability signals predict correctness for each workload class?
- How well can those signals be calibrated?
- How much evaluation is required before switching models safely?
- When is verification cheaper than the expected cost of an error?
- Can the system learn model/task/context affinity?
- Which context interventions remain valuable after a model upgrade?

These questions should be answered empirically through the benchmark methodology rather than becoming architecture assumptions first.
