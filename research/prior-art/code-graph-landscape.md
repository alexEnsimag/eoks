# Code graph and repository-context tooling landscape

Recent tools such as **CodeGraph, GitNexus and Graphify** look similar because they all help an AI agent understand a repository through structured relationships rather than repeated grep/read exploration. They should nevertheless be treated as **different implementations and layers of a broader structural-evidence stack**, not as competing definitions of EOKS.

## The common problem

A coding agent needs more than source-text retrieval for questions such as:

- What calls this function?
- What depends on this interface?
- What execution flows cross this component?
- What is the likely blast radius of this change?
- Which subsystem does this symbol belong to?
- What evidence should I inspect before editing this code?

A graph is a natural representation for these relational questions. But the graph itself is only one representation of engineering reality.

```text
authoritative engineering reality
 code · docs · ADRs · tests · git · runtime
                  |
                  v
        representation builders
                  |
       +----------+----------+
       |                     |
 structural graph       other representations
 symbols/calls/imports   semantic/history/runtime/intent
       |
       v
 graph analysis / query
 paths · impact · flows · clusters
       |
       v
 evidence delivery
 MCP · CLI · skills · hooks · reports
       |
       v
 EOKS context compilation
 loadout · relevance · budget · provenance
       |
       v
      agent
```

This explains why several products can all claim to provide "context" while operating at different layers.

## CodeGraph

The current `codegraph-ai/CodeGraph` project builds a cross-language semantic graph of functions, classes, imports and call chains using Tree-sitter. It exposes a substantial MCP interface, CLI and VS Code integration, and also has a persistent-memory capability.

**EOKS placement:** primarily **structural representation + evidence/query provider**.

The important distinction is that the MCP interface is delivery of graph-derived evidence; it does not make MCP itself a knowledge representation. Likewise, a persistent memory feature should remain conceptually separate from the structural graph.

## GitNexus

GitNexus starts with the same graph foundation but moves more computation into the indexing and query layer. Its current pipeline includes symbol resolution, community detection, process/execution-flow detection, impact analysis, confidence scoring and hybrid search. Its agent interface exposes higher-level operations such as context, impact and trace, and it can install skills/hooks that steer coding agents toward graph-aware exploration.

**EOKS placement:** **structural evidence service + precomputed analysis + agent-facing delivery**.

This makes GitNexus closer to an evidence service than a bare graph store: some of the expensive reasoning has already been compiled into task-oriented views before the model asks for them.

## Graphify

Graphify is another graph-first structural evidence system. Its code graph is built locally with Tree-sitter and is exposed through a CLI/skill/MCP workflow. Its current scope is broader than source-code relationships: project documents and other artifacts can participate in the knowledge graph. A particularly useful design choice for EOKS is explicit edge provenance such as **EXTRACTED**, **INFERRED** and **AMBIGUOUS**.

**EOKS placement:** **structural/derived knowledge representation + context navigation/evidence**.

Graphify is therefore especially interesting as a bridge between a deterministic repository map and a broader project knowledge graph, while still keeping the graph a derived representation rather than the canonical source of truth.


## Code Review Graph (CRG)

**Code Review Graph (CRG)** is a local-first structural code-intelligence system focused on turning a repository graph into compact, change-aware context for AI coding tools. It parses repositories with Tree-sitter, maintains the graph incrementally, and exposes graph/search/review capabilities through CLI and MCP. Its current integration also installs skills and hooks for supported coding agents.

The important distinction from a generic code graph is its emphasis on **change-aware context compilation**. Given a diff, CRG can identify affected functions and dependencies, traverse a bounded blast radius, surface related tests and risk signals, and return a compact review context rather than requiring the agent to rediscover the repository structure through repeated file reads.

CRG also makes context efficiency an explicit observable: review and impact responses carry estimated context-savings metadata, with an optional tokenizer verification path. Its public evaluation reports a median roughly 63x reduction in graph-query tokens versus a whole-corpus baseline across six repositories, while explicitly noting that the whole-corpus baseline is an upper bound and that small changes can have graph overhead. Its impact benchmark reports 0.69 average F1 against graph-derived ground truth; CRG correctly treats the resulting recall as a circular upper bound rather than independent evidence of impact accuracy. These limitations make the benchmark useful evidence about the mechanism, not a general end-to-end agent-quality claim.

A particularly useful implementation detail for EOKS is CRG's explicit build/update state and incremental synchronization. A partial or interrupted update should not silently appear fully fresh, and the graph can be updated from changed files rather than rebuilt on every interaction. This connects **knowledge freshness and state** directly to context compilation.

**EOKS placement:** **structural evidence + change/impact analysis + context compilation**.

CRG therefore sits between the structural graph providers above and the context-delivery layer below:

```text
repository / git diff
        |
        v
incremental structural graph
        |
        +--> search / flows / communities
        +--> impact / blast radius
        +--> tests / risk signals
        |
        v
compact task-specific context
        |
        v
      agent
```

The broader EOKS lesson is not that CRG should become an EOKS component. It is evidence that **Knowledge + State + Intent -> Context** is a useful abstraction: the graph provides structural knowledge, repository/change state determines what is current and affected, and the workload determines which evidence should be compiled for the agent.

CRG also complements rather than replaces Graphify or GitNexus. Graphify emphasizes repository/project representation and provenance; GitNexus provides broader graph-powered search, process tracing and agent navigation; CRG concentrates more strongly on incremental change analysis and bounded review context. These boundaries overlap in implementation, but they are useful distinct evaluation targets for EOKS.


## CodeSight

CodeSight belongs nearby but is not simply another graph implementation. Its emphasis is deterministic repository context and targeted evidence views: structure can be compiled into persistent, agent-readable maps, while knowledge sources can be indexed into compact views.

**EOKS placement:** **derived repository-context provider / context compilation input**.

This is an important complement to graph tooling. A graph answers relational questions efficiently; a context compiler decides which representation and which evidence should actually enter a model's working context.

## Understand Anything

Understand Anything spans structural graph extraction with semantic and LLM-derived repository understanding. It provides architectural/domain views, semantic search, explanations and impact-oriented views.

**EOKS placement:** **structural + semantic evidence provider**.

It is useful prior art for combining deterministic structure with higher-level interpretation, but EOKS should preserve provenance and distinguish extracted facts from model-derived claims.

## OpenWiki and OKF

These should not be collapsed into the graph layer.

- **OpenWiki** is primarily a generated/versioned knowledge-discovery representation. It can consume graph-derived facts, but it is not itself the structural graph substrate.
- **OKF** is a candidate durable/canonical knowledge representation. It should not become synonymous with a repository graph.

This preserves the EOKS distinction between canonical knowledge and derived representations.

## Graph tooling versus analyzers

Graph systems also should not be conflated with deeper static/dataflow analysis.

| Question | Best conceptual layer |
|---|---|
| What imports/calls/inherits from this symbol? | Structural graph |
| What execution path connects A to B? | Graph/process analysis |
| What is the blast radius of this change? | Graph impact analysis |
| What subsystem/community does this belong to? | Graph clustering/community analysis |
| Is this value able to reach a sensitive sink without a required transformation? | Dataflow analysis (e.g. CodeQL/Semgrep where appropriate) |
| Does the implementation satisfy an architectural invariant? | Invariant/policy + deterministic validator |
| Which of these facts should the agent see for this task? | EOKS context compilation |

A graph can establish that two nodes are connected without proving a path-sensitive property. This is the same **barrier principle** already documented in EOKS: when correctness depends on a transformation, validation or state-establishing barrier, explicit dataflow evidence may be required.

## Normalized comparison

| Tool | Primary layer | What is distinctive | EOKS role |
|---|---|---|---|
| Graphify | Structural representation + navigation | Local graph, cross-artifact scope, explicit edge provenance | Structural evidence provider |
| CodeGraph | Structural representation + query | Cross-language graph, large MCP surface, persistent-memory capability | Structural evidence provider |
| GitNexus | Structural evidence + analysis | Resolved graph, process tracing, impact, clustering, confidence, hybrid search | Precomputed evidence service |
| CodeSight | Repository context compilation | Targeted persistent context/evidence views | Derived context provider |
| Understand Anything | Structural + semantic | Graph plus LLM/domain interpretation and semantic views | Structural/semantic evidence provider |
| CodeQL | Semantic/dataflow analysis | Deep queryable program analysis | High-cost deterministic evidence provider |
| Semgrep | Targeted static analysis | Lightweight rules and selected dataflow | Deterministic evidence provider |
| OpenWiki | Knowledge/discovery representation | Generated versioned repository knowledge | Derived knowledge representation |
| OKF | Canonical knowledge representation | Durable structured knowledge model | Canonical knowledge resource |
| GrapeRoot | Context delivery/optimization | Proactive context optimization around an agent | Context compiler/delivery prior art |

## The EOKS boundary

The key conclusion is that **EOKS should not become a better GitNexus/Graphify/CodeGraph**. Those tools are valuable precisely because they can specialize in building and querying one representation.

EOKS owns the higher-level control problem:

```text
workload
   |
   +--> what question/evidence is required?
   |
   +--> which representation can answer it?
   |
   +--> which provider is reliable and cheap enough?
   |
   +--> what scope/revision/access constraints apply?
   |
   +--> how should evidence be compiled into context?
   |
   +--> what evidence was actually used?
   |
   +--> did it improve the outcome?
```

This strengthens the existing **minimum sufficient evidence** principle. The control plane should not invoke every analyzer merely because it is available. It should select the cheapest reliable representation/provider that is sufficient for the workload, while escalating when evidence is incomplete, conflicting or high-risk.

## Why this matters for the EOKS model

The landscape supports the existing compiler analogy:

> **There is no single canonical representation of engineering knowledge.**

A structural graph, semantic index, repository wiki, historical timeline, runtime model and canonical ADR are analogous to different compiler IRs: each is optimized for different questions. The value of EOKS is in **coordinating and compiling between them**, not forcing them into one graph.

The practical composition is therefore:

```text
resource/loadout
      |
      +--> graph provider -------- structural evidence
      +--> semantic provider ----- concepts/rationale
      +--> analyzer ------------- dataflow/invariants
      +--> wiki/knowledge -------- durable project knowledge
      +--> runtime --------------- observed behavior
      |
      v
context compiler
      |
      v
agent workflow
      |
      v
evaluation / outcome
```

This also clarifies why **Graphify, CodeGraph and GitNexus can all be useful simultaneously**. They overlap in the structural graph substrate, but differ in graph construction/resolution, precomputed analysis, provenance, query abstractions and agent integration. EOKS should treat them as alternative or complementary **providers**, not as competing EOKS layers.

## Broader 2026 tool landscape

The expanded [agent codebase-context and repository-intelligence landscape](agent-codebase-context-tool-landscape-2026-09.md) documents the wider family of context packers, repo maps, LSP tools, code graphs, project-memory systems, incremental indexes, compression layers and agent hooks. It also captures the September 2026 LLMDevs benchmark and its methodological implications.

That broader landscape reinforces the boundary described here: structural graph providers are evidence resources, while EOKS coordinates provider selection, context compilation, provenance, freshness and evaluation.

## Research questions

- Should EOKS standardize a provider contract rather than a graph schema?
- What minimum provenance must every graph-derived claim carry: repository revision, extractor version, source locations, confidence and derivation method?
- When should raw graph primitives be exposed to an agent versus a task-oriented evidence view such as impact or trace?
- How should EOKS detect stale graph indexes and decide whether incremental re-indexing is sufficient?
- Can provider selection be learned from historical task outcomes: e.g. when is a structural graph enough, and when is deeper dataflow analysis worth its cost?
