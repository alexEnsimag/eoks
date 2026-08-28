# EOKS for software engineering

Software engineering is a particularly useful proving ground because the system has rich structured evidence: source code, dependency graphs, commits, tests, static analysis and runtime behavior.

## Deterministic + probabilistic reasoning

Tools such as compilers, language servers, Tree-sitter, tests, code graphs, Semgrep, CodeQL and targeted dataflow analysis can answer narrow questions with stronger guarantees than an LLM. The LLM should consume those results when they provide better evidence than inference from raw source.

The important distinction is between **structural evidence** and **semantic/dataflow evidence**. A code graph can establish relationships such as calls, imports and dependencies, but some correctness properties are path-sensitive: a value may reach a sink through a path that bypasses a required transformation or guard.

This is the **barrier principle**: when a source-to-sink property depends on crossing a transformation, validation or state-establishing barrier, an agent may need explicit dataflow analysis rather than text retrieval or graph connectivity alone.

## Codebase context

A useful code context can combine:

- relevant files and symbols;
- callers/callees and dependency relationships;
- recent commits and architectural decisions;
- tests and observed failures;
- static-analysis and dataflow findings;
- configuration and deployment constraints;
- explicit architectural invariants and their enforcement status.

The challenge is deciding which projection is appropriate for the task.

## Graphs are not dataflow proofs

Code graphs are valuable when the question is relational: "what reaches this sink?", "what depends on this interface?", or "what changed the behavior?". They are not a universal replacement for source text, type information or semantic analysis.

A useful four-layer model is:

```text
source code
    |
    +--> structural graph
    |       files, symbols, calls, imports, types
    |
    +--> semantic/dataflow analysis
    |       values, paths, propagation, barriers, sinks
    |
    +--> invariant / policy
    |       what must always be true?
    |
    +--> agent reasoning
            interpretation, repair, synthesis
```

Graphify therefore belongs primarily in the structural/evidence layer. A missed path-sensitive invariant should not be treated as evidence that a structural graph is the wrong representation.

## Invariants

An invariant should be represented independently of the tool used to enforce it. For example:

```yaml
name: workspace-persistence-requires-scope
source: unscoped_workspace
sink: persist_workspace
barrier: establish_scope
rule: source_must_not_reach_sink_without_barrier
```

This is more durable than a particular Semgrep or CodeQL rule. Depending on the invariant, enforcement may come from the TypeScript type system, an ESLint rule, a small compiler-API analyzer, Semgrep, CodeQL, tests, or several of these.

### Prefer prevention when possible

If an invariant can be represented in the type system, making invalid states unrepresentable is preferable to discovering violations later. For example, an API can distinguish `UnscopedWorkspace` from `ScopedWorkspace` and make persistence accept only the latter.

When existing code uses mutable variables, optional state, aliases or implicit conventions, a targeted analyzer can provide detection until the API can be redesigned.

The preferred escalation is:

```text
explicit invariant
       |
       +--> type-system enforcement when practical
       |
       +--> lightweight static rule when needed
       |
       +--> targeted TypeScript analysis when needed
       |
       +--> general dataflow analysis when complexity justifies it
```

See [Software analysis, dataflow and invariants](software-analysis.md) for the detailed model and tool positioning.

## Agents

An engineering agent should be evaluated as a workflow, not just as a model response. A good workflow can use a smaller model for routine exploration, deterministic tools for facts, and a stronger model for difficult synthesis.

The same principle applies to analysis depth: a simple type check should not trigger a repository-wide dataflow analysis, and a structural graph query should not be expected to prove a path-sensitive value-flow invariant.

## Key hypothesis

The highest-leverage AI engineering infrastructure may be the layer that decides **what evidence to expose to which model at which point in the workflow**—including choosing the cheapest reliable analysis that is sufficient for the engineering question.
