# Structured knowledge, ontologies and context graphs

This note records a research pass prompted by recent practitioner writing about building knowledge graphs for AI applications. The goal is not to add another EOKS primitive, but to clarify the abstraction boundary between ontology, knowledge representation, contextualization and access.

## Research conclusion

The durable concept is **structured, contextualized knowledge that can be selectively accessed, validated, transformed and reused**. A graph is one representation that is particularly useful when relationships, traversal or impact matter. It is not the knowledge itself and should not become an EOKS primitive.

The same applies to the emerging term **context graph**. Current usage varies: some systems mean a knowledge graph enriched with temporal validity and provenance; others mean agent memory, workload-specific context, or an organization-specific relationship layer. The terminology is useful as prior art but is not stable enough to define EOKS's core model.

## Ontology versus representation

Ontology is the more formal boundary. An ontology specifies concepts, meanings and allowed relationships or rules. A knowledge representation instantiates or organizes information using some structure. The representation may be a graph, document, table, index, timeline, database, or another derived artifact.

A useful distinction is therefore:

```text
ontology / schema
    -> defines meaning and structure

knowledge / evidence
    -> provides instances, claims and observations

representation
    -> organizes those materials for particular uses

access / computation
    -> retrieves, traverses, validates, analyzes or reasons over them
```

EOKS should preserve these distinctions without requiring a universal ontology or graph storage technology.

## Why context graphs are not a primitive

Recent systems use "context graph" to describe materially different things:

- knowledge graphs augmented with provenance, temporal validity or other contextual dimensions;
- graphs of agent memory, decisions, sources and reasoning history;
- organization-specific knowledge used to ground a workload;
- workload/resource graphs describing what evidence is eligible or useful for a particular task.

The common property is not the graph topology. It is that **relationships and contextual properties are made explicit so they can influence access or reasoning**.

That is already expressible by EOKS's existing distinction between resources, representations, context and control. Introducing "context graph" as a first-class primitive would risk freezing terminology that is still evolving.

## Evidence from recent agent-memory research

### Ontology-Grounded Project Memory for Coding Agents (MOOSEDev)

MOOSEDev models architectural decisions, lessons, constraints and rationales as typed records in a knowledge graph exposed to coding agents. Records carry lifecycle status, provenance and supersession links. In its evaluation, the structured system was substantially stronger than a vector-memory baseline on supersession, set-completeness and negation questions, while relevance recall and token cost were broadly similar.

The useful EOKS lesson is not that graphs always outperform vector retrieval. It is that **explicit structure makes some questions representable and answerable**: current versus superseded knowledge, complete sets, and relationships among records. Semantic relevance retrieval is a different capability.

### SodaMem: Evidence-Grounded Temporal Graph Memory

SodaMem models typed fact events with provenance and temporal validity, including relationships such as supersedes, contradicts and updates. Its design reinforces the value of making currency, provenance and temporal relationships explicit when the workload depends on them.

### MemoryLACE

MemoryLACE is particularly useful for EOKS's abstraction boundary. It reports that explicit local lifecycle relationships—merge, supersession and contradiction—can improve long-term memory reasoning without requiring a comprehensive global knowledge graph. This is evidence against treating a full graph as the necessary architecture for lifecycle-aware knowledge.

### Agent Zero Memory

Agent Zero Memory keeps multiple parallel representations of the same history: an episodic timeline, an entity-event graph and a hierarchical documentary memory. This is strong evidence for the EOKS position that **one canonical representation is often insufficient** and that different representations can be selected for different questions.

## Structure is not the same as graph topology

Recent work on graph-based reasoning also cautions against attributing all benefits to graph structure itself. Selective, agentic access to a structured information space can be valuable even when the graph topology is not the primary causal factor.

The EOKS abstraction should therefore be:

> **structured, selective access to relevant evidence**

rather than:

> graph traversal is the preferred reasoning mechanism.

This distinction protects EOKS from making an implementation choice into a conceptual primitive.

## Provenance, lifecycle and validity are increasingly important

Across these systems, several properties recur:

- provenance — where a claim or representation came from;
- verification/trust state — how it was checked;
- freshness — whether the underlying evidence is current;
- lifecycle — whether information is current, historical, superseded or archived;
- temporal validity — when a statement applies;
- relationships — how claims, artifacts and entities depend on one another;
- attestation/evidence — how a computed or generated result was produced.

These properties should be represented explicitly **when correctness depends on them**. They should not be collapsed prematurely into a universal confidence score or a mandatory EOKS schema.

This aligns with the emerging OKF direction, which makes provenance, trust, lifecycle and attestation first-class while deliberately keeping the format minimally opinionated. EOKS can consume such representations without making OKF mandatory.

## Revised conceptual model

The research supports the following abstraction:

```text
                  authoritative sources / observations
                                  |
                                  v
                         knowledge / evidence
                                  |
                     +------------+------------+
                     |                         |
                     v                         v
              semantic structure       contextual properties
             ontology / schema       provenance / time / status
                     |                         |
                     +------------+------------+
                                  |
                                  v
                         derived representations
              graph · index · document · table · timeline
                                  |
                                  v
                       selective access / compute
             retrieval · traversal · filtering · validation
                         analysis · reasoning
                                  |
                                  v
                              synthesis
```

A representation can itself become a reusable derived artifact, provided it retains enough provenance, freshness and dependency information to be trusted for its intended use.

## Implication for EOKS synthesis

This strengthens the existing working synthesis:

> **authoritative state -> derive representation/computation -> validate -> publish -> reuse**

"Representation" should be understood broadly. It can encode relationships, provenance, temporal state, decisions, constraints or other properties needed by downstream computation.

"Reuse" does not necessarily mean putting the representation into a prompt. It can mean selectively exposing the right evidence or representation to a subsequent computation.

This also reinforces the distinction between knowledge and context:

```text
broad / durable knowledge and evidence
              |
              v
      eligible resources
              |
              v
     workload-specific selection
              |
              v
       compiled context
```

Context is therefore a task-specific view over available evidence, not a replacement for the knowledge layer.

## EOKS boundary

EOKS should **not** add any of the following as mandatory primitives:

- Knowledge Graph;
- Context Graph;
- ontology;
- graph database;
- RDF/OWL;
- GraphRAG;
- a particular memory topology.

Instead, EOKS should provide the coordination semantics that allow these things to participate as resources or representations when useful:

1. identify authoritative sources and evidence;
2. preserve provenance, scope, freshness and lifecycle where required;
3. select an appropriate representation for the workload;
4. compile task-specific context from eligible resources;
5. validate the resulting evidence/context against workload requirements;
6. evaluate outcomes and feed evidence back into future selection or maintenance.

## References

- James Adam, **Ontology-Grounded Project Memory for Coding Agents (MOOSEDev)**, arXiv:2608.13662 (2026).
- Fengrong Wan, Chengcan Wu, Ningtao Lyu, **SodaMem: Evidence-Grounded Temporal Graph Memory for LLM Agents**, arXiv:2608.08055 (2026).
- Meriem Yacoubi et al., **MemoryLACE: Memory Lifecycle-Aware Consolidation and Evidence Retrieval**, arXiv:2609.03201 (2026).
- Ming Wu, Pengyuan Zhu, **Agent Zero Memory: Provenance-Aware Long-Term Memory for LLM Agents**, arXiv:2608.29606 (2026).
- Google Cloud Platform, **Open Knowledge Format (OKF) specification**, current repository specification.
- W3C, **OWL / Web Ontology Language** and RDF specifications, for the formal distinction between ontology and graph-based knowledge representation.

These references are evidence for the synthesis above, not dependencies or proposed EOKS components.
