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

## Benchmark boundary: model, harness and workload loop

Recent coding-agent benchmark research reinforces a distinction already implicit in EOKS: an end-to-end coding result is produced by a **composite execution system**, not by the model in isolation. Gorinova et al., *Position: Coding Benchmarks Are Misaligned with Agentic Software Engineering* (2026), distinguishes an agent harness from a broader system harness and argues that benchmarks should make the measured construct and component contributions explicit. [Paper: arXiv:2606.17799](https://arxiv.org/abs/2606.17799)

For EOKS, this does **not** justify a new harness abstraction. It reinforces existing boundaries:

- the model is an execution resource;
- context is a workload-specific materialization;
- tools, environment and agent runtime are execution resources/integration boundaries;
- orchestration is a conductor responsibility;
- evaluation provides evidence for reconciliation;
- task/workflow/system outcomes should be evaluated at the level actually being claimed.

The paper's inner/middle/outer feedback distinction is likewise a useful classification of observations by scope and latency, not another EOKS loop. Existing EOKS nested reconciliation loops already provide the mechanism; the benchmark implication is to preserve enough trace and outcome information to distinguish component effects from workload-level effects.

LoopsBench, *From Harness Engineering to Loop Engineering in Coding Agent Evaluation* (2026), provides convergent evidence that sustained software-engineering evaluation increasingly needs to measure the **loop** rather than only localized agent behavior. [LoopsBench: arXiv:2608.00267](https://arxiv.org/abs/2608.00267)

The resulting EOKS rule is simple: **benchmark the construct you claim to measure, and record enough execution/context/evaluation state to diagnose changes without turning each implementation boundary into a new ontology object.**

## Research questions

- Which reliability signals predict correctness for each workload class?
- How well can those signals be calibrated?
- How much evaluation is required before switching models safely?
- When is verification cheaper than the expected cost of an error?
- Can the system learn model/task/context affinity?
- Which context interventions remain valuable after a model upgrade?
- How much trace detail is sufficient to attribute workload-level changes to models, context, providers, execution policy and verification without making evaluation prohibitively expensive?

These questions should be answered empirically through the benchmark methodology rather than becoming architecture assumptions first.
