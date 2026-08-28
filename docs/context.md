# Context engineering

Context engineering is the discipline of constructing the information available to a model for a specific reasoning step.

The key distinction is:

> **Knowledge is persistent; context is compiled for a task.**

Context engineering is therefore not a competing layer to knowledge engineering. It is the runtime boundary that selects and assembles the right slice of persistent knowledge and evidence.

## Context is not storage

A repository, knowledge base, graph or memory store can contain far more information than should enter a model's context. The important operation is the **selection and transformation boundary** between external knowledge and model input.

A useful mental model is:

```text
persistent engineering reality
          |
   knowledge representations
          |
   task + workflow + strategy
          |
   context compilation
          |
   minimal sufficient context
          |
         model
```

The model should usually see the **compiled evidence**, not the internal graph, embedding index or storage system itself.

## Navigation versus knowledge

A particularly useful distinction is between two goals:

1. **Navigation optimization** — determine where the relevant evidence lives and in what order it should be read.
2. **Knowledge optimization** — preserve insights that do not otherwise exist in source material, such as rationale, tradeoffs, invariants and lessons learned.

A structural graph or semantic index can often satisfy the first goal by pointing the agent to relevant files, symbols, ADRs or incidents. The agent can then read the authoritative source rather than receiving a duplicated summary.

Synthetic knowledge is different: if a cross-package invariant or debugging discovery exists nowhere else, it needs a durable representation.

## Canonical project knowledge

A practical coding-agent environment can use hierarchical `CLAUDE.md` files as a canonical, human-reviewable representation of local project knowledge:

```text
/
  CLAUDE.md
  api/
    CLAUDE.md
  auth/
    CLAUDE.md
```

The purpose should be mental-model information rather than a second copy of the code:

- why the package exists;
- responsibilities and boundaries;
- invariants and important constraints;
- entry points and reading order;
- common pitfalls;
- links to cross-cutting architecture decisions.

This is especially attractive because the files live beside the code, are versioned with Git and can be reviewed in normal pull requests.

The hierarchy should remain scoped. Repository-wide instruction files should not become encyclopedias. More local guidance is useful when the agent is working in that part of the tree, while unrelated package knowledge should remain out of the context budget.

## Knowledge representations are not context

Graphs, semantic indexes, timelines, ADR collections and runtime stores are **evidence providers**. They should normally be queried by a context compiler rather than dumped into the model context.

For example:

```text
Task: change authentication
        |
        +--> structural query -> affected packages/files
        +--> semantic query   -> relevant auth concepts
        +--> history query    -> authentication ADRs/incidents
        +--> canonical docs   -> api/CLAUDE.md + auth/CLAUDE.md
        |
        v
  task-specific context
```

This also explains why a graph can be valuable without becoming the canonical knowledge base.

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

"Context entropy" can be retained as an intuitive research question, but EOKS should not assume that one scalar entropy measure is sufficient. The more actionable approach is to expose the dimensions above and measure how context composition affects outcomes.

A promising derived metric is **marginal context value**: the change in expected task quality attributable to a block relative to its token/latency cost. Initially this is a benchmark concept rather than a claim that online task-quality probabilities can be estimated precisely.

## Context blocks and workbench

Context should be represented conceptually as **inspectable blocks** rather than an opaque concatenated prompt. Blocks may represent task constraints, canonical knowledge, decisions, structural evidence, dependency slices, raw evidence, tests, runtime observations, history, procedures or working hypotheses.

The proposed Context Workbench provides a human-facing view over these blocks. It should allow automatic selection while making the selection inspectable and optionally editable:

- include/exclude/pin blocks;
- inspect provenance and freshness;
- ask why a block was selected or omitted;
- inspect token cost;
- compare context compositions;
- impose a context budget;
- review a context diff before execution;
- save successful context recipes for future tasks.

A graph view can complement the block view, but should represent relationships among task requirements, knowledge, code and evidence rather than simply reproduce the repository dependency graph.

See [Context Workbench](context-workbench.md) for the proposed model, context layers, quality dimensions, context contracts, learning loop and UI/benchmark ideas.

## Context layers

A useful decomposition is:

```text
L0 task
L1 constraints
L2 persistent knowledge
L3 structural context
L4 evidence
L5 working memory
L6 reasoning state
```

Different workflow nodes can request different layer budgets. This is an information-architecture boundary, not a requirement that models reason in a particular way.

## Static versus dynamic context

Large repository-wide instruction files can create a poor tradeoff: they are always available but may be irrelevant to the current task. EOKS should prefer **progressive disclosure** and task-scoped retrieval.

This does not imply that static documentation is bad. Well-scoped package-level `CLAUDE.md` files can be valuable precisely because they are local, concise and maintained as part of the codebase. The important distinction is between useful local guidance and indiscriminate global context stuffing.

## Context splitting

Different reasoning steps often need different context. Splitting context can reduce noise and make decisions inspectable: discovery context, design/planning context, implementation context, verification context and historical/project context can be assembled separately.

A workflow node can therefore request a different context package than another node even for the same task.

## Progressive disclosure

The system should prefer exposing the minimum sufficient information and retrieving additional detail when evidence shows it is needed. This resembles filesystem/document navigation more than stuffing an entire corpus into a prompt.

A useful pattern is:

```text
compact pointer / summary
        |
        +--> authoritative source
        |
        +--> related evidence
```

## Subagent context contracts

Subagents are a particularly important consumer of compiled context. A fresh subagent should receive an explicit starting contract containing known facts, relevant nodes, task scope and unresolved questions when available. It should still be allowed to discover missing evidence.

This avoids conflating **isolation** with **rediscovery**: isolated reasoning can be valuable, while repeated reconstruction of repository knowledge is often unnecessary cost.

## Relationship to compaction and model routing

Conversation compaction and context compilation solve different problems. Compaction attempts to preserve useful information inside a continuing conversation; context compilation reconstructs task-specific context from durable knowledge and authoritative evidence after a context boundary or fresh subagent starts.

Similarly, model routing chooses a model while context compilation chooses the information supplied to that model. A router cannot fix waste caused by a strong model repeatedly rediscovering the same repository.

For EOKS experiments, context optimization should therefore be measured independently before assuming that model routing is the primary cost lever.

## Continuous knowledge lifecycle

Claude Code plugins and hooks expose a useful concrete workflow for studying this problem. An execution workflow can produce decisions, failures, tool traces and review outcomes that become inputs to the knowledge lifecycle.

```text
session start
    |
retrieve canonical knowledge + relevant evidence + applicable procedure
    |
work / execute / verify
    |
session end
    |
extract candidate facts, episodes, rules, skills and decisions
    |
validate selectively against source + outcome evidence
    |
promote / update / invalidate
    |
future retrieval and workflow selection
```

The important architectural boundary is between **observation**, **candidate extraction**, **validation/promotion**, and **retrieval**. A session-finalizer hook is therefore one instance of the general EOKS knowledge lifecycle rather than a special-case memory feature.

## Knowledge bundles and OKF

OKF is a useful concrete representation because it is intentionally not a hosted memory service. The current OKF v0.2 specification defines a bundle as a directory tree of Markdown concept documents with YAML frontmatter. `type` is the only always-required frontmatter field; additional fields are allowed, and consumers are expected to tolerate unknown fields.

A local Git-tracked directory can therefore be a valid OKF bundle without running a server:

```text
knowledge/
  architecture/
    authentication.md
  decisions/
    event-processing.md
  invariants/
    auth.md
```

EOKS should distinguish **OKF compatibility** from the broader concept of project knowledge. A team can use its own Markdown conventions when interoperability is not required; when it wants an OKF bundle, it should follow the OKF specification rather than merely applying the label to an arbitrary directory.

The important architectural boundary remains:

```text
OKF / Markdown / ADRs / other durable representations
                         |
                         v
                  knowledge provider
                         |
                         v
                  context compiler
```

OKF is therefore a portable representation, not an EOKS runtime or mandatory storage layer.

## GrapeRoot-like context engines and hooks

GrapeRoot is useful prior art for a context engine that sits around an existing coding agent. Its current public documentation describes a first-run project scan that extracts files, functions, classes and import edges into a local graph, followed by a local MCP server and agent integration. It also documents hooks that re-inject context when Claude compacts its memory and record session information.

This suggests a useful event boundary for EOKS, while avoiding a dependency on GrapeRoot itself:

```text
SessionStart / context boundary
        |
        v
context compiler
        |
        +-- knowledge providers
        +-- structural graphs
        +-- evidence providers
        +-- session state
        |
        v
compiled context
        |
        v
agent
        |
PreToolUse (optional policy/evidence check)
        |
tool execution
        |
PostToolUse / outcome observation
```

The exact hook names are agent-specific. EOKS should define lifecycle semantics such as **prepare context**, **before execution**, **observe outcome** and **finalize/persist**, then provide adapters for Claude Code/GrapeRoot-like hooks, MCP, or other agent runtimes.

A key benchmark requirement is to test whether pre-injection actually improves task outcomes. Reduced exploration or token usage alone is not sufficient evidence of better context.

## Open problem

We need empirical benchmarks showing when a context intervention improves task success, rather than assuming that more structure or more retrieved tokens are beneficial. EOKS should measure at least:

- repository-discovery tool calls;
- time/token cost;
- task success;
- regression/error rate;
- stale-knowledge incidents;
- usefulness of persistent knowledge;
- whether a context intervention improves the outcome for a particular model;
- which context blocks were selected, omitted, manually changed and ultimately useful;
- the cost/benefit of context budgets and block-level optimization;
- model/context interaction during model upgrades;
- regression of representative task classes before promoting a new model or context policy.

See [Context evaluation, benchmarking and model migration](../research/context-evaluation-and-model-migration.md) for the detailed experimental methodology.
