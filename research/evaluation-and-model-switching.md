# Evaluation, confidence and model switching

## The problem

A recurring concern was whether an AI system can expose meaningful metrics about its own work. “Confidence” is attractive, but a model's stated confidence is not automatically calibrated evidence of correctness.

The useful target is therefore **operational confidence**: a set of signals that helps EOKS decide whether an outcome is trustworthy enough for the workload's requirements.

## Possible evidence sources

Confidence can be estimated from multiple layers:

- model self-assessment;
- agreement between independent attempts;
- deterministic validators;
- tests;
- static analysis;
- tool output;
- source provenance;
- retrieval quality;
- consistency with known project decisions;
- successful execution;
- historical performance on similar tasks;
- human feedback.

No single signal should be assumed sufficient.

## Confidence as a control signal

The interesting use is not displaying a confidence number to a user. It is feeding evidence back into the control loop:

```text
execution
   -> evidence
   -> evaluation
   -> confidence / risk estimate
   -> policy
      |       |
      |       +--> accept
      +----------> retry / verify / escalate / change model
```

This makes evaluation part of orchestration.

## Context evaluation

We discussed the possibility of measuring context quality independently from final task success. Candidate measurements include:

- relevance;
- coverage;
- redundancy;
- contradictions;
- provenance;
- freshness;
- token cost;
- retrieval precision;
- retrieval recall;
- downstream success.

A key experiment would compare two contexts for the same task and ask not “which looks better?” but “which produces better outcomes under controlled conditions?”

## Model switching

A practical motivation came from observing meaningful behavior differences between model generations. Replacing one model with another can change:

- instruction following;
- coding style;
- reasoning patterns;
- tool use;
- context sensitivity;
- failure modes;
- latency/cost;
- willingness to ask for missing information.

Therefore model switching should be evaluated as a workload-level change.

A useful benchmark should include representative real tasks rather than only generic benchmark scores.

## Evaluation layers

A possible evaluation hierarchy:

1. **unit evaluation** — deterministic functions and tool calls;
2. **component evaluation** — retrieval, context compiler, memory selection;
3. **task evaluation** — complete workload outcomes;
4. **workflow evaluation** — multi-step/agent behavior;
5. **system evaluation** — cost, latency, reliability and operational behavior.

This allows EOKS to identify whether a regression came from the model, context, retrieval, tool, orchestration policy or evaluation itself.

## Continuous evaluation

The broader idea resembles continuous assurance: don't evaluate only before deployment. Observe actual workloads and continuously collect evidence.

This creates a feedback cycle:

```text
                    +------------------+
                    |      policy      |
                    +--------+---------+
                             |
                             v
workload -> context -> model/tools -> outcome
   ^                                   |
   |                                   v
   +----------- memory <--- evaluation
```

## Research questions

- Can confidence be calibrated against actual outcomes?
- Which signals predict correctness best for coding tasks?
- Can context-quality metrics predict outcome quality before execution?
- How much evaluation is required before switching models safely?
- Can the scheduler learn model/task affinity?
- When does verification cost more than the expected error reduction?

These are empirical questions and should become benchmarks before becoming architecture.