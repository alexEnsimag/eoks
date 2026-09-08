# AI coding beyond the agent

## Why this matters to EOKS

Recent AI-coding systems increasingly treat the model as one component inside a larger execution environment. Anthropic frames context engineering as the runtime curation of the information available at each inference step; Spotify's Xirp combines vendor-neutral multi-session execution with Portal's organizational context; Kiro Crew combines persistent memory, lessons, skills, scheduling, checkpoints, validation and multi-agent execution; and Warp presents software factories as infrastructure for repeatable, multi-step agent workflows.

These systems do not establish one common architecture, and their product claims are not interchangeable evidence. They do, however, provide convergent prior art for an EOKS hypothesis:

> **Reliable AI engineering is increasingly a systems problem around probabilistic workers: context, execution state, evidence, validation, policy and coordination have to be engineered as first-class concerns.**

The useful EOKS question is therefore not whether to build a "better agent", but which control-plane semantics are needed to coordinate these capabilities without turning every implementation mechanism into an architectural primitive.

## Primary sources

- Anthropic, [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — context as a finite, dynamically curated resource; just-in-time retrieval, progressive disclosure, compaction, structured notes and sub-agent architectures.
- Spotify, [What we've learned scaling AI coding agents at Spotify](https://portal.spotify.com/blog/introducing-xirp) — Xirp as a vendor-neutral environment for many concurrent sessions, with Portal supplying organizational context and preserving session knowledge across work.
- Kiro, [Kiro Crew](https://kiro.dev/crew/) — persistent sessions, memory, lessons, skills, scheduled/webhook work, checkpoints, validation, retries, multi-agent execution and audit/security controls.
- Warp, [Agent Mode / Warp](https://www.warp.dev/ai) — environment-aware execution, tool-mediated workflows, approval boundaries and self-correction; Warp also positions [Warp Factories](https://www.warp.dev/factories) as infrastructure for software factories.

The primary sources are product/engineering material rather than independent controlled studies. EOKS therefore treats them as **existing-system evidence and hypothesis generators**, not as proof that any particular mechanism improves engineering outcomes.

## 1. Context engineering is a runtime concern

Anthropic explicitly distinguishes prompt engineering from context engineering: the latter concerns curating and maintaining the information available to the model at each inference step. Their description emphasizes a constantly changing context universe, finite attention, just-in-time retrieval and progressive disclosure. They also describe compaction, structured note-taking and sub-agents as different ways to manage long-horizon work.

This strengthens EOKS's existing distinction:

```text
resource / evidence universe
        |
   eligibility + policy
        |
   workload working set
        |
 context acquisition/compilation
        |
   model context
```

The EOKS implication is not "use a context manager" as a new subsystem. It is that **context compilation is part of workload control** and should remain inspectable, budgeted and attributable to a task outcome.

Anthropic's just-in-time model also reinforces the existing proactive/reactive/hybrid distinction in `docs/context.md`. Pre-injecting everything is not automatically superior to progressive discovery, while pure exploration can be expensive. The correct boundary is workload- and model-dependent and should be evaluated end to end.

## 2. Spotify shows that agent execution and institutional context are separable

Spotify describes Xirp as a vendor-neutral environment for managing dozens of concurrent coding-agent sessions across Claude Code, Gemini CLI and Codex, with isolated worktrees and the ability to switch tools without reconstructing the working state. Spotify reports adoption across more than 36,000 sessions; this is a company-reported result, not an independent benchmark.

The more important architectural point is the separation between Xirp and Portal. Portal supplies component architecture, dependency graphs, ownership and architectural decisions, while session transcripts and metadata flow back into the shared environment.

For EOKS:

```text
organizational / engineering reality
          |
   knowledge representations
          |
  evidence providers
          |
  context compilation
          |
 task-specific context
          |
 agent / execution substrate
```

This is consistent with the existing Xirp research in `research/prior-art/xirp.md`: repository structure, organizational context, session continuity and canonical knowledge should remain distinct capabilities. A code graph is evidence/navigation; it is not automatically institutional truth.

## 3. Kiro Crew demonstrates persistent execution around agents

Kiro Crew makes several execution concerns explicit in one development environment:

- memory, lessons and skills persist across sessions;
- work can be triggered by schedules, webhooks and heartbeats;
- long-running work has checkpoints, validation and retries;
- multiple agents can work in parallel;
- project context is represented in a searchable knowledge graph;
- actions are subject to approval, sandboxing and other security controls;
- signed audit logs make activity inspectable.

These mechanisms map cleanly onto existing EOKS concepts rather than requiring a new "crew" abstraction:

| Kiro Crew mechanism | EOKS interpretation |
|---|---|
| persistent memory / lessons | memory and knowledge lifecycle |
| skills | reusable procedural resource |
| schedules / webhooks | workload/event triggers |
| checkpoints | durable execution state |
| validation / retries | evaluation + reconciliation |
| parallel agents | execution modality / workflow topology |
| knowledge graph | representation/evidence provider |
| approvals / sandbox | policy and authority boundary |
| audit log | execution evidence / provenance |

The important lesson is that **persistence, validation and authority are part of the execution environment**, not properties that can safely be delegated to an agent's final message.

## 4. Warp reinforces the execution-substrate boundary

Warp's Agent Mode describes an environment-aware agent that can request and execute commands, obtain additional context as needed and self-correct after invalid commands. It also emphasizes user approval and control over consequential commands. Warp separately positions Factories as infrastructure for software-factory workflows.

For EOKS this is evidence for keeping the execution substrate separate from the control architecture:

```text
EOKS control semantics
        |
   workflow / policy
        |
 existing execution substrate
        |
 model + tools + environment
```

EOKS should not become another terminal agent or coding-agent product. Its value is in deciding what should happen next, what resources/evidence are eligible, what assurance is required and how outcomes feed reconciliation.

## 5. A useful synthesis: model -> harness -> loop -> graph -> factory

The practitioner synthesis that prompted this research proposes a hierarchy of model, harness, loop, graph and factory. The primary sources support parts of this hierarchy but do not establish it as a universal taxonomy.

A safer EOKS interpretation is:

```text
model
  = probabilistic reasoning capability

harness
  = execution environment, tools, permissions and interfaces

loop
  = repeated observe/evaluate/act/reconcile behavior

graph
  = workflow/dependency representation connecting steps and decisions

factory
  = operating many workloads and improving the system from outcomes
```

These are **views over existing EOKS concepts**, not proposed new runtime primitives. In particular, "graph" should normally be a workflow/relationship representation and "factory" an operational scale/feedback concern.

## 6. Evidence and authority are missing dimensions in a model-only taxonomy

Across the primary sources, a recurring concern is not just information flow but **what is allowed to establish state and what is allowed to cause side effects**.

A useful EOKS distinction is:

```text
claim / observation
       |
    evidence
       |
    evaluation
       |
 decision / policy
       |
 authorized action
```

The agent's assertion that a task is complete is therefore one observation among others. Tests, static analysis, runtime observations, independent review, policy checks and human approval can have different authority for different decisions.

This connects directly to EOKS's existing separation:

```text
observability -> what happened?
reliability   -> how much should this result be trusted?
control       -> what should happen next?
```

Do not collapse these into a single confidence score or a generic "agent state".

## 7. Artifacts are the handoff substrate, not necessarily a new primitive

Kiro's persistent work, Spotify's session continuity and Anthropic's structured note-taking all point toward durable artifacts between reasoning steps. EOKS already has the right architectural boundary: artifacts can carry intent, specifications, plans, evidence, evaluations, decisions, outcomes or audit information without automatically becoming a new ontology object.

A useful handoff is therefore:

```text
execution
   |
artifact + evidence + execution state
   |
next reconciliation
   |
context compilation
   |
next reasoning step
```

The important property is **reconstructability**: another run should be able to recover the authoritative state without relying on hidden agent memory.

This reinforces the existing AI-native SDLC research rather than creating a competing artifact model.

## 8. The execution graph should remain subordinate to reconciliation

The word "graph" can obscure an important boundary. A workflow graph describes possible dependencies, transitions and responsibilities; it does not itself decide whether the current state satisfies the desired outcome.

For EOKS:

```text
                 desired state + policy
                          |
                          v
                    reconciliation
                          |
                   workflow graph
                  /       |       \
              action   evidence   escalation
                 |        |          |
                 +--------+----------+
                          |
                    actual state
                          |
                    evaluation
                          |
                    reconcile again
```

This means EOKS should resist introducing a generic "execution graph" runtime primitive unless implementation evidence demonstrates a lifecycle that cannot be represented by the existing Task, Run, Decision, Policy, Evaluation and Outcome model.

The graph can still be extremely important as a **representation of workflow state and relationships**. The control semantics live in reconciliation.

## 9. Factory-level learning is a slower feedback loop

The factory perspective adds a useful time-scale distinction:

```text
step loop
  act -> observe -> evaluate

workload loop
  reconcile until acceptance/escalation

system loop
  aggregate outcomes -> evaluate interventions -> update policy/resources
```

This is compatible with EOKS's existing nested-loop model. A factory does not need to be a new runtime layer; it can be the operational environment in which many workload loops are observed and improved.

This also clarifies the role of synthesis. Synthesis should turn evidence from completed workloads into candidate knowledge, policy changes, resource-selection improvements or research questions. Promotion remains governed and evidence-backed.

## 10. What this changes in EOKS

The review does **not** justify a new agent, graph or factory abstraction. Instead it sharpens several existing boundaries:

1. **Context is a control-plane artifact.** Its composition should be attributable to the workload and evaluation record.
2. **Execution substrates are replaceable.** Claude Code, Codex, Gemini CLI, Kiro-like runtimes and other harnesses are resources/adapters, not EOKS architecture.
3. **Workflow graphs are representations.** Reconciliation remains authoritative for control decisions.
4. **Evidence and authority must be explicit.** An observation can inform a decision without being authorized to establish acceptance.
5. **Durable artifacts bridge execution steps.** They support reconstructability, context compilation and auditability without becoming a new universal primitive.
6. **Factory learning is a slower nested loop.** Aggregate outcomes can improve policy, resource selection and knowledge, but promotion must remain controlled.
7. **Autonomy remains graduated.** More automation is justified by workload-specific assurance, not by the presence of an agent graph or larger model.

## 11. Research questions

The primary sources suggest concrete experiments rather than architecture changes:

- Does a task-specific context compiler outperform broad pre-injection at equal token/cost budgets?
- When does proactive context materially outperform just-in-time exploration, and when does it create harmful context pollution?
- Does institutional context (ownership, architecture, dependency and decision evidence) improve correctness beyond repository-local context?
- How much session-derived knowledge should be promoted, and what validation is required to prevent stale or incorrect knowledge from compounding?
- Do durable execution checkpoints improve recovery and handoff quality enough to justify their operational cost?
- Which evidence/authority combinations best predict safe completion versus additional verification?
- Can workload outcomes improve resource/model selection without overfitting to one repository or agent harness?
- At what scale do workflow-graph representations become operationally useful enough to justify independent lifecycle/identity?

These should be evaluated using the existing EOKS methodology: preserve configuration, context, execution state, evidence and outcomes; compare interventions against representative workloads; record limitations and contradictory results; and promote only mechanisms that earn their place.

## EOKS disposition

**Status: Supported prior-art pattern; no new runtime primitive.**

The strongest conclusion is architectural consolidation, not expansion: the "beyond the agent" shift is already largely represented in EOKS through workload reconciliation, context compilation, resources/loadouts, execution workflows, evaluation, evidence, memory/knowledge lifecycle and graduated autonomy. The research adds confidence in those boundaries and identifies evidence/authority plus multi-timescale feedback as dimensions to make explicit in future experiments.
