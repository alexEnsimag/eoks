# Opik: tracing, evaluation, experiments and optimization as an empirical control loop

## Status

This is a **research/prior-art note**, not an EOKS architecture decision. It records what the open-source Opik project demonstrates and extracts concepts that may inform the EOKS synthesis.

## Executive synthesis

Opik is unusually relevant to EOKS because it provides a concrete implementation of the **empirical improvement side** of the broader EOKS hypothesis:

```text
execution
   ↓
trace / outcome
   ↓
evaluation
   ↓
experiment evidence
   ↓
diagnosis / reflection
   ↓
optimization
   ↓
new execution
```

Its important contribution is not generic LLM observability. The current project connects traces, datasets, evaluations, experiments, regression tests and optimization into a repeatable feedback loop. Opik's optimizer package includes GEPA, a hierarchical reflective optimizer (HRPO/HAPO in current naming), evolutionary and Bayesian approaches, parameter optimization and tool optimization.

The EOKS interpretation is:

> **Opik is strong prior art for the execution → evidence → evaluation → refinement loop; EOKS remains concerned with the additional knowledge lifecycle in which validated evidence and derived computation can become durable, provenance-aware, dependency-aware and selectively reusable state.**

This distinction keeps EOKS from becoming another observability/evaluation platform while making established evaluation infrastructure a potential component of the EOKS control plane.

## 1. What Opik actually provides

The current repository describes coverage across the LLM application lifecycle, including agent tracing, evaluation, production monitoring, experiments and optimization. The optimizer is a separate package with a common optimization surface over prompts, agents, parameters and tools.

The important objects form a useful empirical chain:

```text
Dataset
   │
   ├── item
   │
   ▼
execution / agent run
   │
   ▼
trace / spans
   │
   ▼
evaluation result
   │
   ▼
Experiment
   │
   ▼
optimization run
```

The implementation preserves links between experiment items and dataset items/traces. Current backend/frontend code also models experiment type, evaluation method, metadata and feedback scores rather than treating an evaluation as an ephemeral scalar.

### EOKS mapping

| Opik concept | EOKS interpretation |
|---|---|
| Dataset | versioned workload/test population |
| Dataset item | individual workload case |
| Trace/span | execution evidence |
| Evaluation result | evidence about execution quality |
| Experiment | controlled comparison of execution/configuration |
| Feedback score | evaluation evidence, potentially multi-dimensional |
| Regression/test case | durable executable constraint |
| Optimization run | repeated empirical refinement |
| Prompt/tool/model configuration | candidate intervention/dependency |

These are mappings, not claims that Opik uses EOKS terminology.

## 2. Trace → test is more important than trace → dashboard

One of the strongest EOKS-relevant patterns is the conversion of observed failures into durable regression cases.

The lifecycle is approximately:

```text
production execution
       ↓
      trace
       ↓
   bad outcome
       ↓
  test / dataset item
       ↓
 future experiment
       ↓
 regression evidence
```

This is a stronger form of learning than an unconstrained memory summary. A retained failure becomes an **executable future constraint**.

It suggests a useful EOKS vocabulary distinction:

```text
memory
  = remember this

knowledge
  = this appears to be true

constraint
  = future work should satisfy this

test
  = future work can be checked against this

validated computation
  = this result may be reused if its validity conditions still hold
```

Opik gives concrete prior art for the transition from execution evidence to durable tests. EOKS should investigate the broader transition from evidence to multiple possible durable states: knowledge, constraint, context, policy, representation or reusable computation.

## 3. Experiments are a useful boundary object

An Opik experiment connects a workload population with actual execution and evaluation evidence. This is close to the EOKS research model:

```text
workload
   ↕
execution trace
   ↕
evaluation
   ↕
configuration/version
```

This is useful because it makes controlled comparison explicit. EOKS's benchmark methodology already recommends:

```text
model × repository × task × intervention × budget
```

and separates benchmark, experiment, trace and evaluator responsibilities.

The resulting architectural boundary should therefore remain:

```text
benchmark      → defines representative tasks
experiment     → runs configurations
trace          → records what happened
evaluator      → scores outcomes
EOKS           → uses evidence for policy/control
```

Opik is strong prior art for the first four components. EOKS should prefer integration with such infrastructure over recreating a generic evaluation runner.

## 4. GEPA: reflection plus empirical selection

The Opik GEPA adapter is revealing because it does more than call a reflection model once. It adapts GEPA to Opik datasets and agents, tracks dataset item identities, distinguishes full validation/trial evaluations from mini-batch screening, and records candidate metadata and scores.

The implementation also contains an explicit compatibility boundary around GEPA's native adapter and a fallback local evaluation path. This is useful evidence that an optimizer is not just a prompt transformation function: it is an orchestration layer over candidate generation, execution, scoring, experiment recording and selection.

A simplified model is:

```text
candidate
   ↓
execute on evaluation cases
   ↓
score
   ↓
select / compare
   ↓
reflection
   ↓
new candidate
```

The Opik GEPA integration uses Pareto-oriented candidate handling and supports multi-metric objectives. This matters to EOKS because optimization should not silently collapse correctness, quality, cost, latency and other objectives into one unexamined scalar.

### EOKS implication

The useful abstraction is not "prompt optimizer". It is:

> **an empirical refinement controller that proposes changes, evaluates them against a workload, records the evidence and selects among candidates.**

That abstraction can apply to prompts, context strategies, retrieval policies, tool configurations, model choices and other interventions.

## 5. Hierarchical reflective optimization: reflection becomes synthesis

Opik's hierarchical reflective optimizer is particularly relevant to the EOKS synthesis because its implementation has an explicit two-stage analysis:

```text
full evaluation result
        ↓
 split into batches
        ↓
parallel failure-mode analysis
        ↓
batch analyses
        ↓
synthesis
        ↓
unified failure modes
        ↓
improvement proposal
        ↓
re-evaluation
```

The implementation requires score reasons for the analysis. Each test result contributes its scores and reason text; batches are analyzed concurrently; then the analyses are synthesized into unified failure modes. A subsequent improvement step uses the identified root cause to generate revised prompts/tools, which are evaluated against the dataset.

This is more interesting than generic "reflection" because it explicitly separates:

1. **observation** — evaluation results;
2. **local diagnosis** — batch-level failure analysis;
3. **synthesis** — cross-batch failure-mode consolidation;
4. **intervention** — candidate improvement;
5. **verification** — re-evaluation.

That is close to the synthesis loop EOKS has been developing:

```text
observe → diagnose → synthesize → intervene → verify
```

It also reinforces a key existing EOKS conclusion: **reflection is a strategy, not a correctness guarantee**. The reflective model is itself part of the computation and can be wrong; the candidate still needs evaluation and, where appropriate, external/deterministic verification.

## 6. Evaluation cost is part of the control problem

Opik distinguishes inner-loop evaluations from outer evaluations. Its optimization documentation explicitly discusses reducing evaluation variance with multiple samples and limiting inner-loop evaluation separately from the outer evaluation size.

The GEPA implementation also classifies mini-batch screening separately from full trials, and reflection calls are treated as real optimization activity rather than invisible free reasoning.

This provides concrete support for an EOKS control-economic question:

```text
expected value of another iteration
                 >
 cost of another iteration
```

The cost is multidimensional:

- model tokens;
- tool calls;
- evaluation calls;
- reflection calls;
- wall-clock latency;
- infrastructure cost;
- opportunity cost;
- risk of introducing regressions.

Therefore a stop policy should not be defined solely as "score stopped improving". It should consider improvement magnitude, uncertainty, cost and risk.

## 7. Multi-objective optimization is an EOKS signal

The optimizer package includes a `MultiMetricObjective` and Pareto-oriented candidate selection. This is important because the EOKS benchmark already recommends measuring multiple dimensions:

```text
correctness
quality
groundedness
verification
latency
context/tool cost
model cost
retries
regressions
```

A system can improve one metric while degrading another. For example:

```text
quality ↑
cost    ↑↑
latency ↑
```

may not be an improvement for the workload.

EOKS should therefore treat scalar scores as convenient evaluation outputs rather than assuming that one score is the universal control objective. Pareto-style reasoning is useful prior art for exposing trade-offs.

## 8. Tool optimization broadens the intervention space

Opik's optimizer is not limited to prompt text. Current documentation/code supports tool optimization and an optimizable-agent abstraction, with tool descriptions and parameter descriptions participating in optimization.

This aligns with EOKS's existing conclusion that context acquisition is itself a workload:

```text
raw exploration
semantic retrieval
structural graph
agentic search
precomputed analysis
context compilation
```

The intervention being optimized can therefore be:

```text
prompt
context strategy
tool choice
tool description
retrieval policy
model
reasoning strategy
workflow
```

The important EOKS principle is to evaluate these as interventions in the same workload matrix rather than treating one class as inherently superior.

## 9. Dataset identity and dependency identity

The Opik implementation pays attention to dataset-item identity. The GEPA adapter tracks exact training/validation item IDs and classifies evaluation calls based on those identities. Migration tests also preserve relationships such as `trace_id` and `dataset_item_id`.

This is not full provenance in the EOKS sense, but it is an important concrete precedent:

> **Evaluation evidence is only interpretable when we know what workload item and execution produced it.**

For EOKS, this should be extended rather than copied directly:

```text
result
 ├── workload identity/version
 ├── source state
 ├── context identity/version
 ├── model/version/config
 ├── tool/version/config
 ├── evaluator/version
 ├── execution trace
 └── supporting evidence
```

The research question then becomes which of these dependencies are semantically relevant to reuse and invalidation.

## 10. Where Opik stops and EOKS begins

Opik is primarily concerned with making AI application execution observable, evaluable and optimizable.

EOKS adds a different lifecycle question:

```text
execution
   ↓
 evidence
   ↓
 validated result
   ↓
 what should persist?
```

A result may become:

- a regression test;
- a knowledge artifact;
- a structural representation;
- a context fragment;
- a policy signal;
- a reusable computation;
- a hypothesis awaiting validation;
- or nothing durable.

Then, if it is persisted:

```text
persisted result
      ↓
 dependency tracking
      ↓
 source change
      ↓
 validity check
      ↓
 reuse OR invalidate/recompute
```

Opik provides useful evidence and infrastructure around the left-hand side. The right-hand side remains a core EOKS research problem.

## 11. Two interacting loops

The Opik comparison sharpens the existing EOKS two-loop hypothesis.

### Empirical improvement loop

```text
execute
   ↓
trace
   ↓
evaluate
   ↓
diagnose / reflect
   ↓
refine
   ↓
execute
```

Opik is strong prior art here.

### Knowledge lifecycle loop

```text
observe
   ↓
extract
   ↓
validate
   ↓
materialize
   ↓
relate
   ↓
reuse
   ↓
change detection
   ↓
invalidate / update / recompute
```

This is the broader EOKS research problem.

The loops interact:

```text
                 ┌───────────────────────┐
                 │  KNOWLEDGE LIFECYCLE  │
                 └───────────┬───────────┘
                             │
                     context / evidence
                             │
                             ▼
                 ┌───────────────────────┐
                 │ EMPIRICAL CONTROL LOOP│
                 └───────────┬───────────┘
                             │
                     traces / results
                             │
                             └──────────────→ knowledge
```

This is a synthesis hypothesis, not a claim that Opik implements EOKS.

## 12. Relationship to validated reusable computation

The current EOKS validated-reusable-computation synthesis asks how AI systems can preserve, validate, explain and selectively reuse computation as underlying state changes.

Opik adds a concrete empirical mechanism to that model:

```text
produce candidate
       ↓
evaluate
       ↓
record result
       ↓
reflect / diagnose
       ↓
revise
       ↓
re-evaluate
```

The combined model is:

```text
                       workload
                          │
                          ▼
                     computation
                          │
                          ▼
                  trace / representation
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
        evaluation    provenance   dependencies
             │
             ▼
          evidence
             │
       ┌─────┴─────┐
       ▼           ▼
    accepted     failure
       │           │
       ▼           ▼
 materialize    diagnose
       │           │
       │           ▼
       │        refine
       │           │
       └─────┬─────┘
             ▼
            reuse
             │
          change?
          /      \
        no        yes
        │          │
        ▼          ▼
      reuse    invalidate
                   │
                   ▼
                recompute
```

The key EOKS addition is the **materialize/reuse/invalidate boundary**. Evaluation does not by itself determine what should persist.

## 13. What this changes in the EOKS synthesis

The Opik evidence supports several refinements to the current synthesis:

### 13.1 Evaluation is not just scoring

Evaluation can produce:

```text
score
reason
failure mode
trajectory evidence
regression case
candidate-selection evidence
```

The downstream control system can use these outputs differently.

### 13.2 Reflection is not one operation

At least three distinct operations appear in Opik's reflective optimization:

```text
local failure analysis
cross-case synthesis
candidate generation
```

EOKS should not collapse these into a single "reflection" primitive.

### 13.3 Experiments are evidence-producing computations

An experiment has a workload, configuration, executions, evaluator and results. It can therefore become a provenance-bearing source for later synthesis.

### 13.4 Failures can become durable constraints

A production trace converted into a regression case is a concrete example of **validated experience becoming future executable knowledge**.

### 13.5 Control should optimize more than quality

Cost, latency, uncertainty and regression risk belong in the stop/continue decision.

## 14. What this does *not* establish

The Opik implementation does not establish that:

- reflection is always beneficial;
- GEPA or HRPO is the best optimizer;
- LLM-generated root causes are correct;
- multi-metric optimization solves EOKS policy selection;
- traces constitute sufficient provenance;
- regression tests constitute complete knowledge;
- experiment records are sufficient for safe computation reuse;
- cached AI computations can be safely reused without dependency-aware invalidation;
- an observability platform should become the EOKS control plane.

These remain empirical or architectural questions.

## 15. EOKS experiments suggested by Opik

### Experiment A — trace-derived learning

Compare:

```text
baseline
baseline + production-derived regression cases
baseline + unconstrained memory
baseline + validated regression + memory
```

Measure regression prevention, task success, maintenance cost and stale-case failures.

### Experiment B — reflection placement

Compare:

```text
outcome-only evaluation
case-level diagnosis
batch diagnosis + synthesis
process + outcome evaluation
```

Measure final quality, evaluation cost, false diagnosis and improvement per unit cost.

### Experiment C — multi-objective stopping

Compare a quality-only stop rule with a policy incorporating:

```text
quality improvement
cost
latency
uncertainty
regression risk
```

Measure cost per accepted outcome and failure rate.

### Experiment D — dependency-aware reuse

Persist an evaluated computation with its workload/configuration identity. Change one dependency at a time and measure whether the system can safely distinguish:

```text
still valid → reuse
stale       → recompute
partially affected → partial recomputation
```

### Experiment E — intervention optimization

Use the same workload matrix to optimize different intervention classes:

```text
prompt
context acquisition
retrieval
structural evidence
tool configuration
model selection
```

This tests whether the EOKS control plane should optimize *what the agent sees and how it works*, not only what prompt it receives.

## 16. Sources and implementation references

Primary project sources:

- Opik documentation: https://www.comet.com/docs/opik/
- Opik repository: https://github.com/comet-ml/opik
- Opik optimizer package: https://github.com/comet-ml/opik/tree/main/sdks/opik_optimizer
- GEPA integration: `sdks/opik_optimizer/src/opik_optimizer/algorithms/gepa_optimizer/`
- Hierarchical reflective optimizer: `sdks/opik_optimizer/src/opik_optimizer/algorithms/hierarchical_reflective_optimizer/`

Relevant implementation observations in the inspected revision:

- GEPA tracks dataset-item identity and distinguishes full validation trials from mini-batch screening; it records candidate metadata and per-item evaluation information.
- The hierarchical reflective optimizer performs parallel batch root-cause analysis followed by synthesis into unified failure modes, then uses those failure modes to generate and evaluate improvements.
- Opik's experiment model retains links between dataset items and traces and exposes evaluation/feedback metadata.
- The optimizer package supports multiple algorithms and multi-metric/Pareto-oriented selection.

Related academic prior art already tracked by EOKS:

- Madaan et al., **Self-Refine: Iterative Refinement with Self-Feedback**, NeurIPS 2023 — https://arxiv.org/abs/2303.17651
- Shinn et al., **Reflexion: Language Agents with Verbal Reinforcement Learning**, NeurIPS 2023 — https://arxiv.org/abs/2303.11366
- Gou et al., **CRITIC: Large Language Models Can Self-Correct with Tool-Interactive Critiquing**, ICLR 2024 — https://arxiv.org/abs/2305.11738
- Jia et al., **Do We Need to Verify Step by Step? Rethinking Process Supervision from a Theoretical Perspective**, ICML 2025 — https://proceedings.mlr.press/v267/jia25f.html

## Current EOKS position

Opik should be treated as **high-value prior art for the empirical evaluation/control substrate**, not as an EOKS replacement.

The strongest synthesis is:

> **Reliable AI work may require two coupled feedback systems: an empirical execution/evaluation loop that discovers and improves behavior, and a knowledge lifecycle that decides which validated results become durable, provenance-aware, dependency-aware state that can be reused or selectively recomputed.**

Opik gives us a concrete implementation of much of the first loop. EOKS's distinctive research problem begins where that loop meets durable knowledge, provenance, context compilation, dependency tracking and invalidation.
