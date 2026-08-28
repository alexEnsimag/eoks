# Claude Code learning stack: OKF, Hindsight, Skills and hooks

## Why this belongs in EOKS

The Claude Code discussion exposed a useful decomposition of the "auto-learning knowledge base" problem. The tools are not interchangeable:

- **`CLAUDE.md`** — always-available canonical project instructions and policy.
- **Skills** — reusable procedures that should be loaded when their task class is relevant.
- **Hooks** — deterministic observation/automation points around agent execution.
- **Memory engines such as Hindsight** — persistent, queryable experience and learned mental models.
- **OKF** — a portable representation for durable knowledge that can be read, diffed and versioned independently of a particular runtime.
- **Obsidian / graph viewers / GraphRAG / code graphs** — possible authoring, visualization, retrieval or evidence mechanisms, not the architectural center.

EOKS should coordinate these capabilities rather than turn any one of them into the EOKS model.

## The important distinction

A useful mental model is:

```text
                    CLAUDE CODE
                         |
                  hooks / traces
                         |
                         v
                memory / learning
                         |
              +----------+----------+
              |                     |
         episodic evidence      reflection
              |                     |
              +----------+----------+
                         |
                   candidate knowledge
                         |
              validation / provenance
                         |
                         v
                 durable knowledge
                         |
                +--------+--------+
                |                 |
               OKF              Skills
                |                 |
          facts / patterns    procedures
          decisions / docs    workflows
                |                 |
                +--------+--------+
                         |
                   retrieval / policy
                         |
                    future task
```

The key EOKS boundary is **promotion**. A session observation is evidence, not automatically a new rule. A repeated behavior is a candidate pattern, not automatically a skill. A generated knowledge document is not automatically canonical truth.

## `CLAUDE.md`: canonical baseline

Keep `CLAUDE.md` small and high-signal. It is best for information that should influence almost every Claude Code session:

- project invariants;
- engineering principles;
- conventions;
- important architectural constraints;
- durable decisions that are genuinely global.

It should not become a transcript archive or a giant procedural manual. Detailed workflows belong in Skills or task-specific knowledge.

Conceptually:

```text
CLAUDE.md = "how the agent should behave by default"
```

## Skills: procedural knowledge

Claude Code Skills are a natural target for validated procedural memory. A repeated workflow can become a focused skill rather than another paragraph in `CLAUDE.md`.

Example candidate:

```text
.claude/skills/telemetry-change-investigation/SKILL.md

Use when changing the telemetry pipeline.

1. Inspect consumer topology.
2. Check batching and delivery assumptions.
3. Establish a baseline.
4. Make the smallest change.
5. Run targeted tests/benchmarks.
6. Verify the resulting event flow.
```

EOKS should care about the **skill lifecycle**, not the particular Claude Code file format:

```text
observed procedure
  -> repeated evidence
  -> outcome validation
  -> candidate skill
  -> controlled rollout
  -> evaluation
  -> promote / revise / deprecate
```

## Hooks: observation and deterministic automation

Hooks are particularly valuable at the boundary of an agent runtime because they can capture lifecycle events without requiring the model to remember to do so.

Potential EOKS uses include:

- session start/end;
- tool execution and failures;
- changed files;
- tests and verification;
- commits and PRs;
- explicit human corrections;
- extraction of a completed-session trace;
- triggering asynchronous learning jobs.

A hook should preferably **record evidence or trigger analysis**, not silently mutate trusted policy.

```text
hook
  -> trace / artifact
  -> background extraction
  -> candidate knowledge
```

## Hindsight: persistent memory that learns

Hindsight is important prior art because it goes beyond a persistent transcript/vector store. Its current model separates world facts, experiences and learned mental models, and exposes `retain`, `recall` and `reflect` operations. It combines structured extraction, entities/relationships, temporal information and multiple retrieval strategies. It also provides Claude Code integration with automatic recall/retain and explicit MCP knowledge tools.

This makes Hindsight much closer to an **EOKS memory/learning subsystem** than to OKF.

The useful decomposition is:

```text
Hindsight
  retain  -> accumulate observations / experiences
  recall  -> retrieve relevant past knowledge
  reflect -> synthesize higher-level understanding

EOKS
  observe -> validate -> promote -> control -> evaluate
```

Hindsight can therefore be an implementation/reference for the memory side of the EOKS learning plane, while EOKS remains responsible for deciding how memory relates to canonical knowledge, skills, policies, evidence and evaluation.

### Why Hindsight is not OKF

They solve different problems:

| | Hindsight | OKF |
|---|---|---|
| Primary role | memory / learning engine | knowledge representation format |
| Runtime | service/database + APIs/MCP | none required |
| Main unit | memories, experiences, mental models | knowledge concepts/documents |
| Retrieval | built in | consumer-defined |
| Reflection | built in | not prescribed |
| Provenance/lifecycle | memory-system semantics | represented in document metadata |
| Portability | via API/integrations | plain files/Git |

They are complementary. Hindsight can learn; OKF can provide a portable durable representation of what was promoted.

## OKF: canonical portable knowledge

OKF is attractive for EOKS because it deliberately does **not** require a hosted knowledge service. Its v0.2 specification defines a knowledge bundle as Markdown concepts with YAML frontmatter, standard links, optional indexes/logs, and first-class provenance/trust/lifecycle/attestation metadata.

That makes it a strong candidate for the **durable knowledge interchange layer**:

```text
Hindsight / learning system
          |
          | validated promotion
          v
      OKF bundle
          |
     +----+----+
     |         |
 Claude Code  other agents
     |         |
    Skills / context / retrieval
```

The important caveat is that OKF is a format, not a memory or retrieval engine. A knowledge bundle can be indexed, searched, embedded, displayed in a graph, or read directly from Git. EOKS should not assume one serving mechanism.

## Why not make Obsidian or GraphRAG the center?

These tools can be useful consumers or interfaces, but they solve a different layer.

### Obsidian

Useful for human exploration and editing. It is not necessary for an agent-first architecture and can become another manually maintained system. An OKF-compatible repository could be opened in a human knowledge UI without making that UI the source of truth.

### GraphRAG / knowledge graphs

Graphs are useful when relationships are important and retrieval benefits from traversals. They are not inherently the right canonical representation. EOKS should introduce a graph when evidence demonstrates that graph retrieval improves a workload, rather than because the corpus "looks like a graph".

### Code graphs / GitNexus / Graphify-style tools

These are better understood as **evidence providers** for repository structure: symbols, dependencies, callers, impact and relationships. They can feed the context/learning loop but should not be confused with durable behavioral memory.

## Proposed EOKS learning architecture

For a Claude Code proving ground, the simplest useful architecture is:

```text
Claude Code
    |
    +-- CLAUDE.md --------------------> baseline policy
    |
    +-- Skills -----------------------> reusable procedures
    |
    +-- Hooks ------------------------> session / execution traces
                                           |
                                           v
                                    memory / learning engine
                                      (e.g. Hindsight)
                                           |
                              retain / recall / reflect
                                           |
                                           v
                                  candidate knowledge
                                           |
                              provenance + validation
                                           |
                                           v
                                     OKF knowledge
                                           |
                              +------------+------------+
                              |                         |
                       future retrieval           skill proposals
                              |                         |
                              +------------+------------+
                                           |
                                      next task
                                           |
                                      evaluation
```

This preserves the strongest properties of each layer without requiring EOKS to own a particular database, graph, editor or agent runtime.

## Automatic learning should be background-first

The most promising default is to collect rich evidence during execution but perform expensive generalization asynchronously:

```text
hot path:
  execute -> observe -> trace

background:
  traces -> episodes -> pattern mining -> reflection
         -> candidate Learning Records
         -> validate / compare outcomes
         -> propose knowledge or skills
```

This avoids adding large latency to every coding task and gives the learner enough historical evidence to distinguish one-off behavior from a recurring procedure.

A useful Learning Record remains the EOKS abstraction:

```text
situation
strategy / action
supporting evidence
outcome
confidence
scope
provenance
validity
status
```

## Promotion policy

A practical policy is:

- **Observe automatically.**
- **Extract automatically.**
- **Retrieve automatically when useful.**
- **Promote cautiously.**
- **Evaluate promoted behavior.**
- **Keep provenance and rollback paths.**

Promotion should consider repetition, outcome quality, scope, contradictory evidence, freshness and cost. A single successful session should rarely rewrite global instructions.

For example:

```text
3 sessions observed
      |
      v
same investigation strategy
      |
      +--> 2 successful outcomes
      +--> 1 counterexample
      |
      v
candidate pattern, scoped to telemetry changes
      |
      v
human / evaluation gate
      |
      +--> promote as Skill
      +--> keep as evidence
      +--> reject / wait for more evidence
```

## Evaluation is part of learning

The learning loop is incomplete without measuring whether the promoted knowledge actually helps.

Useful metrics include:

- task success rate;
- time to useful evidence;
- number of retries / corrections;
- test failures;
- human interventions;
- context size/cost;
- regression rate after a learned procedure is applied;
- frequency of stale or contradictory recalls.

The falsifiable question is not whether a memory system can remember more. It is whether the additional memory and learning machinery **improves engineering outcomes enough to justify its complexity**.

## Minimal experiment

Do not start by building GraphRAG, a full knowledge graph and an elaborate second brain.

A useful first experiment is:

1. Claude Code session hooks produce structured traces.
2. Store traces in a simple local durable store.
3. Use a background learner to extract episodes and candidate Learning Records.
4. Use Hindsight, or a similarly capable memory backend, to test recall/reflection.
5. Promote only a small number of validated candidates into an OKF repository.
6. Convert selected procedural knowledge into Claude Code Skills.
7. Compare future task outcomes against a baseline without learned knowledge.

This experiment directly tests the EOKS hypothesis while keeping each implementation replaceable.

## Current architectural conclusion

The strongest synthesis from this research is:

> **Memory, knowledge and behavior are different resources. EOKS should coordinate their lifecycle rather than collapse them into one "AI memory" component.**

Hindsight is compelling prior art for the **learning/memory engine**. OKF is compelling prior art for the **portable durable knowledge layer**. Claude Code hooks and Skills provide a practical **execution integration**. `CLAUDE.md` provides the **small canonical baseline**. EOKS is the layer that connects these resources through observation, validation, promotion, retrieval, execution and evaluation.
