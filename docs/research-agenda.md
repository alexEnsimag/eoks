# Research agenda

The agenda should prioritize experiments that can falsify the EOKS control-plane hypothesis, not additional architectural vocabulary.

## 1. End-to-end vertical slice and attribution

Build one local workload where EOKS can control the complete loop:

```text
Task -> context/evidence selection -> agent execution -> verification
     -> Outcome -> Evaluation -> next Decision
```

Record the exact configuration, context manifest, provider calls, tool calls, model, decisions, costs and outcome. Establish this trace format before building sophisticated schedulers. The key question is whether an intervention can be attributed to an outcome improvement rather than merely correlated with it.

## 2. Context quality

Build a benchmark where the same task is given different context constructions. Measure correctness, completeness, error rate, latency and token cost. Test whether structured context, retrieval, graphs and context splitting produce measurable gains.

Treat context quality as multidimensional rather than assuming a single "context entropy" score. At minimum measure relevance, coverage, redundancy, correctness/reliability, uncertainty, freshness, dependency completeness, provenance, contradiction risk, ordering/structure and token/latency cost.

A useful experimental quantity is **marginal context value**: the change in task quality associated with adding a block relative to its token/latency cost. Initially this is a benchmark metric, not an assumed online probability model.

## 3. Context observability and workbench

Prototype a UI that shows context as inspectable blocks/clusters: source, relevance, provenance, freshness, confidence and token cost. Allow humans to understand what the model saw and what it did not see.

Extend the prototype with include/exclude/pin operations, selection explanations, context budgets, automatic-vs-optimized context diffs, a graph view, block provenance/freshness, saved context recipes and outcome feedback from human edits.

Automatic selection remains the baseline; human interaction is most valuable for inspection, correction and learning.

## 4. Context contracts for subagents

Test whether a fresh subagent receiving a compact task/context contract reduces repository rediscovery without reducing task success. Compare:

1. unconstrained fresh exploration;
2. task + minimal context instructions;
3. typed context blocks + relevant dependency/evidence slice;
4. typed context blocks + explicit exclusion/scope hints.

Measure discovery tool calls, tokens, latency, correctness and omissions.

## 5. Context compilation and budgets

Prototype a local context compiler that accepts a task and produces a reproducible context manifest. Test ranking, deduplication, conflict detection, progressive disclosure and hard token budgets.

A compiled context should make it possible to answer what the model saw, what it did not see, why each item was included, which evidence provider supplied it, what revision/freshness state applied and how many tokens/latency each block consumed.

## 6. Memory and procedural learning

Compare transcript memory, curated Markdown/files, structured records and graph memory. Measure retrieval usefulness, stale-memory failures and maintenance cost.

Test whether structured session traces can produce useful procedural patterns. Compare simple transcript RAG with outcome-linked Learning Records and controlled promotion to Skills/policies. Measure false promotion, stale/contradictory procedures, generalization across task classes and regression after model changes.

Specifically test whether context compilation can reconstruct useful continuity after a session is cleared or a fresh subagent starts, without relying on conversation compaction as the primary persistence mechanism.

## 7. Model routing and migration

Construct a workload with heterogeneous task types. Compare always-strongest-model against capability/cost-aware routing. Include model upgrades and regression testing.

Run routing experiments after controlling context composition where possible. Otherwise it is difficult to distinguish model capability from differences in the information supplied to each model.

## 8. Deterministic evidence

For software engineering, compare LLM-only reasoning against hybrid workflows using structural graphs, tests, type checking, static analysis and dataflow/taint analysis.

An important sub-question is **analysis escalation**: does selecting the minimum sufficient deterministic analyzer improve overall cost/latency without reducing correctness? Compare type/compiler checks, lightweight lint/static rules, targeted TypeScript compiler-API/`ts-morph` analysis, Semgrep-style dataflow and deeper CodeQL-style analysis.

Measure setup cost, runtime, coverage, false positives/negatives and usefulness to the agent. Do not assume the deepest analyzer is the best answer.

## 9. Invariants and barriers

Test whether agents can reliably identify architectural invariants from concrete bug investigations and express them independently of a particular analysis tool.

Use source → barrier → sink properties where structural graph traversal alone is insufficient. For each candidate invariant, compare prevention through types/API design with lightweight rules, targeted analysis, Semgrep dataflow and deeper queries where genuinely required.

Measure whether an invariant discovered from one failure prevents regressions, how much maintenance it costs, and whether it remains valid as the architecture evolves.

> **Invariants are a more durable abstraction than individual analyzer rules.**

## 10. Evidence-provider selection

Prototype a policy that chooses among graph queries, type checking, tests, static analyzers, runtime evidence and LLM reasoning based on the task. Measure whether the control plane can answer engineering questions with a cheaper sufficient provider instead of always invoking the deepest available analysis.

Record provider selection, evidence returned, analysis cost, revision, confidence and final task outcome so the policy itself can be evaluated.

## 11. Control-plane prototype

Implement the smallest scheduler that accepts a task, chooses context sources/tools/model, executes, evaluates and records the decision. Do not begin with a distributed platform; prove the control loop locally first.

The first control loop should expose context selection and evidence-provider selection as policy decisions, not merely internal prompt-building or tool-calling implementation details.

## 12. Orchestration and workflow topology

Test whether explicit orchestration improves outcomes over a single capable agent session. Start with the smallest topology: executor → independent reviewer → verification, with a conductor recording state and deciding whether to retry, branch or escalate.

Compare sequential execution, independent review, parallel specialized workers and no orchestration. Measure task success, defect escape rate, tokens, latency, coordination overhead and the quality of evidence produced by each topology.

The key question is not whether more agents are better; it is whether **explicit control and independent validation** provide enough value to justify orchestration complexity.

## 13. Continuous assurance and adaptive control

Explore whether evaluation results can automatically alter future routing/context policies. The interesting system is not one that merely measures failures, but one that becomes better at choosing how to work.

For context specifically, study whether repeated human include/exclude actions predict useful future selection while guarding against overfitting a preference observed on one task, repository revision or model.

Test stop/continue, retrieve-more, verify, branch, model-switch and human-escalation policies using calibrated evidence. Measure decision utility rather than only prediction accuracy.

## 14. Institutional and system context

Xirp highlights a dimension that is easy to miss when context research is framed only around repository contents: agents may need **organizational/system context** such as service ownership, upstream/downstream boundaries, architectural rationale and knowledge accumulated by previous engineers or agents.

Test repository-only context against repository + structural dependencies, ownership/service metadata, architectural decisions/history and a task-scoped combination selected by a context compiler. Measure task success, incorrect-but-plausible decisions, discovery work, context cost and stale-context failures.

Also test session-to-institutional-knowledge promotion and cross-harness portability: whether promoted knowledge remains useful when execution switches between different coding-agent/model harnesses.

## 15. LLM observability and reliability signals

Test whether AI observability can become a useful **sensor layer** for the EOKS control loop rather than merely a debugging dashboard.

Start with an offline benchmark where every task has a known outcome. Record execution traces, model/version, context manifest, token usage, latency/cost, model self-assessment where available, logprobs/entropy where available, semantic agreement for selected tasks, evidence agreement/contradiction/provenance, deterministic checks, evaluator/human scores and final correctness.

Compare:

1. model self-assessment;
2. model-computation signals such as logprobs/entropy;
3. external evidence and outcome signals;
4. a combined reliability evidence model.

Evaluate calibration and **decision utility**. The important question is not merely whether a signal correlates with correctness, but whether using it reduces bad actions, unnecessary retries and unnecessary expensive analysis.

### Key hypothesis

> **Observability supplies the sensors; evaluation supplies outcome labels; calibration turns raw signals into workload-specific reliability evidence; the control plane uses that evidence to choose the next action.**

See [LLM observability and reliability signals](../research/llm-observability-and-reliability.md).

## 16. Governance, safety and lifecycle

EOKS currently discusses provenance, scope and promotion, but these need an explicit end-to-end experiment. Test access control, secret/PII filtering, retention, deletion, rollback, stale-source invalidation and conflicting-knowledge handling across memory, context and evidence providers.

The research question is whether a useful control plane can remain inspectable and reversible as it accumulates durable state. This should include adversarial cases where a generated summary is plausible but wrong, a source becomes inaccessible, or two authoritative sources disagree.

## 17. The minimum useful semantic model

Instrument the vertical slice and determine which concepts are actually required in traces. Validate whether `Task`, `Context`, `Run`, `Decision`, `Policy`, `Evaluation` and `Outcome` are sufficient as EOKS primitives, and whether Asset/Provider/Representation/Loadout should remain vocabulary-level concepts rather than becoming runtime entities.

Avoid expanding the ontology until a concrete workload demonstrates a missing primitive or relationship.
