# Community and academic evidence on agent bottlenecks

This note is an evidence intake and prioritization aid for EOKS. It is deliberately **not** a list of conclusions. Numbers below are evidence for particular workloads, models or community samples; they should not be generalized without replication.

## 1. Why this belongs in EOKS

EOKS should not choose infrastructure because it is popular or intuitively appealing. Community reports are nevertheless valuable because they expose failure modes that benchmarks may not measure, while academic work provides controlled measurements and falsifiable hypotheses.

Use the evidence in this note to decide **what to benchmark next**, not what architecture to adopt.

Recommended confidence progression:

```text
recurring practitioner pain
        -> reproducible community benchmark
        -> controlled academic result
        -> independent replication
        -> cross-model / cross-repository result
        -> EOKS workload-specific decision
```

Community sentiment is useful as a direction signal. Reddit vote counts, individual benchmarks and project popularity are **not representative surveys** and should never be treated as prevalence estimates.

## 2. The strongest quantitative signal: token consumption is huge, variable and poorly predicted

Microsoft Research's April 2026 study analyzed trajectories from **eight frontier LLMs on SWE-bench Verified**. It reports:

- agentic coding tasks consumed roughly **1000× more tokens** than code reasoning/code-chat tasks in their comparison;
- **input tokens**, rather than output tokens, drove most consumption;
- runs on the same task could differ by **up to 30×** in total tokens;
- higher token usage did **not** reliably mean higher accuracy; accuracy often peaked at intermediate cost and saturated;
- model token efficiency differed substantially, with Kimi-K2 and Claude-Sonnet-4.5 averaging **more than 1.5M additional tokens** versus GPT-5 on the same tasks in their setup;
- models' own token-cost predictions were weak (reported correlations up to **0.39**) and systematically underestimated actual costs.

Source: Bai et al., *How Do AI Agents Spend Your Money? Analyzing and Predicting Token Consumption in Agentic Coding Tasks* (2026): https://arxiv.org/abs/2604.22750

### EOKS implication

Cost should not be represented by a single average token number. Measure **distribution, variance, tail cost and cost conditional on success**. An intervention that reduces average tokens but increases high-cost failures may be worse operationally.

This also makes **token efficiency a model × task × workflow property**, not a static model property.

## 3. Repository exploration is now a separately benchmarked bottleneck

### SWE-Explore

*SWE-Explore: Benchmarking How Coding Agents Explore Repositories* (2026) isolates repository exploration rather than treating coding as a binary pass/fail task. It contains **848 issues, 10 languages and 203 open-source repositories** and evaluates coverage, ranking and context efficiency. The authors report that these exploration metrics strongly track downstream repair behavior and identify **line-level coverage and efficient ranking** as key differentiators.

Source: https://arxiv.org/abs/2606.07297

### Agent Retrieval Bench

*Agent Retrieval Bench* (2026) evaluates **427 samples across 25 repositories**, covering **392,000 files and 7.9M chunks**. It compares lexical retrieval, embeddings, RepoMap and logged agent context selection.

Important results:

- no single retrieval family dominates;
- RepoMap gives the best budgeted context yield at 8K tokens in their setting;
- different tasks have different winners;
- logged agents miss every gold file on **27–35%** of samples;
- retrieval-derived seed context produced better file F1 with less subsequent exploration than random non-gold seed context in a controlled pilot.

Source: https://arxiv.org/abs/2607.24882

### FastContext

*FastContext: Training Efficient Repository Explorer for Coding Agents* (2026) explicitly identifies repository exploration as a token/context bottleneck. A specialized 4B–30B exploration model returns focused file/line evidence to a solver. Across SWE-bench Multilingual, SWE-bench Pro and SWE-QA, the authors report **up to 5.5 percentage points** end-to-end resolution improvement and **up to 60% lower coding-agent token consumption** with marginal overhead in their setup.

Source: https://arxiv.org/abs/2606.14066

### EOKS implication

This is now strong enough to justify **context acquisition as a first-class benchmark dimension**. But the evidence does *not* establish that semantic retrieval, graphs or a separate explorer is universally superior to raw exploration.

The key experiment remains:

> Can infrastructure remove avoidable repository-discovery work without removing useful semantic investigation?

## 4. Raw exploration should remain a first-class baseline

A separate 2026 research line, *Coding Agents are Effective Long-Context Processors*, argues that coding agents can process very large external corpora effectively through ordinary executable tools and filesystem navigation. This is an important counterweight to retrieval-first thinking.

Source: https://arxiv.org/abs/2603.20432

Therefore EOKS must not classify `grep`, `read`, filesystem navigation or repeated inspection as automatically wasteful. Some of that work may be the mechanism through which the model constructs a semantic representation of the system.

The measurable question is whether an alternative obtains equivalent or better understanding with less resource consumption and without increasing omissions.

## 5. Context lifecycle and compaction are recurring practitioner complaints

Community discussions repeatedly report:

- automatic compaction occurring at inconvenient points in a task;
- loss of current state or previously established details after compaction;
- repeated rediscovery of already inspected or modified code;
- users manually restarting sessions to recover quality;
- maintaining handoff/progress files as a workaround;
- large contexts becoming harder to control even when the nominal context window is large.

Examples from recent Codex discussions include reports of repeated compaction causing state loss and rework, and users recommending explicit progress/ledger files as recovery state:

- https://www.reddit.com/r/codex/comments/1uy70sl/context_compactment_is_completely_broken/
- https://www.reddit.com/r/codex/comments/1v788d0/after_context_compaction_how_do_you_tell_codex_what_is_still_current/
- https://www.reddit.com/r/codex/comments/1vl0pl9/context_too_small/

These are **anecdotal community signals**, not prevalence measurements.

### Academic convergence

*Context as a Tool: Context Management for Long-Horizon SWE-Agents* studies active context management rather than passive transcript growth.

More importantly for EOKS, **Ledger: Turning Interaction History into Execution State** reports improvements on all 500 SWE-bench Verified instances in its evaluation, including:

- GPT-5 mini: **56.2 → 64.2 Pass@1**;
- MiniMax M2.5: **75.8 → 81.0**;
- Codex: **+3.4 percentage points** while reducing cost by **24.4%** in the reported setup;
- total cost reductions of roughly **29–32%** for two evaluated models.

Sources:

- https://aclanthology.org/2026.findings-acl.1032/
- https://arxiv.org/abs/2608.00808

### EOKS implication

**Execution state deserves to be evaluated separately from context compression.** The hypothesis is not simply "summarize the transcript"; it is that a compact, durable record of what was observed, changed, attempted, verified and invalidated may prevent redundant work and improve recovery.

## 6. Community reports identify a second cost center: tool-call exploration

Practitioners repeatedly describe agents spending many turns on:

- `grep`/`glob`/`find` for orientation;
- reading large files to discover relevance;
- repeated searches after partial understanding;
- rediscovering callers/callees that an IDE would expose directly;
- spawning expensive subagents for repository exploration.

Examples include:

- a Reddit benchmark claiming **~97% lower input tokens** on a 155K-line Excalidraw repository when comparing a local semantic-search tool with baseline exploration; this is a single author's five-task benchmark, not independently validated: https://www.reddit.com/r/ClaudeAI/comments/1qv0id3/open_source_i_reduced_claude_code_input_tokens_by/
- a 54-run Sonnet 4.6 experiment on a C# repository comparing baseline, structural tooling and architecture preloading: https://www.reddit.com/r/ClaudeAI/comments/1s27dex/i_tracked_exactly_where_claude_code_spends_its/
- a Codex report claiming roughly **50% token reduction** from command-output byte caps and more selective validation; again, this is a practitioner report rather than a controlled independent study: https://www.reddit.com/r/codex/comments/1t6iulo/i_cut_codex_token_usage_50_with_one_agentsmd_rule/

These reports are useful because the failure description is consistent across tools: **agents often use low-level shell exploration where a human IDE would use structural navigation**.

The reported percentage improvements should be treated as hypotheses to reproduce, not as established effect sizes.

## 7. Output size, input size and repeated work need separate accounting

Community discussion sometimes focuses on input-context size, but another recurring observation is that an agent can spend expensive output/reasoning tokens repeatedly rediscovering the repository or retrying a failed approach.

The academic token-consumption study above provides stronger evidence that input dominates aggregate token volume in its SWE-bench setup, while the practitioner reports highlight the behavioral source of that input: repeated tool outputs and re-reading.

Therefore EOKS should record at least:

```text
input tokens
cached input tokens
output tokens
reasoning tokens where available
tool-output tokens
compaction/reconstruction tokens
sub-agent tokens
```

Avoid optimizing only one component of the bill.

## 8. Task specification is another underappreciated cost lever

A very recent 2026 study ran **2,700 Kimi K3 coding-agent runs** at three thinking-effort levels. It reports that reducing a full task specification to a bare user story increased token spend by **29.7%**, while prompt sensitivity varied from **13% to 115% depending on the task**.

Source: https://arxiv.org/abs/2608.25399

### EOKS implication

Context optimization should not start at retrieval. **Task specification is itself context.** EOKS should measure whether clearer task contracts reduce exploration, retries and cost without over-constraining the agent.

This connects directly to subagent context contracts and to the distinction between useful constraints and context bloat.

## 9. Repository-level success remains far from solved

*SWE-Bench Pro* was designed around longer-horizon, more professional software-engineering tasks. Under its unified scaffold, reported performance remained below **25% Pass@1**, with GPT-5 at **23.3%** in the reported evaluation.

Source: https://arxiv.org/abs/2509.16941

A newer *SWE-bench Science* benchmark contains **119 tasks across 98 repositories and 20 scientific domains**. Even Claude Code with Opus-5 (max) remained below **50% Pass@1**. The authors identify recurring failures involving:

- missing scientific knowledge/abstraction;
- misguided exploration or surface-level repair;
- incomplete repair/integration;
- failure to generalize beyond observed cases.

They also find that external guidance can help when well grounded but can cause anchoring when poorly aligned.

Source: https://arxiv.org/abs/2608.19799

### EOKS implication

There is substantial room beyond ordinary bug-fix benchmarks. **Knowledge gaps, exploration quality, integration completeness and domain/context grounding** should remain research dimensions.

## 10. Agent trajectories fail for more than retrieval reasons

A 2025 empirical study manually analyzed **150 failed SWE-bench instances** and developed a taxonomy with **3 primary phases, 9 categories and 25 fine-grained failure modes**. It reports that many agentic failures stem from flawed reasoning and cognitive deadlocks and found that a supervisory expert/executor setup solved **22.2% of previously intractable issues** for its leading single-agent baseline.

Source: https://arxiv.org/abs/2509.13941

A separate trajectory benchmark shows that tool-use failures include tool confusion, parameter errors and ordering/dependency failures, with a bottleneck emerging as trajectories move from short to medium length.

Source: https://arxiv.org/abs/2510.04550

### EOKS implication

The bottleneck map must not collapse into "context retrieval". At minimum distinguish:

```text
problem specification
repository/context acquisition
reasoning / diagnosis
execution/tool use
integration / completeness
verification
long-horizon state/recovery
```

Infrastructure should be tested against the failure mode it is supposed to address.

## 11. Subagent explosion is a real community concern

Practitioners report large token spikes when subagents are allowed to spawn further subagents, particularly when all inherit high reasoning effort. One recent report describes a repository-analysis workflow consuming roughly **2.76M subagent tokens** after six heavy subagents spawned three more layers.

Source: https://www.reddit.com/r/ClaudeAI/comments/1vce5x3/be_careful_running_claude_code_subagents/

This is an anecdote, but it points to a concrete EOKS failure mode:

> **coordination cost can grow superlinearly with agent topology.**

The research agenda should therefore measure:

- number of agents spawned;
- depth/branching factor;
- duplicate work across agents;
- parent-context pollution;
- coordination tokens;
- marginal success improvement per additional agent.

The baseline should include a capable single agent.

## 12. Repository instructions can make things worse

A controlled SRI/ICLR 2026 study evaluated `AGENTS.md` across multiple agents and LLMs. It found **no improvement in task success rates while increasing inference cost by more than 20%**. The instructions encouraged broader exploration and more testing, and unnecessary requirements could make tasks harder.

Source: https://www.sri.inf.ethz.ch/publications/gloaguen2026agentsmd

### EOKS implication

This is a strong warning against a simplistic "add more repository knowledge" strategy. Knowledge/context should be **minimal, relevant, scoped and evidence-backed**. More instructions are an intervention with a measurable cost.

## 13. Retrieval and experience reuse can help—but selection quality matters

*SWE Context Bench* evaluates experience reuse over **300 base tasks plus 99 related tasks**. It reports that correctly selected summarized experience improves accuracy and reduces runtime/token cost, particularly on harder tasks, while unfiltered or incorrectly selected experience provides limited or negative benefits.

Source: https://arxiv.org/abs/2602.08316

This supports a general EOKS principle:

> **The value of memory/context depends more on selection quality than on the amount stored.**

It also motivates measuring stale, misleading and wrongly retrieved knowledge as first-class failure modes.

## 14. What the community appears to be converging on

This is a qualitative synthesis of recurring practitioner reports, not a survey with population-level statistics.

### Strong recurring pain signals

1. **Context gets polluted by repository exploration.**
2. **Agents repeat discovery work.**
3. **Compaction can destroy continuity.**
4. **Long sessions become less reliable or require manual restart/handoff.**
5. **Subagents can multiply cost and duplicate work.**
6. **Structural/IDE-like navigation is missing from low-level agent tool use.**
7. **Token consumption is difficult to predict and control.**
8. **Large context windows do not guarantee stable long-horizon behavior.**
9. **Clear task specifications and durable progress state can materially change behavior.**
10. **Users increasingly want persistent, inspectable state rather than a giant transcript.**

### Less settled / conflicting areas

- graph vs semantic retrieval vs raw exploration;
- proactive context vs just-in-time retrieval;
- summaries vs structured execution state;
- subagents vs one strong model;
- more repository instructions vs minimal instructions;
- larger context windows vs better context selection;
- expensive frontier models vs cheaper models plus infrastructure.

These are exactly the areas where EOKS should prioritize controlled comparisons rather than follow community consensus.

## 15. Bottleneck map for EOKS

The evidence suggests expanding the bottleneck map to:

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

Cross-cutting dimensions:

- model capability;
- repository maturity and institutional knowledge;
- task ambiguity;
- agent topology;
- context budget;
- infrastructure complexity.

This is more useful than assuming a single "context bottleneck".

## 16. Highest-value experiments suggested by the evidence

Prioritize experiments where there is both recurring pain and measurable academic support:

### A. Raw exploration vs specialized acquisition

Measure whether semantic retrieval, RepoMap, graphs, LSP or a dedicated explorer reduce cost **without reducing evidence coverage**.

### B. Context compression vs explicit execution state

Measure whether a ledger/state representation improves continuity more reliably than summarization alone.

### C. Context acquisition × model capability

Determine whether weaker/cheaper models benefit more from structural or retrieval infrastructure than frontier models.

### D. Context acquisition × repository maturity

Compare AI-native/clean repositories with large legacy/heterogeneous systems.

### E. Single agent vs subagent topology

Measure marginal benefit versus coordination and context cost, including bounded spawning.

### F. Task specification quality

Measure whether task contracts reduce exploration and retries, and whether overly prescriptive instructions cause harm.

### G. Long-horizon endurance

Measure useful work before degradation, compaction, recovery or restart—not merely final Pass@1.

### H. Cost distribution and predictability

Report median, P90/P95, variance, success-conditioned cost and failure/retry cost—not only average tokens.

## 17. Evidence ledger

For each future EOKS claim, record:

| Claim | Evidence | Setting | Effect | Confidence | Next test |
|---|---|---|---|---|---|
| Repository exploration is expensive | SWE-Explore, FastContext, practitioner reports | repository SWE | measurable acquisition/token burden | medium-high | cross-model baseline |
| Retrieval helps but no method dominates | Agent Retrieval Bench | 427 samples / 25 repos | task-dependent winners | high for benchmark setting | legacy + agentic tasks |
| Execution state can reduce cost and improve success | Ledger | 500 SWE-bench Verified | +3.4 pp / ~24% cost on Codex setup; larger gains on other models | medium-high | independent reproduction |
| More repository instructions can hurt | SRI AGENTS.md study | multiple agents/models | no success gain, >20% cost | high for tested setting | task-specific instruction ablation |
| Agent token usage is highly variable | Microsoft Research | 8 frontier models / SWE-bench Verified | up to 30× run variance | high for tested setting | production traces |
| Clearer task specification can reduce cost | 2,700-run Kimi K3 study | agentic coding | -29.7% token effect from richer specification | medium | cross-model replication |
| Subagent topology can explode cost | practitioner reports | large repo workflows | multi-million-token anecdotal spikes | low | controlled branching experiment |

This table should grow as EOKS runs its own experiments. The important artifact is not a static list of tools; it is a **living evidence map connecting claims to conditions, effect sizes and unresolved tests**.
