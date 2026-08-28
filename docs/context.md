# Context engineering

Context engineering is the discipline of constructing the information available to a model for a specific reasoning step.

The key distinction is:

> **Knowledge is persistent; context is compiled for a task.**

Context engineering is therefore not a competing layer to knowledge engineering. It is the runtime boundary that selects and assembles the right slice of persistent knowledge and evidence.

## Context is not storage

A repository, knowledge base, graph or memory store can contain far more information than should enter a model's context. The important operation is the **selection and transformation boundary** between external knowledge and model input.

A useful mental model is:

```text
persistent engineering reality
          |
   knowledge representations
          |
   task + workflow + strategy
          |
   context compilation
          |
   minimal sufficient context
          |
         model
```

The model should usually see the **compiled evidence**, not the internal graph, embedding index or storage system itself.

## Navigation versus knowledge

A particularly useful distinction is between two goals:

1. **Navigation optimization** — determine where the relevant evidence lives and in what order it should be read.
2. **Knowledge optimization** — preserve insights that do not otherwise exist in source material, such as rationale, tradeoffs, invariants and lessons learned.

A structural graph or semantic index can often satisfy the first goal by pointing the agent to relevant files, symbols, ADRs or incidents. The agent can then read the authoritative source rather than receiving a duplicated summary.

Synthetic knowledge is different: if a cross-package invariant or debugging discovery exists nowhere else, it needs a durable representation.

## Canonical project knowledge

A practical coding-agent environment can use hierarchical `CLAUDE.md` files as a canonical, human-reviewable representation of local project knowledge:

```text
/
  CLAUDE.md
  api/
    CLAUDE.md
  auth/
    CLAUDE.md
```

The purpose should be mental-model information rather than a second copy of the code:

- why the package exists;
- responsibilities and boundaries;
- invariants and important constraints;
- entry points and reading order;
- common pitfalls;
- links to cross-cutting architecture decisions.

This is especially attractive because the files live beside the code, are versioned with Git and can be reviewed in normal pull requests.

The hierarchy should remain scoped. Repository-wide instruction files should not become encyclopedias. More local guidance is useful when the agent is working in that part of the tree, while unrelated package knowledge should remain out of the context budget.

## Knowledge representations are not context

Graphs, semantic indexes, timelines, ADR collections and runtime stores are **evidence providers**. They should normally be queried by a context compiler rather than dumped into the model context.

For example:

```text
Task: change authentication
        |
        +--> structural query -> affected packages/files
        +--> semantic query   -> relevant auth concepts
        +--> history query    -> authentication ADRs/incidents
        +--> canonical docs   -> api/CLAUDE.md + auth/CLAUDE.md
        |
        v
  task-specific context
```

This also explains why a graph can be valuable without becoming the canonical knowledge base.

## Context quality

A useful context-quality model should consider:

- relevance to the task;
- correctness and source reliability;
- freshness;
- completeness;
- redundancy;
- contradictions;
- provenance;
- ordering/structure;
- token and latency cost;
- interaction with the chosen model;
- applicability of retrieved procedures to the current task.

The goal is not maximum information. It is maximum useful evidence per unit of context and reasoning cost.

## Static versus dynamic context

Large repository-wide instruction files can create a poor tradeoff: they are always available but may be irrelevant to the current task. EOKS should prefer **progressive disclosure** and task-scoped retrieval.

This does not imply that static documentation is bad. Well-scoped package-level `CLAUDE.md` files can be valuable precisely because they are local, concise and maintained as part of the codebase. The important distinction is between useful local guidance and indiscriminate global context stuffing.

## Context splitting

Different reasoning steps often need different context. Splitting context can reduce noise and make decisions inspectable: discovery context, design/planning context, implementation context, verification context and historical/project context can be assembled separately.

A workflow node can therefore request a different context package than another node even for the same task.

## Progressive disclosure

The system should prefer exposing the minimum sufficient information and retrieving additional detail when evidence shows it is needed. This resembles filesystem/document navigation more than stuffing an entire corpus into a prompt.

A useful pattern is:

```text
compact pointer / summary
        |
        +--> authoritative source
        |
        +--> related evidence
```

## Continuous knowledge lifecycle

Claude Code plugins and hooks expose a useful concrete workflow for studying this problem. An execution workflow can produce decisions, failures, tool traces and review outcomes that become inputs to the knowledge lifecycle.

```text
session start
    |
retrieve canonical knowledge + relevant evidence + applicable procedure
    |
work / execute / verify
    |
session end
    |
extract candidate facts, episodes, rules, skills and decisions
    |
validate selectively against source + outcome evidence
    |
promote / update / invalidate
    |
future retrieval and workflow selection
```

The important architectural boundary is between **observation**, **candidate extraction**, **validation/promotion**, and **retrieval**. A session-finalizer hook is therefore one instance of the general EOKS knowledge lifecycle rather than a special-case memory feature.

## Open problem

We need empirical benchmarks showing when a context intervention improves task success, rather than assuming that more structure or more retrieved tokens are beneficial. EOKS should measure at least:

- repository-discovery tool calls;
- time/token cost;
- task success;
- regression/error rate;
- stale-knowledge incidents;
- usefulness of persistent knowledge;
- whether a context intervention improves the outcome for a particular model.
