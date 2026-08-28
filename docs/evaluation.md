# Evaluation

Evaluation is a first-class EOKS subsystem. If the system cannot measure whether a context, model, tool or orchestration decision improved an outcome, it cannot reliably optimize that decision.

## What should be evaluated

- task success and correctness;
- context relevance and sufficiency;
- retrieval precision/recall;
- tool usefulness and failure rate;
- model quality by task class;
- latency and token/cost efficiency;
- robustness to missing, stale or contradictory information;
- regression across model versions;
- calibration of reliability signals;
- usefulness of uncertainty signals for execution decisions.

## Confidence and reliability

An LLM's stated confidence is evidence, not ground truth. EOKS should combine model self-assessment with external signals: tests, static analysis, schema validation, retrieval agreement, execution outcomes and human review where appropriate.

The recent observability discussion adds an important refinement: EOKS should distinguish **observability**, **reliability estimation**, and **control**.

```text
observability
  -> record what happened

reliability estimation
  -> estimate how much the result should be trusted

control
  -> use evidence to decide what happens next
```

Observability systems such as LangSmith/Langfuse-style tracing are therefore useful sensors, not the final confidence mechanism. They can capture prompts, model responses, token usage, latency, tool calls, retrieval steps and evaluation scores, but the control plane still needs to reason about correctness and uncertainty.

Potential reliability signals include:

- token probabilities / logprobs where exposed;
- token-level entropy or related uncertainty metrics;
- semantic agreement across independent generations;
- retrieval/evidence agreement and contradiction;
- deterministic test/static-analysis results;
- tool execution outcomes;
- historical task reliability for a model/workload combination;
- evaluator and human-review scores.

These should be retained as **confidence/reliability evidence**, not collapsed immediately into an opaque scalar.

### Reliability evidence graph

A result can be represented together with the evidence supporting or contradicting it:

```text
candidate conclusion
       |
       +-- model output + uncertainty
       +-- independent answers / agreement
       +-- retrieved evidence + provenance
       +-- tests / static analysis
       +-- tool execution results
       +-- historical outcomes
       |
       v
reliability state
       |
       v
control decision
```

This lets EOKS ask not only "how confident are we?" but also "why should we trust this, what contradicts it, and what evidence would reduce the remaining uncertainty?"

### Calibration

Raw entropy, logprob-derived statistics and model self-ratings should not automatically be interpreted as probabilities of correctness. A useful reliability signal must be calibrated against actual outcomes for the workload and decision being made.

A simplified calibration loop is:

```text
reliability signal
      |
      v
actual task outcome
      |
      v
calibration data
      |
      v
workload-specific reliability model
```

The exact metric should depend on the decision: binary correctness, ranking, expected utility, stop/continue selection, or model routing may require different evaluation methods. Calibration error and proper scoring rules such as Brier-style scores are useful families to investigate.

The principle is:

> **Reliability is workload- and decision-dependent; it must be calibrated against outcomes rather than assumed from a model's raw score.**

## Reliability as a control signal

The most interesting use of observability is not a dashboard but a change in execution policy.

```text
task -> context/model/tool decision -> execution
                         |
                  reliability evidence
                         |
              +----------+----------+
              |          |          |
             high      medium       low
              |          |          |
             stop      verify     branch/retrieve
                         |
                     new evidence
                         |
                      evaluate
```

Possible policies include:

- stop when independent evidence is sufficiently strong;
- run a deterministic check when uncertainty concerns a verifiable invariant;
- retrieve additional context when evidence coverage is poor;
- sample another answer when instability is the main concern;
- route to another model when historical reliability is poor for the workload;
- request human review when the expected cost of an error exceeds the value of more automated work.

The policy should depend on multiple evidence dimensions rather than one universal confidence threshold.

## Model switching

Changing models should be treated like changing an important production dependency. A new model can differ substantially in reasoning behavior even when benchmark scores improve. EOKS should maintain task-specific regression suites and compare candidate models on the actual workload.

A model router therefore needs more than price and latency. It may consider task type, required capabilities, context size, historical reliability, uncertainty and the availability of independent evidence.

## Benchmarking context

A useful context benchmark should compare interventions such as:

- baseline prompt;
- retrieved context;
- structured context;
- graph context;
- compressed context;
- split/progressive context.

The benchmark must measure both quality and cost so that larger contexts are not rewarded merely for consuming more tokens.

Reliability experiments should hold context composition constant when possible; otherwise improvements may be incorrectly attributed to the model or uncertainty signal when the real cause is different information supplied to the model.

## Benchmark before optimization

A practical first benchmark should use roughly 20–30 representative real tasks from the target workload. Include different task classes such as repository navigation, understanding, debugging, impact analysis, implementation, refactoring and verification. Record a baseline before introducing a context engine, knowledge layer or model change.

For each task, record the task contract, configuration, context manifest, execution trace, outcome and evaluation. This makes attribution possible:

```text
task
  -> context decision
  -> context manifest
  -> model/tool decisions
  -> run trace
  -> outcome
  -> evaluation
```

Do not evaluate only textual answer quality for coding agents. Include tests, correctness, completeness, regressions, tool calls, repository rediscovery, tokens, latency and cost.

### Context metrics versus outcome metrics

Retrieval metrics such as precision, recall and relevance are useful diagnostics, but they do not prove that a context intervention improved the task. Conversely, a model may compensate for incomplete retrieval through additional exploration.

A useful experimental quantity is **marginal context value**: the change in task quality attributable to adding a context block relative to its additional token/latency cost. Another useful diagnostic is **context necessity**, labeling selected blocks as essential, useful, irrelevant or misleading after a run.

## Controlled context experiments

Do not introduce multiple context interventions simultaneously. A useful sequence is:

```text
baseline
  -> GrapeRoot-like context engine
  -> + durable knowledge (for example OKF)
  -> Graphify-like structural evidence
  -> combined configuration
```

Hold the model and task set constant when measuring a context intervention. Conversely, hold the context composition constant when comparing models. The goal is to make the causal attribution of an observed improvement or regression as clear as possible.

## Model migration scorecard

A model upgrade should be evaluated as a workload-level dependency migration rather than by a single leaderboard score. A candidate scorecard should include:

| Dimension | Current model | Candidate model |
|---|---:|---:|
| task success | | |
| tests passing | | |
| correctness | | |
| completeness | | |
| groundedness | | |
| serious regressions | | |
| tool calls | | |
| tokens | | |
| latency | | |
| cost | | |

Do not immediately collapse these dimensions into one number. A candidate can be better on average while regressing a high-value task class.

### Model/context interaction

Model and context changes can interact. A useful benchmark matrix is:

| | Baseline context | GrapeRoot | GrapeRoot + OKF | Graph context |
|---|---:|---:|---:|---:|
| Model A | A | B | C | D |
| Model B | E | F | G | H |

This allows separate estimates for model effect, context-engine effect, incremental knowledge/evidence effects and interaction effects. The effects should not be assumed additive.

This matters for model migration because a new model may compensate for weak retrieval, exploit structured context better, or respond differently to large context packs. Therefore model and context migration should be tested together as well as independently.

## Canary and regression workflow

```text
new model/version
       |
       v
run golden set
       |
       v
compare with production model
       |
       +-- critical regression? -- yes --> reject / investigate
       |
       no
       v
staged/canary evaluation
       |
       v
production traces + online evaluation
       |
       v
promote or roll back
```

Online evaluation should feed newly discovered edge cases back into the offline dataset. This creates continuous assurance rather than a benchmark that becomes stale after a model migration.

## Evaluation tools and community prior art

EOKS does not need to implement an evaluation framework from scratch. Existing tools can occupy different roles:

- **Promptfoo** — repeatable comparisons of models, prompts and configurations, including coding-agent evaluation;
- **Langfuse** — datasets, experiments, traces, scores and offline/online evaluation loops;
- **Aider benchmarks** — end-to-end coding-agent evaluation with repository/test outcomes;
- **OpenHands benchmarks** — broader software-engineering and agent benchmark infrastructure;
- **OpenAI Evals and similar frameworks** — reusable evaluation harnesses and private/workload-specific evals.

These should be treated as experiment/evaluation infrastructure, not as the EOKS control plane itself.

## Evaluation loop

`task -> context/model/tool decision -> execution -> outcome -> evidence -> evaluation -> calibration -> policy update`

The long-term goal is continuous assurance: the system should accumulate evidence about which strategies work for which workload classes, and whether its own reliability signals are predictive enough to drive future control decisions.

See [Context evaluation, benchmarking and model migration](../research/context-evaluation-and-model-migration.md) for the detailed experimental methodology and tool map.
