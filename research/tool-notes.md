# Tool and project notes

This note is a **capability map**, not a second source of detailed architecture or methodology. Individual projects are prior art; EOKS does not assume that any of them are dependencies.

For the canonical capability/selection model, see [Tool capability model](../docs/tool-capability-model.md) and [Tool selection](../docs/tool-selection.md). For canonical terminology and resource boundaries, see [Resource model](../docs/resource-model.md). For detailed context treatment, see [Context engineering](../docs/context.md). For knowledge representations, see [Knowledge representations](../docs/knowledge-representations.md). For evaluation, see [Context evaluation](context-evaluation.md) and [Evaluation, reliability and model switching](evaluation-and-model-switching.md).

## Capability map

| Project / family | Primary EOKS-relevant capability | Where it fits |
|---|---|---|
| **GrapeRoot** | repository graph + context assembly around an agent | context compilation / runtime adapter |
| **TencentDB Agent Memory** | multi-resolution memory + Skills + Wiki + CodeGraph resources, governance and agent loadouts | reusable resources / memory & knowledge infrastructure |
| **OKF** | portable Markdown + YAML-frontmatter knowledge bundle | knowledge representation |
| **Graphify** | code relationship/structure graph | structural evidence provider |
| **Codebase Memory MCP** | persistent repository structural graph + MCP query interface for code intelligence | structural evidence provider / context acquisition |
| **CodeSight** | repository code-understanding/context generation | evidence provider |
| **Understand Anything** | code knowledge graph and impact-oriented exploration | structural/semantic evidence provider |
| **Semgrep** | deterministic pattern and dataflow analysis | verification/evidence provider |
| **CodeQL** | deep query and interprocedural/dataflow analysis | verification/evidence provider |
| **TypeScript compiler / ESLint / ts-morph** | lightweight to targeted project-specific static analysis | verification/evidence provider |
| **TrueCourse** | architecture and behavioral guards | assurance / policy enforcement |
| **Superpowers** | structured development workflow and quality gates | execution policy |
| **modularity** | architecture-oriented analysis | architecture evidence/evaluation |
| **Conductor-style systems** | task decomposition and cross-task orchestration | orchestration |
| **`CLAUDE.md` management** | human-reviewed project policy/knowledge | canonical local knowledge |
| **memsearch / LangMem / Mem0 / Zep** | persistent semantic/episodic/procedural memory patterns | memory |
| **Xirp / Spotify** | shared system/organizational context and session continuity | context + execution boundary |
| **LangSmith / Langfuse-style systems** | traces, experiments, evaluation and observability | observability/evaluation infrastructure |
| **Langroid** | multi-agent execution | execution/orchestration |
| **Plano** | operational routing/governance infrastructure | runtime operations |
| **TransformerLab** | model experimentation and evaluation | model experimentation |
| **Promptfoo** | repeatable model/prompt/configuration experiments | evaluation harness |
| **Aider benchmarks** | end-to-end coding-agent task outcomes | benchmark prior art |
| **OpenHands benchmarks** | software-engineering/agent benchmark infrastructure | benchmark prior art |
| **OpenAI Evals-style frameworks** | reusable private/workload-specific eval harnesses | evaluation harness |
| **CodeRabbit / Sourcegraph Cody / Aider / Claude Code** | coding-agent execution and/or review | execution/evaluation prior art |

## How to compare these tools

The table above is intentionally a compact map, not the selection mechanism. For actual comparison, use the structured model in [Tool capability model](../docs/tool-capability-model.md).

The important comparison dimensions are evidence kind, scope, depth, precision/recall, determinism, freshness, latency, cost, setup, explainability and provenance. Profiles should also record strengths, weaknesses, best-fit questions, poor-fit questions, and relationships such as complement, overlap, alternative and escalation.

This supports three useful views:

1. **Capability matrix** — what different providers can establish.
2. **Pairwise comparison** — why one provider is preferable to another for a particular class of question.
3. **Selection matrix** — which provider satisfies a concrete evidence requirement with the lowest acceptable cost/latency.

The goal is to avoid a misleading global ranking. A tool can be excellent overall and still be the wrong provider for a specific question.

## Important boundaries

### GrapeRoot is not TencentDB Agent Memory, and neither is EOKS

GrapeRoot primarily demonstrates proactive context optimization and agent integration around a coding agent. TencentDB Agent Memory is broader reusable-resource infrastructure: it manages multi-resolution chat memory, Skills, Wiki and CodeGraph with governance and agent loadouts. EOKS is broader still: it is exploring the control/evaluation layer that selects among such resources and learns from outcomes.

### Asset is not another semantic knowledge category

EOKS uses **Asset** as a generic lifecycle/governance abstraction for reusable resources. Memory, Skills, documents, decisions and derived representations can be assets, but they retain their own semantics and authority. See [Resource model](../docs/resource-model.md).

### GrapeRoot is not OKF

GrapeRoot is a context/runtime mechanism; OKF is a representation format. An EOKS implementation can use an OKF bundle as one knowledge provider without using GrapeRoot, and can use a GrapeRoot-like context engine without adopting OKF.

### Tencent's four resource families are not an EOKS ontology

Chat Memory, Skill, LLM-Wiki and CodeGraph are a useful implementation decomposition. EOKS should preserve the semantic distinctions—experience, procedure, knowledge representation and evidence—but should not assume these four categories are universal or exhaustive.

### Graphs are not knowledge by default

A repository graph is structural evidence. It can tell an agent where relationships exist, but graph connectivity alone does not prove an invariant, business rule or path-sensitive property. Specialized analyzers and authoritative source evidence may be needed.

### Observability is not reliability

Tracing records what happened. Evaluation scores outcomes. Reliability estimation asks how much a result should be trusted. Control uses those signals to decide what happens next. These should remain separate interfaces even when one product provides several of them.

### Evaluation tools are infrastructure, not the control plane

Promptfoo, Langfuse and similar systems can run experiments, store traces or compute scores. EOKS's proposed role is to use that evidence to make context, model, evidence-provider and execution-policy decisions.

## Existing EOKS research links

- [Agent code understanding and architecture tooling](agent-code-understanding-and-architecture.md)
- [Agent memory and continuous-learning prior art](prior-art/agent-memory.md)
- [TencentDB Agent Memory](prior-art/tencent-agent-memory.md)
- [Codebase Memory MCP](prior-art/codebase-memory-mcp.md)
- [Learning from development sessions](session-learning.md)
- [Xirp / Spotify](prior-art/xirp.md)
- [LLM observability and reliability signals](llm-observability-and-reliability.md)
- [Context evaluation and benchmarking](context-evaluation.md)
- [Evaluation, reliability and model switching](evaluation-and-model-switching.md)

The map should be updated when a tool changes the architectural hypothesis, not merely whenever another tool is mentioned.
