# Agent memory and continuous-learning prior art

This note places agent-memory systems in the EOKS model. These are capability references, not dependencies or endorsements.

## Why this prior art matters

The second-brain discussion exposed a distinction that is easy to miss when comparing memory systems: remembering facts is different from learning procedures.

For EOKS, the relevant progression is:

```text
semantic memory   -> facts / knowledge
episodic memory   -> experiences / examples
procedural memory -> behavior / rules / procedures
```

Agent-memory libraries increasingly expose some combination of these concepts, but they generally operate at the memory-management layer rather than solving the complete EOKS control-loop problem.

## LangMem

LangMem is particularly relevant prior art because it explicitly describes long-term memory in terms of semantic, episodic and procedural memory. Its primitives can extract and consolidate memories, operate in the hot path, or perform background reflection. It also includes prompt-optimization primitives that can update agent behavior based on experience.

For EOKS, this demonstrates that memory can be treated as an evolving state rather than a static vector database.

The important EOKS distinction is that LangMem is primarily a library layer for memory operations and storage integration. EOKS asks the broader control-plane questions:

- which observations should be collected;
- which evidence providers should contribute evidence;
- which patterns are worth learning;
- how learned procedures are validated;
- when a procedure becomes execution policy;
- how policy interacts with task scheduling, context assembly, model selection and evaluation;
- how outcomes feed the next learning cycle.

LangMem therefore fits naturally as potential prior art for the **memory/learning mechanism**, not as the definition of the EOKS architecture.

## Mem0

Mem0 is useful prior art for persistent memory infrastructure and memory extraction. Its main relevance is the idea that an agent can maintain information across interactions instead of treating each context window as isolated.

For EOKS, the important question remains what trust level a memory has and whether it represents a fact, an episode, or a proposed procedure. Persistence alone does not establish correctness or usefulness.

## Zep

Zep represents another approach to long-term agent memory and retrieval. It reinforces the distinction between conversational/session history and information that should persist beyond an individual interaction.

The EOKS concern is the lifecycle around that persistent information: provenance, freshness, invalidation, task relevance and evaluation.

## OpenHands and agent execution traces

Agent execution systems provide a complementary source of evidence: the trajectory of an agent performing a software-engineering task.

The useful EOKS insight is that an execution trace can become an input to a separate reflection process. The execution runtime does not need to learn its own policies synchronously.

```text
execution runtime
      |
      v
trace / outcome
      |
      v
background reflection
      |
      v
candidate memory / procedure
```

This separation is particularly attractive for development workflows because analyzing many completed tasks is more useful for discovering behavioral patterns than analyzing a single task in isolation.

## Memory is not the control loop

A memory system generally answers:

> What information should be retained and retrieved later?

EOKS asks the larger question:

> Given everything the system knows about the project, task, tools, models, previous outcomes and learned procedures, what should happen next?

That difference can be expressed as:

```text
memory
  -> retain / retrieve

EOKS control loop
  -> observe
  -> understand
  -> schedule
  -> assemble context
  -> execute
  -> evaluate
  -> learn
  -> adapt
```

Memory is therefore a resource used by the control loop, not a replacement for it.

## Trust and lifecycle

Agent-memory prior art also strengthens an existing EOKS principle: generated memories need lifecycle semantics.

A useful model is:

```text
observation
    |
    v
candidate memory
    |
    +--> semantic fact
    +--> episode
    +--> candidate procedure
    |
    v
validation / evidence
    |
    v
promoted memory / policy
    |
    v
retrieval + execution
    |
    v
outcome
    |
    +------> new observation
```

This prevents the common failure mode where a plausible generated summary becomes an unquestioned source of truth.

## Capability map

| Capability | Typical memory tooling | EOKS concern |
|---|---|---|
| Persist facts | semantic memory | canonical/project knowledge |
| Recall history | episodic memory | relevant prior episodes |
| Extract memories | reflection/extraction | candidate knowledge |
| Consolidate memories | memory manager | deduplication + correction |
| Learn procedures | procedural memory / prompt optimization | validated skills and policies |
| Trace execution | agent runtime / observability | evidence and outcomes |
| Choose what to do next | usually outside memory library | EOKS control plane |
| Evaluate learned behavior | application-specific | EOKS evaluation |

The boundary is intentional: EOKS should compose memory capabilities instead of reinventing every storage or extraction mechanism.

## Current limitation of the prior art

The interesting gap is not simply "agents need memory." Mature components already address persistence, retrieval and background extraction.

The harder research problem is **developer-specific behavioral learning**:

```text
months of real development work
          |
          v
identify stable patterns in how a developer/team works
          |
          v
separate useful procedures from incidental habits
          |
          v
validate them against outcomes
          |
          v
turn them into reusable agent behavior
```

That is the part most directly aligned with the EOKS continuous-learning hypothesis.
