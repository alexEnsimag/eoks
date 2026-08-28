# Prior art and adjacent systems

EOKS is a synthesis, not a claim that these ideas are new. Recent work suggests that the ecosystem is converging on several distinct layers: canonical project knowledge, organizational/system context, structural/semantic representations, context compilation, workflows, reasoning strategies, memory/learning, trust/evidence and evaluation.

The important EOKS question is not "which tool wins?" but **which layer does a tool implement, what evidence does it produce, and how can that evidence participate in the control loop?**

## A practical scorecard

Scores below are **EOKS-fit scores, not product-quality rankings**. They reflect usefulness for the architecture being explored here, with particular weight on incremental maintenance, provenance, interoperability and practical value for a coding-agent workflow.

| Tool / concept | EOKS role | Fit | Main reason to try | Main caveat |
|---|---|---:|---|---|
| Hierarchical `CLAUDE.md` | canonical local knowledge | **9.5/10** | Cheap, human-reviewable, Git-native, naturally scoped | Can become stale/manual if not reviewed |
| GrapeRoot | context optimization / agent integration | **9.5/10** | Directly attacks repeated repository/context work and demonstrates a sidecar around existing agents | Core graph engine is proprietary; broader assurance/control loop is outside its scope |
| Graphify | structural graph / navigation | **9/10** | Excellent visualization, relationships and queryable structure | Graph is evidence, not complete knowledge or dataflow proof |
| TrueCourse | architecture analysis + spec/behavior guard | **9/10** | Connects deterministic analysis, docs and executable assurance | Not primarily a context/control-plane runtime |
| Xirp | institutional/system context + shared session continuity | **9/10** | Connects coding agents to services, ownership, architecture and prior work | Publicly presented as beta; scope is broader than repository knowledge but narrower than an EOKS control plane |
| Understand Anything | code/domain knowledge graph | **8.5/10** | Interactive graph, guided understanding, impact/change analysis | Full graph generation can itself be expensive; incremental path is still evolving |
| Hermes | agent learning / reflection | **8.5/10** | Interesting direction for turning experience into reusable capability | Requires careful promotion/governance |
| Liza | multi-agent execution + auditability | **8/10** | Strong workflow/review/documentation ideas | More execution-focused than knowledge infrastructure |
| OKF | portable structured knowledge convention | **9/10 conceptually** | Gives durable, human/agent-friendly knowledge a portable representation with provenance/lifecycle concepts | EOKS should consume it rather than duplicate its schema or make it mandatory |
| ADHD-style reasoning strategies | reasoning strategy layer | **8/10** | Useful primitive for divergent/convergent or adversarial thinking | The specific naming is less important than the reusable strategy concept |
| Obsidian | human thinking / research workspace | **8/10** | Good place for cross-cutting architecture/research before promotion to canonical docs | Should not be required at agent runtime |
| OpenWolf | interaction-derived summaries / context optimization | **8/10** | Demonstrates incremental repository memory around agent interactions | Generated summaries need provenance and validation |
| CodeSight | repository understanding / context | **7.5/10** | Useful prior art for codebase context | Less central to the current canonical-knowledge approach |
| Semgrep | static-analysis / dataflow evidence | **8/10 as an evidence provider** | Good fit for source→propagation→barrier→sink rules | Setup and analysis depth can be excessive for one local invariant |
| CodeQL | deep semantic/dataflow evidence | **6.5/10 for general EOKS use** | Powerful path-aware, queryable analysis for difficult questions | Primarily optimized around deep program/security analysis; usually overkill for a narrow project invariant |
| TypeScript types / compiler | invariant prevention | **9/10 for TypeScript projects** | Makes many invalid states unrepresentable at compile time | Cannot express every path-sensitive convention in an existing design |
| ESLint | lightweight local policy | **8.5/10** | Fast developer feedback and easy project-specific rules | Not a general interprocedural dataflow engine |
| ts-morph / compiler API | targeted custom analysis | **8.5/10** | Pragmatic middle ground for TypeScript-specific invariants | Can accidentally grow into a home-grown dataflow engine |

These scores should be treated as experiment priorities, not permanent evaluations.

## The emerging layer model

The recent comparisons suggest a more precise division of labor:

```text
             durable knowledge
                    |
        +-----------+-----------+
        |                       |
       OKF                 project docs / ADRs
        |
        +-------------------+
                            |
                     representations
                  /       |        \
             Graphify   RAG/index   analyzers
                  \       |        /
                   +------+-------+
                          |
                   context compiler
                          |
                    GrapeRoot-like
                          |
                    existing agent
                          |
                 execution + tools
                          |
                     evaluation
                          |
                       outcome
                          |
                       EOKS
                 control / feedback
```

This is not a claim that the tools literally implement this exact stack. It is an EOKS architectural mapping.

## OKF

OKF is best understood as a **knowledge representation/interchange layer**, not as an agent runtime or control plane. Its current direction is particularly relevant because it treats structured knowledge as durable files that can be consumed by both people and agents and adds explicit concepts around provenance, verification/trust and lifecycle/freshness.

That makes OKF a strong candidate for the durable knowledge plane of EOKS, but EOKS should not require all knowledge to be converted to OKF. Source code, compiler facts, runtime observations, tests and other authoritative evidence may be cheaper or more appropriate in their native form.

The EOKS boundary is therefore:

```text
OKF / Markdown / graphs / tests / runtime evidence
                       |
                       v
               knowledge/evidence
                       |
                       v
              EOKS context compiler
                       |
                       v
                     model
```

The important idea from OKF's trust direction is that **knowledge should carry evidence about where it came from, who/what generated it, when it was verified and whether it is stale**. EOKS can use those signals in task-specific policies without turning them into a universal confidence scalar.

See [Knowledge, context and the EOKS control plane](../research/knowledge-context-control-plane.md).

## GrapeRoot

GrapeRoot is particularly relevant to the **context/execution boundary**. Its public documentation describes a local code graph, a context packer that pre-injects relevant structured context before the coding agent reasons, session memory that weights previously touched files, and optional graph-aware MCP tools for deeper exploration.

The important architectural evolution is from reactive graph access:

```text
agent -> MCP graph query -> result -> agent -> more queries
```

toward proactive pre-injection:

```text
question -> graph/context retrieval -> packed context -> agent
```

The latter reduces exploration turns and protocol overhead. GrapeRoot still leaves an MCP escape hatch for cases where deeper exploration is needed.

Its `dgc`/`graperoot` launcher is also instructive: it prepares the local environment, starts a local graph/MCP service, configures agent integration/hooks and launches the existing coding agent. This is strong prior art for an **EOKS sidecar/control layer around an existing agent**.

The public repository contains the launcher/tooling, while the core graph engine is proprietary. Therefore EOKS should treat its architecture as observed prior art and benchmark its claims rather than depending on or reproducing inaccessible internals.

### GrapeRoot's EOKS boundary

GrapeRoot primarily addresses:

- structural repository understanding;
- task-context retrieval and packing;
- token/read budgets;
- session-aware context weighting;
- local MCP exploration;
- agent launch/integration.

It is not, from the public architecture, a complete system for:

- task assurance policies;
- selecting among heterogeneous evidence providers;
- model/reasoning-strategy routing based on policy;
- deterministic outcome evaluation across workflows;
- delayed outcome feedback;
- promotion/invalidation of canonical organizational knowledge.

So EOKS should treat GrapeRoot as a **resource/provider for context compilation and agent integration**, not as an EOKS replacement.

### GrapeRoot versus Graphify

These are complementary examples of two different context architectures:

| | GrapeRoot | Graphify |
|---|---|---|
| Primary role | proactive context optimization | structural graph/navigation |
| Delivery | pre-injected context | agent-queryable graph/MCP |
| Session memory | explicit | not its defining role |
| Structural graph | internal input to context packing | central product output |
| Agent integration | launcher + hooks + MCP | MCP/query integration |
| EOKS layer | context/execution | knowledge/evidence |

This is not an argument that pre-injection is universally better. A useful EOKS experiment should compare proactive, reactive and hybrid context strategies on cost, latency, evidence coverage and task outcome.

## Graphify

Graphify is a particularly strong example of the structural representation discussed in EOKS. Its current implementation uses local Tree-sitter-based parsing for code, produces an interactive `graph.html`, a human-readable `GRAPH_REPORT.md` and a queryable `graph.json`, and distinguishes extracted from inferred relationships. It can also expose the graph through MCP and install guidance/hooks for coding assistants.

This makes it valuable for three separate reasons:

1. **visualization** — a human can see the shape of a codebase;
2. **navigation/impact analysis** — an agent can query relationships instead of rediscovering them through grep;
3. **incremental/compiler thinking** — the graph can serve as a dependency representation used to identify affected areas.

A recent experiment exposed an important boundary: Graphify can provide structural relationships without proving a value-flow invariant. A workspace value that reaches a persistence sink only becomes safe after crossing a scope-stamp/masked-secret barrier; answering whether the value can bypass that barrier is a semantic dataflow question. This is not a reason to turn Graphify into a security scanner. It is evidence that EOKS needs to compose structural graphs with specialized semantic evidence providers.

Graphify should still not be equated with "the knowledge base". The graph is one representation, especially strong for structural questions.

## Understand Anything

Understand Anything is a strong adjacent system because it combines deterministic parsing with LLM analysis to produce an interactive knowledge graph, including structural and business-domain views, guided tours and change-impact capabilities. It is explicitly positioned for both humans and coding agents.

An important implementation detail is its auto-update hook path: it can detect when the stored graph was built from a different Git commit and prompt an update, while its chat/diff skills are instructed to check graph freshness and read only the graph sections needed.

This is valuable EOKS prior art for **incremental knowledge representations** and for the principle that agents should query a representation selectively rather than dump the entire graph into context. Its issue tracker also contains an open discussion about incremental updates consuming more tokens than the initial build, which is a useful warning: incremental does not automatically mean cheap.

## TrueCourse

TrueCourse is best understood as **architecture/code intelligence plus specification-to-behavior assurance**, not primarily as a documentation generator. It combines deterministic rules with LLM review and has a separate `spec -> scenario tests -> guard` workflow for detecting when implementation drifts from documented behavior. Results are stored locally in `.truecourse/` as JSON artifacts.

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

## Xirp / Spotify

[Xirp](https://xirp.spotify.com/) is particularly relevant to EOKS because it makes **institutional/system context** explicit as a coding-agent concern.

Its public site describes an agentic development environment that connects coding sessions to services, ownership, dependencies, documentation and architectural decisions. It also describes a Portal-based Workspace for sharing work items, sessions and docs, and a mechanism for turning knowledge from coding sessions into living documentation for future engineers or agents. Xirp is presented as a beta and as a harness that can work with Claude, Gemini and Codex.

The EOKS-relevant insight is:

> **A coding agent needs context about the system and organization around the code, not just the files in front of it.**

This adds a useful dimension to the existing knowledge model:

```text
service ownership       -> organizational evidence
service dependencies    -> structural evidence
architecture decisions  -> historical/canonical evidence
session discoveries     -> candidate durable knowledge
code/docs               -> authoritative implementation evidence
                    |
                    v
             context compilation
                    |
                  agent
```

Xirp therefore spans the **context + execution boundary**, with a particularly strong emphasis on system awareness and shared session continuity. It is not itself the EOKS control plane, and its generated documentation should not be treated as automatically canonical truth.

The most important EOKS caution is the knowledge lifecycle:

```text
session
  -> observations / decisions / artifacts
  -> candidate knowledge
  -> validation + provenance + scope
  -> promotion / update / invalidation
  -> future retrieval
```

Not every piece of session reasoning deserves to survive as institutional knowledge. A generated summary can be useful evidence while still being wrong, temporary or stale.

### Xirp versus Graphify

These projects illustrate complementary meanings of system understanding:

| | Xirp | Graphify |
|---|---|---|
| Primary focus | System/organizational context for coding agents | Structural code/repository representation |
| Ownership / service context | Central to its public positioning | Not its primary focus |
| Architectural rationale | Explicitly part of system context | Not inherently |
| Code relationships | Part of broader system understanding | Core strength |
| Session continuity | Core part of the product story | Not its defining role |
| Context assembly | Central | Primarily evidence/navigation |
| Deep dataflow/invariants | Not the primary public positioning | Structural graph is not proof of deep semantic flow |

The EOKS conclusion is **composition rather than competition**: a context compiler could combine Xirp-like organizational/system evidence with Graphify-like structural evidence and authoritative source material.

See [the dedicated Xirp research note](../research/prior-art/xirp.md) for the detailed mapping and proposed experiments.

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

Liza explores hardened multi-agent coding with an emphasis on quality, auditability, automated reviews and documentation, including ADR-oriented workflows.

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

## OpenWolf

OpenWolf is relevant to a concrete coding-agent failure mode: repeated reconstruction of repository context across sessions. Its local, hook-driven approach maintains repository-oriented summaries and persistent notes around agent/file interactions.

For EOKS, OpenWolf is best treated as **knowledge extraction + context optimization**, not as the complete knowledge layer. Generated memory should be evidence-bearing and subject to freshness, provenance and promotion rules.

## CodeSight

CodeSight is relevant to codebase context and repository understanding. It illustrates the value of preparing structured knowledge for coding agents and fits primarily in the evidence/navigation side of EOKS.

## Deterministic analysis: structural, dataflow and invariant tools

These tools reinforce an important architectural rule: deterministic questions should be answered deterministically whenever possible, but **analysis depth should match the question**.

| Mechanism | Strongest questions |
|---|---|
| TypeScript compiler/types | Can an invalid state be represented at all? |
| ESLint | Does code violate a local structural/policy rule? |
| Tree-sitter/language tooling | What are the symbols, syntax and basic relationships? |
| Graphify | How are entities structurally connected? |
| ts-morph / compiler API | Can we implement a narrow TypeScript-specific semantic check? |
| Semgrep | Does a pattern or source-to-sink flow violate a rule? |
| CodeQL | Can a rich query/dataflow model establish a difficult semantic or security property? |

The motivating source→barrier→sink case belongs to the last three layers, but the preferred implementation is not automatically the deepest tool. If a TypeScript type invariant can prevent the invalid state, that is better than scanning for it. If a small ESLint or compiler-API check is sufficient, a repository-wide dataflow engine is unnecessary.

See [Software analysis, dataflow and invariants](software-analysis.md).

## Evidence-provider abstraction

The tooling landscape suggests a useful EOKS abstraction: an **evidence provider** answers a bounded question and returns evidence with provenance, scope/revision, validation/confidence and cost/latency characteristics.

Examples:

```text
repository graph     -> dependency evidence
Tree-sitter          -> structural evidence
TypeScript compiler  -> type/invariant evidence
ESLint               -> local policy evidence
ts-morph              -> targeted semantic evidence
Semgrep              -> pattern/dataflow evidence
CodeQL               -> deep dataflow/security evidence
Graphify             -> graph/navigation evidence
Understand Anything  -> synthesized code/domain evidence
TrueCourse           -> architecture/spec compliance evidence
Xirp-like provider   -> organizational/system/session context
GrapeRoot            -> task-context optimization
OpenWolf             -> interaction-derived project context
tests                -> behavioral evidence
observability        -> runtime evidence
CLAUDE.md / ADRs     -> canonical human-reviewed knowledge
```

The control plane should choose the minimum sufficient set of providers for the workload instead of always running the deepest analysis or retrieving the broadest organizational context.

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
      +------------------+-----------------------------+
      |                  |                             |
 canonical docs      structural                  organizational /
 CLAUDE.md / ADRs       graph                    system context
      |                  |                       ownership / services
      |                  |                             |
      +------------------+-----------------------------+
                         |
                    semantic /
                    dataflow /
                    invariant evidence
                         |
                       model
                         |
                 execution + tools
                         |
                    evaluation
                         |
                      outcome
                         |
                      feedback
```

The newer synthesis adds an explicit control-plane boundary around this stack:

```text
Task
  -> Policy
  -> Context compiler
  -> Resource selection
  -> Run
  -> Evaluation
  -> Outcome
  -> Feedback
```

This is deliberately compositional. No single project needs to become EOKS.
