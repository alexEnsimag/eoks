# EOKS

> An experimental, programmable control layer for reliable AI engineering.

EOKS explores what an **AI development environment** could look like if context, knowledge, memory, tools, execution, policy, evaluation and learning were coordinated as parts of one system.

The central question is not simply how to give AI agents more context or better tools. It is how to build an environment in which agents can access the right information and capabilities, operate within explicit constraints, produce verifiable outcomes, and improve from experience.

The working architectural hypothesis is an **operating/control layer around existing agents, harnesses and execution providers**. EOKS does not need to replace an IDE, agent runtime, workflow engine, memory system or analyzer. It explores how their capabilities can be composed, observed, governed and improved as one environment.

## The current thesis

The practical goal is **trustable automated software-engineering workflows with AI agents**. An agent does not need to be perfect; the surrounding system needs to make failures detectable, consequential actions appropriately gated, and increasing autonomy justified by evidence.

The four primary engineering outcomes are **velocity, quality, trust/autonomy, and cost/scale**. Risk, cognitive load, explainability, traceability, consistency, maintainability and reproducibility are useful diagnostics supporting those outcomes. See [Engineering outcomes and graduated autonomy](docs/engineering-outcomes.md).

EOKS is not simply a prompt framework, vector database, memory store, agent framework, workflow engine, observability dashboard, model router or code-search tool. These may be components or providers within an EOKS environment; none alone defines the system.

Several distinctions shape the architecture:

- **Knowledge is not context.** Knowledge is governed, reusable meaning; context is the task-specific information materialized for a reasoning step. A larger context window does not guarantee better understanding.
- **The working set is not model context.** A workload's working set is the useful subset of eligible resources and evidence. Context compilation selects, transforms, orders and budgets a representation of that working set for a particular step.
- **Memory is not one flat store.** Knowledge, experience, reusable procedures, documentation and structural representations have different meanings, authority and update lifecycles, even when they share governance mechanisms.
- **Observability is not confidence, and confidence is not control.** Traces and metrics provide evidence. Reliability estimates must be calibrated against outcomes, and consequential actions remain governed by policy and assurance.
- **Execution is a capability, not the whole environment.** A local agent session, workflow runtime, remote worker, sandbox or fleet can realize work. EOKS coordinates the surrounding resources, constraints, evidence and lifecycle without requiring one execution substrate.
- **Learning is governed, not silent mutation.** Run experience can suggest improvements, but promotion into reusable knowledge, procedures or policy should preserve provenance and be validated.

The architecture combines two complementary lenses: operating systems/computer architecture for resource hierarchy, locality, caching, capacity and isolation; and Kubernetes-style reconciliation for desired state, observed evidence, lifecycle and feedback. These are reference architectures, not implementation prescriptions.

## A compact model of the environment

EOKS organizes the problem across six dimensions:

| Dimension | Question |
|---|---|
| **INTENT** | What outcome is wanted, and what constraints apply? |
| **WORKFLOW** | How is work organized, sequenced and resumed? |
| **CAPABILITIES** | Which tools, models and execution capabilities are available? |
| **KNOWLEDGE** | What durable information and evidence can be reused? |
| **STATE** | What persists across steps and executions? |
| **POLICY** | What is allowed, required, gated or subject to escalation? |

These are conceptual dimensions, not six required services. A workload draws on resources and capabilities through a control loop:

```text
 intent + policy + current state
              |
       reconcile / decide
              |
 select eligible resources and capabilities
              |
     acquire + compile context
              |
       execute a bounded run
              |
       observe + evaluate
              |
 update state / verify outcome / escalate
              |
    govern candidate learning for future runs
              +---------------------------> reconcile again
```

A useful environment-level vocabulary is **tiles**: bounded capabilities such as execution, context, knowledge/memory, policy, evaluation, observability, learning, representation and human interaction. Tiles help describe how an environment is composed; they are not new runtime primitives. A tile may be implemented by several resources, and one resource may support several tiles.

The core loop remains deliberately small: **Task → Context → Run → Decision → Evaluation → Outcome**, with Memory and Policy updated only through their appropriate lifecycles. The environment provides the resources, mechanisms and evidence around that loop.

## Architectural principles

1. **Treat context as a managed resource.** Select and compile information for the reasoning step; do not equate storage, working set and model context.
2. **Prefer transformation before accumulation.** First use existing harness boundaries—hooks, skills, tools and scripts—to select, summarize or route authoritative evidence. Add persistent representations when they preserve valuable information or make recurring work materially cheaper or more reliable.
3. **Keep representations plural and traceable.** Graphs, indexes, documents, diagrams, tests and runtime observations serve different queries. Derived representations should retain provenance, freshness and links to authoritative sources; no single graph or memory substrate is mandatory.
4. **Separate deterministic evidence from semantic judgment.** Prefer the cheapest reliable analyzer that answers the question. Use probabilistic judgment for bounded questions when it adds value, and calibrate it before it drives consequential transitions.
5. **Make execution replaceable and state explicit.** Separate logical work and durable execution state from the process, workspace, sandbox, VM or remote resource currently realizing it.
6. **Evaluate the whole outcome.** Token reduction, cache hit rate, model confidence and agent self-reported success are not sufficient objectives. Measure verified task quality, rework, latency, cost and failure modes.
7. **Learn through controlled promotion.** Extract candidate improvements from traces and outcomes; validate, scope and version them before changing reusable behavior or canonical knowledge.
8. **Graduate autonomy according to assurance.** Use tests, independent review, deterministic analysis, policy gates and human escalation according to risk and ambiguity.
9. **Recompute only what has become invalid.** Treat freshness, dependencies, cache identity and invalidation as first-class concerns for derived context and reusable computation.
10. **Keep the semantic model small.** Adopt a new primitive only when observed workloads demonstrate a real gap that cannot be expressed cleanly with existing concepts.
11. **Keep the system model- and provider-agnostic.** Integrate existing agents, harnesses, execution backends, analyzers and knowledge providers before replacing them.
12. **Make experiments reproducible and falsifiable.** Compare interventions against baselines and measure end-to-end outcomes, not attractive proxy metrics alone.

## Documentation

The repository separates **current architecture and decisions** from **exploratory research**. See [the documentation map](docs/README.md) for canonical document ownership and terminology.

### Architecture and design

- [Vision](docs/vision.md)
- [Architecture](docs/architecture.md)
- [Domain model and architecture diagrams](docs/domain-model.md) — canonical system map, dimensions, domain model and control-loop views.
- [Engineering outcomes and graduated autonomy](docs/engineering-outcomes.md) — outcomes, assurance, autonomy levels and experiment metrics.
- [Resource model](docs/resource-model.md) — resources, Asset, Provider, Representation, Loadout and Context.
- [Context engineering](docs/context.md) — working set, context acquisition/compilation, harness-mediated transformation and derived computation.
- [Context Workbench architecture](docs/context-workbench.md)
- [Knowledge base](docs/knowledge-base.md)
- [Knowledge as a multi-representation system](docs/knowledge-representations.md)
- [Working representations](docs/working-representations.md)
- [Agent workflows, orchestration and reasoning strategies](docs/agent-workflows.md)
- [Memory](docs/memory.md) — semantic, episodic, project and procedural memory plus controlled behavioral learning.
- [Control plane](docs/control-plane.md)
- [Evaluation](docs/evaluation.md)
- [Software engineering](docs/software-engineering.md)
- [Software analysis, dataflow and invariants](docs/software-analysis.md)

### Governance and research management

- [Research and synthesis methodology](docs/methodology.md) — how formal research, community evidence, existing systems and experiments become testable engineering knowledge.
- [Architectural decisions](docs/decisions.md)
- [Research agenda](docs/research-agenda.md)
- [Open questions](docs/open-questions.md)
- [Prior art](docs/prior-art.md)
- [Terminology](docs/terminology.md)
- [History](docs/history.md)

## Research

The [research corpus](research/) preserves exploratory reasoning, comparisons, evidence and experiments. It is intentionally less normative than `docs/`; a research note does not automatically constitute an architectural decision. See the [research index](research/README.md) for the full map.

Selected syntheses and prior art:

- [AI development environment](research/ai-development-environment.md) — the environment/control-layer framing, tiles and boundaries between harness, SDLC and execution.
- [Agentic development environments](research/prior-art/agentic-development-environments-2026.md) — current ADE implementations and the distinction between Work, Run, Workspace, Agent, provider session and Outcome.
- [Google's agentic engineering SDLC](research/prior-art/google-agentic-engineering-sdlc-2026.md) — context, skills, harness evaluation, proactivity and execution state.
- [Knowledge, context and the EOKS control plane](research/knowledge-context-control-plane.md)
- [Knowledge, memory and context graphs](research/prior-art/knowledge-memory-context-graph.md) — semantic boundaries and decision provenance without graph-specific runtime primitives.
- [Durable engineering context](research/durable-engineering-context.md) — lifecycle, freshness, context rot, learning and visual/multi-representation context.
- [Context acquisition comparison](research/context-acquisition-comparison.md) — raw exploration, retrieval, graphs, delegated exploration and persistent structure.
- [Context evaluation](research/context-evaluation.md) — controlled methodology for evaluating context interventions and durable knowledge.
- [Community evidence and bottlenecks](research/community-evidence-bottlenecks.md) — academic evidence, practitioner signals, conflicting results and prioritized experiments.
- [Validated and reusable computation](research/validated-reusable-computation.md) — provenance, evaluation, dependency-aware reuse and incremental recomputation.
- [Incremental context maintenance](research/prior-art/incremental-context-maintenance.md)
- [LLM context caching](research/prior-art/llm-context-caching.md) — computational cache reuse versus semantic result reuse, identity, scope and validity.
- [Enforceable architecture, ADRs, fitness functions and SDD](research/prior-art/enforceable-architecture.md)
- [Probabilistic decision primitives / Jev](research/prior-art/probabilistic-decision-primitives-jev-2026.md) — bounded semantic decisions inside deterministic control loops.
- [Spotify Portal / Shunt](docs/prior-art/spotify-portal-shunt.md) — harness-mediated context transformation before information reaches the frontier model.
- [TencentDB Agent Memory](research/prior-art/tencent-agent-memory.md) — multi-resolution memory, Skills, Wiki, CodeGraph and governed context delivery.
- [Code graph and repository-context tooling](research/prior-art/code-graph-landscape.md) — structural evidence providers and their relationship to context compilation.

## Practical direction

The current implementation hypothesis is conservative and evidence-led:

- **Start at existing harness boundaries.** Use hooks, skills, scripts and tools to acquire, filter, summarize and route information before introducing new stores or a replacement agent runtime.
- **Keep canonical knowledge reviewable.** Use hierarchical `CLAUDE.md` files and a small number of Markdown/ADR documents where they fit; preserve ownership, scope, provenance, freshness and lifecycle for reusable knowledge and procedures.
- **Compose evidence providers.** Prefer deterministic source, type, test and analysis evidence when sufficient; use structural graphs, repository maps, organizational context or deeper analyzers only when the workload benefits from them.
- **Make context inspectable.** Track what was selected, why it was selected, its source and freshness, and how context changes across reasoning steps. Compare reactive retrieval, proactive delivery and hybrid approaches.
- **Keep execution topology small.** Start with logical conductor, executor and independent reviewer roles where useful. Use native subagents or isolated sessions when they offer a concrete benefit; add persistent orchestration only when lifecycle, policy or cross-session coordination requires it.
- **Make verification part of the workflow.** Tests, static analysis, independent review and observed behavior provide completion evidence. An agent's own claim is not enough.
- **Treat learning as a governed pipeline.** Extract candidate improvements from traces and outcomes, then validate and promote them explicitly rather than silently rewriting canonical knowledge.
- **Measure real outcomes.** Compare interventions using verified quality, rework, velocity, cost, latency and failure modes. Calibrate reliability signals before they control stopping, branching, retries or model selection.
- **Keep derived artifacts fresh efficiently.** Track source revisions and dependencies, reuse valid computations and invalidate or recompute affected results instead of rebuilding everything after each change.
- **Separate work from its substrate.** Keep logical Work and execution state distinct from the process, workspace, toolkit, container, VM or remote backend that realizes it; enforce isolation and resource constraints at the appropriate layer.
- **Promote autonomy only with assurance.** Use policy gates and human escalation for consequential, risky or ambiguous actions that current evidence cannot bound.
- **Keep the ontology small.** The environment can be broad and composable without turning every provider, representation, tile or technique into an EOKS primitive.

This gives EOKS a path from a repository-level workflow to a richer AI development environment without requiring a graph database, a new canonical format or a new agent runtime on day one.

## Status

Research / architecture / prototype stage. The architecture is intentionally provisional.

This repository is a **living specification and laboratory**: ideas should become documented hypotheses, experiments, implementations and decisions rather than a transcript of discussions.
