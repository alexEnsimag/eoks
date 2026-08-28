# EOKS core model

This note goes one level deeper than the initial architecture. The goal is to identify the objects EOKS may actually need to coordinate.

## The central shift

The early framing was “how should an agent receive context?” The stronger framing is:

> How should an AI system transform an intent into a reliable outcome while managing information, computation, models, tools and uncertainty?

That makes the unit of EOKS a **workload**, not a prompt.

## Workload

A workload is an intent plus constraints and an expected outcome. It may be a one-shot task, a multi-step workflow or a recurring process.

Conceptually:

```text
Workload
  intent
  inputs
  constraints
  required capabilities
  quality target
  risk tolerance
  cost/latency budget
  deadline
  authorization
```

The workload does not necessarily specify which model or tools to use. Those are resources selected by policy.

## Task

A workload can be decomposed into tasks. A task should have:

- objective;
- dependencies;
- required evidence/capabilities;
- context requirements;
- execution policy;
- completion criteria;
- evaluation criteria.

This allows the scheduler to reason about a task without knowing its eventual implementation.

## Resources

Potential EOKS resources include:

| Resource | Meaning |
| --- | --- |
| Context | Current working information supplied to execution |
| Memory | Persistent experience/knowledge available for retrieval |
| Evidence | Observations establishing facts or supporting claims |
| Model | Reasoning/execution capability |
| Tool | External deterministic or specialized capability |
| Agent | Stateful execution strategy |
| Compute | Execution capacity |
| Budget | Cost/latency/token constraints |
| Policy | Rules governing choices |

The list is deliberately provisional. Some may eventually collapse into one abstraction.

## Claims versus evidence

An important distinction is between a statement produced by a model and evidence supporting that statement.

```text
claim
  |
  +-- source evidence
  +-- tool evidence
  +-- test result
  +-- prior knowledge
  +-- model inference
```

This matters for confidence, memory and context selection. A claim without provenance should not automatically acquire the authority of a verified fact.

## Context package

A context package is a task-specific working set derived from persistent resources.

```text
ContextPackage
  task_id
  blocks
  ordering
  provenance
  exclusions
  budget
  selection_policy
  compiler_version
```

The package is a useful boundary because it makes context reproducible. Two model runs can then be compared with the same context while changing only the model, or vice versa.

## Execution

Execution is not necessarily one model call:

```text
Task
 |
 +--> retrieve
 +--> analyze
 +--> reason
 +--> call tool
 +--> verify
 +--> revise
 +--> produce outcome
```

The control plane decides whether additional steps are justified.

## Outcome

An outcome should contain both the produced result and its evaluation:

```text
Outcome
  artifact
  execution trace
  evidence
  evaluation
  confidence/risk signals
  cost
  latency
  failures
```

This turns execution into something the system can learn from.

## The feedback loop

The minimum conceptual EOKS loop is:

```text
intent
  -> plan
  -> select resources
  -> compile context
  -> execute
  -> observe
  -> evaluate
  -> accept / retry / escalate
  -> update memory and policy
```

This is the architectural heart of the project. The other subsystems exist to make this loop reliable and efficient.

## What EOKS is not yet

This model does not imply that EOKS needs a distributed control plane, microservices, a graph database, a vector database, a special prompt format or a new agent framework. Those are implementation hypotheses that should follow evidence.