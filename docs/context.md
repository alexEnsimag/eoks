# Context engineering

Context engineering is the discipline of constructing the information available to a model for a specific reasoning step.

## Context is not storage

A repository, knowledge base, graph or memory store can contain far more information than should enter a model's context. The important operation is the **selection and transformation boundary** between external knowledge and model input.

A particularly important consequence is that a coding agent should not need to reconstruct the full repository context from zero at the start of every session. Durable project knowledge can move stable, high-value facts outside the model context and make them retrievable on demand.

See [Knowledge base and persistent project knowledge](knowledge-base.md) and [Behavioral memory and learning how developers work](behavioral-memory.md).

## Canonical knowledge vs memory

A useful coding-agent environment can have multiple persistence mechanisms with different trust levels:

- **Canonical project knowledge** — explicit, validated instructions, invariants, architectural constraints and durable decisions. A `CLAUDE.md`-style file is one practical representation.
- **Semantic memory** — historical material retrieved because it may be relevant to the current task.
- **Episodic memory** — records of previous experiences, including the situation, approach and outcome.
- **Procedural/behavioral memory** — generalized ways of working, candidate skills, workflows and policies learned from repeated experience.
- **Candidate knowledge** — automatically extracted observations, proposed rules, skills or decisions that have not yet been validated.

These should not be conflated. A memory retrieval system should not silently become the source of truth, and an automatic session summarizer should not silently rewrite canonical project policy.

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
- interaction with the chosen model;
- applicability of retrieved procedures to the current task.

The goal is not maximum information. It is maximum useful evidence per unit of context and reasoning cost.

## Context splitting

Different reasoning steps often need different context. Splitting context can reduce noise and make decisions inspectable: discovery context, design/planning context, implementation context, verification context and historical/project context can be maintained separately.

This connects to the idea of **context entropy**: a large heterogeneous context may contain substantial information while having poor signal-to-noise for the current task.

## Progressive disclosure

The system should prefer exposing the minimum sufficient information and retrieving additional detail when evidence shows it is needed. This resembles filesystem/document navigation more than stuffing an entire corpus into a prompt.

Persistent knowledge strengthens progressive disclosure: stable project facts can be retrieved as compact summaries first, with links back to source evidence when deeper inspection is required.

Behavioral memory adds another dimension: a validated procedure can be retrieved as a compact execution hint, while the underlying episodes and evidence remain available for inspection when confidence is low or applicability is uncertain.

## Structured representations

Files, Markdown, JSON/YAML, relational data, embeddings and graphs are representations, not ends in themselves. A structure is valuable when it improves retrieval, reasoning, validation or maintenance.

A practical knowledge base does not need to start with a hosted database or graph. A disciplined collection of local files can be a canonical representation if entries have clear semantics, provenance, freshness and lifecycle rules.

## Open problem

We need empirical benchmarks showing when a context intervention improves task success, rather than assuming that more structure or more retrieved tokens are beneficial. In particular, EOKS should measure whether persistent project knowledge reduces redundant repository discovery without increasing errors caused by stale or incorrect memory, and whether learned procedures improve outcomes rather than merely changing agent behavior.

## Coding-agent workflow as a context lifecycle

Claude Code plugins and hooks expose a useful concrete workflow for studying this problem. An execution workflow can produce decisions, failures, tool traces and review outcomes that become inputs to the knowledge lifecycle.

A useful loop is:

```text
session start
    |
retrieve canonical knowledge + relevant memory + applicable procedures
    |
work / execute / verify
    |
session end
    |
extract candidate facts, episodes, rules, skills and decisions
    |
compare with prior sessions / outcomes
    |
validate + promote selectively
    |
updated knowledge / memory / behavioral policy
```

The important architectural boundary is between **observation**, **candidate extraction**, **validation/promotion**, and **retrieval**. This makes a session-finalizer hook an instance of the general EOKS knowledge lifecycle rather than a special-case Claude Code feature.

## Behavioral learning loop

For continuous learning, the session itself becomes an evidence source:

```text
execution
   |
trace + artifacts + corrections + outcome
   |
background reflection
   |
semantic / episodic / procedural candidates
   |
validation against evidence and outcomes
   |
promoted knowledge or execution policy
   |
future context + workflow selection
```

The system should not automatically generalize every repeated action. Repetition establishes evidence that a behavior occurred; outcome-linked evidence is needed to establish that it is useful. Scope, prerequisites, confidence and counterexamples should travel with a learned procedure.
