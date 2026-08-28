# Tool and project notes

This note is a **capability map**, not a second source of detailed architecture or methodology. Individual projects are prior art; EOKS does not assume that any of them are dependencies.

For detailed treatment of context engines and knowledge representations, see [Context engineering](../docs/context.md) and [Context evaluation and benchmarking](context-evaluation.md). For reliability and model migration, see [Evaluation, reliability and model switching](evaluation-and-model-switching.md).

## Capability map

| Project / family | Primary EOKS-relevant capability | Where it fits |
|---|---|---|
| **GrapeRoot** | repository graph + context assembly around an agent | context engine / runtime adapter |
| **OKF** | portable Markdown + YAML-frontmatter knowledge bundle | knowledge representation |
| **Graphify** | code relationship/structure graph | structural evidence provider |
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

## Important boundaries

### GrapeRoot is not OKF

GrapeRoot is a context/runtime mechanism; OKF is a representation format. An EOKS implementation can use an OKF bundle as one knowledge provider without using GrapeRoot, and can use a GrapeRoot-like context engine without adopting OKF.

### Graphs are not knowledge by default

A repository graph is structural evidence. It can tell an agent where relationships exist, but graph connectivity alone does not prove an invariant, business rule or path-sensitive property. Specialized analyzers and authoritative source evidence may be needed.

### Observability is not reliability

Tracing records what happened. Evaluation scores outcomes. Reliability estimation asks how much a result should be trusted. Control uses those signals to decide what happens next. These should remain separate interfaces even when one product provides several of them.

### Evaluation tools are infrastructure, not the control plane

Promptfoo, Langfuse and similar systems can run experiments, store traces or compute scores. EOKS's proposed role is to use that evidence to make context, model, evidence-provider and execution-policy decisions.

## Existing EOKS research links

- [Agent code understanding and architecture tooling](agent-code-understanding-and-architecture.md)
- [Agent memory and continuous-learning prior art](prior-art/agent-memory.md)
- [Learning from development sessions](session-learning.md)
- [Xirp / Spotify](prior-art/xirp.md)
- [LLM observability and reliability signals](llm-observability-and-reliability.md)
- [Context evaluation and benchmarking](context-evaluation.md)
- [Evaluation, reliability and model switching](evaluation-and-model-switching.md)

The map should be updated when a tool changes the architectural hypothesis, not merely whenever another tool is mentioned.
