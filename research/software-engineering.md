# Software-engineering workload research

Software engineering is a useful proving ground for EOKS because repositories contain rich structure, deterministic signals and long-lived project knowledge. It also exposes where LLM-only approaches are weak.

## Code understanding

A coding agent needs more than the files currently open. Useful knowledge can include:

- repository structure;
- symbols and references;
- dependency relationships;
- call/data flow;
- tests;
- configuration;
- historical changes;
- architecture decisions;
- build/runtime behavior;
- issue and task context.

This is naturally a context-management problem, but it is also a knowledge-graph and analysis problem.

## Deterministic tools versus LLMs

We discussed whether tools such as CodeQL or Semgrep are overkill for agentic code understanding. The conclusion was not that they should always be used or avoided.

They solve a different problem. Deterministic/static analysis can establish structural facts that an LLM should not be expected to infer reliably from text alone.

An EOKS workload can therefore combine:

```text
                 task
                   |
          +--------+--------+
          |                 |
   deterministic         LLM reasoning
     analysis                 |
          |                   |
          +--------+----------+
                   |
                evidence
                   |
               evaluation
```

The control layer can decide when the expensive analysis is justified.

## Taint/dataflow example

A concrete discussion involved an agent diagnosing a missed fix as a “taint/dataflow-with-a-barrier” problem.

The useful abstraction was:

- **source** — a workspace value being constructed, including a blank/secret-related value;
- **transformation** — intermediate operations that may preserve or alter the value;
- **barrier** — a transformation that breaks or obscures a naive textual relationship;
- **sink** — persistence or another security-sensitive destination.

This illustrates why textual search can miss a real dataflow relationship. The interesting EOKS role is not to replace a dataflow engine with an LLM, but to orchestrate specialized analysis and reason over its results.

## Graphify, CodeQL and Semgrep

Graph-oriented and static-analysis tools can produce different forms of evidence:

- graph structure;
- symbol relationships;
- control/data flow;
- taint propagation;
- security findings;
- semantic search.

The EOKS question is: **what is the cheapest sufficient analysis for this task?**

A scheduler could choose simple search for a simple task, graph traversal for dependency questions, static analysis for security/dataflow questions, and LLM reasoning where interpretation is required.

## Code as an EOKS testbed

Software engineering makes several EOKS ideas measurable:

- context selection can be evaluated against patch correctness;
- memory can be evaluated against recurring repository tasks;
- model selection can be evaluated on a fixed task corpus;
- deterministic tools provide independent evidence;
- regressions can be validated by tests;
- provenance can be traced to source files and commits.

This makes coding agents a particularly useful first workload for an EOKS prototype, even if EOKS ultimately targets broader AI workloads.