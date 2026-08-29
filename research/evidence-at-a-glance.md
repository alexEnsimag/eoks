# Evidence at a glance

> **Purpose:** give a human-readable snapshot of what the current evidence says about AI software-engineering bottlenecks. This is a **research compass, not a set of conclusions**.

## The signal in one page

| Area | Evidence signal | Representative numbers | Current read | EOKS question |
|---|---|---:|---|---|
| **Agent cost** | 🟢 Strong | ~**1000×** more tokens than code-chat/reasoning in one 8-model SWE-bench study; same-task runs up to **30×** apart | Cost is huge, variable and poorly predictable | Can infrastructure reduce tail cost without reducing success? |
| **Repository exploration** | 🟢 Strong | **848 issues / 203 repos**; agents miss all gold files in **27–35%** of retrieval samples | Finding relevant code is a measurable bottleneck | What beats raw exploration, and when? |
| **Retrieval / indexing** | 🟡 Mixed-positive | FastContext: up to **60%** lower tokens / **+5.5 pp** resolution; no retrieval family dominates | Promising, highly task-dependent | Graph vs RepoMap vs semantic vs raw exploration? |
| **Raw tool exploration** | 🟢 Important counter-signal | Large-context processing can work through ordinary filesystem/tool use | `grep`/`read` is not automatically waste | Which exploration is useful reasoning vs avoidable work? |
| **Context lifecycle** | 🟢 Strong signal | Context-management and execution-state papers report meaningful gains | Long trajectories need more than a larger window | Compression, selection, offloading or state? |
| **Execution state** | 🟢 Strong | Ledger: **+8.0 pp** GPT-5 mini, **+5.2 pp** MiniMax; Codex **+3.4 pp / −24.4% cost** in reported setups | Explicit state is more than transcript summarization | What state is sufficient, durable and safe? |
| **Task specification** | 🟢 Strong emerging | **2,700 runs**; simplifying specs increased tokens **29.7%**; sensitivity **13–115%** | The task itself is part of context engineering | Can better task contracts reduce exploration/retries? |
| **Repository instructions** | 🔴 Counter-evidence | `AGENTS.md`: **>20%** higher inference cost with no task-success gain in one controlled study | More context can hurt | What is the minimum useful instruction set? |
| **Experience / memory** | 🟡 Conditional | **300 + 99** task experience-reuse evaluation | Correct selection helps; irrelevant memory can hurt | How do we select, invalidate and verify memory? |
| **Subagents** | 🟡 Mixed / costly | Community report: **~2.76M** subagent tokens in one runaway topology | Coordination can dominate | When does parallelism beat one strong agent? |
| **Long-horizon SWE** | 🔴 Still hard | SWE-bench Pro reported **<25%** Pass@1 under its scaffold; newer science/refactor tasks remain difficult | Coding agents are far from universally reliable | Which missing capabilities matter most? |

**Legend:** 🟢 repeated/controlled evidence; 🟡 promising or conditional evidence; 🔴 evidence of a serious unresolved problem or counter-result. The colors describe the **research signal**, not whether a proposed solution is good.

## What the community keeps reporting

> **Qualitative signal only — not prevalence statistics.**

```text
██████████  context pollution / oversized transcripts
█████████   repeated repository rediscovery
████████    compaction / continuity loss
████████    expensive grep/read exploration
███████     unpredictable token consumption
███████     subagent cost / duplicate work
██████      missing structural / IDE-like navigation
██████      long-session degradation / manual restart
█████       desire for durable, inspectable progress state
```

The bars are **relative visual emphasis from recurring discussions, not measured frequencies**. They should never be interpreted as survey results.

## Where the evidence agrees

```text
                         ┌──────────────────────┐
                         │   AI coding agents   │
                         └──────────┬───────────┘
                                    │
            ┌───────────────────────┼───────────────────────┐
            ▼                       ▼                       ▼
     acquisition is costly   context can degrade     long tasks fail
            │                       │                       │
            ▼                       ▼                       ▼
       retrieval /             lifecycle /             state /
       exploration             selection              recovery
            │                       │                       │
            └───────────────────────┼───────────────────────┘
                                    ▼
                           **measure the trade-off**
                                    │
                     correctness × cost × autonomy
```

## Where the evidence conflicts

| Question | Evidence on one side | Counter-evidence | What we should do |
|---|---|---|---|
| **Should we replace grep/read?** | Retrieval/explorer papers report large efficiency gains | Raw filesystem/tool use can be effective long-context processing | Keep raw exploration as the baseline |
| **Should we add more context?** | Better seed context can reduce exploration | `AGENTS.md` can raise cost without improving success; long-context positional effects persist | Optimize **selection**, not volume |
| **Are graphs best?** | Graph/RepoMap systems show structural-navigation benefits | Retrieval benchmark has no universal winner | Benchmark graph as one intervention |
| **Are subagents better?** | Parallel/specialized agents can improve difficult tasks | Coordination and duplicate-work costs can explode | Compare against one strong agent |
| **Is memory always useful?** | Correctly selected experience can help | Irrelevant experience can hurt | Measure selection + staleness |
| **Can infrastructure substitute for model capability?** | Several systems improve weaker/cheaper agents | Stronger models can make scaffolding redundant | Test model × infrastructure interactions |

## The bottleneck map

```text
1  Task specification
        ↓
2  Knowledge / context acquisition
        ↓
3  Working-context quality & attention allocation
        ↓
4  Reasoning / diagnosis
        ↓
5  Tool selection & execution
        ↓
6  Long-horizon state & recovery
        ↓
7  Integration & completeness
        ↓
8  Verification & evidence
        ↓
9  Cost / latency / predictability
```

Cross-cutting variables:

**model capability · repository maturity · institutional knowledge · task ambiguity · agent topology · context budget · infrastructure complexity**

## What this directs us to test first

### Highest-priority experiments

1. **Raw exploration vs specialized context acquisition** — same model, task and budget; measure evidence coverage as well as tokens.
2. **Context compression vs execution state** — distinguish "remember the transcript" from "remember what is established and still valid".
3. **Model × infrastructure** — test whether infrastructure gives cheaper models disproportionate gains.
4. **Modern × legacy repositories** — don't assume results transfer between clean AI-native code and undocumented heterogeneous systems.
5. **Single agent × subagents** — measure marginal correctness per coordination token.
6. **Context amount × context quality** — test whether more information helps, hurts or merely changes exploration behavior.

## Evidence ladder

```text
COMMUNITY PAIN
     │  recurring reports / project adoption
     ▼
COMMUNITY BENCHMARK
     │  useful but often single-author / uncontrolled
     ▼
ACADEMIC RESULT
     │  controlled workload + explicit methodology
     ▼
INDEPENDENT REPLICATION
     │
     ▼
CROSS-MODEL / CROSS-REPOSITORY
     │
     ▼
EOKS WORKLOAD EVIDENCE
     │
     ▼
PROJECT DECISION
```

**Rule:** popularity tells us **what deserves investigation**. It does not tell us **what works**.

## Key sources

- Bai et al. — *How Do AI Agents Spend Your Money?* — https://arxiv.org/abs/2604.22750
- *SWE-Explore* — https://arxiv.org/abs/2606.07297
- *Agent Retrieval Bench* — https://arxiv.org/abs/2607.24882
- *FastContext* — https://arxiv.org/abs/2606.14066
- *Coding Agents are Effective Long-Context Processors* — https://arxiv.org/abs/2603.20432
- *Ledger: Turning Interaction History into Execution State* — https://arxiv.org/abs/2608.00808
- *Context as a Tool* — https://aclanthology.org/2026.findings-acl.1032/
- *SWE-bench Pro* — https://arxiv.org/abs/2509.16941
- *SWE-bench Science* — https://arxiv.org/abs/2608.19799
- SRI/ICLR `AGENTS.md` study — https://www.sri.inf.ethz.ch/publications/gloaguen2026agentsmd

For detailed evidence, caveats and practitioner links, see [`community-evidence-bottlenecks.md`](community-evidence-bottlenecks.md).
