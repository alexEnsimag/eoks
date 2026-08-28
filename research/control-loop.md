# EOKS control loop

The Kubernetes analogy is useful because it suggests a control-loop architecture rather than a collection of agent utilities.

## Desired state

A workload starts with intent and constraints. EOKS derives a desired execution plan:

```text
intent
  -> task graph
  -> required capabilities
  -> resource candidates
  -> execution policy
```

The plan is not necessarily fixed. It can change when observations invalidate assumptions.

## Reconciliation

After each meaningful execution step, EOKS observes the actual state and compares it with the desired outcome.

```text
                 +----------------+
                 | desired state  |
                 +-------+--------+
                         |
                         v
                   scheduler
                         |
                         v
                  model / tools
                         |
                         v
                     outcome
                         |
                         v
                    evaluator
                         |
                         +-------> actual state
                                      |
                                      v
                                  reconcile
                                      |
                        +-------------+-------------+
                        |             |             |
                      continue      verify       escalate
                        |             |             |
                        +-------------+-------------+
                                      |
                                      v
                                   policy
```

The important addition for probabilistic AI workloads is that **evaluation produces control evidence**, not just a score for a dashboard. The evidence may include deterministic validation, source coverage, tool outcomes, agreement between attempts, and model-native uncertainty signals such as logprob-derived entropy where available.

## Why a scheduler matters

The scheduler can make choices that are awkward to encode inside an individual agent:

- model selection;
- tool selection;
- context budget;
- parallelization;
- retry policy;
- verification requirements;
- escalation;
- cost/latency tradeoffs;
- stop/continue policy;
- branch selection based on evaluation evidence.

This is the strongest part of the Kubernetes analogy: **policy and resource selection become infrastructure concerns.**

## Model routing

A router should not simply select the most capable model. It should optimize against workload requirements.

For example:

```text
simple deterministic task -> tool
small reasoning task       -> efficient model
complex coding task        -> stronger coding model
high-risk result           -> model + verification
ambiguous task             -> information gathering
high uncertainty           -> retrieve / verify / branch / escalate
```

The exact routing policy should be learned/evaluated empirically.

## Reliability signals as control inputs

EOKS should distinguish **model uncertainty**, **evidence strength** and **probability of correctness**. They may be correlated but are not interchangeable.

Where providers expose token probabilities, candidate signals include:

- mean or length-normalized log probability;
- token-level predictive entropy;
- top-token margin;
- uncertainty on critical fields or tool arguments;
- entropy trajectories during reasoning.

Where token probabilities are unavailable, alternatives include sampled semantic entropy, semantic agreement/self-consistency, external validators and evidence coverage.

Raw signals should not automatically be interpreted as calibrated probabilities of correctness. Calibration requires representative outcomes for the workload and the decision being controlled.

See [LLM uncertainty, semantic entropy and control](llm-uncertainty-and-control.md).

## Stop/continue decisions

An agent graph should not rely only on fixed iteration counts when a measurable termination criterion is available.

A node should define:

```text
goal
  -> evidence
  -> evaluation
  -> acceptance criteria
  -> continuation policy
  -> budget / safety limit
```

Possible stop conditions include:

```text
validator passes
required evidence coverage reached
calibrated uncertainty is below threshold
independent attempts converge
no new relevant evidence is discovered
marginal information gain falls below ε
expected value of another step is below its cost
```

These are workload-specific policies, not universal thresholds. A useful research direction is to estimate the expected value of another action from historical outcomes and compare it with cost/latency.

## Graph branching

The same evaluation state can select different execution paths:

```text
                    evaluation
                        |
          +-------------+-------------+
          |             |             |
       sufficient     uncertain      failed
          |             |             |
         stop       retrieve       repair/replan
                        |
                    evaluate again
```

This is the graph-oriented form of reconciliation. The graph is not merely a fixed chain of prompts; it is a policy over state transitions.

## Verification as a resource

Verification should be schedulable. Some outputs need no additional verification; others justify tests, static analysis, another model or human review.

This makes verification part of the workload's risk policy.

## Escalation

Escalation is another control action, not necessarily failure. EOKS can detect that the current execution path has insufficient evidence and choose to:

- retrieve more context;
- invoke a stronger model;
- use a specialized tool;
- ask a human;
- split the task;
- change the representation or reasoning strategy.

## Policy learning

Outcomes can improve future routing:

```text
(task characteristics, context, model, tools)
                       |
                    outcome
                       |
                    evidence
                       |
                routing policy
```

The policy should be versioned and evaluated like any other production dependency.

## Learning from development sessions

The control loop can also learn from the developer or from previous agent executions. A coding session should produce a structured trace containing plans, observations, tool calls, edits, failures, corrections, verification and outcome.

A separate learning process can transform traces into candidate procedural knowledge:

```text
session trace
    -> episode
    -> recurring pattern
    -> validation
    -> Learning Record
    -> candidate skill / policy
    -> controlled rollout
    -> evaluation
```

This is deliberately different from simply retrieving old transcripts. The system is trying to determine **which behaviors worked in which situations**, and whether they should influence future execution.

Learned behavior should be versioned and evaluated. A pattern observed once is evidence, not automatically a policy. Human corrections, repeated successful outcomes and regression evaluations can provide stronger promotion signals.

See [Behavioral memory and learning how developers work](../docs/behavioral-memory.md) and [Learning from development sessions](session-learning.md).

## Learning records and control

A Learning Record should retain situation, action/strategy, evidence, outcome, evaluation, provenance, confidence, scope/validity and status. This allows EOKS to distinguish a personal preference from a project constraint or a generally useful engineering procedure.

The learning plane should propose changes to skills and policies; it should not silently mutate execution policy after every session. Promotion remains controlled and reversible.

## Limits of the analogy

Kubernetes manages relatively explicit resources and deterministic workloads. AI execution is probabilistic, semantic and open-ended. EOKS therefore cannot simply copy Kubernetes concepts one-for-one.

The useful borrowing is the **control-loop pattern**, not the API surface.
