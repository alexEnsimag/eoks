# Cross-repository, cross-language system representation gap

EOKS has several promising repository/code representation providers, but there is an unresolved gap between **repository-local code understanding** and an **end-to-end representation of a software system**.

## The target capability

The target is not simply a code graph. We want a navigable representation that can connect:

```text
code
  → symbols / modules
  → semantic/domain concepts
  → repositories / projects
  → services
  → APIs / RPC
  → queues / events
  → datastores
  → deployment/runtime relationships
  → evidence (tests, traces, configs)
```

It should work across languages and repository boundaries and support both human exploration and agent retrieval.

## What the current tools contribute

The current research suggests the gap is best understood as several partially overlapping layers:

| Layer | Candidate contribution | What remains unresolved |
|---|---|---|
| **Code / semantic** | **Understand Anything**, Graphify | breadth, relationship accuracy, incremental updates, multi-repo joins |
| **Project / dependency** | **Nx** | whether synthetic-monorepo/project graphs can provide a useful stable cross-repo system boundary without becoming the EOKS architecture |
| **Cross-repository code intelligence** | **Sourcegraph** | how much of its platform-scale indexing/navigation model EOKS actually needs |
| **System / architecture** | **Structurizr** | connecting architecture models to actual code and observed runtime behavior |
| **Visualization** | **CodeSee**, Understand Anything, Structurizr | unified navigation across code, project and system layers |
| **Static/evidence** | **Semgrep**, **CodeQL** | joining deterministic findings to higher-level system entities |
| **Runtime evidence** | traces/telemetry and future EOKS providers | mapping observed behavior back to static/system representation |

This is why a single "best graph tool" is probably the wrong target. The interesting question is whether a small composition can cover the layers needed by real engineering questions.

## Why repository-local graphs are insufficient

A graph can be excellent at answering:

- where is this symbol?
- what calls this function?
- what depends on this module?
- where is this concept implemented?

while still being unable to answer:

- which repository owns the other side of this API?
- which service consumes this event?
- which deployment connects these components?
- what is the end-to-end path from this user action to this datastore?
- which code implements the same system capability across repositories and languages?

Those are the questions the missing system-level representation must address.

## Research directions

Investigate providers in distinct families rather than searching only for "AI code graph":

1. **Polyglot static/code intelligence** — Understand Anything, Graphify and related language-aware providers.
2. **Cross-repository dependency/project graphs** — Nx and similar project-graph approaches.
3. **Cross-repository semantic code intelligence** — Sourcegraph as a strong platform-scale baseline.
4. **Architecture/service maps** — Structurizr and related system-level models.
5. **Architecture visualization** — CodeSee, Understand Anything and Structurizr as different visualization layers.
6. **Static evidence** — Semgrep and CodeQL for deterministic relationships/findings.
7. **Runtime topology** — traces and telemetry that connect static structure to observed behavior.
8. **Unified code + runtime models** — systems that explicitly join static and runtime evidence.

## Evaluation criteria

For each candidate, record:

- languages supported
- repository/project scope
- whether multiple repositories can be represented in one model
- supported relationship types
- code vs project vs system vs runtime evidence
- semantic/LLM-derived vs deterministic relationships
- visualization quality and navigation depth
- machine-readable export/API
- incremental update behavior
- provenance/explainability
- harness/agent integration
- local vs external service requirements
- operational cost

## EOKS representation hypothesis

The likely answer may not be one tool.

A promising architecture is a **provider composition**:

```text
Understand Anything / semantic code
             +
Nx / project and cross-repo dependencies
             +
Sourcegraph-style cross-repo intelligence where needed
             +
Structurizr / system architecture
             +
CodeSee / other visualization
             +
Semgrep / CodeQL / runtime evidence
             ↓
      EOKS system representation
             ↓
      task-specific synthesis
             ↓
      context / artifact
```

The EOKS layer should remain responsible for deciding which representation is needed for a question. No provider should become canonical truth merely because it has the best graph or visualization.

This also avoids the **ultimate graph** trap: EOKS does not need to materialize every possible relationship in one permanent graph if task-specific views can be computed from smaller authoritative representations.

## Relationship to evolving knowledge

The system representation should not be confused with persistent memory. A code/project/system representation answers **what the system is and how its parts relate**; evolving memory answers **what EOKS learned from prior work, why decisions were made, what evidence supported them, and what should influence future work**.

Second-brain tools such as **Obsidian, Notion, Capacities, Tana, Roam and Mem** are therefore relevant as representation/capture/memory prior art, but they do not by themselves solve this software-system representation problem. **Soda-style passive capture** addresses a different upstream problem: how observations enter the system without making the human manually maintain it.

The boundary is:

```text
work
  ↓
observation / capture
  ↓
representation + evidence
  ↓
synthesis
  ↓
knowledge that changes future work
```

## Phase A outcome

Phase A should produce an explicit answer to:

> What is the smallest practical provider combination that can represent a real polyglot, multi-repository system well enough for both human navigation and agent context construction?

The experiment should also determine whether that representation is **computed on demand** for a task or maintained as a durable unified graph.

If no existing combination is satisfactory, the missing capability itself becomes an EOKS design/research target.
