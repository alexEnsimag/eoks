# Engineering Control Plane and Assurance

## Status

Research synthesis / architectural hypothesis. This document extends the personal AI engineering environment research after PR #98. It does not commit EOKS to implementing a fleet platform, agent runtime, IDE, or observability system.

## Core question

How do we build an AI-native engineering system that continuously turns human intent into production-qualified outcomes while coordinating work, agents, resources, engineering state, evidence, and human attention?

The important shift is from **AI that writes code** to an **engineering system that continuously understands, operates, and improves engineering work**.

## 1. The emerging model

The research points to four cooperating capabilities:

1. **Personal engineering environment** — the work surface connecting intent, knowledge, agents, code, Git, CI, environments, production, and outcomes.
2. **Personal engineering assistant** — continuously builds an understanding of the engineering system, observes what is happening, investigates, identifies what matters, proposes improvements, acts when allowed, verifies results, and learns.
3. **Fleet control plane** — operates many asynchronous workloads: scheduling, allocating resources, monitoring dependencies, recovering from failures, enforcing policy, and managing capacity and human attention.
4. **Semantic control / reconciliation** — a possible EOKS role: determine the desired outcome, applicable policy, sufficient evidence, next semantic action, and what should be retained or learned.

These are related but should not be collapsed.

| Capability | Primary question |
|---|---|
| Personal assistant | **What is happening? Why does it matter? What should we do?** |
| Semantic control / EOKS | **What outcome and evidence should govern this work, and what should happen next?** |
| Fleet control plane | **How do we operate the execution reliably at scale?** |
| Engineering environment | **Where can the work be observed, changed, verified, and operated?** |

The distinction matters because EOKS should not become another runtime or fleet infrastructure. The assistant provides engineering intelligence and judgment; the fleet control plane operates execution; EOKS is a candidate semantic reconciler between intent, policy, evidence, and outcomes.

## 2. The personal engineering assistant is an engineering intelligence layer

The assistant should not be limited to responding to explicit coding requests. It should continuously build and maintain an understanding of the engineering system across:

- code and architecture
- infrastructure and environments
- Git and pull requests
- CI, tests, and linters
- reviews and engineering process
- deployments
- logs, metrics, and traces
- production behavior and incidents
- historical changes and outcomes
- recurring manual work and operational friction

This enables the assistant to notice opportunities and problems that are not expressed as tasks:

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

> I noticed X. I investigated Y. The evidence suggests Z. I recommend A. Confidence: medium. I can implement and verify it if allowed.

The resulting assistant loop is:

`observe -> assess significance -> investigate -> recommend -> act -> verify -> learn`

This is broader than a coding agent: the assistant can investigate and improve the engineering system itself.

### Proactivity is an attention policy

Continuous observation does not mean continuous interruption. The assistant must decide whether an observation is significant enough to surface, whether it can resolve it autonomously, whether it can wait or be grouped, and what evidence a human needs.

This makes **attention selection** part of the control problem rather than merely a notification feature.

## 3. Engineering state must be continuously monitored

The engineering environment is not only where code is edited. It is a set of changing states and workflows that the system can observe and reconcile.

Important signals include:

- issues and work items
- branches and commits
- pull requests
- CI and test runs
- linters and static analysis
- code-review requests, comments, approvals, and changes
- merge readiness and blockers
- deployments
- logs, metrics, and traces
- production/downstream behavior

A typical lifecycle is:

```text
work
  -> change
  -> PR opened
  -> CI / tests / linters
  -> review
  -> fixes / new evidence
  -> approval
  -> merge
  -> deployment
  -> runtime / production validation
  -> outcome
```

The important point is that **monitoring these systems is itself part of the engineering workload**. The assistant/control system should be able to notice and act on state transitions instead of requiring the developer to manually poll every system.

Examples:

- A PR has been waiting for review unusually long.
- CI has failed repeatedly for the same unrelated infrastructure reason.
- A linter failure is deterministic and can be fixed automatically.
- Review comments reveal an unresolved architectural decision that needs a human.
- A PR is blocked because a shared dependency is degraded.
- Several workloads are waiting on the same CI failure.
- Code has been changed but required verification has not happened.
- A deployment succeeded technically but downstream behavior did not recover as expected.

This turns **PR/CI/review/deployment monitoring** into a first-class part of the engineering control loop, not a separate notification system.

Git/PR/CI therefore represent engineering state, not merely provenance.

## 4. The engineering environment extends beyond coding

A workload may use:

- repository and code intelligence
- build and test systems
- ephemeral or persistent development environments
- browsers and E2E systems
- Git and pull requests
- CI/CD
- cloud and infrastructure
- logs, metrics, and traces
- production and downstream systems

This creates a continuous chain:

`work -> agent -> code -> PR -> CI/review -> deployment -> runtime -> outcome -> learning`

The personal assistant should be able to reason across this chain rather than treating each system as an isolated tool.

## 5. Fleet control is a separate control loop

Once many asynchronous agents operate concurrently, execution management becomes its own capability. Fleet management is not merely starting many agents; it is understanding collective state and changing execution in response.

A useful fleet loop is:

`observe fleet -> schedule -> allocate -> execute -> observe -> recover/reconcile -> continue`

A control plane may need to manage:

- workload scheduling and priority
- agent/runtime/model selection
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

This is the execution-management counterpart to the assistant's engineering-intelligence loop.

## 6. Shared dependencies and failure domains

Agents and workloads are connected through shared systems:

`work -> agent -> runtime/tools/environment -> GitHub/CI/cloud/credentials/human review`

The control plane should understand these relationships and their failure domains.

For example, if GitHub is degraded, a good system should recognize a shared dependency rather than treating every affected agent as an independent failure:

- continue work that does not require GitHub
- checkpoint work that cannot push
- queue PR/review operations
- avoid starting additional GitHub-dependent workloads
- resume and verify when the dependency recovers

Similarly, if CI has a systemic failure, the system should distinguish infrastructure failure from dozens of workload failures and avoid wasting agents or human attention on duplicate investigation.

This introduces **failure-domain awareness** and **blast-radius-aware recovery**.

## 7. Durable work state and recovery

Long-running engineering work must survive agent, runtime, environment, and dependency failures.

Prefer durable workload state over making a particular agent session authoritative:

`running -> interrupted -> checkpoint -> resume / retry / fork / redirect -> verify`

A checkpoint can contain:

- objective and policy
- current semantic state
- changes and artifacts
- decisions
- relevant context
- evidence gathered
- blocked dependencies
- next-step hypothesis

The agent runtime/session remains replaceable.

## 8. Resource and capability selection

The system should select resources appropriate to a workload rather than always using the same agent or model.

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

## 9. Assurance and autonomy

Autonomy should depend on risk and available evidence rather than being a global setting.

A useful relationship is:

`objective -> risk -> assurance requirement -> autonomy allowed`

Evidence can include:

- tests and CI
- linting/static analysis
- code review
- development-environment verification
- deployment results
- runtime telemetry
- production behavior

Low-risk work may continue with lightweight verification. Higher-risk work may require stronger tests, independent review, human approval, or restricted permissions.

## 10. Reality is the final verifier

Agentic software has two persistent gaps:

- **requirement gap** — stated requirements only approximate what stakeholders actually want
- **model gap** — tests and evaluation environments only approximate the real deployment environment

Therefore, passing a pre-deployment evaluator is not the same as producing an acceptable outcome. The useful loop is:

`requirements -> implementation -> evaluation -> deployment evidence -> revise requirements/model/evaluator`

Deployment and downstream behavior are therefore part of assurance, not merely post-hoc monitoring.

## 11. Human attention is a resource

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

This applies both to individual assistant findings and to fleet-level review/approval capacity.

## 12. Governance and deterministic enforcement

As fleets become heterogeneous, policy cannot live only inside individual prompts or agent implementations.

A useful boundary is:

`agent intent/action -> policy enforcement -> execution -> evidence/audit`

Deterministic enforcement should handle things such as permissions, action boundaries, budgets, auditability, and shutdown/containment. Semantic control can decide what should happen, while enforcement makes sure the resulting action stays within policy.

## 13. Outcome-level evaluation

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

The evaluation target is the complete engineering lifecycle, including whether the system noticed the right things, intervened usefully, recovered from failures, and produced an acceptable downstream outcome.

## 14. Industry evidence

The emerging model is supported by several complementary examples:

- **Spotify** — Portal demonstrates context/harness efficiency; Spotify Fleet Management demonstrates large-scale automated engineering maintenance; Honk extends fleet-wide automation with LLM-based changes.
- **Uber** — Software Factory demonstrates managed agents for code review, CI repair, E2E validation, alert triage, debugging and maintenance, with outcome-oriented economics.
- **Microsoft** — Foundry Control Plane explicitly provides fleet management, observability, lifecycle operations, policy and governance; Azure SRE Agent connects engineering agents with operational investigation and remediation.
- **Google** — Antigravity and related research show agent-native development environments and the importance of proactivity rather than autonomy alone.
- **OpenAI Symphony** — demonstrates the workload-to-agent control-plane model.
- **OpenHands** — provides an explicit Software Agent Control Plane framing.
- **cmux** — provides a human-facing execution/attention surface for many agents.
- **Herdr** — provides persistent runtime/session substrate.
- **OpenWolf** — provides agent-side context and lifecycle optimization.

These systems are evidence for capabilities and boundaries, not a reason for EOKS to reproduce them.

## 15. EOKS boundary

EOKS should continue to avoid becoming another universal infrastructure platform.

Likely EOKS ownership:

- objective and policy interpretation
- semantic workload state and reconciliation
- context/resource selection policy
- engineering-state observation and semantic interpretation
- evidence and assurance policy
- decisions about what should happen next
- durable outcome/evidence relationships
- evolutive context and learning
- autonomy and attention decisions

Likely delegated:

- agent runtime/session management
- terminal/workspace management
- Git hosting
- CI/CD execution
- development environments
- production telemetry collection
- cloud infrastructure
- generic agent SDKs
- fleet infrastructure where an existing system already provides it

The architectural test remains:

> **Can EOKS express the semantic decision without owning the execution substrate?**

This also clarifies the monitoring boundary: EOKS does not need to replace GitHub, CI, review tooling, observability, or fleet systems. It needs enough observation and provenance from them to reason about work and reconcile the desired outcome.

## 16. North-star architecture

```text
                         HUMAN
                           |
                      intent/judgment
                           v
              PERSONAL ENGINEERING ASSISTANT
                  "what matters and why?"
                           |
                    semantic intent
                           v
                  +------------------+
                  | EOKS / semantic  |
                  |     control      |
                  | outcome/policy/  |
                  | evidence/next    |
                  +--------+---------+
                           |
                    policy / decision
                           v
                  FLEET CONTROL PLANE
                   "operate execution"
                           |
             +-------------+-------------+
             v             v             v
          Agent A       Agent B       Agent C
             |             |             |
             +-------------+-------------+
                           v
                 ENGINEERING SYSTEM
          Git / PR / CI / Dev / Cloud / Prod
                           |
                     continuous
                      observation
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

The broader engineering loop adds continuous state monitoring and fleet operation around that semantic loop.

## 17. Remaining research questions

Broad ecosystem collection should now stop. The remaining questions are architectural and experimental:

1. What exactly belongs in the semantic control plane versus fleet infrastructure?
2. What is the minimal durable workload/checkpoint representation?
3. How should dependency graphs and failure domains be represented?
4. What engineering-state signals are sufficient for useful PR/CI/review/deployment monitoring?
5. How should assurance evidence accumulate across development, deployment, and production?
6. How should autonomy and human attention budgets interact?
7. What outcome metrics best represent production-qualified engineering value?

These questions are better candidates for experiments than another broad survey of agent products.
