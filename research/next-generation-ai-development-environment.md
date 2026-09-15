# Next-generation AI development environment

This research pass asks whether the emerging AI developer environment is becoming a new architectural category rather than an IDE with an AI assistant. It surveys IDE/workspace products, agent runtimes, agent-editor protocols, context/intelligence systems, and research on human-agent software engineering.

The goal is not to predict a winner. It is to identify which capabilities are becoming infrastructure, which remain differentiated, and where EOKS should sit above them.

## Executive synthesis

The strongest signal is convergence around a new unit of work: **the agentic software-development workload**, not the editor tab or chat thread.

Several products are independently moving toward the same shape:

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

1. **IDE vendors are becoming agent workspaces.** VS Code has Agent Sessions and now an Agent Host; Cursor has moved to Projects and long-running/cloud agents; JetBrains has made Junie CLI an ecosystem-level agent while retaining deep IDE intelligence; Zed has parallel agent threads.
2. **Agent/editor integration is being unbundled.** ACP is an explicit protocol boundary. VS Code is separately developing AHP for persistent agent-host sessions. The agent itself is becoming a replaceable component rather than an IDE feature.
3. **Agent runtimes are becoming infrastructure.** Claude Agent SDK, Codex APIs/app-server, VS Code Agent Host, Herdr and cmux expose increasingly explicit execution/session/runtime primitives.
4. **Context is becoming a production subsystem.** OpenWolf, Aider, Sourcegraph and similar systems show different forms of context compilation, code intelligence, memory and measurement.
5. **The difficult problem is shifting from generation to supervision and verification.** Research increasingly emphasizes task alignment, verifiability, steerability, adaptability, long-horizon execution, and software quality rather than raw code generation.
6. **EOKS should not become another IDE, agent runtime, or protocol.** Its strongest potential position is the semantic control/evolution layer across these increasingly standardized capabilities.

## 1. AI workspaces and IDE evolution

### Cursor

Cursor's trajectory is especially explicit. Cursor 3 describes a "third era" in which fleets of agents work autonomously, and introduces a unified workspace rather than forcing developers to manage individual agent conversations. Projects, launched September 2026, move the abstraction another level upward: a Project represents a body of work, maintains shared context over months, delegates to many agents, and can run recurring/event-triggered work.

The important architectural shift is:

```text
IDE + agent conversation
        ->
project/work + coordinator + agent fleet
```

Cursor is also investing in long-running agents, cloud/local handoff, remote development environments, computer-use environments and automations. This is evidence that the IDE surface is becoming one client of a larger agent work system.

### VS Code

VS Code is arguably the clearest open architecture example.

Agent Sessions makes the session a first-class unit of work. Sessions can be local, background or cloud; they can be forked, handed off, run in parallel, and coordinated through session-management tools. VS Code can also discover sessions created by Claude Code, Codex and Copilot outside VS Code.

In August 2026, VS Code introduced the **Agent Host**: a dedicated process owning persistent agent sessions, with the open **Agent Host Protocol (AHP)** for client/host communication. The host can support multiple agent harnesses while retaining their provider-specific loops and capabilities. This is a major architectural signal: the editor is separating session/runtime ownership from the editor window.

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

VS Code therefore gives EOKS a useful reference for separating **client UI**, **session host**, **agent harness**, and **semantic workload**.

### JetBrains

JetBrains is approaching the same destination from the opposite direction. Junie started IDE-native, but Junie CLI is now a standalone, LLM-agnostic coding agent usable in terminals, IDEs, CI/CD and GitHub/GitLab.

The key differentiator is not that Junie is an IDE plugin. It is that the agent can use the IDE as a capability provider: semantic index, build configurations, test runners, debugger and database integrations. The GA integration is built around ACP, and JetBrains is explicitly positioning the IDE as the agent's toolbox.

This suggests a future where:

```text
agent
  |
  +-- terminal/runtime capabilities
  +-- IDE semantic capabilities
  +-- CI capabilities
  +-- database capabilities
```

rather than a monolithic IDE-owned assistant.

### Zed

Zed helped establish ACP and has moved beyond one-agent chat. Its Parallel Agents feature provides a Threads Sidebar for multiple agents, projects and repositories, with per-thread access controls. Zed's role is increasingly that of a fast agent client/workspace rather than a vendor-locked agent.

### Replit

Replit demonstrates a different endpoint: the environment and agent are deeply integrated, with design canvas, artifacts, shared project state and increasingly autonomous building. Its evaluation work is notable because the product measures whether the resulting artifact actually works, not merely whether the agent generated plausible code.

Replit is therefore useful prior art for **outcome-oriented agent environments**, especially outside traditional repository/IDE workflows.

## 2. Agent/editor interoperability

### ACP: Agent Client Protocol

ACP, created by Zed and now co-developed with JetBrains, separates the agent from the editor. The ACP registry lets an agent register once and become available to compatible clients. The registry includes Claude Code, Codex CLI, Copilot CLI, OpenCode, Gemini CLI and others.

This is conceptually similar to LSP's unbundling of language intelligence from IDEs:

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

**EOKS implication:** do not make an editor integration a core EOKS primitive. Treat editor/agent protocols as adapters at the environment boundary.

### AHP: Agent Host Protocol

VS Code's AHP solves a different problem. ACP is primarily an agent/client interaction boundary; AHP standardizes the client-facing interface to a persistent agent host that owns sessions and can run different harnesses.

This distinction is useful:

```text
ACP:  editor/client <-> agent
AHP:  client         <-> agent host/session runtime
```

EOKS should not collapse these into one generic protocol prematurely.

## 3. Context and intelligence layer

### OpenWolf

OpenWolf is an unusually useful compact reference implementation. It adds lifecycle hooks, a shared `.wolf/` state area, project anatomy, memory/context artifacts, token accounting, compaction recovery and a local dashboard around existing coding agents.

Its significance is not feature count. It demonstrates that a large portion of useful agent infrastructure can be implemented as **small, explicit context/state conventions around existing agents**.

OpenWolf therefore validates several EOKS hypotheses simultaneously:

- context can be treated as a managed resource;
- session continuity and durable project knowledge are different;
- token usage should be measured, not guessed;
- hooks can turn an existing agent into a more stateful system;
- a small local UI can make the invisible agent workload observable.

It should be treated as a concrete Phase A reference, not as something EOKS needs to reproduce wholesale.

### Aider repository map

Aider is an earlier and particularly clean example of **context compilation**. It computes a repository map containing important symbols and relationships, ranks the map using a dependency graph, and selects the most relevant subset within a token budget.

The lesson is broader than Aider:

> A context source should not be confused with the context delivered to the model.

A repository may be enormous; the working set should be task-specific and budgeted.

### Sourcegraph

Sourcegraph now explicitly positions itself as an intelligence layer for developers and AI agents. Its 2026 CodeScaleBench work demonstrates that deterministic, code-aware retrieval can materially improve agent performance on large repositories, including reducing tool calls and elapsed time.

This reinforces a key EOKS principle: context selection should combine cheap/deterministic structure with semantic retrieval rather than assuming that a vector database or conversation memory is sufficient.

## 4. Agent runtimes and execution substrates

The ecosystem now has several increasingly explicit runtime layers:

- **Claude Agent SDK** — programmatic Claude Code agent loop, tools, hooks, sessions and permissions.
- **Codex app-server / Agents APIs** — programmatic thread/turn/session control and events.
- **VS Code Agent Host** — persistent session host supporting multiple harnesses.
- **Herdr** — persistent terminal/process runtime with agent state and semantic automation APIs.
- **cmux** — agent-oriented terminal/workspace runtime and Claude Teams integration.

These systems are converging on the idea that an agent is not just a model call. It is a running workload with state, tools, events, environment and lifecycle.

This strengthens the EOKS boundary established in the Herdr/cmux and agent-loop mediation research:

> **Execution state belongs to the runtime; semantic state belongs to EOKS.**

## 5. Work/project abstractions

Cursor Projects, VS Code sessions, Zed threads, JetBrains remote/long-running tasks and Replit projects all point toward a richer abstraction than a chat conversation.

A useful decomposition is:

```text
Project / Workload
  |
  +-- objective
  +-- acceptance criteria
  +-- context/resources
  +-- one or more sessions
  +-- artifacts
  +-- execution environments
  +-- verification evidence
  +-- human decisions
  +-- history / learning
```

This is close to the EOKS workload model. The important question is whether EOKS needs to define this abstraction itself or simply consume equivalent project/session/work objects from existing systems. Current evidence favors a semantic EOKS representation with adapters, not a replacement workspace runtime.

## 6. Human-agent collaboration and assurance

Recent research increasingly argues that the bottleneck is moving from autonomous task completion toward effective human-agent collaboration.

The 2026 position paper **Humans are Missing from AI Coding Agent Research** identifies four interaction dimensions: task alignment, verifiability, steerability and adaptability. This maps closely to EOKS concerns around policy, assurance, intervention and outcomes.

Other 2026 work on agentic software engineering emphasizes orchestration, verification, governance and accountable oversight as the engineer's role changes.

This is important because the emerging environment should not optimize only for:

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

The research suggests a provisional set of primitives for an AI development environment:

| Primitive | Meaning | Emerging examples | EOKS ownership? |
|---|---|---|---|
| Work / Project | durable body of software work | Cursor Projects, Replit projects | Semantic representation / policy, not UI runtime |
| Session | bounded agent work execution | VS Code Sessions, Codex/Claude sessions | Observe/coordinate; don't replace vendor state |
| Agent | model + agent loop + tools | Claude, Codex, Junie, Goose | No |
| Harness | provider-specific loop/tool/runtime behavior | Claude Agent SDK, Copilot, Junie | No |
| Agent Host / Runtime | persistent execution/session substrate | VS Code Agent Host, Herdr, cmux | No |
| Agent↔Editor protocol | integration boundary | ACP | No |
| Host↔Client protocol | persistent runtime boundary | AHP | No |
| Context source | information/resource provider | Sourcegraph, MCP servers, OpenWolf, Obsidian | Select and govern |
| Context compiler | turns resources into working set | Aider repo map, OpenWolf | EOKS may orchestrate |
| Evidence | observations supporting decisions | tests, debugger, CI, traces | Yes, semantically |
| Assurance | policy for acceptable outcomes/interventions | IDE approvals, hooks, tests | Yes |
| Outcome | accepted/rejected/partial result | PR/test/review/deploy evidence | Yes |
| Learning | what should affect future work | Cursor Project context, OpenWolf memory | Yes, selectively |

This table should remain provisional. In particular, **Project**, **Session**, **Run**, and **Workload** may overlap and should not become separate EOKS primitives without implementation evidence.

## 8. What is becoming commoditized

The following capabilities increasingly look like platform infrastructure rather than EOKS differentiation:

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

OpenAI, Anthropic, Microsoft, JetBrains, Zed and others are rapidly moving these capabilities into explicit APIs or standardized integration boundaries.

EOKS should therefore avoid implementing another version of them unless an experiment demonstrates a missing semantic capability.

## 9. What still looks under-served

The following remain fragmented and are closer to EOKS's potential territory:

### Semantic context evolution

Not merely remembering information, but deciding what remains relevant after a workload evolves, what should be discarded, what should be promoted to durable knowledge, and why.

### Cross-runtime work continuity

A workload may move from Claude to Codex, from local to cloud, from IDE to terminal, or from interactive work to unattended automation. The runtime can preserve execution state, but there is still a semantic question about what knowledge/context must cross the boundary.

### Outcome-driven control

Current environments are increasingly good at starting agents. The harder question is deciding when to continue, redirect, acquire more evidence, verify, delegate, escalate or stop.

### Assurance as a first-class control input

Tests and approvals are currently distributed across tools. EOKS can potentially treat evidence and assurance policy as inputs to workload reconciliation rather than merely logs attached afterward.

### Resource selection across heterogeneous tools

A task may need code search, repository history, architecture docs, a debugger, a database, CI state, previous decisions, or another agent. The interesting problem is selecting the minimum sufficient set under cost/latency/trust constraints.

### Learning from outcomes

Most systems preserve transcripts or project files. Fewer explicitly ask what the outcome teaches the system about future context, policy, verification or delegation.

## 10. EOKS architectural hypothesis after this pass

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

## 11. Implications for Phase A

The research changes the next experiment priority.

The Codex agent-loop adapter from the previous experiment remains useful, but it should now be treated as one concrete adapter experiment rather than the immediate center of EOKS implementation.

A stronger Phase A sequence is:

1. **Observe OpenWolf in real use.** Record which EOKS concepts it actually makes useful without requiring a larger framework.
2. **Compare the same workload across OpenWolf + Claude/Codex, VS Code Agent Host, and an ACP client where practical.** Identify which state is execution state versus semantic state.
3. **Exercise one cross-environment workload.** Move a task/session between terminal, IDE and/or agent implementations and identify what must survive.
4. **Implement only the missing semantic control point.** For example, choose context/verification/intervention based on outcome rather than adding another runtime abstraction.
5. **Only then revisit the Codex adapter and shared AgentAdapter abstraction.**

The key experiment should test:

> Can EOKS add semantic value when the underlying agent, IDE, context middleware and runtime are already good enough?

If the answer is yes, that is a much stronger validation of EOKS than building another complete agent environment.

## Sources and further reading

- Cursor — Projects: https://cursor.com/blog/projects
- Cursor — Cursor 3 / unified agent workspace: https://cursor.com/blog/cursor-3
- Cursor — long-running agents: https://cursor.com/blog/long-running-agents
- Cursor — automations: https://cursor.com/blog/automations
- VS Code — Agent Host and AHP: https://code.visualstudio.com/blogs/2026/08/26/agent-host-architecture
- VS Code — multi-agent development: https://code.visualstudio.com/blogs/2026/02/05/multi-agent-development
- VS Code — sessions: https://code.visualstudio.com/docs/agents/run/sessions/manage-sessions
- VS Code — subagents: https://code.visualstudio.com/docs/agents/run/subagents
- JetBrains — Junie GA: https://blog.jetbrains.com/junie/2026/06/junie-coding-agent-out-of-beta/
- JetBrains — Junie CLI + IDE: https://blog.jetbrains.com/junie/2026/04/junie-cli-inside-your-jb-ide/
- JetBrains — ACP in IntelliJ: https://blog.jetbrains.com/idea/2026/08/how-to-use-ai-agents-in-intellij-idea-with-acp/
- Zed — ACP: https://zed.dev/acp
- Zed — ACP registry: https://zed.dev/blog/acp-registry
- Zed — parallel agents: https://zed.dev/blog/parallel-agents
- OpenWolf: https://github.com/cytostack/openwolf
- Aider repository map: https://github.com/Aider-AI/aider/blob/main/aider/website/docs/repomap.md
- Sourcegraph — context engineering: https://sourcegraph.com/blog/context-engineering
- Sourcegraph — agentic coding: https://sourcegraph.com/blog/agentic-coding
- Sourcegraph — intelligence layer: https://sourcegraph.com/blog/a-new-era-for-sourcegraph-the-intelligence-layer-for-ai-coding-agents-and-developers
- Replit — Agent 4: https://replit.com/blog/whats-changed-agent3-to-agent4
- Replit — evaluation loop: https://replit.com/blog/evaluating-and-improving-agent-at-scale
- Research — Humans are Missing from AI Coding Agent Research: https://arxiv.org/abs/2608.12355
- Research — Human-AI Collaboration and the Transformation of Software Engineering Work: https://arxiv.org/abs/2606.03394
- Research — Rethinking Software Engineering for Agentic AI Systems: https://arxiv.org/abs/2604.10599
