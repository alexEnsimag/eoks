# Context Workbench

EOKS should treat model context as an explicit, inspectable artifact rather than an opaque side effect of a long conversation or a sequence of repository searches.

The **Context Workbench** is a proposed human-facing and machine-facing layer for constructing, inspecting, optimizing and evaluating task-specific context before it is supplied to a reasoning step.

## Motivation

A strong model can still waste substantial context on repository discovery. This is especially visible with coding subagents: a subagent starts with an isolated context, reconstructs architecture through `grep`/search/read operations, and may spend tens of thousands of tokens rediscovering information that another agent already established.

The desired architecture is:

```text
persistent knowledge + evidence providers
                    |
                    v
             context compiler
                    |
             context workbench
          /                     \
     automatic                human
      selection              inspection/editing
          \                     /
           +---------+---------+
                     |
              compiled context
                     |
                   model
```

This is different from model routing. Routing chooses **which model** should perform a reasoning step. Context compilation chooses **what information that model should see**. Context should generally be optimized before introducing model routing as the primary cost intervention.

## Context blocks

The fundamental unit should be an inspectable **context block** rather than an undifferentiated prompt fragment.

A block may represent:

- canonical knowledge;
- a decision or invariant;
- structural code evidence;
- semantic/domain evidence;
- a dependency slice;
- a test or runtime observation;
- a Git change or historical event;
- a previous workflow discovery;
- a procedure or skill;
- a task constraint;
- a working hypothesis.

A conceptual block schema is:

```yaml
id: auth-architecture
type: knowledge
content: "..."
source:
  kind: canonical
  references:
    - api/CLAUDE.md
    - docs/adr/017-auth.md
provenance:
  revision: abc123
confidence: high
freshness:
  verified_at: 2026-08-08
relevance:
  task: 0.93
dependencies:
  - request-lifecycle
token_cost: 420
```

The schema is deliberately illustrative rather than normative. EOKS should first validate the abstraction before standardizing a serialization format.

## Context layers

A useful decomposition is:

| Layer | Purpose | Typical contents |
|---|---|---|
| L0 — Task | What is being attempted? | user request, objective |
| L1 — Constraints | What must or must not change? | requirements, scope, policies |
| L2 — Persistent knowledge | What is already known? | architecture, decisions, invariants, domain facts |
| L3 — Structural context | Where is relevant evidence? | packages, symbols, dependency/call graphs |
| L4 — Evidence | What does the source actually show? | code, tests, logs, Git history, runtime observations |
| L5 — Working memory | What has this workflow discovered? | hypotheses, findings, intermediate conclusions |
| L6 — Reasoning state | What is the model currently considering? | alternatives, assumptions, unresolved questions |

Different workflow nodes can request different layer budgets. A planning step may need more L2/L3 and less raw L4; implementation may need a narrow L3/L4 slice; verification may need tests, changed files and relevant invariants.

This separation should be an **information-architecture boundary**, not a prescription for how the model must reason. If the structure does not improve cost, quality or observability, it should not be imposed merely for aesthetic reasons.

## Context quality

"Context entropy" is useful as an intuition, but EOKS should avoid assuming that a single scalar entropy metric captures context quality. Instead, expose several measurable dimensions:

- **Relevance** — how directly does the block help with the task?
- **Coverage** — what necessary information is still missing?
- **Redundancy** — how much useful information is duplicated?
- **Correctness / reliability** — how trustworthy is the source?
- **Uncertainty** — which claims are inferred rather than established?
- **Freshness** — could the information be stale relative to the repository/world revision?
- **Dependency completeness** — are required related facts missing?
- **Provenance** — can claims be traced to evidence?
- **Contradiction risk** — does this block conflict with another included block?
- **Structure / ordering** — is the information presented in a useful order?
- **Token and latency cost** — what resources does inclusion consume?
- **Model interaction** — does this context construction work well for the selected model?

The objective is **useful evidence per unit of context and reasoning cost**, not maximum context utilization.

## Marginal context value

A promising research metric is the marginal value of adding a block:

```text
marginal context value ~=
    change in expected task quality
    -------------------------------
            token / latency cost
```

This should not be treated as an exact online probability calculation initially. It is a useful experimental framing for comparing context-selection policies.

A workbench could therefore show, for example:

```text
Block                         Value / cost
------------------------------------------------
auth/middleware.go            +++++++ / 1.2k
Slack architecture             +++++ / 0.4k
ADR-017                         ++++ / 0.2k
old Slack client                + / 2.4k
unrelated integration tests    + / 3.0k
```

Over time, observed task outcomes could train or calibrate these estimates.

## Interactive context construction

The workbench should support both automatic and human-controlled construction.

Useful operations include:

- **include** a block;
- **exclude** a block;
- pin a block as required;
- mark a block as optional;
- expand a block to authoritative evidence;
- collapse evidence into a pointer/summary;
- inspect provenance;
- inspect why a block was selected;
- inspect why another block was omitted;
- compare two context compositions;
- set a token/latency budget;
- ask the compiler to optimize within the budget;
- save a successful context recipe for future tasks.

A graph view can complement the block view. The graph should show relationships between knowledge, code, evidence, decisions and task requirements rather than merely reproducing the repository's file dependency graph.

## Explainable context

Every automatically selected block should have an inspectable selection rationale, for example:

```text
Why is slack/events.go included?

✓ task mentions Slack authentication
✓ imports slack/auth.go
✓ referenced by decision ADR-017
✓ modified by the recent failing change

Relevance: 0.91
Estimated cost: 1,240 tokens
```

This makes context a first-class observable decision rather than hidden prompt magic.

## Context diff

A high-value interaction is a **context diff** between an automatically assembled context and an optimized context:

```diff
- src/old/slack_client.go        4.2k
- tests/integration/slack_old.go 2.1k
- docs/general-architecture.md  1.8k
- unrelated API routes           3.4k

+ ADR-017                         0.2k
+ Slack architecture             0.4k
+ relevant dependency slice      4.8k
+ failing test                    0.6k

42.8k -> 19.6k estimated input tokens
```

The user should be able to review the diff and then run the task with the optimized composition.

## Context contracts for subagents

Subagents should receive an explicit context contract when possible instead of being asked to rediscover everything:

```yaml
task: investigate Slack authentication failure
known_facts:
  - API authentication uses middleware X
  - Slack requests use validator Y
relevant_nodes:
  - auth/middleware.go
  - slack/auth.go
  - slack/events.go
question: determine why Slack requests bypass X
do_not_explore:
  - frontend/
  - unrelated services/
```

This does not prohibit exploration. It establishes a starting context and an explicit scope. If the subagent discovers that the contract is incomplete, the additional evidence should be recorded and can feed back into the context compiler.

## Context as a compiler artifact

The compiled context should be treated as a reproducible artifact:

```text
Task + policy + workflow + repository revision
                  |
           evidence providers
                  |
           selection/ranking
                  |
        deduplication/conflict checks
                  |
         budget optimization
                  |
          compiled context
                  |
                model
```

The artifact should be observable enough to answer:

- what the model saw;
- what it did not see;
- which provider supplied each item;
- why each item was selected;
- how much each item cost;
- which revision/freshness state applied;
- which context policy produced the result.

## Learning from human edits

Manual include/exclude actions are valuable feedback. If users repeatedly remove the same blocks from successful workflows, EOKS can learn that those blocks are poor defaults for similar task classes. If users repeatedly add a particular decision or dependency slice, the compiler can learn to prioritize it.

The learning loop should remain evidence- and outcome-aware:

```text
compiler selection
       |
 human edits / model actions
       |
 task outcome / evaluation
       |
 candidate context policy update
       |
 validation
       |
 future selection
```

A manual edit alone is not proof that a block is universally irrelevant; task class, model, repository revision and outcome must be considered.

## Relationship to `/compact` and session boundaries

Conversation compaction and context compilation solve different problems. Compaction attempts to preserve useful information inside a continuing conversation. Context compilation makes durable knowledge and authoritative evidence available again after context is cleared or a fresh subagent starts.

A long-lived EOKS workflow should therefore prefer:

```text
session
  |
  +--> discoveries / decisions / evidence
  |
  v
promote durable knowledge selectively
  |
  v
new session or subagent
  |
  v
compile minimum sufficient context
```

This makes `/compact` optional rather than the only mechanism for preserving continuity.

## Relationship to model routing

Model routing is orthogonal:

```text
                  task
                   |
            context compilation
                   |
             optimized context
                   |
            complexity/capability
                   |
             model selection
                   |
                 model
```

For workflows that prefer a strong model such as Opus, context optimization can be evaluated independently before introducing routing. A router cannot solve waste caused by a strong model repeatedly rediscovering the same repository.

Once context is controlled, routing becomes easier to evaluate because model comparisons can be performed on a more stable input context.

## UI concept

A first prototype does not need to be a full IDE. A useful workbench can have four views:

1. **Blocks** — cards grouped by layer/type, with token cost and provenance.
2. **Graph** — relationships among task, knowledge, code and evidence.
3. **Context diff** — automatically selected versus optimized composition.
4. **Quality** — relevance, coverage, redundancy, uncertainty, freshness, cost and outcome history.

The workbench should make context **inspectable without making it mandatory to edit manually**. Automatic selection remains the default; humans intervene when they want control or when the compiler is uncertain.

## Initial experiment

The smallest useful experiment is not a production UI. Build a local context compiler that can:

1. accept a task;
2. query a few deterministic and semantic evidence providers;
3. produce typed context blocks;
4. assemble a bounded context;
5. explain inclusion/exclusion;
6. emit a context manifest;
7. run the same benchmark task with alternative compositions;
8. record tokens, tool calls, latency and task outcome.

Only after this demonstrates a useful signal should EOKS invest in a richer interactive graph/workbench.
