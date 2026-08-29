# Empirical agent context and intelligence infrastructure

## Why this note exists

Recent discussion sharpened an important EOKS principle: **do not turn plausible agent infrastructure into architecture by intuition**. Frontier models are improving quickly, and a technique that helps one model, repository or task may be redundant—or harmful—for another.

EOKS should therefore treat graphs, retrieval, semantic indexes, context management, durable knowledge, sub-agents, execution state and other "intelligence infrastructure" as **experimental interventions**. The baseline is the strongest practical agent available with ordinary tools for the workload.

The central research question is:

> **Which external capabilities measurably improve an agent's software-engineering outcome, for which model × repository × task × budget conditions, and at what cost?**

This preserves the original EOKS scope around learning, context optimization, knowledge, code understanding and control without assuming that any particular representation or tool is universally useful.

## Evidence that the problem is real

The recent literature provides several independent signals, while also containing important counter-evidence.

### Long-horizon context degradation

**Context as a Tool: Context Management for Long-Horizon SWE-Agents** (Liu et al., Findings of ACL 2026) explicitly studies context explosion, semantic drift and degraded reasoning in long-running SWE interactions. Its context-management tool and trained SWE-Compressor report 57.6% on SWE-Bench Verified and outperform the paper's ReAct and static-compression baselines under a bounded context budget.

- Paper: https://aclanthology.org/2026.findings-acl.1032/
- arXiv: https://arxiv.org/abs/2512.22087

This is evidence for the **problem**, not proof that proactive compression is the universally correct solution.

### Execution state can be better than replaying history

**Turning Interaction History into Execution State: A Runtime Layer for Long-Horizon Coding Agents** (Wang et al., 2026) introduces Ledger, a deterministic execution-state layer that tracks what has been observed, modified and attempted. Across all 500 SWE-Bench Verified instances, the paper reports Pass@1 improvements from 56.2% to 64.2% for GPT-5 mini and 75.8% to 81.0% for MiniMax M2.5, with total cost reductions of 28.9% and 31.8%. Attached to Codex, it reports +3.4 percentage points at 24.4% lower cost.

- Paper: https://arxiv.org/abs/2608.00808

This is particularly relevant to EOKS because it suggests that **explicit execution state is a different intervention from generic transcript summarization**.

### Agents can also be good long-context processors themselves

**Coding Agents are Effective Long-Context Processors** (Cao et al., 2026) finds that coding agents using executable tools and filesystem navigation can outperform published long-context baselines by 17.3% on average across its evaluated benchmarks, including corpora up to three trillion tokens.

- Paper: https://arxiv.org/abs/2603.20432

This is an important counterweight: repeated grep/read/tool exploration is not automatically "waste". Tool-driven exploration can be a useful form of externalized reasoning and memory access. EOKS should therefore optimize exploration selectively rather than replace it wholesale with a graph or summary.

### Repository retrieval is a measurable upstream bottleneck

**Agent Retrieval Bench: Evaluating Repository Context Retrieval for Coding Agents** (Qin & Xie, 2026) contains 427 samples across 25 repositories, with 392K files and 7.9M chunks. It evaluates lexical retrieval, RepoMap, embeddings, selective retrieval and logged agent context selection. No retrieval family dominates; the best method changes with the metric/task/budget. Logged trajectories miss every gold file in 27–35% of samples, while a retrieval-derived seed context reduces subsequent exploration versus random context in a controlled pilot.

- Paper: https://arxiv.org/abs/2607.24882
- Benchmark: https://agent-retrieval-bench.github.io/

This is a strong argument for **measuring context acquisition separately from final patch success**.

### Retrieval can beat agentic search in some workloads

**Deep Agentic Search for Repository-Level Code Question Answering: An Empirical Study** (Oskooei et al., 2026) compares indexed semantic retrieval with a planner delegating repository exploration to a sub-agent. On its SWE-QA benchmark, semantic search answered 65.2% correctly versus 46.2% for deep agentic search and produced correct answers at less than half the cost. The study is read-only repository QA, not autonomous code modification, so it should not be generalized beyond that workload.

- Paper: https://arxiv.org/abs/2608.01507

This is an important warning against assuming that isolated sub-agent exploration is automatically better context engineering.

### Long-horizon repository transformation remains difficult

**SWE Refactor Bench: Can Coding Agents Complete a Long-Horizon, Whole-Repository Stack Migration?** evaluates 20 whole-repository migrations. Its public results report 520 runs across 8 frontier models and 26 effort configurations, with only 28/520 runs (5.4%) passing all three stages of migration audit, behavioural tests and independent agentic verification; 13 of 20 tasks received no accepted solution.

- Paper: https://arxiv.org/abs/2608.23564
- Benchmark: https://github.com/Einsia/SWE-Refactor-Bench

This is a useful counterpoint to high scores on narrower SWE benchmarks: **repository-scale semantic preservation and long-horizon transformation are not solved**.

## What the evidence currently supports

| Hypothesis | Current evidence | EOKS stance |
|---|---|---|
| Long-running agents can suffer context degradation | Strong, multiple studies | Measure directly |
| More context is automatically better | Not supported | Treat as a false default |
| Context acquisition is a distinct bottleneck | Strong | Benchmark separately |
| Raw tool exploration can be useful | Strong evidence | Do not suppress blindly |
| Retrieval can reduce exploration cost | Promising/strong in some workloads | Compare against baseline |
| Graphs universally beat grep/search | Not established | Hypothesis only |
| One retrieval family dominates | Evidence against | Keep alternatives |
| Context compression is universally beneficial | Not established | Test by workload |
| Explicit execution state can improve agents | Strong early evidence | High-priority experiment |
| Infrastructure can compensate for weaker models | Plausible, partial evidence | Test model × infrastructure interactions |
| Frontier models make external infrastructure unnecessary | Evidence against as a universal claim | Continue testing |

## The model × repository × task interaction

An intervention should never be evaluated only as "tool X improves agents". At minimum, EOKS should preserve these dimensions:

```text
                 MODEL
                   ×
              REPOSITORY
                   ×
                 TASK
                   ×
              INTERVENTION
                   ×
              CONTEXT BUDGET
```

Examples of plausible but unproven outcomes:

```text
Opus + modern repository + graph      -> marginal benefit
small model + legacy repository + graph -> large benefit
Opus + huge legacy repository + retrieval -> large cost reduction
small model + poor retrieval           -> worse than raw exploration
```

These are **experiment hypotheses, not conclusions**.

This also creates a potentially important economic question:

> Can infrastructure narrow the performance/cost gap between expensive frontier models and cheaper models?

EOKS should measure this rather than assume it.

## Modern versus legacy repositories

The EOKS benchmark should deliberately distinguish repository classes.

### AI-friendly / modern

- clear architecture;
- consistent naming;
- tests;
- current documentation;
- explicit boundaries;
- relatively recent design decisions.

### Mature / legacy

- multiple languages and generations;
- missing or unreliable tests;
- generated code;
- implicit contracts;
- undocumented dependencies;
- tribal or historical knowledge;
- inconsistent conventions;
- large repository-wide blast radius.

A model's pretrained knowledge cannot supply repository-specific historical facts it never observed. Conversely, a graph or index can expose structure without necessarily supplying the semantics or rationale. EOKS should therefore test **repository structure, durable knowledge, history and organizational/system context separately**.

## Intelligence infrastructure as a research category

"Intelligence infrastructure" is useful terminology, but it is **not an EOKS architectural conclusion**.

Use it as a category for mechanisms that augment an agent's ability to:

1. acquire information;
2. organize information;
3. maintain useful working state;
4. execute and observe actions;
5. validate beliefs against evidence;
6. recover and continue across long trajectories.

The design principle is:

> **Do not replace model reasoning with infrastructure; eliminate work the model should not have to perform, while preserving useful exploration and semantic reasoning.**

A graph may therefore be valuable as a shortcut through a structural search space, while the agent still reads and reasons over the underlying code. Likewise, an execution ledger may reduce repeated work without deciding what the agent should believe.

## Three research layers

The recent literature suggests organizing EOKS experiments into three related layers.

### A. Knowledge acquisition

> How does the agent find what it needs?

Compare:

- raw grep/search/read;
- lexical retrieval;
- embeddings;
- RepoMap-like summaries;
- LSP/semantic tooling;
- structural graphs;
- agentic sub-search;
- hybrid retrieval;
- pre-computed repository analysis.

### B. Knowledge lifecycle / working state

> How does the agent retain what matters without drowning in history?

Compare:

- raw append-only trajectory;
- static summarization;
- agent-controlled context management;
- selective historical retrieval;
- execution-state ledgers;
- structured session state;
- durable task/session records;
- fresh subagent context with typed task contracts.

### C. Knowledge validation

> How does the agent know that its current understanding is correct?

Compare:

- model-only reasoning;
- tests;
- compiler/type checks;
- static analysis;
- graph evidence;
- dataflow analysis;
- runtime evidence;
- independent reviewers;
- combined evidence policies.

These layers should not be collapsed into a single "context quality" score.

## The EOKS benchmark matrix

The next-generation EOKS benchmark should compare interventions against the strongest practical baseline.

### Baselines

1. Strong agent with ordinary filesystem/search/edit/test tools.
2. Strong agent with the project's existing instructions/skills.
3. Same agent plus one intervention at a time.
4. Combined interventions only after individual effects are measured.

### Models

At least:

- frontier/high-capability model;
- mid-tier model;
- lower-cost model.

The exact model roster should be versioned with the benchmark because model behaviour changes rapidly.

### Repository classes

- small/modern;
- medium/mature;
- large/legacy;
- multi-language/multi-service where possible.

### Task classes

- navigation / codebase question;
- bug localization;
- impact analysis;
- implementation;
- refactoring;
- repository-wide migration;
- review;
- verification.

### Interventions

- raw exploration;
- retrieval;
- RepoMap/structural summary;
- graph;
- semantic/LSP tools;
- durable knowledge;
- context management;
- execution state;
- sub-agent exploration;
- hybrid combinations.

### Measurements

**Outcome**
- correctness;
- completeness;
- regression rate;
- verification success.

**Efficiency**
- input/output tokens;
- total cost;
- wall-clock time;
- tool calls;
- repository rediscovery.

**Context health**
- context size over time;
- repeated/duplicate searches;
- stale information reuse;
- irrelevant context;
- contradiction rate;
- useful-context retention.

**Autonomy**
- successful steps before intervention;
- session restarts;
- recovery after failure;
- completion without human reset.

**Evidence quality**
- provenance;
- deterministic evidence coverage;
- reviewer agreement;
- false confidence;
- evidence-to-outcome calibration.

## A particularly important metric: reasoning efficiency

For EOKS, raw token reduction is insufficient.

Define an experimental notion of **reasoning efficiency** as the useful task progress obtained per unit of agent effort:

```text
useful task progress
--------------------
 tokens + latency + tool/analysis cost
```

The numerator should be operationalized through intermediate task milestones where possible, not by asking the same model to judge its own progress.

A tool that reduces tokens but causes more errors is not an optimization. A tool that adds tokens but prevents a major regression may be valuable. A tool that saves 70% of search work while preserving semantic exploration is especially interesting.

## What not to conclude

EOKS should explicitly resist these shortcuts:

- "The model has a large context window, so context engineering is obsolete."
- "The agent uses grep, therefore grep is inefficient."
- "Graphs encode semantics, therefore the model no longer needs to explore."
- "A popular project proves the approach works."
- "A benchmark improvement on one model generalizes to all models."
- "Retrieval metrics automatically imply task success."
- "A smaller context is always better."
- "More agents are better."
- "A better summary is equivalent to repository understanding."

Popularity is useful **research prioritization signal**; adoption and community reports tell us what is worth investigating. Controlled benchmarks and independent evidence determine whether it works.

## Proposed EOKS research loop

```text
community signal / paper / tool
              |
              v
        research hypothesis
              |
              v
       controlled benchmark
              |
       +------+------+
       |             |
       v             v
   improvement     regression
       |             |
       +------+------+
              v
     model/task/repo analysis
              |
              v
       EOKS evidence record
              |
       +------+------+
       |             |
       v             v
   provisional     reject /
   guidance        revisit
       |
       v
  architecture decision
       only when evidence is sufficient
```

This should become a defining property of EOKS: **the repository is a laboratory, not a catalog of beliefs**.

## Relationship to existing EOKS work

This research extends, rather than replaces, the existing work on:

- [Context engineering](context-engineering.md);
- [Context evaluation](context-evaluation.md);
- [Agent code understanding and architecture](agent-code-understanding-and-architecture.md);
- [Evaluation and model switching](evaluation-and-model-switching.md);
- [Session learning](session-learning.md);
- [Memory and knowledge lifecycle](memory-and-knowledge.md).

The practical next step is to make the benchmark matrix and evidence records executable, so that new tools and papers can be added without repeatedly rewriting the architecture around the latest promising technique.
