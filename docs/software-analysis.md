# Software analysis, dataflow and invariants

Software engineering exposes an important class of EOKS problems that sits between ordinary code navigation and full security analysis: **semantic invariants over execution paths**.

A code graph can tell us that a workspace variable is related to a persistence function. It may not be enough to establish that every value reaching that sink has crossed the required safety condition. This is a path/dataflow question.

## The motivating pattern

Consider a simplified invariant:

```text
source:   unscoped / blank-secret workspace value
sink:     workspace persistence
barrier:  scope stamp or masked-secret guard

required:
  source must not reach sink without crossing the barrier
```

The interesting property is therefore not merely:

> Does `persistWorkspace` reference or call the code that establishes the scope?

It is:

> Can a value originating in the unsafe state reach the persistence sink along a path on which the required barrier has not been established?

This is a **taint/dataflow-with-a-barrier** problem. Security scanners use similar machinery for source → propagator → sanitizer/barrier → sink analysis, but the same abstraction applies to architectural and correctness invariants that are not security vulnerabilities.

## Four useful layers

EOKS should distinguish several kinds of code understanding:

```text
source code
    |
    +--> structural representation
    |       files, symbols, calls, imports, types
    |
    +--> semantic/dataflow analysis
    |       values, paths, barriers, sinks, propagation
    |
    +--> invariant/policy layer
    |       "what must always be true?"
    |
    +--> agent reasoning
            interpretation, repair, synthesis
```

### Structural graph

Answers questions such as:

- What calls this function?
- What imports this module?
- What depends on this interface?
- What changed around this symbol?

Graphify is a strong example of this representation. Its graph is evidence and navigation infrastructure, not a complete semantic model of program execution.

### Semantic/dataflow analysis

Answers questions such as:

- Can this value reach that sink?
- Which paths can reach the operation?
- Did a value cross a required transformation?
- Does a guard dominate the relevant use?

This requires more than simple graph connectivity when aliases, assignments, branches, functions or transformations matter.

### Invariants

An invariant describes the property we care about independently of the implementation mechanism. A useful abstract form is:

```yaml
name: workspace-persistence-requires-scope
source: unscoped_workspace
sink: persist_workspace
barrier: establish_scope
rule: source_must_not_reach_sink_without_barrier
```

The same invariant might eventually be enforced by the type system, an ESLint rule, a small TypeScript analyzer, Semgrep, CodeQL, tests, or a combination of these.

This separation is important: **the invariant is the EOKS concept; the analyzer is an evidence provider.**

## Prevention versus detection

When an invariant can be represented in the type system, prevention is preferable to post-hoc scanning.

For example, a conceptual TypeScript API can distinguish an unscoped value from a scoped value:

```ts
type UnscopedWorkspace = { kind: "unscoped" /* ... */ }
type ScopedWorkspace = { kind: "scoped" /* ... */ }

function establishScope(value: UnscopedWorkspace): ScopedWorkspace {
  // ...
}

function persistWorkspace(value: ScopedWorkspace) {
  // ...
}
```

The compiler then becomes the barrier checker: an unscoped workspace cannot be passed to `persistWorkspace` without first obtaining the scoped type.

Not every invariant can be expressed this way. Existing code may represent state with optional fields, mutable variables, aliases or implicit conventions. In those cases targeted static analysis can provide detection until the API can be redesigned.

The preferred progression is therefore:

```text
make the invariant explicit
        |
        +--> encode in types when practical
        |
        +--> use lightweight static checks when needed
        |
        +--> use general dataflow analysis only when complexity justifies it
```

## Tool positioning

These tools overlap, but they solve different problems and have different cost profiles.

| Tool / mechanism | Typical context | EOKS role |
|---|---|---|
| TypeScript compiler/types | everyday TypeScript development | compile-time invariant enforcement |
| ESLint | local project rules and fast feedback | lightweight policy/pattern checks |
| ts-morph / TypeScript compiler API | custom TypeScript analysis and refactoring | targeted project-specific analyzer |
| Tree-sitter / language tooling | syntax and structural extraction | structural evidence |
| Graphify | code graph, navigation and scoped structural context | graph evidence/navigation |
| Semgrep | security, bug patterns and custom static-analysis rules | configurable pattern/dataflow evidence |
| CodeQL | deep queryable program/dataflow analysis, especially security | high-power semantic evidence |

### Semgrep

Semgrep is the closest general-purpose fit when an invariant naturally maps to source → propagation → barrier/sanitizer → sink. It is useful when the project has several such rules or when the rules need to be shared and run consistently.

It should not automatically become an EOKS dependency. If one project-specific invariant can be expressed more simply as a TypeScript type or small check, a full taint-analysis setup may add unnecessary complexity.

### CodeQL

CodeQL is especially strong for deep, queryable program analysis and vulnerability/code-scanning workflows. Its dataflow/path machinery is relevant to EOKS, but introducing it solely for a small architectural invariant is usually disproportionate.

CodeQL should therefore be considered a later escalation path when analysis repeatedly requires rich interprocedural dataflow, complex predicates, aliases, path explanations or a broad query corpus.

### ts-morph / custom TypeScript analysis

A small TypeScript-specific analyzer can be the pragmatic middle ground. For example, it could find calls to a persistence function, resolve the argument symbol, follow assignments and check whether the required scope state was established.

The warning sign is scope creep: once the analyzer needs sophisticated interprocedural flow, alias analysis and control-flow semantics, the project is effectively rebuilding a dataflow engine. At that point Semgrep or another mature analyzer may be the better evidence provider.

## Do not turn Graphify into a security scanner

A missed invariant does not mean that a structural graph is the wrong abstraction. It identifies a boundary between structural knowledge and semantic execution evidence.

A useful composition is:

```text
                 codebase
                    |
          +---------+---------+
          |                   |
     structural            semantic
       graph               analysis
          |                   |
          +---------+---------+
                    |
                invariants
                    |
              agent reasoning
                    |
          repair / verification
```

Graphify can remain responsible for efficient structural navigation while specialized analyzers answer deeper bounded questions. The control plane can select the minimum sufficient evidence provider for a task.

## Invariants as first-class knowledge

An invariant is more durable than a single analyzer result. A failed analysis says:

> This revision contains a path that appears to violate rule X.

The invariant says:

> Rule X is part of the system's intended behavior.

That distinction creates a useful lifecycle:

```text
agent discovers repeated pattern
          |
    candidate invariant
          |
   human / evidence review
          |
   invariant + provenance
          |
   compile to one or more
   enforcement mechanisms
          |
      verification
          |
   learned / maintained rule
```

The agent should not silently turn an observed pattern into policy. Invariants need scope, provenance, confidence and a way to be corrected when the architecture changes.

## Evidence-provider principle

EOKS should treat static analyzers as **evidence providers**, not as architectural authorities.

A provider should ideally report:

- the question/rule it evaluated;
- source and sink or other relevant entities;
- the path or structural evidence supporting the result;
- repository revision and analysis configuration;
- whether the result is deterministic, inferred or heuristic;
- cost/latency and coverage limitations.

The control plane can then choose the minimum sufficient provider. A simple type check should not trigger a repository-wide deep analysis, just as a structural graph query should not be expected to prove a value-flow invariant.

## Practical strategy for EOKS

For the current research/prototype stage:

1. Keep Graphify as structural graph/navigation evidence.
2. Represent important architectural invariants explicitly before choosing an analyzer.
3. Prefer TypeScript type invariants when the API can make invalid states unrepresentable.
4. Use ESLint or a small `ts-morph`/compiler-API check for narrow project-specific rules.
5. Introduce Semgrep when multiple invariants genuinely require taint/dataflow semantics.
6. Treat CodeQL as a later option for deep or broad semantic analysis, not as the default.
7. Feed analyzer results into context/evaluation rather than making the analyzer itself the knowledge layer.

The hypothesis to test is not "which static-analysis tool is best?" It is:

> **Can EOKS select the cheapest reliable evidence source that is sufficient to answer the current engineering question, and can repeated evidence be promoted into explicit, enforceable invariants?**
