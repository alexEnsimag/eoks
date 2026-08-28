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

For software engineering, compare LLM-only reasoning against hybrid workflows using structural graphs, tests, type checking, static analysis and dataflow/taint analysis.

An important sub-question is **analysis escalation**: does selecting the minimum sufficient deterministic analyzer improve overall cost/latency without reducing correctness? Compare, for representative tasks:

1. type/compiler checks;
2. lightweight lint/static rules;
3. targeted TypeScript compiler-API/`ts-morph` analysis;
4. Semgrep-style dataflow analysis;
5. deeper CodeQL-style analysis.

Do not assume the deepest analyzer is the best answer. Measure setup cost, runtime, coverage, false positives/negatives and usefulness to the agent.

## 8. Invariants and barriers

Test whether agents can reliably identify architectural invariants from concrete bug investigations and express them independently of a particular analysis tool.

Use cases should include source → barrier → sink properties where structural graph traversal alone is insufficient. For each candidate invariant, compare:

- prevention through TypeScript types/API design;
- ESLint or other lightweight rules;
- targeted `ts-morph`/compiler-API analysis;
- Semgrep dataflow rules;
- CodeQL queries where the problem genuinely requires deeper program analysis.

Measure whether an invariant discovered from one failure prevents regressions, how much maintenance it costs, and whether the resulting rule remains valid as the architecture evolves.

A useful hypothesis is:

> **Invariants are a more durable abstraction than individual analyzer rules.**

The analyzer should provide evidence for the invariant; EOKS should retain the invariant's scope, provenance, confidence and enforcement status.

## 9. Evidence-provider selection

Prototype a policy that chooses among graph queries, type checking, tests, static analyzers, runtime evidence and LLM reasoning based on the task. Measure whether the control plane can answer engineering questions with a cheaper sufficient provider instead of always invoking the deepest available analysis.

Record provider selection, evidence returned, analysis cost, revision, confidence and final task outcome so the policy itself can be evaluated.

## 10. Control-plane prototype

Implement the smallest scheduler that accepts a task, chooses context sources/tools/model, executes, evaluates and records the decision. Do not begin with a distributed platform; prove the control loop locally first.

The first control loop should expose context selection and **evidence-provider selection** as policy decisions, not merely as internal prompt-building or tool-calling implementation details.

## 11. Continuous assurance

Explore whether evaluation results can automatically alter future routing/context policies. The interesting system is not one that merely measures failures, but one that becomes better at choosing how to work.

For context specifically, study whether repeated human include/exclude actions predict useful future selection, while guarding against overfitting a preference observed on one task, repository revision or model.

## 12. Institutional and system context

Xirp highlights a dimension that is easy to miss when context research is framed only around repository contents: agents may need **organizational/system context** such as service ownership, upstream/downstream boundaries, architectural rationale and knowledge accumulated by previous engineers or agents.

Test whether adding this context improves software-engineering outcomes compared with repository-only context. The benchmark should distinguish at least:

1. repository/code context only;
2. repository context + structural dependency graph;
3. repository context + ownership/service metadata;
4. repository context + relevant architectural decisions/history;
5. a task-scoped combination selected by a context compiler.

Measure task success, incorrect-but-plausible decisions, discovery work, context cost and stale-context failures. The key question is whether broader context improves decisions **when selected selectively**, rather than whether more information is always better.

Also test **session-to-institutional-knowledge promotion**: whether useful discoveries from one coding session can be validated and made available to later engineers or agents without turning transient or incorrect reasoning into canonical knowledge.

Important lifecycle properties to measure include provenance, source revision, scope/applicability, freshness, contradiction handling, human correction and rollback/invalidation.

Finally, test **cross-harness portability**: whether the same promoted knowledge and compiled context remain useful when execution switches between different coding-agent/model harnesses. This helps separate durable EOKS knowledge/context capabilities from vendor-specific agent state.

## 13. LLM observability and reliability signals

Test whether AI observability can become a useful **sensor layer** for the EOKS control loop rather than merely a debugging dashboard.

Start with an offline benchmark where every task has a known outcome. Record:

- full execution trace and tool/retrieval steps;
- model/version and context manifest;
- token usage, latency and cost;
- model self-assessment where available;
- token probabilities/logprobs and entropy where available;
- semantic agreement across multiple generations for a selected subset;
- evidence agreement, contradiction and provenance;
- deterministic checks and test outcomes;
- evaluator/human scores;
- final correctness.

Compare four increasingly rich reliability signals:

1. model self-assessment;
2. model-computation signals such as logprobs/entropy;
3. external evidence and outcome signals;
4. a combined reliability evidence model.

Evaluate both calibration and **decision utility**. The important question is not merely whether a signal correlates with correctness, but whether using it reduces bad actions, unnecessary retries and unnecessary expensive analysis.

Test control policies such as:

- high reliability → stop;
- medium reliability → verify with a deterministic provider;
- low reliability → retrieve more context, sample another answer, switch model or branch the workflow.

Do not allow a reliability signal to control the live loop until it has demonstrated useful calibration on the target workload.

### Key hypothesis

> **Observability supplies the sensors; evaluation supplies outcome labels; calibration turns raw signals into workload-specific reliability evidence; the control plane uses that evidence to choose the next action.**

See [LLM observability and reliability signals](../research/llm-observability-and-reliability.md).
