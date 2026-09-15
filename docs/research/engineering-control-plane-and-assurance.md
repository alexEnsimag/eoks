# Engineering Control Plane and Assurance

## Status

Research synthesis / architectural hypothesis. This document extends the personal AI engineering environment research after PR #98. It does not commit EOKS to implementing a fleet platform, agent runtime, IDE, or observability system.

## Core question

How do we build an AI-native engineering system that continuously turns human intent into production-qualified outcomes while coordinating agents, resources, environments, evidence, and human attention?

The important shift is from **AI that writes code** to an **engineering system that continuously helps run and improve engineering work**.

## Emerging model

The environment can be understood as four cooperating capabilities:

1. **Personal engineering environment** — the work surface connecting intent, knowledge, agents, code, Git, CI, environments, production, and outcomes.
2. **Personal engineering assistant** — continuously understands the system, observes changes, investigates problems, proposes improvements, acts when allowed, verifies results, and learns.
3. **Fleet control plane** — schedules and coordinates agent workloads, resources, environments, dependencies, budgets, recovery, policy, and human attention.
4. **Semantic control / reconciliation** — a possible EOKS role: determine what outcome is desired, what evidence is sufficient, what should happen next, and what should be retained or learned.

The control plane and semantic control are related but should not be collapsed. The control plane operates execution; semantic control reasons about the meaning and desired state of work.

## 1. Personal engineering assistant

The assistant should not be limited to responding to explicit coding requests. It can continuously observe the engineering system and surface useful findings:

- recurring CI or test failures
- bugs reproducible in development environments
- deployment regressions
- runtime anomalies and incidents
- architectural duplication or drift
- infrastructure inefficiency or cost opportunities
- repeated manual work
- review and process bottlenecks
- opportunities for automation or new skills

A useful interaction is evidence-first:

> I noticed X. I investigated Y. The evidence suggests Z. I recommend A. Confidence: medium. I can implement and verify it if you want.

This creates a loop:

`observe -> assess significance -> investigate -> recommend -> act -> verify -> learn`

Proactivity must include an attention policy: not every observation deserves an interruption.

## 2. Engineering environment, not only coding environment

The execution surface should include development environments and operational evidence, not only an editor and terminal.

A workload may use:

- repository and code intelligence
- build and test systems
- ephemeral or persistent dev environments
- browsers and E2E systems
- Git and pull requests
- CI/CD
- cloud and infrastructure
- logs, metrics, and traces
- production and downstream systems

This enables a continuous chain:

`work -> agent -> code -> PR -> CI -> deployment -> runtime -> outcome -> learning`

Git/PR/CI therefore represent engineering state, not merely provenance.

## 3. Fleet control plane

Once many asynchronous agents operate concurrently, execution management becomes its own capability. Fleet management is not merely starting many agents; it is understanding collective state and changing execution in response.

A control plane may need to manage:

- workload scheduling and priority
- agent/model selection
- environment selection
- tool/capability availability
- budgets and capacity
- dependency health
- checkpointing and recovery
- lifecycle operations
- policy and permissions
- fleet observability
- human review/attention capacity

The important unit is **workload**, not agent process. Agents and runtimes can be replaced while the workload's durable state survives.

## 4. Shared dependencies and failure domains

Agents are connected to shared systems:

`work -> agent -> runtime/tools/environment -> GitHub/CI/cloud/credentials/human review`

The control plane should understand these relationships.

For example, if GitHub is degraded, a good system should recognize a shared dependency rather than treating every affected agent as an independent failure:

- continue work that does not require GitHub
- checkpoint work that cannot push
- queue PR/review operations
- avoid starting additional GitHub-dependent workloads
- resume and verify when the dependency recovers

This introduces **failure-domain awareness** and **blast-radius-aware recovery**.

## 5. Durable work state and recovery

Long-running engineering work must survive agent/runtime/dependency failures.

Prefer durable workload state over making a particular agent session authoritative:

`running -> interrupted -> checkpoint -> resume / retry / fork / redirect -> verify`

A checkpoint can contain:

- objective and policy
- current state
- changes/artifacts
- decisions
- relevant context
- evidence
- next-step hypothesis

The agent runtime/session remains replaceable.

## 6. Resource and capability selection

The system should select the resources appropriate to a workload rather than always using the same agent or model.

Selection can include:

- agent/runtime
- model
- environment
- tools/capabilities
- context
- budget
- autonomy level

Selection should consider objective, required capability, risk, availability, cost, expected quality, urgency, and current system state.

This is broader than model routing: it is **work/resource allocation**.

## 7. Assurance and autonomy

Autonomy should depend on risk and available evidence rather than being a global setting.

A simple progression is:

`low risk -> autonomous -> execute + verify -> prepare + ask -> human decision`

This extends the EOKS relationship:

`objective -> risk -> assurance -> autonomy`

Evidence can include tests, reviews, static analysis, environment checks, runtime telemetry, deployment results, and production behavior.

## 8. Reality is the final verifier

Recent research argues that agentic software has two persistent gaps:

- **requirement gap** — the stated requirements only approximate what stakeholders actually want
- **model gap** — tests and evaluation environments only approximate the real deployment environment

Therefore, passing a pre-deployment evaluator is not the same as producing an acceptable outcome. The useful loop is:

`requirements -> implementation -> evaluation -> deployment evidence -> revise requirements/model/evaluator`

This strengthens EOKS's existing outcome/evidence model: deployment and downstream behavior are part of assurance, not merely post-hoc monitoring.

## 9. Human attention as a resource

At fleet scale, human attention becomes another constrained resource.

The system should distinguish:

`observation -> significance -> attention -> intervention`

It should ask:

- Can the system resolve this itself?
- Is human judgment actually required?
- How urgent is it?
- Can it wait or be grouped with other events?
- What evidence should the human see?

The goal is not maximum autonomy or minimum notifications. It is **maximum useful engineering progress per unit of human attention**.

## 10. Governance and enforcement

As fleets become heterogeneous, policy cannot live only inside individual prompts or agent implementations.

A useful boundary is:

`agent intent/action -> policy enforcement -> execution -> evidence/audit`

Deterministic enforcement should handle things such as permissions, action boundaries, budgets, auditability, and shutdown/containment. Semantic control can decide what should happen, while enforcement makes sure the resulting action stays within policy.

This is consistent with emerging control-plane designs such as Microsoft Foundry Control Plane and OpenAgentFlow.

## 11. Outcome-level evaluation

Agent quality should not be measured only by tokens, turns, or whether code was generated.

The system should increasingly evaluate production-qualified engineering outcomes:

- task/workload success
- verification evidence
- rework and revert rate
- deployment success
- downstream/production behavior
- cost
- latency
- human review time
- human intervention rate
- blocked time
- recovery time
- unnecessary interventions

This suggests evaluating the complete engineering lifecycle rather than a one-shot agent task.

## 12. Industry evidence

The emerging model is supported by several complementary examples:

- **Spotify** — Portal demonstrates context/harness efficiency; Spotify Fleet Management demonstrates large-scale automated engineering maintenance; Honk extends fleet-wide automation with LLM-based changes.
- **Uber** — Software Factory demonstrates managed agents for code review, CI repair, E2E validation, alert triage, debugging and maintenance, with outcome-oriented economics.
- **Microsoft** — Foundry Control Plane explicitly provides fleet management, observability, lifecycle operations, policy and governance; Azure SRE Agent connects engineering agents with operational investigation and remediation.
- **Google** — Antigravity and related research show agent-native development environments and the importance of proactivity rather than autonomy alone.
- **OpenAI Symphony** — demonstrates the simple workload-to-agent control-plane model.
- **OpenHands** — provides an explicit Software Agent Control Plane framing.
- **cmux** — provides the human-facing execution/attention surface for many agents.
- **Herdr** — provides persistent runtime/session substrate.
- **OpenWolf** — provides agent-side context and lifecycle optimization.

These systems should be treated as evidence for capabilities and boundaries, not as a reason for EOKS to reproduce them.

## 13. EOKS boundary

EOKS should continue to avoid becoming another universal infrastructure platform.

Likely ownership:

- objective and policy interpretation
- workload state and reconciliation
- context/resource selection policy
- evidence and assurance policy
- semantic decisions about what should happen next
- durable outcome/evidence relationships
- evolutive context and learning
- autonomy/attention decisions

Likely delegated:

- agent runtime/session management
- terminal/workspace management
- Git hosting
- CI/CD
- dev environments
- production telemetry
- cloud infrastructure
- generic agent SDKs
- fleet infrastructure where an existing system already provides it

The architectural test remains:

> Can EOKS express the semantic decision without owning the execution substrate?

## 14. North-star loop

The resulting model is:

```text
                         HUMAN
                           |
                      intent/judgment
                           v
                  PERSONAL ASSISTANT
                           |
                    semantic intent
                           v
                  +----------------+
                  | EOKS / semantic |
                  |    control      |
                  +-------+--------+
                          |
                    policy / decision
                          v
                  FLEET CONTROL PLANE
                          |
             +------------+------------+
             v            v            v
          Agent A      Agent B      Agent C
             |            |            |
             +------------+------------+
                          v
                 ENGINEERING SYSTEM
              Git / CI / Dev / Cloud / Prod
                          |
                          v
                       EVIDENCE
                          |
                          v
                        OUTCOME
                          |
                          v
                       LEARNING
                          |
                          +------> next work
```

The core EOKS control loop remains:

`intent -> policy -> select context/resources -> execute -> observe -> decide -> verify -> outcome -> learn -> reconcile`

The new research adds an important distinction: **execution control can operate the fleet, while EOKS can remain the semantic reconciler for the work.**

## 15. Remaining research questions

Broad ecosystem collection should now stop. The remaining questions are architectural:

1. What exactly belongs in the semantic control plane versus fleet infrastructure?
2. What is the minimal durable workload/checkpoint representation?
3. How should dependency graphs and failure domains be represented?
4. How should assurance evidence accumulate across development, deployment, and production?
5. How should autonomy and human attention budgets interact?
6. What outcome metrics best represent production-qualified engineering value?

These questions are better candidates for experiments than another broad survey of agent products.
