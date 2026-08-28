# Control-plane research

## The Kubernetes analogy

One of the strongest architectural questions was whether an AI system needs something analogous to a Kubernetes control plane.

The analogy is useful if it is treated as an architectural pattern rather than a literal implementation prescription:

```text
Kubernetes
  desired state -> scheduler -> controllers -> workloads -> observations -> reconciliation

EOKS
  task intent -> planner/scheduler -> model/tools -> execution -> evaluation -> memory/policy update
```

The proposed EOKS control plane would coordinate AI workloads rather than merely run containers.

## What is an AI workload?

A workload may be:

- a coding task;
- repository analysis;
- incident investigation;
- research task;
- document generation;
- evaluation run;
- an agent workflow;
- a recurring autonomous process.

The workload should declare intent and constraints. EOKS decides how to execute it.

## Scheduler

The scheduler idea became more interesting than simply “choose an LLM.” A scheduler could consider:

- task type;
- urgency;
- expected difficulty;
- context size;
- available models;
- model capabilities;
- cost/latency budget;
- tool requirements;
- reliability requirements;
- historical performance;
- evaluation results.

This suggests a future where “which model?” is a scheduling decision, not hard-coded application logic.

## Model selection

We discussed the practical problem of switching between model generations. A newer model can behave differently enough that a naive replacement is unsafe. Evaluation must therefore be part of the switching mechanism.

A possible policy loop is:

```text
workload
   -> candidate models
   -> representative evaluation set
   -> choose model/policy
   -> production execution
   -> observe outcome
   -> update evidence
```

This is analogous to deployment control: model changes should be treated as changes to a production dependency, not as a string replacement in configuration.

## Reconciliation

The Kubernetes analogy becomes particularly valuable around reconciliation. EOKS could maintain a desired execution state and continuously compare it with observed state:

- Is the task progressing?
- Is the model producing useful evidence?
- Is context becoming polluted?
- Has a tool failed repeatedly?
- Should the task be decomposed?
- Should another model be invoked?
- Is additional evidence required?

This points toward a control loop rather than a single agent invocation.

## Policies

A control plane needs explicit policies for:

- model selection;
- context budgets;
- tool permissions;
- retries;
- escalation;
- evaluation thresholds;
- memory promotion;
- human approval;
- cost and latency.

The important idea is to separate policy from individual agents so that behavior can be changed globally and evaluated systematically.

## Why this is more than an agent framework

An agent framework usually focuses on making one agent execute tools or follow a workflow. EOKS is hypothesized to operate one level above that abstraction:

```text
                 EOKS
                   |
        +----------+----------+
        |          |          |
      Agent      Model       Tools
        |          |          |
        +----------+----------+
                   |
                workload
```

The control plane owns the coordination and feedback loop; agents and models are execution resources.

## Open architectural question

The biggest unresolved issue is whether this abstraction genuinely provides value over a well-designed agent runtime. The answer should come from workload-level experiments, especially those involving model switching, context management, evaluation and recurring tasks.