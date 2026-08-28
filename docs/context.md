# Context engineering

Context engineering is the discipline of constructing the information available to a model for a specific reasoning step.

> **Knowledge is persistent; context is compiled for a task.**

Context engineering is therefore the runtime boundary between durable knowledge/evidence and model input. A repository, graph, memory store or knowledge bundle can contain much more information than should enter a model context.

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

## Evidence providers

Graphs, semantic indexes, timelines, ADRs, static analysis, tests, runtime observations and knowledge bundles are **providers**, not context themselves. The context compiler selects and transforms their output.

```text
Task: change authentication
        |
        +--> structural provider -> affected symbols/files
        +--> knowledge provider  -> relevant invariants/decisions
        +--> history provider   -> incidents/recent changes
        +--> verification       -> tests/static checks
        |
        v
  task-specific context
```

EOKS should prefer the cheapest provider that can reliably answer the question and retain provenance with the resulting evidence.

## Context/knowledge assets

Recent agent-memory systems suggest a useful abstraction above individual stores: a **reusable asset** is a durable, governed resource that may contribute information, procedures or evidence to future workloads.

Examples include:

- semantic/episodic memory;
- procedural Skills;
- project documentation and Wiki pages;
- structural CodeGraph/indexes;
- ADRs and decisions;
- test/evaluation records;
- runtime or incident knowledge.

These assets should not be treated as semantically identical. A Skill is a procedure; a CodeGraph is a representation; a memory item is an experience-derived claim; an ADR is human-reviewed rationale. They can nevertheless share lifecycle metadata such as provenance, scope, freshness, revision, ownership/access and validation state.

TencentDB Agent Memory is useful prior art for this model: it currently manages Chat Memory, Skill, LLM-Wiki and CodeGraph as reusable assets, with agent/team ownership and loadout concepts. See [TencentDB Agent Memory](../research/prior-art/tencent-agent-memory.md).

## Asset universe, loadout and compiled context

Context selection is clearer when split into three stages:

```text
asset / knowledge universe
            |
       access + scope
            |
      agent/task loadout
            |
     task + policy + budget
            |
      context compiler
            |
      compiled context
```

The **loadout** is the set of assets the workload is allowed and expected to use. It is not the final prompt. The compiler still has to decide which portions are useful for the current reasoning step.

This distinction matters because relevance is not the only eligibility criterion. An asset can be relevant but inaccessible, out of scope, stale, contradictory, unverified or too expensive for the task.

## Proactive, reactive and hybrid context delivery

Context engines can differ in *when* they make information available:

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
      |
      v
minimum sufficient task context
```

TencentDB Agent Memory is useful prior art for this hybrid model. Its current Memory Proxy does not simply dump all four asset families into every prompt: different memory levels, Skills and session/team context have different delivery modes, while Wiki and CodeGraph can be discovered and queried on demand.

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

GrapeRoot is useful prior art for a context engine around an existing coding agent. EOKS should not depend on its implementation. Instead, define agent-neutral lifecycle semantics:

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
