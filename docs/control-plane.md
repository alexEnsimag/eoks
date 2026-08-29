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
- reliability history;
- temporal and concurrency constraints.

This is analogous to Kubernetes scheduling, but the resources are semantic and probabilistic.

## Workload state and model

The control plane should maintain enough explicit workload state to make execution decisions and resume/audit runs. This is distinct from the context supplied to a model.

A useful conceptual model is:

```text
resources / evidence
        |
        v
   workload state
  /      |       \
state  history  beliefs
  \      |       /
       policies
          |
   context compilation
          |
       reasoning
          |
   proposed action/plan
          |
       execution
          |
      observation
          +------> update / verify / re-plan
```

The state model may remain lightweight for simple workloads. It can become richer when history, uncertainty, temporal constraints or multi-agent coordination materially affect decisions. The LLM is a reasoning component; it should not silently become the canonical owner of durable state.

This model-based planning/execution perspective is informed by recent work from Ronen Brafman and collaborators; see [Ronen Brafman prior art](../research/prior-art/ronen-brafman-agent-architecture.md).

## Reconciliation

The control plane should observe execution rather than assume success. A task can move through states such as:

`planned -> contextualized -> executing -> verifying -> completed | retry | branch | escalated`

Verification can trigger a different model, additional retrieval, a deterministic tool, human review or re-planning. A plan is an executable hypothesis: observations or failures can invalidate assumptions and require a new decision.

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

## Temporal and concurrent execution

Some actions are long-running, stochastic or concurrent. The control plane should be able to represent activity state and react to observations rather than assuming that a plan is a sequence of instantaneous tool calls.

```text
start activity -> observe -> update state -> verify
                         |                |
                         +--> continue --+
                         +--> retry / branch / re-plan / escalate
```

Scheduling should be dependency-aware and should optimize for reliable completion rather than maximal parallelism. Durable activity state is particularly important when external systems, CI, deployments or scans can outlive an individual model invocation.

## Policies

Policies should be explicit and inspectable. Examples include maximum cost, required verification for risky changes, allowed tools, model constraints, context sources and persistence rules. Policies can also define when a workload must stop for human input—for example, ambiguous requirements, destructive operations, security-sensitive changes, insufficient validation evidence or production-impacting changes.

Capability/action metadata can make some policies enforceable before execution. Where available, an action can declare inputs, preconditions, effects, duration, side effects, permissions and validation requirements. EOKS should adopt this incrementally; a formal planning language is not required for every resource.

## Model selection

Model selection is a scheduling problem. The best model is not necessarily the strongest model; it is the model that provides the required quality for the task at acceptable cost and latency.

The same principle applies to agent topology: the best topology is not necessarily the largest one. A single strong agent may be preferable to several agents when the work is sequential; a focused subagent can be preferable when a side task would otherwise pollute the main context; and a coordinated team can be justified when independent workers need to communicate.

## Multi-agent state

For multi-agent workloads, the control plane should distinguish shared state from agent-local observations and beliefs. Communication and signaling can be explicit workflow actions, and provenance should preserve who observed, inferred or communicated a piece of information.

```text
shared state
   + agent-local state/belief
   + communication history
   + coordination state
```

This prevents an uncertain local belief from silently becoming canonical shared knowledge merely because it appeared in an agent message.

## Why a control plane?

Without a control layer, individual agents independently reinvent retrieval, memory, tool choice and evaluation. This creates duplicated infrastructure and makes behavior difficult to observe or optimize globally.
