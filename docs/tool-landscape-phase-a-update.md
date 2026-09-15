# Phase A tool landscape update

This addendum records the current Phase A positions after the latest tool review. It is intentionally additive: the richer historical landscape remains useful, while this document records the newer mechanisms and the system-representation gap.

## Current Phase A positions

| Tool / mechanism | Phase A position | Primary EOKS role | Rationale / boundary |
|---|---|---|---|
| **OpenWolf mechanisms** | **Core** | harness lifecycle/state/measurement | Strong prior art for hooks, session state, pre-compaction preservation, handoff and token measurement. Use mechanisms, not the whole product. |
| **Spotify Portal/Shunt mechanisms** | **Core** | interception/transformation/delegation | Concrete prior art for hard-gating large reads and delegating predictable I/O/code generation. The 82–94% savings are for its intercepted benchmark scenarios, not a general 90% total-cost claim. |
| **Soda-style passive capture** | **Experiment / candidate** | work observation/capture | Directly attacks the capture bottleneck: derive context from work rather than asking the human to record it. Treat the private-beta claims as unvalidated; test privacy, provenance, freshness, noise and usefulness. |
| **Understand Anything** | **Core representation experiment** | code/semantic representation | Promoted for semantic repository/code representation and visualization. Explicitly test polyglot and multi-repository limits. |
| **Nx** | **Core representation candidate** | project/dependency/cross-repo representation | Synthetic-monorepo/project-graph direction directly targets the cross-repository gap. Test as a provider, not as the EOKS platform. |
| **Sourcegraph** | **Reference / baseline** | cross-repository code intelligence | Strong baseline for persistent cross-repo code search/navigation; useful for defining the capability EOKS would need without adopting the full platform. |
| **Structurizr** | **Reference / companion candidate** | system architecture representation | Useful system/service/API architecture model complementary to code-level graphs. |
| **CodeSee** | **Reference / visualization candidate** | codebase/dependency visualization | Useful comparison for human navigation and maps; not currently a required runtime dependency. |
| **Obsidian** | **Phase A companion** | human inspection/curation/visualization | Consume EOKS Markdown/artifacts. Do not put it in the agent execution loop. |
| **Notion** | **Comparison / reference** | structured workspace/documentation | Useful benchmark for databases, collaboration and operational documentation; not an EOKS runtime dependency. |
| **Capacities** | **Comparison / reference** | typed object representation | Useful benchmark for object-based, linked knowledge and low-friction organization. |
| **Tana** | **Comparison + Phase B bridge** | typed evolving knowledge graph / capture | Particularly relevant to structured work-derived capture, current knowledge and agent access; defer persistent memory adoption until Phase A fundamentals are measured. |
| **Roam Research** | **Comparison / reference** | linked temporal knowledge | Useful benchmark for daily-note and deliberate-link workflows, not a runtime dependency. |
| **Apple Notes** | **Capture-friction baseline** | minimal human capture | Important because the article found its low-friction capture more durable than richer systems. It is a design constraint/comparison, not an EOKS substrate. |
| **Mem** | **Phase B prior art** | persistent recall / evolving memory | AI-assisted capture and recall are directly relevant, but durable memory should follow current-session and capture experiments. |
| **Hindsight / LangMem-style memory** | **Phase B** | durable semantic/episodic/procedural memory | Targets persistent learning/evolution. |
| **Mem0 / Zep-style memory** | **Phase B** | persistent memory/retrieval | Useful comparison for memory lifecycle and retrieval, not Phase A substrate. |
| **Opik / LangSmith / Langfuse-style observability** | **Phase B** | richer tracing/evaluation | Start with minimal local measurements; add richer observability once interventions become complex enough to need it. |
| **GrapeRoot** | **Reference / prior art** | proactive context compilation | Valuable example, but its critical graph engine is proprietary and therefore weak as a first implementation substrate. |
| **Graphify / Semgrep / CodeQL** | **Targeted providers** | structural/evidence layers | Use where a concrete representation or evidence question calls for them; none is the Phase A system representation alone. |

## Relationship changes

The tools should not be treated as competing product categories. They provide mechanisms at different layers:

```text
work / coding-agent harness
          │
          ├── OpenWolf mechanisms
          │      lifecycle / state / measurement
          │
          ├── Portal/Shunt mechanisms
          │      intercept / transform / delegate
          │
          └── Soda-style capture
                 observe / capture / preserve provenance

representation providers
          │
          ├── Understand Anything  → code / semantic
          ├── Nx                  → project / dependency / cross-repo
          ├── Sourcegraph         → cross-repo code intelligence baseline
          ├── Structurizr         → system / architecture
          ├── CodeSee             → visualization
          └── Graphify/CodeQL/etc → targeted evidence

human-facing / capture comparison
          │
          ├── Obsidian
          ├── Notion
          ├── Capacities
          ├── Tana
          ├── Roam
          └── Apple Notes

persistent evolution / observability
          │
          ├── Mem
          ├── Hindsight / LangMem
          ├── Mem0 / Zep
          └── Opik / LangSmith / Langfuse
```

## Capture lesson from the second-brain review

The second-brain article explicitly tested **Notion, Obsidian, Capacities, Mem, Tana and Roam Research**. It used **Apple Notes** as the practical low-friction baseline and identified the common failure as capture friction: the valuable information often arrives while the person is already in a call, meeting, Slack thread or focused task.

The article's second important observation is that the problem does not end at capture. Information can be difficult to trust when it is hidden behind automation, and any persistent knowledge system must deal with information becoming stale or misleading. That makes **capture, representation, synthesis, maintenance and future use separate capabilities**.

Soda is the article's proposed counter-example: passive work observation rather than manual capture. The article explicitly says Soda was still in private beta and that its efficacy and privacy implications were unknown. EOKS should therefore treat the approach as a hypothesis, not established evidence.

The useful EOKS conclusion is not that one of these is the best second brain. It is that EOKS should be a **work-coupled context/synthesis system rather than a second-brain application**.

Core principles:

> **Knowledge has value when it changes future work, not merely when it is successfully stored.**

> **EOKS should be work-coupled: the human should not have to stop working just to keep EOKS up to date.**

> **Captured knowledge must be allowed to evolve: stale, contradicted or low-value information should not become permanent simply because it was successfully stored.**

## System representation gap

The old **Repository structure** category is too narrow. The landscape must distinguish:

1. **repository-local representation** — files, symbols, calls, semantic relationships;
2. **cross-repository representation** — ownership and relationships across repository boundaries;
3. **system representation** — services, APIs, events, datastores and architecture;
4. **runtime representation** — observed execution relationships and telemetry;
5. **end-to-end representation** — navigable paths across those layers.

Understand Anything is a Phase A provider for the first layer and potentially parts of the second. Nx is a promising provider for project/dependency boundaries. Sourcegraph is a cross-repository code-intelligence baseline. Structurizr addresses system architecture. CodeSee provides a visualization comparison. Runtime/evidence providers can supply observed behavior.

No single candidate should be assumed to solve all layers. The missing capability is tracked in `research/system-representation-gap.md`.

## Landscape comparison matrix

This is the current comparison to carry into experiments; ratings are capability-oriented, not EOKS benchmark scores.

| Tool | Capture from work | Representation | Cross-repo/system | Human inspection | Agent/runtime integration | Persistent evolution | Current EOKS role |
|---|---:|---|---:|---:|---:|---:|---|
| OpenWolf | low | session/context artifacts | no | medium | high | medium | Phase A harness mechanisms |
| Portal/Shunt | low | transformed context | no | low | high | low | Phase A harness mechanisms |
| Soda | **candidate** | work/customer context | potentially | medium | high | **candidate** | capture experiment; private-beta evidence remains limited |
| Understand Anything | low | code/semantic graph | partial/unknown | high | medium | medium | Phase A representation provider |
| Nx | low | project/dependency graph | **high candidate** | high | medium | medium | Phase A cross-repo candidate |
| Sourcegraph | low | code intelligence | **high** | high | high | high | reference/baseline |
| Structurizr | low | system architecture model | high | **high** | medium | medium | architecture companion/reference |
| CodeSee | low | code/dependency maps | limited | **high** | medium | medium | visualization reference |
| Obsidian | manual | linked Markdown knowledge | user-defined | **high** | low/medium | medium | Phase A inspection surface |
| Notion | manual/assisted | databases/pages | workspace-oriented | high | medium | high | reference |
| Capacities | manual/assisted | typed objects/links | workspace-oriented | high | medium | high | reference |
| Tana | **candidate** | typed knowledge graph | workspace/team oriented | high | **high** | **high candidate** | bridge/reference; Phase B candidate |
| Roam | manual | linked notes/graph | no | high | low/medium | medium | reference |
| Apple Notes | **very low friction** | simple notes | no | high | low | low | capture baseline |
| Mem | assisted | AI memory/workspace | connected apps | high | high | **high candidate** | Phase B prior art |
| Hindsight / LangMem | programmatic | memory records | application-defined | low | high | **high** | Phase B |
| Mem0 / Zep | programmatic | memory store | application-defined | low | high | **high** | Phase B |
| Opik / LangSmith / Langfuse | automatic traces | traces/evals | system-dependent | medium | **high** | high | Phase B observability |
| GrapeRoot | automatic/proactive | repository context graph | limited/unknown | high | high | medium | prior art |
| Graphify | automatic | structural/code graph | repository-focused | medium/high | medium | medium | targeted provider |
| Semgrep | automatic | deterministic findings | repository/project | low | high | low | evidence provider |
| CodeQL | automatic | deep dataflow graph/findings | repository/project | medium | medium/high | low | evidence provider |

The important comparison is not a winner column. It is the **composition opportunity**:

```text
work capture
   + session/context control
   + code representation
   + project/cross-repo representation
   + system architecture
   + runtime/evidence
   ↓
EOKS representation + synthesis
   ↓
task-specific context / artifact
```

## Tool-selection rule for Phase A

Do not choose a tool because it has the richest graph, memory or UI. Choose a provider based on the evidence it contributes to a concrete engineering question and whether that evidence can be composed with other providers.

The desired architecture is:

```text
provider(s)
    ↓
representation / evidence / observation
    ↓
EOKS selection + context policy
    ↓
agent
```

not:

```text
tool
    ↓
canonical EOKS knowledge graph
```
