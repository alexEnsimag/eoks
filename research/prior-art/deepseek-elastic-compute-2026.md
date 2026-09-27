# DeepSeek Elastic Compute (DSec) and the EOKS execution plane

## Why this matters

DeepSeek's **DeepSeek Elastic Compute (DSec)** paper (arXiv:2609.22978, September 2026) provides unusually concrete production evidence for the execution side of agentic systems.

DSec is not a new EOKS abstraction and should not be treated as one. Its value to EOKS is as **execution-substrate prior art**: it shows what a stateful, elastic, heterogeneous execution plane looks like when agent environments become a large-scale infrastructure workload.

The paper reports a production-scale unit of roughly 160 CPU nodes, about 30,000 CPU cores and ~250 TB DRAM, serving about 3 million sandboxes/day, more than 380,000 concurrent sandboxes and more than 5,000 sandbox creations/sec. A single training/evaluation job can request up to 32K sandboxes. The exact scale is not the architectural conclusion; it demonstrates that agent execution can become a fleet-management problem rather than a single-runtime problem. https://arxiv.org/abs/2609.22978

## DSec's central model

DSec exposes multiple sandbox backends through a unified SDK:

- function-call execution;
- containers;
- microVMs;
- full VMs.

It coordinates placement and lifecycle, composes environments from independently versioned layers, manages memory/resource reclamation and CPU scheduling, and loads image data on demand.

The important EOKS boundary is:

```
semantic/control layer
    |
    | Work / policy / capabilities / context / state
    v
agent harness / execution
    |
    v
stateful execution environment
    |
    +-- process/container/microVM/VM
    +-- filesystem/workspace
    +-- tools/dependencies
    +-- network
    +-- resource limits
    +-- lifecycle/checkpoint
```

EOKS should reason about the semantics exposed by this plane without owning its implementation.

## Environment is more than an image

DSec composes environments from independently versioned layers rather than treating a monolithic image as the only unit.

This strengthens the EOKS **environment/tile** hypothesis:

```
base environment
    +
workspace
    +
toolkits / dependencies
    +
runtime state
```

These components can have different lifecycles and can be materialized according to the workload.

For EOKS, a useful distinction is:

- **Capability** — what an execution is able/allowed to do.
- **Environment** — where/how that capability is realized.
- **Loadout** — the concrete environment/resources selected for a Work.

This avoids making Docker, Kubernetes, microVMs or any particular runtime part of the semantic model.

## State is not the same as a running process

DSec coordinates stateful rollout execution with preemptible training resources. It can preserve rollout state while reclaiming idle execution resources.

That gives EOKS a useful principle:

> **Execution state should be separable from the compute resources currently allocated to it.**

A more complete state model is therefore:

```
Agent state
  decisions, observations, pending work

Work state
  progress, dependencies, recovery position

Environment state
  files, processes, services, installed/runtime changes

Execution state
  lifecycle, allocation, placement, checkpoint/suspend state

Evidence state
  artifacts, traces, evaluations, outcomes
```

These are related but should not automatically collapse into one session object.

## Checkpoint / suspend / resume

DSec makes checkpointing and lifecycle management an infrastructure concern rather than merely a workflow feature.

The important semantic sequence is:

```
RUNNING
   |
checkpoint / suspend
   |
resources reclaimed
   |
RESUMED
   |
RUNNING
```

For EOKS, this suggests that **checkpoint/resume is a portable execution/state semantic** even when the storage and implementation remain runtime-specific.

It also reinforces the existing EOKS distinction between:

- logical Work;
- execution;
- agent session;
- durable state.

A Work should be able to survive replacement or suspension of the particular execution resource that realizes it.

## Capability and policy have an enforcement boundary

DSec's production environment has to deal with agent behavior that can consume excessive resources or exploit unintended access paths.

This reinforces a two-level interpretation of EOKS policy:

```
POLICY
  |
  +-- semantic policy
  |     what the Work is allowed/required to do
  |
  +-- enforcement policy
        what the execution substrate physically permits
        filesystem / network / credentials / resources / isolation
```

The harness can reason about policy, but consequential constraints need enforcement below the model.

This does **not** imply that sandboxing solves agent safety. The paper's experience instead supports treating isolation, resource controls, observability and policy enforcement as layered infrastructure mechanisms.

## Agent-built environments

DSec also supports interactive environment construction followed by packaging/diffing of resulting environment state for reuse.

The EOKS-relevant idea is:

```
agent
  |
  | construct / modify environment
  v
validated environment state
  |
  | package / version / reuse
  v
future execution loadout
```

This is interesting prior art for an **environment materialization loop**.

It should not become an EOKS trust rule: an environment constructed by an agent still needs validation and provenance before it becomes a reusable loadout.

## Execution fleet implications

At small scale:

```
Work -> agent session -> local environment
```

At larger scale:

```
Work
  |
  +--> execution 1
  +--> execution 2
  +--> remote execution
  +--> parallel agents
  +--> fleet
```

DSec strengthens the existing EOKS spectrum:

**one agent session → coordinated executions → remote execution → fleet**

The semantic model should remain stable while placement, isolation and resource-management mechanisms vary.

## Relationship to the six dimensions

DSec does not require a seventh EOKS dimension.

It also reinforces a broader EOKS principle: **execution modality is a workload decision, not an agent taxonomy**. DSec's four backends are selected according to workload functionality, isolation and resource characteristics; similarly, EOKS should choose the smallest/least costly execution substrate that satisfies the Work's requirements and assurance constraints.

Instead, it grounds the **Execution plane** beneath the existing dimensions:

| EOKS dimension | DSec-relevant relationship |
|---|---|
| Intent | determines the Work/environment requirements |
| Workflow | may determine execution topology, but is not required |
| Capabilities | semantic description of allowed/needed actions |
| Knowledge | may contribute environment/context requirements |
| State | must survive execution-resource lifecycle |
| Policy | must be translated into enforceable execution constraints |

This supports the current model:

```
INTENT · WORKFLOW · CAPABILITIES · KNOWLEDGE · STATE · POLICY
                         |
                         v
                  Execution plane
                         |
             local / container / VM /
             remote / fleet execution
```

Execution is therefore broader than workflow and should remain a replaceable substrate.

## Relationship to Jev / probabilistic control

The recent Jev research and DSec research address different layers:

```
control / decision loop
        |
   probabilistic
   decision signals
        |
        v
   execution choice
        |
        v
stateful execution substrate
        |
        v
outcome / evidence
```

Jev-style mechanisms ask questions such as **what should happen next and with what decision signal**.

DSec addresses **where and how that decision can execute durably, elastically and with isolation**.

Neither should become an EOKS primitive merely because it is useful prior art.

## What should not be imported into EOKS

The paper does not justify adding DSec's implementation mechanisms to EOKS. In particular, EOKS should not adopt as semantic primitives:

- a specific container or VM technology;
- a particular distributed filesystem;
- a particular CPU/memory scheduler;
- a specific image distribution mechanism;
- DeepSeek's RL infrastructure;
- DSec's internal sandbox APIs.

Those are implementation choices for an execution provider.

The reusable research findings are the **boundaries and semantics**:

1. execution environments can be heterogeneous;
2. environment composition can be layered;
3. execution state can outlive allocated compute;
4. checkpoint/suspend/resume is a meaningful portable lifecycle semantic;
5. capability and policy need enforceable substrate boundaries;
6. agent-created environments can become reusable artifacts only after validation;
7. execution can scale from individual sessions to fleets without changing the higher-level Work model.

## Additional production observations

Several DSec measurements are useful as hypotheses for EOKS experiments:

- Production sandbox lifetimes are long relative to active CPU use; roughly 90% of containers and microVMs average below 5% of allocated CPU.
- The reported median lifetimes are about 17.4 minutes for containers and 15.5 minutes for microVMs, with p99 lifetimes above three hours.
- Eager image pulling increases completion time by 1.7× in the paper's ablation, while on-demand loading reduces cumulative disk writes by 57%.
- In one production week, the container backend served 11,266 base images, 102,171 workspaces and 103 toolkits; 67.8% of sandboxes required at least one workspace or toolkit in addition to the base image.
- DSec uses cloud bursting for transient peaks; this reinforces that execution capacity can be elastic without changing the logical Work model.

These measurements are specific to DSec's workloads and infrastructure. They should inform hypotheses, not be treated as universal agent-runtime constants.

## Co-design with the control loop

DSec is co-designed with the RL framework rather than exposed as a completely opaque infrastructure service. Stateful rollout execution is separated from preemptible GPU training, allowing GPU resources to be reclaimed while sandbox state remains available for later continuation. The broader architectural lesson is that **control-loop state and execution resources can have different lifecycles**.

The same separation is useful for EOKS software-engineering Work: a controller, model session or worker may be replaced without necessarily losing the Work's authoritative state, evidence or artifacts.

## Agent-built environments and the self-improvement boundary

Section 6 also introduces a loop in which agents construct environments interactively and package their resulting changes into reusable environment state. This is relevant to EOKS because environment construction itself can become part of agentic Work.

The important boundary is validation:

```
agent-built environment
        |
 provenance + tests + policy checks
        |
 reusable loadout
```

An environment produced by an agent should not become a trusted reusable loadout merely because the construction run succeeded. This is analogous to EOKS's broader candidate-learning and controlled-promotion model.

## Open research questions

1. What is the smallest execution contract that can represent local sessions, containers, VMs and remote/fleet execution?
2. What state must survive replacement of an execution resource?
3. How should environment/loadout identity and provenance be represented?
4. Which capability declarations must have substrate-level enforcement?
5. When does a durable execution substrate materially improve a Work enough to justify its operational complexity?
6. Can a Work move between execution backends without losing evidence/provenance?
7. Can the same environment be reused safely across Work items without leaking state?
8. How should resource exhaustion and shared dependency failures appear in the EOKS control loop?
9. What evidence is sufficient to promote an agent-built environment into a reusable loadout?
10. Can execution lifecycle events be normalized alongside artifacts, evaluations and outcomes?

## Proposed experiment

Extend the existing portable-Execution experiment with a small environment dimension.

For one software-engineering Work, compare:

1. a local Claude Code session;
2. a deterministic process;
3. an isolated container;
4. a checkpointable remote execution, where available.

Keep the Work and model/harness as constant as practical. Record:

- environment/loadout identity;
- execution lifecycle;
- state preserved across interruption;
- setup/materialization latency;
- resource usage;
- recovery time;
- artifacts/evidence;
- outcome;
- human intervention;
- cross-run state leakage.

The primary question is not whether a particular backend wins. It is:

> **Which execution semantics are actually necessary for reliable long-horizon engineering Work?**

## Sources

- Huang et al., *DeepSeek Elastic Compute (DSec): A Sandbox Infrastructure for Effective Agentic Training at Scale*, arXiv:2609.22978v1, September 2026. The paper is a 31-page systems report and is the primary source for the architecture and measurements.
- https://arxiv.org/abs/2609.22978

This note records architectural prior art and research hypotheses. It does not make DSec an EOKS dependency or claim that its production design is universally applicable.
