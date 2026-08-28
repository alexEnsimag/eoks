# Context engineering

Context engineering is the discipline of constructing the information available to a model for a specific reasoning step.

## Context is not storage

A repository, knowledge base, graph or memory store can contain far more information than should enter a model's context. The important operation is the **selection and transformation boundary** between external knowledge and model input.

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
- interaction with the chosen model.

The goal is not maximum information. It is maximum useful evidence per unit of context and reasoning cost.

## Context splitting

Different reasoning steps often need different context. Splitting context can reduce noise and make decisions inspectable: discovery context, implementation context, verification context and historical/project context can be maintained separately.

This connects to the idea of **context entropy**: a large heterogeneous context may contain substantial information while having poor signal-to-noise for the current task.

## Progressive disclosure

The system should prefer exposing the minimum sufficient information and retrieving additional detail when evidence shows it is needed. This resembles filesystem/document navigation more than stuffing an entire corpus into a prompt.

## Structured representations

Files, Markdown, JSON/YAML, relational data, embeddings and graphs are representations, not ends in themselves. A structure is valuable when it improves retrieval, reasoning, validation or maintenance.

## Open problem

We need empirical benchmarks showing when a context intervention improves task success, rather than assuming that more structure or more retrieved tokens are beneficial.
