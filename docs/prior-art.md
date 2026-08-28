# Prior art and adjacent systems

EOKS is a synthesis, not a claim that these ideas are new. Recent research suggests that the ecosystem is converging on several distinct layers: canonical project knowledge, structural/semantic representations, context compilation, workflows, reasoning strategies, memory/learning, and evaluation.

The important EOKS question is not "which tool wins?" but **which layer does a tool implement, what evidence does it produce, and how can that evidence participate in the control loop?**

## A practical scorecard

Scores below are **EOKS-fit scores, not product-quality rankings**. They reflect usefulness for the architecture being explored here, with particular weight on incremental maintenance, provenance, interoperability and practical value for a coding-agent workflow.

| Tool / concept | EOKS role | Fit | Main reason to try | Main caveat |
|---|---|---:|---|---|
| Hierarchical `CLAUDE.md` | canonical local knowledge | **9.5/10** | Cheap, human-reviewable, Git-native, naturally scoped | Can become stale/manual if not reviewed |
| GrapeRoot | context optimization / agent context | **9.5/10** | Directly attacks repeated repository/context work | More runtime-oriented than canonical knowledge |
| Graphify | structural graph / navigation | **9/10** | Excellent visualization, relationships and queryable structure | Graph is evidence, not complete knowledge |
| TrueCourse | architecture analysis + spec/behavior guard | **9/10** | Connects deterministic analysis, docs and executable assurance | Not primarily a `CLAUDE.md` generator |
| Understand Anything | code/domain knowledge graph | **8.5/10** | Interactive graph, guided understanding, impact/change analysis | Full graph generation can itself be expensive; incremental path is still evolving |
| Hermes | agent learning / reflection | **8.5/10** | Interesting direction for turning experience into reusable capability | Requires careful promotion/governance |
| Liza | multi-agent execution + auditability | **8/10** | Strong workflow/review/documentation ideas | More execution-focused than knowledge infrastructure |
| OKF | portable structured knowledge convention | **8/10 conceptually** | Useful interoperability target if richer machine-readable knowledge becomes necessary | Adds a format/lifecycle before it is clear the project needs one |
| ADHD-style reasoning strategies | reasoning strategy layer | **8/10** | Useful primitive for divergent/convergent or adversarial thinking | The specific "ADHD" framing is less important than the reusable strategy concept |
| Obsidian | human thinking / research workspace | **8/10** | Good place for cross-cutting architecture/research before promotion to canonical docs | Should not be required at agent runtime |
| OpenWolf | interaction-derived summaries / context optimization | **8/10** | Demonstrates incremental repository memory around agent interactions | Generated summaries need provenance and validation |
| CodeSight | repository understanding / context | **7.5/10** | Useful prior art for codebase context | Less central to the current canonical-knowledge approach |

These scores should be treated as experiment priorities, not permanent evaluations.

## GrapeRoot

GrapeRoot was discussed as a context-aware CLI around Claude. Its most relevant EOKS role is the **context/execution side**: reduce repeated repository exploration and improve the information supplied to an agent for a task.

It should not be treated as the canonical knowledge base. A useful EOKS composition is:

```text
canonical knowledge + derived evidence
                 |
                 v
          context compilation
                 |
              GrapeRoot-like
                 |
               agent
```

The key evaluation is whether it reduces discovery/tool-call cost without hiding relevant evidence or increasing errors.

## Graphify

Graphify is a particularly strong example of the structural representation discussed in EOKS. Its current implementation uses local Tree-sitter-based parsing for code, produces an interactive `graph.html`, a human-readable `GRAPH_REPORT.md` and a queryable `graph.json`, and distinguishes extracted from inferred relationships. It can also expose the graph through MCP and install guidance/hooks for coding assistants. citeturn1search0turn1search2

This makes it valuable for three separate reasons:

1. **visualization** — a human can see the shape of a codebase;
2. **navigation/impact analysis** — an agent can query relationships instead of rediscovering them through grep;
3. **incremental/compiler thinking** — the graph can serve as a dependency representation used to identify affected areas.

Graphify should still not be equated with "the knowledge base". The graph is one representation, especially strong for structural questions.

## Understand Anything

Understand Anything is a strong adjacent system because it combines deterministic parsing with LLM analysis to produce an interactive knowledge graph, including structural and business-domain views, guided tours and change-impact capabilities. It is explicitly positioned for both humans and coding agents. citeturn0search0turn0search8

An important recent implementation detail is its auto-update hook path: it can detect when the stored graph was built from a different Git commit and prompt an update, while its chat/diff skills are instructed to check graph freshness and read only the graph sections needed. citeturn0search5turn0search9

This is valuable EOKS prior art for **incremental knowledge representations** and for the principle that agents should query a representation selectively rather than dump the entire graph into context. Its own issue tracker also contains an open discussion about incremental updates consuming more tokens than the initial build, which is a useful warning: incremental does not automatically mean cheap. citeturn0search3

## TrueCourse

TrueCourse is best understood as **architecture/code intelligence plus specification-to-behavior assurance**, not primarily as a documentation generator. It combines deterministic rules with LLM review and has a separate `spec -> scenario tests -> guard` workflow for detecting when implementation drifts from documented behavior. Results are stored locally in `.truecourse/` as JSON artifacts. citeturn0search1

For EOKS, this belongs primarily in **evaluation/policy/evidence**, with a secondary relationship to canonical knowledge:

```text
PRDs / ADRs / docs
       |
       v
  executable specs
       |
       v
 implementation
       |
       v
   guard result
       |
       +--> evaluation / confidence / feedback
```

It can help maintain the truthfulness of the knowledge system, but it should not be assumed to automatically author good `CLAUDE.md` files.

## OKF

OKF was discussed as a structured, portable file-oriented knowledge convention. The EOKS conclusion is intentionally conservative: **a representation can be meaningful without requiring hosting infrastructure**, and a local collection of Markdown/JSON files can implement a useful subset of the same idea.

Hierarchical `CLAUDE.md` files can therefore be seen as a pragmatic subset of the canonical-knowledge problem: scoped, human-readable, Git-versioned and loaded near the code where needed. OKF becomes more valuable if EOKS later needs machine-readable interoperability across many tools, explicit schemas, richer provenance or cross-project federation.

## Hermes

Hermes is relevant to the **learning/reflection** layer. Its interesting idea is that useful work should leave behind reusable knowledge or capability rather than disappearing at session end.

EOKS should generalize that into a governed pipeline:

```text
session / workflow
       |
trace + artifact + outcome
       |
 candidate learning
       |
 validation + scope + confidence
       |
 promoted memory / procedure / knowledge
```

The important warning is that repeated behavior alone is not sufficient evidence that a behavior is good.

## Liza

Liza explores hardened multi-agent coding with an emphasis on quality, auditability, automated reviews and documentation, including ADR-oriented workflows. citeturn0search4

Its strongest EOKS contribution is to the **execution/evaluation layer**: workflows should make quality gates and evidence explicit rather than treating a successful-looking demo as sufficient.

## ADHD Stack / cognitive strategies

The "ADHD" projects discussed in research are useful primarily as a naming accident for a broader idea: **reasoning strategies can be reusable execution components**.

Examples include:

- divergent exploration;
- independent alternatives;
- adversarial critique;
- convergence;
- explicit state externalization.

EOKS should model this as a **reasoning strategy layer**, independent of the model and separate from knowledge and workflow.

## Obsidian

Obsidian is useful as a **human thinking workspace**, not necessarily as an EOKS runtime dependency. It can hold research, architecture sketches and long-form thinking that later becomes a reviewed ADR or project knowledge file.

The important boundary is:

```text
human thinking / research
          |
       Obsidian
          |
  reviewed canonical docs
          |
       EOKS knowledge
```

## OpenWolf

OpenWolf is relevant to a concrete coding-agent failure mode: repeated reconstruction of repository context across sessions. Its local, hook-driven approach maintains repository-oriented summaries and persistent notes around agent/file interactions.

For EOKS, OpenWolf is best treated as **knowledge extraction + context optimization**, not as the complete knowledge layer. Generated memory should be evidence-bearing and subject to freshness, provenance and promotion rules.

## CodeSight

CodeSight is relevant to codebase context and repository understanding. It illustrates the value of preparing structured knowledge for coding agents and fits primarily in the evidence/navigation side of EOKS.

## Deterministic analysis: Tree-sitter, Semgrep, CodeQL

These tools reinforce an important architectural rule: deterministic questions should be answered deterministically whenever possible.

- Tree-sitter/language tooling -> symbols, syntax, imports, calls;
- Semgrep -> structural/security patterns;
- CodeQL -> deeper dataflow/security relationships.

They can be evidence providers underneath the context and evaluation planes without requiring an LLM to rediscover facts that static analysis can establish cheaply.

## Evidence-provider abstraction

The tooling landscape suggests a useful EOKS abstraction: an **evidence provider** answers a bounded question and returns evidence with provenance, scope/revision, validation/confidence and cost/latency characteristics.

Examples:

```text
repository graph     -> dependency evidence
Tree-sitter          -> structural evidence
Semgrep              -> structural/security evidence
CodeQL               -> deep dataflow evidence
Graphify             -> graph/navigation evidence
Understand Anything  -> synthesized code/domain evidence
TrueCourse           -> architecture/spec compliance evidence
GrapeRoot            -> task-context optimization
OpenWolf             -> interaction-derived project context
tests                -> behavioral evidence
observability        -> runtime evidence
CLAUDE.md / ADRs     -> canonical human-reviewed knowledge
```

The control plane should choose the minimum sufficient set of providers for the workload instead of always running the deepest analysis.

## The emerging stack

The combined prior art suggests the following practical architecture:

```text
                 human intent / task
                         |
                      workflow
                         |
                 reasoning strategy
                         |
                 context compilation
                         |
      +------------------+------------------+
      |                  |                  |
 canonical docs      structural        semantic /
 CLAUDE.md / ADRs       graph            search
      |                  |                  |
      +------------------+------------------+
                         |
                      model
                         |
                 execution + tools
                         |
                    evaluation
                         |
                  learning/update
```

This is deliberately compositional. No single project needs to become EOKS.
