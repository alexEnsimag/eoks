# Agent workflow runtimes and EOKS

## Why this matters

Recent investigation of agent workflow tooling clarifies an important part of EOKS, but **does not make EOKS workflow-centric**.

The new evidence is better understood through the existing **Execution tile**: modern agent runtimes provide increasingly rich mechanisms for turning a piece of work into one or more executions, coordinating them, persisting their state, and observing what happened.

Frameworks such as LangGraph, CrewAI, Microsoft Agent Framework, Google ADK, OpenAI Agents SDK and AutoGen are useful prior art for **execution topology and lifecycle**. They are not the EOKS architecture, and EOKS does not need to own or standardize a workflow engine.

The relevant EOKS question is:

> **What execution semantics does EOKS need to understand so that a Claude Code session, a workflow runtime, a remote agent, a deterministic process, or a fleet of agents can all participate in the same Work?**

## Current ecosystem signal

The 2026 framework landscape is converging around several execution models:

- **Graph/state-machine orchestration** — explicit nodes, edges and state; LangGraph is a prominent example.
- **Role/task orchestration** — agents are assigned responsibilities and collaborate; CrewAI is a prominent example.
- **Agent-loop / handoff orchestration** — a lighter loop where the model chooses tools or hands off to another agent; OpenAI Agents SDK is an example.
- **Event/workflow-oriented runtimes** — workflows combine agent steps with deterministic event-driven execution; Google ADK and Microsoft Agent Framework expose this broader direction.

AutoGen is important prior art for conversational multi-agent execution, but Microsoft has moved new development toward Microsoft Agent Framework. EOKS should therefore study AutoGen for its execution patterns without treating it as the preferred current dependency.

The ecosystem evidence also reinforces that a framework is not the whole production stack. Frameworks address application-level orchestration; observability/evaluation, security, persistence, deployment and infrastructure remain separate concerns.

## EOKS interpretation: Execution is broader than workflow

The current EOKS model should preserve **Execution** as the broader concept.

~~~text
Work
  |
  +-- objective / context / resources / assurance
  |
  +-- Execution
       |
       +-- one coding-agent session
       +-- deterministic process
       +-- workflow / graph
       +-- parallel agent executions
       +-- remote agent
       +-- fleet / distributed execution
       +-- human participation
       +-- hybrid combinations
~~~

A workflow is therefore one possible **execution topology**, not the ontology of EOKS.

The Execution tile concerns at least four dimensions:

~~~text
EXECUTION
├── topology
│   ├── single
│   ├── sequential
│   ├── graph / branching
│   ├── parallel
│   ├── iterative loop
│   └── fleet / hierarchical / handoff
│
├── placement
│   ├── local
│   ├── remote
│   ├── sandbox
│   ├── container / Kubernetes
│   └── CI / external service
│
├── lifecycle
│   ├── start
│   ├── state
│   ├── checkpoint
│   ├── interrupt
│   ├── resume
│   ├── retry / recover
│   ├── fork / redirect
│   └── terminate
│
└── coordination
    ├── agent
    ├── agent ↔ agent
    ├── agent ↔ tool
    ├── agent ↔ human
    └── external event
~~~

This is why the recent framework research is relevant to EOKS even though EOKS is not a workflow engine.

## Relationship to Claude sessions and harness tools

The new runtime research also clarifies a boundary that was previously easy to blur.

### Harness

A harness shapes **how an individual agent operates**:

~~~text
Claude Code session
├── CLAUDE.md / project policy
├── skills
├── hooks
├── MCP / tools
├── permissions
├── context delivery
├── memory/session mechanisms
└── local execution environment
~~~

Portal/Shunt, OpenWolf and related harness mechanisms can modify or observe this local agent operating environment.

### Execution runtime

An execution runtime shapes **how one or more executions are coordinated**:

~~~text
single session
      |
      +-- workflow
      +-- graph
      +-- parallel agents
      +-- handoffs
      +-- human gates
      +-- remote execution
      +-- checkpoint / resume
~~~

LangGraph, CrewAI, Microsoft Agent Framework, Google ADK and OpenAI Agents SDK can occupy this layer to different degrees.

### EOKS

EOKS is concerned with the larger Work:

~~~text
Work
 |
 +-- intent / objective
 +-- context / knowledge
 +-- resources / capabilities
 +-- execution
 +-- evidence
 +-- evaluation
 +-- human decisions
 +-- outcome
~~~

A Claude Code session can therefore be an Execution without a workflow runtime. A workflow runtime can orchestrate Claude Code sessions. EOKS can potentially observe, select or combine both.

This preserves the earlier EOKS principle:

> **Use the minimum coordination structure that provides the evidence, assurance and outcome required by the workload.**

## What the runtime capabilities add

The recent evidence makes several capabilities worth treating as **portable execution semantics**, even if their implementation remains runtime-specific:

| Capability | EOKS interpretation |
|---|---|
| Topology | How work is distributed across executions |
| State | State associated with an active execution |
| Checkpoint | Durable recovery boundary |
| Interrupt | Explicit pause/decision boundary |
| Resume | Continue a prior execution from persisted state |
| Retry / recovery | Continue after partial failure |
| Human-in-the-loop | Human becomes an execution participant/decision boundary |
| Handoff | Transfer responsibility between executions |
| Observability | Record execution events and traces |
| Visualization | Inspect execution topology/state |
| Evaluation | Assess execution outcome; not equivalent to tracing |
| Placement | Where the execution actually runs |

The important distinction is between **semantics EOKS may need to understand** and **features EOKS must implement itself**.

For example, EOKS may need to know that an execution is checkpointed and waiting for human input without implementing the checkpoint storage itself.

## Frameworks versus EOKS

| Concern | Runtime | EOKS |
|---|---|---|
| Define/execute a topology | Often | Select or adapt topology |
| Execute agent/tool steps | Yes | Coordinate/observe |
| Agent-local harness | Sometimes | Treat as an execution resource |
| State/checkpoint | Often | Govern/observe execution state |
| Interrupt/resume | Increasingly | Use as control/assurance boundaries |
| Human approval | Often | Policy/assurance decision |
| Placement | Varies | Select resources/environment |
| Context construction | Varies | First-class context/resource concern |
| Evidence selection | Usually not central | First-class control concern |
| Cross-execution / cross-run learning | Limited/varies | Core research concern |
| Outcome/evidence reconciliation | Usually application-specific | Core EOKS loop |

The boundary is deliberately porous. One product can provide capabilities from several rows. EOKS should reason about capabilities and contracts, not force vendor products into fixed layers.

## The EOKS execution control loop

This also connects the runtime research to the earlier Kubernetes/control-loop analogy without making EOKS a workflow engine:

~~~text
desired Work / objective
          |
          v
observe context + resources + prior state
          |
          v
select execution modality
          |
          v
execute
          |
          v
observe events / artifacts / evidence
          |
          v
evaluate outcome / assurance
          |
     +----+----+
     |         |
     v         v
continue    adapt/reconcile
     |         |
     +----<----+
~~~

A workflow can implement the execution portion. A single agent loop can also implement it. The control loop is the broader EOKS concern.

## Research / experiment implications

The useful experiment is therefore not a framework bake-off and not a decision to make EOKS a workflow engine.

Instead, test whether a small **portable Execution model** is sufficient across different execution substrates.

For the same small software-engineering Work, compare:

1. one Claude Code session;
2. a deterministic workflow;
3. a graph-based agent runtime;
4. a role-based multi-agent execution;
5. optionally, remote or parallel executions.

Capture comparable EOKS-level observations:

- objective/context;
- execution topology and placement;
- lifecycle events;
- state/checkpoint/interruption boundaries;
- human interventions;
- artifacts and evidence;
- context cost;
- latency;
- retries/recovery;
- coordination overhead;
- outcome/evaluation;
- reproducibility.

The question is:

> **Can EOKS reason about these executions without owning their implementation?**

A stronger hypothesis is:

> **Execution topology and lifecycle are EOKS-relevant semantics; the mechanism that realizes them is replaceable infrastructure.**

## Design posture

Do not introduce an AgentWorkflowRuntime as a new EOKS primitive merely because these frameworks exist.

Instead:

- preserve **Execution** as the broader EOKS tile;
- treat workflow/graph runtimes as execution substrates;
- treat Claude Code sessions and harness tools as execution resources/local operating environments;
- preserve the possibility of simple execution without a workflow runtime;
- model checkpoint, interrupt, resume, human participation and evidence as useful execution semantics;
- let experiments determine which semantics actually need stable EOKS interfaces.

This keeps EOKS open to a spectrum from **one agent session → coordinated agents → remote fleets**, without prematurely making any one topology the center of the architecture.

## Sources / prior art

- LangGraph / LangChain framework landscape: https://www.langchain.com/resources/ai-agent-frameworks
- CrewAI: https://docs.crewai.com/
- Microsoft Agent Framework: https://learn.microsoft.com/en-us/agent-framework/
- Google Agent Development Kit: https://google.github.io/adk-docs/
- OpenAI Agents SDK: https://openai.github.io/openai-agents-python/
- AutoGen: https://microsoft.github.io/autogen/

These sources are ecosystem evidence, not EOKS requirements.


The framework landscape gives concrete implementation evidence for concepts already present in EOKS:

- **Loop engineering** — repeated act/observe/evaluate/retry cycles.
- **Graph engineering** — explicit dependencies, branches, parallelism and joins.
- **Workflow** — the reusable execution topology and control conditions.
- **Run** — one execution of a workload/task or subgraph.
- **Role** — a responsibility implemented by a node, agent, deterministic function or human.
- **Conductor** — decides which topology/resource/execution modality is appropriate; it does not have to be implemented by the framework.
- **Evaluation** — determines whether the execution produced sufficient evidence/outcome; framework traces are observations, not acceptance by themselves.

This validates the existing EOKS distinction between **execution graph** and **control plane** rather than requiring a new EOKS primitive.

## Frameworks versus EOKS primitives

| Concern | Agent workflow runtime | EOKS |
|---|---|---|
| Define workflow topology | Yes | Select/adapt topology |
| Execute graph/loop | Yes | Coordinate/observe execution |
| Tool calling | Yes | Treat tools as resources/providers |
| State/checkpointing | Often | Govern run/workload state |
| Human approval | Often | Policy/assurance decision |
| Context construction | Varies | First-class context compilation |
| Knowledge/memory lifecycle | Varies | Resource/knowledge lifecycle |
| Evidence-provider selection | Usually not the central concern | First-class control decision |
| Cross-run learning/promotion | Limited/varies | Core research concern |
| Outcome/evidence-based reconciliation | Usually application-specific | Core EOKS loop |
| Production observability/evaluation | Often integrated or adjacent | Consumes evaluation/observability as control evidence |

The distinction is deliberately porous: a runtime can provide some context, memory, evaluation or observability features. EOKS should model those as capabilities rather than assuming the framework boundary is universal.

## Relationship to existing EOKS tooling

This also clarifies why agent workflow runtimes belong beside, rather than inside, the harness/tooling already studied:

```
                 EOKS environment
                       |
       +---------------+----------------+
       |               |                |
   context/          harness          execution
   knowledge         hooks/skills      workflows
       |               |                |
 OpenWolf/Portal   Claude Code       LangGraph/
 Understand        cmux/Herdr        CrewAI/MAF/
 Anything          MCP              ADK/etc.
       |               |                |
       +---------------+----------------+
                       |
                 observe/evaluate
```

These are complementary mechanisms:

- **Harness mechanisms** shape the agent's local operating environment and information flow.
- **Workflow runtimes** define/execute multi-step agentic work.
- **Execution surfaces/runtimes** such as cmux or Herdr manage live processes, sessions and human attention.
- **Knowledge/context systems** determine what information is available and how it is compiled.
- **Evaluation/observability systems** record and assess what happened.
- **EOKS** is the proposed coordinating/control layer that decides how these resources should be combined for a workload.

A single product can span several of these categories; the categories describe capabilities, not vendor boundaries.

## Important design consequence

EOKS should **not build a graph runtime merely because graph runtimes exist**.

Instead, the research question is:

> Can an EOKS conductor select among a simple agent loop, deterministic workflow, graph runtime, coding-agent session, or richer multi-agent topology based on workload requirements and evidence?

That is consistent with the existing principle:

> **Use the minimum coordination structure that provides the evidence, assurance and outcome required by the workload.**

A framework should earn its place by reducing implementation cost or improving durability, inspectability, reliability or operational behavior—not by making a workflow look more agentic.

## Research / experiment implications

The highest-value next experiment is not a framework bake-off. It is a **runtime substitution experiment**:

1. Express the same small software-engineering workflow as:
   - one coding-agent run;
   - a deterministic workflow;
   - a graph-based agent runtime;
   - optionally a role-based multi-agent workflow.
2. Capture the same EOKS Run-level observations.
3. Compare outcome quality, evidence coverage, context cost, latency, retries, coordination overhead, human intervention and reproducibility.
4. Test whether the EOKS-level control decisions remain portable across runtimes.

This would test the hypothesis that **workflow topology is an EOKS control concern while the mechanism used to execute that topology is replaceable infrastructure**.

## Sources / prior art

- LangGraph / LangChain framework landscape: https://www.langchain.com/resources/ai-agent-frameworks
- CrewAI: https://docs.crewai.com/
- Microsoft Agent Framework: https://learn.microsoft.com/en-us/agent-framework/
- Google Agent Development Kit: https://google.github.io/adk-docs/
- OpenAI Agents SDK: https://openai.github.io/openai-agents-python/
- AutoGen: https://microsoft.github.io/autogen/

These sources are ecosystem evidence, not EOKS requirements.
