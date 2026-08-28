# Engineering knowledge as a multi-representation system

A central conclusion from recent EOKS research is that **knowledge is not a graph**. A graph is one representation of knowledge, optimized for questions about relationships and structure.

This distinction matters because several emerging coding-agent projects appear to overlap when described as "knowledge graphs" even though they solve different problems.

## The compiler analogy

A compiler does not have one canonical representation of a program. It may maintain an AST, symbol table, control-flow graph, SSA form and machine code because each representation is useful for a different analysis or execution stage.

Engineering knowledge should be treated similarly:

```text
                    Engineering reality
             code · docs · git · runtime · humans
                              |
                              v
                    Knowledge compilation
                              |
       +----------+-----------+-----------+----------+
       |          |           |           |          |
       v          v           v           v          v
   structural  semantic   historical  runtime    intent /
     graph      index      timeline     model     workflow
       |          |           |           |          |
       +----------+-----------+-----------+----------+
                              |
                              v
                    task-specific context
                              |
                              v
                           model
```

None of these representations is "the knowledge". They are views/IRs optimized for different queries.

## Useful representations

### Structural representation

Answers: **How is the system connected?**

Typical contents:

- files, packages, modules and symbols;
- imports, calls, inheritance and configuration relationships;
- data/control flows;
- dependency and impact relationships.

A graph is a natural representation here. Deterministic parsers should do as much of this work as possible.

### Semantic representation

Answers: **What concepts are related?**

Typical mechanisms include lexical indexes, embeddings, concept labels and semantic clustering. This is useful for questions such as "find code related to idempotency" where exact symbols are unknown.

### Historical representation

Answers: **How did we get here?**

Useful artifacts include commits, PRs, incidents, migrations, rejected alternatives and architectural decisions.

### Runtime representation

Answers: **What actually happens in production?**

Examples are logs, traces, metrics, profiles, incidents and observed failure modes. Runtime evidence can contradict assumptions in static documentation.

### Intent / workflow representation

Answers: **What should happen, and in what order?**

Examples include requirements, policies, acceptance criteria, playbooks and agent workflows.

### Rationale / knowledge representation

Answers: **Why is this true or why was this decision made?**

This is where durable human-readable knowledge is especially valuable:

- invariants;
- tradeoffs;
- architectural rationale;
- patterns and anti-patterns;
- known gotchas;
- lessons learned.

A package-level `CLAUDE.md` can be a very effective canonical representation for this category because it is close to the code, reviewed in Git, and naturally scoped to the package.

## Canonical knowledge does not need a special format

EOKS should not require a graph database, ontology, or a new knowledge format to become useful.

For a personal repository or small team, a strong default is:

```text
repository/
  CLAUDE.md                 # repository mental model
  api/
    CLAUDE.md               # package/domain mental model
  auth/
    CLAUDE.md
  architecture/
    *.md                    # cross-cutting decisions when needed
```

The important distinction is between **canonical knowledge** and **derived indexes**:

```text
Canonical, human-reviewable
    |
    +-- CLAUDE.md / ADRs / design docs
    |
    v
Derived representations
    +-- graph
    +-- semantic index
    +-- symbol index
    +-- impact cache
    +-- retrieval metadata
```

The canonical files should not be forced into a machine-oriented schema merely because a downstream tool wants one.

## Pointers versus duplication

A knowledge system should usually prefer **locating and ranking evidence** over copying the whole evidence into summaries.

For example, a structural index can say:

```text
PaymentService -> payments/retry.go
Reason: direct call edge
Confidence: high
```

The agent can then read `payments/retry.go` when needed.

This avoids paying the cost of maintaining a second, potentially stale copy of the implementation.

However, some knowledge is **synthetic**: it does not exist anywhere in the source. Examples are a design tradeoff discovered during a debugging session or a cross-package invariant. Those facts deserve durable representation.

This gives EOKS two complementary goals:

1. **Navigation optimization** — find the right evidence cheaply.
2. **Knowledge optimization** — preserve important insights that otherwise exist only in people's heads or transient sessions.

## Graphs are also compiler dependencies

A graph is useful not only at retrieval time. It can identify which derived artifacts are affected by a code change.

For example:

```text
payments/retry.go changes
        |
        v
affected structural nodes
        |
        +--> impacted semantic entries
        +--> stale package context
        +--> candidate knowledge review
```

This makes incremental knowledge maintenance possible without rebuilding the world.

## Evidence and confidence

Derived relationships should carry provenance and confidence where practical. A useful distinction is:

- **EXTRACTED** — directly supported by source/tool analysis;
- **INFERRED** — derived from multiple observations;
- **AMBIGUOUS/CANDIDATE** — plausible but not yet trusted.

Confidence is not only an LLM property. It can describe the strength of an individual knowledge claim and its evidence chain.

## Design principle

> There is no single canonical representation of engineering knowledge. There are representations optimized for different questions, and EOKS should compile between them rather than forcing everything into one graph.
