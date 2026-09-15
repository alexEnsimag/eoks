# Next-generation AI development environment

This research pass asks whether the emerging AI developer environment is becoming a new architectural category rather than simply an IDE with an AI assistant. It surveys AI workspaces, agent runtimes, agent-editor protocols, context/intelligence systems, and human-agent software-engineering research.

The goal is not to predict a winner. It is to identify which capabilities are increasingly becoming infrastructure or replaceable, which remain differentiated, and where EOKS should sit above them.

## Executive synthesis

The strongest signal is convergence around a new unit of work: **the agentic software-development workload**, not the editor tab or chat thread.

Several products are independently moving toward a similar shape:

```text
human intent
    |
    v
work / project / task
    |
    +-------------------------------+
    |                               |
    v                               v
agent sessions                 project context
    |                               |
    v                               v
agent / harness  <---------->  code intelligence
    |
    +-------------------------------+
    |
    v
execution environment
    |
    v
artifacts / tests / PRs / runtime evidence
```

The important developments are:

1. **IDE vendors are becoming agent workspaces.** VS Code has Agent Sessions and an Agent Host; Cursor is moving toward Projects, agent fleets and long-running/cloud work; JetBrains is exposing external agents through ACP while retaining deep IDE capabilities; Zed has parallel agent threads.
2. **Agent/editor integration is being unbundled.** ACP is an explicit agent↔client boundary. VS Code's AHP addresses a different client↔persistent-agent-host boundary. These should not be assumed to form one universal protocol stack.
3. **Agent runtimes are becoming explicit infrastructure.** Claude Agent SDK, Codex APIs/app-server, VS Code Agent Host, Herdr and cmux expose increasingly explicit execution, session and lifecycle primitives.
4. **Context is becoming a production subsystem.** OpenWolf, Aider and Sourcegraph demonstrate different forms of context compilation, code intelligence, state, memory and measurement.
5. **The difficult problem is shifting from generation to supervision and verification.** Research increasingly emphasizes task alignment, verifiability, steerability, adaptability, long-horizon execution and software quality rather than raw code generation.
6. **EOKS should not become another IDE, agent runtime or protocol.** Its strongest potential position is semantic control and context evolution across these increasingly interchangeable capabilities.

The central architectural question is therefore not "what should the next IDE contain?" but:

> **What semantic state and control remains necessary when agents, environments and runtimes become replaceable?**

## 1. AI workspaces and IDE evolution

### Cursor

Cursor's trajectory is an especially strong signal. Cursor has moved from an IDE-centered agent experience toward a workspace for multiple agents, long-running work and larger units of software work. Its Projects concept moves the abstraction above an individual conversation toward persistent project context, delegation and recurring work.

The important shift is:

```text
IDE + agent conversation
        ->
project/work + coordinator + agent fleet
```

This is evidence that the IDE surface can become one client of a larger agent work system. It does **not**, by itself, establish that the IDE will cease to be the primary control plane.

### VS Code

VS Code is a particularly clear open architecture example.

Agent Sessions make sessions a first-class unit of work, including local, background and cloud execution, parallel sessions and coordination. More importantly, the August 2026 Agent Host separates persistent agent execution from the editor window. The Agent Host owns sessions, can continue work without a connected editor, supports remote clients, and can host multiple agent harnesses while preserving provider-specific loops and capabilities. AHP standardizes the client-facing interaction with that host. citeturn0search1turn0search3

The resulting boundary is roughly:

```text
VS Code / other client
        |
       AHP
        |
   Agent Host
    /      \
 Copilot   Claude
 harness   harness
```

This gives EOKS a useful reference for separating **client UI**, **session host**, **agent harness** and **semantic workload**.

### JetBrains

JetBrains is approaching the same destination from the opposite direction. The IDE remains the place where developers navigate, understand and review the code, while external agents can connect through ACP. JetBrains explicitly describes the ACP boundary as allowing the agent to retain its own models, behavior, authentication and agent-side tools while the IDE remains the environment for project inspection and review. citeturn0search11turn0search4

This suggests an important inversion:

```text
agent
  |
  +-- terminal/runtime capabilities
  +-- IDE semantic capabilities
  +-- CI capabilities
  +-- database capabilities
```

The IDE can be a **capability provider to an agent**, rather than necessarily owning the agent.

### Zed

Zed helped establish ACP and has moved beyond one-agent chat toward parallel agent threads and an editor that can host external agents. ACP's current ecosystem includes Zed, JetBrains and VS Code among its clients, as well as agents such as Claude Code and Codex CLI. The ACP Registry further turns interoperability into a distribution mechanism: agents can register once and become available to compatible clients. citeturn0search2turn0search6

This is strong evidence that the agent and editor are becoming independently replaceable components, although it does not imply that all agents will expose identical capabilities.

### Replit

Replit demonstrates a different endpoint: an environment and agent deeply integrated around project artifacts and outcome-oriented building. It is useful prior art for environments that optimize around whether the resulting artifact works rather than only around code generation.

## 2. Agent/editor interoperability

### ACP: Agent Client Protocol

ACP separates an agent from the editor. JetBrains and Zed describe it as analogous to LSP: an agent implements the protocol once and can work with compatible editors without bespoke integrations. The ACP Registry now includes agents such as Claude Code, Codex CLI, Copilot CLI, OpenCode and Gemini CLI. citeturn0search5turn0search7

```text
             ACP
              |
    +---------+---------+
    |         |         |
  Zed     JetBrains  other clients
    |         |         |
    +---------+---------+
              |
       interchangeable agents
```

**EOKS implication:** editor/agent integration should be an environment adapter, not a core EOKS primitive.

### AHP: Agent Host Protocol

AHP addresses a different boundary. VS Code describes it as the protocol connecting clients to a persistent Agent Host that owns sessions. AHP standardizes session-oriented client interaction while the underlying harness retains its own loop, tools and capabilities. It also supports multiple clients observing and controlling the same session. citeturn0search1turn0search3

```text
ACP:  editor/client <-> agent
AHP:  client         <-> agent host/session runtime
```

These are **adjacent boundaries, not necessarily layers of one future universal stack**. EOKS should consume whichever boundary an environment exposes rather than prematurely standardizing them into one protocol.

## 3. Context and intelligence layer

### OpenWolf

OpenWolf is an unusually useful compact reference implementation. It adds lifecycle hooks, a shared project state area, project anatomy, memory/context artifacts, token accounting, compaction recovery and a local dashboard around existing coding agents.

Its significance is not feature count. It demonstrates that useful agent infrastructure can be implemented as **small, explicit context/state conventions around existing agents**.

OpenWolf provides concrete evidence supporting several EOKS hypotheses:

- context can be treated as a managed resource;
- session continuity and durable project knowledge are different;
- token usage should be measured, not guessed;
- hooks can make an existing agent more stateful and observable;
- a small UI can expose otherwise invisible workload state.

It is evidence and prior art, **not experimental validation of EOKS as a whole**.

### Aider repository map

Aider is an early and particularly clean example of **context compilation**. It computes a repository map containing important symbols and relationships, ranks the map and selects the most useful subset within a token budget.

The broader lesson is:

> A context source should not be confused with the context delivered to the model.

A repository may be enormous; the working set should be task-specific and budgeted.

### Sourcegraph

Sourcegraph increasingly positions itself as an intelligence layer for developers and AI agents. Its large-codebase research provides evidence that deterministic, code-aware retrieval can improve agent performance, reducing unnecessary exploration and tool calls.

This reinforces an EOKS principle: context selection should combine cheap/deterministic structure with semantic retrieval rather than assuming that conversation memory or a vector store is sufficient.

## 4. Agent runtimes and execution substrates

The ecosystem now has several increasingly explicit runtime layers:

- **Claude Agent SDK** — programmatic Claude Code agent loop, tools, hooks, sessions and permissions.
- **Codex app-server / Agents APIs** — programmatic thread/turn/session control and events.
- **VS Code Agent Host** — persistent session host supporting multiple harnesses.
- **Herdr** — persistent terminal/process runtime with agent state and semantic automation APIs.
- **cmux** — agent-oriented terminal/workspace runtime and Claude Teams integration.

These systems converge on the idea that an agent is not merely a model call. It is a running workload with state, tools, events, environment and lifecycle.

This strengthens the boundary established in the Herdr/cmux and agent-loop mediation research:

> **Execution state belongs to the runtime; semantic state belongs to EOKS.**

## 5. Work and project abstractions

Cursor Projects, VS Code sessions, Zed threads and Replit projects all point toward a richer abstraction than a chat conversation.

A useful decomposition is:

```text
Workload
  |
  +-- objective
  +-- acceptance criteria
  +-- context/resources
  +-- one or more execution sessions
  +-- artifacts
  +-- execution environments
  +-- verification evidence
  +-- human decisions
  +-- history / learning
```

This is close to the EOKS workload model. But there is an important warning:

> **Do not add an EOKS primitive merely because a vendor gives an existing concept a different name.**

Project, Session, Thread, Turn, Run, Task and Workload may overlap substantially. EOKS should only introduce a distinct runtime abstraction when implementation evidence shows that the existing model cannot represent the required state or control.

Current evidence favors a semantic EOKS representation with adapters rather than a replacement workspace runtime.

## 6. Human-agent collaboration and assurance

Recent research increasingly shifts attention from autonomous task completion toward effective human-agent collaboration. Work on human-agent coding highlights task alignment, verifiability, steerability and adaptability as important dimensions.

This matters because the environment should not optimize only for:

```text
more autonomy
```

but for:

```text
appropriate autonomy
+ steerability
+ evidence
+ verification
+ recoverability
```

That supports EOKS's control-loop framing rather than a pure autonomous-agent framing.

## 7. Emerging architectural primitives

The following is deliberately provisional:

| Primitive | Meaning | Emerging examples | EOKS ownership? |
|---|---|---|---|
| Work / Project | durable body of software work | Cursor Projects, Replit projects | Semantic representation / policy, not UI runtime |
| Session | bounded agent execution | VS Code Sessions, Codex/Claude sessions | Observe/coordinate; don't replace vendor state |
| Agent | model + agent loop + tools | Claude, Codex, Junie, Goose | No |
| Harness | provider-specific loop/tool/runtime behavior | Claude Agent SDK, Copilot, Junie | No |
| Agent Host / Runtime | persistent execution/session substrate | VS Code Agent Host, Herdr, cmux | No |
| Agent↔Editor protocol | integration boundary | ACP | No |
| Host↔Client protocol | persistent runtime boundary | AHP | No |
| Context source | information/resource provider | Sourcegraph, MCP servers, OpenWolf, Obsidian | Select and govern |
| Context compiler | turns resources into working set | Aider repo map, OpenWolf | EOKS may orchestrate |
| Evidence | observations supporting decisions | tests, debugger, CI, traces | Yes, semantically |
| Assurance | policy for acceptable outcomes/interventions | approvals, hooks, tests | Yes |
| Outcome | accepted/rejected/partial result | PR/test/review/deploy evidence | Yes |
| Learning | what should affect future work | project context, memory | Yes, selectively |

The table is a research model, not a proposed final EOKS runtime API.

## 8. What is increasingly becoming infrastructure or replaceable

The following capabilities increasingly look like platform infrastructure rather than unique EOKS territory:

- running an agent loop;
- model/provider integration;
- terminal/tool execution;
- session persistence;
- subagents/handoffs;
- editor integration;
- basic token/usage telemetry;
- standard agent↔editor communication;
- local/remote execution;
- generic memory/session storage.

**This is not the same as saying these capabilities are solved or interchangeable today.** Agent runtimes still differ substantially in quality and features. The observation is that major vendors and open projects are increasingly exposing them as explicit interfaces, making it less attractive for EOKS to reimplement them.

EOKS should therefore avoid implementing another version of them unless an experiment demonstrates a missing semantic capability.

## 9. What still looks under-served

### Semantic context evolution

Not merely remembering information, but deciding what remains relevant after a workload evolves, what should be discarded, what should be promoted to durable knowledge, and why.

### Cross-environment work continuity

A workload may move from Claude to Codex, from local to cloud, from IDE to terminal, or from interactive work to unattended automation. The runtime can preserve execution state, but there remains a semantic question about what knowledge/context must cross the boundary.

### Outcome-driven control

Current environments are increasingly good at starting and supervising agents locally. The harder semantic question is deciding when to continue, redirect, acquire more evidence, verify, delegate, escalate or stop.

### Assurance as a first-class control input

Tests and approvals are distributed across tools. EOKS can potentially treat evidence and assurance policy as inputs to workload reconciliation rather than merely logs attached afterward.

### Resource selection across heterogeneous tools

A task may need code search, repository history, architecture docs, a debugger, a database, CI state, previous decisions or another agent. The interesting problem is selecting the minimum sufficient set under cost, latency and trust constraints.

### Learning from outcomes

Most systems preserve transcripts or project files. Fewer explicitly ask what the outcome teaches the system about future context, policy, verification or delegation.

## 10. Alternative hypothesis: the IDE remains the control plane

The convergence evidence does **not** establish that the IDE will become merely a thin client.

A competing model is:

```text
human
  |
  v
IDE / development workspace
  |
  +-- agent A
  +-- agent B
  +-- runtime
  +-- code intelligence
  +-- review / verification
```

JetBrains in particular explicitly preserves the IDE as the place where developers navigate, understand and review the code, while ACP makes the agent replaceable. VS Code's Agent Host can also be interpreted as strengthening the workspace rather than replacing it. citeturn0search11turn0search3

The current evidence therefore supports a weaker and more useful conclusion:

> **The IDE is becoming one component of a broader agentic development environment, but it may remain the primary human control surface.**

This is an open empirical question, not an EOKS architectural assumption.

## 11. EOKS architectural hypothesis after this pass

The emerging environment suggests a stronger formulation of the EOKS hypothesis:

> **EOKS is a semantic control and context-evolution layer for agentic software-development workloads. It coordinates objectives, policy, resources, context, assurance and learning across replaceable agents, harnesses, runtimes and developer environments.**

It should consume and mediate existing primitives rather than reimplement:

- IDEs/editors;
- agent loops;
- terminal/process runtimes;
- session stores;
- agent/editor protocols;
- generic model routing;
- generic memory databases.

The architecture becomes:

```text
                         HUMAN
                           |
                     intent / policy
                           |
                    +------v------+
                    |    EOKS     |
                    |-------------|
                    | Workload    |
                    | Context     |
                    | Knowledge   |
                    | Assurance   |
                    | Conductor   |
                    | Outcomes    |
                    | Learning    |
                    +------+------+
                           |
             semantic decisions / working set
                           |
          +----------------+----------------+
          |                                 |
     Agent-loop APIs                  Runtime APIs
          |                                 |
   Claude / Codex / ...              Herdr / cmux / ...
          |                                 |
          +----------------+----------------+
                           |
                  developer environment
               VS Code / JetBrains / Zed /
                    Cursor / Obsidian / ...
```

The diagram is intentionally not a proposed product architecture. It is a boundary hypothesis to test experimentally.

## 12. Implications for Phase A

The research changes the next experiment priority.

The Codex agent-loop adapter from the previous experiment remains useful, but it should now be treated as one concrete adapter experiment rather than the immediate center of EOKS implementation.

A stronger Phase A sequence is:

1. **Observe OpenWolf in real use.** Record which EOKS concepts it actually makes useful without requiring a larger framework.
2. **Compare the same workload across OpenWolf + Claude/Codex, VS Code Agent Host and an ACP client where practical.** Identify which state is execution state versus semantic state.
3. **Exercise one cross-environment workload.** Move a task/session between terminal, IDE and/or agent implementations and identify what must survive.
4. **Implement only the missing semantic control point.** For example, choose context, verification or intervention based on outcome rather than adding another runtime abstraction.
5. **Only then revisit the Codex adapter and shared AgentAdapter abstraction.**

The key experiment should test:

> **Can EOKS add semantic value when the underlying agent, IDE, context middleware and runtime are already good enough?**

If the answer is yes, that is much stronger validation of the EOKS thesis than demonstrating another agent runtime or another session abstraction.

## 13. Research boundary

This document should remain a landscape and architectural hypothesis, not a commitment to build an "AI IDE."

Before adding a new EOKS abstraction, ask:

1. Is this genuinely missing across existing agents, environments or runtimes?
2. Is it semantic state/control, or merely execution infrastructure?
3. Can an existing protocol, adapter or context source provide it?
4. What experiment would demonstrate that a new primitive is necessary?
5. What evidence would cause us to reject the hypothesis?

The purpose of this research is therefore to **move EOKS upward only where the ecosystem does not already provide a good lower layer**.
