# EOKS tool landscape

This is the **current research map of tools and mechanisms relevant to EOKS**. It is broader than a product shortlist: a project may belong to several categories because many systems combine representation, retrieval, context compilation, execution, memory, evaluation or governance.

This is a research snapshot, not a benchmark. Ratings reflect current public documentation, repository inspection, EOKS research notes, and recent ecosystem evidence through October 2026. They are not measured EOKS scores unless explicitly stated.

For the formal selection model, see [Tool capability model](tool-capability-model.md) and [Tool selection](tool-selection.md). For detailed source notes, see [`research/tool-notes.md`](../research/tool-notes.md).

## 1. How to read the comparison

### Categories overlap by design

The old map forced each tool into one EOKS layer. That is misleading. Examples:

- **GrapeRoot** = repository graph + proactive context compilation + runtime/agent integration.
- **Understand Anything** = deterministic parsing + LLM semantic analysis + knowledge graph + impact exploration.
- **Xirp / Spotify** = system/organizational context + session continuity + context assembly + execution integration.
- **Hindsight** = persistent memory + retrieval + reflection + evolving beliefs.
- **Opik** = tracing + observability + evaluation + experiments + optimization.
- **TencentDB Agent Memory** = memory + Skills + Wiki + CodeGraph + governance/loadouts.
- **Spotify Portal/Shunt** = harness-level transformation/caching/I/O optimization rather than another knowledge database.

A tool therefore has a **primary role** and **secondary capabilities** rather than one permanent category.

### Stars are evidence/status signals, not quality scores

| Signal | Meaning |
|---|---|
| **Popularity ★★★★★** | Breadth of ecosystem adoption, community visibility, production use and/or project traction. Qualitative; not a raw GitHub-star count. |
| **Maturity ★★★★★** | How established and operationally stable the project/mechanism appears. |
| **Evidence ★★★★★** | Strength of public evidence: production use, measured results, evaluations, independent discussion, or mature implementation. |
| **EOKS fit ★★★★★** | Relevance to an EOKS hypothesis. Architectural assessment, not product quality. |
| **Experiment priority ★★★★★** | Expected research value now, considering distinctiveness, integration cost and information gained. |

High EOKS fit with low evidence means **interesting research**, not something EOKS should depend on blindly. High popularity/maturity can still be low experiment priority if a tool adds little information beyond an existing mechanism.

---

## 2. Current capability map

```text
                              EOKS control / selection
                                       │
             ┌─────────────────────────┼─────────────────────────┐
             │                         │                         │
       KNOWLEDGE / RESOURCES      CONTEXT ENGINEERING       EXECUTION
             │                         │                         │
       representation             acquire / retrieve       agent runtime
       canonical docs             transform / compile      orchestration
       memory / procedures        cache / compress         tool use
       session artifacts          deliver / budget         policies
             │                         │                         │
             └──────────────┬──────────┴──────────┬──────────────┘
                            │                     │
                     EVIDENCE / ANALYSIS      EVALUATION
                            │                     │
                  structural / semantic       traces
                  static / runtime            task outcomes
                  architecture / tests       trajectory quality
                            │                     │
                            └──────────┬──────────┘
                                       │
                              LEARNING / LIFECYCLE
                                       │
                       promotion / invalidation / evolution
                       reflection / memory / feedback
```

The important EOKS question is no longer simply **“which tool provides context?”**. It is:

> **Which mechanism provides the minimum sufficient evidence/resource/context transformation for this task, at acceptable cost, with enough provenance and lifecycle control to be trusted?**

---

## 3. Master comparison matrix

### Execution substrates and agent workflow runtimes

The Execution tile is broader than workflow. A Claude Code session, deterministic process, workflow/graph runtime, remote agent or fleet can all be an execution. Workflow runtimes are therefore one class of execution substrate.

They should be compared on the execution semantics they expose: **topology, state, checkpoint/interrupt/resume, recovery, human participation, placement, observability and portability**. EOKS should not assume that one runtime is the canonical implementation, or that a workflow is required for every Work.

### Quick mental models

These analogies are intentionally concise: they describe the role of each tool relative to EOKS, not a claim that the tools are technically equivalent.

| Tool | Mental model | EOKS role |
|---|---|---|
| **LangGraph** | **Kubernetes controller + state machine for an agent Run** | explicit graph/stateful execution |
| **CrewAI** | **n8n for agent teams** | crews + event-driven Flows |
| **Microsoft Agent Framework** | **Step Functions + agent runtime** | durable workflow execution + HITL/operations |
| **Google ADK** | **Kubernetes ecosystem for agents** | agent build/eval/deploy/observe/runtime |
| **OpenAI Agents SDK** | **Go standard library for agents** | small composable agent runtime |
| **AutoGen** | **actor/message-passing model for agents** | conversational multi-agent execution |

The analogy is useful for orientation; the capability matrix and source notes remain authoritative.

| Tool / mechanism | Primary role | Also provides | Popularity | Maturity | Evidence | EOKS fit | Experiment priority |
|---|---|---|---:|---:|---:|---:|---:|
| **Claude Code** | coding-agent execution substrate | hooks, MCP/tools, project instructions, session/context management | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ |
| **OpenHands** | open coding-agent runtime | tools, workflows, benchmarks, multi-agent research | ★★★★☆ | ★★★★☆ | ★★★★☆ | ★★★★☆ | ★★★★☆ |
| **Aider** | terminal coding agent | repo map, benchmarks, edit strategies | ★★★★★ | ★★★★★ | ★★★★☆ | ★★★★☆ | ★★★★☆ |
| **GrapeRoot** | proactive repository context compilation | semantic code graph, session-aware retrieval, MCP, agent launcher | ★★☆☆☆ | ★★★☆☆ | ★★★☆☆ | ★★★★★ | ★★★★★ |
| **Graphify** | structural code graph / navigation | graph report, JSON representation, MCP, relationship queries | ★★☆☆☆ | ★★★☆☆ | ★★★☆☆ | ★★★★★ | ★★★★★ |
| **Understand Anything** | code/domain knowledge graph | parsing, LLM semantic analysis, tours, impact/change analysis | ★★☆☆☆ | ★★★☆☆ | ★★★☆☆ | ★★★★★ | ★★★★☆ |
| **Codebase Memory MCP** | persistent repository graph/context | MCP queries, structural repository memory | ★★☆☆☆ | ★★★☆☆ | ★★☆☆☆ | ★★★★☆ | ★★★★☆ |
| **CodeSight** | repository understanding/context | structured codebase context/navigation | ★★☆☆☆ | ★★★☆☆ | ★★☆☆☆ | ★★★★☆ | ★★★☆☆ |
| **Spotify Portal / Shunt** | harness-level context/I/O transformation | caching, transformation, deduplication, tool mediation | ★★★★☆ | ★★★★☆ | ★★★★★ | ★★★★★ | ★★★★★ |
| **Xirp / Spotify** | system/organizational context | ownership, dependencies, docs, architecture decisions, session continuity | ★★★☆☆ | ★★★☆☆ | ★★★★☆ | ★★★★★ | ★★★★☆ |
| **OKF** | portable durable knowledge representation | Markdown/YAML structure, provenance/lifecycle concepts | ★☆☆☆☆ | ★★☆☆☆ | ★★☆☆☆ | ★★★★★ | ★★★★☆ |
| **CLAUDE.md / project instructions** | canonical local policy/knowledge | scope hierarchy, human review, Git-native lifecycle | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ |
| **TencentDB Agent Memory** | reusable resource/memory system | multi-resolution memory, Skills, Wiki, CodeGraph, loadouts/governance | ★★★☆☆ | ★★★☆☆ | ★★★★☆ | ★★★★☆ | ★★★★☆ |
| **Hindsight** | evolving agent memory | semantic/episodic/procedural memory, reflection, belief evolution | ★★☆☆☆ | ★★★☆☆ | ★★★☆☆ | ★★★★★ | ★★★★★ |
| **MemoryLACE** | relational/lifecycle memory | relationships between memories and evolving context | ★☆☆☆☆ | ★★☆☆☆ | ★★☆☆☆ | ★★★★★ | ★★★★☆ |
| **LCM / Lossless Context Management** | lossless context lifecycle | structured compaction, recoverability, lineage-oriented handling | ★★☆☆☆ | ★★☆☆☆ | ★★★☆☆ | ★★★★★ | ★★★★★ |
| **OpenWolf** | interaction-derived repository memory | hooks, summaries, persistent repository notes | ★★☆☆☆ | ★★☆☆☆ | ★★☆☆☆ | ★★★★☆ | ★★★★☆ |
| **memsearch / LangMem / Mem0 / Zep family** | persistent memory patterns | semantic/episodic/procedural retrieval and storage integrations | ★★★★☆ | ★★★★☆ | ★★★★☆ | ★★★★☆ | ★★★★☆ |
| **QMD** | local retrieval/indexing | lexical/semantic retrieval and context-oriented search | ★★☆☆☆ | ★★★☆☆ | ★★★☆☆ | ★★★★☆ | ★★★★☆ |
| **ClawMem / OpenClaw memory patterns** | session/persistent memory | hooks, memory files, retrieval and lifecycle experiments | ★★☆☆☆ | ★★☆☆☆ | ★★☆☆☆ | ★★★★☆ | ★★★☆☆ |
| **Obsidian** | human research/thinking workspace | linked notes, graph/navigation, plugins | ★★★★★ | ★★★★★ | ★★★★☆ | ★★★★☆ | ★★★☆☆ |
| **Semgrep** | deterministic code/pattern/dataflow evidence | taint/dataflow, policy rules, CI integration | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ |
| **CodeQL** | deep semantic/dataflow analysis | interprocedural queries, security analysis, path explanations | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★☆ |
| **TypeScript compiler** | deterministic type/invariant evidence | type checking, compiler facts | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ |
| **ESLint** | lightweight deterministic policy | AST rules, custom project rules | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★★ |
| **ts-morph / compiler APIs** | targeted custom static analysis | symbols, types, AST and project-specific queries | ★★★★☆ | ★★★★☆ | ★★★★☆ | ★★★★☆ | ★★★★☆ |
| **TrueCourse** | architecture/specification assurance | deterministic rules, LLM review, executable behavior guards | ★☆☆☆☆ | ★★☆☆☆ | ★★★☆☆ | ★★★★★ | ★★★★★ |
| **modularity** | architecture analysis | dependency/architecture evidence and constraints | ★★☆☆☆ | ★★★☆☆ | ★★★☆☆ | ★★★★☆ | ★★★★☆ |
| **Superpowers** | structured development workflow | planning, review/quality gates, execution discipline | ★★★☆☆ | ★★★☆☆ | ★★★☆☆ | ★★★★☆ | ★★★★☆ |
| **Opik** | agent observability/evaluation | traces, datasets, experiments, judges, optimization, monitoring | ★★★★☆ | ★★★★☆ | ★★★★★ | ★★★★★ | ★★★★★ |
| **LangSmith / Langfuse-style systems** | tracing/evaluation infrastructure | traces, experiments, datasets, observability | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★☆ | ★★★★☆ |
| **Promptfoo** | repeatable eval/experiment harness | prompt/model comparison, assertions, red-team/eval workflows | ★★★★☆ | ★★★★☆ | ★★★★★ | ★★★★☆ | ★★★★☆ |
| **TransformerLab** | model experimentation/evaluation | local experimentation and model/eval workflows | ★★☆☆☆ | ★★★☆☆ | ★★★☆☆ | ★★★☆☆ | ★★★☆☆ |
| **Aider benchmarks** | coding-agent outcome benchmark | repeatable software-engineering tasks | ★★★★☆ | ★★★★☆ | ★★★★★ | ★★★★☆ | ★★★★☆ |
| **OpenHands benchmarks** | coding-agent benchmark infrastructure | SWE tasks and agent evaluation | ★★★★☆ | ★★★★☆ | ★★★★★ | ★★★★☆ | ★★★★☆ |
| **OpenAI Evals-style frameworks** | reusable evaluation harness | private/workload-specific evals | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★☆ | ★★★★☆ |
| **Conductor-style systems** | task decomposition/orchestration | multi-agent/task topology, coordination | ★★★☆☆ | ★★★☆☆ | ★★★☆☆ | ★★★★☆ | ★★★★☆ |
| **LangGraph** | stateful execution-graph runtime | explicit graphs, state, durable lifecycle, interrupts/resume, human gates | ★★★★★ | ★★★★☆ | ★★★★★ | ★★★★★ | ★★★★★ |
| **CrewAI** | agent-team + event-driven execution runtime | crews, Flows, state, resumability, multi-agent coordination | ★★★★☆ | ★★★★☆ | ★★★★☆ | ★★★★☆ | ★★★★☆ |
| **Microsoft Agent Framework** | agent + workflow execution runtime | durable workflows, checkpoint/resume, HITL, observability, visualization, orchestration | ★★★★☆ | ★★★★☆ | ★★★★☆ | ★★★★★ | ★★★★★ |
| **Google ADK** | agent development + execution runtime | orchestration, tools, eval, deployment, observability, runtime environments | ★★★★☆ | ★★★★☆ | ★★★★☆ | ★★★★☆ | ★★★★☆ |
| **OpenAI Agents SDK** | lightweight agent execution SDK | tools, handoffs, guardrails, sessions, HITL, tracing | ★★★★★ | ★★★★☆ | ★★★★☆ | ★★★★☆ | ★★★★☆ |
| **AutoGen** | conversational multi-agent execution (legacy prior art) | agent conversations, group collaboration | ★★★★☆ | ★★★☆☆ | ★★★★☆ | ★★★★☆ | ★★☆☆☆ |
| **Langroid** | multi-agent execution/orchestration | agent communication and task coordination | ★★★☆☆ | ★★★★☆ | ★★★☆☆ | ★★★★☆ | ★★★☆☆ |
| **Plano** | operational routing/governance | runtime operations and control mechanisms | ★★☆☆☆ | ★★☆☆☆ | ★★☆☆☆ | ★★★☆☆ | ★★★☆☆ |
| **CodeRabbit / Sourcegraph Cody** | coding/review execution prior art | review, repository context, developer workflow integration | ★★★★☆ | ★★★★☆ | ★★★★☆ | ★★★☆☆ | ★★★☆☆ |
| **ADHD-style reasoning strategies** | reusable reasoning strategy | divergence, alternatives, critique, convergence | ★★☆☆☆ | ★★☆☆☆ | ★★☆☆☆ | ★★★★☆ | ★★★★☆ |

The stars are intentionally **not a global ranking**.

### Execution isolation and sandbox-provider submatrix

The master matrix above is useful for discovery, but its qualitative stars are not enough to choose an execution environment for a workload with security or portability requirements. Use this focused comparison for the execution boundary. It complements—not replaces—the detailed [agent execution and sandbox landscape research](../research/prior-art/agent-execution-sandbox-landscape-2026.md).

| Project / approach | Layer | Isolation boundary | Policy enforcement and action visibility | Credentials | Lifecycle / portability | Evidence caveat |
|---|---|---|---|---|---|---|
| **Google ACE** | Compute substrate | Hardware-isolated microVMs | Compute isolation; workload access policy must be assessed separately | Not the defining abstraction | Agent-oriented start/pause/resume lifecycle; provider-specific | MicroVM isolation alone does not prove that a Work's requested policy was applied |
| **Google AX** | Execution runtime / orchestration | Sandboxed execution substrate; backend details matter | Task, Workspace and Gateway provide runtime-level composition and access boundaries | Depends on gateway/provider setup | Durable task/workspace state and suspend/resume | Keep AX's execution API distinct from ACE's compute substrate |
| **NVIDIA OpenShell** | Trusted runtime / policy boundary | Sandboxed runtime with gateway and supervisor | Policy checks at documented enforcement points; policy prover checks supported policy changes | Credential mediation is a first-class concern | Runtime and policy lifecycle; provider-specific | Prover coverage is bounded; verification is not authorization or proof of total enforcement |
| **AWS Strands Box / Dogwood** | Containment + action policy | OS-level containment plus supported gateways/interpreters | History-aware semantic checks for operations routed through supported enforcement points | Evaluate separately for each access path | Sandbox lifecycle with shared policy/event context | Direct OS access may not have the same visibility as gateway-mediated operations |
| **Microsoft MXC** | Portable containment SDK | Multiple OS- and VM-based backends | Typed policy/configuration mapped to backend capabilities | Backend/integration dependent | Common API across backends | A common schema does not imply equivalent enforcement; unsupported controls must remain visible |
| **OpenSandbox** | Sandbox API / platform | Depends on selected backend | Provides lifecycle/API integration; policy guarantees depend on backend and surrounding controls | Integration dependent | Common API/SDK across supported providers | API compatibility is not a security-equivalence contract |
| **OpenAI Agents API** | Managed agent harness / sessions | Depends on managed or self-hosted execution setup | Harness guardrails and environment enforcement are distinct concerns | Deployment/integration dependent | Managed sessions and orchestration; ownership varies | Do not assume the harness owns or enforces every property of the execution environment |
| **OpenAI Sandbox Agents** | Application-owned execution environment | Container-like sandbox environment | Environment controls depend on configured sandbox and application policy | Application/integration dependent | Filesystem, commands, packages, ports and resumable state | Keep this environment abstraction distinct from the managed Agents API |
| **Anthropic Claude / Cowork** | Product-specific local or cloud execution | Depends on execution mode; cloud isolation and local protections differ | Egress and connector controls vary by mode and integration | Some cloud paths use server-side connectors and short-lived credentials | Temporary cloud environment or local execution | Never transfer a security claim from one execution mode to another without checking its boundary |
| **Docker Sandboxes** | Developer / agent sandbox implementation | Depends on the Docker sandbox runtime and configuration | Isolation is provided by the sandbox; additional policy coverage depends on configured controls | Depends on integration | Familiar workspace and agent workflows | Do not infer identical guarantees from the generic term “sandbox” |
| **Cloudflare Sandbox SDK** | Container-backed execution API | Container-backed execution controlled through Workers | SDK provides execution/lifecycle APIs; authorization and policy guarantees need separate evaluation | Assess credential handling separately from sandbox APIs | Worker-controlled sandbox lifecycle | Do not infer credential brokering or workload-wide policy enforcement from the sandbox API alone |

#### How to use this submatrix

- **Start with the guarantee, not the product.** Specify required filesystem, network, process, identity and credential constraints for the Work.
- **Separate requested, supported, applied and observed.** A configuration accepted by an API is not proof that the backend enforced it.
- **Record enforcement coverage.** Identify which actions pass through the policy engine and which can use another path (shell, Python, HTTP, MCP, child processes or direct OS access).
- **Preserve backend differences.** If a provider cannot enforce a required control, mark it unsupported or weaker; do not silently normalize it into success.
- **Treat lifecycle as a security transition.** On resume, replacement or migration, re-check runtime identity, effective policy and credential scope rather than assuming continuity.
- **Treat evidence gaps as results.** Missing telemetry or unverifiable enforcement should be reported as unknown, not as a successful check.

The first useful experiment is a small provider-contract test suite: attempt forbidden filesystem/network/process actions, inspect credential exposure, exercise child-process and alternate-tool paths, then suspend/resume or replace the backend. Record **requested policy, effective policy, enforcement point, verification coverage, telemetry gaps, task outcome, latency/cost and human interventions**. This is an evaluation protocol, not a new EOKS primitive.

---

## 4. Capability families and overlap

| Capability | Strong examples | Also relevant |
|---|---|---|
| **Durable canonical knowledge** | `CLAUDE.md`, OKF | Tencent Wiki, Obsidian |
| **Human research workspace** | Obsidian | Markdown/ADR systems |
| **Repository representation** | Graphify, Understand Anything, Codebase Memory MCP | GrapeRoot, CodeSight |
| **Context acquisition** | GrapeRoot, CodeSight, Xirp | Graphify, QMD |
| **Context retrieval** | QMD, GrapeRoot | memory systems, graph MCPs |
| **Context transformation/compilation** | Spotify Portal/Shunt, GrapeRoot | Xirp, memory systems |
| **Context compression/lifecycle** | LCM | memory systems, Portal/Shunt |
| **Persistent memory** | Hindsight, Mem0/LangMem/Zep | OpenWolf, MemoryLACE, Tencent |
| **Evolving memory/learning** | Hindsight, MemoryLACE | LCM, session-learning systems |
| **Structural evidence** | Graphify, Codebase Memory MCP | Understand Anything, CodeQL |
| **Semantic/dataflow evidence** | CodeQL, Semgrep | Understand Anything |
| **Local deterministic invariants** | TypeScript, ESLint, ts-morph | Semgrep |
| **Architecture evidence/assurance** | TrueCourse, modularity | Superpowers |
| **Agent execution** | Claude Code, OpenHands, Aider | Cody, CodeRabbit |
| **Execution isolation / sandboxing** | OpenShell, Strands Box, MXC, ACE | OpenSandbox, OpenAI sandbox environments, Docker Sandboxes |
| **Durable execution substrate** | Google AX, ACE | OpenAI sessions, provider-specific sandbox lifecycle |
| **Orchestration** | Conductor-style systems, Langroid | OpenHands workflows |
| **Observability** | Opik, LangSmith/Langfuse | agent-native traces |
| **Evaluation** | Opik, Promptfoo, benchmark suites | LangSmith/Langfuse, OpenAI Evals |
| **Outcome feedback/optimization** | Opik, agentic eval systems | benchmark harnesses |
| **Reasoning strategies** | ADHD-style strategies | Superpowers, reviewer workflows |

This many-to-many view is the intended replacement for the old one-tool/one-layer categorization.

---

## 5. Context engineering: six distinct mechanisms

“Context tool” is too broad. The current research separates:

```text
source
  ↓
1. acquisition       → find potentially relevant resources
  ↓
2. representation    → structure them for navigation/retrieval
  ↓
3. retrieval         → select relevant evidence
  ↓
4. transformation    → compress/filter/rewrite/cache/route
  ↓
5. compilation       → assemble task-specific model context
  ↓
6. delivery          → inject/expose it through the agent/harness
```

| Mechanism | Representative tools | EOKS question |
|---|---|---|
| Acquisition | Xirp, CodeSight, Graphify | What sources can we reach? |
| Representation | Graphify, Understand Anything, OKF | What structure makes them reusable? |
| Retrieval | QMD, GrapeRoot, memory systems | What should be selected? |
| Transformation | Spotify Portal/Shunt, LCM | Can we reduce/reshape context without losing required evidence? |
| Compilation | GrapeRoot | Can sufficient context be prepared before reasoning? |
| Delivery | GrapeRoot launcher, MCP, hooks, Portal/Shunt | Where should context enter the execution loop? |

Spotify's September 2026 Portal work is particularly important here: its published result reports a 90% reduction in Claude Code token usage in the described workflow, showing that significant optimization can happen at the harness/I/O boundary rather than through another persistent knowledge system. citeturn0search33

GrapeRoot is complementary: its public repository describes a launcher that builds a semantic code graph and pre-loads relevant code before the coding agent reasons; the launcher is open source while its graph engine is proprietary. citeturn0search4

Therefore Portal/Shunt should **not** be categorized as memory/RAG, and GrapeRoot should **not** be reduced to “a graph.”

---

## 6. Memory, caching, knowledge and context evolution

These are distinct:

```text
cache
  = avoid recomputing/re-reading

memory
  = retain information from prior experience

knowledge
  = durable information intended to remain useful

context evolution
  = decide what survives, changes, is superseded or discarded

learning
  = change future behavior based on outcomes
```

This distinction is important for LCM, Hindsight, MemoryLACE, Mem0/LangMem/Zep, OpenWolf, Tencent Agent Memory and session-learning systems.

The useful comparison questions are:

- What is retained?
- Why is it retained?
- What evidence supports it?
- How does it evolve?
- Can it be invalidated/superseded?
- Can the source/lineage be recovered when it becomes wrong?
- Does it demonstrably improve a future task?

A memory system is therefore not automatically a knowledge system, and a context cache is not automatically learning.

---

## 7. Evidence-provider comparison

EOKS should treat deterministic and semantic tools as an **evidence stack**, not a ladder where the deepest tool is always better.

| Provider | Best evidence | Determinism | Typical depth | Cost | Typical escalation |
|---|---|---|---|---|---|
| TypeScript compiler | type/state constraints | high | local/project | low | Semgrep/tests |
| ESLint | local structural/policy rules | high | local/project | low | Semgrep |
| ts-morph | targeted TS semantics | high | project | low-medium | Semgrep/CodeQL |
| Semgrep | pattern + targeted dataflow | high | local→project | low-medium | CodeQL/tests |
| Graphify | structural relationships | high-ish | repository | low-medium | semantic analysis |
| CodeQL | deep interprocedural/dataflow | high | repository/system | medium-high | runtime/tests |
| Understand Anything | structural + LLM semantic interpretation | mixed | repository/domain | medium-high | deterministic/runtime |
| Tests/runtime | observed behavior | high for observed execution | system | variable/high | independent review |
| LLM review | interpretation/attack/coverage | low | broad | medium | deterministic/runtime |
| TrueCourse | architecture/spec behavior guards | mixed | project/system | medium | tests/review |

The EOKS policy hypothesis is **minimum sufficient evidence**, not maximum analysis depth.

---

## 8. Evaluation and observability

Tracing, evaluation and reliability should remain separate concepts:

```text
execution
   ↓
trace / telemetry
   ↓
trajectory analysis
   ↓
outcome evaluation
   ↓
diagnosis
   ↓
policy/context/tool change
   ↓
next execution
```

**Opik** is particularly relevant because its current public documentation spans agent tracing, end-to-end and step-level evaluation, datasets/experiments, monitoring and agent optimization. citeturn0search3turn0search14turn0search27

Spotify's background-agent work is useful complementary production evidence: Spotify reports more than 1,500 merged AI-generated PRs in one line of work and describes feedback loops for unsupervised/background coding agents. citeturn0search16turn0search25

Thus Opik belongs not merely under “observability,” but across **observability → evaluation → optimization feedback**.

---

## 9. Tool relationships

Pairwise comparisons should be sparse and semantic. The useful relationship types are:

```text
overlap
complement
alternative
specialization
escalation
precondition/dependency
```

Representative relationships:

```text
GrapeRoot ──compiles──> Graphify-like structural evidence
GrapeRoot ──alternative──> native agent retrieval
Portal/Shunt ──complements──> GrapeRoot
Graphify ──complements──> Semgrep/CodeQL
Semgrep ──escalates──> CodeQL
TypeScript/ESLint ──cheaper alternative──> deeper analysis for suitable invariants
Hindsight ──extends──> session memory with reflection/evolution
LCM ──complements──> memory systems through recoverable context lifecycle
OKF ──represents──> durable knowledge; it is not a runtime/context compiler
TrueCourse ──assures──> behavior/architecture; it is not a general memory system
Opik ──measures──> execution/evaluation; it is not itself the EOKS control plane
```

This is more useful than a complete pairwise matrix, which would become stale and unreadable.

---

## 10. What should we actually experiment with?

The landscape contains many projects but relatively few distinct hypotheses.

### Context

```text
native agent retrieval
        vs
Portal/Shunt-style transformation
        vs
GrapeRoot-style proactive compilation
        vs
hybrid
```

### Knowledge

```text
CLAUDE.md / ordinary project docs
        vs
structured OKF-style knowledge
```

### Structure

```text
native search
        vs
Graphify
        vs
Understand Anything / Codebase Memory MCP
```

### Memory / evolution

```text
no persistent memory
        vs
retrieval memory
        vs
Hindsight-style evolving memory
        vs
lossless/lifecycle-oriented context
```

### Evidence

```text
compiler/types
   → lightweight rules
   → structural graph
   → Semgrep
   → CodeQL
   → runtime/tests
   → independent attack/review
```

### Evaluation

```text
agent trace
   → step/tool-call metrics
   → task outcome
   → diagnosis
   → intervention
   → repeat task
```

Do not assume that more agents, more memory, more context or deeper analysis is better. The experiment should measure the marginal benefit against cost and complexity.

---

## 11. Remaining gaps

### Tool facts need provenance

The comparison itself is knowledge. Eventually important claims should record:

```text
claim → source/version → observation type → confidence → last verified
```

### Lifecycle must be explicit

Compare:

```text
snapshot → refresh → incremental update → dependency-aware invalidation
         → semantic evolution → promotion/supersession
```

### Losslessness/recoverability matters

Especially for context systems:

```text
raw evidence → compression/summary → context
```

Can the source evidence be recovered when the derived context is challenged?

### Retrieval utility is downstream

Measure:

```text
retrieved → included → used → affected action → affected outcome
```

not only retrieval relevance.

### Causal attribution remains weak

When a tool appears to help, separate:

- better context;
- better evidence;
- better workflow;
- more tokens;
- more retries;
- model differences;
- reduced I/O/tool overhead.

Otherwise the landscape remains descriptive rather than empirical.

---

## 12. Research posture

This document should answer four questions:

1. **What mechanism does this tool demonstrate?**
2. **How established is the evidence that the mechanism works?**
3. **Where does it overlap or compose with other mechanisms?**
4. **What experiment would tell EOKS something we do not already know?**

The intended evolution is:

```text
research notes
    ↓
capability profiles
    ↓
this landscape
    ↓
controlled EOKS experiments
    ↓
measured evidence
    ↓
selection policy
    ↓
adaptive control loop
```

The landscape is therefore the **map of the territory and its evidence status**, while `tool-capability-model.md` remains the formal basis for provider selection.
