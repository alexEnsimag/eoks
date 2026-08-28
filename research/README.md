# EOKS research corpus

This directory preserves the **research layer** behind EOKS: the questions, ideas, comparisons, experiments and changes in direction that led to the current architecture.

It is intentionally less polished and less normative than [`docs/`](../docs/). A research note may contain competing hypotheses. A concept in this directory is not automatically an EOKS decision.

## How this relates to `docs/`

- **`docs/`** describes the current working architecture, shared terminology, decisions, open questions and research agenda.
- **`research/`** preserves the reasoning and evidence behind that architecture, including exploratory notes and prior art.
- **`docs/decisions.md`** is the place for conclusions that are strong enough to be treated as project constraints.

A research note can therefore challenge the current architecture without being rewritten immediately. Conversely, when an idea becomes a stable architectural commitment, its canonical statement should move into the appropriate document under `docs/` and the research note should point to it rather than becoming a competing source of truth.

## Why this exists

The EOKS work started as a sequence of conversations about context engineering and agent tooling, then expanded into memory, evaluation, model selection, orchestration and an AI control-plane analogy. A concise architecture document loses too much of that reasoning.

The corpus therefore separates three things:

1. **Research** — what we explored and what we observed.
2. **Architecture** — what the project currently proposes.
3. **Decisions** — conclusions that we are willing to treat as constraints.

## Reconstruction status

These notes are a reconstruction of the EOKS discussions available to the project, not verbatim transcripts. They deliberately preserve uncertainty and evolution. Where an idea was only floated, it is marked as a hypothesis rather than presented as fact.

## Current research map

### Core architecture and context

- [`knowledge-context-control-plane.md`](knowledge-context-control-plane.md) — current synthesis of knowledge, context, execution and control, including OKF, GrapeRoot and Graphify boundaries and the minimal runtime model.
- [`context-engineering.md`](context-engineering.md) — context as a managed resource; quality, entropy, blocks, graphs, compression and interactive control.
- [`context-quality-model.md`](context-quality-model.md) — dimensions and candidate metrics for context quality.
- [`context-workbench.md`](context-workbench.md) — exploratory workbench model and experiments; the current architectural counterpart is [`docs/context-workbench.md`](../docs/context-workbench.md).
- [`control-loop.md`](control-loop.md) — reconciliation/control-loop model and feedback.
- [`control-plane.md`](control-plane.md) — earlier exploratory treatment of scheduling, model selection and workload management.
- [`core-model.md`](core-model.md) — resource and semantic-model exploration.
- [`design-patterns.md`](design-patterns.md) — recurring patterns extracted from the research.
- [`evolution.md`](evolution.md) — how the EOKS hypothesis changed over time.

### Memory and learning

- [`memory-and-knowledge.md`](memory-and-knowledge.md) — memory, knowledge lifecycle, provenance, retrieval and the distinction between memory and context.
- [`memory-lifecycle.md`](memory-lifecycle.md) — lifecycle and invalidation questions.
- [`session-learning.md`](session-learning.md) — learning from development-session traces.
- [`prior-art/agent-memory.md`](prior-art/agent-memory.md) — LangMem, Mem0, Zep and execution-trace prior art.
- [`claude-learning-okf-hindsight.md`](claude-learning-okf-hindsight.md) — Claude Code knowledge/memory mechanisms, OKF and Hindsight.
- [`prior-art/hindsight-and-okf.md`](prior-art/hindsight-and-okf.md) — explicit Hindsight/OKF comparison.

### Evaluation and reliability

- [`evaluation-and-model-switching.md`](evaluation-and-model-switching.md) — confidence, metrics, evaluation, regressions and changing models.
- [`llm-observability-and-reliability.md`](llm-observability-and-reliability.md) — observability, model-level uncertainty, external evidence, calibration and control.
- [`observations-and-questions.md`](observations-and-questions.md) — observations and unresolved questions captured during the research.

### Software engineering and tools

- [`agent-code-understanding-and-architecture.md`](agent-code-understanding-and-architecture.md) — code understanding, structural evidence, deterministic analysis and architecture assurance.
- [`software-engineering.md`](software-engineering.md) — exploratory software-engineering workload model.
- [`tool-notes.md`](tool-notes.md) — capability-oriented notes on the tools and projects investigated.
- [`minimal-vertical-slice.md`](minimal-vertical-slice.md) — proposed first proving ground for the control-plane hypothesis.
- [`research-agenda.md`](research-agenda.md) — earlier research agenda; the current canonical agenda is [`docs/research-agenda.md`](../docs/research-agenda.md).

### Prior art

The [`prior-art/`](prior-art/) directory contains deeper notes on individual projects. The consolidated landscape is [`docs/prior-art.md`](../docs/prior-art.md).

Notable entries include:

- [`openwolf.md`](prior-art/openwolf.md)
- [`xirp.md`](prior-art/xirp.md)
- [`hindsight-and-okf.md`](prior-art/hindsight-and-okf.md)
- [`agent-memory.md`](prior-art/agent-memory.md)

## A useful rule

The repository should not optimize for the appearance of coherence. If two approaches conflict, record the conflict. If an experiment disproves an assumption, preserve the failed assumption and the evidence. The history is part of the knowledge system.
