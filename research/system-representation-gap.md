# Cross-repository, cross-language system representation gap

EOKS currently has several promising repository/code representation providers, but there is an unresolved gap between **repository-local code understanding** and an **end-to-end representation of a software system**.

## The target capability

The target is not simply a code graph. We want a navigable representation that can connect:

```text
code
  → symbols / modules
  → semantic/domain concepts
  → repositories
  → services
  → APIs / RPC
  → queues / events
  → datastores
  → deployment/runtime relationships
  → evidence (tests, traces, configs)
```

It should ideally work across languages and repository boundaries and support both human exploration and agent retrieval.

## Why repository-local graphs are insufficient

A graph can be excellent at answering:

- where is this symbol?
- what calls this function?
- what depends on this module?
- where is this concept implemented?

while still being unable to answer:

- which repository owns the other side of this API?
- which service consumes this event?
- which deployment connects these components?
- what is the end-to-end path from this user action to this datastore?
- which code implements the same system capability across repositories and languages?

Those are the questions the missing system-level representation must address.

## Research directions

Investigate providers in distinct families rather than searching only for "AI code graph":

1. **Polyglot static/code intelligence** — repository and symbol relationships across languages.
2. **Cross-repository dependency graphs** — service/API/package boundaries and ownership.
3. **Architecture/service maps** — logical services, APIs, queues, databases and dependencies.
4. **Runtime topology** — traces and telemetry that can connect static structure to observed behavior.
5. **Architecture visualization** — interactive exploration of the resulting multi-level graph.
6. **Unified code + runtime models** — systems that explicitly join static and runtime evidence.

## Evaluation criteria

For each candidate, record:

- languages supported
- repository scope
- whether multiple repositories can be represented in one model
- supported relationship types
- static vs runtime evidence
- semantic/LLM-derived vs deterministic relationships
- visualization quality and navigation depth
- machine-readable export/API
- incremental update behavior
- provenance/explainability
- harness/agent integration
- local vs external service requirements
- operational cost

## EOKS hypothesis

The likely answer may not be one tool.

A promising architecture may be a **provider composition**:

```text
Understand Anything / semantic code provider
             +
static/dependency provider
             +
service/API/runtime provider
             ↓
      EOKS system representation
             ↓
      context selection / compilation
```

The EOKS layer should remain responsible for deciding which representation is needed for a question. No provider should become the canonical truth merely because it has the best visualization.

## Phase A outcome

Phase A should produce an explicit answer to:

> What is the smallest practical provider combination that can represent a real polyglot, multi-repository system well enough for both human navigation and agent context construction?

If no existing combination is satisfactory, the missing capability itself becomes an EOKS design/research target.
