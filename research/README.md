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
- [`context-acquisition-comparison.md`](context-acquisition-comparison.md) — evidence-led comparison of raw exploration, retrieval, graphs, FastContext-style delegated exploration and GrapeRoot-style persistent structure.
- [`evidence-at-a-glance.md`](evidence-at-a-glance.md) — concise evidence dashboard: numbers, community signals, conflicts and research priorities.
- [`community-evidence-bottlenecks.md`](community-evidence-bottlenecks.md) — quantitative academic evidence, practitioner/community signals, recurring bottlenecks, contradictory results and prioritized next experiments.
- [`control-loop.md`](control-loop.md) — reconciliation/control-loop model, including uncertainty-aware stop/continue and branching policies.
- [`control-plane.md`](control-plane.md) — scheduling, model selection and workload management.
- [`ai-os-analogies.md`](ai-os-analogies.md) — AI-OS framing and cross-domain analogies.
- [`core-model.md`](core-model.md) — resource and semantic-model exploration.
- [`design-patterns.md`](design-patterns.md) — recurring patterns, including uncertainty-aware graph control.
- [`evolution.md`](evolution.md) — how the EOKS hypothesis changed over time.
- [`validated-reusable-computation.md`](validated-reusable-computation.md) — synthesis of refinement, evaluation, provenance, dependency-aware reuse and incremental recomputation for AI workloads; recent evaluation/optimization tooling is treated as evidence for the empirical execution→trace→evaluation→candidate-refinement side, not as an EOKS primitive.
- [`prior-art/validated-reusable-computation-sources.md`](prior-art/validated-reusable-computation-sources.md) — preserved source register for the validated/reusable-computation synthesis, including foundational research, standards, evaluation/optimization tooling and recent research leads.
- [`prior-art/knowledge-memory-context-synthesis.md`](prior-art/knowledge-memory-context-synthesis.md) — synthesis of knowledge, experience, context compilation, decision provenance and graph/memory/context representations; argues for sharper existing boundaries rather than new graph primitives.

### Memory and learning

- [`memory-and-knowledge.md`](memory-and-knowledge.md) — memory, knowledge lifecycle and retrieval.
- [`memory-lifecycle.md`](memory-lifecycle.md) — lifecycle and invalidation.
- [`session-learning.md`](session-learning.md) — learning from development-session traces.
- [`prior-art/agent-memory.md`](prior-art/agent-memory.md) — memory-system prior art.
- [`prior-art/tencent-agent-memory.md`](prior-art/tencent-agent-memory.md) — TencentDB Agent Memory: multi-resolution memory, Skills, Wiki, CodeGraph, governance/loadouts and hybrid context delivery.
- [`prior-art/knowledge-memory-context-graph.md`](prior-art/knowledge-memory-context-graph.md) — synthesis of knowledge, experience, context compilation and evaluation prompted by the knowledge/memory/context graph distinction.
- [`claude-learning-okf-hindsight.md`](claude-learning-okf-hindsight.md) — Claude Code knowledge/memory mechanisms, OKF and Hindsight.
- [`prior-art/hindsight-and-okf.md`](prior-art/hindsight-and-okf.md) — Hindsight/OKF comparison.

### Evaluation and reliability

- [`evaluation-and-model-switching.md`](evaluation-and-model-switching.md) — reliability, confidence, model/task affinity, safe model migration, and benchmark boundaries for model/harness/workload evaluation.
- [`llm-uncertainty-and-control.md`](llm-uncertainty-and-control.md) — probabilistic model signals, entropy, semantic entropy, calibration, semantic agreement and using uncertainty as a graph/control signal.
- [`intermediate-evidence-and-model-signals.md`](intermediate-evidence-and-model-signals.md) — intermediate evidence across external, execution, process and model-native sources, including token probabilities, semantic uncertainty, hidden-state probing, calibration and the boundary to computation reuse.
- [`llm-observability-and-reliability.md`](llm-observability-and-reliability.md) — observability, uncertainty, external evidence and calibration.
- [`prior-art/agent-trajectory-evaluation.md`](prior-art/agent-trajectory-evaluation.md) — agent trajectory evaluation, outcome-vs-process evidence, stochastic evaluation and trajectory capture.
- [`observations-and-questions.md`](observations-and-questions.md) — unresolved questions.
- [`prior-art/faraday-replica.md`](prior-art/faraday-replica.md) — Faraday/Replica: learned scientific judgment over coding agents, constrained experimentation, rubric-based evaluation and trajectory-level credit.

### Software engineering and tools

- [`agent-code-understanding-and-architecture.md`](agent-code-understanding-and-architecture.md) — code understanding, structural evidence, deterministic analysis and architecture assurance.
- [`software-engineering.md`](software-engineering.md) — software-engineering workload model.
- [`tool-notes.md`](tool-notes.md) — capability-oriented map of investigated tools.
- [`minimal-vertical-slice.md`](minimal-vertical-slice.md) — proposed first proving ground.
- [`research-agenda.md`](research-agenda.md) — earlier agenda; the current canonical agenda is [`docs/research-agenda.md`](../docs/research-agenda.md).

### Prior art

- [`prior-art/ai-native-sdlc.md`](prior-art/ai-native-sdlc.md) — AI-native SDLC and spec-driven development mapped onto EOKS control loops, durable artifacts, context compilation and continuous maintenance.
- [`prior-art/beyond-the-agent.md`](prior-art/beyond-the-agent.md) — primary-source synthesis of context engineering, vendor-neutral coding-agent execution, persistent workspaces, evidence/authority boundaries and multi-timescale feedback.
- [`prior-art/reflection-harness-sdd.md`](prior-art/reflection-harness-sdd.md) — practitioner evidence on reflection over intermediate SDD artifacts, progressive validation/freezing, persistent review state and the boundary between durable artifacts and reusable computation.
- [`prior-art/posthog-context-lifecycle.md`](prior-art/posthog-context-lifecycle.md) — PostHog practitioner evidence on resident context cost, stale guidance, context regression evaluation and feedback-driven maintenance; mapped to existing EOKS context and control-loop concepts.
- [`prior-art/github-copilot-task-efficiency.md`](prior-art/github-copilot-task-efficiency.md) — GitHub Copilot practitioner evidence that local token/tool-output savings can create downstream recovery work; maps task-level efficiency onto existing EOKS evaluation and intervention semantics.
- [`prior-art/structured-knowledge-and-context-graphs.md`](prior-art/structured-knowledge-and-context-graphs.md) — ontology, knowledge graphs, context graphs and recent agent-memory research; argues for structured/contextualized evidence as the abstraction rather than a mandatory graph primitive.
- [`prior-art/codebase-knowledge-graph-relationships.md`](prior-art/codebase-knowledge-graph-relationships.md) — Sachin Kasana's 500K-line codebase knowledge-graph case study, connected to Code Property Graphs, relationship-aware context construction, provenance and evaluation of traversal.

The [`prior-art/`](prior-art/) directory contains deeper notes on individual projects. The consolidated landscape is [`docs/prior-art.md`](../docs/prior-art.md).

## Useful rule

The repository should not optimize for the appearance of coherence. If two approaches conflict, record the conflict. If an experiment disproves an assumption, preserve the failed assumption and the evidence. The history is part of the knowledge system.
