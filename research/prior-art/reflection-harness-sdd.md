# Reflection harnesses, durable artifacts and progressive validation

## Source

Alexandre Cukier, *Reflection SDD: Use a Reflection Harness to Level Up Your OpenSpec Workflow* (Data Leads Future, 2026).

[Source article](https://www.dataleadsfuture.com/reflection-sdd-use-a-reflection-harness-to-level-up-your-openspec-workflow/)

This note records the article as **practitioner evidence**, not as a validated benchmark or an EOKS architectural decision.

## Why this matters to EOKS

The article describes a Spec-Driven Development workflow in which an agent produces durable proposal artifacts and a separate reflection step reviews those artifacts before implementation. The author reports that an initially broad review/fix loop created inconsistencies between artifacts, and changed the workflow to generate, review and freeze artifacts incrementally.

The EOKS-relevant observation is broader than "use a reflection agent":

> **A workflow can insert explicit evaluation boundaries between computational stages, materialize intermediate results, and progressively constrain subsequent computation by freezing validated state.**

This is closely related to existing EOKS work on durable artifacts, intermediate evidence, evaluation, trajectories and reusable computation, but does not require a new EOKS primitive.

## Reflection is not simply "thinking first"

In this workflow, reflection is an explicit evaluation operation over an already-produced representation:

```text
produce artifact
      |
      v
independent inspection / critique
      |
      v
identify defects
      |
      v
revise
      |
      v
re-evaluate
```

Reflection can therefore happen **before implementation** when the object being evaluated is a plan, specification or design, rather than code. It is better understood as a control/evaluation stage than as a claim about hidden model reasoning.

This distinction matters because EOKS should be concerned with observable workload state and evidence, not with assuming that an additional internal reasoning pass is intrinsically reliable.

## Artifact types and the pre-computed-layer hypothesis

The article uses artifacts such as an exploration brief, proposal/specification documents and review logs. These are persistent representations of decisions, requirements, reasoning outcomes and evaluation state.

They connect to the EOKS hypothesis that **computation can be materialized into reusable representations**:

```text
computation / reasoning
          |
          v
    representation
          |
       validate
          |
          v
     persist state
          |
          v
   later computation
```

However, a document artifact should not be conflated with a pre-computed computational layer.

- An exploration brief is primarily a **persisted knowledge/decision representation**.
- A review log is primarily **evaluation/provenance state**.
- A hypothetical cached API-flow model or dependency computation would be a **pre-computed representation** that can avoid repeating computation.

The common architectural property is **materialization and reuse of validated intermediate state**. The semantics and reuse mechanism differ.

## Progressive freezing

The article's most interesting mechanism is the change from reviewing a collection of mutually dependent artifacts at once to a sequential pattern:

```text
artifact A
    |
 review
    |
 freeze A
    |
    v
artifact B, constrained by A
    |
 review
    |
 freeze B
    |
    v
artifact C, constrained by A + B
```

The benefit claimed by the author is reduction of oscillating edits such as fixing A, breaking B, fixing B, and then discovering a new inconsistency in A.

The architectural interpretation should be stated carefully. Freezing does **not guarantee semantic correctness**. It provides structural immutability and reduces the space of later regressions: subsequent stages cannot silently rewrite already-frozen state. Correctness still depends on the evaluation evidence used at each boundary.

This suggests a useful EOKS hypothesis:

> **Validated intermediate state can act as a constraint on later computation, reducing rework and regression surface when dependencies are predominantly one-way.**

That hypothesis is testable without adopting the author's particular file structure or workflow.

## Externalizing ephemeral reasoning

The article introduces an exploration brief because useful findings from an earlier conversation may otherwise be lost when context is compressed or a new session starts. A review log similarly preserves findings from iterative evaluation.

This reinforces an existing EOKS principle:

```text
session / transient computation
             |
             v
      durable representation
             |
       provenance + scope
             |
             v
     later context/workload
```

The important property is not Markdown itself. It is the decision to materialize information that later computation cannot reliably reconstruct from transient context.

This also supports the EOKS distinction between **context** and **knowledge/state**: putting more text into a context window is not equivalent to preserving a reusable representation outside that context.

## What the source actually demonstrates

The article provides useful practitioner evidence that:

1. Reviewing intermediate SDD artifacts can reveal defects before implementation.
2. Persisting exploration and review information can preserve useful state across sessions.
3. Incremental review/freeze can make a multi-artifact workflow easier to control than unrestricted simultaneous editing.
4. A bounded number of reflection rounds plus escalation provides an explicit termination policy.

These are observations from one workflow, not controlled causal evidence.

## Claims that should not be adopted as established fact

Several stronger claims in the article need qualification:

- A reported comparison between models is not a controlled benchmark and should not be used as evidence of a general reflection uplift.
- Reflection is not demonstrated to require a different LLM. Model diversity, independent context, additional sampling and an additional pass are confounded in the described workflow.
- Repeated review does not prove correctness. A reviewer can share the generator's assumptions or reinforce an incorrect artifact.
- Freezing does not provide a semantic consistency guarantee; it mainly constrains later mutation.
- A fixed reflection-round limit such as five is a workflow policy, not an EOKS constant.

These caveats are important because EOKS should preserve competing hypotheses rather than turn a practitioner pattern into a universal mechanism.

## EOKS mapping

| Source mechanism | EOKS interpretation |
|---|---|
| Reflection harness | explicit evaluation stage over an intermediate representation |
| Proposal/spec/design artifact | durable workload representation |
| Exploration brief | materialized knowledge/decision state from transient work |
| Review log | evaluation/provenance state |
| Freeze | constrain subsequent mutation of validated state |
| Sequential artifact generation | dependency-aware workflow with validation boundaries |
| Reflection rounds | bounded iterative control policy |
| Human escalation | fallback when current evidence cannot establish sufficient assurance |

The workflow itself should remain a **workload-level strategy**, not a new EOKS ontology.

## Relationship to existing EOKS work

This source strengthens several existing lines of research:

- **AI-native SDLC / SDD:** durable artifacts are already modeled as representations of intent, specifications, plans, evidence and outcomes. Reflection adds explicit validation between handoffs.
- **Control loop:** reflection is a local evaluate/reconcile cycle within a larger workload loop.
- **Intermediate evidence:** artifacts can make otherwise transient process state inspectable and reusable.
- **Trajectory evaluation:** review logs and iteration history expose process evidence in addition to final outcomes.
- **Pre-computed/reusable representations:** materialized intermediate state may reduce repeated reasoning or context reconstruction, but only when its freshness, validity and applicability are established.

The article therefore supplies a concrete practitioner example that helps connect these existing concepts; it does not justify adding a standalone "reflection" primitive.

## Research questions

The useful follow-up questions are empirical:

1. Does evaluating a proposal before implementation reduce downstream defects compared with code-only review?
2. Does progressive freezing reduce rework or inconsistency compared with unrestricted artifact revision?
3. When does an independently generated critique outperform another pass by the original generator?
4. Does model diversity improve defect detection, and independently of that, does context diversity matter?
5. When can a materialized intermediate computation be reused safely instead of recomputed?
6. How should freshness and invalidation be represented for such reusable state?
7. What evidence should terminate a reflection loop: evaluator agreement, deterministic checks, outcome evidence, bounded rounds, or some combination?

These questions fit EOKS's existing evaluation agenda and are more valuable than prescribing a particular reflection harness.

## Boundary

EOKS should not require:

- a reflection agent;
- a particular SDD framework or OpenSpec workflow;
- Markdown artifacts;
- a second model for critique;
- a fixed number of review rounds;
- freezing every intermediate artifact.

The reusable EOKS concepts are **evaluation boundaries, durable representations, provenance, validation, constrained state transitions, reuse and explicit termination/escalation policy**.
