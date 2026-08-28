# Claude Code workflow systems and plugin taxonomy

This note captures the Claude Code workflow discussion as EOKS prior art. The goal is not to recommend one plugin stack, but to identify reusable capabilities that EOKS may eventually model as policies, resources and feedback loops.

## The useful decomposition

Claude Code extensions are easier to reason about as capabilities than as a list of plugins. A practical taxonomy is:

```text
                    CODING WORKLOAD
                           |
        +------------------+------------------+
        |                  |                  |
   UNDERSTAND /        DESIGN /          EXECUTE / REVIEW
   IMPROVE              PLAN              DELIVER
        |                  |                  |
 architecture       workflow discipline   tests / review
 code understanding  decomposition         refactoring
        |                  |                  |
        +------------------+------------------+
                           |
                    KNOWLEDGE LOOP
                           |
             observe -> extract -> validate
                       -> promote
                       -> retrieve
```

The earlier "existing code vs new features" split is useful as a workload distinction, but it is not the best architectural taxonomy. The same capabilities can be used in both workflows.

## 1. Execution workflow: Superpowers

**Superpowers** is a strong example of an execution-discipline layer. Its value is not merely that it can plan work; it provides an opinionated development workflow around activities such as specification/brainstorming, implementation, testing and review.

For EOKS, the important abstraction is **workflow policy**:

- what stages a task should pass through;
- which evidence is required before advancing;
- which checks are deterministic;
- when an implementation should be reviewed or simplified;
- what artifacts should survive the task.

Superpowers should therefore be treated as execution/workflow prior art, not as something EOKS needs to reproduce wholesale.

## 2. Architecture/codebase analysis: modularity and related tools

**modularity** is interesting because it applies an explicit architectural concept rather than simply asking an LLM to "review the code". That makes it valuable prior art for EOKS even if it is not a must-have in an eventual user stack.

The key lesson is more general:

> Prefer bounded analyses with an explicit model of correctness over generic agent intuition when evaluating architecture.

This belongs near the **evidence/evaluation plane**. Architecture analysis can produce structured evidence about boundaries, coupling or dependencies; the control plane can decide when that evidence is worth obtaining.

It is therefore complementary to execution workflows rather than a replacement for them.

## 3. Conductor and orchestration overlap

Conductor-style systems overlap substantially with Superpowers around planning and execution. Combining two opinionated workflow engines without a clear ownership boundary can produce duplicate planning, additional context and conflicting task state.

The useful EOKS abstraction is not "install both". It is to distinguish:

- **workload orchestration** — decomposition, dependency management and scheduling across tasks;
- **task execution policy** — how one task is designed, implemented, tested and reviewed.

An EOKS control plane could support both without coupling itself to either project's command vocabulary.

## 4. Memory and CLAUDE.md management

A project-level `CLAUDE.md` and a semantic memory system solve different problems.

### Project instructions / canonical knowledge

A `CLAUDE.md`-style file is useful for high-signal, relatively stable information that should be available reliably:

- project invariants;
- conventions;
- architectural constraints;
- important commands;
- durable decisions;
- explicit instructions.

This is closer to **canonical project knowledge / policy** than to general memory.

### Semantic memory

A system such as **memsearch** is useful for retrieving potentially relevant historical information. Retrieval is probabilistic and should therefore not automatically make a recalled item a source of truth.

A useful EOKS distinction is:

```text
canonical knowledge / policy  -> stable, validated, explicit
semantic memory               -> recall candidates / historical evidence
```

These layers should be composable rather than treated as substitutes.

## 5. Session lifecycle and the missing feedback loop

A particularly useful pattern is a session-finalization step. A hook or explicit `/finalize` command could turn an agent session into structured candidate knowledge.

```text
session work
    |
    v
observations + decisions + failures + outcomes
    |
    v
candidate extraction
    |
    +--> candidate rules
    +--> candidate skills
    +--> candidate project facts
    +--> candidate architectural decisions
    |
    v
validation / promotion
    |
    +--> canonical project knowledge
    +--> semantic memory
    +--> evaluation/telemetry
```

Important design constraint: **automatic extraction should produce candidates, not silently rewrite canonical knowledge**. A session-end hook can suggest updates, but promotion should consider evidence, freshness and contradictions.

A session-start hook can then retrieve relevant durable knowledge and task-specific memory. This creates a genuine feedback loop rather than simply adding a larger static prompt.

## 6. Skills and rules as learned artifacts

Repeated agent interactions can expose opportunities for reusable skills and rules.

Useful signals include:

- the same user correction recurring across sessions;
- repeated sequences of tool calls;
- recurring debugging procedures;
- repeated architecture constraints;
- a task repeatedly requiring the same evidence assembly;
- a workaround that becomes a project convention.

A candidate skill/rule should carry provenance: what sessions or evidence caused the proposal, what scope it applies to, and whether it has been validated.

This suggests treating skills and rules as **knowledge artifacts with lifecycle**, not as arbitrary prompt files:

```text
candidate -> review -> validated -> active -> superseded
```

## 7. Simplicity as a continuous policy

The discussion around `/simplify` exposed another useful distinction. A post-task simplification pass is different from a policy that influences design continuously.

Three levels are useful:

1. **Design constraint** — prefer the simplest solution while planning.
2. **Execution/review policy** — periodically ask whether new code introduced unnecessary complexity.
3. **Post-task simplification** — run a focused cleanup pass after implementation.

A project can encode a small number of explicit simplicity rules in its canonical project knowledge, while keeping a `/simplify`-style pass as a separate verification activity.

The EOKS lesson is not that "minimalism" should always win. It is that **complexity is an evaluable workload property** and can become a policy signal alongside correctness, cost and maintainability.

## Recommended minimal stack as an experiment

For an individual Claude Code user, a low-overlap starting point is:

- one execution workflow (for example, Superpowers);
- one memory/retrieval system (for example, memsearch);
- project-level canonical instructions/knowledge (`CLAUDE.md` or an equivalent EOKS representation);
- optional architecture/evidence tooling when the workload warrants it;
- a small session-finalization workflow that proposes durable updates.

Avoid stacking multiple workflow engines merely because they are popular. The important question is which policy each component owns and what state it produces.

## EOKS mapping

| Claude Code capability | EOKS interpretation |
| --- | --- |
| Superpowers-style workflow | execution policy / workflow controller |
| Conductor-style decomposition | task orchestration / scheduling |
| modularity-style architecture analysis | evidence provider / evaluation |
| `CLAUDE.md` | canonical project knowledge + policy |
| memsearch-style retrieval | memory retrieval / context assembly |
| `/simplify` | evaluation/refactoring pass |
| session-finalizer hook | observation -> knowledge extraction |
| skill/rule suggestions | candidate knowledge generation |

The architectural principle is to compose these capabilities through explicit contracts rather than make EOKS another monolithic coding-agent plugin.