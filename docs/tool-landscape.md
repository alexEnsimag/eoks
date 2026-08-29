# EOKS tool landscape

This is the **current-state tool map** for EOKS. It is deliberately more visual and opinionated than the canonical capability model: its purpose is to let a researcher look across the ecosystem and decide which 2–3 tools are worth trying next.

It is a snapshot, not a benchmark. Ratings below represent what EOKS currently knows from research and inspection; they are **not measured EOKS scores** unless explicitly marked as such.

For the formal selection model, see [Tool capability model](tool-capability-model.md) and [Tool selection](tool-selection.md). For detailed source notes, see [`research/tool-notes.md`](../research/tool-notes.md).

## 1. One-glance EOKS map

```mermaid
flowchart LR
    T[Task / Question]
    C[Context]
    K[Knowledge]
    E[Evidence]
    X[Execution]
    V[Evaluation]
    O[Observability]
    P[Policy / Assurance]
    L[Learning / Memory]
    CP[EOKS Control Plane]

    T --> CP
    K --> C
    E --> C
    C --> X
    X --> O
    X --> V
    O --> V
    V --> L
    L --> K
    V --> CP
    P --> CP
    CP --> C
    CP --> E
    CP --> X

    subgraph KnowledgeTools[Knowledge / durable resources]
      OKF[OKF]
      ClaudeMD[CLAUDE.md management]
      Wiki[Tencent Agent Memory Wiki]
      Mem[Memory systems: LangMem / Mem0 / Zep]
    end

    subgraph ContextTools[Context compilation]
      GR[GrapeRoot]
      CAS[CodeSight]
      Xirp[Xirp / Spotify]
    end

    subgraph StructureTools[Structural evidence]
      Graphify[Graphify]
      UAnything[Understand Anything]
    end

    subgraph VerificationTools[Verification / analysis]
      TS[TypeScript / ESLint / ts-morph]
      SG[Semgrep]
      CQ[CodeQL]
      Mod[modularity]
    end

    subgraph AssuranceTools[Architecture / policy]
      TC[TrueCourse]
      SP[Superpowers]
    end

    subgraph ExecutionTools[Agent execution / orchestration]
      CC[Claude Code]
      OH[OpenHands]
      Aider[Aider]
      LR[Langroid]
      Cond[Conductor-style systems]
    end

    subgraph EvalTools[Evaluation / observability]
      LS[LangSmith / Langfuse-style]
      PF[Promptfoo]
      TL[TransformerLab]
      AB[Aider / OpenHands benchmarks]
    end

    OKF --> K
    ClaudeMD --> K
    Wiki --> K
    Mem --> L
    GR --> C
    CAS --> C
    Xirp --> C
    Graphify --> E
    UAnything --> E
    TS --> E
    SG --> E
    CQ --> E
    Mod --> E
    TC --> P
    SP --> P
    CC --> X
    OH --> X
    Aider --> X
    LR --> X
    Cond --> X
    LS --> O
    LS --> V
    PF --> V
    TL --> V
    AB --> V
```

### How to read the map

The arrows are **architectural fit**, not dependency claims. A tool can touch several areas. For example, a context engine can consume knowledge and structural evidence while remaining an execution-side integration rather than becoming EOKS itself.

The most important distinction is:

```text
EOKS layer       → what the layer needs
Tool             → one possible provider
EOKS             → decides which provider(s) to use
```

---

## 2. Current shortlist by EOKS layer

The table below is intentionally selective. It is the **experimentation shortlist**, not a list of every project mentioned in the research corpus.

Legend:

- **★** = particularly promising to experiment with now
- **○** = useful comparison / secondary candidate
- **△** = interesting prior art but not an immediate experiment
- **?** = insufficient evidence; research before relying on it

| EOKS area | ★ Most promising now | ○ Compare / complement | What we currently know | Main unknown to test |
|---|---|---|---|---|
| **Context compilation** | **GrapeRoot**, **CodeSight** | Xirp / Spotify | GrapeRoot is strong prior art for proactive repository context around an existing coding agent; CodeSight focuses on code understanding/context generation. | Does structured/proactive context improve outcomes over the agent's native retrieval? |
| **Knowledge representation** | **OKF** | `CLAUDE.md`, Tencent Wiki | OKF is a portable Markdown/YAML representation; `CLAUDE.md` is simple, human-reviewed local knowledge; Tencent's Wiki is a broader resource family. | Does a portable structured representation provide enough value to justify another format? |
| **Memory / learning** | **Hindsight / LangMem-style**, **Mem0/Zep-style** | Tencent Agent Memory | Strong prior art for persistent semantic/episodic/procedural memory; Tencent additionally connects memory, Skills, Wiki and CodeGraph. | Does persistent learned memory improve repeated engineering work without accumulating harmful/stale knowledge? |
| **Repository structure** | **Graphify**, **Understand Anything** | CodeSight | Both provide graph/relationship-oriented code understanding; useful for navigation and impact analysis. | When does a graph materially outperform search/retrieval or static analyzers? |
| **Lightweight verification** | **Semgrep**, **TypeScript/ESLint/ts-morph** | modularity | Fast, deterministic evidence for patterns, types and targeted project rules. | Can these satisfy most questions before deeper analysis is necessary? |
| **Deep verification / dataflow** | **CodeQL** | Semgrep | Strong candidate for interprocedural/dataflow/security questions. | Is the additional setup/runtime justified by materially stronger evidence? |
| **Architecture assurance** | **TrueCourse**, **modularity** | Superpowers | Different approaches to architecture constraints/analysis; useful for testing preventive vs detective assurance. | Which architectural invariants can be enforced mechanically and where should they live? |
| **Workflow / execution** | **Claude Code**, **OpenHands** | Aider | Existing coding-agent runtimes provide realistic execution substrates for EOKS experiments. | Does EOKS coordination improve a strong single-agent baseline? |
| **Orchestration** | **Conductor-style systems**, **Langroid** | OpenHands-style workflows | Prior art for decomposition and multi-agent execution. | When does orchestration beat a single agent plus verification? |
| **Evaluation** | **Promptfoo**, **Aider/OpenHands benchmarks** | TransformerLab | Useful for repeatable task/model/configuration evaluation and end-to-end coding benchmarks. | Can evaluations attribute gains to context/tool/workflow changes rather than just detect them? |
| **Observability** | **LangSmith/Langfuse-style** | execution traces from agents | Strong infrastructure for traces, experiments and operational evidence. | Which trace signals actually predict useful EOKS interventions? |

### What I would actually play with

If the goal is to avoid benchmarking twenty tools, I would start with roughly this set:

```text
CONTEXT
  GrapeRoot
  CodeSight

KNOWLEDGE
  OKF
  CLAUDE.md

MEMORY / LEARNING
  Hindsight/LangMem-style
  Mem0/Zep-style

STRUCTURE
  Graphify
  Understand Anything

VERIFICATION
  Semgrep
  CodeQL
  TypeScript/ESLint/ts-morph

ASSURANCE
  TrueCourse
  modularity

EXECUTION
  Claude Code
  OpenHands

EVALUATION
  Promptfoo
  Aider/OpenHands benchmark infrastructure
```

This is **not** a claim that these are the objectively best products. They are the most useful *current experiments* because they cover distinct hypotheses with relatively little redundancy.

---

## 3. Tool families and their relative position

A second useful view is to compare tools by the **kind of evidence/resource they provide**, rather than by vendor/project.

```text
                         SEMANTIC / HUMAN INTERPRETATION
                                      ↑
                                      |
                     LLM / agent reasoning
                                      |
              CodeSight / Understand Anything
                                      |
              Graphify — structural relationships
                                      |
        Semgrep — patterns / targeted dataflow
                                      |
       CodeQL — deep interprocedural/dataflow
                                      |
 TypeScript / ESLint — types / local deterministic facts
                                      |
                                      ↓
                         MECHANICAL / DETERMINISTIC

     local/file  ←──────── scope ────────→ repository/system

     fast/cheap  ←──── operational cost ──→ slow/expensive
```

This is intentionally conceptual rather than numeric. A provider can move along several axes depending on configuration and question.

### The evidence ladder

For software-engineering questions, a useful **starting hypothesis** is:

```text
                 Is existing evidence sufficient?
                            |
                         no | yes → continue
                            ↓
             Type/compiler/language tooling
                            |
                         no |
                            ↓
                  lightweight rules
                    (Semgrep etc.)
                            |
                         no |
                            ↓
              structural / graph evidence
                            |
                         no |
                            ↓
             deep static/dataflow analysis
                         (CodeQL)
                            |
                         no |
                            ↓
                tests / runtime evidence
                            |
                         no |
                            ↓
                independent review / LLM
```

This is **not a universal ordering**. The Evidence Requirement determines the appropriate path. In some questions, runtime evidence should come first; in others, a graph is merely supporting evidence.

---

## 4. Relationship map: overlap vs complementarity

Rather than making a giant pairwise table, this graph identifies the relationships most useful for choosing experiments.

```mermaid
flowchart TD
    subgraph Context[Context]
      GR[GrapeRoot]
      CS[CodeSight]
      XA[Understand Anything]
    end

    subgraph Structure[Structure]
      GF[Graphify]
      XA
    end

    subgraph Verify[Verification]
      TS[TypeScript / ESLint]
      SG[Semgrep]
      CQ[CodeQL]
    end

    subgraph Knowledge[Knowledge]
      OKF[OKF]
      CMD[CLAUDE.md]
      TM[Tencent Agent Memory]
    end

    GR -. overlaps .- CS
    CS -. consumes .- GF
    GF -. complements .- SG
    SG -. escalates to .- CQ
    TS -. cheaper alternative for type questions .- SG
    SG -. cheaper alternative for simple patterns .- CQ
    OKF -. representation alternative/complement .- CMD
    TM -. broader resource system .- OKF
    GF -. structural evidence .- CQ
    XA -. structural/semantic overlap .- CS
```

Relationship semantics come from the canonical capability model: **overlap, complement, alternative, escalation, specialization and dependency**. The graph should remain small and curated; a complete pairwise graph would become unreadable and stale.

---

## 5. What information do we actually have today?

The current evidence is uneven. This is important: the landscape should expose uncertainty rather than make every tool look equally understood.

| Information | Current state | Useful for choosing experiments? |
|---|---|---|
| Primary capability | **Good** for most tools | Yes |
| EOKS layer / architectural fit | **Good** | Yes |
| Strengths / weaknesses | **Moderate** | Yes, especially for shortlist |
| Overlap / complementarity | **Moderate** | Yes |
| Evidence kind | **Moderate/good** for analysis tools | Yes |
| Precision / recall | **Weak / mostly unmeasured by EOKS** | Not yet for hard ranking |
| Cost / latency | **Weak / environment-dependent** | Need local measurements |
| Setup / integration effort | **Moderate** | Yes |
| Provenance / explainability | **Moderate** | Yes |
| Real EOKS workload outcomes | **Very weak** | This is the main gap |
| Cross-tool causal comparison | **Very weak** | Major research opportunity |

Therefore the landscape should currently answer:

> **What looks promising and why?**

not:

> **Which tool is objectively best?**

---

## 6. Recommended experimentation strategy

The efficient strategy is **one baseline + 2–3 providers per hypothesis**, not a benchmark of every product.

### Context

Compare:

```text
native agent context
       vs
GrapeRoot
       vs
CodeSight
```

Hold the agent/model constant.

### Knowledge

Compare:

```text
CLAUDE.md / ordinary project docs
       vs
OKF bundle
```

Then ask whether the richer representation changes context selection or outcome quality.

### Structure

Compare:

```text
repository search/native tooling
       vs
Graphify
       vs
Understand Anything
```

Focus on impact analysis, navigation and relationship questions.

### Verification

This is the clearest candidate for an evidence ladder experiment:

```text
TypeScript / ESLint
       → Semgrep
       → CodeQL
       → tests/runtime evidence
```

Measure whether escalation actually improves correctness enough to justify its cost.

### Execution / orchestration

Compare:

```text
strong single coding agent
       vs
agent + verification
       vs
orchestrated executor/reviewer
```

Don't assume more agents are better.

### Memory

Compare:

```text
no persistent memory
       vs
episodic/semantic memory
       vs
procedural/learned memory
```

The critical metric is not just retrieval quality; it is **future task outcome after memory has been allowed to influence behavior**.

---

## 7. Gaps that should drive research

The landscape reveals several gaps that the capability model alone cannot fill.

### A. We need a real tool registry

The information is currently Markdown. A future machine-readable registry should encode at least:

```yaml
tool:
category:
capabilities:
evidence_kinds:
scope:
depth:
strengths:
weaknesses:
best_fit:
poor_fit:
relationships:
operational:
evidence_status:
```

The Markdown landscape and comparison matrices could then be generated from it.

### B. We need confidence on the *tool facts themselves*

There is an important second-order problem: EOKS currently records claims about tools without always recording how those claims were established.

Eventually each important capability should have something like:

```text
claim
  ↓
source / version
  ↓
observation type
  ↓
confidence
  ↓
last verified
```

This prevents the tool-selection layer from treating old research notes as timeless facts.

### C. We need capability coverage, not just categories

“Static analysis” is too broad. The useful question is:

```text
Can this provider establish:
  local pattern?
  type property?
  dependency relationship?
  architectural rule?
  interprocedural flow?
  runtime behavior?
  semantic intent?
```

This is where the structured capability model should evolve through experiments.

### D. We need to test **minimum sufficient evidence**

This is arguably the most EOKS-specific experiment:

> Can we avoid expensive tools most of the time while retaining the correctness of a stronger evidence stack?

If yes, evidence-aware control may be a genuinely useful contribution of EOKS.

### E. We need causal experiments

If GrapeRoot, Graphify or CodeQL appears to improve an agent, we need to distinguish:

```text
better context
better evidence
better workflow
better model
more tokens
more retries
```

from one another. Otherwise the landscape remains a collection of plausible tool descriptions rather than an empirical basis for control.

---

## 8. Status of this map

**Current status: research snapshot.**

The stars are prioritization recommendations for experimentation, not benchmark results. They should change as EOKS tests tools on real workloads.

The intended evolution is:

```text
Today
  tool notes + capability profiles
          ↓
Next
  structured tool registry + landscape
          ↓
Then
  measured capability / operational facts
          ↓
Then
  workload-specific evidence outcomes
          ↓
Eventually
  EOKS selection policy learns which provider to use
```

The purpose of this document is to make the **current map of the territory** visible while the formal capability/selection model remains the source of truth for future automated selection.
