# Hindsight and OKF: memory engine vs knowledge format

This note captures a key distinction from the Claude Code auto-learning research: **Hindsight and OKF are complementary rather than competing knowledge-base solutions.**

## Hindsight

Hindsight is an agent-memory system designed around learning over time. Its model distinguishes world facts, experiences and mental models, with `retain`, `recall` and `reflect` as the core operations. Its implementation combines structured extraction, entities/relationships, temporal information and multiple retrieval strategies rather than treating memory as only a vector store.

For EOKS, this makes Hindsight useful prior art for the **memory/learning subsystem**:

```text
execution
   -> retain observations / experiences
   -> recall relevant history
   -> reflect across accumulated evidence
   -> learned mental model
```

Its Claude Code integration is particularly relevant: it combines hooks for automatic recall/retain with MCP tools for explicit memory operations, and supports isolated/dynamic memory banks for agents/projects/sessions.

The important EOKS caveat is that memory retrieval is not the same as controlled policy promotion. A useful memory may remain evidence; a generalized procedure needs stronger validation before becoming a Skill or policy.

## OKF

The Open Knowledge Format is deliberately much simpler. OKF v0.2 represents a knowledge bundle as Markdown concepts with YAML frontmatter and standard links. Provenance, trust, lifecycle and attestation are represented in metadata, while storage, retrieval and serving are deliberately left unspecified.

For EOKS, OKF is therefore interesting as a **portable durable knowledge representation**:

```text
candidate learning
       |
 validation / promotion
       v
   OKF bundle
       |
  +----+----+
  |         |
agents    humans
```

It can live in Git, be reviewed through diffs/PRs, and be consumed by many different tools. It does not require a database, vector index, graph service or particular model provider.

## Comparison

| Dimension | Hindsight | OKF |
|---|---|---|
| Problem | persistent memory and learning | interoperable knowledge representation |
| Primary unit | memory / experience / mental model | concept document |
| Runtime | memory service/API/MCP | none required |
| Recall | built in, multi-strategy | consumer-defined |
| Reflection | built in | not prescribed |
| Persistence | memory database | files / Git |
| Provenance | memory metadata/relationships | first-class document metadata |
| Human review | possible, but not the core abstraction | natural through Git/editor workflows |
| Claude Code integration | direct integrations exist | files are directly consumable |

## EOKS interpretation

The strongest composition is:

```text
Claude Code
   |
 hooks / traces
   v
Hindsight-like memory
   |
 retain / recall / reflect
   v
candidate Learning Records
   |
 validation + provenance + outcomes
   v
OKF durable knowledge
   |
   +--> context retrieval
   +--> Skill proposals
   +--> project policy candidates
```

This preserves a clean separation between:

- **experience** — what happened;
- **memory** — what may be useful later;
- **knowledge** — validated durable understanding;
- **procedure** — how to perform a task;
- **policy** — what the control plane should prefer or require.

## Why not simply use Hindsight as the whole system?

A memory engine is optimized for recall and learning. EOKS also needs:

- explicit promotion semantics;
- version-controlled canonical knowledge;
- provenance and review;
- policy ownership;
- retrieval/context decisions;
- evaluation of downstream behavior;
- model/tool/workflow selection.

Those concerns can surround a memory engine without being implemented by it.

## Why not simply use OKF as the whole system?

OKF is a format, intentionally not a runtime. It does not prescribe:

- how session traces are captured;
- how memories are extracted;
- semantic/temporal retrieval;
- reflection;
- memory consolidation;
- agent hooks;
- context injection.

This is a feature rather than a deficiency: EOKS can use OKF as a stable boundary while experimenting with different learning and retrieval implementations.

## Research hypothesis

A useful EOKS implementation should be able to replace Hindsight with another memory backend, or replace OKF with another interoperable representation, without changing the control-plane concepts.

The abstraction to test is therefore not "Hindsight + OKF". It is:

```text
observe
  -> retain
  -> recall / reflect
  -> candidate knowledge
  -> validate / promote
  -> durable representation
  -> retrieve
  -> execute
  -> evaluate
```

The backend and file format are implementation choices; the lifecycle is the architectural hypothesis.
