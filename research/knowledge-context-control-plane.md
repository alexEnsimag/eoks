# Knowledge, context and the EOKS control plane

This note consolidates a recent line of investigation around OKF, Graphify, GrapeRoot and the broader EOKS architecture. The central conclusion is that these projects are not best understood as competing implementations of one thing. They occupy different layers of an emerging AI engineering stack.

The working EOKS hypothesis is:

> **EOKS should be a control/data model for AI workloads rather than another agent framework.**
>
> It should coordinate tasks, context, execution, policies, evidence and outcomes while consuming specialized knowledge and execution systems underneath.

The terminology remains provisional. This document is an architectural hypothesis, not a claim that the industry has standardized these boundaries.

## 1. The layered model

A useful decomposition is:

```text
                         EOKS CONTROL PLANE
        task · scheduling · policies · resource selection
                         · evaluation · feedback
                                  |
          +-----------------------+-----------------------+
          |                       |                       |
   KNOWLEDGE PLANE         CONTEXT PLANE          EXECUTION PLANE
   durable knowledge       selection/assembly      workflows/runs
   representations         provenance/budgets      models/tools/agents
          |                       |                       |
          +-----------------------+-----------------------+
                                  |
                         EVALUATION / OUTCOMES
                                  |
                              FEEDBACK
```

The planes are deliberately not products. They are responsibilities.

### Knowledge

Knowledge is what the system can preserve and reuse. It can have several representations:

- human-reviewed Markdown and ADRs;
- OKF bundles;
- structural code graphs;
- semantic indexes;
- timelines and decision records;
- runtime evidence;
- episodic and procedural memory.

A graph is therefore **a representation of knowledge, not knowledge itself**.

### Context

Context is what is intentionally made available to a particular reasoning step. It is compiled from knowledge and evidence for a task; it is not the knowledge store.

### Execution

Execution is where models, agents, tools and workflows act on the task and produce artifacts and observations.

### Control

The control plane chooses what should happen next and under what policy: which evidence providers to use, how much context to construct, which model or strategy to run, whether verification is required, and whether to stop, retry, branch or escalate.

---

## 2. OKF: a knowledge representation, not the EOKS runtime

OKF is particularly relevant because it addresses a part of the problem that EOKS should not reinvent casually: a portable, human- and agent-friendly representation of durable knowledge.

The recent OKF work is especially relevant to EOKS because the format/tooling direction includes concepts such as:

- structured knowledge files rather than opaque prompt state;
- typed frontmatter and relationships;
- provenance and source attribution;
- generation metadata;
- verification/trust state;
- lifecycle/freshness information;
- validation/linting and graph views.

The exact OKF schema/version should be tracked against the upstream specification rather than copied into EOKS. The important architectural boundary is:

```text
OKF / other knowledge representations
              |
              v
        durable knowledge
              |
              v
      EOKS context compiler
              |
              v
            model
```

EOKS should be able to consume OKF without making OKF mandatory. Likewise, EOKS should not require every useful engineering fact to be expressed in OKF if a more authoritative or cheaper representation exists.

### Trust is evidence-oriented

A useful distinction is:

```text
model confidence
    != evidence strength
    != context quality
    != outcome quality
```

For example, a knowledge item can carry evidence about its source, revision, freshness and verification status. EOKS can then use those signals when deciding whether the item is sufficient for a task.

Do not collapse these signals into one universal confidence number prematurely. Keep the dimensions visible and let policies determine which assurance is required.

---

## 3. GrapeRoot: context/execution infrastructure

GrapeRoot is a particularly useful reference point because it demonstrates a practical architecture around an existing coding agent rather than replacing the agent.

Its public documentation describes the following flow:

```text
project
  |
  v
local graph scan
  |
  v
relevant files/symbols
  |
  v
context packer
  |
  v
pre-injected context
  |
  v
Claude Code / other agent
```

The current public architecture also retains MCP graph tools for deeper exploration. The important design is therefore hybrid: proactively provide likely-relevant context, while leaving an agent an escape hatch for deeper queries.

GrapeRoot's launcher is also architecturally interesting. A command such as `dgc`/`graperoot` prepares the local environment, starts the local graph/MCP service, installs or configures agent hooks, and launches the existing coding agent. This makes the tool a **sidecar/control layer around the agent**, rather than a replacement model runtime.

The public repository exposes launcher/tooling code, while the core `graperoot` graph engine is distributed as a proprietary compiled package. Therefore EOKS should treat the observed architecture as prior art, not assume knowledge of the proprietary ranking implementation.

### GrapeRoot's important architectural lesson

The strongest lesson is not simply "use a graph". It is:

> **A control layer can prepare and observe an existing agent without owning the agent's reasoning loop.**

That allows a conservative EOKS integration model:

```text
                  EOKS
              /          \
      prepare/context     observe/evaluate
            |                    |
            v                    |
        Claude Code ------------+
```

This is materially different from building another monolithic agent framework.

### GrapeRoot versus EOKS

GrapeRoot primarily addresses:

- structural repository understanding;
- task-context retrieval and packing;
- token/read budgets;
- session-aware context weighting;
- local MCP exploration;
- agent launch/integration.

EOKS would additionally need to coordinate:

- task and assurance requirements;
- multiple knowledge/evidence providers;
- model and reasoning-strategy selection;
- policies and stop conditions;
- evaluation and deterministic verification;
- outcomes and delayed feedback;
- promotion/invalidation of durable knowledge.

Therefore GrapeRoot is better viewed as a **context/execution resource that an EOKS-like control plane could orchestrate**, not as an EOKS replacement.

---

## 4. Graphify: structural evidence, not canonical truth

Graphify belongs primarily in the knowledge/evidence plane.

A structural graph can answer questions such as:

- what imports what;
- what calls what;
- where a symbol is defined;
- which files are connected;
- what neighborhood or impact slice is relevant.

That is extremely useful for context compilation. But structural connectivity does not automatically prove a semantic invariant.

For example:

```text
source -> transformation -> persistence sink
```

is not equivalent to proving:

```text
source value can bypass the required validation/barrier
```

The latter may require a specialized dataflow or invariant analyzer.

This supports an EOKS principle:

> **Use the cheapest reliable evidence provider that can answer the question.**

Possible providers include a compiler/type checker, lightweight lint rule, AST analysis, Graphify-like structural graph, Semgrep, CodeQL, tests, runtime observations and human review. The control plane should select providers based on task requirements rather than always running the deepest analyzer.

---

## 5. Context is a compiled artifact

The previous discussions repeatedly used YAML examples for things like context blocks, provenance and trust. YAML is useful because it makes structure visible, but it is **not the proposed EOKS canonical representation**.

The underlying abstraction is a typed object/graph; YAML is merely one possible serialization or debugging view.

A context block might conceptually contain:

```text
ContextBlock
  content
  source/provenance
  revision
  freshness
  relevance
  evidence
  relationships
  cost
```

The system should be able to answer:

1. What was included?
2. What was omitted?
3. Why was each item selected?
4. Which evidence supports it?
5. What revision was it derived from?
6. How much did it cost?
7. Did the resulting task succeed?

This makes context an inspectable, budgeted artifact rather than an opaque concatenated prompt.

### Context compiler

The intended flow is:

```text
Task
  |
  +-- constraints / assurance policy
  |
  +-- available knowledge
  |
  +-- evidence providers
  |
  v
Context compiler
  |
  +-- retrieve
  +-- rank
  +-- deduplicate
  +-- resolve/flag conflicts
  +-- compress
  +-- order/structure
  +-- enforce budget
  |
  v
Compiled context
  |
  v
Model / reasoning strategy
```

This also clarifies why context engineering and knowledge management should remain separate. Knowledge can be durable and broad; context is task-specific and ephemeral.

---

## 6. A minimal EOKS semantic model

The current recommendation is to resist inventing a large ontology. Seven runtime primitives are enough to start:

| Primitive | Meaning |
|---|---|
| **Task** | Bounded work with an objective, constraints and required assurance |
| **Context** | Evidence/instructions intentionally supplied to a reasoning step |
| **Run** | One attempt to execute a task or subtask |
| **Decision** | A control-plane choice about what happens next |
| **Policy** | Constraints/requirements governing decisions |
| **Evaluation** | Measurement of intermediate or final quality/assurance |
| **Outcome** | What actually happened, including artifacts and delayed results |

Resources such as knowledge stores, models, tools, agents and analyzers should initially be treated as **resources/providers**, not as a sprawling ontology of first-class EOKS objects.

A useful relationship model is:

```text
Task
  |- requires -> Policy
  |- produces -> Context
  |- has -> Run
  `- ends-in -> Outcome

Context
  |- derived-from -> Evidence/Knowledge
  |- relevant-to -> Task
  `- selected-by -> Decision

Run
  |- uses -> Context
  |- invokes -> Model/Tool/Agent
  |- produces -> Decision/Artifact
  `- evaluated-by -> Evaluation

Evaluation
  |- measures -> Context / Decision / Run / Outcome
  `- feeds -> Policy / future Context selection
```

The exact graph/event schema should remain experimental until a real implementation demonstrates which relationships are needed.

---

## 7. Why `Run` matters more than `Agent` as a core primitive

A user-facing agent may contain many loops and subagents. For observability and evaluation, the more useful atomic unit is often a **Run**: one attempt to achieve a task under a particular context, policy and resource configuration.

```text
Task #123
  |
  +-- Run #1 -> failed verification
  |
  `-- Run #2 -> passed tests -> accepted
```

This makes it possible to compare context, model, tool and policy choices between attempts without forcing EOKS to own the internal implementation of every agent framework.

Runs can be nested when a workflow creates subtasks.

---

## 8. Confidence, trust and evaluation

EOKS should not make model self-reported confidence the authority for stopping.

A more useful evaluation vector can include:

```text
knowledge/evidence provenance
knowledge freshness
context relevance
context coverage
contradiction risk
model uncertainty
verification results
tool success
historical outcome quality
```

For example, a policy might require:

```text
high-assurance task
  -> authoritative evidence required
  -> verification required
  -> context coverage above a threshold
  -> human escalation if verification fails
```

The exact numeric thresholds are workload-specific and should be calibrated against actual outcomes before they are allowed to drive automatic model switching, retries or stopping.

This is where observability becomes a **sensor layer for the control loop**, rather than merely a dashboard.

---

## 9. The control loop

The resulting loop is:

```text
                  +-------------------+
                  |       Task        |
                  +---------+---------+
                            |
                            v
                    context compilation
                            |
                            v
                           Run
                            |
                            v
                         Decision
                            |
                            v
                      model / tools
                            |
                            v
                         Outcome
                            |
                            v
                        Evaluation
                            |
              +-------------+-------------+
              |                           |
              v                           v
       continue / stop             update policy /
       retry / branch              context / memory
```

The stop condition should ideally be based on **sufficient trustworthy evidence for the task**, not merely on the model saying it is confident.

---

## 10. What EOKS should and should not become

### EOKS should be

- a control/data model for AI workloads;
- a context compiler and resource-selection layer;
- an evidence-aware execution/evaluation loop;
- an interoperability layer across knowledge and agent systems;
- an observability/control surface for why information and execution decisions were made.

### EOKS should not initially be

- another vector database;
- another RAG framework;
- another code graph implementation;
- another coding agent;
- a replacement for Claude Code/Codex/etc.;
- a mandatory canonical knowledge format;
- a giant ontology defined before experiments validate it.

The system should be able to use these components underneath it.

---

## 11. A practical architecture

The resulting architecture is intentionally compositional:

```text
                         EOKS CONTROL PLANE

  Task -> Policy -> Context compiler -> Resource selection
                       |                     |
             +---------+----------+          +----------------+
             |                    |                           |
          knowledge            evidence                    execution
             |                    |                           |
        +----+----+       +------+------+              +-----+------+
        |         |       |             |              |            |
       OKF     project   Graphify    analyzers       Claude      tools/tests
               docs      / graphs    / tests         / agents
        |         |       |             |              |            |
        +---------+-------+-------------+--------------+------------+
                                  |
                                 Run
                                  |
                             Evaluation
                                  |
                              Outcome
                                  |
                               Feedback
```

GrapeRoot-like context engines, OKF, Graphify, TrueCourse-like assurance tools, Xirp-like organizational context, deterministic analyzers and coding agents can therefore coexist as specialized resources.

---

## 12. Product/integration strategy

There are three plausible implementation strategies:

### A. Minimal runtime

Implement Task → Context → Run → Evaluation → Outcome, with Policy and Decision as control mechanisms.

**Pros:** small, testable, low competition with existing agent frameworks.

**Cons:** less ambitious initially; many integrations remain external.

### B. Full control plane

Implement scheduling, context compilation, model routing, policy, execution graphs, evaluation, telemetry and feedback as one system.

**Pros:** strongest end-to-end control-plane story.

**Cons:** large scope and high risk of rebuilding existing agent infrastructure.

### C. Protocol/semantic model

Define the objects/events and provide adapters for existing coding agents, context engines and knowledge systems.

**Pros:** interoperability and a clearer long-term "AI control plane" position.

**Cons:** adoption is harder without a compelling reference implementation.

### Recommended direction

Combine **A + C**:

1. define the smallest useful semantic/event model;
2. build a reference runtime around it;
3. integrate with existing systems rather than replacing them;
4. use real runs to discover which primitives are actually necessary;
5. expand toward a broader control plane only where measurements justify it.

---

## 13. The key experiment

The first serious prototype should answer five questions for a coding task:

1. **What did the agent know?**
2. **Why was that context selected?**
3. **What did the agent decide?**
4. **What evidence supported the decision?**
5. **Did it actually work?**

A minimal trace could therefore be:

```text
Task
  -> Context + selection rationale
  -> Run + resources
  -> Decisions
  -> Evidence / tool calls
  -> Outcome
  -> Evaluation
```

If EOKS can reconstruct and evaluate this reliably, it has demonstrated a useful control-plane primitive without first building a complete agent platform.

---

## 14. Open questions

- What is the minimum event schema needed to reconstruct a run?
- Which context-quality signals correlate with real task outcomes?
- How should conflicting evidence be represented and resolved?
- Which trust/provenance fields should be delegated to OKF rather than duplicated?
- Can a context compiler expose a stable provider interface across Graphify-like graphs, RAG, tests and organizational systems?
- How much of an agent's internal reasoning must EOKS observe to make useful control decisions?
- Can existing coding-agent CLIs be wrapped non-invasively through launchers, hooks and MCP, as GrapeRoot demonstrates?
- When should EOKS intervene: before a run, during a run, after verification, or only between runs?
- How should delayed outcomes update trust and future context selection?
- Which parts of the model should become a protocol versus remain implementation-specific?

The guiding constraint is to **measure before standardizing**. EOKS should earn each abstraction through a concrete workload.
