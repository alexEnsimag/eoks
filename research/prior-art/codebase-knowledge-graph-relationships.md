# Codebase knowledge graphs and relationship-aware context

## Source

Sachin Kasana, [How We Turned a 500K-Line Codebase Into an AI Knowledge Graph](https://sachinkasana.medium.com/how-we-turned-a-500k-line-codebase-into-an-ai-knowledge-graph-0f6e69fb11e6), June 10, 2026.

The article is practitioner evidence from a large TypeScript-heavy codebase. It describes replacing a document-oriented retrieval view with an explicit representation of entities and relationships to support architectural questions, impact analysis and dependency discovery. The article is useful evidence about a workload and mechanism, but it is not independent experimental proof of a general knowledge-graph advantage.

## What the article contributes

The most useful observation is not simply that a graph can represent a codebase. It is that **relationships provide a different retrieval and reasoning primitive from similarity**.

A question such as:

> What services are affected if I change this payment workflow?

is not naturally a nearest-neighbor question. It requires identifying relevant entities and traversing relationships such as calls, imports, event publication/consumption and dependencies. The article explicitly frames impact analysis in this relationship-oriented way.

This gives a useful distinction:

```text
similarity retrieval
    "what looks related?"

relationship traversal
    "what is connected, affected, dependent or reachable?"
```

Both can be useful for context construction, and neither subsumes the other.

## Formal prior art: Code Property Graphs

This observation has a stronger foundation in program-analysis research than the article alone provides. The Code Property Graph (CPG) is a directed, edge-labeled, attributed graph representation of program code. It can combine representations such as syntax, control flow and data flow and make their relationships queryable through graph traversal.

See:

- [Joern Code Property Graph documentation](https://docs.joern.io/code-property-graph/)
- [Code Property Graph specification](https://cpg.joern.io/)
- Yamaguchi et al., *Modeling and Discovering Vulnerabilities with Code Property Graphs* (2014).

The important EOKS lesson is not to adopt CPG as an EOKS schema. It is that **explicit relationship structure can be a reusable intermediate representation optimized for particular questions**.

## Relationship-aware context construction

The useful architecture is therefore better expressed as:

```text
authoritative artifacts
  |
  +-- code / docs / specs / tests / configuration / decisions
  |
  v
analysis / extraction
  |
  +-- entities
  +-- relationships
  +-- semantic indexes
  +-- other derived representations
  |
  v
task-specific context construction
  |
  +-- similarity retrieval
  +-- relationship traversal
  +-- reachability / impact analysis
  +-- filtering / ranking / budgeting
  |
  v
authoritative evidence retrieval
  |
  v
synthesis
  |
  v
validation
```

The graph is therefore not necessarily the context and does not replace the underlying artifacts. It can help determine **which evidence should be inspected and why**.

This is compatible with GraphRAG-style research, where a graph index is used to organize relationships and support retrieval/summarization. GraphRAG is a separate document-oriented approach and should not be conflated with code property graphs or repository dependency graphs. Its useful general lesson is that explicit relational structure can support questions for which flat similarity retrieval is insufficient.

## Relationships are derived claims

EOKS should distinguish an authoritative artifact from a relationship representation derived from that artifact.

For example:

```text
payments/retry.go
       |
       | static analysis
       v
PaymentService --CALLS--> RetryPolicy
```

The `CALLS` edge is useful evidence, but it is not itself the authoritative implementation. It should therefore be possible to trace a derived relationship back to its supporting source and analysis where practical.

Different relationship claims may have different derivation and trust characteristics:

```text
EXTRACTED
  directly supported by deterministic/static analysis

INFERRED
  derived from multiple observations or semantic analysis

CANDIDATE / AMBIGUOUS
  plausible but not yet trusted
```

This reinforces the existing EOKS treatment of provenance, freshness and confidence for derived representations.

## Relationships across heterogeneous artifacts

The strongest EOKS generalization is broader than code structure. Relationships can connect heterogeneous engineering artifacts:

```text
Service
  |-- IMPLEMENTS --> Specification
  |-- VALIDATED_BY --> Test
  |-- DOCUMENTED_BY --> ADR / documentation
  |-- PUBLISHES --> Event
  |-- CONSUMED_BY --> Service
  |-- OBSERVED_BY --> Runtime evidence
  |-- CHANGED_BY --> Commit / PR
```

The resulting structure can support different workloads:

- dependency and impact analysis;
- evidence discovery;
- architectural navigation;
- change review;
- runtime/static contradiction discovery;
- context compilation;
- incremental invalidation of derived representations.

This should remain workload-dependent rather than becoming a mandatory EOKS ontology.

## Relationship traversal is itself evaluable

This prior art suggests a concrete evaluation family for EOKS. Instead of asking only whether an agent's final answer is correct, some workloads can evaluate the intermediate relationship operation:

```text
Given X:
  identify relevant entities
  traverse the expected relationship types
  retrieve supporting evidence
  synthesize an answer
```

Potential signals include:

- relationship precision: how many traversed relationships are actually relevant;
- relationship recall: how many required dependencies were found;
- path correctness: whether the discovered relationship chain matches the reference;
- evidence coverage: whether claims can be traced to authoritative artifacts;
- context efficiency: useful evidence versus irrelevant traversal/context;
- downstream task outcome: whether relationship-aware context improves the complete workload.

These are **evaluation hypotheses**, not proposed universal EOKS metrics.

## Architectural conclusion

This article does **not** justify a new EOKS primitive called `KnowledgeGraph` or `RelationshipGraph`.

It strengthens an existing abstraction:

> **EOKS should be able to represent and exploit meaningful relationships when a workload benefits from relationship-shaped evidence.**

Graphs remain derived representations. The authoritative state remains in the underlying artifacts. Context construction can combine relationship traversal with similarity and other acquisition mechanisms. Synthesis still needs to inspect and reason over authoritative evidence, and validation remains necessary because a correct-looking graph path does not by itself establish semantic correctness.

This is consistent with the current EOKS synthesis:

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

The new emphasis is that **relationships are a first-class capability of derived representations**, and relationship-aware retrieval can be evaluated as part of context construction without turning a graph into canonical knowledge.

## Research implications

1. Compare similarity-only context construction with hybrid similarity + relationship traversal on representative coding workloads.
2. Measure impact-analysis and dependency-discovery tasks separately from semantic retrieval tasks.
3. Test whether relationship provenance and freshness improve trust and correction cost.
4. Measure whether reusable relationship representations reduce repeated repository reconstruction without increasing stale-context failures.
5. Evaluate relationship extraction quality separately from downstream synthesis quality.

The key empirical question is therefore not *"does a knowledge graph make agents better?"* but:

> **For which workloads, which relationship representations, and which traversal strategies produce better validated outcomes than alternative context-construction mechanisms under the same evidence, cost and latency constraints?**
