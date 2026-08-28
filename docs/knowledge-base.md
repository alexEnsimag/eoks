# Knowledge base and persistent project knowledge

## Motivation

A recurring development problem is that an agent such as Claude Code can spend a substantial part of every new session rediscovering the same repository facts. A large context window does not solve this: the model still needs to locate, select, and re-read the evidence required for the current task.

This suggests treating **project knowledge as a durable resource** rather than rebuilding repository understanding inside every session.

The goal is not to create a giant second copy of the repository. It is to preserve high-value mental-model information and make authoritative evidence easier to find.

## Knowledge base vs context

These are different layers:

- **Knowledge base** — durable, reusable information about the project and its history.
- **Retrieval / navigation** — selection of relevant knowledge and evidence.
- **Context** — the concrete information assembled for one reasoning step.
- **Memory** — information intentionally retained about previous work, decisions, outcomes and preferences.

The knowledge base should therefore not be dumped wholesale into the prompt.

```text
                 durable project reality
                         |
                 knowledge extraction
                         |
             multiple representations
             / graph / semantic / docs \
                         |
                     retrieval
                         |
                 context compilation
                         |
                       model
                         |
                      outcome
                         |
                 evaluation / updates
```

## Knowledge is multi-representational

A graph is one representation of knowledge, not knowledge itself. Different representations answer different questions:

- **Structural graph** — what connects to what?
- **Semantic index** — what concepts are related?
- **Historical timeline** — how did we get here?
- **Runtime model** — what actually happens?
- **Canonical documents** — what do humans explicitly say is true or required?
- **Knowledge objects** — why does a decision/invariant/pattern exist?
- **Workflow model** — what should happen next?

This resembles compiler intermediate representations: the same underlying system is represented differently for different analyses.

See [Engineering knowledge as a multi-representation system](knowledge-representations.md).

## Canonical knowledge: start with files

A first implementation does not need a graph database or a new machine-readable knowledge format.

For an individual developer or small team, hierarchical `CLAUDE.md` files are a strong canonical representation because they are:

- close to the code they describe;
- human-readable;
- versioned with Git;
- reviewable in pull requests;
- naturally scoped by directory;
- directly useful to coding agents.

A practical layout is:

```text
/
  CLAUDE.md
  api/
    CLAUDE.md
  auth/
    CLAUDE.md
  architecture/
    *.md
```

A package `CLAUDE.md` should primarily contain the mental model that code alone does not express:

- purpose and responsibilities;
- boundaries;
- important invariants and constraints;
- entry points / recommended reading order;
- common pitfalls;
- links to cross-cutting decisions.

It should not become a function-by-function duplicate of the implementation.

Cross-cutting decisions can remain ordinary Markdown/ADR documents. They do not need to be forced into a separate OKF-like format unless a later use case justifies it.

## Canonical knowledge vs derived indexes

A useful architecture is:

```text
Human-reviewable canonical knowledge
  CLAUDE.md / ADRs / design docs
              |
              v
       derived representations
  graph / semantic index / symbol index
  impact cache / retrieval metadata
```

Machines can maintain derived indexes. Humans should own the promotion of important project intent and rationale.

This avoids making a generated graph or summary the accidental source of truth.

## Referential vs synthetic knowledge

This distinction helps control maintenance cost.

### Referential knowledge

The information already exists somewhere authoritative. The system should usually **point to it and rank it**, rather than duplicate it.

Examples:

- source code;
- API specifications;
- commits and pull requests;
- tests;
- incident records;
- runtime dashboards.

### Synthetic knowledge

The information does not exist in one authoritative source and must be preserved explicitly.

Examples:

- architectural rationale;
- cross-package invariants;
- tradeoffs and rejected alternatives;
- lessons learned;
- a debugging discovery that would otherwise disappear with the session.

This is where durable Markdown knowledge and later knowledge objects provide the most value.

## Automatic extraction vs curation

Full manual curation does not scale. Full automatic generation is also unsafe because summaries can become stale, lose nuance, or promote an incorrect inference into durable knowledge.

A better model is **automatic extraction followed by controlled promotion**:

```text
raw observations
      |
      v
automatic candidates
      |
      v
validation + provenance
      |
      v
canonical project knowledge
```

The important distinction is between a **candidate** and a **trusted fact**.

## Incremental knowledge maintenance

The knowledge compiler should behave more like an incremental build system than a nightly full rebuild.

A small code change should not cause every representation to be recomputed. Different passes should have different dependencies and update costs:

```text
commit
  |
  +--> structural update (deterministic)
  |
  +--> affected semantic entries (if needed)
  |
  +--> invalidate affected context caches
  |
  +--> candidate knowledge review only if the change is meaningful
```

Most commits should not invoke an LLM knowledge pass. Deterministic analysis should handle cheap changes; deeper reasoning can be triggered by higher-signal events such as merged PRs, incidents, architectural changes or completed agent workflows.

A useful operational model is:

- **fast/incremental** — graph/index/cache updates;
- **normal** — merged-change and documentation/decision extraction;
- **deep** — periodic consolidation, contradiction detection and stale-knowledge review.

## Knowledge lifecycle

Durable knowledge needs lifecycle semantics:

1. **Observe** — capture repository changes, agent interactions, analysis results and decisions.
2. **Extract** — generate candidate facts and summaries.
3. **Validate** — check candidates against evidence and outcomes.
4. **Promote** — make validated knowledge durable.
5. **Retrieve** — select only task-relevant knowledge.
6. **Invalidate/update** — detect when source evidence changes.
7. **Evaluate** — measure whether the knowledge actually improved task outcomes.

Hooks are best understood as event boundaries in this lifecycle, not merely scripts that run after an agent command.

## Evidence and confidence

Important derived statements should ideally point back to evidence:

- source files and symbols;
- commits or pull requests;
- tests;
- documentation;
- analysis results;
- runtime observations.

Confidence should describe the evidence behind a claim, not only the model's subjective confidence. Useful statuses include `EXTRACTED`, `INFERRED`, and `CANDIDATE/AMBIGUOUS`.

## The long-term direction

A mature EOKS may eventually provide a portable knowledge protocol or richer structured objects such as OKF. But the first implementation should not introduce a new format merely for theoretical interoperability. The canonical artifact should be the simplest representation that developers will keep accurate.

For many repositories, that may be hierarchical `CLAUDE.md` + a small number of ADR/architecture documents, with graphs and semantic indexes acting as derived evidence providers.
