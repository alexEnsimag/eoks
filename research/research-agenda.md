# EOKS research agenda

The goal is to turn the architecture into falsifiable experiments.

## A. Does context quality matter independently of model quality?

Construct a fixed set of engineering tasks and generate multiple contexts with controlled changes:

- relevant vs irrelevant material;
- redundant vs deduplicated material;
- fresh vs stale information;
- contradictory vs consistent information;
- source-backed vs unsupported claims.

Measure task success, error rate, latency and token cost.

**Hypothesis:** context composition can explain a meaningful fraction of outcome variance even when the underlying model is fixed.

## B. Can context quality be predicted before execution?

Build candidate metrics for relevance, redundancy, contradictions, provenance and coverage. Test whether these metrics predict downstream task success.

A useful metric should generalize across tasks instead of merely correlating with prompt length or human preference.

## C. Does an interactive context workbench help?

Compare:

1. model-selected context;
2. human-selected context;
3. model + human context editing;
4. a policy-driven context compiler.

Measure both outcome quality and human effort.

## D. Can EOKS choose the cheapest sufficient tool?

For code tasks, compare search, AST/indexing, graph traversal, Semgrep-style analysis, CodeQL-style analysis and LLM reasoning.

The objective is not maximum analysis. It is **minimum sufficient evidence**.

## E. Can model selection become workload-aware?

Maintain a corpus of representative tasks and evaluate multiple models. Track performance by task class rather than a single global score.

Then test whether a scheduler can learn useful model/task affinity.

## F. Can confidence become operationally useful?

Compare self-reported confidence with independent evidence and actual outcomes. Test whether combined signals improve escalation decisions.

The output of this experiment should be a calibration/error analysis, not merely a confidence field in an API.

## G. Does persistent memory improve recurring work?

Run repeated tasks over the same project with and without memory. Evaluate:

- task success;
- context size;
- repeated discovery work;
- stale-memory failures;
- retrieval precision;
- maintenance cost.

## H. Is the control plane worth the complexity?

The most important falsification experiment may be a direct comparison:

```text
well-designed agent runtime
              vs
agent runtime + EOKS coordination
```

Use identical models and tools. If EOKS cannot improve reliability, cost, latency or maintainability enough to justify itself, the control-plane abstraction is not yet justified.

## I. What should become a first implementation?

A minimal vertical slice is likely more informative than implementing every plane:

```text
one coding workload
      |
context compiler
      |
model router
      |
verification tool
      |
evaluation
      |
record outcome
```

The system should be small enough to understand end-to-end and instrumented enough to answer the research questions above.

## Success criteria

EOKS should eventually demonstrate at least one concrete advantage over a conventional agent stack, such as:

- better outcomes from better context;
- fewer expensive model calls through scheduling;
- safer model upgrades through continuous evaluation;
- better recurring-task performance through memory;
- stronger correctness through deterministic evidence;
- lower operational cost at equivalent quality.

A diagram alone is not evidence.