# Continuous engineering maintenance: from data-model factories to self-maintaining environments

**Reviewed:** 2026-11-21  
**Status:** research synthesis / architectural implications, not an architectural decision

## Executive synthesis

Cube's *Building the Data Model Factory* describes a shift from manually maintaining a semantic model to designing a system that continuously discovers, proposes, tests and maintains model changes from production feedback. The important idea is broader than analytics: once agents depend on shared engineering knowledge and reusable capabilities, the quality of those foundations becomes a continuously maintained engineering responsibility.

EOKS already has the core ingredients: continuous knowledge maintenance, dependency-aware invalidation, evaluation, governed learning, policy, isolated execution and outcomes. The gap is less about another maintenance primitive and more about connecting them into a **complete, observable maintenance lifecycle for all reusable engineering assets**.

The strongest synthesis is:

> Treat maintenance as evidence-driven engineering work over versioned assets. Production signals discover candidate work; isolated executions prepare changes; independent checks establish whether the original problem was solved without regressions; accountable owners authorize promotion; post-promotion observations determine whether the change remains useful.

This is a research synthesis, not a proposal to add a new EOKS plane, primitive, permanent agent role or mandatory automation service.

## 1. What the data-model factory pattern contributes

Cube makes several concrete observations:

- The cost of a semantic model is not only its initial creation; it is keeping definitions useful as source systems, business rules and consumer needs change.
- More consumers—including agents—create more feedback about missing definitions, incorrect descriptions, slow queries and recurring work.
- Useful maintenance signals exist on both sides of a model: upstream schema/pipeline changes and downstream query history, conversations, errors and performance.
- Repeated ad-hoc calculations can become candidates for shared reusable measures, but repetition alone does not establish a correct business definition.
- User corrections can reveal missing documentation or context; those corrections can also become evaluation cases.
- Independent development environments and branches let multiple maintenance automations prepare inspectable changes in parallel.
- Candidate changes should be checked against the current model and existing consumers, not only against the specific issue that triggered the change.

A subtle but important distinction is the boundary between **local composition** and **shared canonicalization**. A consumer should be able to combine already-governed measures for a one-off analysis without every calculation becoming a permanent shared model change. Repeated use may justify proposing a shared abstraction, but domain meaning and ownership still need validation.

For EOKS, the analogous distinction is between task-local reasoning/context, reusable procedures or representations, and governed durable knowledge. Not every useful answer, context block, tool sequence or generated artifact should be promoted into shared state.

Source: Cube, [Building the Data Model Factory](https://cube.dev/blog/data-modeling-is-becoming-system-design), 2026-10-29.

## 2. Triangulation from production examples

### OpenAI: layered, code-enriched context

OpenAI describes an internal data agent whose context combines usage and lineage, human annotations, code-derived enrichment, institutional knowledge, memory and live runtime context. Some layers are refreshed through offline pipelines; live queries handle cases where stored context is missing or stale. Corrections can become persistent learnings, and curated question/answer cases with expected SQL are used as regression evaluations.

The useful point is not the six-layer taxonomy as an EOKS model. It is the combination of different evidence sources with different authority and refresh mechanisms. Code can explain how a data asset is produced; human annotations explain intended meaning; runtime inspection checks current state; memory preserves non-obvious corrections; evaluation checks whether the system continues to behave correctly.

OpenAI also reports that overlapping tools confused agents, and that higher-level goal guidance performed better than rigidly prescribed paths. This supports EOKS's preference for bounded capabilities and explicit evidence contracts over a large overlapping tool surface or a fixed maintenance script.

Source: OpenAI, [Inside OpenAI's in-house data agent](https://openai.com/index/inside-our-in-house-data-agent/), 2026-01-29.

### Anthropic: maintenance debt appears as quality drift

Anthropic describes three major failure modes for analytics agents: concept/entity ambiguity, stale data or definitions, and retrieval failure. Cube's account of the Anthropic example reports offline accuracy falling from roughly 95% to 65% over a month before skill maintenance was treated as an engineering problem. Anthropic's approach includes keeping skills aligned with data models and turning stakeholder corrections into documentation changes and evaluation cases.

The broader implication is that reusable instructions and skills are executable dependencies. A skill can become wrong even when its text has not changed, because the system it describes has changed. Updating a source model without updating dependent skills, or updating a skill without evaluating its dependent tasks, leaves a split-brain system.

Sources:
- Anthropic, [How Anthropic enables self-service data analytics with Claude](https://claude.com/resources/articles/how-anthropic-enables-self-service-data-analytics-with-claude), 2026-06-03.
- Cube, [Building the Data Model Factory](https://cube.dev/blog/data-modeling-is-becoming-system-design), 2026-10-29.

## 3. The gap: maintenance applies to more than knowledge

EOKS's canonical [Continuous knowledge maintenance](../../docs/continuous-knowledge-maintenance.md) document already covers change detection, retrieval, transformation, validation, promotion, invalidation and consolidation of deterministic tools. [Evaluation](../../docs/evaluation.md) already makes evaluation evidence part of control decisions. [Memory](../../docs/memory.md) already defines Learning Records and controlled promotion.

The additional research question is whether the same lifecycle should be explicitly considered across the full set of reusable engineering assets:

| Asset under maintenance | Example drift or signal | Candidate maintenance action |
|---|---|---|
| Canonical knowledge and decisions | Code, architecture or external dependency changes | Propose a scoped knowledge/decision update |
| Context representations and indexes | Source revision, retrieval misses, stale dependencies | Refresh or selectively invalidate derived context |
| Skills and reusable procedures | Repeated corrections, changed CLI/API behavior, task regressions | Update the procedure and its task regression set |
| Workflows and automation definitions | Repeated manual interventions, retries, failure clusters | Simplify or revise workflow behavior |
| Evaluation datasets and rubrics | Newly observed failure classes, changed task semantics | Curate candidate cases; version and validate rubric changes |
| Tool/provider configurations | API changes, capability drift, permission or reliability failures | Revalidate capability claims and adapter behavior |
| Policies and constraints | New risks, ownership changes, incidents or regulatory requirements | Propose a policy change for accountable approval |
| Derived artifacts and reusable computations | Changed source dependencies or invalid assumptions | Recompute, invalidate, or retire affected results |

These assets do not need a common storage format or one universal update mechanism. They do need enough shared lifecycle metadata to answer: what is this, who owns it, what does it depend on, what evidence supports it, where is it valid, how was it evaluated, and what should happen when those assumptions change?

A useful extension of the existing knowledge-maintenance document would be a broader asset-coverage table and operational guidance. This research note deliberately records the gap without duplicating that canonical document.

## 4. A complete maintenance lifecycle

The data-model factory pattern becomes more useful when combined with EOKS's existing Work/Run, evidence, evaluation, policy and outcome semantics:

```text
upstream change / production usage / failure / correction
                         |
                         v
                 observe and correlate
                         |
                         v
              detect candidate maintenance
                         |
                         v
            triage scope, owner, risk, value
                         |
                         v
            define desired change + oracle
                         |
                         v
          isolated workspace / branch / run
                         |
                         v
             candidate change + evidence
                         |
                         v
       independent validation + regression suite
                         |
                         v
            accountable approval / policy gate
                         |
                         v
                 promote / release
                         |
                         v
             observe post-change behavior
                         |
                         v
            retain, revise, revert or retire
                         +-------------------->
```

The lifecycle has several important boundaries:

1. **Signal is not intent.** An error spike, recurring query or user correction is an observation. It must be interpreted into a bounded problem and desired outcome.
2. **Proposal is not authority.** The agent that detects or implements a change should not silently grant itself permission to promote it.
3. **A passing local test is not sufficient.** The candidate must address the triggering problem and preserve relevant existing behavior.
4. **Promotion is not proof of lasting success.** Post-promotion observations can reveal regressions, unexpected usage or changed assumptions.
5. **Retirement is part of maintenance.** Stale skills, unused abstractions, obsolete eval cases and redundant automations should be deprecated rather than accumulated indefinitely.

## 5. Operational signals and trigger quality

Cube's upstream/downstream distinction suggests two broad signal classes.

### Upstream signals

- Source schema, API, dependency or repository changes.
- Changes to the code that produces a derived artifact.
- Changed ownership, policy, permissions or runtime configuration.
- Version changes in models, tools, harnesses or execution environments.

These often support deterministic impact analysis and targeted invalidation.

### Downstream signals

- Repeated user corrections or unresolved questions.
- Recurrent ad-hoc calculations or repeated manual procedures.
- Retrieval misses, context discovery overhead or contradictory guidance.
- Test/evaluation regressions and recurring failure signatures.
- Tool errors, slow execution, rising latency or cost.
- Human overrides, rejected proposals and repeated review comments.

These signals are noisier. A repeated behavior may indicate a missing abstraction, but it may also be a one-off need, a temporary incident or an incorrect convention. Frequency is a discovery heuristic, not an acceptance criterion.

A useful trigger should retain its provenance, affected workload/asset, time window, frequency, severity, confidence, and the evidence needed to reproduce or investigate it. Deduplication and correlation matter: ten alerts from one underlying incident should not automatically create ten maintenance tasks.

## 6. Safety, independence and accountability

Self-maintenance creates a special risk: the system may change the artifacts or checks used to judge its own changes. EOKS should preserve separation between proposal authority and acceptance authority.

Useful constraints include:

- **Protected oracles:** the implementation being evaluated must not be able to weaken or silently rewrite the acceptance criteria used to judge it. Changes to tests, eval datasets, rubrics and policy require their own review and provenance.
- **Independent verification:** use deterministic checks where possible and a separate reviewer/evaluator for semantic claims; record when independence is only partial (for example, shared model, prompt, data or failure mode).
- **Explicit ownership:** identify the accountable human or team for business semantics, shared knowledge, policy, evaluation criteria and operational outcomes. Agent execution does not transfer accountability.
- **Least authority:** separate read, propose, write, approve, promote and deploy permissions. A branch/PR workflow is a useful boundary, not a substitute for authorization.
- **Risk-based gates:** low-impact documentation refreshes may be automated; changes to security policy, access, production data, deployment, irreversible state or evaluation authority require stronger approval.
- **Reversibility and recovery:** retain the previous version, dependency/provenance record and a tested rollback or compensating action where feasible. Verify the restored state rather than treating a successful rollback command as proof.
- **Auditability:** preserve the trigger, candidate diff, evaluation version/results, approvals, promotion event and post-change observations.

A recent working paper on agentic CI/CD argues that conventional pipeline metrics do not capture decision quality, constraint adherence, escalation behavior or long-term governance impact. This is adjacent rather than direct evidence for EOKS, but it reinforces the need to evaluate the control system, not only whether a patch passed tests.

Source: [From Assistance to Agency: Rethinking Autonomy and Control in CI/CD Pipelines](https://openreview.net/attachment?id=xiTMi1r1wE&name=pdf), 2026.

## 7. Maintenance economics

Continuous automation can create more work than it removes. The objective is not maximum maintenance activity or maximum autonomy; it is better verified outcomes at an acceptable total cost and risk.

For each candidate, compare:

- expected impact and recurrence of the underlying problem;
- confidence that the signal identifies the actual cause;
- implementation, evaluation, review and operational cost;
- regression/blast-radius risk and reversibility;
- downstream human attention and ongoing maintenance burden;
- expected reuse across workloads;
- the cost of doing nothing or handling the issue manually.

A useful conceptual decision is:

```text
expected maintenance value
  = expected avoided failure / rework / manual effort
  + expected reuse benefit
  - detection and implementation cost
  - validation and review cost
  - ongoing ownership and regression risk
```

This is a decision aid, not a universal numeric score. It should account for uncertainty and asymmetric consequences: a low-frequency security defect may deserve action even when its average expected cost is difficult to estimate.

## 8. What to measure

Do not collapse maintenance quality into the number of automated PRs or the percentage of tasks completed by agents.

| Dimension | Candidate measures |
|---|---|
| Signal quality | actionable-signal precision, duplicate rate, missed-issue rate, time to detect |
| Resolution | time to validated fix, recurrence rate, problem-resolution rate |
| Change safety | regression rate, escaped defects, rollback/compensation rate, policy violations |
| Review burden | human minutes per accepted change, rejection/rework rate, approval latency |
| Asset health | freshness coverage, dependency completeness, stale-asset rate, unused/duplicated assets |
| Evaluation integrity | protected-oracle violations, eval-set provenance, judge agreement/calibration, benchmark drift |
| Economics | cost per validated improvement, avoided rework, total maintenance effort |
| Long-term outcome | task quality, reliability, velocity, cost and trust/autonomy by workload class |

Measure distributions and workload slices. Averages can hide a dangerous failure class or a large review burden concentrated in one team.

## 9. Experiments for EOKS

The most useful experiments are small and falsifiable:

1. **Signal-to-task experiment:** select one repository or service and compare manual issue discovery with correlated upstream/downstream signals. Measure actionable precision, duplicates, missed issues and time to validated task.
2. **Skill maintenance experiment:** use a versioned skill and a fixed task set. Introduce a representative dependency/API change, detect affected skill content, propose a patch, and compare task outcomes before and after. Preserve the original eval set.
3. **Correction-to-regression experiment:** turn a sample of human corrections into candidate eval cases. Measure curation cost, false generalization, regression detection and future recurrence.
4. **Isolated maintenance experiment:** run two independent maintenance candidates against separate branches/workspaces. Compare conflict rate, reviewer effort, reproducibility and regression coverage with a shared-workspace baseline.
5. **Protected-oracle experiment:** allow an agent to propose implementation and test changes, but require separate authorization for changes to the evaluation contract. Test whether this prevents reward hacking or accidental weakening of checks.
6. **Maintenance economics experiment:** compare scheduled broad refresh, event-triggered targeted refresh and manual maintenance. Measure total cost, freshness, missed drift and human interventions.

These experiments should reuse the existing [context evaluation methodology](../context-evaluation.md), [community evidence and bottlenecks](../community-evidence-bottlenecks.md), and canonical evaluation concepts rather than create another benchmark framework.

## 10. Mapping to existing EOKS concepts

| Existing EOKS concept | Role in this pattern |
|---|---|
| Work / Task / Intent | Defines the maintenance problem and desired outcome |
| State and resources | Identifies current asset versions, dependencies and observed state |
| Knowledge and representations | Holds authoritative assets and derived views |
| Context / Loadout | Supplies the evidence needed for the maintenance workload |
| Capabilities / providers | Supplies detectors, analyzers, editors, evaluators and execution backends |
| Workflow / conductor | Coordinates triage, isolated execution, validation and recovery |
| Policy | Defines authority, approval and promotion constraints |
| Run / execution evidence | Records what was attempted and what changed |
| Evaluation | Determines whether the change solves the problem and preserves behavior |
| Outcome | Records accepted result and post-promotion effect |
| Memory / Learning Records | Retains validated lessons and reusable procedures |

No new primitive is required by this mapping. Maintenance is a workload/control-loop pattern that composes existing concepts.

## 11. Research implications and open questions

- Which asset types benefit from event-triggered maintenance versus scheduled review?
- How can EOKS identify semantic dependencies that are not represented in code or explicit metadata?
- How should ownership be discovered and verified as teams and systems change?
- What minimum evidence is needed before a correction becomes a reusable procedure?
- How can eval sets and rubrics be protected from the system whose behavior they evaluate?
- How should partially independent evaluators be represented when they share models, data or assumptions?
- What is the right balance between local composition and promotion into shared canonical abstractions?
- When should an asset be refreshed, invalidated, deprecated or deleted?
- How can maintenance value be estimated without optimizing for alert volume, PR count or easy-to-game metrics?
- What post-promotion monitoring is necessary for each risk class?

## 12. Relationship to canonical EOKS documents

This note is exploratory. Canonical ownership remains:

- [Continuous knowledge maintenance](../../docs/continuous-knowledge-maintenance.md) — lifecycle, roles, incremental updates, promotion and invalidation.
- [Evaluation](../../docs/evaluation.md) — evaluation evidence, calibration and control.
- [Memory](../../docs/memory.md) — experience, Learning Records and governed behavioral promotion.
- [Policy and decisions](../../docs/decisions.md) — authority, enforcement and controlled knowledge promotion.
- [Research methodology](../../docs/methodology.md) — evidence quality and research-to-architecture process.

If experiments establish a stable general requirement, update the existing canonical owner rather than creating a parallel maintenance architecture.

## Sources

- Cube, [Building the Data Model Factory](https://cube.dev/blog/data-modeling-is-becoming-system-design), 2026-10-29.
- OpenAI, [Inside OpenAI's in-house data agent](https://openai.com/index/inside-our-in-house-data-agent/), 2026-01-29.
- Anthropic, [How Anthropic enables self-service data analytics with Claude](https://claude.com/resources/articles/how-anthropic-enables-self-service-data-analytics-with-claude), 2026-06-03.
- [From Assistance to Agency: Rethinking Autonomy and Control in CI/CD Pipelines](https://openreview.net/attachment?id=xiTMi1r1wE&name=pdf), 2026.
