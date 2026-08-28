# Context evaluation and benchmarking

This note owns the **controlled evaluation of context systems and evidence providers**. It answers a practical question that should precede a larger EOKS implementation:

> **How do we know that a context intervention, knowledge representation or evidence provider actually improves the workload?**

The reliability and model-migration policy that consumes these results lives in [Evaluation, reliability and model switching](evaluation-and-model-switching.md).

## 1. Evaluation before optimization

Build a benchmark before assembling a context stack. Otherwise an improvement cannot be attributed to the model, retrieval, graph, durable knowledge, prompt, workflow or evaluator.

Start with roughly **20–30 representative real software-engineering tasks**. Cover navigation, codebase understanding, debugging, impact analysis, implementation, refactoring and verification. Real tasks are preferable to synthetic puzzles because they expose the repository-specific failures EOKS is meant to control.

Each task should have an evaluation contract that defines success without prescribing exact wording:

```yaml
id: auth-07
question: "Why can this request bypass the authentication middleware?"
expected:
  - identify the relevant route and middleware
  - identify the bypass path
  - explain the invariant that should hold
  - cite authoritative evidence
```

Record a baseline before adding a context engine or changing the model.

## 2. Evaluate the whole task

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
- cost.

Keep the causal chain visible:

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

A retrieval metric can improve without improving the task. A model can also compensate for incomplete retrieval by exploring more. Both are useful observations.

## 3. Context metrics

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
- token and latency cost.

Existing RAG evaluation terminology such as contextual precision, recall and relevance is useful as a starting point.

### Marginal context value

Measure the incremental value of a context block relative to its cost:

```text
quality(task, context + block) - quality(task, context)
-------------------------------------------------------
       additional context token / latency cost
```

Treat this as an experimental comparison, not a universal online probability.

### Context necessity

After a run, classify selected blocks as:

- **essential**;
- **useful**;
- **irrelevant**;
- **misleading**.

This distinguishes “retrieves more” from “retrieves better” and provides training/evaluation data for future context selection.

## 4. Controlled context experiments

Do not introduce GrapeRoot, OKF and Graphify simultaneously. A useful sequence is:

```text
A  baseline agent
B  + GrapeRoot-like context engine
C  + durable knowledge
D  + structural evidence provider
E  combined configuration
```

Hold model and task set constant when measuring a context intervention. Hold context and task set constant when comparing models. Use an explicit model × context matrix when interaction is the question.

The exact products are less important than keeping variables separable.

## 5. OKF: representation, not runtime

The current OKF v0.2 specification defines a **knowledge bundle** as a directory tree of Markdown concept documents with YAML frontmatter. `type` is the only always-required frontmatter field; additional fields are allowed and consumers are expected to tolerate unknown fields. The format does not prescribe a server, database, schema registry or runtime. A bundle can live in Git, a subdirectory, an archive or another file-serving system. citeturn0search0turn0search1

Therefore EOKS can experiment locally without hosting anything:

```text
knowledge/
  index.md
  architecture/
    authentication.md
  decisions/
    event-processing.md
  invariants/
    auth.md
```

A conformant project should follow the specification rather than merely calling an arbitrary Markdown directory “OKF”. At the same time, EOKS should not require OKF: a project can use its own local representation when interoperability is not needed.

The architectural boundary is:

```text
OKF / CLAUDE.md / ADRs / other durable representations
                         |
                         v
                  knowledge provider
                         |
                         v
                  context compiler
```

OKF v0.2 also makes provenance, generated/verified state and lifecycle signals available as optional frontmatter families. This is particularly relevant if agents start writing the bundle: generated knowledge should remain distinguishable from verified knowledge. citeturn0search2turn0search5

## 6. GrapeRoot: context-engine prior art

GrapeRoot is useful as an experimental context/execution sidecar around a coding agent. Its public documentation describes a first-run project scan/graph construction, local MCP integration and Claude Code hooks; its graph engine is not simply equivalent to the OKF knowledge representation.

For EOKS, the important lifecycle question is:

```text
project
  |
  +-- repository analysis / graph cache
  +-- context retrieval/packing
  +-- agent integration
  |
  v
agent session
```

Do not assume that pre-injection is beneficial. Measure whether it reduces repository rediscovery and whether that reduction improves task outcomes without adding irrelevant context or hiding authoritative evidence.

Also distinguish lifecycle events from tool-call policy:

```text
context boundary / session start
          -> prepare context

before tool execution
          -> optional policy/evidence check

after tool execution
          -> observe outcome

session end
          -> finalize / persist candidates
```

EOKS should define these semantics independently of GrapeRoot or Claude Code so another runtime can implement them.

## 7. Graphify and evidence providers

Graphify-like graphs should be treated as **structural evidence/navigation providers**. They can answer questions about symbols, callers, imports, dependencies and graph neighborhoods. They do not automatically establish semantic invariants or path-sensitive dataflow properties.

This fits a broader evidence-provider escalation strategy:

```text
type/API design
    -> lightweight lint/static rule
    -> targeted AST/compiler analysis
    -> dataflow analysis
    -> deep interprocedural analysis
```

The control plane should select the cheapest provider that can reliably answer the question and attach provenance, scope, freshness and validation information to the result.

## 8. Community evaluation tooling

EOKS should reuse evaluation infrastructure rather than build another generic eval runner.

- **Promptfoo** — useful for repeatable comparisons of models, prompts and configurations and for attaching assertions/evaluators.
- **Langfuse** — useful when offline datasets/experiments need to connect with production traces and scores.
- **Aider benchmarks** — useful prior art for evaluating coding agents through end-to-end repository outcomes and tests.
- **OpenHands benchmarks** — useful prior art for software-engineering agent evaluation across models and execution environments.
- **OpenAI Evals-style frameworks** — useful examples of reusable/private workload-specific evaluation harnesses.

These tools occupy infrastructure roles—experiment runner, trace store, evaluator or benchmark reference. They are not the EOKS control plane.

A useful separation is:

```text
benchmark      -> defines representative tasks
experiment     -> runs configurations
trace          -> records what happened
evaluator      -> scores outcomes
EOKS           -> uses the evidence for policy/control
```

## 9. Initial benchmark workflow

1. Create 20–30 representative real tasks.
2. Define task-specific success contracts.
3. Run the baseline and capture traces/outcomes.
4. Run the context engine alone.
5. Inspect improved, unchanged and regressed tasks.
6. Add a small manually curated durable-knowledge bundle and rerun.
7. Add structural evidence and rerun.
8. Compare context diagnostics with end-to-end outcomes.
9. Repeat the strongest configurations with a candidate model.
10. Version the dataset and evaluator; add regressions and new production edge cases back into the set.

The objective is falsifiability: discover **which intervention helps which workload class, at what cost, and with which failure modes**.

## 10. Evaluation/control boundary

The resulting loop is:

```text
workload
   |
   v
context / evidence policy
   |
   v
agent run
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
   +--> evidence-provider selection
   +--> knowledge promotion/invalidation
   +--> model-migration evidence
```

The benchmark is therefore not an external scorecard. It is an evidence source for the EOKS control loop.
