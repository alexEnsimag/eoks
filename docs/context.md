# Context engineering

Context engineering is the broader discipline of constructing the information available to a model for a specific reasoning step.

> **Knowledge is persistent; context is compiled for a task.**

The concrete EOKS operation is **context compilation**: selecting, transforming, ordering, compressing and budgeting information for a particular reasoning step. See [Resource model](resource-model.md) for the vocabulary boundary between resources, assets, providers, loadouts and context.

## Context is not storage

```text
engineering reality
       |
knowledge / evidence providers
       |
task + workflow + policy
       |
context compiler
       |
minimal sufficient context
       |
model
```

The model should normally receive the compiled evidence, not the internal graph, index or storage system.

## Navigation versus knowledge

Two related but different goals are useful:

1. **Navigation** — determine where relevant authoritative evidence lives and how it is connected.
2. **Knowledge** — preserve information that does not otherwise exist in authoritative sources, such as rationale, invariants, trade-offs and lessons learned.

A code graph is often excellent navigation evidence. It does not automatically become semantic truth. Conversely, a durable invariant may need an explicit knowledge representation even when the relevant source files are easy to find.

CodeSight is useful prior art for the navigation side: it deterministically derives repository structure and exposes compact indexes, topic-oriented wiki views and targeted MCP queries. Its generated maps should still be treated as derived evidence, not as a replacement for source code or canonical reviewed knowledge.

## Evidence providers

Graphs, semantic indexes, timelines, ADRs, static analysis, tests, runtime observations and knowledge bundles are evidence sources/representations. Providers retrieve or derive evidence from them; the context compiler selects and transforms the result.

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

EOKS should prefer the cheapest provider that can reliably answer the question and retain provenance with the resulting evidence.

CodeSight fits here as a **repository-context/evidence provider**. It is particularly useful for deterministic structure, dependency/impact information and progressive-disclosure views. It should not be required as an EOKS dependency; equivalent providers can implement the same conceptual contract.

## Context sources are heterogeneous

Memory, Skills, project documentation, decisions, CodeGraph/indexes, test results, runtime observations and historical records can all contribute to context. They should retain their semantic distinctions and authority rather than being flattened into one "memory" store.

These resources can share the generic governance/lifecycle abstraction described in [Resource model](resource-model.md), but **Asset is not a new semantic knowledge category**.

TencentDB Agent Memory is useful prior art because it explicitly combines Chat Memory, Skills, LLM-Wiki and CodeGraph with governance and agent loadouts. Its detailed mapping and current delivery model are documented in [TencentDB Agent Memory](../research/prior-art/tencent-agent-memory.md).

## Asset universe, loadout and compiled context

The selection boundary is:

```text
all reusable resources
          |
 governance: access / ownership / scope / lifecycle
          |
       agent/task loadout
          |
 relevance / applicability / value / budget
          |
    context compilation
          |
     compiled context
```

A **loadout** is the workload-scoped set of assets an agent/task is allowed and expected to use. It is not the final prompt. Context compilation still decides what is useful for the current reasoning step.

This distinction matters because relevance is not the only eligibility criterion. A resource can be relevant but inaccessible, out of scope, stale, contradictory, unverified or too expensive for the task.

For the formal vocabulary, use [Resource model](resource-model.md) rather than redefining "asset" in multiple documents.

## Proactive, reactive and hybrid context delivery

Context engines can differ in *when* they make information available.

### Proactive

Likely-relevant evidence is compiled and supplied before the model starts reasoning.

```text
Task -> retrieval/packing -> context -> model
```

GrapeRoot is useful prior art for this style around coding agents.

### Reactive

The model discovers and requests information through tools as needed.

```text
Task -> model -> query -> evidence -> model -> query ...
```

This preserves flexibility but can increase discovery turns, latency and context overhead.

### Hybrid

A compact bootstrap is supplied proactively, while larger or less-certain knowledge remains available through on-demand tools.

```text
bootstrap context
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

TencentDB Agent Memory is useful prior art for this hybrid model. Its current Memory Proxy does not simply dump all four asset families into every prompt: different memory levels, Skills and session/team context have different delivery modes, while Wiki and CodeGraph can be discovered and queried on demand.

CodeSight's index → targeted article/tool query pattern is another concrete example of progressive disclosure. It is useful evidence for the proposition that a small bootstrap plus targeted retrieval can reduce unnecessary context loading, but the reported token savings should be validated against task outcomes rather than treated as an architectural guarantee.

EOKS should not assume that one delivery mode is universally superior. Compare proactive, reactive and hybrid strategies by task outcome, evidence coverage, discovery work, latency and cost.

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

They are useful for mental-model information such as responsibilities, boundaries, invariants, entry points, pitfalls and links to decisions. They should not become encyclopedias or duplicate the source code.

Other representations can coexist. **OKF is one portable representation, not the EOKS knowledge layer itself.** The canonical OKF details and interoperability rules belong in the knowledge-representation research; this document only establishes the boundary:

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

GrapeRoot is useful prior art for a context engine around an existing coding agent. CodeSight is complementary: it is more naturally a repository evidence provider and progressive-disclosure interface than a complete agent lifecycle controller. EOKS should not depend on either implementation. Instead, define agent-neutral lifecycle semantics:

```text
prepare context
      |
      v
agent execution
      |
before execution (optional policy/evidence check)
      |
tool execution
      |
observe outcome
      |
finalize / persist
```

Adapters can map those semantics to Claude Code hooks, MCP, or another runtime. A session-start/compaction event is naturally suited to reconstructing context; a pre-tool event can optionally perform a policy or evidence check. The exact hook names are runtime-specific.

## Context quality

Context quality should consider relevance, correctness, freshness, completeness, redundancy, contradiction risk, provenance, ordering, token/latency cost and interaction with the chosen model. The goal is not maximum information but maximum useful evidence per unit of reasoning cost.

Context should be represented conceptually as **inspectable blocks** rather than an opaque prompt. Blocks can represent constraints, knowledge, decisions, structural evidence, tests, runtime observations, history or working hypotheses. A Context Workbench can expose why blocks were selected, their provenance and cost, and allow controlled include/exclude/pin operations.

See [Context Workbench](context-workbench.md) for the proposed block/workbench model.

## Progressive and split context

Prefer progressive disclosure over indiscriminate repository-wide context stuffing:

```text
compact pointer / summary
        |
        +--> authoritative source
        +--> related evidence
```

CodeSight provides a concrete example of this pattern through a compact wiki index followed by targeted topic articles and specialized queries. The EOKS generalization is broader: the pointer may lead to a source file, ADR, graph query, analyzer result, test, runtime trace or another provider.

Different workflow nodes can request different context packages—for example discovery, planning, implementation and verification. Fresh subagents should receive an explicit starting contract rather than repeatedly rediscovering the repository.

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

## Relationship to compaction and model routing

Conversation compaction preserves useful information inside a continuing conversation. Context compilation reconstructs task-specific context from durable knowledge and authoritative evidence. They are related but different operations.

Model routing chooses **which model** to use; context compilation chooses **what information** to give it. Both should be evaluated independently before optimizing them jointly.

## Evaluation boundary

Context interventions must be evaluated on end-to-end task outcomes, not only retrieval or token metrics. The canonical methodology, benchmark design, model-migration process and prior-art evaluation tools live in [Context evaluation and benchmarking](../research/context-evaluation.md) and [Evaluation](evaluation.md).
