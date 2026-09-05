# EOKS Evaluation Signals and Metrics

## Purpose

The conceptual synthesis identifies what EOKS concepts mean; the operational synthesis identifies what to use, prototype, or investigate. This document adds the next layer: **how we would know whether a mechanism actually helps**.

The goal is not to define one universal agent score. Different mechanisms solve different control problems, and a useful evaluation must measure the capability being tested, its effect on the end-to-end outcome, and its cost and failure modes.

The basic chain is:

`concept → operational hypothesis → observable signals → metrics → evaluation method → failure signatures → decision`

This matrix should be populated from the research corpus and refined by experiments. Metrics are hypotheses about useful measurements, not canonical EOKS primitives.

## Evaluation principles

### 1. Measure the capability, not the tool

A graph, search system, cache, model router, evaluator, or telemetry system is not inherently valuable. State the capability it is expected to provide and evaluate that capability.

Example:

> Structural repository representation should reduce exploration effort and context misses on dependency-oriented tasks without reducing correctness.

The comparison is therefore between acquisition strategies, not between brands of graph tooling.

### 2. Separate outcome from process

Final outcome quality is necessary but insufficient. Two runs can both succeed while differing substantially in exploration, verification, cost, and robustness. Conversely, a shorter trajectory can be worse if it skipped necessary evidence.

Use both:

- **Outcome signals:** correctness, completeness, acceptance criteria, defect escape.
- **Process signals:** trajectory, tool choices, intermediate evidence, retries, backtracking, context acquisition, verification.

Trajectory evaluation is especially useful for long-horizon workloads where a terminal result does not reveal where or why a failure occurred.

### 3. Separate evidence quantity from evidence quality

More evidence is not automatically better. Multiple providers can be correlated or derive from the same source. Evaluate whether evidence is relevant, sufficient, independent where appropriate, fresh, and actually changes the control decision.

### 4. Always measure cost and latency

A mechanism that improves quality by multiplying model calls, tools, or verification passes may not improve the system. Report quality together with tokens/computation, latency, tool/provider cost, and human effort.

### 5. Measure negative and dangerous signals

For every hypothesis, define:

- **Positive signal:** expected improvement.
- **Neutral signal:** capability works but does not materially change the workload.
- **Negative signal:** cost without corresponding benefit.
- **Danger signal:** apparent confidence/efficiency improves while correctness, grounding, or safety degrades.

### 6. Prefer paired and repeated comparisons

Use the strongest practical baseline, controlled interventions, representative scenarios, and repeated trials where behavior is stochastic. Report variance rather than treating one successful run as evidence.

### 7. Do not collapse unlike metrics into one score too early

A composite score can hide important tradeoffs. Preserve the dimensions first; aggregate only when the decision actually requires a utility function.

---

## Metric families

These families recur across the research and provide a common vocabulary without becoming EOKS primitives.

| Family | What it measures | Representative metrics |
|---|---|---|
| Outcome quality | Did the task achieve the intended result? | correctness, completeness, acceptance rate, defect escape, regression rate |
| Evidence / assurance | Why should we trust the result? | evidence coverage, evidence sufficiency, grounding, verification pass rate, false-stop rate, calibration, evaluator agreement |
| Context health | Was the right information available? | context size, relevant-context ratio, context misses, redundancy, churn, stale-context rate |
| Acquisition | How efficiently was information obtained? | acquisition iterations, provider calls, discovery recall/precision, missed-resource rate, redundant acquisition |
| Trajectory / process | Was the path effective? | steps, retries, backtracking, wasted calls, subgoal completion, trajectory quality |
| Computation efficiency | How much computation was required? | tokens, model calls, computation steps, latency, repeated computation, cache/artifact hit rate |
| Reuse / lifecycle | Did prior work retain value safely? | reuse rate, avoided recomputation, stale-hit rate, invalidation rate, invalidation precision, maintenance cost |
| Autonomy / intervention | How much oversight was required? | escalation rate, human interventions, verification burden, unsupported autonomous actions |
| Economics | What did the result cost? | cost per successful task, cost per verified task, marginal assurance cost, cost distribution |
| Robustness | Does behavior hold across trials and changes? | success variance, consistency, confidence intervals, drift, sensitivity to perturbation |

These are measurement dimensions, not requirements that every experiment must report every metric.

---

## Concept → signals → metrics matrix

### Context and working set

| Concept / hypothesis | Observable signals | Strong metrics | What to look for | Failure signatures |
|---|---|---|---|---|
| Context acquisition | resources requested, returned, selected, rejected, later found relevant | context misses, relevant-resource recall/precision, acquisition calls, context tokens, redundant acquisition, latency | Equal/better task outcome with less unnecessary acquisition and fewer missed relevant resources | over-pruning, retrieval thrashing, missed dependencies, stale information |
| Working-set construction | resources entering/leaving working set, selection rationale, provenance, freshness | relevant-context ratio, coverage, working-set size, churn, stale-resource rate, verification effort | Smaller or more stable working set without loss of required evidence | context too narrow, representation bias, excessive churn |
| Context contracts | expected information, supplied information, missing information, contract violations | contract coverage, missing-field rate, downstream recovery cost, failure rate | Explicit requirements reduce avoidable context failures | contracts become rigid, false completeness, high maintenance |
| Context observability | context manifest, source, version, timestamps, selection reason | provenance completeness, reproducibility of working set, stale-context detection | Ability to explain what context the run actually used | invisible context mutation, unverifiable provenance |
| Context reuse | prior context/artifacts reused, cache hit/miss, invalidation | reuse rate, tokens avoided, latency avoided, stale-hit rate | Reuse produces real savings without reducing correctness | stale reuse, cache pollution, low hit rate |

### Tool and provider selection

| Concept / hypothesis | Observable signals | Strong metrics | What to look for | Failure signatures |
|---|---|---|---|---|
| Evidence-aware provider selection | requested evidence, candidate providers, selected provider, outputs, selection rationale | evidence coverage, provider precision/recall, cost, latency, unnecessary-provider rate | Minimum sufficient provider set achieves the required assurance | blind spots, premature stopping, provider thrashing |
| Tool selection | subgoal, chosen tool, arguments, result, necessity | tool-selection accuracy, argument correctness, unnecessary-call rate, recovery rate | Correct tools selected for the evidence gap rather than by habit | wrong-tool loops, excessive calls, missed specialized capability |
| Model routing | task characteristics, selected model, outcome, cost, latency | quality-adjusted cost, success rate by route, routing regret, latency | Cheaper/faster routes preserve required quality | routing instability, provider-specific blind spots, quality cliffs |

### Verification and evidence

| Concept / hypothesis | Observable signals | Strong metrics | What to look for | Failure signatures |
|---|---|---|---|---|
| Deterministic verification | generated result, check invoked, check result | verification pass rate, defect detection rate, false-positive/false-negative rate, verification cost | Deterministic checks catch errors that generation alone misses | checks are too weak, too expensive, or create false confidence |
| Intermediate evidence | intermediate artifacts, claims, tool outputs, checkpoints | evidence coverage, evidence-to-decision usefulness, unsupported-claim rate | Intermediate evidence allows earlier detection/correction | evidence volume grows without improving decisions |
| Evidence sufficiency | evidence requested, obtained, accepted/rejected, final decision | sufficient-evidence rate, false-stop rate, unnecessary-escalation rate | Controller stops when evidence is sufficient, not merely when a model sounds confident | premature stopping, endless verification |
| Evaluator validity | evaluator score, reference/ground truth, human judgments, repeated judgments | agreement, calibration, ranking consistency, position flip rate, false-positive/negative rates | Evaluator changes reliably track real quality | judge preference, instability, disagreement with trusted outcomes |
| Model-native / hidden signals | provider-native scores/states when observable, outcome correlation | calibration, predictive value, stability across versions/tasks, incremental information | Native signals provide useful predictive evidence beyond external checks | unstable signal, provider lock-in, correlation mistaken for assurance |

### Trajectory and execution

| Concept / hypothesis | Observable signals | Strong metrics | What to look for | Failure signatures |
|---|---|---|---|---|
| Trajectory capture | ordered actions, observations, intermediate states, retries | task success, subgoal completion, step count, wasted-step rate, backtracking, trajectory quality | Process metrics explain outcome differences and expose avoidable work | optimizing path length causes skipped evidence |
| Durable execution state | checkpoints, resumptions, state transitions, recovery events | recovery success, duplicated work, state-loss rate, resume latency | Interrupted work resumes without silently repeating or losing critical state | inconsistent state, hidden assumptions, duplicate side effects |
| Adaptive orchestration | branch decisions, feedback, route changes, stopping decisions | quality/cost tradeoff, adaptation gain, unnecessary branching, recovery rate | Feedback changes behavior beneficially rather than merely adding steps | oscillation, overreaction, uncontrolled complexity |

### Computation and reuse

| Concept / hypothesis | Observable signals | Strong metrics | What to look for | Failure signatures |
|---|---|---|---|---|
| Computation/artifact reuse | computation identity, inputs, dependencies, artifact, reuse/invalidation events | reuse rate, recomputation avoided, latency/cost avoided, correctness, stale rate | Expensive computation is reused safely across tasks | hidden dependencies, false validity, artifact pollution |
| Dependency-aware invalidation | dependency graph, source changes, invalidations, recomputations | invalidation precision/recall, stale-use rate, unnecessary recomputation | Only affected artifacts are invalidated and stale artifacts are not reused | invalidation storms, missed dependencies |
| Deterministic promotion | repeated trajectories, extracted procedure, validation, deterministic execution | repeatability, success rate, verification cost, drift detection | Stable successful behavior can become cheaper/more predictable | accidental habit promotion, brittle procedure, distribution shift |
| General reusable state | reusable information/computation/behavior, provenance, scope, validity | reuse benefit, invalidation rate, maintenance cost, stale-hit rate | Different reuse mechanisms share lifecycle properties without forcing one implementation | abstraction hides important differences between mechanisms |

### Memory and learning

| Concept / hypothesis | Observable signals | Strong metrics | What to look for | Failure signatures |
|---|---|---|---|---|
| Durable memory | memory created, retrieved, relied upon, invalidated | retrieval usefulness, reuse benefit, stale-memory rate, contradiction rate | Memory reduces repeated work while remaining trustworthy | polluted memory, contradictions, irrelevant retrieval |
| Procedural learning | behavior/skill candidate, validation, rollout, later executions | repeatability, success improvement, regression rate, rollback rate | Learned procedures improve repeated workloads | accidental habits, overfitting, model-version sensitivity |
| Knowledge maintenance | source change, impacted representations, derivations, validation, invalidation | update latency, stale-knowledge rate, impact precision, maintenance cost | Derived knowledge tracks authoritative change | silent staleness, over-invalidation, expensive rebuilds |

### Autonomy, risk, and governance

| Concept / hypothesis | Observable signals | Strong metrics | What to look for | Failure signatures |
|---|---|---|---|---|
| Graduated autonomy | evidence, consequence/risk, selected autonomy level, verification/escalation | autonomous-success rate, escalation rate, unsupported-action rate, consequence-weighted error | More reliable evidence permits appropriate autonomy without weakening safeguards | confidence mistaken for authority, risky actions taken with weak evidence |
| Risk/consequence | action consequence, uncertainty, required assurance, chosen action | consequence-weighted error, assurance coverage, risk-adjusted cost | Higher-consequence actions receive stronger assurance | same verification policy applied to every task |
| Authority/governance | actor/provider, resource/action, policy decision, approval/escalation | unauthorized-action rate, policy violations, valid escalation rate | Capability and authorization are evaluated separately | technically correct but unauthorized actions |

### Research-first areas

| Concept | Signals to investigate | Candidate metrics | Current status |
|---|---|---|---|
| Latent/iterative computation | observable compute budget proxies, intermediate outputs, outcome under budgets | quality vs compute curve, marginal utility of compute, stopping accuracy | Research first |
| Hidden/model-native state | availability, task correlation, version stability, reproducibility | predictive value, calibration, cross-version stability, incremental information | Research first |
| Learned cache/replacement policies | cache decisions, reuse, eviction, future demand | hit rate, avoided compute, stale-hit rate, policy regret | Research first |
| General reusable-state abstraction | common lifecycle across information/computation/behavior | reuse benefit, lifecycle cost, semantic loss from abstraction | Research first |

---

## Evaluation methods by evidence situation

No single evaluator is appropriate for every workload.

### Deterministic checks

Prefer deterministic evaluation when correctness can be specified directly:

- tests
- schema/type validation
- invariant checks
- static analysis
- exact artifact comparison
- state-transition checks
- dependency validation

These are particularly valuable as independent verification of probabilistic generation.

### Reference-based evaluation

Useful when a trusted reference path or result exists. Measure exact or relaxed trajectory/path agreement, but do not assume there is only one valid path for open-ended tasks.

For known narrow workflows, trajectory precision/recall or ordered matching can expose unnecessary or missing steps. For open-ended workflows, reference-free trajectory rubrics are often more appropriate.

### Rubric / judge evaluation

Use structured rubrics when correctness or trajectory quality cannot be completely specified deterministically. Separate:

- outcome quality
- grounding/evidence
- trajectory quality
- tool-use quality

Validate the evaluator itself against trusted examples or human judgments rather than treating judge output as ground truth.

### Repeated-trial evaluation

For stochastic systems, repeat the same scenario. Measure consistency and variance and report uncertainty around aggregate results. A single successful trajectory is weak evidence of a capability.

### Counterfactual / intervention evaluation

For an EOKS mechanism, compare a baseline with one controlled intervention:

`same workload → baseline`
`same workload → mechanism enabled`

The strongest evidence comes when the intervention changes the predicted signals without introducing unacceptable regressions elsewhere.

---

## Tool evaluation protocol

When evaluating a concrete tool or workflow mechanism:

1. **State the capability hypothesis.**
2. **Define the evidence gap it is intended to close.**
3. **Choose a strongest practical baseline.**
4. **Instrument the signals before judging the result.**
5. **Run representative scenarios, including failure cases.**
6. **Repeat stochastic workloads.**
7. **Measure outcome, process, evidence, and economics.**
8. **Look for negative and danger signals.**
9. **Compare against the hypothesis, not against a preferred implementation.**
10. **Record the resulting epistemic and action status in the operational synthesis.**

A useful result should answer:

> What changed, why did it change, did it improve the intended capability, what did it cost, and under what conditions did it fail?

---

## Minimum evaluation record

Each experiment should preserve enough information to reproduce the reasoning behind the decision:

```text
Concept:
Operational hypothesis:
Workload/scenario:
Baseline:
Intervention:
Signals captured:
Primary metrics:
Secondary metrics:
Positive signals:
Negative signals:
Danger signals:
Failure cases:
Repeated-trial / variance method:
Evaluator / ground truth:
Result:
Epistemic status change:
Action status change:
EOKS architectural implication:
Decision rationale:
```

This is deliberately a lightweight record rather than a new runtime primitive.

---

## First vertical evaluation slice

The first practical implementation should instrument one representative workflow end to end:

`Task → intent/acceptance criterion → acquisition/provider selection → working set/context → execution → trajectory/observations → verification → Outcome → Evaluation → reusable artifact / learning candidate → next Decision`

At minimum, capture:

- task and acceptance criterion;
- resources requested and supplied;
- provider/tool choices and rationale;
- context size and provenance;
- model/computation calls;
- trajectory and intermediate evidence;
- verification performed and result;
- final outcome;
- cost and latency;
- reuse/invalidation events;
- human intervention/escalation;
- failure and recovery events.

The first interventions should remain the three already identified in the operational synthesis:

1. context / working-set selection;
2. minimum-sufficient evidence / provider selection;
3. computation / artifact reuse.

The goal is not to build a complete evaluation platform first. The goal is to establish a measurable feedback loop that can tell us whether these mechanisms deserve further investment or architectural promotion.

---

## Relationship to EOKS synthesis

This document intentionally does **not**:

- add `Metric`, `Evidence`, `Computation`, `Risk`, `Autonomy`, `Authority`, `Reuse`, `Lineage`, or `DerivedState` as canonical runtime primitives;
- replace the conceptual synthesis;
- replace the operational synthesis;
- replace the subcategory-based tool-selection matrix;
- canonize specific vendors or tools;
- claim experimental results that have not yet been measured.

Its role is to make the synthesis falsifiable and operational:

`concept → hypothesis → signals → metrics → experiment → evidence → synthesis update → decision`

The strongest future research PRs should therefore be able to say not only **where a concept fits**, but also **what we would observe if it were useful**.

## Research grounding

The matrix builds on the evaluation themes already incorporated into the EOKS research corpus: trajectory/process evaluation, evaluator validity and agreement, intermediate and model-native evidence, computation/reuse, context/working-set selection, provider/tool selection, deterministic verification, stochastic evaluation, and economic efficiency.

Recent external evaluation practice reinforces several of these distinctions: agent evaluation frameworks increasingly score tool trajectories and multi-turn task success separately, while agentic evaluation suites expose step count, latency, token consumption, goal completion, and trajectory matching as distinct signals. These are useful supporting evidence for the matrix, not reasons to adopt any particular framework.
