# Research agenda

## 1. Context quality

Build a benchmark where the same task is given different context constructions. Measure correctness, completeness, hallucination/error rate, latency and token cost. Test whether structured context, retrieval, graphs and context splitting produce measurable gains.

Treat context quality as multidimensional rather than assuming a single "context entropy" score. At minimum measure relevance, coverage, redundancy, correctness/reliability, uncertainty, freshness, dependency completeness, provenance, contradiction risk, ordering/structure and token/latency cost.

A useful experimental quantity is **marginal context value**: the change in task quality associated with adding a block relative to its token/latency cost. This should initially be treated as a benchmark metric, not as an assumed online probability model.

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

Test whether a fresh subagent receiving a compact task/context contract reduces repository rediscovery without reducing task success. Compare:

1. unconstrained fresh exploration;
2. task + minimal context instructions;
3. typed context blocks + relevant dependency/evidence slice;
4. typed context blocks + explicit exclusion/scope hints.

Measure discovery tool calls, tokens, latency, correctness and omissions.

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

## 6. Model routing

Construct a workload with heterogeneous task types. Compare always-strongest-model against capability/cost-aware routing. Include model upgrades and regression testing.

Run routing experiments after controlling context composition where possible. Otherwise it is difficult to distinguish model capability from differences in the information supplied to each model.

## 7. Deterministic evidence

For software engineering, compare LLM-only reasoning against hybrid workflows using static analysis, code graphs, tests and dataflow/taint analysis.

## 8. Control-plane prototype

Implement the smallest scheduler that accepts a task, chooses context sources/tools/model, executes, evaluates and records the decision. Do not begin with a distributed platform; prove the control loop locally first.

The first control loop should expose context selection as a policy decision, not merely as an internal prompt-building implementation detail.

## 9. Continuous assurance

Explore whether evaluation results can automatically alter future routing/context policies. The interesting system is not one that merely measures failures, but one that becomes better at choosing how to work.

For context specifically, study whether repeated human include/exclude actions predict useful future selection, while guarding against overfitting a preference observed on one task, repository revision or model.
