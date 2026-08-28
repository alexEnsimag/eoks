# Control plane

The EOKS control plane is the proposed coordinating layer for AI workloads.

## Scheduler

A scheduler can classify tasks and select an execution strategy based on requirements such as:

- reasoning complexity;
- context requirements;
- deterministic-analysis availability;
- latency/cost constraints;
- required tools;
- model capabilities;
- reliability history.

This is analogous to Kubernetes scheduling, but the resources are semantic and probabilistic.

## Reconciliation

The control plane should observe execution rather than assume success. A task can move through states such as:

`planned -> contextualized -> executing -> verifying -> completed | retry | escalated`

Verification can trigger a different model, additional retrieval, a deterministic tool, or human review.

## Policies

Policies should be explicit and inspectable. Examples include maximum cost, required verification for risky changes, allowed tools, model constraints, context sources and persistence rules.

## Model selection

Model selection is a scheduling problem. The best model is not necessarily the strongest model; it is the model that provides the required quality for the task at acceptable cost and latency.

## Why a control plane?

Without a control layer, individual agents independently reinvent retrieval, memory, tool choice and evaluation. This creates duplicated infrastructure and makes behavior difficult to observe or optimize globally.
