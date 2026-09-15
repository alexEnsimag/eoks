# Personal AI Engineering Environment

## Status

Research direction / architectural hypothesis. This document records a synthesis of the current ecosystem; it is not a commitment to build a universal IDE, agent runtime, or orchestration platform.

## Motivation

Recent research suggests that many capabilities needed for AI-assisted software development already exist, but are split across different tools:

- **Obsidian** can provide a personal workspace for notes, projects, goals, research, decisions, and a connected knowledge graph.
- **OpenWolf** provides practical agent context optimization, lifecycle hooks, project/session state, handoff, and token/context measurement.
- **cmux** provides a live workspace for running and organizing multiple agent sessions, terminals, panes, notifications, and agent-specific integrations.
- **Herdr** provides a programmable execution/runtime layer for persistent agent sessions and runtime control.
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
| Proactive assistance | Notice relevant changes, synthesize what matters, suggest or initiate next work | Emerging proactive coding agents / assistants | Strong EOKS hypothesis |
| Workflow improvement | Learn repeated friction and propose changes to skills, context, tools, policies, or routines | Personalized skills research; workflow research | Strong EOKS hypothesis |
| Scheduled work | Run recurring synthesis, checks, reviews, or background work | Scheduled agents / automation | Integrate / experiment |

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

Action
  User approves the change.

Evaluation
  Future sessions require fewer repeated reads and preserve task success.
```

Or:

```text
Observation
  Review agents repeatedly perform the same checks after CI.

Suggestion
  Move those checks into the pre-review workflow.

Evaluation
  Compare review latency, failures, and unnecessary agent work.
```

This suggests a conservative improvement loop:

```text
observe
  -> detect pattern/friction
  -> form hypothesis
  -> suggest improvement
  -> approve or auto-apply if low risk
  -> evaluate outcome
  -> keep / revert / revise
```

The important boundary is that the system should not silently rewrite its own operating policy just because a model believes it found an optimization. Improvements should have evidence, provenance, an explicit risk level, and a measurable outcome.

Low-risk changes could eventually be automated. Higher-impact changes should remain proposals requiring human approval.

## What recent research adds

Several recent results strengthen this direction without proving the full architecture.

### Proactive coding agents

Google Research's 2026 work argues that coding agents are moving beyond autonomous execution toward proactive, long-horizon behavior: noticing relevant changes, connecting signals across tools, deciding when to interrupt, and carrying preferences across sessions. It proposes evaluating the quality of the agent's **insight policy**, including Insight Decision Quality, Context Grounding Score, and Learning Lift.

This is unusually close to the personal-environment hypothesis: the interesting capability is not another agent that can execute commands, but a system that can decide what matters next from accumulated evidence.

### Proactive planning and reflection

A 2026 CHI longitudinal study of a proactive planning/reflection agent found that users accepted, negotiated, corrected, and sometimes resisted proactive suggestions. It also identified failure modes including rigidity, premature turn-taking, and overpromising.

For EOKS, this suggests that proactivity needs its own evaluation signals and an explicit intervention policy. More interventions are not inherently better.

### Personalized developer skills

A 2026 empirical study proposes extracting reusable developer preferences from interaction histories so coding agents can adapt without changing model parameters. This supports the idea that repeated interactions can produce durable developer-specific behavior, but it does not establish that all preferences should become durable memory.

That distinction fits the existing context-evolution principle: preserve useful work state and validated preferences, not raw transcripts.

### Real-world coding-agent misalignment

A large 2026 observational study of 20,574 coding-agent sessions found recurring developer-agent misalignment around project understanding, intent, rules, action boundaries, implementation, and reporting. Most observed episodes imposed effort and trust costs, and visible resolutions frequently required explicit user correction.

This strengthens the case for learning from **corrections and outcomes**, not merely storing successful agent outputs. Repeated correction is itself a valuable signal that the environment, context, policy, or agent setup may need improvement.

### Workflow-level context management

A 2026 practitioner study of coding-agent workflows argues that context management is central and that upstream research/planning mistakes can compound downstream. It also identifies a lack of useful metrics for workflow effectiveness.

This reinforces a core EOKS hypothesis: context quality should ultimately be evaluated through workflow outcomes rather than context size or summary quality alone.

## Evaluation of proactive behavior

The personal environment should measure whether proactivity actually helps.

Potential signals include:

| Signal | Question |
| --- | --- |
| Insight quality | Was the surfaced issue/action genuinely useful? |
| Evidence grounding | Could the suggestion be traced to relevant observations? |
| Acceptance rate | Did the developer accept, reject, or ignore it? |
| Intervention cost | Did it interrupt or distract unnecessarily? |
| Action success | If executed, did the action produce the intended result? |
| Learning lift | Did feedback improve later suggestions or workflows? |
| Repetition reduction | Did the system eliminate repeated manual work or corrections? |
| Outcome impact | Did the intervention improve task success, latency, cost, or quality? |
| Trust | Did the developer become more or less willing to delegate? |

A useful anti-metric is **unnecessary activity**: an assistant that generates many suggestions, starts many agents, or performs many actions without improving outcomes is not becoming better.

This is consistent with the broader EOKS principle that downstream workload outcomes matter more than intermediate representation quality.

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
  ├── triggers / schedules
  ├── suggestions / interventions
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
9. Is there something the system should proactively surface or do?
10. Did that intervention actually improve the outcome?

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
- A scheduler can remain a generic trigger mechanism where possible.

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

Can useful decisions, evidence, failures, corrections, and outcomes flow back into the personal engineering workspace without turning the workspace into an automatically generated transcript dump?

### 7. Proactivity

Can the environment decide what is worth surfacing or doing without becoming noisy, interruptive, or overconfident?

### 8. Workflow improvement

Can repeated corrections, friction, cost, and successful patterns produce evidence-backed suggestions for changing the developer's setup?

### 9. Scheduled control

Can morning synthesis, recurring evaluation, and background maintenance be represented as scheduled workloads rather than separate assistant infrastructure?

### 10. EOKS boundary

Which of the coordination questions are already adequately solved by existing systems, and which remain genuinely open?

## Alternative interpretations

This direction could be wrong in several ways:

- A rich Obsidian-based environment may become too complex compared with a simpler IDE/harness workflow.
- cmux, Herdr, IDEs, and agent APIs may converge enough that an additional coordination layer is unnecessary.
- Fleet execution may remain mostly useful at organizational scale and add little to an individual developer.
- Existing agent platforms may absorb workspace, context, memory, evaluation, and orchestration capabilities.
- The unified work model may become an unnecessary abstraction if existing tools can already exchange the required information.
- Proactive assistance may create more interruption and trust cost than value.
- Automated setup improvement may overfit to short-term behavior or optimize proxy metrics such as token savings instead of engineering outcomes.

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
- Proactivity can be expressed as policy over observed work state rather than as a separate assistant runtime.
- Setup improvement can be expressed as a controlled reconciliation loop: observe, hypothesize, change, evaluate.

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
8. Add one scheduled **observation-only** morning synthesis that reports current work, unresolved decisions, active agents, and evidence-backed suggested priorities.
9. Add one periodic setup review that identifies repeated corrections, repeated context acquisition, or repeated workflow friction and produces suggestions without changing configuration automatically.
10. Measure acceptance, usefulness, intervention cost, and downstream outcomes before allowing autonomous changes.

The goal is to test whether the unified view, proactive layer, and cross-system work model provide value before introducing new runtime infrastructure.

## Selected recent evidence

- Google Research, *Agentic Coding Needs Proactivity, Not Just Autonomy* (2026): argues for proactive coding agents and an insight-policy framing with evaluation targets including Insight Decision Quality, Context Grounding Score, and Learning Lift.
- Abbas et al., CHI 2026, *Having Lunch Now: Understanding How Users Engage with a Proactive Agent for Daily Planning and Self-Reflection*: longitudinal evidence on acceptance, negotiation, correction, and failure modes of proactive agents.
- Huang, Du, Lan, *Do Personalized Skills Help Coding Agents? An Empirical Study of Developer Interaction Histories* (2026): studies extracting reusable developer preferences from interaction histories.
- Tang et al., *How Coding Agents Fail Their Users* (2026): observational study of 20,574 real-world coding-agent sessions, highlighting persistent misalignment and the cost of developer correction.
- Kapetanovic et al., *A Phased Workflow for Operating LLM-Based Coding Agents* (2026): practitioner evidence that workflow-level context management matters and needs better effectiveness metrics.

These results are evidence for research questions, not validation that the proposed personal AI engineering environment is the right architecture.