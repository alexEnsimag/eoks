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

## Why a scheduler matters

The scheduler can make choices that are awkward to encode inside an individual agent:

- model selection;
- tool selection;
- context budget;
- parallelization;
- retry policy;
- verification requirements;
- escalation;
- cost/latency tradeoffs.

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
```

The exact routing policy should be learned/evaluated empirically.

## Verification as a resource

Verification should be schedulable. Some outputs need no additional verification; others justify tests, static analysis, another model or human review.

This makes verification part of the workload's risk policy.

## Escalation

Escalation is another control action, not necessarily failure. EOKS can detect that the current execution path has insufficient evidence and choose to:

- retrieve more context;
- invoke a stronger model;
- use a specialized tool;
- ask a human;
- split the task.

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
    -> learning record
    -> candidate skill / policy
    -> controlled rollout
    -> evaluation
```

This is deliberately different from simply retrieving old transcripts. The system is trying to determine **which behaviors worked in which situations**, and whether they should influence future execution.

Learned behavior should be versioned and evaluated. A pattern observed once is evidence, not automatically a policy. Human corrections, repeated successful outcomes and regression evaluations can provide stronger promotion signals.

See [Developer second brain and behavioral learning](developer-second-brain-and-behavioral-learning.md) for the fuller model.

## Limits of the analogy

Kubernetes manages relatively explicit resources and deterministic workloads. AI execution is probabilistic, semantic and open-ended. EOKS therefore cannot simply copy Kubernetes concepts one-for-one.

The useful borrowing is the **control-loop pattern**, not the API surface.