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
