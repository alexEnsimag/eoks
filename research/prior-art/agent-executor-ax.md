# Google AX / Agent Executor

## Why this matters to EOKS

[Google AX](https://agentexecutor.io/) is an open, declarative orchestration runtime for agent workloads. It treats agents as a distinct workload class: stateful, isolated, bursty, long-running processes that frequently wait on model APIs, tools or humans. AX currently exposes four small primitives—**Task, Workspace, Gateway and Model**—and runs on top of **Agent Substrate**, a runtime for sandboxed stateful actor execution. The repository is explicitly pre-stable and warns that its core concepts, protocols and specifications may change substantially before a stable release.

The EOKS-relevant question is not whether AX should become an EOKS dependency. It is whether AX provides useful evidence for the **execution layer** and for the boundary between environment composition and durable execution.

## AX's model

The public architecture is deliberately Kubernetes-like:

- **Task** — isolated agent execution with resource limits and a lifecycle that can be suspended/resumed.
- **Workspace** — declarative environment composition around Git repositories, MCP servers, skills and an optional natural-language environment goal.
- **Gateway** — network policy and credential plumbing.
- **Model** — model configuration and credentials.
- **Agent Substrate** — the lower runtime that provides sandboxed execution, stateful actors, checkpointing/resumption and dense resource sharing.

A task can reference one or more workspaces, which makes the Workspace closer to a reusable environment/loadout than to a simple working directory.

The lifecycle is also materially different from a conventional batch job:

```text
create
  -> running
  -> checkpoint / suspend
  -> resume
  -> running
  -> terminal / delete
```

AX's public examples show state surviving suspension, while its architecture describes lightweight actors that can be checkpointed when waiting on model, tool or human responses.

## Relationship to the EOKS model

AX maps most directly to the lower part of the current EOKS architecture:

```text
                 EOKS control plane
        task / policy / context / evaluation
                         |
                  environment/loadout
                         |
              +----------+----------+
              |                     |
       context resources       execution substrate
       knowledge/evidence       Task / runner
              |                     |
              +----------+----------+
                         |
                    agent + tools
                         |
                      outcome
```

This suggests a useful separation:

| Concern | AX | EOKS hypothesis |
|---|---|---|
| Isolated execution | Task | execution primitive / workload |
| Environment composition | Workspace | environment/loadout + eligible resources |
| Network/capability boundary | Gateway | policy/capability layer |
| Model configuration | Model | model resource/policy |
| Durable execution lifecycle | Agent Substrate + Task | execution substrate |
| Context selection | not the primary abstraction | context compiler |
| Durable knowledge | not the primary abstraction | knowledge/evidence plane |
| Workflow/reasoning strategy | intentionally outside the core runtime | workflow + reasoning resources |
| Evaluation and outcome feedback | supports agent workloads/research, but not the EOKS control loop | evaluation → outcome → feedback |

The important conclusion is therefore **composition rather than replacement**. AX is useful prior art for an execution substrate, while EOKS is exploring the control loop around resources, context, workflows, evaluation and graduated autonomy.

## Particularly relevant ideas

### 1. Execution as a first-class workload

AX strengthens the case that agent execution should not be modeled simply as a function call or a stateless service. A durable agent may spend substantial time waiting, accumulate state, require isolation and later resume.

For EOKS this makes the execution primitive worth keeping explicit rather than hiding it inside a workflow abstraction.

### 2. Environment composition

The Workspace model is close to the EOKS idea of **composing an environment from small primitives**. Git, MCP, skills and environment preparation can be declared independently and then attached to a Task.

This is useful evidence for keeping environment composition separate from the agent itself.

### 3. Durable suspend/resume

Checkpointed execution raises a deeper EOKS question: what is the durable unit?

Possible boundaries include:

- session;
- workflow;
- task;
- model turn;
- tool execution.

AX strongly favors a durable **task/actor** abstraction while allowing individual model/tool waits to become cheap suspension points. This is worth comparing with EOKS's workflow and run model rather than adopting directly.

### 4. Runner boundary

AX exposes a runner contract and allows the task container/runtime to be replaced. This is especially interesting for EOKS because it suggests an execution backend can remain separate from higher-level workflow semantics.

A future EOKS execution abstraction could potentially target different backends—local coding-agent sessions, containers, Kubernetes-based execution, or AX—without making any one runtime the semantic center of the project.

### 5. Generative environment preparation

AX can use an agent to turn a natural-language workspace goal into a prepared environment. This is interesting but should be treated carefully: using an agent to construct the environment introduces another agentic step whose result needs validation.

For EOKS, this is better viewed as a possible **environment materialization workflow** than as a primitive that should automatically become trusted state.

## What AX does not appear to solve

Based on the public architecture, AX is not attempting to be:

- a durable engineering knowledge system;
- a context compiler;
- a provenance/governance layer for heterogeneous evidence;
- a repository knowledge or structural-analysis system;
- a workflow evaluation/control loop;
- a system for promoting session experience into canonical knowledge;
- a graduated-autonomy policy engine.

Those remain separate EOKS concerns.

## Research questions

1. **What is the right durable execution unit?** Compare AX's Task/actor with EOKS Run, workflow and session concepts.
2. **Where should environment composition end and context compilation begin?** A Workspace can specify repositories, MCP and skills, but EOKS also needs task-specific evidence selection.
3. **What belongs in an execution backend contract?** Identify the minimum interface needed for local, containerized and AX-like durable execution.
4. **How should suspension interact with workflow state?** Distinguish runtime checkpointing from durable workflow/artifact state.
5. **Can execution evidence feed the EOKS control loop?** AX can expose lifecycle and runtime information; EOKS could combine this with tests, traces, outcomes and model signals.
6. **What are the economics of dense agent execution?** Validate claims around suspension, multiplexing and resource utilization against real workloads rather than treating scale claims as architectural proof.
7. **What is the security boundary?** Compare AX's Task/Gateway isolation with the capability/policy model EOKS needs for consequential tools.

## Proposed EOKS interpretation

AX should be treated as **execution-substrate prior art**, not as another tool to install in the EOKS Phase-A stack.

The useful hypothesis is:

> EOKS may define the semantics around agent execution—resource selection, environment/loadout, context compilation, workflow, policy, evaluation and feedback—while delegating isolated durable execution to a backend such as a local harness, container runtime or AX/Agent Substrate.

This preserves the project's current preference for a small semantic model and integration/sidecar architecture while leaving room for a stronger execution substrate when experiments demonstrate that durable isolated execution is necessary.

## Sources

- [AX / Agent Executor](https://agentexecutor.io/)
- [google/ax](https://github.com/google/ax)
- AX repository documentation: Concepts, Manifests, Sandbox, Runners, Networking and Architecture
- Agent Substrate, linked from the AX project as its underlying execution runtime

This note records **architectural prior art and hypotheses**, not a product-quality assessment. AX is currently marked as pre-stable by its maintainers.
