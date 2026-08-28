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

## Evaluation loop

`task -> context/model/tool decision -> execution -> outcome -> evidence -> evaluation -> calibration -> policy update`

The long-term goal is continuous assurance: the system should accumulate evidence about which strategies work for which workload classes, and whether its own reliability signals are predictive enough to drive future control decisions.
