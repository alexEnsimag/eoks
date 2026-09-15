# Personal AI Engineering Environment

## Status

Research direction / architectural hypothesis. This document records a synthesis of the current ecosystem; it is not a commitment to build a universal IDE, agent runtime, or orchestration platform.

## Motivation

Recent research suggests that many capabilities needed for AI-assisted software development already exist, but are split across different tools:

- **Obsidian** can provide a personal workspace for notes, projects, goals, research, decisions, and a connected knowledge graph.
- **OpenWolf** provides practical agent context optimization, lifecycle hooks, project/session state, handoff, and token/context measurement.
- **cmux** provides a live workspace for running and organizing multiple agent sessions, terminals, panes, notifications, and agent-specific integrations.
- **Herdr** provides a programmable execution/runtime layer for persistent agent sessions and runtime control, including model/agent switching capabilities.
- **Claude Agent SDK** and **Codex app-server** expose direct programmatic control of agent sessions and their execution loops.
- **ACP** and IDE integrations provide a replaceable boundary between agents and development environments.
- **Code-intelligence systems** provide repository and software-system understanding.
- **Evaluation/observability systems** provide evidence about whether work succeeded and what it cost.
- **Spotify's fleet work** demonstrates that the same general model can extend from one local agent to large numbers of background agents working across repositories.

The hypothesis is that these capabilities could be composed into one expandable environment for an individual engineer rather than treated as isolated tools.

## Core idea

> A personal AI engineering environment gives one developer a unified place to understand their work, manage knowledge, run agents, manage local and remote execution, observe costs and outcomes, and continuously improve how they work with AI.

This is broader than an AI IDE and broader than an agent framework. The goal is to empower the developer while keeping underlying agents, runtimes, context systems, and execution infrastructure replaceable.

## Capability map

| Capability | What it means | Existing examples | Possible EOKS role |
| --- | --- | --- | --- |
| Personal engineering workspace | Projects, goals, notes, research, decisions, architecture, history | Obsidian | Integrate / extend |
| Connected engineering knowledge | Link projects, decisions, sessions, experiments, and evidence | Obsidian graph; memory systems | Integrate / evolve |
| Code understanding | Understand files, symbols, dependencies, services, and architecture | Code intelligence / graph tools | Integrate |
| Context optimization | Give agents only useful information and avoid repeated/expensive context | OpenWolf; Portal-style mechanisms | Integrate / coordinate |
| Token and usage tracking | Measure tokens, context, tool calls, latency, and cost | OpenWolf and related tooling | Integrate / correlate with outcomes |
| Session state and handoff | Preserve useful state across agent sessions | OpenWolf; agent runtimes | Integrate |
| Live local execution | Organize and observe currently running agents and tools | cmux | Integrate |
| Runtime/session management | Start, persist, inspect, resume, and control agent execution environments | Herdr / cmux | Integrate |
| Direct agent API control | Programmatically start/resume/observe/continue/intervene in agent sessions | Claude Agent SDK; Codex app-server | Integrate; potentially coordinate |
| Agent/environment interoperability | Allow different agents to work with different development environments | ACP; IDE agent integrations | Integrate |
| Tools and capabilities | Git, shell, browser, databases, IDE capabilities, MCP tools, etc. | Existing harness/tool ecosystems | Integrate |
| Permissions and guardrails | Control what an agent can read, change, execute, or deploy | Agent/IDE/runtime permission systems | Coordinate policies |
| Evaluation | Determine whether work actually succeeded | Tests, evaluation and observability tools | Integrate / correlate |
| Background execution | Run work without keeping the developer in a terminal | Agent APIs; fleet systems | Integrate |
| Agent fleets | Run many agents against many tasks/repositories and track exceptions | Spotify Fleetshift/Honk direction | Integrate / experiment |
| Unified work tracking | Treat the goal, context, agents, execution, artifacts, metrics, and outcome as one piece of work | No clear dominant solution found | Strong EOKS hypothesis |
| Cross-system coordination | Connect workspace, context, agents, runtimes, permissions, evaluation, and history | No clear dominant solution found | Strong EOKS hypothesis |
| Learning from outcomes | Use successful/failed work to improve future context, choices, and workflows | Memory/self-evolving systems; emerging research | Strong EOKS hypothesis |

## One environment, different execution modes

The developer should not have to think in terms of implementation-specific session types.

The same work item could contain:

```text
EOKS migration

Agents
  Claude       local       working
  Codex        local       reviewing
  Claude       remote      12/20 repositories complete
  Claude       remote      3 PRs need attention

Context
  project architecture
  current decisions
  relevant code
  previous experiment results

Execution
  local cmux session
  remote fleet jobs

Metrics
  tokens / cost / duration / tool calls

Outcome
  tests / PRs / review / evidence
```

A local single-agent session, a local multi-agent workspace, and a remote fleet job should ideally be different execution modes of the same higher-level work model.

## Obsidian as the personal engineering home

Obsidian is particularly interesting because it can potentially become more than a note-taking surface.

A future engineering workspace could organize:

- projects
- quarterly and long-term goals
- technical decisions and their rationale
- architecture
- research
- experiments
- bugs and recurring problems
- agent sessions
- work items
- outcomes and reviews
- personal engineering knowledge

The graph can connect these objects over time. For example:

```text
Goal
  -> Project
    -> Decision
      -> Implementation
        -> Agent session
          -> Experiment
            -> Result
              -> New decision
```

The important hypothesis is that agent execution and its measurements could become part of this workspace rather than a disconnected dashboard.

## OpenWolf as an agent-efficiency subsystem

OpenWolf is a useful reference for the practical layer that sits between an agent and its context. Its mechanisms include project indexing, persistent session state, lifecycle hooks, handoff, pre-compaction preservation, context reduction, and token measurement.

In a larger environment, these capabilities could appear as agent-efficiency information attached to work items and projects:

```text
Task: implement X

Tokens              240k
Context saved        38%
Agent turns           31
Repeated reads         7
Tests                 18
Result             success
```

The interesting extension is correlating these measurements with outcomes rather than treating token savings as the objective by itself.

## cmux and Herdr

These should not be collapsed into the same capability.

### cmux

cmux is most naturally the live execution workspace: a place to organize running sessions, terminals, panes, notifications, and agent teams. It is the developer's view of work that is happening now.

### Herdr

Herdr is more naturally an execution/runtime control layer: persistent sessions, runtime state, agent integration, and programmatic control over the execution environment.

They may overlap, and Phase A should determine whether they are complementary or alternative implementations. Neither needs to become the knowledge/workspace layer.

## Direct agent APIs

Claude Agent SDK and Codex app-server introduce an important capability that is distinct from terminal/runtime management: direct programmatic interaction with the agent loop.

This can support operations such as:

- start or resume a session
- send work
- observe events
- continue a session
- interrupt when supported
- collect results
- make a new decision based on what happened

This is what makes it possible for a higher-level system to participate in agent execution without owning the agent runtime itself.

## Local and remote fleets

Spotify's direction suggests a second execution mode beyond the local developer workstation: fleets of background agents.

The key idea for a personal environment is not reproducing Spotify's infrastructure scale. It is making local and remote work appear through the same work/session/result model.

For example:

```text
Work item: migrate API

Local:
  Claude -> explore
  Codex  -> review

Remote:
  Claude x 50 -> repository migration

Results:
  42 complete
   6 waiting for review
   2 failed
```

The developer should be able to inspect, approve, redirect, or stop work without needing to know whether an agent is a local process, a remote worker, or a fleet job.

## What EOKS may actually add

This synthesis does **not** imply that EOKS should implement all of these capabilities.

The more interesting hypothesis is that EOKS could provide the connective layer around a common work model:

```text
Work
  ├── goal
  ├── project
  ├── relevant knowledge/context
  ├── agents
  ├── execution environments
  ├── permissions/policy
  ├── metrics
  ├── artifacts
  ├── decisions
  └── outcome
```

The EOKS questions then become practical:

1. What work is the developer trying to accomplish?
2. What knowledge/context is relevant right now?
3. Which agent or agents should work on it?
4. Where should they run: local or remote?
5. What tools and permissions should they have?
6. What should be observed and measured?
7. What evidence says the work succeeded?
8. What should be kept as useful knowledge afterward?

Existing systems can answer many of these questions individually. The open question is whether a common work model and coordination layer can connect them without replacing the systems that already do each job well.

## Important boundary

This is an architectural hypothesis, not a proposal for a monolithic platform.

EOKS should prefer integration over reimplementation:

- Obsidian can remain the workspace.
- OpenWolf can remain a context/efficiency mechanism.
- cmux can remain a live execution environment.
- Herdr can remain a runtime/session layer.
- Claude/Codex APIs can remain vendor-native agent interfaces.
- ACP can remain an interoperability boundary.
- Fleet infrastructure can remain a separate execution backend.

EOKS should only introduce new abstractions where the ecosystem does not already provide a useful capability and where experiments show that the missing connection materially improves software work.

## Research questions

### 1. Unified work model

Can one model represent a single local session, a local multi-agent team, and a remote fleet job without exposing implementation-specific details to the developer?

### 2. Workspace ↔ execution

Can an Obsidian-centered workspace launch and inspect local and remote work while preserving the existing strengths of cmux, Herdr, and native agent APIs?

### 3. Context ↔ outcome

Can OpenWolf-style context optimization be connected to actual task outcomes, so that the system learns which context reductions and additions were useful rather than optimizing tokens in isolation?

### 4. Fleet ↔ personal workflow

Can fleet-style background execution become a natural extension of a personal developer workflow rather than a separate enterprise control plane?

### 5. Permissions and guardrails

Can permissions follow the work item and agent role across local and remote execution?

### 6. Learning

Can useful decisions, evidence, failures, and outcomes flow back into the personal engineering workspace without turning the workspace into an automatically generated transcript dump?

### 7. EOKS boundary

Which of the coordination questions are already adequately solved by existing systems, and which remain genuinely open?

## Alternative interpretations

This direction could be wrong in several ways:

- A rich Obsidian-based environment may become too complex compared with a simpler IDE/harness workflow.
- cmux, Herdr, IDEs, and agent APIs may converge enough that an additional coordination layer is unnecessary.
- Fleet execution may remain mostly useful at organizational scale and add little to an individual developer.
- Existing agent platforms may absorb workspace, context, memory, evaluation, and orchestration capabilities.
- The unified work model may become an unnecessary abstraction if existing tools can already exchange the required information.

These are reasons to experiment rather than reasons to commit to the architecture now.

## Relation to current EOKS architecture

This direction is consistent with the existing EOKS boundary work:

- EOKS does not need to become another agent runtime.
- EOKS does not need to own terminal/pane management.
- EOKS does not need to replace vendor agent APIs.
- Context evolution remains about what useful work state survives and changes over time.
- Conductor/coordination remains a hypothesis to validate through experiments.
- Agent-loop mediation provides a possible direct control surface.
- Outcomes provide the feedback needed to decide whether a capability actually helped.

The new insight is that these mechanisms could eventually be presented to the developer as one expandable environment rather than as a collection of independent tools.

## Phase A follow-up

The next useful experiment is not to build the full environment. It is to connect a small number of existing pieces:

1. Use Obsidian as the project/workspace surface.
2. Use an existing agent runtime/harness for local execution.
3. Capture OpenWolf-style context/token measurements where available.
4. Use a direct agent API where available.
5. Represent the running work as a common work item.
6. Record outcome/evidence back into the workspace.
7. Add one remote/background execution path if practical.

The goal is to test whether the unified view and cross-system work model provide value before introducing new runtime infrastructure.
