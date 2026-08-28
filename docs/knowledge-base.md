# Knowledge base and persistent project knowledge

## Motivation

A recurring development problem is that an agent such as Claude Code can spend a substantial part of every new session rediscovering the same repository facts. A large context window does not solve this: the model still needs to locate, select, and re-read the evidence required for the current task.

This suggests treating **project knowledge as a durable resource** rather than rebuilding repository understanding inside every session.

## Knowledge base vs context

These are different layers:

- **Knowledge base** — durable, reusable information about the project and its history.
- **Retrieval** — selection of knowledge relevant to a task.
- **Context** — the concrete information assembled for one reasoning step.
- **Memory** — information intentionally retained about previous work, decisions, outcomes, and preferences.

The knowledge base should therefore not be dumped wholesale into the prompt. It should sit behind a retrieval boundary.

```text
                 durable project reality
                         |
                 knowledge extraction
                         |
                 +-------+-------+
                 |               |
             knowledge        provenance
               store           + freshness
                 |
             retrieval
                 |
          task-specific context
                 |
              model
                 |
             outcome
                 |
          evaluation / updates
```

## What should be stored

A useful project knowledge base should contain at least four categories:

### 1. Structural knowledge

Facts that change relatively slowly:

- repository/module structure;
- important components and responsibilities;
- dependencies and interfaces;
- architectural relationships;
- configuration and deployment topology;
- important data flows.

### 2. Semantic/project knowledge

Knowledge that is difficult to infer reliably from file structure alone:

- why a component exists;
- important invariants;
- domain concepts;
- known constraints;
- recurring patterns and conventions;
- known failure modes and gotchas.

### 3. Historical knowledge

Knowledge accumulated while developing the project:

- architectural decisions and their rationale;
- rejected alternatives;
- important debugging discoveries;
- changes in direction;
- lessons from incidents or failed approaches.

### 4. Evidence links

Every important derived statement should ideally point back to evidence:

- source files and symbols;
- commits or pull requests;
- tests;
- documentation;
- analysis results;
- runtime observations.

This prevents a generated summary from silently becoming the source of truth.

## Automatic extraction vs curation

Full manual curation does not scale. Full automatic generation is also unsafe because summaries can become stale, lose nuance, or promote an incorrect inference into durable knowledge.

A better model is **automatic extraction followed by controlled promotion**:

```text
raw observations
      |
      v
automatic summaries / candidate facts
      |
      v
candidate knowledge
      |
  validation + provenance
      |
      v
canonical project knowledge
```

The important distinction is between a **candidate** and a **trusted fact**. An automated tool may be allowed to generate candidates continuously, while promotion into canonical knowledge requires stronger evidence or explicit validation.

## A practical representation

The first implementation does not need a graph database or hosted service. Plain files can be the canonical representation if they have a clear contract and provenance.

A useful layout is:

```text
knowledge/
  architecture/
  concepts/
  decisions/
  patterns/
  gotchas/
  history/
  evidence/
  index.md
```

Each entry can be Markdown with structured metadata and links to evidence. The representation should remain portable so that Claude Code, another coding agent, scripts, or a future EOKS runtime can consume the same knowledge.

## The OpenWolf lesson

OpenWolf is interesting because it automates part of this problem: it maintains repository-oriented summaries and persistent notes around an agent's file interactions. That makes it a useful **knowledge extraction/context optimization layer** even if it should not automatically be treated as the canonical knowledge base.

The architectural lesson is more general than OpenWolf itself:

> Automatic observation of agent/repository interactions can dramatically reduce the cost of reconstructing project context, but generated memory should be treated as evidence-bearing knowledge with lifecycle and provenance—not as an unquestioned database.

## Knowledge lifecycle

Durable knowledge needs lifecycle semantics:

1. **Observe** — capture repository changes, agent interactions, analysis results and decisions.
2. **Extract** — generate candidate facts and summaries.
3. **Validate** — check candidates against evidence.
4. **Promote** — make validated knowledge durable.
5. **Retrieve** — select only task-relevant knowledge.
6. **Invalidate/update** — detect when source evidence changes.
7. **Evaluate** — measure whether the knowledge actually improved task outcomes.

This is a stronger abstraction than "memory" because it makes freshness, provenance and invalidation explicit.

## Key design principle

The goal is not to remember everything. It is to make the **minimum sufficient, high-signal project knowledge available cheaply and reliably**.

That makes knowledge-base quality a first-class EOKS concern, alongside context quality, model selection and execution quality.
