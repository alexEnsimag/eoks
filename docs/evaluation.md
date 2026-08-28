# Evaluation

Evaluation is a first-class EOKS subsystem. If the system cannot measure whether a context, model, tool or orchestration decision improved an outcome, it cannot reliably optimize that decision.

The detailed benchmark methodology and community-tool survey live in [Context evaluation, benchmarking and model migration](../research/context-evaluation-and-model-migration.md). This page defines the canonical evaluation concepts; the research note contains the experimental detail.

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
- usefulness of uncertainty signals for execution decisions.

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

Keep three concepts separate:

```text
observability
  -> what happened?

reliability estimation
  -> how much should the result be trusted?

control
  -> what should happen next?
```

Potential reliability signals include logprobs where exposed, semantic agreement, evidence contradictions, deterministic checks, tool outcomes, historical task success and evaluator/human scores. Raw uncertainty signals should be calibrated against actual outcomes for the relevant workload and decision; they should not automatically be treated as probabilities of correctness.

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
+--> context-selection update
+--> model migration decision
+--> evidence-provider selection
+--> knowledge promotion / invalidation
```

The long-term objective is continuous assurance: EOKS should accumulate evidence about which strategies work for which workload classes, at what cost, and with what failure modes.
