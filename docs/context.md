# Context engineering

Context engineering is the discipline of constructing the information available to a model for a specific reasoning step.

> **Knowledge is persistent; context is compiled for a task.**

The concrete EOKS operation is **context compilation**: selecting, transforming, ordering, compressing and budgeting information for a particular reasoning step. See [Resource model](resource-model.md) for the vocabulary boundary between resources, assets, providers, loadouts and context.

## Context is not storage

```text
engineering reality
       |
knowledge / evidence / execution-state providers
       |
task + workflow + policy
       |
context acquisition + selection
       |
context compiler
       |
model
```

The model should normally receive the evidence it needs, not the internal graph, index or storage system itself.

## Context, acquisition and execution state

Three concepts must remain distinct:

**Context** — information presented to the model for the current reasoning step.

**Context acquisition** — the work required to discover or derive that information: search, grep/read exploration, retrieval, graph queries, LSP, analyzers, repository indexes or sub-agents.

**Execution state** — durable workflow state describing what has already been observed, established, changed, attempted, verified or invalidated.

```text
acquire information -> working context -> reason/act -> update execution state
```

Raw exploration is not automatically waste. An agent can use tools to build semantic understanding. EOKS therefore treats retrieval, graphs and other infrastructure as competing interventions rather than assuming they should replace exploration.

## Navigation versus knowledge

Two related but different goals are useful:

1. **Navigation** — determine where relevant authoritative evidence lives and how it is connected.
2. **Knowledge** — preserve information that does not otherwise exist in authoritative sources, such as rationale, invariants, trade-offs and lessons learned.

A code graph is often excellent navigation evidence. It does not automatically become semantic truth. Conversely, a durable invariant may need an explicit knowledge representation even when the relevant source files are easy to find.

## Evidence providers

Graphs, semantic indexes, timelines, ADRs, static analysis, tests, runtime observations and knowledge bundles are evidence sources/representations. Providers retrieve or derive evidence from them; context acquisition and compilation select and transform the result.

```text
Task: change authentication
        |
        +--> structural provider -> affected symbols/files
        +--> repository-context provider -> routes/schema/wiki evidence
        +--> knowledge provider  -> relevant invariants/decisions
        +--> history provider   -> incidents/recent changes
        +--> verification       -> tests/static checks
        |
        v
  task-specific context
```

EOKS should prefer the cheapest provider that can reliably answer the question, while retaining provenance, scope and freshness with the resulting evidence.

## Context sources are heterogeneous

Memory, Skills, project documentation, decisions, CodeGraph/indexes, test results, runtime observations and historical records can all contribute to context. They should retain their semantic distinctions and authority rather than being flattened into one “memory” store.

These resources can share the generic governance/lifecycle abstraction described in [Resource model](resource-model.md), but **Asset is not a new semantic knowledge category**.

## Asset universe, loadout and compiled context

```text
all reusable resources
          |
 governance: access / ownership / scope / lifecycle
          |
       agent/task loadout
          |
 relevance / applicability / value / budget
          |
    context acquisition
          |
    context compilation
          |
     compiled context
```

A **loadout** is the workload-scoped set of assets an agent/task is allowed and expected to use. It is not the final prompt. Context compilation still decides what is useful for the current reasoning step.

A resource can be relevant but inaccessible, out of scope, stale, contradictory, unverified or too expensive for the task.

## Proactive, reactive and hybrid delivery

Context engines can differ in when they make information available.

### Proactive

```text
Task -> retrieval/packing -> context -> model
```

### Reactive

```text
Task -> model -> query -> evidence -> model -> query ...
```

### Hybrid

```text
compact bootstrap
      |
      +--> memory recall
      +--> Skill retrieval/execution
      +--> Wiki drill-down
      +--> CodeGraph queries
      +--> repository-context queries
      |
      v
minimum sufficient task context
```

EOKS should not assume one delivery mode is universally superior. Compare proactive, reactive and hybrid strategies by task outcome, evidence coverage, discovery work, context growth, latency and cost.

## Context quality

Context quality should consider relevance, correctness, freshness, completeness, redundancy, contradiction risk, provenance, ordering, dependency completeness, acquisition cost and token/latency cost. The goal is not maximum information but maximum useful evidence per unit of total reasoning cost.

Context should be represented conceptually as **inspectable blocks** rather than an opaque prompt. Blocks can represent constraints, knowledge, decisions, structural evidence, tests, runtime observations, history or working hypotheses.

See [Context Workbench](context-workbench.md) and [Context quality model](../research/context-quality-model.md).

## Progressive disclosure and exploration

Prefer progressive disclosure over indiscriminate repository-wide context stuffing when experiments show that it preserves understanding:

```text
compact pointer / structural answer
        |
        +--> authoritative source
        +--> related evidence
        +--> deeper graph/analyzer query
```

However, exploration itself can be useful semantic work. EOKS should measure whether an intervention reduces *avoidable* discovery without removing the investigation needed to understand unfamiliar code.

Different workflow nodes can request different context packages—for example discovery, implementation and verification. Fresh subagents should receive an explicit starting contract when evidence exists, while retaining the ability to investigate beyond it.

## Canonical project knowledge

Hierarchical `CLAUDE.md`-style files are one possible representation for concise, human-reviewed local guidance:

```text
/
  CLAUDE.md
  api/
    CLAUDE.md
  auth/
    CLAUDE.md
```

They are useful for responsibilities, boundaries, invariants, entry points, pitfalls and links to decisions. They should not become encyclopedias or duplicate source code.

**OKF is one portable representation, not the EOKS knowledge layer itself.** The canonical OKF details and interoperability rules belong in knowledge-representation research.

```text
OKF / CLAUDE.md / ADRs / other durable representations
                         |
                         v
                 knowledge providers
                         |
                         v
                  context compiler
```

## Context engines and lifecycle hooks

A context engine can prepare information around an existing coding agent, while repository evidence providers can remain queryable during execution. EOKS should define agent-neutral lifecycle semantics rather than depend on one runtime:

```text
prepare / acquire context
      |
      v
agent execution
      |
tool execution / evidence requests
      |
observe outcome
      |
update execution state
      |
finalize / persist candidates
```

Adapters can map these semantics to hooks, MCP or another runtime. Session boundaries/compaction are opportunities to reconstruct working context, not necessarily the persistence mechanism itself.

## Knowledge lifecycle

Execution can produce candidate facts, decisions, rules, skills and episodes. They should not automatically become canonical knowledge:

```text
observation
    -> candidate extraction
    -> provenance + validation
    -> promote / update / invalidate
    -> future retrieval
```

This separates observation from durable truth and prevents an incorrect session summary from silently becoming future context.

## Model and repository dependence

Context policies should be evaluated across models and repository classes. An intervention may be unnecessary for a frontier model but useful for a cheaper model, or valuable on a legacy repository but redundant on a clean AI-native one.

Therefore EOKS should evaluate:

```text
model × repository × task × intervention × budget
```

rather than treating context engineering as a model-independent optimization.

## Relationship to compaction and model routing

Conversation compaction preserves useful information inside a continuing conversation. Context compilation reconstructs task-specific context from durable knowledge and authoritative evidence. Execution state records what the workflow has already established. These are related but different operations.

Model routing chooses **which model** to use; context compilation chooses **what information** to give it. Both should be evaluated independently before optimizing them jointly.

## Evaluation boundary

Context interventions must be evaluated on end-to-end task outcomes, not only retrieval or token metrics. The canonical methodology and benchmark matrix live in [Context evaluation and benchmarking](../research/context-evaluation.md).