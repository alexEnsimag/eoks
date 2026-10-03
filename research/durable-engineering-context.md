# Durable engineering context

## Why this matters

AI coding sessions are ephemeral, but software projects accumulate information that must survive individual sessions, agents and model calls. Practitioners increasingly encode that information in repository artifacts such as `CLAUDE.md`, `AGENTS.md`, plans, decision records, progress ledgers and generated project documentation.

The important EOKS abstraction is not the file format. It is the **durable engineering artifact**: persistent, inspectable information or state that can be selected for a later workload, evaluated for relevance/freshness, and updated from evidence.

This distinction matters because persistent context has mixed evidence. A 2026 study of 124 pull requests across 10 repositories found `AGENTS.md` associated with 28.64% lower median runtime and 16.58% lower output-token consumption, with comparable task-completion behavior. A separate controlled ablation across 288 runs found no measurable correctness improvement from `AGENTS.md`/`CLAUDE.md` context injection. These results are not contradictory if context primarily changes efficiency for some workloads while many coding failures arise from implementation, design or wiring rather than missing repository knowledge.

Sources:
- Lulla et al., 2026: https://arxiv.org/abs/2601.20404
- Khatri, 2026: https://arxiv.org/abs/2607.27250

## Durable context is not one kind of memory

EOKS should distinguish artifacts by the role they play in the control loop rather than putting everything into a generic memory store.

| Role | What it preserves | Examples |
|---|---|---|
| **Intent** | what is being achieved and why | task contract, objective, acceptance criteria |
| **Knowledge** | reusable facts and meaning | architecture, conventions, ADRs, domain knowledge |
| **State** | where work currently stands | progress ledger, checkpoints, unresolved questions |
| **Policy** | what is allowed or required | repository instructions, safety/quality rules, autonomy limits |
| **Experience / evidence** | what actually happened | observations, test results, failed approaches, traces |
| **Representation** | a view optimized for a task | code map, diagram, summary, generated specification |
| **Evaluation** | whether an artifact or intervention remains useful | validation result, freshness check, outcome linkage |

These roles can share a storage substrate. A Markdown file can contain knowledge, state or policy; a graph can represent structural knowledge; a generated view can be a task-specific representation. The semantic role should not be inferred from the storage technology.

## The lifecycle

Durable context should have an explicit lifecycle:

```text
create
  ↓
scope / provenance
  ↓
select for a workload
  ↓
consume
  ↓
observe outcomes
  ↓
verify / evaluate
  ↓
update, promote, supersede or retire
  ↺
```

This makes learning a controlled transformation rather than silent rewriting of project truth:

```text
experience / evidence
        ↓
candidate learning
        ↓
human or policy-controlled promotion
        ↓
durable artifact
        ↓
future context
        ↓
new evidence
```

This aligns with EOKS's existing principle of candidate extraction and controlled promotion.

## Selection matters more than storage

Persistent context is useful only when the right information reaches the right workload.

Relevant dimensions include:

- **scope** — repository, package, service, task, organization;
- **provenance** — who/what produced the artifact and from which evidence;
- **freshness** — source revision, last verification and invalidation conditions;
- **authority** — canonical, derived, advisory or experimental;
- **relevance** — why it applies to the current workload;
- **cost** — tokens, latency and retrieval/processing overhead;
- **risk** — consequences if the artifact is wrong or stale.

This supports the existing EOKS distinction between resources, loadouts and task-specific context compilation.

The key hypothesis is therefore not:

> More persistent context makes agents better.

It is:

> **Well-scoped, fresh, evidence-backed durable context can reduce avoidable rediscovery and improve continuity, while irrelevant or stale context can add cost and induce errors.**

## Context rot

Persistence creates a new failure mode: **context rot**.

As code and architecture evolve, instructions and generated knowledge can become stale. Treude and Baltes' 2026 study argues that existing documentation-consistency techniques can be repurposed for AI configuration artifacts and reports stale code-element references in 23% of a statistically representative sample of 356 repositories.

Source: https://arxiv.org/abs/2606.09090

For EOKS, freshness should therefore be a property of durable artifacts rather than an afterthought.

Potential mechanisms include:

- source revision references;
- generated artifacts carrying the revision they describe;
- explicit ownership;
- freshness/validation checks;
- dependency tracking between artifacts and source;
- supersession rather than destructive rewriting;
- confidence/authority metadata;
- periodic or change-triggered revalidation.

## Relationship to hooks and learning

Hooks are not themselves memory. They are **lifecycle mechanisms** that can observe or trigger transitions around durable artifacts.

Examples:

- session-end hook → extract candidate experience;
- commit/change hook → invalidate or flag derived knowledge;
- task-start hook → assemble relevant context;
- validation hook → attach evidence to an artifact;
- promotion workflow → turn repeated observations into governed knowledge.

This makes the recent EOKS research on learning hooks particularly relevant: the valuable unit is not merely "a hook that writes a memory," but a controlled path from experience to durable, reviewable engineering knowledge.

## Plans and execution state

Plans deserve separate treatment from generic documentation.

A plan can preserve:

- intent;
- expected implementation steps;
- affected files/components;
- assumptions;
- validation criteria;
- current execution state.

That makes plans a bridge between **INTENT, STATE and EXECUTION**.

They should not automatically become canonical knowledge: completed plans can be historical evidence, while reusable discoveries extracted from them may deserve promotion into knowledge or policy.

## Visual and generated artifacts

The same model applies to visual artifacts such as architecture diagrams and Excalidraw drawings.

Their value is not that they are "another memory format." They can provide a representation that makes relationships, boundaries and system state easier to inspect or communicate.

Generated artifacts should carry provenance and freshness where practical. A diagram derived from code is evidence-bearing derived context; a manually curated architecture decision is a different authority level.

This distinction prevents EOKS from conflating:

- human-authored canonical knowledge;
- machine-derived representations;
- transient task context.

## What the evidence does not establish

The current evidence is not sufficient to claim that any single context strategy is universally better.

In particular, EOKS should not assume:

- more `CLAUDE.md`/ `AGENTS.md` content is better;
- retrieval is always better than raw repository exploration;
- summaries are equivalent to execution state;
- graphs are necessary for durable context;
- external memory stores are necessary;
- persistent context necessarily improves correctness.

A controlled 2026 study specifically found no measurable correctness improvement from context files in its tested setting, while another study found efficiency improvements. This is a reason to benchmark interventions by workload and outcome rather than to prescribe a universal architecture.

## Research questions

The concept suggests a focused research agenda:

1. **Artifact value** — which durable artifacts measurably reduce rediscovery, retries or cost?
2. **Selection** — how much does artifact selection quality matter relative to storage quality?
3. **Freshness** — how quickly does persistent context become harmful as repositories evolve?
4. **Lifecycle automation** — which hooks can safely detect, invalidate or promote artifacts?
5. **State vs summary** — when does explicit execution state outperform transcript summarization?
6. **Representation** — when do diagrams, graphs, generated maps or structured records provide value beyond source files?
7. **Governance** — what provenance/authority model prevents learned experience from silently becoming project truth?
8. **Cross-session continuity** — which durable state actually improves long-horizon work?
9. **Cost** — when does maintaining/retrieving durable context cost more than the exploration it replaces?

## EOKS interpretation

Durable engineering context is best treated as a **resource lifecycle**, not a new memory subsystem.

It sits across the EOKS model:

```text
INTENT ───────────────┐
KNOWLEDGE ────────────┤
STATE ────────────────┼──> durable artifacts
POLICY ───────────────┤         │
EXPERIENCE / EVIDENCE ┘         │
                               ▼
                         context selection
                               │
                               ▼
                           execution
                               │
                               ▼
                     evaluation / outcomes
                               │
                               ▼
                            learning
                               │
                               └──> candidate artifact
```

The practical consequence is deliberately modest: **EOKS should model durable context through existing resource, context, learning and evaluation primitives before introducing a dedicated "memory artifact" primitive.**

That keeps the semantic model small while giving EOKS a place to reason about persistent project context, its lifecycle and its failure modes.
