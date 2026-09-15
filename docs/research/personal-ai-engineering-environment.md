# Personal AI Engineering Environment

## Status

Research direction / architectural hypothesis. This document records a synthesis of the current ecosystem; it is not a commitment to build a universal IDE, agent runtime, or orchestration platform.

## Motivation

Recent research suggests that many capabilities needed for AI-assisted software development already exist, but are split across different tools:

- **Obsidian** can provide a personal workspace for notes, projects, goals, research, decisions, architecture, and history.
- **OpenWolf** provides practical agent context optimization, lifecycle hooks, project/session state, handoff, and token/context measurement.
- **cmux** provides a live workspace for running and organizing multiple agent sessions, terminals, panes, notifications, and agent-specific integrations.
- **Herdr** provides a programmable execution/runtime layer for persistent agent sessions and runtime control.
- **Claude Agent SDK** and **Codex app-server** expose direct programmatic control of agent sessions and their execution loops.
- **ACP** and IDE integrations provide a replaceable boundary between agents and development environments.
- **Code-intelligence systems** provide repository and software-system understanding.
- **Evaluation/observability systems** provide evidence about whether work succeeded and what it cost.
- **Spotify's fleet work** demonstrates that the same general model can extend from one local agent to large numbers of background agents working across repositories.
- **Cursor's Origin and cloud-agent direction** increasingly combine source control, agents, PRs, automations, and remote execution around the same codebase/work surface.

The hypothesis is that these capabilities could be composed into one expandable environment for an individual engineer rather than treated as isolated tools.

## Core idea

> A personal AI engineering environment gives one developer a unified place to understand their work, manage knowledge, run agents, manage local and remote execution, observe costs and outcomes, receive only the human attention requests that matter, and continuously improve how they work with AI.

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
| Agent fleets | Run many agents against many tasks/repositories and track exceptions | Spotify fleet direction; Cursor cloud agents | Integrate / experiment |
| Monitoring and observability | See live and historical agent activity, state, actions, cost, failures, and outcomes | Agent dashboards; runtime telemetry; traces | Integrate / correlate |
| Attention and intervention | Surface only events that require or materially benefit from human attention | cmux notifications; agent permission/idle events | Integrate delivery; potentially coordinate policy |
| Provenance | Navigate from code/work artifacts back to agent sessions, prompts, decisions, and evidence | Agent blame/provenance systems; Git metadata | Integrate / extend |
| Unified work tracking | Treat the goal, context, agents, execution, artifacts, metrics, and outcome as one piece of work | No clear dominant solution found | Strong EOKS hypothesis |
| Cross-system coordination | Connect workspace, context, agents, runtimes, permissions, evaluation, and history | No clear dominant solution found | Strong EOKS hypothesis |
| Learning from outcomes | Use successful/failed work to improve future context, choices, and workflows | Memory/self-evolving systems; emerging research | Strong EOKS hypothesis |
| Proactive assistance | Notice relevant changes, synthesize what matters, suggest or initiate next work | Emerging proactive coding agents / assistants | Strong EOKS hypothesis |
| Workflow improvement | Learn repeated friction and propose changes to skills, context, tools, policies, or routines | Personalized skills research; workflow research | Strong EOKS hypothesis |
| Scheduled work | Run recurring synthesis, checks, reviews, or background work | Scheduled agents / automation | Integrate / experiment |

## Agent-native source control: Cursor and the changing role of Git

Cursor's recent direction is important evidence that source control itself is becoming part of the agent environment rather than remaining a separate developer tool.

Cursor's **Origin** is an agent-oriented Git forge. Its current beta combines repositories, standard Git operations, code browsing/search, pull requests, and agents in one surface. Origin repositories can be connected to Cursor Automations and cloud agents; source-control events such as pushes and pull requests can trigger agent work. Cursor describes the system explicitly as a forge built for agent scale. The underlying Git infrastructure work is motivated in part by the increase in code, PRs, and CI activity caused by agents.

This is not evidence that EOKS should build another Git forge. It is evidence for a broader architectural observation:

> As agent concurrency and autonomy increase, Git, CI, PRs, agent execution, and codebase state increasingly form one operational system.

Cursor's longer-running-agent and self-driving-codebase research reinforces this trajectory: agents operate for longer periods, parallelize work, produce merge-ready changes, and increasingly need infrastructure for the volume of generated code and validation work.

The implication for the personal AI engineering environment is that **Git/PR/CI should be treated as part of the work and provenance model**, not merely as external links.

A useful evidence hierarchy is:

```text
Agent transcript
  -> what the agent said, considered, and tried

Agent session
  -> what actually happened during execution

Git / PR / CI
  -> what became repository state and how it was reviewed/validated

Production / downstream outcome
  -> whether that state actually worked
```

These sources are not interchangeable. A rejected idea in a transcript can be useful context; a merged commit is stronger evidence of accepted engineering state; verification and downstream outcomes can provide stronger evidence still. This reinforces EOKS's existing distinction between raw history, authoritative evidence, provenance, and outcome utility.

It also suggests that the causal provenance model should extend beyond the agent session:

```text
Work
  -> agent session
    -> change
      -> commit / PR
        -> CI / review
          -> merge / deployment
            -> downstream outcome
```

The environment should make these transitions navigable without requiring a separate provenance product.

## Monitoring is a first-class capability

The environment should not only **run** agents; it should make their work observable. Monitoring is the feedback surface between execution and control.

A useful distinction is:

```text
Live monitoring
  What is happening now?

Historical observability
  What happened, and why?

Outcome monitoring
  Did the work actually succeed?

System monitoring
  Is the environment itself healthy and efficient?
```

A unified monitoring view could show:

```text
AGENTS

Claude   EOKS research       running   14m   82k tokens
Codex    implementation      waiting   review needed
Claude   migration fleet     31/47     3 failed

CURRENT WORK

EOKS research
  agent: Claude
  phase: evidence synthesis
  current file: docs/research/...
  last action: searched prior-art
  context: 42k / 64k
  cost: $...

HEALTH
  repeated tool failures: 2
  context reacquisition: high
  pending verification: 1

OUTCOMES
  PRs: 3
  tests: 47/48 passing
  review blockers: 2
```

The important point is that this should not become another generic observability dashboard. Monitoring should connect directly to the work model: **what is happening, what it is doing, what evidence it has produced, what is blocked, and what decision should happen next**.

## Attention and human intervention

A particularly useful refinement from cmux is that monitoring is not only about seeing state. It is about **knowing when the human needs to pay attention**.

cmux provides notification rings, unread workspace indicators, a notification panel, desktop notifications, and agent-specific notification categories. Its built-in policy can notify when an agent is blocked waiting for permission, when a turn completes, or when an agent has been idle waiting for input. It can suppress completion noise while background work is still running, and notification hooks can filter or transform delivery.

This suggests a useful distinction:

```text
Observation
  What happened?

Significance
  Does it matter?

Attention
  Does the human need to know or decide something?

Intervention
  What decision/action is required?
```

An **attention event** can therefore be treated as a semantic signal without making it a new EOKS runtime primitive. Examples include:

- permission required
- clarification required
- agent blocked
- verification failed
- policy/risk threshold crossed
- conflicting agent results
- task completed and awaiting review
- an important unexpected outcome worth surfacing
- a proactive suggestion judged useful enough to interrupt

The control boundary can then look like:

```text
Agent / runtime
      |
      | events
      v
Observation
      |
      v
Attention policy
      |
   +--+----------------+
   |                   |
continue          human attention
                       |
                       v
                    decision
```

This is a better model than treating notifications as generic UI telemetry. The key question is **whether an event changes the need for human attention or intervention**.

### cmux vs EOKS

The boundary should remain consistent with the broader architecture:

```text
                    EOKS
                     |
              attention policy
                     |
        +------------+------------+
        |                         |
   semantic reason          delivery adapter
        |                         |
        +---------------------- cmux
                                  |
                         notification / UI
                                  |
                                human
```

cmux can remain responsible for delivering attention through its native workspace and notification mechanisms. EOKS, if it participates, would reason about **why** something deserves attention and what decision is needed; it should not own notification UI or desktop delivery.

This also creates an important evaluation dimension for a multi-agent environment:

- notification volume
- proportion of notifications that required action
- false/unnecessary notifications
- time to human awareness
- time to intervention
- human action taken
- work blocked while awaiting attention
- whether the agent could have safely continued without intervention

The goal is not to maximize notifications. It is to **scale autonomous throughput without scaling human cognitive load proportionally**.

## Provenance and the IDE's familiar mechanisms

A major opportunity is to take mechanisms developers already understand and reinterpret them for agent-mediated software development rather than inventing entirely new UX.

### Git blame → agent/session provenance

Traditional `git blame` answers roughly:

> Who last changed this line?

In an agent-native environment, a richer answer could be:

```text
src/context/compiler.go:184

Produced by
  Agent: Claude
  Session: 8f31...
  Work: EOKS context-evolution experiment
  Prompt / instruction: ...
  Commit: a91c...
  PR: #98

Evidence
  session transcript
  tool calls
  tests
  review

Human review
  Alex — approved / modified
```

The important UX is not a new provenance database exposed as a separate application. It is **clicking the familiar blame information and being able to navigate into the agent session that produced the change**.

This is now an emerging pattern in the ecosystem: tools such as Cursor's AI attribution and independent agent-provenance projects extend blame-like views to agent/model/session attribution. GitHub's cloud agent also links agent-authored commits to session logs. These examples validate the UX direction, while leaving open the more general EOKS question of how provenance should connect to work, evidence, decisions, and outcomes.

### Other traditional IDE mechanisms that can become agent-aware

The same principle applies broadly:

| Traditional mechanism | AI-native extension |
| --- | --- |
| `git blame` | Agent/session/model provenance |
| Git history | Work/session/evidence timeline |
| Diff view | Agent intent + tool/evidence + human changes alongside diff |
| Code lens | Agent/task/session status on symbols or files |
| Find references | Find references + related agent sessions/decisions |
| Go to definition | Jump from implementation to the work/decision that introduced it |
| Problems panel | Agent failures, unresolved verification, policy violations, stale context |
| Test runner | Test result + which agent/work item caused the change |
| Debugger | Navigate from failure to the agent session and relevant reasoning/evidence |
| TODO/FIXME | Link to work item, owner, agent, and proposed next action |
| Project tree | Live indication of files currently being worked on by agents |
| Search | Search code, knowledge, sessions, decisions, outcomes, and provenance together |
| Refactoring tools | Agent-assisted transformation with preview, provenance, and verification |
| Code review | Diff + agent execution evidence + validation + provenance |
| Terminal | Live agent/runtime session as a first-class workspace object |
| Notifications | Agent completion, blockage, risk, intervention request, or useful insight |
| Recent files | Recent work across human and agent sessions |
| Local history | Agent session checkpoints and recoverable work state |
| Breakpoints/watchpoints | Conditions that trigger agent observation or intervention |
| Project settings | Agent policies, context rules, permissions, and workflow configuration |
| IDE telemetry | Work effectiveness, agent behavior, context efficiency, and outcome metrics |

This suggests that the personal AI engineering environment may be less about inventing a completely new UI and more about **making existing engineering concepts agent-aware and connecting them through a common work/provenance model**.

### Provenance should preserve causality, not transcripts

The target should not be:

```text
line -> enormous transcript
```

It should be a navigable causal chain:

```text
Code line
  -> change
    -> commit / PR
      -> work item
        -> agent session
          -> instruction
          -> relevant context
          -> decisions
          -> tool/evidence events
          -> verification
            -> outcome
```

Each layer should expose enough evidence to understand the next layer without forcing the user to replay an entire session. Detailed transcripts can remain recoverable when needed.

This aligns with the existing EOKS principles of provenance, context evolution, and preserving the causal spine rather than accumulating raw history.

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
- monitoring and historical activity
- provenance links back into code

The graph can connect these objects over time. For example:

```text
Goal
  -> Project
    -> Work item
      -> Agent session
        -> Code change
          -> Review
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

cmux is most naturally the live execution workspace: a place to organize running sessions, terminals, panes, notifications, and agent teams. It is the developer's view of work that is happening now, including the **attention boundary** where an agent signals that the human needs to act.

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

Cursor's cloud-agent direction provides another concrete example: cloud agents can work against repositories, branch/commit/push, open pull requests, and run from source-control-triggered automations. This makes repository state, agent execution, and background scheduling increasingly interdependent.

The key idea for a personal environment is not reproducing this infrastructure scale. It is making local and remote work appear through the same work/session/result model.

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

## Proactive assistance and the personal-assistant layer

The environment does not have to remain purely reactive. A natural extension is for it to notice useful changes, synthesize what matters, and propose the next action.

This is not necessarily a separate "personal assistant" product. It can be viewed as another workload/control mode over the same work model.

### Morning synthesis

A scheduled morning workload could build a concise briefing from the current state of work:

```text
Good morning.

Priority work
  1. Review the EOKS research PR: one unresolved architectural question.
  2. Follow up on the migration: 6 repositories need review.
  3. Continue the current project goal: validation is the next missing step.

Since yesterday
  - background agent completed 12 repositories
  - one agent encountered the same failure twice
  - a new decision changed the preferred implementation

Suggested
  - spend 45 min on the review before starting new implementation
  - let a background agent investigate the repeated failure

Setup improvement
  - the same architecture context was reacquired in 3 sessions;
    consider making the compact architecture artifact part of the project context.
```

The important property is that this is synthesized from the same work, context, agent, outcome, and history model. It should not require a second memory system or task database merely to produce a briefing.

### Scheduled workloads

A scheduler could trigger different classes of work:

- morning work synthesis
- project status checks
- recurring validation/evaluation
- periodic review of agent costs and context efficiency
- detection of repeated failures or repeated manual corrections
- weekly workflow/setup review
- background research or maintenance

The scheduler is therefore a trigger for the existing control loop, not necessarily a new EOKS runtime primitive.

### Proactivity should be treated as a policy

A useful distinction is between **autonomy** and **proactivity**:

- autonomy: the agent can execute a task without step-by-step human control
- proactivity: the system decides that something is worth surfacing or doing before the developer explicitly asks

Recent research on proactive coding agents frames this as an "insight policy": decide what matters next, what evidence supports it, whether to surface it, and how to adapt after feedback. It also distinguishes reactive, scheduled, and situation-aware proactivity.

This maps naturally onto EOKS's existing policy/control-loop framing. The difficult problem is not simply generating a good morning summary; it is deciding **when an unsolicited intervention is useful enough to justify interrupting the developer**.

Attention policy is the closely related inverse problem: decide when an observed event is important enough to interrupt or request a human decision. Both should be evaluated by downstream utility and intervention cost rather than raw activity.

## Auto-learning and setup improvement

The strongest extension is not just remembering preferences. It is learning how the engineering environment itself could improve.

Examples:

```text
Observation
  Architecture context was reacquired 3 times.

Diagnosis
  Existing project context is too weak for this class of task.

Suggestion
  Add a compact architecture artifact to the project's durable context.

Approval
  Human approves the change.

Evaluation
  Later sessions require less reacquisition and produce equal or better outcomes.

Decision
  Keep the improvement.
```

The general loop is:

```text
observe
  -> detect pattern / friction
    -> form hypothesis
      -> suggest improvement
        -> approve / safely auto-apply
          -> evaluate
            -> keep / revert / revise
```

High-impact changes should require explicit human approval. Low-risk improvements could eventually be auto-applied when sufficient evidence exists.

The important point is that the environment learns **how to improve itself**, not merely what the developer likes.

Evaluation should include:

- insight quality
- evidence grounding
- suggestion acceptance
- intervention cost
- action success
- learning lift
- reduction in repeated friction
- downstream work/outcome impact
- trust
- unnecessary activity as an explicit anti-metric

## Research questions

The main research questions are now:

1. **Work model:** Can local agents, local multi-agent sessions, and remote fleets be represented as execution modes of one common work model?
2. **Context:** How should context selection and context evolution connect to work state, evidence, and outcomes?
3. **Runtime boundary:** Where should EOKS semantic control end and agent/runtime/harness mechanics begin?
4. **Agent-loop mediation:** When does direct programmatic control of an agent loop provide value beyond terminal/runtime control?
5. **Source control:** How should Git, PR, CI, deployment, and downstream outcomes participate in the same provenance chain as agent sessions?
6. **Monitoring:** What observations are actually useful for controlling or evaluating work rather than merely displaying telemetry?
7. **Attention:** What events genuinely require human attention, and how can notification volume stay below the value threshold as agent concurrency grows?
8. **Provenance:** Can familiar IDE mechanisms provide causal navigation from code to agent work, evidence, decisions, and outcomes?
9. **Proactivity:** When should the environment surface or initiate work without an explicit request?
10. **Learning:** Can repeated friction and outcomes safely improve context, skills, tools, policies, and workflows?
11. **Human load:** Can increasing autonomous throughput avoid a proportional increase in review, notification, and decision-making burden?
12. **Interoperability:** Can these capabilities remain replaceable across agents, runtimes, IDEs, context systems, and execution providers?

## Phase A follow-up experiment

A small experiment should validate the environment concept without building a new platform.

### A.1 Workspace

Use Obsidian as the human-facing workspace for:

- project/work items
- decisions
- research
- agent/session links
- outcomes
- selected monitoring/provenance information

### A.2 Agent execution

Use existing agent runtimes and harnesses:

- OpenWolf for agent context/token optimization and lifecycle mechanisms
- cmux for local multi-agent execution and human attention/notification handling
- direct Claude/Codex APIs where useful for programmatic control
- one remote/background execution path if practical

### A.3 Common work item

Represent one real task with:

- objective
- policy/permissions
- working context
- agent/session
- execution mode
- evidence
- attention events
- artifacts
- evaluation
- outcome

### A.4 Monitoring + attention

Build a minimal live view that answers:

- what agents are running?
- what are they working on?
- which are blocked?
- which need human attention?
- what evidence/outcomes have appeared?
- what can safely continue without the developer?

Use cmux's existing notification mechanism as the delivery baseline rather than building a new notification system.

Measure notification/attention quality separately from generic observability:

- attention events generated
- actionable vs informational events
- false/unnecessary attention events
- time to awareness
- time to intervention
- human decisions/actions
- blocked time
- autonomous continuation rate

### A.5 Provenance

Connect at least one traditional IDE mechanism to agent/work provenance, preferably a blame/history/diff path:

```text
code
  -> change
    -> work item
      -> agent session
        -> evidence
          -> verification
            -> outcome
```

### A.6 Scheduled/proactive workload

Implement or simulate one scheduled workload such as a morning synthesis, recurring project check, or repeated-failure detector. Evaluate whether its suggestions are useful enough to justify the attention they consume.

### A.7 Setup learning

Capture one repeated friction pattern and test the observe → suggest → approve → apply → evaluate loop for a low-risk environment improvement.

## Architectural hypothesis

The current evidence does **not** justify creating a dedicated "personal AI environment" runtime inside EOKS.

A stronger hypothesis is:

> EOKS can become the semantic connective/control layer that makes existing engineering environments, agents, runtimes, context systems, source-control state, evidence, attention, and outcomes participate in one coherent work model.

This preserves the existing EOKS boundary:

```text
             Personal AI Engineering Environment
                         │
       ┌─────────────────┼──────────────────┐
       │                 │                  │
   Workspace          Execution          Control
   / knowledge        / agents           / semantics
       │                 │                  │
   Obsidian          cmux / Herdr       EOKS
   Git / PR / CI     Claude / Codex     policies
   code intelligence local / remote      context
                                         evidence
                                         attention
                                         outcomes
```

The environment should therefore be evaluated as a **system of composable capabilities**, not as a single application that replaces all existing tools.

## Current synthesis

The emerging picture is not simply "build an AI IDE." It is:

> **Make the existing software-engineering environment agent-aware, connect its state through a common work/provenance model, and add semantic control over context, evidence, attention, and outcomes.**

Cursor demonstrates that source control, agents, PRs, cloud execution, and automation are converging. cmux demonstrates that once many agents run concurrently, the critical human UX is not just visibility but **knowing which agent needs attention and why**. OpenWolf demonstrates that context efficiency can be measured and optimized at the harness layer. Obsidian demonstrates a plausible human-facing knowledge/work surface. Direct agent APIs expose a control surface below the semantic layer. Spotify's fleet work demonstrates the scale direction.

The potentially differentiated EOKS contribution is therefore not another one of these subsystems. It is the connective model and control loop that answers:

```text
What work matters?
What context is sufficient?
What is happening?
What evidence do we have?
Does this require human attention?
What should happen next?
Did it work?
What should the environment learn from it?
```
