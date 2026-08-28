# EOKS architecture

EOKS is best understood as a set of cooperating planes rather than a single agent.

```text
                         EOKS CONTROL PLANE
              scheduling · policies · resource selection
                                  |
        +-------------------------+-------------------------+
        |                         |                         |
  KNOWLEDGE PLANE          CONTEXT PLANE            EXECUTION PLANE
 canonical knowledge       selection · assembly      workflows · agents
 graphs · semantic         ranking · compression     reasoning strategies
 history · runtime         progressive disclosure    tools · artifacts
 evidence
        |                         |                         |
        +-------------------------+-------------------------+
                                  |
                         EVALUATION / FEEDBACK
                  evidence · quality · confidence · outcomes
                                  |
                           OBSERVABILITY
                decisions · cost · latency · provenance
```

## Control plane

The control plane decides **what should happen next**. It can schedule tasks, select models, evidence providers and tools, impose policies, and react to evaluation signals.

The Kubernetes analogy is useful here: a desired task state can be reconciled against observed execution. AI workloads differ because reasoning quality is probabilistic, context is mutable, and resource requirements are semantic rather than only CPU/memory based.

## Knowledge plane

The knowledge plane maintains durable project information and the representations used to navigate it. It should not assume that a single graph is the canonical form.

Possible representations include:

- hierarchical `CLAUDE.md` / Markdown as human-reviewable canonical project knowledge;
- ADRs and cross-cutting architecture documents;
- deterministic structural graphs and symbol indexes;
- semantic indexes and concept clusters;
- historical timelines and decision records;
- runtime observations;
- episodic and procedural memory;
- provenance and confidence metadata.

The key distinction is **canonical knowledge versus derived evidence providers**. Humans can maintain a concise mental model while machines maintain graphs/indexes/caches that make authoritative evidence easier to locate.

See [Engineering knowledge as a multi-representation system](knowledge-representations.md) and [Knowledge base and persistent project knowledge](knowledge-base.md).

## Context plane

The context plane constructs the information supplied to a reasoning step. It should support retrieval, ranking, compression, deduplication, conflict detection, provenance, progressive disclosure and explicit context budgets.

The context plane is not a storage system. It is a **compiler from task + workflow + available evidence into task-specific model context**.

A graph, semantic index or memory store should normally be queried by this plane rather than dumped into the model context.

A useful mental model is a **context budget**, not a context window: every item has relevance, freshness, reliability, cost and interaction effects.

## Execution plane

The execution plane runs workflows, reasoning strategies, agents and tools; obtains artifacts; modifies repositories; executes tests; and records outcomes.

Workflows answer **what should happen next**. Reasoning strategies answer **how a reasoning step should approach its problem**. These are distinct from knowledge, which answers **what the system knows**.

See [Agent workflows and reasoning strategies](agent-workflows.md).

## Evaluation and feedback

Evaluation closes the loop. Results should feed back into model selection, context construction, memory/knowledge updates and task scheduling.

Confidence should be evidence-oriented rather than only model-reported. Examples include:

- deterministic extraction versus LLM inference;
- test and static-analysis results;
- review outcomes;
- runtime observations;
- freshness and provenance of the underlying source.

## Continuous knowledge updates

Knowledge maintenance should be incremental, like a modern build system rather than a full rebuild after every change.

```text
code/doc/event change
        |
   impact detection
        |
  +-----+-----+-----+-----+
  |           |           |
 graph     semantic    knowledge
 update     update     candidate
  |           |           |
  +-----+-----+-----+-----+
        |
 context-cache invalidation
```

Cheap deterministic updates should happen frequently. LLM-heavy reasoning should be reserved for higher-signal events such as merged PRs, incidents, architecture changes or completed workflows.

Hooks are event boundaries into this lifecycle. They should not imply that the entire knowledge system is recomputed after every tool call.

## Observability

Observability should expose not only latency and token counts, but **why the system made information and execution decisions**:

- retrieved and omitted context;
- evidence providers queried;
- model/reasoning strategy selected;
- tools invoked;
- evidence considered;
- evaluation outcome;
- confidence and provenance signals;
- knowledge updates and invalidations.
