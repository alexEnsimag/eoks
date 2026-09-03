# Context evaluation and benchmarking

This note owns the **controlled evaluation of context systems, context acquisition strategies, execution-state mechanisms and evidence providers**. It answers the question that should precede a larger EOKS implementation:

> **Does an intervention improve the end-to-end software-engineering workload, for which model/repository/task, at what cost, and with which failure modes?**

A popular project, community report or academic paper is a research signal—not proof of general superiority. The EOKS baseline is always the strongest practical baseline available for the workload.

## 1. Evaluation before optimization

Build a benchmark before assembling a context stack. Otherwise an improvement cannot be attributed to the model, retrieval, graph, durable knowledge, prompt, workflow, execution state or evaluator.

Start with roughly **20–30 representative real software-engineering tasks** for an initial local benchmark. Cover navigation, codebase understanding, debugging, impact analysis, implementation, refactoring, review and verification. Expand the set as failures are discovered. Prefer real tasks over synthetic puzzles when the goal is repository-level engineering behavior.

Each task should have an evaluation contract defining success without prescribing exact wording:

```yaml
id: auth-07
question: "Why can this request bypass the authentication middleware?"
expected:
  - identify the relevant route and middleware
  - identify the bypass path
  - explain the invariant that should hold
  - cite authoritative evidence
```

Record a baseline before changing the model or adding infrastructure.

## 2. The experimental matrix

EOKS should treat effectiveness as conditional on several variables:

```text
model × repository × task × intervention × budget
```

At minimum vary:

**Model**
- frontier/high-capability model;
- cheaper or smaller model;
- model/version changes where migration is relevant.

**Repository**
- AI-native/well-structured;
- modern/mature;
- large or heterogeneous;
- legacy/poorly documented.

**Task**
- retrieval/navigation;
- understanding/explanation;
- debugging;
- implementation;
- refactoring/migration;
- review;
- verification.

**Intervention**
- raw filesystem/tool exploration;
- lexical/semantic retrieval;
- RepoMap-style summaries;
- LSP/semantic tooling;
- structural graphs;
- agentic search/sub-agents;
- precomputed repository analysis;
- context selection/compilation;
- context lifecycle management;
- execution state;
- durable knowledge/memory;
- deterministic verification;
- combinations of the above.

Reasoning-format changes belong in this same intervention axis rather than requiring a separate EOKS abstraction. Chain-of-Draft is useful evidence for why the effect must be measured by workload: its original evaluation reported very large token reductions on its tested reasoning tasks, while a subsequent software-engineering evaluation on 300 SWE-bench samples reported a smaller reduction to 55.4% of CoT tokens while retaining over 90% of the reported CoT code-quality measures. The contrast supports the existing EOKS methodology: evaluate representation changes against task outcomes and total resource cost under the model/repository/task/budget matrix, rather than treating token reduction as a universal objective. [Xu et al., *Chain of Draft: Thinking Faster by Writing Less*, arXiv:2502.18600](https://arxiv.org/abs/2502.18600) · [Yang, *Chain of Draft for Software Engineering*, arXiv:2506.10987](https://arxiv.org/abs/2506.10987)

**Budget**
- model tokens;
- tool calls;
- wall-clock time;
- infrastructure/preprocessing cost;
- total monetary cost.

This matrix is a guard against statements such as “graphs help agents” when the actual result is “graphs help this model on this task at this budget”.

## 3. Evaluate the whole task

For coding-agent workloads, textual answer quality is only one layer. Record, where applicable:

- task success;
- correctness and completeness;
- groundedness/evidence quality;
- tests and deterministic checks;
- inappropriate changes or regressions;
- files changed;
- tool calls and repository rediscovery;
- input/output/context tokens;
- latency;
- total cost;
- number of retries/branches;
- session resets or recovery events;
- verification failures.

Keep the causal chain visible:

```text
acquisition / retrieval
        |
        v
working-context quality
        |
        v
agent behavior + execution state
        |
        v
verification
        |
        v
task outcome
```

A retrieval metric can improve without improving the task. An agent can compensate for incomplete retrieval by exploring more. Both are useful observations.

## 4. Context acquisition is itself a workload

Do not classify all grep/read/tool exploration as waste. Agents can use executable tools and filesystem navigation as an external mechanism for processing information, and exploration can be part of semantic understanding.

The research question is instead:

> **Which exploration is productive reasoning, and which is avoidable information acquisition?**

Compare raw exploration with retrieval, RepoMap, graphs, LSP, semantic indexes, agentic search and hybrid strategies. Measure both the final context/outcome and the work required to obtain it.

Useful acquisition metrics include:

- discovery tool calls;
- repeated/near-duplicate queries;
- files or symbols inspected;
- relevant evidence found;
- relevant evidence missed;
- irrelevant evidence inspected;
- time to first useful evidence;
- context growth during exploration;
- task outcome per acquisition token/call.

A tool that reduces exploration but causes the model to miss an important dependency is not an improvement.

## 5. Working context and execution state are different interventions

**Working context** asks what the model should see now.

**Execution state** asks what the workflow has already established, changed, attempted, verified or invalidated.

Evaluate execution-state mechanisms separately from transcript summarization and context compression. A compact state record may prevent redundant commands without requiring the whole history to remain in the active prompt.

A useful experiment is:

```text
A  baseline transcript/tool history
B  + context compression
C  + explicit execution state
D  + context compression + execution state
```

Measure task success, redundant actions, recovery from mistakes, tokens, latency and stale-state failures.

## 6. Context metrics

Evaluate context independently where possible, but never replace end-to-end outcomes with retrieval scores.

Useful dimensions include:

- relevance;
- coverage/recall of required evidence;
- precision/irrelevant material;
- redundancy;
- contradiction risk;
- provenance and source quality;
- freshness;
- dependency completeness;
- ordering/structure;
- token and latency cost;
- acquisition cost.

After a run, classify selected blocks as:

- **essential**;
- **useful**;
- **irrelevant**;
- **misleading**.

This distinguishes “retrieves more” from “retrieves better”.

## 7. Context size is not context quality

Long-context models can process very large inputs, but academic evidence shows that relevant information can become harder to use depending on position, density and interaction with other information. Conversely, coding-agent research shows that agents can process large external corpora effectively through tools.

Therefore test both:

```text
same budget + different composition
```

and:

```text
different budgets + best available composition
```

Do not treat a larger context window as proof that indiscriminate context stuffing is beneficial.

## 8. Context lifecycle

Compare proactive, reactive and hybrid delivery:

```text
proactive: Task -> retrieval/packing -> model

reactive:  Task -> model -> query -> evidence -> model ...

hybrid:    compact bootstrap -> targeted retrieval as needed
```

Also test active context management, where the agent can decide when to compress, offload, reconstruct or otherwise manage its working context.

The important metric is not compression ratio. It is whether the lifecycle mechanism preserves useful information while improving correctness, autonomy and resource efficiency.

## 9. Model substitution experiments

Infrastructure may have different value at different capability levels. A particularly important EOKS hypothesis is:

> **Can external knowledge/context/execution infrastructure reduce the capability or cost gap between models?**

Use the same task/repository/intervention matrix:

```text
frontier model + baseline
frontier model + intervention

cheaper model + baseline
cheaper model + intervention
```

Report both absolute improvement and change in the gap between models. Do not assume that an intervention that helps a cheaper model will help a frontier model equally.

## 10. Repository maturity experiments

A context technique that is unnecessary on a clean AI-native repository may be valuable on a large legacy system.

Benchmark at least:

```text
AI-native / structured
        |
modern / mature
        |
legacy / heterogeneous / poorly documented
```

Record repository characteristics such as size, language count, generated-code share, test coverage, documentation density and architectural age where practical.

For repository transformations and migrations, measure completeness and behavioral correctness separately from patch-level success. Whole-repository change remains a substantially harder workload than many conventional bug-fix benchmarks imply.

## 11. Controlled intervention sequence

Do not introduce multiple mechanisms simultaneously. A useful sequence is:

```text
A  strongest practical baseline agent
B  + retrieval / semantic navigation
C  + structural evidence / graph
D  + durable knowledge
E  + context lifecycle management
F  + execution state
G  + deterministic verification
H  combined configuration
```

The exact order can change by workload. The important property is attribution.

Also compare against **raw exploration** explicitly. Infrastructure should not receive credit merely because it looks more sophisticated.

## 12. Evidence hierarchy

Use different evidence types for different purposes:

```text
community signal
      ↓
project adoption / repeated reports
      ↓
academic controlled result
      ↓
independent reproduction
      ↓
cross-model / cross-repository replication
      ↓
EOKS workload-specific evidence
```

Popularity is a reason to investigate a project, not evidence that it works. A benchmark result is evidence for its measured setting, not a universal architecture rule.

Community discussions are especially useful for discovering failure modes and recurring pain points. Academic work is useful for controlled hypotheses and measurement methods. EOKS should preserve both while keeping confidence levels explicit.

## 13. Relevant research directions

Recent academic work motivating this methodology includes research on:

- long-context positional degradation and “lost in the middle” effects;
- coding agents as effective long-context processors using tools and filesystem navigation;
- repository context acquisition and retrieval benchmarks;
- active context management for long-horizon software agents;
- explicit execution-state/ledger mechanisms;
- semantic retrieval versus deep agentic repository search;
- graph-based repository representations;
- precomputed/speculative repository context;
- whole-repository refactoring/migration benchmarks.

These results are deliberately treated as **evidence for experiments, including contradictory experiments**, rather than as EOKS architectural decisions.

## 14. OKF: representation, not runtime

OKF is a durable knowledge representation, not an EOKS runtime requirement. It should be evaluated against other representations such as reviewed Markdown, ADRs, repository-local instructions and structured records.

The useful boundary is:

```text
OKF / CLAUDE.md / ADRs / other durable representations
                         |
                         v
                  knowledge provider
                         |
                         v
                  context compiler
```

If agents write durable knowledge, generated material must remain distinguishable from reviewed/verified material, with provenance and lifecycle state.

## 15. Evidence-provider escalation

Structural graphs are evidence/navigation providers, not automatic semantic truth. Compare the cheapest provider capable of answering a question with deeper analysis:

```text
type/API design
    -> lightweight lint/static rule
    -> AST/compiler analysis
    -> dataflow analysis
    -> deep interprocedural analysis
```

Measure setup cost, runtime, coverage, false positives/negatives, evidence usefulness and final task outcome. Do not assume the deepest analyzer is best.

## 16. Reuse evaluation infrastructure

EOKS should reuse established evaluation and tracing infrastructure rather than building another generic runner. Prompt/evaluation frameworks, trace stores and coding-agent benchmarks are useful components, but they remain components rather than the EOKS control plane.

A clean separation is:

```text
benchmark      -> defines representative tasks
experiment     -> runs configurations
trace          -> records what happened
evaluator      -> scores outcomes
EOKS           -> uses evidence for policy/control
```

## 17. Initial benchmark workflow

1. Create 20–30 representative real tasks.
2. Define task-specific success contracts.
3. Capture a strong baseline trace and outcome.
4. Run one intervention at a time.
5. Inspect improved, unchanged and regressed tasks.
6. Measure acquisition, context, execution-state and outcome metrics separately.
7. Repeat the strongest results across another model.
8. Repeat on a materially different repository class.
9. Test combined configurations only after individual effects are understood.
10. Version the dataset/evaluator and add important production failures as regression cases.

The objective is falsifiability:

> **Discover which intervention helps which workload class, at what cost, with which model, and with which failure modes.**

## 18. Evaluation/control boundary

The resulting loop is:

```text
workload
   |
   v
context / acquisition / evidence policy
   |
   v
agent run + execution state
   |
   v
outcome + trace
   |
   v
evaluation
   |
   v
policy evidence
   |
   +--> context-selection update
   +--> acquisition/provider selection
   +--> execution-state update
   +--> knowledge promotion/invalidation
   +--> model-routing/migration evidence
```

The benchmark is therefore not an external scorecard. It is an evidence source for the EOKS control loop.