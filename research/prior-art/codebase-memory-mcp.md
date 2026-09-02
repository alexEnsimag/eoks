# codebase-memory-mcp

`codebase-memory-mcp` is prior art for **structural evidence acquisition** in coding-agent workflows. It indexes a repository into a persistent knowledge graph and exposes structural queries through MCP, allowing an agent to query relationships such as callers/callees, architecture and impact rather than repeatedly reconstructing them through file-by-file exploration.

## EOKS interpretation

The important EOKS contribution is not “codebase memory” as a new knowledge category. It is a concrete implementation of an existing pattern:

```text
repository
    |
structural analysis / indexing
    |
persistent structural representation
    |
queryable evidence provider
    |
context acquisition / compilation
    |
agent
```

This makes it useful prior art for the distinction between **structural representation**, **evidence provider**, and **compiled context**. A graph remains derived structural evidence; it is not automatically canonical knowledge or semantic truth.

The project reports a benchmark across 31 repositories with 83% answer quality, 10× fewer tokens and 2.1× fewer tool calls versus file-by-file exploration. These are project-reported results and should be treated as evidence to investigate, not as EOKS benchmark results.

## Position relative to EOKS tools

- **Graphify / CodeGraph / GitNexus / Understand Anything:** same broad structural-evidence family, with different parsing, analysis, query and semantic capabilities.
- **GrapeRoot:** complementary context/runtime layer; codebase-memory-mcp can act as a structural evidence provider to a context compiler.
- **CodeSight:** closer to repository-context compilation and targeted evidence views; structural indexing can be an upstream provider.
- **Semgrep / CodeQL:** complementary verification providers. Structural relationships can guide or narrow deeper analysis, while analyzers can establish properties that graph connectivity alone cannot prove.

## Research question

The useful EOKS question is not whether a graph is “better” than exploration in general, but **when deterministic structural indexing materially reduces avoidable discovery work while preserving the semantic exploration needed for correctness**.

See also [Context engineering](../../docs/context.md) and [Knowledge representations](../../docs/knowledge-representations.md).
