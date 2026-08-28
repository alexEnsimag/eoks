# Control-plane research

The control-plane hypothesis is one part of the broader **AI OS** framing. The useful claim is not that EOKS should reproduce Kubernetes, but that AI workloads may benefit from a layer that coordinates tasks, context, resources, evaluation and feedback above individual agents and models.

For the broader comparison across Kubernetes, operating systems, senior-developer onboarding, compilers/build systems and SRE feedback loops, see [`ai-os-analogies.md`](ai-os-analogies.md).

## The Kubernetes analogy

One of the strongest architectural questions was whether an AI system needs something analogous to a Kubernetes control plane.

The analogy is useful if it is treated as an architectural pattern rather than a literal implementation prescription:

```text
Kubernetes
  desired state -> scheduler -> controllers -> workloads -> observations -> reconciliation

EOKS
  task intent -> planner/scheduler -> model/tools -> execution -> evaluation -> memory/policy update
```

Kubernetes scheduling is specifically about matching Pods to Nodes according to requirements and available resources. The EOKS analogy generalizes that idea: a workload declares intent and constraints, and the control plane chooses an execution strategy. citeturn0search1turn0search3

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

This is deliberately different from making **Agent** the fundamental abstraction. An agent is an execution mechanism; a workload is the thing the control plane is responsible for bringing to an acceptable outcome.

A candidate workload declaration is:

```text
objective
+ constraints
+ required capabilities
+ context requirements
+ assurance / evaluation criteria
+ execution budget
```

## Scheduler

The scheduler idea became more interesting than simply “choose an LLM.” A scheduler could consider:

- task type;
- urgency;
- expected difficulty;
- context size and quality requirements;
- available models;
- model capabilities;
- deterministic analyzers and other workers;
- cost/latency budget;
- tool requirements and permissions;
- reliability requirements;
- historical performance;
- evaluation results;
- evidence freshness and locality.

This suggests a future where “which model?” is one scheduling decision among several, rather than hard-coded application logic.

In other words, model routing is a **subset** of workload scheduling. The scheduler may choose a model, but it may also choose context providers, static analyzers, tools, verification steps and an execution graph.

## Model selection

We discussed the practical problem of switching between model generations. A newer model can behave differently enough that a naive replacement is unsafe. Evaluation must therefore be part of the switching mechanism.

A possible policy loop is:

```text
workload
   -> candidate resources / models
   -> representative evaluation set
   -> choose model/policy
   -> production execution
   -> observe outcome
   -> update evidence
```

This is analogous to changing a production dependency: model changes should be evaluated as behavioral changes, not treated as a string replacement in configuration.

## Reconciliation

The Kubernetes analogy becomes particularly valuable around reconciliation. EOKS could maintain a desired execution state and continuously compare it with observed state:

- Is the task progressing?
- Is the model producing useful evidence?
- Is context becoming polluted or incomplete?
- Has a tool failed repeatedly?
- Should the task be decomposed?
- Should another model be invoked?
- Is additional evidence required?
- Should the workflow branch, retry or escalate?

This points toward a control loop rather than a single agent invocation.

```text
desired outcome
      |
   plan/execute
      |
    evaluate
      |
   +--+----------------+
   |                   |
 acceptable        insufficient
   |                evidence/result
  done                  |
                  reconcile/replan
```

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
                 EOKS control plane
                        |
        +---------------+---------------+
        |               |               |
      Agents          Models          Tools
        |               |               |
        +---------------+---------------+
                        |
                     workload
```

The control plane owns coordination and feedback; agents, models, analyzers and tools are execution resources.

This does **not** require EOKS to replace existing agents. Existing coding-agent CLIs, MCP servers and specialist analyzers can remain the execution layer while EOKS prepares context, selects resources, observes runs and evaluates outcomes.

## Open architectural question

The biggest unresolved issue is whether this abstraction genuinely provides value over a well-designed agent runtime. The answer should come from workload-level experiments, especially those involving:

- model switching;
- context management;
- repository-discovery cost across sessions;
- evaluation and verification;
- adaptive replanning;
- recurring tasks;
- durable knowledge and memory.

The analogies are therefore **hypothesis-generating tools**, not evidence that a full AI operating system is necessarily required.
