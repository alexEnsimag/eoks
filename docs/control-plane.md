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

## Orchestration

Agent orchestration is a concrete execution/control function of reconciliation. The control plane should coordinate the smallest execution topology that satisfies the workload rather than assuming that every task requires multiple agents.

A minimal software-engineering topology is:

```text
              workload
                  |
              conductor
             /    |    \
        context executor reviewer
             \    |    /
              verification
                  |
          complete / retry / escalate
```

The **conductor** owns workflow state, task decomposition, handoffs and escalation. The **executor** performs implementation or other side effects. The **reviewer** independently challenges the result. Verification supplies deterministic and behavioral evidence. These roles may be separate sessions, subagents, or sequential phases of one agent; the semantic roles are more important than the runtime topology.

The conductor should not become another knowledge base. It requests context from the context plane, resources from the resource model and evidence from evaluation/observability. It records workflow state and provenance so that runs can be resumed and audited.

Parallel agents are justified when tasks are independent or benefit from competing hypotheses or independent review. Sequential execution is preferable when steps share context, have tight dependencies or modify overlapping files. Isolation should be used when concurrent workers can interfere with one another.

See [Agent orchestration](agent-orchestration.md) for the detailed model and the mapping to concrete coding-agent runtimes.

## Policies

Policies should be explicit and inspectable. Examples include maximum cost, required verification for risky changes, allowed tools, model constraints, context sources and persistence rules. Policies can also define when a workload must stop for human input—for example, ambiguous requirements, destructive operations, security-sensitive changes, insufficient validation evidence or production-impacting changes.

## Model selection

Model selection is a scheduling problem. The best model is not necessarily the strongest model; it is the model that provides the required quality for the task at acceptable cost and latency.

The same principle applies to agent topology: the best topology is not necessarily the largest one. A single strong agent may be preferable to several agents when the work is sequential; a focused subagent can be preferable when a side task would otherwise pollute the main context; and a coordinated team can be justified when independent workers need to communicate.

## Why a control plane?

Without a control layer, individual agents independently reinvent retrieval, memory, tool choice and evaluation. This creates duplicated infrastructure and makes behavior difficult to observe or optimize globally.
