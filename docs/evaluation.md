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
- calibration of confidence signals.

## Confidence

An LLM's stated confidence is evidence, not ground truth. EOKS should combine model self-assessment with external signals: tests, static analysis, schema validation, retrieval agreement, execution outcomes and human review where appropriate.

This suggests a **confidence evidence graph** rather than a single confidence number.

## Model switching

Changing models should be treated like changing an important production dependency. A new model can differ substantially in reasoning behavior even when benchmark scores improve. EOKS should maintain task-specific regression suites and compare candidate models on the actual workload.

A model router therefore needs more than price and latency. It may consider task type, required capabilities, context size, historical reliability and uncertainty.

## Benchmarking context

A useful context benchmark should compare interventions such as:

- baseline prompt;
- retrieved context;
- structured context;
- graph context;
- compressed context;
- split/progressive context.

The benchmark must measure both quality and cost so that larger contexts are not rewarded merely for consuming more tokens.

## Evaluation loop

`task -> context/model/tool decision -> execution -> outcome -> evidence -> evaluation -> policy update`

The long-term goal is continuous assurance: the system should accumulate evidence about which strategies work for which workload classes.
