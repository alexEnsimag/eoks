# Research agenda

The agenda should prioritize experiments that can falsify the EOKS control-plane hypothesis, not additional architectural vocabulary.

A central rule is:

> **Treat every infrastructure capability as an intervention whose value depends on the model, workload, repository, budget and baseline.**

The goal is not to prove that graphs, retrieval, memory, context management, computer-systems techniques or multi-agent workflows are good. The goal is to discover **what improves engineering outcomes, when, by how much, at what cost, and with which failure modes**.

The agenda now operates as an evidence-driven loop:

```text
concept / research signal
        ↓
operational hypothesis
        ↓
signals + metrics + evaluation method
        ↓
strong practical baseline
        ↓
controlled experiment / workflow trial
        ↓
evidence
        ↓
use / prototype / targeted research / reject
        ↓
synthesis + agenda update
        ↓
architectural promotion only when warranted
```

The purpose of this loop is not to finish the research before building. Research and experimentation should proceed in parallel, with experiments determining which unanswered questions are worth pursuing.

The evaluation layer in [Evaluation signals and metrics](evaluation-signals.md) defines the common concept → hypothesis → signals → metrics → evaluation → failure-signatures → decision chain. The [Operational synthesis](operational-synthesis.md) defines the current action triage. This agenda defines **what to do next and what evidence should move the work forward**.

The evidence intake that currently informs prioritization is [Community and academic evidence on agent bottlenecks](../research/community-evidence-bottlenecks.md). It records effect sizes, experimental settings, community failure reports, contradictory evidence and next tests. It is deliberately non-normative.

## EOKS research methodology

EOKS should deliberately combine **academic research, community/practitioner evidence, existing systems and controlled experimentation** rather than treating any one source as authoritative.

These sources play different roles:

- **Academic research** provides formal concepts, terminology, controlled methods and evidence that can be difficult to obtain from practice alone. It may also lag rapidly evolving agent practice.
- **Community/practitioner evidence** provides early signals about emerging capabilities, real-world failure modes, useful workflows and tooling. It is faster and often more workload-specific, but usually less controlled and more difficult to compare directly.
- **Existing systems and tools** provide concrete mechanisms that can be evaluated without assuming EOKS needs to reinvent them.
- **EOKS experiments** test whether a signal or mechanism transfers to the workloads, objectives and control problems EOKS cares about.
- **Synthesis** connects these sources into explicit hypotheses and architectural decisions without silently promoting any source into EOKS truth.

The relationship is therefore better understood as a triangulation than as a linear authority ladder:

```text
                 ┌── academic research ─────┐
                 │                          │
emerging signal ─┼── community practice ────┼─→ EOKS hypothesis
                 │                          │
                 └── existing systems/tools ┘
                                            ↓
                                      experiment
                                            ↓
                                         evidence
                                            ↓
                                  synthesis / decision
```

A community observation can motivate research, an academic result can motivate an experiment, and an EOKS workload can reveal a question that neither literature nor community practice has answered. The important property is that **claims that affect EOKS architecture eventually become testable, traceable and explicit about their evidence status**.

This is also why research intake remains part of the agenda even though experiments are now the execution priority: research should feed experiments, and experiments should determine which research deserves more attention.

## Current execution priority

The immediate objective is to establish an **EOKS evaluation laboratory**, not to add another broad architectural layer.

### P0 — Instrument one real vertical workload

Build one local workload where EOKS can observe the complete loop:

```text
Task → acquisition/context/evidence selection → agent execution
     → verification → Outcome → Evaluation → next Decision
```

Record model/version, configuration, context manifest, acquisition/provider calls, tool calls, execution state, decisions, costs, intermediate evidence and outcome. Establish the trace before building sophisticated schedulers.

Use a representative repository-engineering workload plus at least one real workflow from the intended LLM workflow environment. The controlled workload provides comparability; the real workflow provides ecological validity.

### P0 — Establish a strong practical baseline

Every intervention needs a baseline that is competitive with what a capable engineer would actually use, not an artificially weak implementation. Record the baseline's model, tools, context strategy, budget, retries and verification procedure.

A result should not be considered useful merely because it beats a naive baseline.

### P0 — First capability experiments

Run these in order:

1. **Context / working-set construction** — can acquisition infrastructure reduce avoidable exploration while preserving useful evidence?
2. **Minimum-sufficient evidence / provider selection** — can a controller choose the cheapest evidence mechanism sufficient for the engineering question?
3. **Computation / artifact reuse** — can validated derived artifacts avoid repeated work without increasing stale or incorrect reuse?

These form a natural progression:

```text
what information is needed?
        ↓
which provider can establish it?
        ↓
what computation should be performed?
        ↓
what result can safely be reused?
```

### P1 — Evaluate mechanisms, not brands

Evaluate promising existing tools only as candidate mechanisms for explicit hypotheses. Examples include structural/graph representations, lexical/semantic retrieval, code intelligence, static analysis, OTel/telemetry, caching, model routing, durable knowledge stores and workflow mechanisms.

The question is not “is this tool good?” but:

> **Which workload capability does it provide, what signal should improve if it works, and does the end-to-end engineering outcome justify its cost and complexity?**

### P1 — Close research gaps exposed by experiments

When an experiment reveals that measurement, interpretation or mechanism selection is inadequate, perform targeted research. Do not delay experiments until every open research question is resolved.

### P1/P2 — Prototype EOKS-specific capabilities

Prototype only where experiments demonstrate a recurring control problem that existing mechanisms do not adequately express or coordinate. Candidate capabilities include evidence-aware provider selection, context/working-set policy, a computed-artifact registry and assurance-to-autonomy policy.

### P2 — Architectural promotion

Promote a concept only after repeated evidence demonstrates a stable missing relationship or control capability that cannot be adequately represented by the existing EOKS model.

## Experiment tracks

The detailed agenda below remains the research program; these tracks define its current execution order.

| Track | Primary sections | Immediate purpose |
|---|---|---|
| Evaluation laboratory | 1, 19, 20, 26, 27 | instrumentation, attribution, repeatability, decisions |
| Context / working set | 3–6, 21–23 | test acquisition, locality, lifecycle, caching and context control |
| Evidence / assurance | 12–14, 17 | test deterministic evidence, provider selection and reliability |
| Execution / learning | 7–9, 15–16 | test continuity, memory, topology, routing and adaptive control |
| Representations / reuse | 8, 11, 21–23 | test durable knowledge, derived artifacts and reuse |
| Systems transfer | 21–23 | test OS/computer-systems mechanisms without assuming the analogy |
| Targeted research | 2, 10, 17, plus experiment-exposed gaps | close decision-blocking evidence/concept gaps |
| Architecture | 18 | promote only after repeated empirical evidence |

This is a prioritization layer, not a replacement for the detailed agenda.

## Evidence hierarchy and research intake

Use community projects, practitioner reports, academic papers and benchmarks as **signals of what to investigate**. Do not equate popularity with effectiveness.

Use an explicit confidence ladder:

```text
community signal
  -> adoption / repeated reports
  -> academic controlled result
  -> independent reproduction
  -> cross-model/repository replication
  -> EOKS workload-specific evidence
```

This ladder is a useful **evidence-strength progression for a particular claim**, not a ranking of communities versus academia. Different claims may enter at different points, and a strong community signal may justify an experiment before academic formalization exists.

Record contradictory results rather than selecting only supporting evidence. For quantitative claims, preserve the workload, model, sample size, metric, effect size and uncertainty/limitations rather than copying a headline percentage.

**Exit condition:** each major claim driving an experiment has a traceable evidence basis and an explicit uncertainty/status rather than an implicit assumption.

## Existing research agenda

The remaining agenda sections retain the existing detailed research program: context quality and acquisition; context lifecycle; context workbench; context contracts; execution state; durable memory and procedural learning; model × infrastructure; repository maturity; knowledge representations; deterministic evidence and analysis escalation; invariants; evidence-provider selection; orchestration; control-plane/adaptive policies; assurance/reliability/governance; minimum semantic model; benchmark methodology; evidence-driven bottleneck map; computer-systems optimization transfer; systems benchmark matrix; analogy boundaries; non-assumptions; and AI-native SDLC as a proving ground.

These sections should now be executed through the priority and measurement framework above rather than treated as a flat list of equally urgent research questions.

## Decision and exit criteria

Each agenda item should end in an explicit disposition rather than remaining indefinitely “open”. The available outcomes are:

- **Use / integrate** — existing mechanism demonstrates sufficient benefit and reliability.
- **Prototype** — the capability appears valuable but existing mechanisms do not provide the required control semantics.
- **Experiment further** — evidence is mixed, workload-dependent or underpowered.
- **Research first** — a measurement or conceptual gap prevents a meaningful experiment.
- **Watch** — plausible but currently low leverage or insufficient evidence.
- **Reject / archive** — the hypothesis is contradicted, redundant or not worth its cost.
- **Architectural promotion** — repeated evidence demonstrates a stable missing EOKS concept/relationship.

A research question should be promoted in priority when it changes an imminent experiment, resolves a decision-blocking ambiguity, or explains an important observed failure. A research question should be deferred when it is interesting but does not currently affect an actionable decision.

### Minimum experiment record

Every completed experiment should record:

```text
concept / capability
operational hypothesis
task + workload class
model + version
baseline
intervention
signals captured
metrics
sample / repetitions
effect + uncertainty
failure signatures
limitations / confounders
result disposition
EOKS synthesis impact
next action
```

This record makes the agenda self-updating: experimental evidence changes priorities rather than merely adding another research note.

## Near-term sequence

The practical sequence is intentionally narrow:

```text
1. instrument workload
2. measure strong baseline
3. context / working-set intervention
4. evidence-provider selection intervention
5. computation / artifact reuse intervention
6. compare outcomes, assurance, efficiency and cost
7. evaluate promising mechanisms against the demonstrated gaps
8. perform targeted research where evidence is insufficient
9. prototype missing EOKS control capabilities
10. repeat across workload/model/repository conditions
11. update synthesis + agenda
12. promote architecture only when evidence warrants it
```

This sequence does not prohibit parallel research or exploration. It establishes a default priority so the project does not drift back into research accumulation without empirical closure.

The central EOKS objective remains:

> **Discover which capabilities improve trustworthy software-agent outcomes, under which conditions, and whether their benefit justifies their complexity and cost.**
