# EOKS

> An experimental operating/control layer for reliable AI workloads.

EOKS explores what an **AI operating system / control plane** could look like if context, memory, models, tools, tasks, execution, evaluation and observability were treated as coordinated resources.

The project grew out of a recurring question: **why does giving models more context not necessarily give them better understanding?** The working answer is that AI systems need infrastructure for selecting, structuring, persisting, evaluating and routing information—not simply larger prompts.

## The current thesis

EOKS is not a prompt framework, vector database, memory store, agent framework or model router. Those are potential components. The proposed abstraction is the layer that coordinates them around a workload and continuously learns from outcomes.

A recent refinement is important: **knowledge is not a graph, and context engineering is not the knowledge layer**. EOKS should maintain multiple representations of engineering reality and compile task-specific context from them.

A second refinement is equally important: **observability is not confidence, and confidence is not control**. Tracing and monitoring systems provide execution evidence; EOKS can combine that evidence with model-level uncertainty, deterministic checks and outcomes to estimate workload-specific reliability and decide what to do next.

A third refinement is that **memory is not one flat store**. Agent systems increasingly separate experience-derived memory, reusable procedures, documentation/knowledge and structural code representations. EOKS should preserve those semantic distinctions while giving them a common lifecycle boundary for provenance, scope, freshness, access, versioning and evaluation.

The current architectural hypothesis is deliberately compositional: OKF can provide durable structured knowledge; Graphify-like systems can provide structural evidence; **CodeSight-like systems can provide deterministic repository context and targeted evidence views**; GrapeRoot-like systems can provide proactive context optimization around existing coding agents; TencentDB Agent Memory demonstrates multi-resolution memory, Skills, Wiki and CodeGraph as governed reusable resources; specialized analyzers and tests can provide deterministic evidence; and EOKS can coordinate these resources through task, context, run, decision, policy, evaluation and outcome primitives. The canonical vocabulary for how these resources relate is in [Resource model](docs/resource-model.md).

```text
                         EOKS CONTROL PLANE
              scheduling · policies · resource selection
                                  |
        +-------------------------+-------------------------+
        |                         |                         |
       RESOURCES             CONTEXT PLANE            EXECUTION PLANE
  memory · skills · docs    selection · assembly      workflows · agents
  graphs · decisions        ranking · compression     reasoning strategies
  evidence                  progressive disclosure    tools · artifacts
        |                         |                         |
        +-------------------------+-------------------------+
                                  |
                         EVALUATION / FEEDBACK
                evidence · quality · reliability · outcomes
                                  |
                           OBSERVABILITY
          traces · cost · latency · provenance · uncertainty
```

## Documentation

The repository separates **current architecture** from **exploratory research**. See [`docs/README.md`](docs/README.md) for the document map and ownership rules.

### Architecture and design

- [Vision](docs/vision.md)
- [Architecture](docs/architecture.md)
- [Resource model](docs/resource-model.md) — canonical vocabulary for reusable resources, Asset, Provider, Representation, Loadout and Context.
- [Context engineering](docs/context.md)
- [Context Workbench architecture](docs/context-workbench.md)
- [Knowledge base](docs/knowledge-base.md)
- [Knowledge as a multi-representation system](docs/knowledge-representations.md)
- [Agent workflows and reasoning strategies](docs/agent-workflows.md)
- [Memory](docs/memory.md)
- [Behavioral memory and learning](docs/behavioral-memory.md)
- [Control plane](docs/control-plane.md)
- [Evaluation](docs/evaluation.md)
- [Software engineering](docs/software-engineering.md)
- [Software analysis, dataflow and invariants](docs/software-analysis.md)

### Governance and research management

- [Architectural decisions](docs/decisions.md)
- [Research agenda](docs/research-agenda.md)
- [Open questions](docs/open-questions.md)
- [Prior art](docs/prior-art.md)
- [Terminology](docs/terminology.md)
- [History](docs/history.md)

## Research

The [`research/`](research/) directory preserves exploratory reasoning, comparisons and experiments. It is intentionally less normative than `docs/`; research notes may contain competing hypotheses. See [`research/README.md`](research/README.md) for its map.

- [Knowledge, context and the EOKS control plane](research/knowledge-context-control-plane.md) — synthesis of the recent OKF, GrapeRoot and Graphify discussion, including the proposed minimal runtime model and the boundary between knowledge, context, execution and control.
- [CodeSight](research/prior-art/codesight.md) — deterministic repository context, persistent wiki/knowledge maps, MCP evidence queries and how CodeSight fits as an evidence/context provider rather than a control plane.
- [TencentDB Agent Memory](research/prior-art/tencent-agent-memory.md) — multi-resolution memory, Skills, Wiki, CodeGraph, governance/loadouts and hybrid proactive/on-demand context delivery.
- [Agent code understanding and architecture tooling](research/agent-code-understanding-and-architecture.md) — how repository knowledge graphs, deterministic analysis, architecture governance and AI coding tools can map onto EOKS.
- [Context engineering](research/context-engineering.md)
- [Context quality model](research/context-quality-model.md)
- [Context workbench](research/context-workbench.md)
- [Evaluation and model switching](research/evaluation-and-model-switching.md)
- [LLM observability and reliability signals](research/llm-observability-and-reliability.md) — how tracing, model-level uncertainty, external evidence and calibration can become sensors for the EOKS control loop.
- [Control loop](research/control-loop.md)
- [Learning from development sessions](research/session-learning.md)
- [Agent memory and continuous-learning prior art](research/prior-art/agent-memory.md)
- [Claude Code learning stack](research/claude-learning-okf-hindsight.md) — decomposition of `CLAUDE.md`, Skills, Hooks, memory engines, OKF and optional graph/retrieval systems.
- [Hindsight and OKF](research/prior-art/hindsight-and-okf.md) — why persistent learning engines and portable knowledge formats are complementary layers.
- [Xirp / Spotify](research/prior-art/xirp.md) — institutional/system context, shared coding-session continuity and living documentation as coding-agent prior art.
- [Tool notes](research/tool-notes.md)

## Practical coding-agent direction

The current practical hypothesis is intentionally conservative:

1. Use hierarchical `CLAUDE.md` files as human-reviewable, canonical package/project knowledge and policy where appropriate.
2. Keep cross-cutting architectural rationale in a small number of Markdown/ADR documents.
3. Derive structural information automatically with deterministic analysis and optional graphs; **CodeSight is useful prior art for compiling repository structure into persistent, targeted, agent-readable evidence.**
4. Treat organizational/system context—such as ownership, service boundaries and architectural rationale—as another evidence source when the task requires it; do not assume repository-local context is sufficient.
5. Maintain reusable memory, Skills and knowledge resources with explicit provenance, scope, freshness, ownership/access and lifecycle rather than treating them as one undifferentiated memory store.
6. Use an agent/task loadout to constrain which resources are eligible for a workload before context relevance is optimized.
7. Use context compilation to select only task-relevant evidence from the eligible resource/evidence set.
8. Treat context as an inspectable, budgeted artifact; the proposed Context Workbench provides blocks, provenance, selection rationale, context diffs and a graph view without requiring manual curation for every task.
9. Treat agent workflows and reasoning strategies as execution-layer resources.
10. Learn continuously through candidate extraction and controlled promotion rather than silently rewriting canonical knowledge.
11. Update representations incrementally; do not rebuild the entire knowledge base after every change. Derived repository maps should carry source revision/freshness information.
12. For software engineering, prefer the cheapest reliable deterministic evidence source that answers the question: types before custom analysis, lightweight rules before deep dataflow, and deeper analyzers only when the invariant requires them.
13. Treat observability traces as sensors for the control loop. Combine operational metrics, model uncertainty, evidence agreement, deterministic checks and outcomes rather than trusting a single model confidence number.
14. Calibrate reliability signals against actual task outcomes before allowing them to drive automatic stop/verify/branch/model-switch decisions.
15. Prefer an integration/sidecar model around existing coding agents before building a new agent runtime; the GrapeRoot launcher architecture is useful prior art for this approach.
16. Compare proactive, reactive and hybrid context delivery rather than assuming that pre-injection or tool-driven exploration is universally better. CodeSight's index/article/MCP pattern is a useful example of progressive disclosure.
17. Keep the EOKS semantic model small until real run traces demonstrate that additional primitives are necessary.

This gives EOKS a path from a simple Git repository to richer knowledge infrastructure without requiring a graph database or a new canonical format on day one.

## Status

Research / architecture / prototype stage. The architecture is intentionally provisional.

This repository is a **living specification and laboratory**: ideas should become documented hypotheses, experiments, implementations and decisions rather than a transcript of discussions.
