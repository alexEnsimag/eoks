# Compound Engineering

[Compound Engineering](https://github.com/EveryInc/compound-engineering-plugin) is useful EOKS prior art because it combines a structured software-engineering workflow with an explicit **experience -> learning -> future context** loop.

The important research question is not whether its workflow is preferable to another coding-agent workflow. It is which mechanisms it introduces or combines that are relevant to an AI engineering environment.

## Core mechanism

The public plugin organizes engineering around a recurring loop:

```text
brainstorm -> plan -> work -> review -> compound
                                      |
                                      v
                              durable learning
                                      |
                                      v
                              future planning/work
```

The distinctive part is `compound`: completed work is analyzed for knowledge that would otherwise be lost, checked against existing repository knowledge, and written as a reusable engineering artifact when it passes an admission test.

This makes Compound Engineering useful prior art for **learning as a control-loop transition**, rather than merely for workflow orchestration.

## What is genuinely interesting

### 1. Learning admission

The compound step does not treat every observation as memory. Its guidance asks whether a future engineer could reasonably rediscover the lesson from the final code/tests/docs, and whether forgetting it would cause repeated mistakes or substantial rediscovery.

This is an important distinction:

```text
observation
   |
   v
candidate learning
   |
   +--> incidental / recoverable -> discard
   |
   +--> durable + non-obvious + material -> retain
```

EOKS already has the same hypothesis in [session learning](../session-learning.md), but CE provides concrete engineering-workflow prior art for it.

### 2. Knowledge placement

CE distinguishes between information that belongs in:

- tests/types/assertions;
- code comments;
- commits/PRs;
- shared vocabulary;
- durable solution documentation.

This is more useful than a generic "memory store" because persistence is selected according to **authority and lifecycle**.

For EOKS this reinforces:

```text
experience != knowledge != procedure != policy
```

and suggests that learning admission should also choose an appropriate resource representation.

### 3. Consolidation and canonicalization

Before creating a learning, CE searches existing solutions and tries to avoid duplicate knowledge. Its refresh path can keep, update, consolidate, replace or remove existing material.

This is directly relevant to EOKS's memory-lifecycle work:

```text
candidate learning
      |
 overlap / contradiction check
      |
 canonical existing knowledge?
    /                    \\
 yes                      no
 |                         |
update/consolidate       create
          \\             /
           v             v
             durable knowledge
```

The new primitive is therefore better described as **knowledge consolidation** than as "automatic documentation."

### 4. Progressive context disclosure

CE's session-history machinery separates cheap discovery from expensive synthesis and limits deep inspection to relevant sessions. Other parts of the plugin similarly try to avoid loading all possible context up front.

This provides concrete prior art for:

> **progressive context disclosure** — acquire more context only when relevance or uncertainty justifies the cost.

That is compatible with EOKS's existing loadout -> working-set -> context-compilation model.

### 5. Evidence-backed review

The code-review system selects reviewer capabilities based on the actual change, runs independent reviews, then synthesizes/deduplicates findings and validates them against the changed code.

The useful abstraction is not "multi-agent review":

```text
change
  |
capability selection
  |
independent claims
  |
evidence validation / adjudication
  |
accepted findings
```

This connects directly to EOKS's evidence-provider and assurance model.

### 6. Execution state separate from the plan

CE's work phase uses repository/Git state as an important source of execution progress rather than assuming the generated plan is the source of truth.

This reinforces EOKS's separation between:

- intent/plan;
- execution state;
- durable artifacts;
- outcome/evidence.

## What CE does not add to EOKS

The following are useful implementations but should not become new EOKS dimensions merely because CE contains them:

- brainstorm/plan/work as a specific workflow;
- named reviewer personas;
- the `/lfg` autonomous workflow;
- repository-specific Markdown conventions;
- its particular skill taxonomy.

These are implementations of existing EOKS capabilities.

## Comparison with existing EOKS prior art

| Prior art | Strongest overlap with CE | Important difference |
|---|---|---|
| **Superpowers** | disciplined engineering workflow, planning and quality gates | stronger methodology/workflow focus; less centered on engineering-memory consolidation |
| **Conductor-style systems** | task decomposition, execution state, handoffs | orchestration/state is the main abstraction |
| **OpenWiki** | durable generated knowledge and knowledge lifecycle | knowledge representation/lifecycle rather than the complete engineering loop |
| **LangMem / Mem0 / Zep / Hindsight** | extraction, persistence, consolidation and procedural/episodic memory | general memory infrastructure rather than engineering-specific learning loop |
| **TencentDB Agent Memory** | memory + Skills + Wiki + CodeGraph + governed loadouts | reusable resource/memory infrastructure rather than a prescriptive engineering loop |
| **CLAUDE.md management** | canonical local knowledge/policy exposed to future sessions | representation/policy substrate rather than learning extraction |
| **Xirp / Spotify** | session continuity, institutional context, living documentation | broader system/organizational context and session surface |
| **OpenWolf** | lifecycle hooks, experience capture, corrections/known-fixes memory and automatic context updates | **direct prior art for experience capture → learning/memory → future context**; less explicit about learning admission/validation |
| **GrapeRoot** | proactive context optimization | context compilation/runtime integration rather than learning promotion |

The strongest direct matches are therefore **Superpowers + OpenWiki + agent-memory/learning systems + Conductor-style execution**, with Xirp/OpenWolf/GrapeRoot providing complementary mechanisms.

## EOKS synthesis: primitives and MVP

The strongest synthesis is not another EOKS dimension. It is a small set of mechanisms connecting execution experience to future context.

### Semantics vs. mechanisms

Keep the six EOKS dimensions as semantic dimensions:

- INTENT — what/why
- WORKFLOW — what can happen
- CAPABILITIES — what can act
- KNOWLEDGE — what is known
- STATE — what is currently true/in progress
- POLICY — what constrains or governs action

Define operational primitives underneath them:

| Primitive | Meaning |
|---|---|
| Experience | observed execution history: traces, outcomes, corrections, discoveries |
| Learning | a candidate durable insight extracted from experience |
| Resource | the durable representation selected for that learning: knowledge, procedure, policy, test, comment, etc. |
| Evidence | provenance/grounds supporting a claim or resource |
| Context | task-specific materialization of eligible resources for an agent |

Dimensions describe what information means; primitives describe how information moves through EOKS.

### Learning is not memory

A Learning should pass an explicit admission gate rather than automatically preserving session history:

Experience
  -> Candidate Learning
  -> admission
     -> discard if recoverable/incidental
     -> retain if durable, non-obvious, material and supported
  -> Consolidation
  -> Resource

### Consolidation is a distinct primitive

Consolidation decides how a candidate learning interacts with existing resources:

CREATE | UPDATE | MERGE | REPLACE | INVALIDATE | DISCARD

The goal is canonical knowledge rather than an ever-growing pile of memories.

### Context is compilation, not storage

A resource existing does not imply that it belongs in the current prompt. Context should be compiled from resources using relevance, authority, freshness, evidence, cost and task. Progressive disclosure means acquiring deeper context only when relevance or uncertainty justifies the cost.

### The MVP

The smallest meaningful EOKS experiment is:

1. Capture an experience from an engineering episode.
2. Extract candidate learnings.
3. Admit or discard them using explicit criteria.
4. Consolidate an admitted learning into a durable resource with provenance.
5. Compile relevant resources into the next task's context.
6. Measure whether the next episode improves.

The core loop is:

Experience -> Learning -> Admission -> Resource -> Context -> Outcome
     ^                                               |
     +-----------------------------------------------+

The MVP does not require a new workflow engine, autonomous fleet, universal memory database, or seventh EOKS dimension. It needs a reliable feedback loop and instrumentation to compare it against simpler baselines such as raw session history or static repository instructions.

Core hypothesis:

> Can an AI engineering environment improve over repeated engineering episodes by selectively converting experience into validated reusable resources and compiling those resources into future work?

### Resource authority and provenance

A useful resource model should preserve:

Resource
  - content
  - scope
  - provenance
  - evidence
  - authority
  - freshness
  - lifecycle

This supports a promotion path:

experience -> candidate -> evidence -> knowledge -> procedure -> policy

Promotion represents increasing authority, not merely changing a label. It also makes invalidation possible when source evidence or implementation changes.

### OpenWolf changes the comparison

OpenWolf should be treated as **direct learning-loop prior art**, not merely complementary context infrastructure. Its lifecycle hooks observe agent activity; its project memory separates session state from candidate conventions/corrections and known problems/fixes; and its saved context is reused by later sessions. The current implementation exposes this through files such as `cerebrum.md`, `buglog.json`, `memory.md`, `STATUS.md` and handoff state. This closely matches the EOKS transition from **Experience to reusable Resource to future Context**.

The important remaining gap is that OpenWolf's learning is comparatively operational and harness-driven: it records/maintains useful project memory around observed work. It does not provide the same explicit **admission test + evidence-backed consolidation + outcome validation** that EOKS is proposing to study. OpenWolf is therefore strong prior art for the **capture/integration side** of the loop, while Compound Engineering is stronger prior art for the **reflection/admission/consolidation side**.

This suggests a cleaner decomposition:

```text
agent hooks / runtime
        |
        v
     Experience
        |
        +------ OpenWolf: capture + operational memory
        |
        v
 Candidate Learning
        |
        +------ CE: admission + consolidation
        |
        v
 Durable Resource
        |
        +------ context systems: retrieval / compilation
        |
        v
      Context
        |
        v
   next execution
```

Other current learning-oriented tools reinforce the same decomposition. Everything Claude Code demonstrates lifecycle hooks that persist session state and extract recurring patterns into reusable skills; Engram adds session-learning hooks for persistent memory; and LangMem provides explicit extraction/consolidation primitives across semantic, episodic and procedural memory. These are useful evidence that **learning is a cross-cutting mechanism spanning harness events, reflection, memory and context**, rather than a single storage category.

`Learning` therefore remains a useful EOKS concept, but the MVP should distinguish three different operations: **capture experience**, **admit/consolidate learning**, and **compile resources into context**. Existing tools tend to cover one or two of these rather than the complete loop.
## EOKS interpretation

The combined prior art suggests a more explicit feedback path:

```text
intent
  |
workflow / execution
  |
trace + artifacts + corrections + outcome
  |
reflection
  |
candidate learning
  |
admission + evidence + scope
  |
consolidation / promotion
  |
knowledge / procedure / policy
  |
context compilation
  |
next execution
```

This is already compatible with EOKS's six dimensions. It does not require a seventh dimension.

The useful refinement is to make the **transition between Experience, Knowledge, State and Policy explicit**:

- **Experience** is observed execution history.
- **Knowledge** is a durable representation with provenance and scope.
- **Procedure/Skill** is reusable behavioral knowledge.
- **Policy** is trusted/authoritative constraint or decision logic.
- **State** records current execution/work status and other mutable facts.
- **Context** is the task-specific materialization of eligible resources/evidence.

Compound Engineering is evidence that these transitions can be implemented as part of a coding-agent environment rather than left as manual documentation work.

## Research questions

1. Does learning admission improve future task outcomes compared with simply retaining more session history?
2. How should consolidation handle contradictory learnings?
3. What evidence is sufficient to promote a candidate procedure to trusted project policy?
4. How should freshness and invalidation propagate from source changes to derived learnings?
5. Can progressive context disclosure reduce cost without reducing evidence coverage?
6. Can capability selection based on the change/question outperform fixed reviewer sets?
7. Which learning artifacts are better represented as documents, Skills, tests, executable guards or structured resources?

The most valuable EOKS experiment is therefore not "reimplement Compound Engineering." It is to measure the **experience -> candidate -> validated reusable resource -> future outcome** loop and compare it with simpler memory/context baselines.
