# Knowledge, memory and context graphs — synthesis note

> Research/prior-art note prompted by Sergey Vasiliev's 2026 discussion of knowledge graphs, memory graphs and context graphs. This note does not introduce new EOKS runtime primitives. It uses the distinction to sharpen existing EOKS boundaries and the synthesis layer.

## Why this matters

The useful contribution is not the proposal to create three graph platforms. Vasiliev explicitly warns that knowledge graphs, memory graphs and context graphs can become three products, schemas and governance programs before the separation is justified. The stronger architectural interpretation is that they are **three roles over related information**, with different authority, lifecycle and purpose.

For EOKS, this reinforces an existing direction:

```text
knowledge + experience + live state + policy
                    |
                    v
            context compilation
                    |
                    v
              context artifact
                    |
                    v
                  run
                    |
                    v
                 outcome
                    |
                    v
                evaluation
                    |
                    v
               experience
```

The graphs are optional representations underneath this loop. The control problem is deciding what information should participate in a particular workload, why, under which policy, and with what evidence.

## 1. Three roles, not three EOKS subsystems

A useful mapping is:

| External role | Working meaning | EOKS interpretation |
|---|---|---|
| Knowledge graph | governed/canonical meaning, entities, relationships and accepted assertions | durable knowledge representations and authoritative sources |
| Memory graph | retained observations, attempts, decisions, outcomes and feedback | experience-derived memory and run/learning history |
| Context graph | task-specific view assembled for a decision | compiled working set/context artifact |

The important boundaries are **authority, lifecycle and purpose**, not whether the data happens to be stored in a graph.

This is consistent with the existing EOKS rule that a graph is a representation rather than knowledge itself. A repository graph, knowledge graph, memory graph, index, timeline, Markdown document or structured record can all be evidence/resources with different validity and provenance semantics.

## 2. Context should be treated as a compiled projection

The strongest architectural implication is to sharpen the definition of context:

> **Context is the task-specific projection of eligible knowledge, experience, live state, evidence and policy for a reasoning or execution step.**

Therefore context is neither the permanent knowledge store nor merely the final prompt string.

A context compiler should be able to expose:

- what was eligible;
- what was selected;
- what was omitted;
- why an item was selected;
- source/provenance and revision;
- freshness/validity;
- evidence strength and conflicts;
- cost/budget;
- the materialized representation supplied to the model or tool.

This directly supports the existing Context Workbench direction: context can be inspected, diffed, evaluated and reconstructed instead of being an opaque concatenation of text.

## 3. Knowledge and experience should remain distinct

The graph terminology makes a useful distinction explicit:

```text
knowledge  = what the system currently accepts as reusable meaning
experience = what happened when a workload was attempted
memory     = durable representation of useful experience
```

Experience should not silently rewrite canonical knowledge.

A safer lifecycle is:

```text
observation
    -> candidate claim / learning
    -> evidence and validation
    -> scoped promotion
    -> reusable memory/procedure/knowledge
    -> later evaluation
    -> update or invalidate
```

This matches EOKS's existing candidate-extraction and controlled-promotion model and gives reflection a concrete architectural destination: reflection can produce candidate experience or learning records; promotion remains governed.

## 4. Negative experience is valuable memory

Memory should preserve more than successful facts or procedures. Rejected alternatives, failed approaches, known limitations and corrections can prevent repeated work.

For software engineering, examples include:

```text
attempted approach
      -> failed verification
      -> reason / evidence
      -> retained rejection
      -> future context when the same decision recurs
```

This is not canonical truth about the repository. It is experience about a prior attempt. Its scope, freshness and evidence therefore matter.

The distinction prevents an agent from treating "we tried this once" as an immutable architectural fact while still allowing the system to avoid repeating expensive mistakes.

## 5. Decision provenance and context snapshots

A particularly useful extension is to treat the assembled context itself as an inspectable artifact.

For a consequential run, EOKS should eventually be able to reconstruct:

```text
Task
  -> eligible resources/loadout
  -> selection/ranking decisions
  -> context artifact
  -> Run
  -> tool/model decisions
  -> evidence
  -> Outcome
  -> Evaluation
```

The context artifact should preserve enough temporal provenance to answer:

> **What did the controller know, from which representations and source revisions, when it made this decision?**

This connects context engineering with EOKS evaluation and reproducibility without introducing a new `ContextGraph` primitive. A graph is one possible representation of the artifact; a structured trace or block model can serve the same role.

## 6. A sharper EOKS control loop

The synthesis can therefore be expressed as:

```text
                  TASK / INTENT
                       |
                     POLICY
                       |
                       v
              eligible resources
                       |
          knowledge + experience
          + live state + evidence
                       |
                       v
              CONTEXT COMPILER
                       |
                       v
              CONTEXT ARTIFACT
              + provenance/rationale
                       |
                       v
                      RUN
                       |
                 actions/tools
                       |
                       v
                    OUTCOME
                       |
                       v
                  EVALUATION
                       |
          +------------+------------+
          |                         |
       sufficient              insufficient
          |                         |
        stop             retrieve / verify / retry /
                         branch / escalate
          |
          +-------------------------+
                                    |
                              EXPERIENCE
                                    |
                              validate/promote
                                    |
                          future knowledge/memory
```

This is an extension of the existing EOKS loop, not a competing architecture.

## 7. What this changes in the synthesis

This research strengthens four existing EOKS statements:

1. **Context is compiled, not stored as permanent knowledge.**
2. **Memory is multi-form and should preserve experience without collapsing it into canonical knowledge.**
3. **Decision/context provenance is part of reliable control, not merely observability metadata.**
4. **Graphs are representations selected when their relational structure provides sufficient value; they are not mandatory EOKS primitives.**

It also gives the synthesis a useful four-part vocabulary:

```text
Knowledge   — governed/reusable meaning
Experience  — what happened
Context     — what is selected now
Evaluation  — whether the loop worked
```

Everything else should remain mapped to these through the existing resource, run, decision, policy and outcome model until experiments demonstrate a need for more primitives.

## 8. Relation to prior EOKS research

This note should be read together with:

- [`knowledge-context-control-plane.md`](../knowledge-context-control-plane.md) — existing knowledge/context/control-plane synthesis;
- [`context-workbench.md`](../context-workbench.md) — inspectable context construction;
- [`memory-and-knowledge.md`](../memory-and-knowledge.md) — memory/knowledge lifecycle;
- [`session-learning.md`](../session-learning.md) — learning from development sessions;
- [`prior-art/agent-memory.md`](agent-memory.md) and [`agent-memory-lifecycle-2026.md`](agent-memory-lifecycle-2026.md) — memory prior art and lifecycle synthesis;
- [`docs/conceptual-synthesis.md`](../../docs/conceptual-synthesis.md) — cross-cutting conceptual synthesis.

## 9. Open questions

- What minimum provenance is required to reproduce a context decision?
- Which context-selection signals predict downstream task success rather than merely retrieval relevance?
- How should conflicts between canonical knowledge and experience-derived memory be surfaced?
- When does a graph representation materially outperform simpler indexed/structured representations for a coding workload?
- How should negative experience decay or become invalid when the underlying repository or architecture changes?
- Can context artifacts become a stable evaluation unit across different agents and models?

These should remain experiments/research questions rather than immediate ontology changes.

## Sources

- Sergey Vasiliev, “Knowledge Graph, Memory Graph and Context Graph: Three Roles in One Architecture” (2026): https://sergeyvasiliev.substack.com/p/knowledge-graph-memory-graph-and
- Sergey Vasiliev, “The Applied Knowledge Graph as the Runtime Layer for Agentic AI” (2026): https://sergeyvasiliev.substack.com/p/the-applied-knowledge-graph-as-the
- Sergey Vasiliev, “From LPG to GraphRAG: The Minimum Semantic Contract” (2026): https://sergeyvasiliev.substack.com/p/from-lpg-to-graphrag-the-minimum
