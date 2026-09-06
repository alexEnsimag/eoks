# Engineering knowledge as a multi-representation system

A central conclusion from EOKS research is that **knowledge is not a graph**. A graph is one representation of knowledge, optimized for questions about relationships and structure.

This distinction matters because emerging coding-agent projects can appear to overlap when described as "knowledge graphs" even though they solve different problems.

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

## Two different roles for graphs

The word **graph** is overloaded in AI-agent architecture. EOKS should distinguish at least two materially different graph roles:

1. **System-understanding graphs** describe the thing being reasoned about: code symbols, dependencies, calls, data/process relationships, architecture, runtime relationships, etc.
2. **Context/coordination graphs** describe the relationship between a workload and the resources/evidence available to reason about it: what is relevant, eligible, selected, dependent, or useful for a particular context.

The first is primarily a **representation of the world/system**. The second is primarily a **representation of the reasoning environment**.

GrapeRoot's dual-graph approach is useful prior art for the second category. It should therefore not be grouped with Graphify, CodeGraph and GitNexus merely because all of them use graph structures.

A useful conceptual model is:

```text
             SYSTEM / WORLD GRAPH
       code · architecture · runtime
                    |
                    | evidence
                    v
             knowledge resources
                    |
                    v
          CONTEXT / COORDINATION GRAPH
       workload · resources · evidence
       relevance · selection · dependencies
                    |
                    v
             context compilation
                    |
                    v
                   agent
```

The two graphs may be related, but they answer different questions. A system graph asks *what is connected to what?* A context graph asks *what should be connected to this workload's reasoning process?*

**EOKS should not require either graph to be the canonical representation.** The useful abstraction is the ability to represent, query and govern relationships when a workload benefits from them.

## Useful representations

### Structural representation

Answers: **How is the system connected?**

Typical contents:

- files, packages, modules and symbols;
- imports, calls, inheritance and configuration relationships;
- data/control flows;
- dependency and impact relationships;
- routes, schemas, events and other framework-level contracts.

A graph is a natural representation here. Deterministic parsers should do as much of this work as possible.

### The structural graph family

Several recent coding-agent tools occupy this same family. They should be grouped together in EOKS rather than treated as unrelated concepts:

| Tool | Primary contribution | What distinguishes it |
|---|---|---|
| **Graphify** | Local structural graph and navigation | Tree-sitter-based graph, cross-artifact scope, explicit edge provenance such as extracted/inferred/ambiguous |
| **CodeGraph** | Cross-language structural graph and query | Symbol/dependency/call relationships with MCP/CLI/editor integration |
| **GitNexus** | Structural graph plus compiled analysis | Resolution, process/flow tracing, impact analysis, clustering, confidence and hybrid search |
| **Understand Anything** | Structural graph plus semantic interpretation | Architectural/domain views, semantic search, explanations and impact-oriented views |

These tools overlap substantially in the graph substrate, but not necessarily in the layer above it. A useful normalization is:

```text
                 structural representation
                          |
        +-----------------+------------------+
        |                 |                  |
    Graphify          CodeGraph          GitNexus
        |                 |                  |
        +-----------------+------------------+
                          |
                  graph analysis/query
                  paths · impact · flows
                          |
                          v
                   evidence provider
                          |
                          v
                 context compilation
```

The arrows describe capability relationships, not software dependencies. A tool may implement several layers itself.

**EOKS should not standardize on any one graph implementation.** A graph is a representation selected when the workload needs graph-shaped evidence. The EOKS abstraction is the provider/evidence contract and the policy that decides whether graph evidence is sufficient, unnecessary or should be supplemented by another representation.

This also clarifies the role of **CodeSight**: it is closer to repository-context compilation and targeted evidence views, and can consume structural information from graph-like providers. It should therefore not be presented as simply another competing graph implementation.

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

### Rationale / canonical knowledge representation

Answers: **Why is this true or why was this decision made?**

This is where durable human-readable knowledge is especially valuable:

- invariants;
- tradeoffs;
- architectural rationale;
- patterns and anti-patterns;
- known gotchas;
- lessons learned.

A package-level `CLAUDE.md` can be a very effective canonical representation for this category because it is close to the code, reviewed in Git, and naturally scoped to the package.

## Derived context maps are not canonical knowledge

Tools such as CodeSight demonstrate an important intermediate layer between source material and the final model context:

```text
authoritative sources
  |  code / ADRs / docs / tests
  v
analysis / indexing / classification
  |
  +--> structural map
  +--> wiki index
  +--> topic article
  +--> knowledge index
  |
  v
context compiler
  |
  v
model
```

These generated maps are **derived representations**. Their purpose is to make authoritative evidence cheap to navigate and retrieve. They should carry source revision, provenance and freshness information and should not silently replace the canonical sources they summarize.

CodeSight's code wiki and Markdown knowledge mode are a concrete example: code structure can be compiled into topic-oriented articles, while ADRs, meeting notes, retrospectives, specs and research can be indexed into a compact knowledge map. This is useful prior art, but classification and summaries remain derived evidence unless separately reviewed and promoted.

## Assets are a lifecycle abstraction, not a representation

EOKS uses **Asset** as a generic governance/lifecycle abstraction for reusable resources. It deliberately does not imply that all resources are the same kind of knowledge.

For example:

```text
Asset
 ├── Memory              -> experience-derived information
 ├── Skill               -> reusable procedure
 ├── Wiki / document     -> structured domain/project knowledge
 ├── ADR / decision      -> reviewed rationale
 ├── CodeGraph           -> structural representation
 ├── Repository map      -> derived structural/navigation representation
 ├── Test result         -> verification evidence
 └── Incident record     -> historical/runtime evidence
```

These have different authority, provenance and update semantics. The shared abstraction exists so they can carry common lifecycle metadata—provenance, scope, freshness, revision, ownership/access, validation state and version—without being forced into one ontology.

See [Resource model](resource-model.md) for the canonical definitions of Asset, Provider, Representation, Loadout and Context. TencentDB Agent Memory is useful prior art because it explicitly governs Chat Memory, Skills, LLM-Wiki and CodeGraph as reusable resources.

## Asset universe versus loadout

The asset universe can be much larger than what a particular agent or task should use:

```text
all available assets
        |
 access / ownership / scope / applicability
        |
 agent + task loadout
        |
 relevance / value / budget
        |
 compiled context
```

The **loadout** is a workload-scoped eligibility/availability boundary. Context compilation is a separate selection and transformation step.

This matters for access control, stale-state control, task applicability, specialization, portability and reproducibility. Retrieval should not be the only gate.

## Canonical knowledge does not need a special format

EOKS should not require a graph database, ontology, or new knowledge format to become useful.

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

The important distinction is between **canonical knowledge** and **derived representations**:

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
    +-- repository wiki/map
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

## Reusable representations reduce reconstruction work

Recent comparisons of Graphify, GitNexus and CodeGraph illustrate a broader pattern: structural representations can turn repeated repository reconstruction into a reusable, queryable intermediate artifact. The practical benefit is not simply "using a graph"; it is that relationships can be computed once, retained with provenance/freshness, and queried as evidence when a workload needs them. citeturn0view0

This suggests a useful EOKS distinction:

```text
raw / authoritative artifacts
        |
        v
representation / analysis
        |
        +--> reusable intermediate artifact
        |       + provenance
        |       + freshness
        |       + confidence
        |
        v
workload-specific evidence
        |
        v
context compilation
```

The representation remains **derived**, not canonical. Its value comes from reducing repeated reconstruction while preserving a path back to authoritative evidence. Different representations may therefore coexist, and EOKS can select among them—or combine them—according to workload, evidence requirements and budget.

The article also usefully frames the progression from simple navigation toward analysis and cross-artifact evidence. EOKS should preserve that distinction rather than collapsing all graph-based systems into one capability: a representation can support navigation, analysis, impact reasoning or evidence delivery, and those capabilities have different validation requirements.

## Structured knowledge, provenance and lifecycle

Recent work on agent memory and knowledge representations reinforces this abstraction boundary. Ontology-grounded project memory systems model decisions, constraints and rationales with explicit provenance, lifecycle and supersession; temporal graph memory makes validity and update relationships explicit; other systems combine multiple representations such as timelines, entity-event graphs and hierarchical documents. These results suggest that **explicit structure is valuable when the workload needs questions about relationships, completeness, supersession, contradiction or temporal validity**, without implying that a graph is always the best representation.

The recurring properties are:

- provenance — where a claim or representation came from;
- verification/trust state — how it was checked;
- freshness — whether the evidence is current;
- lifecycle — whether information is current, historical, superseded or archived;
- temporal validity — when a statement applies;
- relationships — how artifacts or claims depend on one another.

EOKS should represent these properties explicitly when they matter to correctness, while avoiding a mandatory universal schema. The representation may be a graph, document, table, timeline, index or another derived artifact.

This also sharpens the existing synthesis:

```text
authoritative state
      |
      v
derive representation / computation
      |
      v
validate
      |
      v
publish
      |
      v
reuse
```

Here, **reuse is not synonymous with prompt injection**. A derived representation can be selectively queried, traversed, filtered or used to compile task-specific context. The goal is to expose the right evidence to the next computation, not to maximize the amount of information visible to a model.

The emerging term **context graph** is useful prior art for this family of systems, but its meaning is not yet stable enough to become an EOKS primitive. EOKS should instead preserve the more general capability: **structured, contextualized evidence that can be selectively accessed, validated, transformed and reused**.

See [`research/prior-art/structured-knowledge-and-context-graphs.md`](../research/prior-art/structured-knowledge-and-context-graphs.md) for the supporting research and references.

## Design principle

> There is no single canonical representation of engineering knowledge. There are representations optimized for different questions, and EOKS should compile between them rather than forcing everything into one graph.
