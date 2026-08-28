# EOKS architecture

EOKS is best understood as a set of cooperating planes rather than a single agent.

```text
                    EOKS CONTROL PLANE
          scheduling · policies · model/tool selection
                              |
       +----------------------+----------------------+
       |                      |                      |
  CONTEXT PLANE          MEMORY / KNOWLEDGE      EXECUTION
 retrieval · assembly    persistence · graphs    tools · agents
       |                      |                      |
       +----------------------+----------------------+
                              |
                     EVALUATION / FEEDBACK
              quality · confidence · outcomes
                              |
                       OBSERVABILITY
                  traces · costs · decisions
```

## Control plane

The control plane decides **what should happen next**. It can schedule tasks, select models and tools, impose policies, and react to evaluation signals.

The Kubernetes analogy is useful here: a desired task state can be reconciled against observed execution. But AI workloads differ because reasoning quality is probabilistic, context is mutable, and resource requirements are semantic rather than only CPU/memory based.

## Context plane

The context plane constructs the information supplied to a reasoning step. It should support retrieval, ranking, compression, deduplication, conflict detection, provenance and progressive disclosure.

A useful mental model is a **context budget**, not a context window: every item has relevance, freshness, reliability, cost and interaction effects.

## Memory and knowledge plane

Memory should distinguish at least:

- episodic/session history;
- durable facts and decisions;
- structured project knowledge;
- learned retrieval/indexing artifacts;
- external source material.

A graph can be useful when relationships, provenance and dependency structure matter. A graph is not automatically better than files or a database; representation should follow the query and reasoning task.

## Execution plane

The execution plane runs tools and agents, obtains artifacts, modifies repositories, executes tests, and records outcomes. Deterministic tools should handle deterministic questions whenever possible.

## Evaluation and feedback

Evaluation closes the loop. Results should feed back into model selection, context construction, memory updates and task scheduling. This is the key distinction between a static agent framework and a control system.

## Observability

Observability should expose not only latency and token counts, but **why the system made information and execution decisions**: retrieved context, omitted context, model selected, tools invoked, evidence considered, evaluation outcome and confidence signals.
