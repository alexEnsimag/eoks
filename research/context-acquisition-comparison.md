# Repository context acquisition: FastContext, GrapeRoot, RTK and alternatives

This note captures a specific research gap identified from the 2026 evidence review: several approaches attempt to reduce the cost of repository exploration or tool-output context, but they operate at different layers and have not yet been compared in a common, independent evaluation.

This is **research evidence, not an architectural recommendation**.

## 1. The problem

A coding agent needs to acquire enough repository knowledge to solve a task. Current approaches differ in where that work happens and what they optimize:

```text
                         Agent + repository
                                |
          +---------------------+----------------------+
          |                     |                      |
     REPRESENTATION         ACQUISITION             STATE
     make outputs           find information        retain what
     cheaper/cleaner        more efficiently        is already known
          |                     |                      |
       RTK-like          retrieval / graphs /      ledger / memory /
       transforms        explorers / RepoMap       session state
          |                     |                      |
          +---------------------+----------------------+
                                |
                         main solver model
```

The key EOKS question is not whether exploration is wasteful. Some exploration may be the model's mechanism for constructing semantic understanding. The question is:

> Can an intervention remove avoidable work while preserving or improving the evidence and reasoning the solver needs?

## 2. A useful taxonomy: where repository understanding lives

```text
1. RECONSTRUCT
   Agent -> grep/read/search -> constructs understanding in its trajectory

2. DELEGATE
   Agent -> explorer -> compact evidence -> solver

3. PRECOMPUTE
   Repository -> persistent knowledge model -> relevant context -> solver

4. REPRESENT
   Agent -> tool -> output transformer -> smaller/cleaner output -> solver

5. HYBRID
   Repository knowledge -> candidate regions -> explorer -> evidence -> solver
```

This gives EOKS a more useful axis than simply calling all of these "context tools":

> **FastContext asks: who should do the exploration?**
>
> **GrapeRoot asks: what should the agent already know about the repository?**
>
> **RTK asks: can the same tool interaction be represented more cheaply?**

The hybrid category is deliberately a hypothesis: structural or semantic indexing could narrow the search space while an explorer performs semantic investigation, potentially preserving capabilities that would be lost by forcing all understanding through a graph.

### Intervention ladder

Prefer the least constraining intervention that addresses the measured bottleneck:

```text
LOW INTERVENTION / LOW ASSUMPTION

1. REPRESENTATION
   reduce tool-output cost without changing discovery

2. ACQUISITION
   help the agent find relevant information faster

3. DELEGATION
   move exploration outside the main solver trajectory

4. PERSISTENT KNOWLEDGE / STATE
   maintain repository or session understanding across turns

HIGHER INFRASTRUCTURE / HIGHER ASSUMPTION
```

This is a research ordering, not a claim that higher levels are worse. If a lower-level intervention cannot address the bottleneck, the higher-level intervention may have much greater leverage.

## 3. FastContext

FastContext is a dedicated repository-exploration subagent. The solver delegates a natural-language context query; the explorer uses read-only `Read`, `Glob` and `Grep`, including parallel tool calls, and returns compact file/line citations rather than its full exploration trajectory.

The 2026 paper reports, across SWE-bench Multilingual, SWE-bench Pro and SWE-QA, up to **+5.5 percentage points** end-to-end resolution and up to **60.3% lower main-agent token consumption** in its Mini-SWE-Agent experiments. The released exploration models span 4B–30B parameters, with a 4B RL model among the reported configurations.

Source: Zhang et al., *FastContext: Training Efficient Repository Explorer for Coding Agents*, arXiv:2606.14066.

FastContext should be classified as **research/implementation with availability and provenance risk**, rather than as simply discontinued. The research artifact's availability has changed over time; the paper and public implementations/forks remain discoverable.

Research repository: https://github.com/evalstate/fastcontext
Paper: https://arxiv.org/abs/2606.14066

### What FastContext actually tests

FastContext is primarily a test of:

- **exploration/solving separation**;
- specialized low-cost repository exploration;
- parallel low-level search;
- compact evidence handoff;
- avoiding pollution of the solver trajectory.

It does **not** establish that graphs or retrieval are unnecessary. Its explorer itself uses low-level repository search.

## 4. GrapeRoot

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

### What GrapeRoot actually tests

GrapeRoot is **not a model trained to understand context**. It is a context/knowledge system around an ordinary coding model. Its key hypothesis is that a persistent representation of repository structure and session state can make context selection cheaper and more reliable.

For EOKS terminology, it is best described as **persistent structural/context infrastructure** or a **dynamic context/knowledge model**, not a "dynamic model" in the machine-learning sense.

## 5. RTK: representation optimization, not repository understanding

RTK (Rust Token Killer) is a much smaller intervention. It proxies or rewrites supported command output so the model receives a more compact representation of the same tool interaction. It is therefore best classified as **tool-output representation optimization**, not retrieval, repository understanding, or a replacement for exploration.

The appeal is important for EOKS: the agent can still choose `git`, `grep`, tests, filesystem commands, etc. The intervention changes **what reaches the context**, not which repository facts the agent is allowed to discover.

Conceptually:

```text
agent -> git / test / find / grep -> RTK -> compact output -> agent
```

This makes RTK a useful **low-intervention baseline** for testing whether a bottleneck is simply verbose tool output rather than poor information acquisition.

### Evidence is contradictory and must be treated carefully

RTK's project documentation claims **60–90% token savings** for supported command outputs. However, independent 2026 evaluations report much smaller or even negative end-to-end effects:

- JetBrains' paired SkillsBench evaluation of 86 tasks with Claude Sonnet 5 reported **+7.6% median cost at low reasoning effort** (p=0.004), approximately **0% at high effort**, with task quality statistically indistinguishable between arms. Their analysis also found that only about a third of Bash calls were eligible for rewriting and that the affected output represented a small share of total billed input context.
- An independent `tokbench` pilot (N=1 per arm, replication in progress at the time of review) reported **higher provider-billed input tokens** for RTK in its particular harness, emphasizing that tool self-reported savings and provider-billed savings are different quantities.
- Other practitioner reports still claim substantial savings in Bash-heavy workflows, showing that workload/tool coverage matters.

Sources:

- https://blog.jetbrains.com/ai/2026/07/rtk-claude-code-token-savings/
- https://github.com/Entelligentsia/tokbench
- https://github.com/rtk-ai/rtk

**EOKS evidence classification:** the claim "RTK saves 60–90% of agent cost" is **not established**. The stronger, more defensible claim is that RTK can substantially compress **eligible command output**, while its end-to-end value depends on which tools the agent actually uses, how the harness bills cached/repeated context, and whether compression causes extra turns or re-reads.

### Why this matters conceptually

RTK is a useful control because it tests the least invasive hypothesis:

> **Can we reduce context cost without constraining model-driven exploration?**

If RTK-like transformations produce most of the benefit of a more complex context system on a workload, building a repository knowledge graph may be unnecessary there.

Conversely, if token savings are negligible because tool output is not the dominant cost, the bottleneck lies elsewhere.

## 6. Why these are not direct competitors

They attack different costs:

| Approach | Where knowledge lives | Acquisition mechanism | Main hypothesis |
|---|---|---|---|
| Raw agent | solver trajectory | grep/read/glob/etc. | exploration itself is useful reasoning |
| RTK-like | tool-output representation | transparent output rewriting | remove representational waste without changing discovery |
| Semantic retrieval | repository index | retrieve/rerank | pre-indexing reduces search cost |
| RepoMap | structural summary | map-guided selection | compact structure improves context yield |
| Graph | persistent structure | graph traversal/query | explicit relations reduce rediscovery |
| FastContext | explorer trajectory | delegated search | separate exploration protects solver context |
| GrapeRoot | persistent repository/session representation | code map + packing + session state | persistent structure reduces repeated exploration |

This distinction matters: **FastContext and GrapeRoot can plausibly be complementary rather than mutually exclusive**, and RTK-like representation optimization can potentially sit underneath either one.

For example:

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
RTK-like output reduction
    |
    v
compact evidence
    |
    v
main solver
```

That hybrid is an EOKS hypothesis, not a recommendation.

## 7. What has actually been compared?

As of the 2026-08 review, there is **no credible common benchmark found that directly compares FastContext and GrapeRoot** under the same repository, tasks, model, budget and judge.

There is useful adjacent evidence:

- FastContext has end-to-end and standalone exploration evaluations.
- GrapeRoot has a 110-prompt project-authored comparison against Sourcebot and vanilla Claude.
- Agent Retrieval Bench compares lexical retrieval, embeddings, RepoMap and logged agent selection over **427 samples / 25 repositories / 392k files / 7.9M chunks**, finding no universally dominant retrieval family.
- A repository-QA study compares semantic retrieval with deep agentic/grep-search subagents and reports **65.2% vs 46.2%** answer accuracy, with correct semantic-search answers costing less than half as much in that workload. This is read-only repository QA, so it should not be generalized directly to autonomous editing.
- A 2026 community benchmark directly compared **Serena, Graphify, CodeGraph, Archex, RTK and other token/context tools** on a small codebase. Its headline result showed large reductions for some structural tools, but the benchmark is community-authored, small and not a full autonomous SWE evaluation; it is best treated as exploratory evidence.

Sources:

- https://arxiv.org/abs/2607.24882
- https://arxiv.org/abs/2608.01507
- https://github.com/vagkaratzas/token-consumption-benchmark

## 8. Important counter-evidence

The repository-QA result above is particularly useful because it challenges a currently popular pattern: delegating repository search to another agent may protect the main context but introduce a **planner -> explorer handoff failure mode**. The study attributes **41.8% of deep-agentic-search failures** to that handoff in its taxonomy.

Conversely, long-context processing research indicates that ordinary filesystem/tool use can be an effective way for coding agents to process large external corpora.

RTK provides another counterpoint: even a very effective command-output compressor may have little or negative end-to-end value when the agent's billed context is dominated by other inputs, repeated cached reads, or uncovered tools.

Therefore EOKS should evaluate at least:

1. raw exploration;
2. representation optimization;
3. semantic retrieval;
4. structural map/graph;
5. dedicated exploration;
6. hybrids.

## 9. Proposed common benchmark

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
B. RTK-like output transformation
C. semantic retrieval
D. RepoMap / structural summary
E. graph-based context
F. FastContext-style explorer
G. GrapeRoot-style persistent structure
H. graph/retrieval -> explorer hybrid
I. combinations where representation optimization is layered underneath
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

## 10. Metrics

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

For representation optimizers specifically, also measure:

- fraction of tool calls affected;
- raw vs transformed output size;
- provider-billed input before/after;
- re-read/retry rate induced by transformation;
- commands/tool classes not covered.

## 11. Research questions

1. Does separating exploration from solving help frontier models as much as smaller models?
2. Does persistent structure become more valuable as repository size and heterogeneity increase?
3. Does a graph improve acquisition efficiency because it supplies relationships, or merely because it supplies a compact summary?
4. Does FastContext-style exploration preserve semantic understanding better than aggressive retrieval?
5. Does GrapeRoot-style structure reduce repeated exploration over long sessions?
6. Does graph/retrieval seeding make a small explorer substantially cheaper or more accurate?
7. When does the infrastructure cost exceed the saved agent cost?
8. Does any intervention reduce the capability gap between a cheaper and frontier model?
9. How much of the observed cost problem can be solved by **representation optimization alone**?
10. Does representation optimization remain beneficial when the solver already uses native structured tools rather than shell commands?
11. Do graph/retrieval systems and explorers become more valuable on legacy repositories where structural information is incomplete or inconsistent?

## 12. Evidence status

| Claim | Current evidence | Confidence | EOKS action |
|---|---|---|---|
| Repository exploration is a measurable bottleneck | multiple academic benchmarks | **high** | benchmark |
| Separating exploration from solving can reduce solver tokens | FastContext | **medium-high** | reproduce |
| Persistent code structure can improve cost/quality | GrapeRoot self-reported benchmark + graph literature | **medium** | independent replication |
| Semantic retrieval can beat delegated search in some workloads | repository-QA study | **medium-high for that workload** | reproduce across editing tasks |
| Tool-output rewriting can compress eligible outputs | RTK implementation + command-level tests | **high at command-output level** | measure end-to-end |
| RTK universally reduces agent cost | contradictory independent evidence | **low** | do not assume |
| Graphs universally beat raw exploration | insufficient evidence | **low** | do not assume |
| FastContext universally beats retrieval/graphs | no direct common benchmark | **low** | direct comparison |
| Graph + explorer is better than either alone | hypothesis only | **very low** | high-value experiment |

## 13. EOKS conclusion

The important discovery is not that one tool wins. It is that **repository context acquisition and representation have become measurable systems problems with multiple competing mechanisms and no established universal winner**.

FastContext, GrapeRoot and RTK should therefore be retained in EOKS as **distinct experimental archetypes**:

- FastContext = **delegated dynamic acquisition**;
- GrapeRoot = **persistent structural/context infrastructure**;
- RTK = **low-intervention tool-output representation optimization**.

The missing evidence is a controlled, cross-model, cross-repository comparison that includes raw exploration as the baseline, accounts for full infrastructure cost, and distinguishes **saving tokens from saving the work needed to reach a correct result**.
