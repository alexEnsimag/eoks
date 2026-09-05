# EOKS research and synthesis methodology

EOKS is a living specification and laboratory. Its architecture should emerge from the interaction between research, observed practice, existing systems and experiments rather than from any one source of authority.

The purpose of this methodology is to describe **how EOKS turns signals into testable engineering knowledge and, when justified, into architectural decisions**.

## The central idea

EOKS deliberately combines four evidence inputs:

- **Academic research** — formal concepts, terminology, controlled methods, theoretical framing and measured results. Research can clarify and formalize concepts that are difficult to name from practice alone, while also lagging rapidly evolving agent practice.
- **Community and practitioner evidence** — emerging techniques, real-world failure modes, workflow reports, implementation experience and tooling. This evidence can arrive substantially earlier than formal research and can be highly relevant to real workloads, while often being less controlled or harder to compare directly.
- **Existing systems and tools** — concrete mechanisms that show what can already be implemented. They provide prior art and candidate interventions, not automatic architectural requirements.
- **EOKS experiments** — controlled or workflow-based tests that determine whether an observed mechanism or research result transfers to the workloads, objectives and control problems EOKS cares about.

None of these sources is automatically authoritative. Their roles are complementary.

A useful mental model is triangulation:

```text
                 ┌── academic research ────────┐
                 │                             │
  observations ──┼── community / practice ──────┼──→ hypothesis
                 │                             │
                 └── existing systems / tools ─┘
                                               ↓
                                          EOKS experiment
                                               ↓
                                            evidence
                                               ↓
                                      synthesis / decision
                                               ↓
                                  architecture, research or use
```

This is **not a mandatory linear workflow**. A paper can reveal a concept that leads directly to an EOKS experiment. A community failure report can expose a problem before there is a paper about it. An existing implementation can reveal a capability that prompts a research question. An EOKS experiment can expose a gap that sends us back to the literature or community.

The important property is the feedback between the sources.

## From signal to architecture

EOKS should preserve the path by which important claims are formed:

```text
signal / observation
      ↓
interpretation
      ↓
operational hypothesis
      ↓
signals + metrics + evaluation method
      ↓
experiment / comparison / workflow trial
      ↓
evidence + limitations
      ↓
synthesis
      ↓
decision or disposition
      ↓
implementation / research follow-up / architectural promotion
```

The chain is deliberately explicit because a useful observation is not the same thing as a validated mechanism, and a validated mechanism is not automatically an EOKS architectural primitive.

Architecture should be promoted only when the evidence demonstrates a stable capability, relationship or control problem that the existing model cannot adequately express. Otherwise the result may remain a research finding, implementation choice, workflow practice or candidate capability.

This gives EOKS a useful separation:

| Layer | Question |
|---|---|
| **Signal** | What did we observe or discover? |
| **Interpretation** | What might it mean? |
| **Hypothesis** | What should happen if the interpretation is useful? |
| **Experiment** | How can we test it? |
| **Evidence** | What happened, under which conditions, and with what limitations? |
| **Synthesis** | How does this change our model of the problem? |
| **Decision** | What should we use, prototype, research, watch or reject? |
| **Architecture** | Has a stable missing abstraction or relationship actually earned promotion? |

## Evidence is multidimensional

EOKS should not use a single universal ladder in which community evidence is simply a lower-confidence form of academic evidence.

Evidence strength depends on the claim and on dimensions such as:

- methodological control;
- measurement quality;
- replication and independence;
- workload and task relevance;
- model, repository and environment coverage;
- recency;
- effect size and uncertainty;
- observed failure modes and contradictory results;
- ability to reproduce the result;
- direct evidence from the EOKS workload.

Different evidence sources are strong on different dimensions. A recent practitioner report may be the best signal that an emerging failure mode exists; a controlled paper may provide the best formalization of the underlying mechanism; an existing system may provide the strongest evidence that a mechanism is practical; and an EOKS experiment may provide the strongest evidence about whether that mechanism matters for the project's workload.

Therefore EOKS should record **what kind of evidence supports a claim and what remains uncertain**, rather than assigning the source a fixed rank.

## Community evidence is a first-class research input

The community evidence corpus is not merely a backlog of informal links. It is a research instrument for detecting emerging patterns, failure modes, techniques and unanswered questions.

The current evidence intake is maintained in [Community and academic evidence on agent bottlenecks](../research/community-evidence-bottlenecks.md). It should preserve effect sizes where available, experimental settings, implementation details, community failure reports, contradictory evidence and proposed next tests.

The corpus is deliberately non-normative: inclusion means that something is worth tracking or testing, not that EOKS accepts the claim.

Its role is especially important because the development cycle of agent systems can move faster than academic publication. Community evidence can therefore lead formal research rather than merely follow it. Conversely, academic work can provide the terminology and experimental discipline needed to turn an initially vague practitioner observation into a precise hypothesis.

## Synthesis is the bridge

Synthesis is where EOKS connects evidence without silently converting any individual source into architecture.

For a concept or mechanism, synthesis should answer questions such as:

1. **What was actually observed?**
2. **Which sources independently support or contradict it?**
3. **What concept or relationship best explains the observations?**
4. **What remains hypothesis rather than established knowledge?**
5. **What workload-specific experiment would reduce the uncertainty?**
6. **Does an existing EOKS concept already represent the capability?**
7. **If not, is the gap architectural, implementation-level, methodological or simply a missing tool/mechanism?**
8. **What decision follows, and what evidence would change it?**

This prevents two common failure modes:

- **research accumulation without closure** — collecting papers, projects and concepts without determining what they imply for EOKS;
- **premature architecture** — turning an interesting paper, popular tool or compelling practitioner pattern into a canonical abstraction before testing its boundaries.

## Research and experimentation proceed together

The methodology does not require EOKS to finish research before building or experimenting.

Instead:

```text
research ───────────────┐
community signals ──────┤
existing mechanisms ────┼──→ hypotheses → experiments → evidence
observed failures ──────┘                         ↑        │
                                                  └────────┘
```

Experiments should determine which unanswered research questions deserve deeper investigation. Research should improve the quality of experiments and help explain unexpected results.

This is why the [Research agenda](research-agenda.md) is experiment-first without being research-free: the agenda decides **what to investigate next**, while this methodology defines **how research, practice and experiments interact**.

## Traceability and epistemic status

Claims that can influence EOKS architecture should be traceable to their supporting evidence and should retain an explicit epistemic status. Useful statuses include:

- **Observed** — directly observed in a workload or implementation.
- **Supported** — supported by sufficient evidence for the current use.
- **Mixed** — evidence exists but results vary by condition or source.
- **Hypothesis** — plausible interpretation that still needs testing.
- **Research** — important unanswered question or concept under investigation.
- **Rejected** — evidence currently contradicts or invalidates the claim.
- **Canonical** — explicitly adopted into the current EOKS architecture.

These statuses are not permanent. New evidence can move a claim in either direction.

The project should also preserve the distinction between **epistemic status** and **action status**. A hypothesis may be worth prototyping; a well-supported mechanism may still be unsuitable for the current workload; and a canonical architectural relationship can later be revised.

## What makes this EOKS-specific

EOKS is not intended to become another literature survey. Nor is it intended to become a collection of practitioner tricks.

Its distinctive goal is to use both as inputs to a **testable engineering knowledge loop**:

> **Turn formal research, community practice and existing mechanisms into explicit hypotheses; test them against real engineering workloads; preserve the evidence and limitations; synthesize what transfers; and only promote abstractions that earn their place through repeated evidence.**

This methodology also means that contradictions are valuable. A paper, community report, tool and EOKS experiment do not need to agree. Disagreement can reveal workload boundaries, missing variables, measurement problems or concepts that have been conflated.

The objective is therefore not to maximize agreement with prior art. It is to improve the quality of EOKS's explanations, experiments and decisions over time.
