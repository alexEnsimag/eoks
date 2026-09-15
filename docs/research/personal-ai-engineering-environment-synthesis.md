# Personal AI Engineering Environment — Synthesis Addendum

## Emerging model

The research now points to a stronger unifying model than a collection of AI-development tools: a developer work system in which human intent, agents, context, execution, engineering state, evidence, attention, and outcomes form one continuous loop.

```text
                         HUMAN INTENT
                              |
                           POLICY
                              |
             context / resources / agent selection
                              |
                           EXECUTE
                              |
                           OBSERVE
                       +------+------+
                       |             |
                    continue      attention
                                     |
                                   HUMAN
                                  DECISION
                                     |
                               VERIFY / REVIEW
                                     |
                                  OUTCOME
                                     |
                                  LEARN
                                     |
                                  next work
```

This is not a proposal for EOKS to own every component. The personal AI engineering environment is the research target; EOKS is a candidate semantic/control layer that connects existing components.

### Four conceptual layers

The capability map is easier to reason about when capabilities are grouped into four levels:

1. **Human work** — goals, work items, decisions, knowledge, attention, judgment, exceptions.
2. **Agent work** — sessions, context, tools, delegation, execution, collaboration, checkpoints.
3. **Engineering state** — code, Git, PRs, CI, deployments, production/downstream results.
4. **Control and learning** — policy, observation, assurance, intervention, evaluation, outcomes, learning.

The same capability may cross layers, but this distinction prevents infrastructure, user concepts, and EOKS responsibilities from being conflated.

## Work is the common unit

The most useful abstraction emerging from the research is not an IDE, agent, session, or dashboard. It is **work**.

A work item can carry:

- objective and acceptance criteria
- constraints, permissions, and risk policy
- selected context and resources
- one or more agent sessions
- execution mode and environment
- artifacts and engineering-state changes
- observations and evidence
- attention/intervention events
- verification and review
- outcome and cost
- lessons or durable context candidates

Local single-agent execution, local multi-agent execution, and remote fleets can then be execution modes of the same work model rather than separate product concepts.

## Work lifecycle and recovery

The environment should make the lifecycle of work explicit:

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

Failure and interruption should not require starting over:

```text
executing
  -> interrupted / failed
    -> checkpoint
      -> resume / retry / fork / redirect
        -> executing
```

This is a lifecycle model, not necessarily a new EOKS runtime primitive. Existing runtimes and workspaces can own checkpoints and process/session mechanics while EOKS reasons about semantic state and the next decision.

## Resource and capability selection

As agents become interchangeable execution workers, another control question becomes important:

> Given this work, which agent, runtime, context, tools, environment, budget, and assurance level should be used?

Selection should be treated as a policy decision rather than hard-coding a universal agent router. The useful minimum may be as simple as choosing among existing local/remote agents and execution environments. More sophisticated allocation should be justified by measured benefit.

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

## Human attention is a constrained resource

OpenWolf/Portal-style mechanisms optimize what reaches the **agent**. cmux-style attention mechanisms optimize what reaches the **human**. These are two sides of the same context problem.

```text
agent-side optimization              human-side optimization
what information is useful?          what deserves attention?
what context should be retained?     what requires intervention?
what can be compressed?              what can be suppressed?
```

The environment should therefore evaluate not only token/context efficiency but also attention efficiency. Autonomous throughput is only useful if human cognitive load does not grow proportionally.

## Git, PR, and CI are engineering state

Git/PR/CI should not be treated merely as provenance links. They are state transitions in the work lifecycle:

```text
intent -> planned -> change -> PR -> verification -> review
      -> accepted -> merged -> deployed -> downstream validated
```

The existing provenance chain therefore becomes stronger when engineering state is included:

```text
work -> agent session -> change -> commit/PR -> CI/review
     -> deployment -> downstream outcome
```

Agent transcripts remain useful evidence, but they are not equivalent to accepted repository state or downstream validation.

## What this means for EOKS

The evidence does **not** justify making EOKS the owner of the whole personal AI engineering environment. A cleaner boundary is:

```text
Workspace / knowledge       Obsidian or alternatives
Agent efficiency            OpenWolf / Portal-like harnesses
Live execution / attention  cmux or alternatives
Runtime                     Herdr or alternatives
Agent-loop control          Claude/Codex APIs or alternatives
Interoperability             ACP / environment protocols
Engineering state            Git / PR / CI
Fleet execution              existing cloud/fleet systems
                         |
                         v
             EOKS semantic/control layer

objective • policy • context selection/evolution
observation • evidence • assurance • attention
work lifecycle • outcomes • learning
```

EOKS should remain replaceable at the same boundary: it coordinates semantic state and decisions without reimplementing terminals, runtimes, Git forges, notification UIs, agent SDKs, or knowledge stores.

## Phase A interpretation

Obsidian should be treated as a **Phase A workspace candidate**, not an architectural dependency. The experiment should validate the work model and control loop using whatever workspace proves useful.

The next experiments should prioritize evidence over additional tool collection:

1. Represent one real task as a common work item across local and/or remote execution.
2. Connect an agent session to context, engineering state, verification, and outcome.
3. Test one semantic intervention where EOKS would choose differently from simply letting the agent continue.
4. Measure context efficiency and human attention separately.
5. Test checkpoint/resume or fork/retry recovery.
6. Connect one familiar Git/IDE mechanism to causal agent provenance.
7. Test one scheduled/proactive workload and measure intervention utility.
8. Test one low-risk setup improvement through observe → suggest → approve → apply → evaluate.

The key question is no longer whether a large number of tools can be assembled. It is whether the **work/control model actually reduces friction and improves outcomes** when those tools are composed.
