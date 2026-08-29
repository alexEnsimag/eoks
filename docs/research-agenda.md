# Research agenda

The agenda should prioritize experiments that can falsify the EOKS control-plane hypothesis, not additional architectural vocabulary.

A central rule is:

> **Treat every infrastructure capability as an intervention whose value depends on the model, workload, repository, budget and baseline.**

The goal is not to prove that graphs, retrieval, memory, context management or multi-agent workflows are good. The goal is to discover **what improves engineering outcomes, when, by how much, at what cost, and with which failure modes**.

## 1. End-to-end vertical slice and attribution

Build one local workload where EOKS can control the complete loop:

```text
Task -> acquisition/context/evidence selection -> agent execution
     -> verification -> Outcome -> Evaluation -> next Decision
```

Record model/version, configuration, context manifest, acquisition/provider calls, tool calls, execution state, decisions, costs and outcome. Establish this trace before building sophisticated schedulers.

The key question is whether an intervention can be attributed to an outcome improvement rather than merely correlated with it.

## 2. Evidence hierarchy and research intake

Use community projects, practitioner reports, academic papers and benchmarks as **signals of what to investigate**. Do not equate popularity with effectiveness.

Use an explicit confidence ladder:

```text
community signal
  -> adoption / repeated reports
  -> academic controlled result
  -> independent reproduction
  -> cross-model/repository replication
  -> EOKS workload-specific evidence
```

Record contradictory results rather than selecting only supporting evidence.

## 3. Context quality and context acquisition

Evaluate the distinction between:

- **context quality** — whether the working set is useful;
- **context acquisition** — how much work is required to obtain it.

Compare raw grep/read/tool exploration with lexical/semantic retrieval, RepoMap-style indexes, LSP/semantic tooling, graphs, agentic search, precomputation and hybrids.

Measure correctness/completeness, evidence coverage, discovery calls, repeated exploration, irrelevant exploration, context growth, tokens, latency and cost.

The key hypothesis is not “infrastructure should replace exploration”, but:

> **Can infrastructure remove avoidable information-acquisition work without removing useful semantic investigation?**

## 4. Context lifecycle and context quality

Test proactive, reactive and hybrid context delivery plus active context-management mechanisms such as compression, offloading and reconstruction.

Measure:

- task outcome;
- relevant evidence retained;
- redundancy/contradiction;
- stale information;
- context growth;
- tokens/latency/cost;
- recovery after context reset.

Explicitly test whether more context helps, hurts or simply changes the agent's exploration behavior.

## 5. Context workbench and observability

Prototype a UI that makes the model's working context inspectable: source, relevance, provenance, freshness, authority and cost. Support include/exclude/pin operations and context diffs.

The purpose is measurement and diagnosis first. Human editing should be evaluated as an intervention rather than assumed to improve the agent.

## 6. Context contracts for subagents

Test whether a fresh subagent receiving a compact task/context contract reduces repository rediscovery without reducing task success.

Compare:

1. unconstrained fresh exploration;
2. task + minimal context instructions;
3. typed context blocks + dependency/evidence slice;
4. typed blocks + explicit scope/exclusion hints.

Measure discovery calls, tokens, latency, correctness, omissions and context pollution in the parent agent.

## 7. Execution state and long-horizon continuity

Treat execution state as distinct from context and memory.

Test whether an explicit state/ledger of observations, changes, attempts, verifications and invalidations can reduce redundant actions and improve recovery without creating stale assumptions.

Compare:

```text
baseline history
+ transcript/context compression
+ explicit execution state
+ both
```

Measure task success, repeated actions, recovery, tokens, latency and stale-state failures.

## 8. Durable memory and procedural learning

Compare transcript memory, curated Markdown/files, structured records, graph memory and outcome-linked learning records.

Measure retrieval usefulness, stale-memory failures, contradiction, false promotion, maintenance cost, generalization and regression after model changes.

Specifically test whether a fresh session can reconstruct useful continuity without treating conversation compaction as the primary persistence mechanism.

## 9. Model × infrastructure experiments

Construct paired experiments:

```text
frontier model + baseline
frontier model + intervention

cheaper model + baseline
cheaper model + intervention
```

Measure whether infrastructure produces absolute gains, reduces cost, or narrows the capability gap between models.

Do not assume an intervention valuable to one model transfers to another.

## 10. Repository maturity and institutional context

Test repository classes separately:

```text
AI-native / well structured
        |
modern / mature
        |
large / heterogeneous
        |
legacy / poorly documented
```

Include organizational/system context where available: ownership, service boundaries, architectural rationale, incidents and historical decisions.

Measure repository discovery work, incorrect-but-plausible decisions, stale-context failures, context cost and task outcomes.

## 11. Knowledge representations

Compare representations such as OKF, reviewed Markdown, ADRs, project-local instructions and structured records.

Test whether a representation preserves useful knowledge more reliably than raw source exploration and whether it reduces acquisition cost without introducing stale or incorrect claims.

Keep the representation/runtime boundary explicit: a format such as OKF is not itself a knowledge server or control plane.

## 12. Deterministic evidence and analysis escalation

For software engineering, compare LLM-only reasoning against hybrid workflows using structural graphs, tests, type checking, static analysis and dataflow/taint analysis.

Test **minimum sufficient analysis**:

```text
type/API design
 -> lightweight rule
 -> AST/compiler analysis
 -> dataflow
 -> deep interprocedural analysis
```

Measure setup cost, runtime, coverage, false positives/negatives and usefulness to the agent. Do not assume the deepest analyzer is best.

## 13. Invariants and barriers

Test whether agents can identify durable architectural invariants from concrete failures and express them independently of a particular analyzer.

Use source → barrier → sink properties where structural traversal is insufficient. Compare prevention through types/API design with lightweight rules, targeted analysis and deeper analysis.

Measure whether an invariant prevents regressions, maintenance cost and validity as architecture evolves.

> **Invariants are a more durable abstraction than individual analyzer rules.**

## 14. Evidence-provider selection

Prototype a policy choosing among graph queries, retrieval, type checking, tests, static analyzers, runtime evidence and LLM reasoning.

The policy should seek the **cheapest provider sufficient for the engineering question**, not automatically the deepest available analyzer.

Record provider selection, evidence, revision/freshness, cost and final outcome so provider choice itself can be evaluated.

## 15. Orchestration and workflow topology

Test whether explicit orchestration improves outcomes over a single capable agent session.

Start with the smallest useful topology: executor → independent reviewer → verification, with a conductor recording state and deciding whether to retry, branch or escalate.

Compare sequential execution, independent review, parallel specialized workers and no orchestration. Measure success, defect escape, tokens, latency, coordination overhead and evidence quality.

The question is not whether more agents are better; it is whether explicit control and independent validation justify their cost.

**Planning should not be treated as a presumed bottleneck.** If a strong model already plans adequately, external planning infrastructure should have to demonstrate a measurable benefit.

## 16. Control plane and adaptive policies

Implement the smallest scheduler that accepts a task, chooses context/acquisition/evidence/model policies, executes, evaluates and records the decision.

Expose context selection, provider selection, execution-state updates and model routing as policy decisions rather than hiding them inside prompt/tool implementations.

Then test whether evaluation results can improve future policies: retrieve-more, stop, verify, branch, switch model, escalate to deeper analysis or request human input.

Measure decision utility, not only prediction accuracy.

## 17. Continuous assurance, reliability and governance

Test whether observability can become a sensor layer for EOKS rather than merely a debugging dashboard. Compare model self-assessment, model-computation signals where available, external evidence and deterministic outcomes.

Evaluate calibration and whether reliability signals reduce bad actions, unnecessary retries and unnecessary expensive analysis.

Separately test access control, secret/PII filtering, retention, deletion, rollback, stale-source invalidation and conflicting-knowledge handling across memory, context and evidence providers.

## 18. Minimum useful semantic model

Instrument the vertical slice and determine which concepts are actually required in traces. Validate whether `Task`, `Context`, `Run`, `Decision`, `Policy`, `Evaluation` and `Outcome` are sufficient EOKS primitives, and whether Asset/Provider/Representation/Loadout should remain vocabulary-level concepts rather than runtime entities.

Avoid expanding the ontology until a concrete workload demonstrates a missing primitive or relationship.

## 19. Benchmark methodology

For context and agent infrastructure experiments, use the canonical methodology in [Context evaluation and benchmarking](../research/context-evaluation.md).

At minimum report:

**Outcome** — correctness, completeness, regressions, evidence quality.

**Efficiency** — model tokens, tool calls, latency, total cost.

**Context health** — relevance, coverage, redundancy, contradiction, freshness, context growth.

**Acquisition** — discovery work, repeated searches, missed relevant evidence.

**Autonomy** — retries, recovery, session resets and successful completion without intervention.

Always compare against the strongest practical baseline and report the experimental conditions.

## 20. What EOKS should not assume

The research agenda deliberately does **not** assume that:

- graphs are better than grep/read exploration;
- retrieval is better than agentic search;
- more context is better;
- context compression preserves everything important;
- durable memory is better than re-discovery;
- sub-agents are better than one capable agent;
- stronger infrastructure makes a cheaper model equivalent to a frontier model;
- planning is a bottleneck for every capable model;
- a popular project is effective outside its demonstrated workload.

These are hypotheses to test.

The central EOKS objective is:

> **Discover which capabilities improve trustworthy software-agent outcomes, under which conditions, and whether their benefit justifies their complexity and cost.**
