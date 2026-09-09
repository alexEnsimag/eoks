# Agent trajectory evaluation

## Why this matters to EOKS

Recent agent-evaluation work reinforces an existing EOKS premise: **an outcome does not fully characterize the computation that produced it**. An agent can reach the correct result through a fragile or faulty trajectory, while another can take a recoverable detour and still complete the task robustly. Evaluation therefore needs enough execution evidence to distinguish outcome quality from process quality and to attribute failures.

This note records the relevant prior art without turning any particular evaluation framework into an EOKS abstraction.

## Agent Evaluation: How Do You Know Your Agent Actually Works?

Sai Bhargav Rallapalli's 2026 article proposes three evaluation scopes:

1. **End-to-end/task** — whether the intended task was actually completed against ground truth.
2. **Trajectory** — whether the sequence of actions was sound, efficient and stable, including unnecessary loops, wrong tool calls and recovery behavior.
3. **Component** — which retriever, tool, router or other component contributed to a failure.

It also distinguishes deterministic checks from judgment-based evaluation and offline evaluation from sampled online evaluation. The useful EOKS contribution is not the particular three-level taxonomy or sampling recommendation; it is the observation that **final-output evaluation can hide materially different execution paths**.

Source: [Agent Evaluation: How Do You Know Your Agent Actually Works?](https://blog.gopenai.com/agent-evaluation-how-do-you-know-your-agent-actually-works-1c6b7cef5461)

## Agentic Evals: A Software Engineer's Perspective

Yesha's *Agentic Evals: A Software Engineer's Perspective* is useful because it frames evaluation as an engineering feedback loop rather than as a leaderboard exercise. The article emphasizes evaluating the **trajectory and execution behavior** of an agent, not only the final answer, and combining deterministic checks with judgment where deterministic verification is insufficient.

A particularly useful pattern is:

```text
eval set
   -> environment / task
   -> agent execution
   -> trace / trajectory
   -> evaluators
   -> metrics / failures
   -> new or refined eval cases
   -> next evaluation cycle
```

For EOKS, the important synthesis is that the evaluation corpus itself becomes a governed resource. Production failures and newly discovered edge cases can become candidate evaluation cases, but should be validated and promoted rather than silently changing the canonical benchmark. This connects evaluation to the existing EOKS candidate-extraction and controlled-promotion model.

The article also highlights repeated-run reliability rather than treating one successful execution as sufficient evidence. Concepts such as **pass@k** and **pass^k** are useful examples of different reliability questions: whether at least one attempt succeeds versus whether repeated attempts succeed consistently. EOKS should retain the underlying distribution and workload slice rather than collapsing these into a universal reliability score.

The article's strongest architectural implication is therefore:

```text
run
  -> trace / intermediate evidence
  -> evaluation
  -> outcome
  -> policy evidence
  -> control decision
       |-> stop
       |-> verify
       |-> retry / replan
       |-> branch
       |-> switch resource/model
       `-> create candidate eval case
```

This is consistent with EOKS's existing separation of **observability**, **reliability estimation**, and **control**. Evaluation should produce evidence that can inform the control loop; it should not itself become an opaque control policy.

Source: [Agentic Evals: A Software Engineer's Perspective](https://yeshas93.substack.com/p/agentic-evals-a-software-engineers)

### What this adds to the synthesis

This article does **not** justify a new EOKS `trajectory` primitive or a generic agent-evaluation subsystem. The existing execution trace and evaluation record are sufficient representations.

It does strengthen four existing hypotheses:

- **Trajectory evidence is first-class evaluation evidence.** Final-output success can hide inefficient, fragile or silently faulty execution.
- **Evaluation is part of the control loop.** Evaluation results can become evidence for stop/verify/retry/branch/resource-selection decisions, subject to policy and calibration.
- **Eval sets are governed knowledge/evidence resources.** They need provenance, versioning, task coverage, freshness and controlled promotion of production-derived cases.
- **Reliability is workload-specific and distributional.** Repeated-run behavior matters; a single aggregate score is insufficient for graduated autonomy.

These points reinforce the broader EOKS synthesis: **knowledge, experience, context, execution evidence and evaluation are different roles in one closed loop**. The architecture should coordinate them without prematurely turning each role into a mandatory subsystem.

## Trajectory-Judge

*Trajectory-Judge: What Outcome-Only LLM Judges Miss on Agent Trajectories* directly studies the gap between evaluating the final outcome and evaluating the trajectory. The authors construct cases in which faults are visible in the outcome versus silently masked by later recovery. Their reported experiments show substantially lower recall for silent trajectory faults when judging outcomes alone, while step-level evaluation improves detection.

This is especially relevant to EOKS because it provides empirical support for retaining **execution/trajectory evidence alongside outcomes**. The exact benchmark and reported percentages should remain research evidence rather than EOKS constants.

Source: [arXiv:2609.00038](https://arxiv.org/abs/2609.00038)

## Stochasticity in agent evaluation

*Stochasticity in Agentic Evaluations: Quantifying Inconsistency with Intraclass Correlation* argues that a single agent-evaluation score can obscure run-to-run variance. The associated open-source work explores statistically rigorous evaluation of stochastic agent behavior.

This complements EOKS's existing emphasis on calibration and controlled comparison: an observed improvement should be distinguished from sampling variance where the workload and agent are stochastic.

Source: [Stochasticity in Agentic Evaluations](https://arxiv.org/abs/2512.07010)

Implementation/reference: [stochastic-agent-evals](https://github.com/youdotcom-oss/stochastic-agent-evals)

## Agentic evaluation as trajectory capture

Google Cloud's EvalBench provides a useful implementation example: multi-turn scenarios capture text, tool calls, parameters, latency and tokens, then score the resulting trajectory with separate trajectory, turn-count, latency, token-consumption, goal-completion and behavioral metrics.

The important EOKS observation is that **evaluation requires an execution record rich enough to support the construct being measured**. This fits the existing EOKS evaluation record rather than requiring a new trajectory object.

Source: [EvalBench agentic evaluations](https://github.com/GoogleCloudPlatform/evalbench/blob/main/docs/agentic-evals.md)

## LLM-as-judge reliability

The broader LLM-as-judge literature documents systematic biases including position bias and self-preference. These findings support an existing EOKS rule: evaluator outputs are evidence and should be validated/calibrated against suitable ground truth or independent evidence rather than treated as unquestionable truth.

Examples:

- [Position bias in LLM-as-a-Judge](https://aclanthology.org/2025.ijcnlp-long.18.pdf)
- [LLM-as-a-Judge: Self-Preference Bias in LLM Evaluations](https://arxiv.org/abs/2410.21819)

These are supporting references, not additional EOKS primitives.

## Implications for EOKS

The evidence supports the following existing concepts:

```text
task
  -> configuration/context
  -> execution trace
  -> intermediate evidence
  -> outcome
  -> evaluation
  -> policy / reconciliation
```

Evaluation can therefore operate at different scopes—unit, component, task, workflow and system—while preserving enough trace and outcome information to attribute effects. This is already reflected in [Evaluation](../../docs/evaluation.md).

The more interesting open question is whether evaluation evidence can become an **active control signal**:

```text
execute
   -> observe
   -> evaluate
   -> continue / verify / branch / reuse / stop
```

That should remain a research question. The cited work supports the value of richer execution evidence; it does not establish a universal control policy.

## What should *not* be imported

- No mandatory three-level evaluation architecture.
- No universal trajectory score.
- No fixed online sampling percentage.
- No assumption that an LLM judge is ground truth.
- No new EOKS ontology object merely for "trajectory" when the existing execution trace and evaluation record are sufficient.
- No replacement of deterministic evaluation principles already established elsewhere in EOKS.
