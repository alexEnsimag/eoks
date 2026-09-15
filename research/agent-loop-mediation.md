# Agent-loop mediation: model APIs as a control surface

This note records the September 2026 investigation into direct programmatic control of Claude and Codex agent loops and its implications for EOKS.

## Why this is different

Most of the EOKS prior art investigated so far falls into one of several categories:

- **harness mechanisms:** hooks, skills, terminal/workspace integration, agent runtime;
- **knowledge systems:** files, memory stores, databases, wikis and graphs;
- **context engineering:** acquisition, representation, retrieval, transformation, compilation and delivery;
- **execution/orchestration:** agents, delegation and workflow policy;
- **evaluation/observability:** traces, assurance and outcome signals.

Direct agent-loop mediation is a different boundary. Modern model vendors increasingly expose APIs and SDKs that let an application observe, initiate, resume, steer and evaluate agent work without controlling the agent through a terminal or maintaining a separate generic agent runtime.

The relevant EOKS question is therefore not simply **how do we call a model?** It is:

> **How can EOKS mediate an existing agent loop so that semantic context, policy, assurance and learning influence the work while the vendor/runtime continues to own model execution mechanics?**

## The emerging control surfaces

### Claude

Anthropic's Claude Agent SDK embeds the agent loop used by Claude Code into an application. It provides programmatic access to agent execution, tools, permissions, subagents and lifecycle events. Claude Code hooks provide another boundary around prompt, tool, permission, session and compaction events.

This gives EOKS two useful modes:

1. **Augmentation:** EOKS observes and influences an existing Claude Code workflow through supported hooks and integration points.
2. **Ownership:** an EOKS application runs the Claude Agent SDK directly and therefore owns the higher-level workflow while Claude's SDK owns the agent loop and execution mechanics.

The second mode is the cleaner architectural integration. It avoids pretending that an arbitrary Claude Code process is a universally interceptable service.

### Codex

OpenAI's Codex app-server exposes a programmatic control surface around Codex threads and turns. The protocol provides lifecycle operations such as starting, resuming, reading and forking threads, starting turns and consuming streamed events.

This makes Codex unusually interesting as an EOKS runtime target: an application can control agent work without a terminal multiplexer being the primary integration boundary.

The interface is still evolving, so EOKS should depend on a small adapter rather than treating every app-server capability as a stable cross-vendor abstraction.

### OpenAI Agents SDK / Responses API

OpenAI's Agents SDK provides agents, tools, sessions, handoffs, guardrails and tracing. It explicitly distinguishes the SDK from using the Responses API directly: the SDK manages the higher-level agent loop, while the Responses API is appropriate when an application wants to own the loop, tool dispatch and state handling itself.

This distinction is useful for EOKS. EOKS should not reproduce the SDK's agent-loop primitives merely to create a second generic agent framework. Instead, it can choose the appropriate level of control depending on the workload.

OpenAI's agent tooling also exposes useful control/evidence signals: sessions maintain working context, handoffs transfer work between agents, and tracing records model generations, tool calls, handoffs, guardrails and turn-level activity.

## Three control planes

The research clarifies three distinct layers:

```text
                         EOKS
                 semantic control plane
                 ----------------------
                 objective / context
                 policy / assurance
                 learning / evolution
                         |
              +----------+----------+
              |                     |
       Agent-loop API          Runtime API
              |                     |
     Claude Agent SDK        Herdr / cmux / ...
     Codex app-server               |
              |                     |
              +----------+----------+
                         |
                      agent
```

### Runtime control

Herdr and cmux operate around the running process/workspace:

- terminals and panes;
- process lifetime;
- workspace topology;
- runtime state;
- agent detection;
- low-level interaction.

This is the boundary described in [Herdr and cmux: agent runtime versus harness surface](prior-art/herdr-cmux.md).

### Agent-loop control

Claude Agent SDK, Codex app-server and related vendor APIs expose higher-level execution concepts:

- sessions/threads;
- turns;
- model interaction;
- tool execution;
- subagents/delegation;
- permissions/guardrails;
- lifecycle events;
- resume/fork/continuation;
- in some APIs, steering or intervention.

This is not terminal control. It is control over the **agent's execution loop**.

### Semantic control

EOKS should operate above both:

- what objective is being pursued;
- what context is relevant;
- which resources/agent/model are appropriate;
- what should happen next;
- when validation or escalation is required;
- what evidence is sufficient;
- what should become durable knowledge;
- what future work should inherit from the outcome.

## What EOKS should own

The new boundary suggests the following responsibilities for EOKS:

### Context selection and compilation

Select the minimum sufficient evidence/resources for the current task and compile them into agent-appropriate context.

### Semantic workflow policy

The Conductor decides what should happen next: direct work, delegation, parallelism, review, escalation, retry, continuation or termination.

### Agent/model selection

Select among available agent runtimes or models when the workload and assurance requirements justify doing so. This should be policy- and outcome-driven rather than a static model ranking.

### Agent-loop mediation

Observe agent-loop events and, where the underlying runtime supports it, influence the loop through supported mechanisms such as new turns, handoffs, steering, interruption, continuation or revised context.

### Assurance

Use observed work and deterministic validation/evaluation to decide whether to continue, review, escalate or accept.

### Context evolution and learning

Interpret workflow outcomes as evidence about what should persist, change or disappear from future context. A whole conversation should not automatically become durable memory.

## What EOKS should not own

The new boundary reinforces several exclusions already established in the Herdr/cmux research.

EOKS should not reimplement:

- terminal multiplexing;
- process/session infrastructure;
- generic agent loops;
- vendor-specific tool execution;
- generic handoff/subagent machinery already provided by an agent SDK;
- model API transport;
- vendor tracing infrastructure;
- generic conversation/session persistence when the underlying runtime already supplies it.

EOKS can consume these capabilities and add semantic policy around them.

## Conductor after this research

The Conductor boundary becomes more precise.

It is not a generic agent framework and not a terminal orchestrator. It is the semantic controller that decides **what should happen next** and expresses that decision through the available runtime/API capabilities.

```text
                 objective
                    |
                    v
              observe state
                    |
                    v
                Conductor
                    |
          +---------+---------+
          |         |         |
       continue  delegate   intervene
          |         |         |
       API turn  subagent   steering/
                          revised input
          |         |         |
          +---------+---------+
                    |
                    v
                 result
                    |
          +---------+---------+
          |                   |
       validate          learn/evolve
          |                   |
          +---------+---------+
                    |
                    v
              next decision
```

The Conductor therefore becomes a **semantic closed-loop controller**, not merely a planner.

## Evolutive context becomes a tighter loop

This architecture gives evolutive context a direct execution boundary:

```text
EOKS context/policy
       |
       v
agent turn
       |
       +---- model output
       +---- tool events
       +---- validation
       +---- handoff/delegation
       +---- runtime events
       |
       v
semantic interpretation
       |
   +---+---+---+
   |       |   |
 discard evolve persist
   |       |   |
   +---+---+---+
       |
       v
next-turn / future-work context
```

This is different from simply extending a context window or persisting a conversation. Vendor sessions answer how execution history remains available to the agent; EOKS decides what that history **means for future work**.

A useful design principle follows:

> **Session state is execution state; EOKS context is semantic state.**

The two may overlap, but neither should be assumed to be the other.

## Transparency: what is and is not practical

A universal transparent proxy underneath both Claude Code and Codex should not be assumed.

The supported integration boundaries differ:

```text
Claude Code CLI
    |
    +-- hooks / supported integration
    |
Claude Agent SDK
    |
    +-- application-owned agent loop

Codex
    |
    +-- app-server protocol
    |
    +-- application-owned thread/turn control
```

Therefore EOKS should support two operating modes:

### Assisted mode

The user continues using a normal agent CLI. EOKS observes supported lifecycle/hook signals and supplies semantic context or post-hoc learning where the integration permits it.

### Autonomous mode

EOKS owns the higher-level workflow and directly uses Claude Agent SDK, Codex app-server or another agent runtime. This provides much stronger control over context, delegation, intervention and evaluation.

The common abstraction should be deliberately thin. EOKS should normalize only the semantic operations it actually needs, for example:

```text
start(objective, context)
resume(session)
send(input)
steer(input)          # when supported
observe()
interrupt()           # when supported
wait()
result()
```

Vendor-specific capabilities should remain behind adapters rather than being flattened into an oversized EOKS API.

## Relationship to existing EOKS prior art

This mechanism is complementary to the existing layers rather than replacing them.

| Existing concept | Role relative to agent-loop mediation |
|---|---|
| **Portal / Shunt** | reduce/transform context around existing harnesses and I/O |
| **Herdr / cmux** | runtime and harness control |
| **Knowledge / wiki / graph** | optional durable resource representations |
| **Context engineering** | decide how resources become model input |
| **Conductor** | decide what should happen next |
| **Agent-loop mediation** | express semantic decisions through model/agent execution APIs |
| **Evaluation / assurance** | determine whether resulting work is trustworthy |
| **Learning / evolutive context** | update future semantic state from outcomes |

The important difference is that agent-loop mediation is **not another storage mechanism** and **not another harness**. It is a control boundary between semantic EOKS policy and an existing agent execution loop.

## Architectural hypothesis

The research supports a stronger EOKS hypothesis:

> **EOKS should mediate existing agent runtimes rather than build another one: observe execution, supply evolving semantic context, steer or delegate when appropriate, evaluate outcomes, and feed the resulting knowledge back into future work.**

A broader formulation is:

> **Given an objective, available knowledge, runtime state and model capabilities, EOKS should select, shape, steer, evaluate and evolve agent work—and determine what should survive to influence future work.**

This extends the Herdr/cmux boundary without collapsing EOKS into either a runtime or a generic agent SDK.

## Research conclusions

1. **The model/agent API is a genuine new EOKS integration boundary.** It is materially different from terminal/harness control and from persistent knowledge storage.
2. **Claude Agent SDK and Codex app-server make the idea practical today**, although their control surfaces differ and should be treated through adapters.
3. **EOKS should not reproduce vendor agent-loop infrastructure.** Sessions, turns, tools, handoffs, tracing and execution should remain with the underlying runtime where available.
4. **The Conductor becomes a semantic closed-loop controller.** It decides what should happen next and expresses that decision through agent/runtime APIs.
5. **Evolutive context gets a direct execution feedback loop.** Agent events and outcomes become evidence for deciding what context should persist or evolve.
6. **Universal transparent interception is not the right initial goal.** Support an assisted mode for existing CLIs and a stronger autonomous mode where EOKS owns the agent loop through supported SDK/API boundaries.
7. **The first implementation should be a thin adapter experiment**, not a new orchestration framework or storage system.

## Evidence and sources

Primary vendor documentation should be treated as the source of truth because these APIs are evolving quickly:

- Anthropic Claude Agent SDK documentation: agent-loop and lifecycle integration, hooks, sessions and subagents.
- OpenAI Agents SDK documentation: agents, sessions, handoffs, guardrails, tracing and the distinction between SDK-managed versus application-owned loops.
- OpenAI Codex app-server documentation/source: thread/turn protocol and application-facing Codex control.

The purpose of this note is to capture the architectural interpretation for EOKS; it should not attempt to duplicate vendor API references that change independently.
