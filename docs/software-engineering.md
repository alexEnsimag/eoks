# EOKS for software engineering

Software engineering is a particularly useful proving ground because the system has rich structured evidence: source code, dependency graphs, commits, tests, static analysis and runtime behavior.

## Deterministic + probabilistic reasoning

Tools such as CodeQL, Semgrep, language servers, compilers, tests and dataflow analysis can answer narrow questions with stronger guarantees than an LLM. The LLM should consume those results when they provide better evidence than inference from raw source.

This is the **barrier principle**: if a source-to-sink dataflow crosses a transformation or validation barrier, an agent may need explicit structural analysis rather than text retrieval alone.

## Codebase context

A useful code context can combine:

- relevant files and symbols;
- callers/callees and dependency relationships;
- recent commits and architectural decisions;
- tests and observed failures;
- static-analysis findings;
- configuration and deployment constraints.

The challenge is deciding which projection is appropriate for the task.

## Graphs

Code graphs are valuable when the question is relational: "what reaches this sink?", "what depends on this interface?", or "what changed the behavior?". They are not a universal replacement for source text.

## Agents

An engineering agent should be evaluated as a workflow, not just as a model response. A good workflow can use a smaller model for routine exploration, deterministic tools for facts, and a stronger model for difficult synthesis.

## Key hypothesis

The highest-leverage AI engineering infrastructure may be the layer that decides **what evidence to expose to which model at which point in the workflow**.
