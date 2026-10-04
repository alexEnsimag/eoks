# EOKS — Six-Dimensional Control Model

## Status

Working architectural hypothesis, not a final EOKS specification. This consolidates the recent personal AI engineering environment research into a testable control-loop model.

## Model

EOKS can be modeled through six semantic dimensions:

```text
                         INTENT
                       what / why
                           |
                           v
                    desired situation
                           |
              +------------+------------+
              |                         |
              v                         v
          KNOWLEDGE                 WORKFLOW
       what we know             how work progresses
              |                         |
              +------------+------------+
                           |
                    CAPABILITIES
                    what can act
                           |
                         POLICY
                   what may happen
                           |
                           v
                       EXECUTION
                           |
                           v
                         STATE
                    what is true now
                           |
                           v
                        EVIDENCE
                  what actually happened
                           |
                           v
                        OUTCOME
                    did it work?
                           |
                           v
                      KNOWLEDGE
                           |
                           +------> next cycle
```

The six dimensions are semantic responsibilities, not necessarily six EOKS components. Existing systems can provide the mechanisms. Execution, evidence, and outcome are shown in the loop because they are lifecycle mechanics through which the semantic dimensions are reconciled.

A useful distinction is **desired state versus observed state**: intent expresses what should become true, while state records what is currently observable. Workflow and policy determine how the system may move between them.

### Intent

The desired outcome, motivation, acceptance criteria, constraints, and non-goals. Intent should survive individual agent sessions; a prompt is not automatically durable intent.

### Knowledge

Project facts, architecture, decisions, conventions, prior experience, assumptions, and derived representations. Provenance and freshness matter; generated summaries are not automatically truth.

Knowledge should also preserve enough lineage to answer why a fact or decision is believed, what evidence supports it, and when it may have become stale. A useful current synthesis does not require collapsing all history into one canonical representation.

### Workflow

The progression of work: investigation, implementation, validation, review, deployment, recovery, and related stages. Generic automation is one implementation mechanism, not the definition of workflow.

Workflow should describe semantic progression rather than force EOKS to own process execution. Existing workflow engines, agent loops, CI systems, and automation platforms may provide the mechanics.

### Capabilities

Actions available to the environment: read/write code, run tests, create branches, call APIs, deploy, query telemetry, invoke another agent, or trigger an external workflow. Capabilities should remain composable and provider-owned where possible.

Capability selection is itself a control question: given the work, which agent, runtime, context, tools, environment, budget, and assurance level should be used? A universal router is not assumed; measured benefit should justify additional allocation machinery.

### State

The current observable situation: work phase, changed files, branch/PR status, CI status, deployment state, blockers, pending decisions, active agents, and relevant external conditions. State is not the same as knowledge: knowledge may describe architecture while state says that the current branch has an unreviewed change and failing integration tests.

The Kubernetes/control-loop analogy is useful because desired state and observed state are distinct and repeatedly reconciled. EOKS should borrow that distinction without becoming a Kubernetes-like runtime.

State should be able to represent interruption and recovery without losing semantic continuity: waiting, blocked, failed, checkpointed, resumed, retried, forked, or redirected are meaningful work states, not necessarily reasons to restart from scratch.

### Policy

Constraints on capabilities and transitions: permissions, approval requirements, evidence requirements, budgets, autonomy limits, risk constraints, and forbidden actions.

```text
Capability = can the environment do this?
Policy     = may it do this now, under these conditions?
```

Deterministic enforcement should remain close to execution where possible; semantic control should determine which policy-relevant situation applies. Policy therefore connects risk and assurance to allowed autonomy rather than being only an access-control list.

## Implementation mechanisms: what each dimension looks like in practice

The six dimensions are semantic responsibilities, not six services. This pass maps each one to concrete mechanisms already appearing in the ecosystem. The goal is to identify what EOKS should coordinate versus what it should consume from existing infrastructure.

| Dimension | Concrete mechanisms / prior art | Typical implementation technique | EOKS role |
| --- | --- | --- | --- |
| **Intent** | Work items, project/task objects, acceptance criteria, durable instructions | versioned structured state, human edits, links to artifacts/evidence | preserve intent across sessions/providers and evaluate outcomes against it |
| **Knowledge** | Git/docs, OpenWolf, Hindsight, Mem0, GraphRAG, OKF | documents + metadata, temporal/entity memory, graph/index retrieval, provenance | manage promotion, freshness, contradiction, supersession and selection |
| **Workflow** | Temporal, LangGraph, n8n, agent SDKs | durable state machines, checkpoints, events, retries, signals | represent semantic progression and choose next actions without owning execution |
| **Capabilities** | MCP, IDE APIs, agent tools, Sourcegraph MCP | discoverable schemas, protocol adapters, provider-owned tools | select the minimum sufficient capability set under policy and cost |
| **State** | runtime sessions, LangGraph checkpoints/stores, Git/CI/PR state | events + durable projections + checkpoints + external state adapters | maintain semantic work state across independent systems |
| **Policy** | OPA, Cedar, guardrails, tool permissions, human approval | policy-as-code, authorization decisions, approval/admission gates | express work-level autonomy, risk and evidence requirements; leave enforcement to providers |
| **Context** | Aider repo map, Sourcegraph, OpenWolf, RAG, memory systems | retrieval + ranking + structural lookup + token budgeting + compilation | compile a workload-specific working set from heterogeneous resources |
| **Observability** | OpenTelemetry GenAI, Langfuse, Phoenix, LangSmith | traces/spans, events, metrics, structured model/tool telemetry | consume observations and interpret them semantically rather than replace telemetry |
| **Evidence / evaluation** | tests, CI, static analysis, evaluators, trajectory evaluation | validators + evaluation events + provenance/lineage | determine what evidence is sufficient for a claim or transition |
| **Learning** | Hindsight, OpenWolf, skills/hooks, agent-learning systems | retain/recall/reflect, extraction, candidate procedures, background processing | control promotion from experience into knowledge, skills, policy or context rules |
| **Outcome** | tests, PR/review, deployment/runtime checks, benchmarks | acceptance predicates + linked artifacts + verification evidence | compare actual state with intent and retain the resulting evidence/lesson |

### Intent: durable desired state

Implementation usually starts with a durable Work object rather than a prompt: objective, acceptance criteria, constraints, non-goals, risk and desired outcome. Existing issue/project systems and agent platforms already provide much of this storage mechanism.

EOKS does not need another task database. The interesting requirement is that intent survives agent/session/runtime changes and remains the reference point for evaluation.

### Knowledge: storage plus lifecycle

Current implementations use several mechanisms: Markdown/Git and portable knowledge bundles; structured memory; entity/temporal memory; graph representations; hybrid search indexes; and procedural knowledge such as Skills. Hindsight is especially relevant because it separates retain, recall and reflect instead of reducing memory to top-k retrieval. Mem0 provides another concrete pattern: extract and consolidate salient information, then retrieve it later rather than replaying the whole history. citeturn0academia8turn0academia7

The EOKS mechanism is therefore a lifecycle:

source/observation -> candidate knowledge -> provenance/validation -> promote/retain/reject -> retrieve -> observe outcome -> supersede/revise/invalidate.

### Workflow: use durable execution, do not recreate it

Temporal provides durable workflow state, timers, signals, retries and recovery. LangGraph provides graph execution with checkpoints and persistent stores. n8n provides integration-oriented deterministic workflows around AI agents. Agent SDKs provide provider-specific loops and tool orchestration.

These mechanisms answer how to execute a known process reliably. EOKS should instead decide whether the current phase is complete, whether more evidence is needed, whether context should change, or whether work should continue, retry, delegate, wait or escalate. Those decisions can invoke an existing workflow engine.

### Capabilities: provider plus protocol

MCP demonstrates a general mechanism for exposing capabilities through schemas and a protocol. IDE APIs, cloud APIs, databases and code-intelligence services use similar provider boundaries.

The EOKS-specific mechanism is selection: work + policy + state produce candidate capabilities, which can be filtered by availability, trust, cost and expected value. This is different from building another tool registry.

### State: events plus durable projections

A practical pattern is: events/observations -> state projection -> durable semantic state -> reconciliation -> new actions.

LangGraph's separation of checkpoints from longer-lived stores is a useful implementation reference. Git, CI, PR and deployment systems provide additional external state sources. EOKS therefore needs semantic projections and adapters more than another generic database.

### Policy: decision engine plus enforcement point

OPA and Cedar demonstrate the established pattern: request + context -> policy engine -> decision -> enforcement point. OPA separates policy decisions from distributed enforcement and supports decision telemetry; Cedar separates authorization logic from application business logic. citeturn0search5turn0search6

For EOKS, the higher-level extension is semantic policy: work + risk + state + evidence -> continue / verify / approve / restrict / stop. Enforcement should remain close to the action: runtime permissions, tool authorization, cloud IAM, GitHub permissions and CI gates.

### Context: a compiler pipeline, not a database

Current context systems make the implementation pattern increasingly clear: acquire candidates from knowledge, tools, history and evidence; rank/filter them; manage a token budget; build a task-specific working set; then compile it into the model/harness representation.

Aider's repository map, Sourcegraph's code-aware retrieval and OpenWolf's lifecycle context mechanisms implement different parts of this pipeline. Sourcegraph is especially useful because its MCP surface combines semantic/keyword retrieval with deterministic symbol and dependency information, while its 2026 context-engineering work treats retrieval quality and token budgeting as explicit engineering concerns. citeturn0search4

EOKS therefore does not need another retriever. Its potential role is deciding which context pipeline and working set the current Work requires.

### Observability: standardized telemetry plus semantic interpretation

OpenTelemetry is becoming a common representation layer for GenAI telemetry. Current conventions cover agent/workflow spans, model calls, tool execution, retrieval, memory operations and evaluation. citeturn0search0turn0search1turn0search2

The implementation pattern is agent/tools/runtime -> traces, events and metrics -> OTel/backend. EOKS should consume these observations rather than replace the telemetry pipeline.

The missing semantic step is interpretation: what does an observation imply for this Work? Repeated retrieval failures, for example, may indicate a context problem rather than an agent problem. That interpretation belongs above raw observability.

### Evidence and evaluation: validators plus provenance

Software engineering already has strong external evidence mechanisms: tests, type checks, static analysis, CI, review, deployment and runtime behavior. Agent evaluation adds trajectory, tool-call and context-level evaluation.

Evidence should remain typed and provenance-linked rather than being immediately collapsed into one confidence score. EOKS can then ask: what claim are we establishing, what evidence is sufficient, which evidence is authoritative, and what is missing?

### Learning: asynchronous extraction and controlled promotion

A strong implementation pattern from OpenWolf, Hindsight and skills/hooks is: execution -> hooks/events -> trace; then, in background, trace -> episode/experience -> pattern/candidate learning -> validation -> promotion.

Hindsight provides retain/recall/reflect as a concrete persistent-memory mechanism, while OpenWolf demonstrates lifecycle hooks and project state around an existing coding agent. The EOKS-specific mechanism is the promotion gate: observations and memories should not automatically become trusted project rules. citeturn0academia8

A useful learning record contains situation, action/strategy, evidence, outcome, scope, confidence, provenance and validity. Promotion can require repetition, successful outcomes, absence of strong counterexamples, appropriate scope and/or human approval.

### Outcome: predicates over real-world state

An agent finishing a session is not an outcome. An outcome should be evaluated through predicates over observable state and evidence:

intent -> acceptance criteria -> evidence providers -> predicate evaluation -> accepted / rejected / uncertain.

For coding work, these predicates may include tests, static checks, review state, deployment health and downstream behavior. Where possible, outcome evaluation should be external to the agent's self-report.

## Mechanism boundaries

The resulting split is:

```
EOKS semantics
  intent
  knowledge lifecycle
  context selection
  semantic policy
  evidence sufficiency
  outcome interpretation
  learning / promotion
  cross-system work state

Existing mechanisms
  OTel / Langfuse / Phoenix
  OPA / Cedar
  MCP
  Temporal / LangGraph / n8n
  Hindsight / OpenWolf
  Sourcegraph / Aider
  Git / CI / validators
  runtimes / agent SDKs
```

The architectural test is simple:

> If a mature external mechanism already solves the mechanics, EOKS should integrate with it and own only the semantic decision that is missing across providers.

## What this changes in the six-dimensional model

The dimensions should not be interpreted as six EOKS services:

- **Intent** -> durable desired state
- **Knowledge** -> persistent representations plus lifecycle
- **Workflow** -> semantic progression over execution mechanisms
- **Capabilities** -> provider-exposed actions
- **State** -> semantic projection over observed systems
- **Policy** -> constraints plus autonomy decisions

Cross-cutting lifecycle mechanisms are:

- **Context** -> working-set compilation
- **Observability** -> observations
- **Evidence** -> validation and provenance
- **Outcome** -> acceptance predicates
- **Learning** -> promotion and change

This is a mechanism map, not a proposed EOKS component diagram. It gives Phase A experiments concrete implementations to plug together and compare while preserving the hypothesis that EOKS is a semantic control layer rather than another collection of infrastructure services.

## Work as the common unit

The six dimensions become operationally useful when attached to a common unit of **work** rather than to an IDE, agent, session, or dashboard.

A work item can carry:

- objective, motivation, acceptance criteria, and non-goals;
- constraints, permissions, risk, and assurance requirements;
- selected context, resources, capabilities, and execution environment;
- one or more agent sessions or human interventions;
- semantic and engineering-state transitions;
- artifacts, observations, and evidence;
- verification, review, and attention events;
- outcome, cost, and remaining uncertainty;
- lessons or durable knowledge candidates.

This allows local single-agent execution, local multi-agent execution, and remote/fleet execution to be execution modes of the same work model rather than separate product concepts.

The lifecycle may look like:

```text
intent
  -> planned
    -> delegated
      -> executing
        -> waiting / needs attention
          -> verifying
            -> review
              -> accepted / rejected
                -> completed
                  -> learned
```

Failure and interruption should preserve the work item:

```text
executing
  -> interrupted / failed
    -> checkpoint
      -> resume / retry / fork / redirect
        -> executing
```

This is a semantic lifecycle model, not a claim that EOKS should implement a new runtime. Existing runtimes and workspaces can own process/session mechanics while EOKS reasons about semantic state and the next decision.

## Execution environment as a realization boundary

DSec provides concrete systems evidence for a distinction already implicit in the six-dimensional model: **execution capability is not the same thing as the environment that realizes it, and neither is identical to semantic Work state**.

A Work may require a capability such as repository execution, but that capability can be realized by different execution environments—local process, container, microVM, VM or remote/fleet execution—depending on functionality, isolation, resource and policy requirements. DSec also demonstrates that environment state can remain meaningful while compute resources are reclaimed or replaced.

Therefore:

```text
CAPABILITIES -> what can/may be done
POLICY       -> under what constraints
LOADOUT      -> which resources/environment are eligible
EXECUTION    -> concrete realization
STATE        -> what remains true across lifecycle changes
```

This does not add a seventh dimension. It sharpens the boundary between the six semantic dimensions and the implementation substrate beneath them.

See [DeepSeek Elastic Compute prior art](../../research/prior-art/deepseek-elastic-compute-2026.md).

## Execution, evidence, and outcome

Execution, evidence, and outcome are better treated as lifecycle mechanics than additional semantic dimensions:

```text
Intent
  |
  v
plan / decide
  |
  +---- Policy + Capabilities ----+
  |                               |
  v                               v
execute ---------------------> observe
                                  |
                                State
                                  |
                               Evidence
                                  |
                               Outcome
                                  |
                                  +----> Knowledge
```

Evidence should not be treated as equivalent to agent narration. A transcript can explain what an agent claims to have done, while Git state, CI results, review decisions, deployment observations, benchmarks, and downstream behavior may provide stronger evidence of what actually happened.

The outcome should be evaluated against the original intent and acceptance criteria, not only against whether an agent session completed successfully.

## Assurance and autonomy

More autonomy increases the importance of evidence, not just execution speed. The environment should connect:

```text
objective -> risk -> assurance requirement -> autonomy allowed
                         |
                  evidence / verification
                         |
                       outcome
```

Low-risk work may continue with lightweight verification. Higher-risk work may require stronger tests, independent review, human approval, or restricted permissions. This extends EOKS's existing objective → risk → assurance → autonomy direction into the personal environment.

## Human attention

Human attention is another constrained resource in the environment. OpenWolf/Portal-style mechanisms optimize what reaches the **agent**; cmux-style mechanisms can optimize what reaches the **human**. These are two sides of the same context problem.

```text
agent-side optimization              human-side optimization
what information is useful?          what deserves attention?
what context should be retained?     what requires intervention?
what can be compressed?              what can be suppressed?
```

EOKS should therefore evaluate not only token/context efficiency but also attention efficiency. Autonomous throughput is only useful if human cognitive load does not grow proportionally.

## AI engineer environment

The environment should be treated as replaceable providers:

| Environment component | Primary contribution | EOKS relationship |
| --- | --- | --- |
| IDEs / editors | human context and control | intent, inspection, intervention |
| Coding agents | agent execution | workflow, capabilities, execution |
| Skills / hooks | lifecycle and harness extension | workflow, policy, context selection |
| OpenWolf / Portal-like mechanisms | context/session optimization | context and execution evidence |
| cmux / terminal workspaces | execution and attention surface | state, execution, attention |
| Herdr / runtimes | agent/process lifecycle | execution and workload state |
| Git / PR / CI | engineering state and verification | state, evidence, outcome |
| n8n / workflow automation | external workflows and integrations | workflow and capabilities |
| Obsidian / knowledge workspaces | human-facing knowledge | knowledge and intent |

This follows the existing personal AI engineering environment synthesis: EOKS should not own IDEs, runtimes, terminals, Git forges, notification surfaces, agent SDKs, or knowledge stores. It should coordinate semantic state and decisions across them.

## n8n: adjacent, not foundational

n8n is useful prior art for the Workflow + Capabilities dimensions. Its current AI platform combines AI agents with deterministic workflow logic, integrations, monitoring, guardrails, human approval, and MCP. It can therefore be consumed as a workflow/capability provider rather than something EOKS needs to replace.

The EOKS question is not "should we build an n8n competitor?" but "how should EOKS coordinate workflow capabilities such as n8n?"

An automation workflow can execute a sequence without necessarily owning durable intent, unified engineering state, evidence sufficiency, evolving knowledge, or outcome-driven learning. Those remain the potential EOKS gap.

## IDEs and harnesses

Recent IDE/agent systems reinforce the need to keep EOKS above the harness/runtime boundary. Cursor exposes lifecycle hooks around sessions, tool calls, subagents, file access, MCP, compaction, responses, and cloud agents. JetBrains Junie provides multi-step execution, external tools, CLI access, skills, and IDE/terminal surfaces.

These mechanisms are prior art for integration, but they argue against creating a new universal EOKS harness. The semantic control should work across multiple execution surfaces.

## What EOKS should not become

This model does not imply building:

- another IDE;
- another agent runtime;
- another workflow automation platform;
- a universal MCP server;
- a mandatory knowledge graph;
- a mandatory vector database;
- a universal model router;
- replacements for Git, CI, deployment, or observability systems.

Existing research should remain intact. New mechanisms should only be introduced when experiments show that an existing boundary cannot express the required semantic behavior cleanly.

## Prior-art research map

| Dimension | Prior-art families | Key question |
| --- | --- | --- |
| Intent | requirements, declarative systems, planning | how is desired outcome represented and evolved? |
| Knowledge | databases, Git, provenance, memory systems | how is useful knowledge preserved without treating every derivation as truth? |
| Workflow | workflow engines, compilers, n8n | how is progression represented without coupling to one runtime? |
| Capabilities | operating systems, APIs, capability security | how are actions exposed and constrained? |
| State | Kubernetes, distributed systems, VCS | how do desired and observed state interact? |
| Policy | authorization, OPA, admission control | which transitions are allowed and who enforces them? |
| Cross-cutting | observability, CI, provenance, evaluation | what evidence is sufficient to establish an outcome? |

The research question is therefore broader than finding an "AI second brain": what mechanisms do mature systems provide for intent, knowledge, workflow, capabilities, state, policy, evidence, and outcome, and where are those abstractions insufficient for AI-driven engineering?

## Experimental direction

The next step should be an experiment rather than another broad tool-collection pass:

1. Represent one real engineering task as a durable work item with intent, acceptance criteria, risk, and assurance requirements.
2. Run it through an existing agent/harness rather than an EOKS-owned runtime.
3. Capture semantic state transitions and evidence from existing Git/CI/harness mechanisms.
4. Test interruption and recovery through checkpoint/resume or fork/retry rather than restarting the work from scratch.
5. Apply one semantic intervention where EOKS chooses whether to continue, request attention, change context, or change execution resources.
6. Measure both agent-side context efficiency and human attention/intervention cost.
7. Evaluate the result against the original intent rather than only agent/session success.
8. Feed the outcome back into knowledge/context evolution.
9. Compare with the same task executed without the semantic control layer.

The falsifiable hypothesis is:

> A thin semantic/control layer can improve long-horizon engineering outcomes by making intent, state, policy, evidence, and outcome explicit across otherwise independent execution systems, without requiring EOKS to replace those systems.

If the experiment does not demonstrate that benefit, the model should be revised rather than implemented by assumption.

## Relationship to existing research

This document builds on, rather than replaces:

- `docs/research/personal-ai-engineering-environment.md`
- `docs/research/personal-ai-engineering-environment-synthesis.md`
- `docs/context-evolution.md`
- `docs/evaluation.md`
- `research/system-representation-gap.md`
- the Phase A tool landscape and harness/context research.

The six-dimensional model is a synthesis layer connecting those existing lines of work, not a reason to remove their detailed research or collapse them into one implementation abstraction.
