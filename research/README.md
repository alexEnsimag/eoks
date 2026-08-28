# EOKS research corpus

This directory preserves the **research layer** behind EOKS: the questions, ideas, comparisons, experiments and changes in direction that led to the current architecture.

It is intentionally less polished than `docs/`. A research note may contain competing hypotheses. A concept in this directory is not automatically an EOKS decision.

## Why this exists

The EOKS work started as a sequence of conversations about context engineering and agent tooling, then expanded into memory, evaluation, model selection, orchestration and an AI control-plane analogy. A concise architecture document loses too much of that reasoning.

The corpus therefore separates three things:

1. **Research** — what we explored and what we observed.
2. **Architecture** — what the project currently proposes.
3. **Decisions** — conclusions that we are willing to treat as constraints.

## Reconstruction status

These notes are a reconstruction of the EOKS discussions available to the project, not verbatim transcripts. They deliberately preserve uncertainty and evolution. Where an idea was only floated, it is marked as a hypothesis rather than presented as fact.

## Current research map

- [`context-engineering.md`](context-engineering.md) — context as a managed resource; quality, entropy, blocks, graphs, compression and interactive control.
- [`memory-and-knowledge.md`](memory-and-knowledge.md) — memory, knowledge lifecycle, provenance, retrieval and the distinction between memory and context.
- [`control-plane.md`](control-plane.md) — the Kubernetes/control-plane analogy, scheduling, model selection and workload management.
- [`evaluation-and-model-switching.md`](evaluation-and-model-switching.md) — confidence, metrics, evaluation, regressions and changing models without guessing.
- [`software-engineering.md`](software-engineering.md) — code understanding, graphs, deterministic analysis, taint/dataflow and hybrid LLM/tool workflows.
- [`prior-art.md`](prior-art.md) — GrapeRoot, CodeSight, EKOS, TencentDB Agent Memory, XIRP, OKF and analysis/observability tools.
- [`evolution.md`](evolution.md) — how the EOKS hypothesis changed over time.
- [`research-agenda.md`](research-agenda.md) — experiments that could distinguish the useful parts of the idea from attractive architecture.

## A useful rule

The repository should not optimize for the appearance of coherence. If two approaches conflict, record the conflict. If an experiment disproves an assumption, preserve the failed assumption and the evidence. The history is part of the knowledge system.