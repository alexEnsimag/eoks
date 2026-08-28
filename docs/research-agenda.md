# Research agenda

## 1. Context quality

Build a benchmark where the same task is given different context constructions. Measure correctness, completeness, hallucination/error rate, latency and token cost. Test whether structured context, retrieval, graphs and context splitting produce measurable gains.

Treat context quality as multidimensional rather than assuming a single "context entropy" score. At minimum measure relevance, coverage, redundancy, correctness/reliability, uncertainty, freshness, dependency completeness, provenance, contradiction risk, ordering/structure and token/latency cost.

A useful experimental quantity is **marginal context value**: the change in task quality associated with adding a block relative to its token/latency cost. Also test **context necessity**: after a run, label selected blocks as essential, useful, irrelevant or misleading.

The detailed controlled methodology is [Context evaluation and benchmarking](../research/context-evaluation.md).

## 2. Context observability and workbench

Prototype a UI that shows context as inspectable blocks/clusters: source, relevance, provenance, freshness, confidence and token cost. Allow humans to understand what the model saw and what it did not see.

Extend the prototype with:

- include/exclude/pin operations;
- "why was this selected?" explanations;
- context budgets;
- automatic-vs-optimized context diffs;
- a graph view connecting tasks, knowledge, code and evidence;
- block-level provenance and freshness;
- saved context compositions/recipes;
- outcome feedback from human edits.

The UI should not require manual context curation for every task. Automatic selection remains the baseline; human interaction is most valuable for inspection, correction and learning.

## 3. Context contracts for subagents

Test whether a fresh subagent receiving a compact task/context contract reduces repository rediscovery without reducing task success. Compare unconstrained exploration with progressively more explicit context contracts and measure discovery calls, tokens, latency, correctness and omissions.

## 4. Context compilation and budgets

Prototype a local context compiler that accepts a task and produces a reproducible context manifest. Test ranking, deduplication, conflict detection, progressive disclosure and hard token budgets.

A compiled context should make it possible to answer:

- what the model saw;
- what it did not see;
- why each item was included;
- which evidence provider supplied it;
- what revision/freshness state applied;
- how many tokens/latency each block consumed.

## 5. Memory lifecycle

Compare transcript memory, curated Markdown/files, structured records and graph memory. Measure retrieval usefulness, stale-memory failures and maintenance cost.

Specifically test whether context compilation can reconstruct useful continuity after a session is cleared or a fresh subagent starts, without relying on conversation compaction as the primary persistence mechanism.

Also compare **flat versus multi-resolution memory**. Test whether L0/L1/L2/L3-style representations—or another hierarchical scheme—improve continuity and retrieval cost while preserving provenance and allowing drill-down to source evidence.

## 6. Reusable context/knowledge assets

Treat reusable memory, Skills, Wiki pages, CodeGraph indexes, decisions and evaluation records as candidate **context/knowledge assets** with different semantics but common lifecycle metadata.

Test:

- asset ownership and access boundaries;
- scope/applicability filtering;
- freshness and invalidation;
- versioning and provenance;
- asset-level evaluation;
- portability across agents and harnesses.

TencentDB Agent Memory is useful prior art for a governed asset model spanning Chat Memory, Skill, LLM-Wiki and CodeGraph. The EOKS hypothesis should remain broader and implementation-agnostic.

## 7. Agent/workload loadouts

Test whether an explicit loadout—assets available and intended for a particular agent or workload—improves context quality compared with unrestricted retrieval.

Compare:

```text
unrestricted asset universe
        vs
fixed loadout
        vs
task-specific loadout
```

Measure irrelevant retrieval, unauthorized exposure, stale-asset use, task outcome and context cost. Separate **eligibility** (access/scope/applicability) from **selection** (relevance/value) and from **compilation** (what actually enters the context).

## 8. Proactive, reactive and hybrid context delivery

Compare:

- proactive context packing before reasoning;
- reactive tool-based exploration;
- hybrid bootstrap + on-demand retrieval.

GrapeRoot is useful prior art for proactive context around an existing coding agent. TencentDB Agent Memory is useful prior art for a hybrid architecture where some memory/Skills are surfaced proactively while Wiki/CodeGraph can be discovered and queried on demand.

Hold the workload and asset universe constant where possible. Measure discovery turns, tool calls, latency, token cost, evidence coverage and end-to-end task outcome.

## 9. Model routing and migration

Construct a heterogeneous workload and compare always-strongest-model against capability/cost-aware routing. Include model upgrades and regression testing. Hold context composition constant when comparing models where possible.

The detailed reliability, model-switching and model/context-interaction methodology is [Evaluation, reliability and model switching](../research/evaluation-and-model-switching.md).

## 10. Deterministic evidence

For software engineering, compare LLM-only reasoning against hybrid workflows using structural graphs, tests, type checking, static analysis and dataflow/taint analysis.

Test **analysis escalation**: type/compiler checks → lightweight rules → targeted compiler/AST analysis → dataflow → deeper interprocedural analysis. Measure setup cost, runtime, coverage, false positives/negatives and usefulness to the agent rather than assuming the deepest analyzer is best.

## 11. Invariants and barriers

Test whether agents can reliably identify architectural invariants from concrete bug investigations and express them independently of a particular analysis tool.

Compare prevention through API/type design with increasingly powerful verification providers. Measure whether an invariant discovered from one failure prevents regressions, how much maintenance it costs, and whether it remains valid as the architecture evolves.

> **Hypothesis:** invariants are a more durable abstraction than individual analyzer rules.

## 12. Evidence-provider selection

Prototype a policy that chooses among graph queries, type checking, tests, static analyzers, runtime evidence and LLM reasoning based on the task. Measure whether the control plane can answer engineering questions with a cheaper sufficient provider instead of always invoking the deepest available analysis.

## 13. Control-plane prototype

Implement the smallest scheduler that accepts a task, chooses context sources/tools/model, executes, evaluates and records the decision. Do not begin with a distributed platform; prove the control loop locally first.

## 14. Continuous assurance

Explore whether evaluation results can automatically alter future routing/context policies. The interesting system is not one that merely measures failures, but one that becomes better at choosing how to work.

## 15. Institutional and system context

Test whether adding service ownership, upstream/downstream boundaries, architectural rationale and validated session knowledge improves software-engineering outcomes compared with repository-only context.

Distinguish repository context, structural dependency context, ownership metadata, architecture/history and a task-scoped combination selected by a context compiler. Measure task success, incorrect-but-plausible decisions, discovery work, context cost and stale-context failures.

Also test session-to-institutional-knowledge promotion and cross-harness portability, with provenance, scope, freshness, contradiction handling and rollback/invalidation.

## 16. LLM observability and reliability signals

Test whether observability can become a useful sensor layer for the control loop. Record execution traces, model/version, context manifest, token usage, latency/cost, uncertainty signals where available, evidence agreement, deterministic checks, evaluator/human scores and final correctness.

Compare model self-assessment, model-computation signals, external evidence/outcomes and combined reliability evidence. Evaluate calibration and **decision utility**, not correlation alone.

Test policies such as:

- high reliability → stop;
- medium reliability → verify;
- low reliability → retrieve more context, sample another answer, switch model or branch the workflow.

Do not allow a reliability signal to control the live loop until it has demonstrated useful calibration on the target workload.

See [LLM observability and reliability signals](../research/llm-observability-and-reliability.md).

## 17. Controlled context and model benchmark

Use a versioned golden set of approximately 20–30 representative software-engineering tasks before introducing a new context system or model. Run controlled context configurations and then compare candidate models on the same configurations. Measure task success, tests, correctness, completeness, groundedness, serious regressions, tool calls, tokens, latency and cost.

Community tooling to investigate includes Promptfoo, Langfuse, Aider benchmarks, OpenHands benchmarks and OpenAI Evals-style frameworks. The goal is to reuse experiment/trace/evaluation infrastructure beneath EOKS rather than duplicate it.

See [Context evaluation and benchmarking](../research/context-evaluation.md) for the detailed experiment design.
