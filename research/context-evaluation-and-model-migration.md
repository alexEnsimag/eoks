# Context evaluation, benchmarking and model migration

The recent GrapeRoot / OKF / Graphify discussion exposed a practical problem that should precede a larger EOKS implementation:

> **How do we know that a context intervention, knowledge representation, tool, model or control policy actually improves the workload?**

This note turns that question into an experimental methodology. It complements the current evaluation architecture rather than replacing it.

## 1. Evaluation comes before optimization

It is tempting to assemble a context stack first and measure it afterward. That makes attribution difficult: if results improve, we do not know whether the model, retrieval, graph, durable knowledge, prompt, workflow or evaluator caused the change.

The recommended sequence is:

```text
representative workload
        |
        v
baseline + trace
        |
        v
controlled intervention
        |
        v
same workload / comparable conditions
        |
        v
outcome + cost + trace
        |
        v
comparison
```

The first useful benchmark should therefore be small and local: roughly 20–30 representative tasks from a real software-engineering workload. Tasks should cover navigation, codebase understanding, debugging, impact analysis, implementation, refactoring and verification rather than only synthetic coding puzzles.

Each task should have an explicit evaluation contract where practical:

```yaml
id: auth-07
question: "Why can this request bypass the authentication middleware?"
expected:
  - identify the relevant route and middleware
  - identify the bypass path
  - explain the invariant that should hold
  - cite the authoritative implementation/evidence
```

The contract does not need to prescribe the exact wording of the answer. It should define what constitutes a successful outcome.

## 2. Evaluate the whole task, not just the answer

For coding-agent workloads, answer quality is only one layer. Record at least:

- task success;
- correctness;
- completeness;
- groundedness / evidence quality;
- regressions or incorrect changes;
- tests and deterministic checks;
- files changed and whether they were appropriate;
- tool calls / repository rediscovery work;
- input/output/context tokens;
- latency;
- cost.

A useful benchmark should distinguish:

```text
retrieval quality
      |
      v
context quality
      |
      v
agent behavior
      |
      v
task outcome
```

A retrieval intervention can improve retrieval metrics without improving the final task. Conversely, a model can compensate for imperfect retrieval through exploration. Both facts matter.

## 3. Context evaluation

Context should be evaluated independently where possible, while still measuring downstream task outcomes.

Candidate context metrics include:

- relevance;
- coverage / recall of required evidence;
- precision / amount of irrelevant material;
- redundancy;
- contradiction risk;
- provenance and source quality;
- freshness;
- dependency completeness;
- ordering / structure;
- token and latency cost.

RAG evaluation literature provides useful starting concepts such as contextual precision, recall and relevance. These should be treated as retrieval diagnostics, not as substitutes for end-to-end task success.

A particularly useful EOKS experiment is **marginal context value**:

```text
quality(task, context + block) - quality(task, context)
-------------------------------------------------------
       additional context token / latency cost
```

This is initially an experimental comparison, not a claim that online task-quality probabilities can be estimated exactly.

### Context necessity

A further useful dimension is whether retrieved context was actually needed. After a run, label selected blocks as:

- essential;
- useful;
- irrelevant;
- misleading.

This helps distinguish a system that retrieves more from one that retrieves better. A context engine should ideally learn which blocks are necessary for particular task classes and models.

## 4. Baseline and ablation matrix

Do not introduce GrapeRoot, OKF and Graphify simultaneously. A useful initial matrix is:

| Configuration | Structural context | Durable semantic knowledge | Context engine |
|---|---:|---:|---:|
| A | none | none | baseline agent |
| B | GrapeRoot-like | none | GrapeRoot |
| C | GrapeRoot-like | OKF | GrapeRoot |
| D | Graphify-like | none | context compiler |
| E | Graphify-like | OKF | context compiler |

The exact products are less important than keeping the variables separable.

The same idea applies to model comparisons. Hold the context and task constant when comparing models; hold the model constant when measuring a context intervention.

## 5. OKF is a representation, not a service

OKF is useful precisely because it does not require EOKS to deploy a knowledge server. The current OKF v0.2 specification defines a knowledge bundle as a directory tree of Markdown concept documents with YAML frontmatter. `type` is the only always-required frontmatter field; producers may add fields and consumers should tolerate unknown fields. The format has no required runtime, central authority or schema registry.

Therefore a local Git-tracked directory can be an OKF bundle:

```text
knowledge/
  architecture/
    authentication.md
  decisions/
    event-processing.md
  invariants/
    auth.md
```

A minimal concept can be ordinary Markdown with frontmatter:

```yaml
---
type: Invariant
title: Authentication boundary
---

Every externally exposed API route must pass through the authentication boundary.
```

EOKS should not make the OKF taxonomy mandatory. The useful boundary is:

```text
OKF / Markdown / ADRs / other durable representations
                         |
                         v
                 knowledge provider
                         |
                         v
                  context compiler
```

OKF compatibility means following the OKF conventions; a project is also free to use its own local Markdown structure when interoperability is not required. The benchmark should test whether the *knowledge itself* improves outcomes, not whether the OKF label does.

## 6. GrapeRoot as an experimental context engine

Current GrapeRoot documentation describes a local graph scan that extracts files, functions, classes and import relationships, stores a local graph, starts a local MCP server, and installs Claude Code hooks. Its public documentation says hooks re-inject context when Claude compacts its memory and log token usage at session end. The launcher can be used to start Claude Code with the graph service; the public repository states that the graph engine itself is distributed as a proprietary package while the launcher/tooling is open source.

This makes GrapeRoot useful for a controlled experiment because it can be evaluated as a context/execution sidecar around an existing agent rather than requiring a new agent runtime.

The first-run question should be explicit in the benchmark:

```text
project
  |
  +-- graph scan / cache
  |
  +-- local MCP server
  |
  +-- agent integration / hooks
  |
  v
agent session
```

Do not assume that a pre-injected context is automatically better. Measure whether it reduces repository rediscovery and whether that reduction improves task outcomes without increasing irrelevant context or hiding evidence the agent would otherwise discover.

Also distinguish **lifecycle hooks** from **tool-call hooks**. A session-start or compaction hook is a natural place to prime/reconstruct context; a pre-tool hook can potentially enforce or augment policy immediately before a tool executes. EOKS should define the desired event contract independently of a particular Claude/GrapeRoot hook implementation.

## 7. Graphify and other evidence providers

Graphify-like structural graphs are best treated as evidence/navigation providers. They can answer questions about imports, calls, symbols, dependencies and graph neighborhoods. They do not automatically establish semantic invariants or path-sensitive dataflow properties.

This fits the existing EOKS principle of **minimum sufficient evidence**:

```text
type/compiler check
      -> lightweight static rule
      -> targeted AST/compiler analysis
      -> dataflow analysis
      -> deep interprocedural analysis
```

The control plane should select the cheapest provider that can reliably answer the question, and retain the resulting evidence with provenance rather than making the analyzer itself canonical knowledge.

## 8. Evaluation tools from the community

Existing tools cover different parts of this methodology:

### Promptfoo

Promptfoo is useful for repeatable comparisons of prompts, models and configurations, including coding-agent evaluation. It is a good candidate for a lightweight first experiment because the same test cases can be run against multiple configurations and assertions/evaluators can be attached to the results.

### Langfuse

Langfuse provides datasets, experiments, traces and scores. Its documented evaluation loop is especially relevant to EOKS: create a fixed dataset, run the application against it, evaluate outputs, compare experiment runs, then add interesting production traces back into the dataset. Deterministic code evaluators, human annotation and LLM-as-a-judge can coexist.

### Aider benchmarks

Aider is useful prior art for coding-agent evaluation because it emphasizes end-to-end repository outcomes such as whether generated changes pass tests, rather than treating textual answer quality as the only target.

### OpenHands benchmarks

OpenHands maintains benchmark infrastructure around software-engineering and agent tasks. This is useful prior art for treating the agent, tools and execution environment as part of the evaluated system.

### OpenAI Evals and similar eval frameworks

General evaluation frameworks are useful for defining private, workload-specific evals. The important EOKS principle is to maintain a representative golden set for the actual workload rather than relying only on public aggregate benchmarks.

These tools are not competing EOKS components. They can serve as experiment runners, trace stores, evaluators or benchmark references beneath an EOKS evaluation/control layer.

## 9. Model evaluation and migration

A model upgrade should be treated like a production dependency migration, not as a one-time leaderboard decision.

The right question is:

> **Is the candidate model better for this workload under the context and execution policy we actually use?**

Maintain a versioned golden dataset containing representative real tasks. For each model/configuration record:

```text
model/version
context manifest
prompt/configuration
agent/tool versions
outcome
quality scores
cost
latency
tool calls
regressions
```

A model migration scorecard should expose multiple dimensions:

| Dimension | Current model | Candidate model |
|---|---:|---:|
| task success | | |
| tests passing | | |
| correctness | | |
| completeness | | |
| groundedness | | |
| serious regressions | | |
| tool calls | | |
| tokens | | |
| latency | | |
| cost | | |

Do not collapse this immediately into one score. A candidate that is better on average but regresses a high-value task class may not be safe to deploy.

## 10. Model/context interaction matrix

Model and context interventions should also be tested together because their effects can interact.

For example:

| | Baseline context | GrapeRoot | GrapeRoot + OKF | Graph context |
|---|---:|---:|---:|---:|
| Model A | A | B | C | D |
| Model B | E | F | G | H |

This allows separate questions:

- model effect: `E - A`;
- GrapeRoot effect: `B - A`;
- OKF incremental effect: `C - B`;
- Graphify/context-provider effect: `D - B`;
- interaction effects, such as `(G - F) - (C - B)`.

The numbers are not assumed to be additive. The point is to expose whether a context intervention is useful for one model but not another.

This is especially important for model upgrades: a new model may compensate for weak retrieval, exploit structured context better, or react differently to large context packs. Therefore model migration and context migration should not be evaluated as unrelated changes.

## 11. Canary and regression workflow

A practical migration loop is:

```text
new model/version
       |
       v
run golden set
       |
       v
compare with production model
       |
       +-- critical regression? -- yes --> reject / investigate
       |
       no
       v
staged/canary evaluation
       |
       v
production traces + online evaluation
       |
       v
promote or roll back
```

Online evaluation should feed new edge cases back into the offline dataset. This creates a continuous assurance loop rather than a benchmark that becomes stale after the initial migration.

## 12. Evaluation architecture for EOKS

The resulting EOKS evaluation stack can be represented as:

```text
                         EOKS evaluation/control
                                  |
             +--------------------+--------------------+
             |                    |                    |
        experiment runner     trace store          evaluators
             |                    |                    |
       Promptfoo/etc.       Langfuse/etc.       code / human / LLM
             |                    |                    |
             +--------------------+--------------------+
                                  |
                           golden datasets
                                  |
                     +------------+------------+
                     |                         |
                task outcomes             context metrics
                     |                         |
                     +------------+------------+
                                  |
                            policy evidence
```

The evaluator should make attribution possible:

```text
Task
 -> context decision
 -> context manifest
 -> model/tool decisions
 -> run trace
 -> outcome
 -> evaluation
 -> policy update
```

This trace is more important to EOKS than any individual benchmark framework.

## 13. Proposed initial experiment

The first proving ground should be deliberately small:

1. Create 20–30 real software-engineering tasks.
2. Run the baseline agent and capture traces/outcomes.
3. Run GrapeRoot alone against the same tasks.
4. Inspect which tasks improved, regressed or were unchanged.
5. Add a small manually curated OKF bundle and rerun.
6. Add Graphify-like structural evidence and rerun.
7. Compare context metrics with end-to-end outcomes.
8. Repeat the strongest configurations with the candidate model.
9. Keep the dataset and evaluation logic versioned.
10. Add regressions and newly discovered edge cases back into the dataset.

The goal is not to prove that any named tool is universally useful. The goal is to discover **which context/evidence/model decisions improve which workload classes, at what cost, and with what failure modes**.

## 14. Architectural implication

This methodology strengthens the EOKS control-plane hypothesis:

> **Evaluation is not merely a reporting subsystem. It is the feedback mechanism that lets EOKS learn whether context, knowledge, evidence providers, models and execution policies are actually useful.**

That suggests a deeper control loop:

```text
workload
   |
   v
context / evidence / model policy
   |
   v
agent run
   |
   v
outcome + trace
   |
   v
evaluation + calibration
   |
   v
policy evidence
   |
   +----> next run
   +----> model migration decision
   +----> context-selection update
   +----> knowledge promotion / invalidation
```

The benchmark therefore becomes part of the control plane's evidence model, not an external scorecard attached after the fact.
