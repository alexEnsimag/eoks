# Research agenda

The agenda should prioritize experiments that can falsify the EOKS control-plane hypothesis, not additional architectural vocabulary.

A central rule is:

> **Treat every infrastructure capability as an intervention whose value depends on the model, workload, repository, budget and baseline.**

The goal is not to prove that graphs, retrieval, memory, context management, computer-systems techniques or multi-agent workflows are good. The goal is to discover **what improves engineering outcomes, when, by how much, at what cost, and with which failure modes**.

The evidence intake that currently informs prioritization is [Community and academic evidence on agent bottlenecks](../research/community-evidence-bottlenecks.md). It records effect sizes, experimental settings, community failure reports, contradictory evidence and next tests. It is deliberately non-normative.

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

Record contradictory results rather than selecting only supporting evidence. For quantitative claims, preserve the workload, model, sample size, metric, effect size and uncertainty/limitations rather than copying a headline percentage.

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

For context and agent infrastructure experiments, use the canonical methodology in [Context evaluation and benchmarking](../research/context-evaluation.md) and maintain the [community/academic evidence ledger](../research/community-bottlenecks.md).

At minimum report:

**Outcome** — correctness, completeness, regressions, evidence quality.

**Efficiency** — model tokens, tool calls, latency, total cost.

**Context health** — relevance, coverage, redundancy, contradiction, freshness, context growth.

**Acquisition** — discovery work, repeated searches, missed relevant evidence.

**Autonomy** — retries, recovery, session resets and successful completion without intervention.

**Cost distribution** — median/P90/P95 where practical, variance, success-conditioned cost and tail/retry cost rather than only averages.

Always compare against the strongest practical baseline and report the experimental conditions.

## 20. Evidence-driven bottleneck map

Current evidence suggests that EOKS should not treat “context” as one bottleneck. Track at least:

```text
1. Task specification
       |
2. Repository/context acquisition
       |
3. Working-context quality / attention allocation
       |
4. Reasoning / diagnosis
       |
5. Tool selection and execution
       |
6. Long-horizon execution state / recovery
       |
7. Integration / completeness
       |
8. Verification / evidence
       |
9. Cost / latency / predictability
```

Cross-cutting variables include model capability, repository maturity, task ambiguity, agent topology, context budget and infrastructure complexity.

The bottleneck map is a **research hypothesis**. EOKS should update it as controlled experiments reveal which stage actually limits a workload.

## 21. Computer-systems optimization transfer

Operating systems and computer architecture provide a particularly rich source of established optimization techniques. EOKS should systematically test whether these techniques transfer to probabilistic reasoning workloads rather than rediscovering them ad hoc.

The canonical mapping and detailed rationale live in [OS and computer-architecture lens](../research/prior-art/computer-systems-architecture.md).

### Working-set management

Test whether explicit working-set estimation improves context quality/cost over fixed context budgets. Measure working-set hit/miss rates, context churn, task progress and total cost.

### Locality

Test temporal, structural, semantic and workflow locality. Determine whether access history predicts future evidence needs well enough to support retention, clustering or prefetching.

### Demand retrieval and context misses

Treat absent required evidence as a measurable context miss. Test whether miss-driven retrieval reduces upfront context cost without causing excessive latency or thrashing.

### Prefetching

Compare reactive retrieval against proactive prediction. Include the cost of false-positive prefetches; unlike hardware cache prefetch, evidence acquisition can require expensive model/tool calls.

### Admission and replacement

Compare simple baselines such as LRU/LFU with relevance-, authority-, freshness-, dependency- and retrieval-cost-aware policies. Establish simple baselines before learned policies.

### Pinning

Test whether pinning task objectives, safety/policy constraints, active invariants and critical evidence prevents harmful eviction without causing resident-context pollution.

### Evidence clustering and I/O optimization

Test batching/coalescing of structurally or semantically related retrievals, asynchronous acquisition and ordering for both latency and reasoning coherence.

### Compression and representation demotion

Test whether less-active evidence can be represented as summaries, structural slices or pointers while retaining enough authority/fidelity to recover the source when needed.

### Context thrashing

Detect repeated acquisition/eviction/re-expansion cycles and test control responses such as changing budget, pinning, representation, retrieval policy or task decomposition.

### Scheduling

Test priority, fairness, aging, work stealing, load balancing and deterministic-first scheduling across reasoning, retrieval, verification, testing, review and maintenance. Compare expensive probabilistic execution against the cheapest deterministic capability that can satisfy the requirement.

### Resource protection and shared state

Test workload-scoped resource namespaces, quotas, versioned state, invalidation and copy-on-write-like derived state. Measure stale-state failures and governance overhead rather than assuming one consistency model.

### Navigation-resolution caching

Test caching of logical-resource/provider resolution separately from caching the evidence itself. This may reduce repeated navigation work without retaining large evidence blocks.

### Async/event-driven maintenance

Test whether resource maintenance and reconciliation can be triggered by meaningful events rather than continuous polling, while preserving freshness and recovery guarantees.

## 22. Systems optimization benchmark matrix

The systems-optimization experiments should use the existing EOKS benchmark dimensions:

```text
model × repository × task × intervention × budget
```

Add systems-specific measurements:

- working-set size and composition;
- context miss rate;
- prefetch precision and recall where measurable;
- admission/eviction counts;
- resident critical evidence;
- context churn;
- repeated retrievals;
- navigation-resolution cache hits;
- acquisition batching/coalescing;
- representation promotion/demotion;
- thrashing indicators;
- scheduling queue time and modality choice;
- deterministic execution ratio and amortization.

Always evaluate against end-to-end correctness, completeness, verification and cost. A better cache statistic is not a successful EOKS result if the engineering outcome does not improve.

## 23. Analogy boundaries and inference cache

Keep the following distinctions explicit in experiments:

- semantic/context cache vs model-serving KV cache;
- working set vs context window;
- durable knowledge vs resident evidence;
- provider resolution vs evidence retrieval;
- deterministic capability vs agent role;
- workload control loop vs execution mechanism.

The OS/Kubernetes analogies are useful because they expose known optimization and control techniques. They should not become assumptions that EOKS must reproduce a Unix process model, fixed memory tiers, a particular cache algorithm or a particular Kubernetes topology.

## 24. What EOKS should not assume

The research agenda deliberately does **not** assume that:

- graphs are better than grep/read exploration;
- retrieval is better than agentic search;
- more context is better;
- context compression preserves everything important;
- durable memory is better than re-discovery;
- sub-agents are better than one capable agent;
- stronger infrastructure makes a cheaper model equivalent to a frontier model;
- planning is a bottleneck for every capable model;
- a popular project is effective outside its demonstrated workload;
- a reported token reduction is a productivity improvement unless task quality is preserved;
- benchmark Pass@1 alone measures deployment usefulness;
- classical cache policies will automatically transfer to semantic context;
- proactive context is automatically better than reactive exploration;
- lower context size is automatically better than a larger useful working set;
- Kubernetes or OS architecture should be copied literally.

These are hypotheses to test.

The central EOKS objective is:

> **Discover which capabilities improve trustworthy software-agent outcomes, under which conditions, and whether their benefit justifies their complexity and cost.**
