# Repository context acquisition: FastContext, GrapeRoot and alternatives

This note captures a specific research gap identified from the 2026 evidence review: several approaches attempt to reduce the cost of repository exploration, but they operate at different layers and have not yet been compared in a common, independent evaluation.

This is **research evidence, not an architectural recommendation**.

## 1. The problem

A coding agent needs to acquire enough repository knowledge to solve a task. Current approaches differ in where that work happens:

```text
                 Repository understanding
                         |
        +----------------+----------------+
        |                |                |
     raw search       retrieval        structure
        |                |                |
   grep/read/glob   semantic/lexical    graphs/maps
        |                |                |
        +----------------+----------------+
                         |
                dedicated explorer
                         |
                    FastContext
                         |
                    main solver
```

The key EOKS question is not whether exploration is wasteful. Some exploration may be the model's mechanism for constructing semantic understanding. The question is:

> Can an intervention remove avoidable discovery work while preserving or improving the evidence the solver needs?

## 2. FastContext

FastContext is a dedicated repository-exploration subagent. The solver delegates a natural-language context query; the explorer uses read-only `Read`, `Glob` and `Grep`, including parallel tool calls, and returns compact file/line citations rather than its full exploration trajectory.

The 2026 paper reports, across SWE-bench Multilingual, SWE-bench Pro and SWE-QA, up to **+5.5 percentage points** end-to-end resolution and up to **60.3% lower main-agent token consumption** in its Mini-SWE-Agent experiments. The released exploration models span 4B–30B parameters, with a 4B RL model among the reported configurations.

Source: Zhang et al., *FastContext: Training Efficient Repository Explorer for Coding Agents*, arXiv:2606.14066.

Important status correction: the original Microsoft repository/research artifact has had availability changes, but the paper, code and multiple public forks/mirrors remain accessible. EOKS should therefore classify FastContext as **research/implementation with availability and provenance risk**, not as simply discontinued.

Official research repository: https://github.com/evalstate/fastcontext
Paper: https://arxiv.org/abs/2606.14066

### What FastContext actually tests

FastContext is primarily a test of:

- **exploration/solving separation**;
- specialized low-cost repository exploration;
- parallel low-level search;
- compact evidence handoff;
- avoiding pollution of the solver trajectory.

It does **not** establish that graphs or retrieval are unnecessary. Its explorer itself uses low-level repository search.

## 3. GrapeRoot

GrapeRoot represents a different hypothesis: precompute and maintain a structural representation of the repository, then use that representation to guide context selection. Its public material describes a code map, context packing and session/action state, including cross-service dependency awareness.

A May 2026 project-authored benchmark reports **110 prompts** over a **34-service monorepo spanning 12 languages**, using Claude Sonnet 4.6:

| System | Mean quality | Avg cost/prompt | Prompts below Q=80 |
|---|---:|---:|---:|
| GrapeRoot | **87.12** | **$0.71** | **0** |
| Sourcebot | 84.71 | $0.87 | 24 |
| Vanilla Claude | 85.21 | $0.94 | 19 |

The same report gives GrapeRoot 36 wins vs Sourcebot and 29 vs vanilla, with open benchmark data claimed by the project.

Source: https://graperoot.dev/benchmarks/agentic-v1

**Evidence classification: vendor/project-authored benchmark.** The result is highly relevant and should be reproduced independently before being treated as a general effect size.

## 4. Why these are not direct competitors

They attack different costs:

| Approach | Where knowledge lives | Acquisition mechanism | Main hypothesis |
|---|---|---|---|
| Raw agent | solver trajectory | grep/read/glob/etc. | exploration itself is useful reasoning |
| Semantic retrieval | repository index | retrieve/rerank | pre-indexing reduces search cost |
| RepoMap | structural summary | map-guided selection | compact structure improves context yield |
| Graph | persistent structure | graph traversal/query | explicit relations reduce rediscovery |
| FastContext | explorer trajectory | delegated search | separate exploration protects solver context |
| GrapeRoot | persistent repository/session representation | code map + packing + session state | persistent structure reduces repeated exploration |

This distinction matters: **FastContext and GrapeRoot can plausibly be complementary rather than mutually exclusive.** For example:

```text
repository
    |
    v
graph / semantic index
    |
    v
candidate regions
    |
    v
specialized explorer
    |
    v
compact evidence
    |
    v
main solver
```

That hybrid is an EOKS hypothesis, not a recommendation.

## 5. What has actually been compared?

As of the 2026-08 review, there is **no credible common benchmark found that directly compares FastContext and GrapeRoot** under the same repository, tasks, model, budget and judge.

There is useful adjacent evidence:

- FastContext has end-to-end and standalone exploration evaluations.
- GrapeRoot has a 110-prompt project-authored comparison against Sourcebot and vanilla Claude.
- Agent Retrieval Bench compares lexical retrieval, embeddings, RepoMap and logged agent selection over **427 samples / 25 repositories / 392k files / 7.9M chunks**, finding no universally dominant retrieval family.
- A repository-QA study compares semantic retrieval with deep agentic/grep-search subagents and reports **65.2% vs 46.2%** answer accuracy, with correct semantic-search answers costing less than half as much in that workload. This is read-only repository QA, so it should not be generalized directly to autonomous editing.

Sources:
- https://arxiv.org/abs/2607.24882
- https://arxiv.org/abs/2608.01507

## 6. Important counter-evidence

The repository-QA result above is particularly useful because it challenges a currently popular pattern: delegating repository search to another agent may protect the main context but introduce a **planner → explorer handoff failure mode**. The study attributes **41.8% of deep-agentic-search failures** to that handoff in its taxonomy.

Conversely, long-context processing research indicates that ordinary filesystem/tool use can be an effective way for coding agents to process large external corpora.

Therefore EOKS should evaluate at least:

1. raw exploration;
2. semantic retrieval;
3. structural map/graph;
4. dedicated exploration;
5. hybrids.

## 7. Proposed common benchmark

The missing experiment should use the **same**:

- repository snapshots;
- task set;
- base solver model;
- effort/reasoning setting;
- context/tool budget;
- success judge;
- accounting rules.

Compare:

```text
A. raw grep/read/glob
B. semantic retrieval
C. RepoMap / structural summary
D. graph-based context
E. FastContext-style explorer
F. GrapeRoot-style persistent structure
G. graph/retrieval -> explorer hybrid
```

Use at least three repository regimes:

```text
small / clean
large / modern
large / legacy / heterogeneous
```

And at least four workload types:

- repository question answering;
- targeted bug fixing;
- cross-component change;
- architecture/refactoring task.

## 8. Metrics

Do not measure only final task success. Record:

### Outcome
- correctness;
- completeness;
- regression rate;
- evidence-grounded correctness.

### Acquisition
- relevant-file recall;
- irrelevant-file/context ratio;
- time to first useful evidence;
- exploration turns;
- duplicate searches;
- missed dependencies.

### Resource use
- solver input tokens;
- explorer/retriever/graph tokens;
- total tokens;
- cached input;
- latency;
- infrastructure/indexing cost.

### State quality
- repeated discovery;
- context growth;
- stale knowledge;
- handoff failures;
- recovery/restart rate.

The primary optimization target should be **total cost to a correct, verified result**, not solver-token reduction alone.

## 9. Research questions

1. Does separating exploration from solving help frontier models as much as smaller models?
2. Does persistent structure become more valuable as repository size and heterogeneity increase?
3. Does a graph improve acquisition efficiency because it supplies relationships, or merely because it supplies a compact summary?
4. Does FastContext-style exploration preserve semantic understanding better than aggressive retrieval?
5. Does GrapeRoot-style structure reduce repeated exploration over long sessions?
6. Does graph/retrieval seeding make a small explorer substantially cheaper or more accurate?
7. When does the infrastructure cost exceed the saved agent cost?
8. Does any intervention reduce the capability gap between a cheaper and frontier model?

## 10. Evidence status

| Claim | Current evidence | Confidence | EOKS action |
|---|---|---|---|
| Repository exploration is a measurable bottleneck | multiple academic benchmarks | **high** | benchmark |
| Separating exploration from solving can reduce solver tokens | FastContext | **medium-high** | reproduce |
| Persistent code structure can improve cost/quality | GrapeRoot self-reported benchmark + graph literature | **medium** | independent replication |
| Semantic retrieval can beat delegated search in some workloads | repository-QA study | **medium-high for that workload** | reproduce across editing tasks |
| Graphs universally beat raw exploration | insufficient evidence | **low** | do not assume |
| FastContext universally beats retrieval/graphs | no direct common benchmark | **low** | direct comparison |
| Graph + explorer is better than either alone | hypothesis only | **very low** | high-value experiment |

## 11. EOKS conclusion

The important discovery is not that one tool wins. It is that **repository context acquisition has become a measurable systems problem with multiple competing mechanisms and no established universal winner**.

FastContext and GrapeRoot should therefore be retained in EOKS as **distinct experimental archetypes**:

- FastContext = **delegated dynamic acquisition**;
- GrapeRoot = **persistent structural/context infrastructure**.

The missing evidence is a controlled, cross-model, cross-repository comparison that includes raw exploration as the baseline and accounts for the full infrastructure cost.
