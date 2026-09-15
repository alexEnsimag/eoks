# Herdr and cmux: agent runtime versus harness surface

This note records the September 2026 investigation of [Herdr](https://herdr.dev/) and [cmux](https://cmux.com/) and its relevance to EOKS.

## Why this matters

Herdr initially looks like another terminal multiplexer, but its architecture exposes a useful boundary for EOKS: the distinction between a **terminal/workspace surface**, an **agent runtime/control plane**, and the **semantic context/control layer** EOKS is trying to define.

cmux is also more than terminal organization. Its Claude Teams integration translates Claude Code's tmux-oriented team operations into native cmux workspaces/splits, while surfacing teammate state and notifications. Herdr instead treats the runtime itself as the durable substrate: a background server owns the real terminal processes, clients attach to it, and agents have first-class lifecycle state and automation APIs.

These are different emphases, not simply two competing terminal UIs.

## Herdr

Herdr describes itself as a terminal workspace manager/runtime. Its server owns real terminal processes, while attached clients provide the UI. It recognizes coding agents, tracks states such as `working`, `blocked`, `done` and `idle`, and exposes automation primitives for panes and agents.

Important capabilities for EOKS:

- persistent server-owned agent terminals independent of the UI client;
- agent detection and lifecycle/state information;
- structured workspace/tab/pane topology;
- APIs for reading panes, sending input, waiting, starting agents and coordinating work;
- agent-native operation: one agent can inspect or coordinate other agents;
- support for multiple agent CLIs rather than being tied to one model vendor;
- remote/SSH-oriented operation and multiple clients.

Herdr therefore supplies substantial **execution/runtime infrastructure** that EOKS should not reimplement.

## cmux

cmux is a terminal/workspace application with an increasingly capable agent-oriented control surface. The Claude Teams integration is particularly relevant: Claude Code's team-created tmux sessions/windows/panes are mapped into native cmux surfaces, with teammate metadata and attention/notification behavior.

For EOKS, cmux demonstrates an important pattern: an existing agent orchestration protocol can be adapted to a richer harness without requiring the harness to become the semantic knowledge layer.

cmux is therefore useful both as a practical harness for Claude Teams and as prior art for an **agent-aware workspace/control plane**.

## The EOKS boundary

The useful decomposition is:

```text
                         EOKS
          semantic context / knowledge layer
          ----------------------------------
          what should persist?
          what should evolve?
          what context is relevant now?
          what evidence/assurance is required?
          what should the next agent receive?
                         |
                  harness interface
                         |
          +--------------+--------------+
          |                             |
       Claude Teams                  Herdr
          |                             |
         cmux                       runtime
          |                             |
          +---------- agent CLIs ------+
```

The responsibilities should remain distinct:

| Layer | Primary responsibility |
|---|---|
| **cmux** | terminal/workspace surface, agent-team integration, notifications and interactive control |
| **Herdr** | persistent agent runtime, lifecycle/state, terminal topology and runtime automation |
| **Portal/Shunt** | harness-level context/I/O transformation, caching and reduction |
| **EOKS** | semantic knowledge, evolving context, context selection/compilation, policy, assurance and outcome-driven learning |
| **Conductor** | higher-level workflow policy: what should happen next, which roles/resources are needed, and when validation/escalation is required |

EOKS should consume capabilities and events from the runtime/harness rather than rebuilding them.

## Implication for evolutive context

This distinction strengthens the current EOKS hypothesis about evolutive context.

Runtime events such as agent completion, blocking, handoff, task boundaries and validation results are useful **semantic boundaries**. EOKS can use them to identify candidate durable knowledge—decisions, directions, constraints, rationale and outcomes—without treating every terminal transcript as permanent memory.

A possible loop is:

```text
runtime / harness events
        |
        v
meaningful boundary
        |
        v
candidate extraction
        |
        +----> durable knowledge / decision
        +----> experience / outcome
        +----> discard / transient context
        |
        v
next-workload context compilation
```

This is different from simply keeping sessions alive or extending a context window. Runtime persistence answers **where the work continues**; EOKS context evolution answers **what from that work should continue to matter**.

## Architectural conclusion

EOKS should **not** become another terminal multiplexer or agent runtime. It should integrate with these systems where useful.

In particular:

1. Keep cmux as a strong practical harness for Claude Teams rather than replacing it merely because Herdr exists.
2. Treat Herdr as important prior art for a first-class agent runtime/control plane and as a possible runtime integration target.
3. Keep Portal/Shunt in the context-transformation category rather than treating it as memory.
4. Put semantic context evolution, knowledge lifecycle, context selection, assurance and outcome learning above the runtime layer.
5. Let the Conductor use runtime primitives instead of owning terminal/process orchestration itself.

The resulting EOKS question becomes clearer:

> **Given the runtime state and available resources, what information should survive, evolve and reach the next agent, and what evidence is sufficient to trust the resulting workflow?**

## Evidence

- Herdr documentation describes the server/client runtime model, persistent terminal processes, agent states and agent automation primitives.
- Herdr's comparison explicitly positions itself as a runtime rather than an application, contrasting that with cmux as a terminal app.
- cmux's Claude Teams documentation describes the tmux compatibility layer that maps Claude's team operations into native cmux surfaces.

See the project documentation for current implementation details; this note captures the EOKS architectural interpretation rather than attempting to mirror either project's API documentation.
