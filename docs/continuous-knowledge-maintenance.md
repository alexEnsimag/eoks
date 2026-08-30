# Continuous knowledge maintenance

EOKS treats continuous knowledge update as a **reconciliation lifecycle**, not as a new permanent agent role.

The purpose is to keep durable knowledge and derived representations synchronized with authoritative changes while avoiding unnecessary full recomputation.

## Core principle

> **Knowledge maintenance is reconciliation of derived state against authoritative state. It composes roles; it is not itself a core agent role.**

A typical lifecycle is:

```text
authoritative change / observation
        |
        v
  observe dependencies / detect impact
        |
        v
    retrieve evidence
        |
        v
     transform / derive
        |
        v
      validate
        |
   +----+----+
   |         |
 accept    reject
   |         |
   v         v
update /   repair / investigate
invalidate
   |
   v
verify resulting state
   |
   +-------------> reconcile again when dependencies change
```

The conductor coordinates this loop, but should not become the knowledge store or perform every operation itself. The authoritative source remains the reference state; graphs, indexes, summaries and caches are derived representations that can be updated, invalidated or rebuilt.

## Roles involved

### Retriever

Retrieves the authoritative or supporting evidence needed to understand a change.

Examples include changed files, affected symbols, related decisions, historical events, runtime observations and existing knowledge relevant to the update.

### Transformer

Converts observations into updated representations or candidate knowledge.

Examples include structural graph updates, semantic-index updates, timelines, derived relationships, summaries and candidate procedural or synthetic knowledge.

### Validator

Checks whether the proposed update is sufficiently supported and internally consistent before it is promoted.

Validation may be deterministic or agent-assisted. The required assurance depends on the representation and workload risk.

### Conductor

Coordinates when to run maintenance, which evidence and resources to use, what dependencies are affected, and whether reconciliation should update, invalidate, retry, investigate or escalate.

### Reviewer

May be introduced when a change has semantic consequences that cannot be adequately established through deterministic validation alone, such as a proposed architecture invariant or synthetic rationale.

### Executor

May perform the actual write/update operations. For many deterministic representations, this can instead be a non-agentic tool or provider.

## Deterministic tools are maintained artifacts, not another agent role

EOKS should distinguish between **roles** and **execution resources**. A deterministic tool is normally a resource/capability that the conductor can select for a step; it does not need to become a new agent role merely because it is important.

The conductor should be able to select an available deterministic tool when the step is sufficiently specified and the tool's capability, preconditions and assurance are adequate. The executor (or an ordinary workflow worker) can then invoke it without inserting an unnecessary LLM call.

```text
step requirement
      |
      v
   conductor
      |
      +--> deterministic tool available + sufficient specification
      |              |
      |              v
      |          executor/worker
      |
      `--> ambiguity / missing capability
                     |
                     v
                 reasoning
```

This should be evaluated per step. EOKS should not turn every useful script into a permanent tool, and should not preserve a deterministic implementation when maintaining it costs more than the probabilistic alternative or when the task is inherently better handled by reasoning.

### Continuous maintenance of deterministic tools

Continuous maintenance may also operate on the tools themselves:

```text
repeated workflow traces / failures / corrections
              |
              v
       detect stable procedure
              |
              v
       candidate deterministic tool
              |
       +------+------+
       |             |
   validate       insufficient
       |             |
       v             v
 register/update   keep reasoning path
       |
       v
 monitor / invalidate when assumptions change
```

The goal is **simple, stable, high-value automation**, not replacing every successful LLM behavior with bespoke code. A tool should be promoted when its behavior is sufficiently understood, its inputs/outputs and dependencies can be specified, and the expected reuse justifies its maintenance cost.

This gives continuous maintenance a second useful responsibility: not only keeping knowledge and representations current, but identifying opportunities to **consolidate repeated, well-understood procedures into simpler deterministic capabilities**.

## Change and impact detection

Impact detection is an important supporting capability, but does not need to be an agent role.

Its responsibility is to determine which durable or derived artifacts may have become stale after an observation:

```text
source revision
      |
      v
 impact detection
      |
 +----+---------+----------------+
 |              |                |
graph         index          context cache
stale?        stale?             stale?
 |              |                |
 +--------------+----------------+
                |
          targeted update
```

Cheap deterministic dependency information should be preferred where available. Agent reasoning can be used when impact is semantic, ambiguous or difficult to derive mechanically.

## Update versus promotion

Not every observed change should become canonical knowledge.

EOKS should distinguish:

- **representation update** — refresh a derived representation from authoritative sources;
- **candidate knowledge** — information inferred or synthesized from observations;
- **promotion** — accepting a candidate into durable knowledge or memory after sufficient evidence and governance;
- **invalidation** — marking derived information stale when its dependencies change.

This preserves the boundary between authoritative knowledge and derived evidence. Repeated agent behavior alone is not sufficient evidence that a new procedure, invariant or fact should become durable knowledge.

## Examples

### Code change

```text
commit
  -> impact detection
  -> retrieve changed/affected evidence
  -> update structural representation
  -> validate consistency
  -> invalidate affected context caches
  -> verify representation state
```

### Architecture change

```text
merged architectural change
  -> retrieve ADRs, code and related decisions
  -> identify affected knowledge
  -> propose updated invariant/rationale
  -> independent review + validation
  -> promote or reject
  -> verify resulting canonical state
```

### Completed agent workflow

```text
workflow outcome
  -> collect trace, corrections, tests and review findings
  -> transform into candidate procedure/lesson
  -> validate against outcome evidence and scope
  -> promote only when governance requirements are met
```

## Relationship to EOKS planes

Continuous maintenance crosses the existing planes rather than introducing a new one. It is a nested reconciliation loop:

```text
             CONTROL PLANE
               Conductor
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
 Knowledge     Context     Execution
   update     invalidation  / tools
       ^           ^           |
       |           |           |
       +----- Evaluation ------+
                   |
                   v
              reconcile
```

The knowledge plane owns durable knowledge and derived representations. The context plane owns task-specific compilation and cache invalidation. The execution plane performs workflow actions. Evaluation supplies evidence for promotion and control decisions.

## Design constraint

Continuous maintenance should be incremental, dependency-aware and evidence-driven. It should not imply that the entire knowledge system is recomputed after every tool call.

The default should be:

```text
cheap deterministic detection
        -> targeted update
        -> selective validation
        -> expensive reasoning only when justified
```

This keeps maintenance aligned with EOKS's broader objectives of improving trust, velocity and cost together rather than maximizing autonomous activity.
