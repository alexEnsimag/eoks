# EOKS research corpus

This directory preserves the **research layer** behind EOKS: questions, ideas, comparisons, experiments and changes in direction. It is less normative than [`docs/`](../docs/); a research note is not automatically an EOKS decision.

## How this relates to `docs/`

- **`docs/`** describes the current working architecture, terminology, decisions, open questions and research agenda.
- **`research/`** preserves the reasoning and evidence behind that architecture, including exploratory notes and prior art.
- **`docs/decisions.md`** contains conclusions strong enough to be treated as project constraints.

When a research conclusion becomes stable, its canonical statement should move into the appropriate `docs/` document. The research note should then point to that canonical statement rather than becoming a competing source of truth.

## Current research map

### Core architecture and context

- [`knowledge-context-control-plane.md`](knowledge-context-control-plane.md) — synthesis of knowledge, context, execution and control, including the boundaries between OKF, GrapeRoot-like context engines and structural graphs.
- [`context-engineering.md`](context-engineering.md) — context as a managed resource.
- [`context-quality-model.md`](context-quality-model.md) — dimensions and candidate metrics for context quality.
- [`context-workbench.md`](context-workbench.md) — exploratory workbench model.
- [`context-evaluation.md`](context-evaluation.md) — canonical controlled methodology for evaluating context interventions, durable knowledge, structural evidence and community evaluation tooling.
- [`control-loop.md`](control-loop.md) — reconciliation/control-loop model, including uncertainty-aware stop/continue and branching policies.
- [`control-plane.md`](control-plane.md) — scheduling, model selection and workload management.
- [`ai-os-analogies.md`](ai-os-analogies.md) — AI-OS framing and cross-domain analogies.
- [`core-model.md`](core-model.md) — resource and semantic-model exploration.
- [`design-patterns.md`](design-patterns.md) — recurring patterns, including uncertainty-aware graph control.
- [`evolution.md`](evolution.md) — how the EOKS hypothesis changed over time.

### Memory and learning

- [`memory-and-knowledge.md`](memory-and-knowledge.md) — memory, knowledge lifecycle and retrieval.
- [`memory-lifecycle.md`](memory-lifecycle.md) — lifecycle and invalidation.
- [`session-learning.md`](session-learning.md) — learning from development-session traces.
- [`prior-art/agent-memory.md`](prior-art/agent-memory.md) — memory-system prior art.
- [`prior-art/tencent-agent-memory.md`](prior-art/tencent-agent-memory.md) — TencentDB Agent Memory: multi-resolution memory, Skills, Wiki, CodeGraph, governance/loadouts and hybrid context delivery.
- [`claude-learning-okf-hindsight.md`](claude-learning-okf-hindsight.md) — Claude Code knowledge/memory mechanisms, OKF and Hindsight.
- [`prior-art/hindsight-and-okf.md`](prior-art/hindsight-and-okf.md) — Hindsight/OKF comparison.

### Evaluation and reliability

- [`evaluation-and-model-switching.md`](evaluation-and-model-switching.md) — reliability, confidence, model/task affinity and safe model migration.
- [`llm-uncertainty-and-control.md`](llm-uncertainty-and-control.md) — probabilistic model signals, entropy, semantic entropy, calibration, semantic agreement and using uncertainty as a graph/control signal.
- [`context-evaluation.md`](context-evaluation.md) — controlled context/evidence benchmark methodology and community evaluation tooling.
- [`llm-observability-and-reliability.md`](llm-observability-and-reliability.md) — observability, uncertainty, external evidence and calibration.
- [`observations-and-questions.md`](observations-and-questions.md) — unresolved questions.

### Software engineering and tools

- [`agent-code-understanding-and-architecture.md`](agent-code-understanding-and-architecture.md) — code understanding, structural evidence, deterministic analysis and architecture assurance.
- [`software-engineering.md`](software-engineering.md) — software-engineering workload model.
- [`tool-notes.md`](tool-notes.md) — capability-oriented map of investigated tools.
- [`minimal-vertical-slice.md`](minimal-vertical-slice.md) — proposed first proving ground.
- [`research-agenda.md`](research-agenda.md) — earlier agenda; the current canonical agenda is [`docs/research-agenda.md`](../docs/research-agenda.md).

### Prior art

The [`prior-art/`](prior-art/) directory contains deeper notes on individual projects, including [`OpenWiki`](prior-art/openwiki.md) as prior art for git-native, reviewable generated codebase knowledge and its operational freshness lifecycle. The consolidated landscape is [`docs/prior-art.md`](../docs/prior-art.md).

## Useful rule

The repository should not optimize for the appearance of coherence. If two approaches conflict, record the conflict. If an experiment disproves an assumption, preserve the failed assumption and the evidence. The history is part of the knowledge system.
