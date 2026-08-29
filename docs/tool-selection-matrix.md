# Tool selection matrix

This is the **human-facing companion** to the EOKS capability model and evidence ledger.

- [`tool-capability-model.md`](tool-capability-model.md) defines the dimensions used to compare providers.
- [`tool-selection.md`](tool-selection.md) defines the selection process.
- The prior-art evidence ledger records how strongly individual claims are supported.

This document answers a different question:

> **I know what I need to accomplish. Which existing tools should I investigate first?**

It is deliberately organized by **problem/subcategory**, not by product. It is a decision aid, not a universal ranking. Evidence strength is shown where useful; it does not mean that a tool is objectively "better".

## Quick decision tree

```text
What do you need?
│
├── Understand a codebase
│   ├── Find known information ........ Search / RAG
│   ├── Understand relationships ...... Code graph / structural analysis
│   ├── Understand semantics/domain ... Semantic code understanding
│   └── Understand the surrounding system
│       .............................. System / organizational context
│
├── Build/retrieve context for an agent
│   ├── Known relevant files .......... Search / RAG
│   ├── Relationship-aware context .... Graph-based retrieval
│   ├── Repeated repository tasks ..... Proactive context compilation
│   └── Cross-session context .......... Memory / repository summaries
│
├── Verify an implementation
│   ├── Types / representable states .. Compiler / type system
│   ├── Local rule .................... ESLint / Semgrep
│   ├── Structural architecture ...... Modularity / graph analysis
│   ├── Deep dataflow ................. CodeQL / Semgrep
│   ├── Runtime behavior ............. Tests / traces / observability
│   └── Spec-to-behavior .............. Executable architecture/spec guards
│
├── Remember / reuse knowledge
│   ├── Facts / experiences ........... Mem0 / Zep / Hindsight
│   ├── Procedures / learned skills ... LangMem / Tencent-style memory
│   └── Repository interaction memory . OpenWolf / similar local memory
│
├── Execute a process
│   ├── Deterministic orchestration ... Conductor
│   ├── Structured development method . Superpowers
│   └── Adaptive planning/reasoning ... Planning / reasoning strategies
│
├── Review / evaluate
│   ├── AI code review ................ CodeRabbit / agent reviewers
│   ├── Prompt/model evaluation ....... Promptfoo and evaluation suites
│   └── End-to-end coding outcome ..... Aider / OpenHands-style benchmarks
│
└── Need context outside the repository
    └── Ownership/services/architecture Xirp / portal-style system context
```

## 1. Codebase understanding

### 1.1 Find known information

**Use first:** repository search, language tooling, conventional RAG/search.

**Best when:** you know a symbol, file, concept or phrase to locate.

**Why:** cheapest path when the question is fundamentally retrieval rather than relationship reasoning.

**Escalate to a graph when:** the question becomes "what depends on this?", "what calls this?", or "what is affected if this changes?".

### 1.2 Structural relationships

**Candidates:** Graphify, GitNexus, CodeGraph and similar code-graph systems.

**Best when:** the question concerns callers/callees, dependencies, imports, symbols, components or impact paths.

**Strength:** structured relationships that are difficult to recover reliably from isolated search results.

**Caution:** a structural graph does not automatically establish semantic dataflow or an invariant. Prefer a specialized analyzer when the question requires proving a value-flow property.

**Evidence:** independent evaluations suggest that graph usefulness is workload-dependent; do not assume graph retrieval beats disciplined search for every task.

### 1.3 Semantic/domain understanding

**Candidates:** Understand Anything, CodeSight and LLM-assisted repository understanding systems.

**Best when:** the question is about business meaning, architectural intent, domain concepts or a higher-level explanation of code.

**Caution:** distinguish parser-derived facts from LLM-generated interpretation. Preserve provenance and freshness so generated explanations are not silently treated as authoritative source facts.

### 1.4 System and organizational context

**Candidate:** Xirp and similar portal/service-aware coding environments.

**Best when:** the implementation depends on ownership, service boundaries, dependencies, architecture decisions, documentation or prior engineering work outside the repository.

**EOKS note:** this is an important category because repository-only context can be insufficient for real engineering decisions.

---

## 2. Agent context

### 2.1 Reactive retrieval

**Candidates:** search/RAG, repository MCP tools, Graphify MCP, conventional agent tools.

**Best when:** the agent can cheaply retrieve information as questions arise.

**Advantages:** simple, flexible, no need to predict the complete context up front.

**Weakness:** repeated exploration can cost tool calls and tokens.

### 2.2 Proactive context compilation

**Candidate:** GrapeRoot-style systems.

**Best when:** tasks repeatedly require repository context and exploration overhead is significant.

**Advantages:** relevant structured context can be prepared before the agent starts reasoning.

**Weakness:** retrieval mistakes happen before reasoning begins; pre-injected context can also become irrelevant or stale.

**Selection rule:** compare proactive, reactive and hybrid strategies empirically rather than assuming proactive injection is universally better.

### 2.3 Persistent repository/session context

**Candidates:** OpenWolf, GrapeRoot session weighting, repository-summary systems.

**Best when:** agents repeatedly revisit the same repository and reconstructing context is expensive.

**Caution:** generated summaries need source references, revision/freshness information and invalidation rules.

---

## 3. Verification and static evidence

### 3.1 Types and compiler invariants

**Candidate:** TypeScript compiler/type system and equivalent language tooling.

**Use when:** the property can be expressed as a type or compile-time invariant.

**Preferred because:** if the language can make an invalid state unrepresentable, a dedicated downstream analyzer is often unnecessary.

### 3.2 Local structural/pattern rules

**Candidates:** ESLint, Semgrep.

**Use when:** the invariant is syntactic, local or expressible as a pattern/dataflow rule supported by the tool.

**Preferred because:** low setup and fast feedback.

**Escalate when:** the property requires deeper interprocedural reasoning than the rule can reliably establish.

### 3.3 Deep dataflow / semantic analysis

**Candidates:** CodeQL, Semgrep where its supported dataflow model is sufficient, other specialized analyzers.

**Use when:** the requirement is a repository-wide or interprocedural source-to-sink/dataflow property.

**CodeQL:** strong candidate for rich query/dataflow questions, at the cost of greater setup/analysis complexity.

**Semgrep:** strong candidate when its supported rule/dataflow model is enough and faster iteration is valuable.

**Important:** neither should be selected merely because the question sounds sophisticated. Match analyzer depth to the actual evidence requirement.

### 3.4 Runtime and behavioral evidence

**Candidates:** tests, runtime traces, observability systems.

**Use when:** observed behavior is relevant or static analysis cannot establish the property completely.

**Caution:** passing tests establish tested behavior, not necessarily universal behavior. Runtime evidence and static evidence are complementary rather than interchangeable.

### 3.5 Architecture/specification assurance

**Candidates:** TrueCourse, modularity/architecture analyzers.

**Use when:** the requirement is about architectural boundaries, documented behavior, or keeping implementation aligned with an explicit specification.

**TrueCourse-style approach:** particularly useful when durable specification is connected to executable scenarios/guards.

---

## 4. Memory and durable learning

### 4.1 Facts and experiences

**Candidates:** Mem0, Zep, Hindsight and related agent-memory systems.

**Use when:** future sessions genuinely benefit from retaining facts, observations, experiences or temporal context.

**Compare:** retrieval quality, temporal validity, deletion/retirement behavior, provenance and operational cost.

**Caution:** memory that retrieves successfully can still be stale or wrong. Durable promotion needs governance.

### 4.2 Procedures and learned skills

**Candidates:** LangMem, Tencent-style agent memory, Hermes-style self-improvement systems.

**Use when:** repeated successful behavior should become a reusable procedure or capability.

**Caution:** repetition is not proof of correctness. Promote procedures based on independently evaluated outcomes, not frequency alone.

### 4.3 Repository interaction memory

**Candidates:** OpenWolf and similar local repository-memory systems.

**Use when:** the valuable memory is specifically what agents have learned about a repository across sessions.

**Caution:** interaction-derived memory should remain distinguishable from authoritative project knowledge.

---

## 5. Workflow and execution

### 5.1 Deterministic orchestration

**Candidate:** Conductor and similar explicit orchestration systems.

**Use when:** ordering, gates, retries, approvals or tool execution must be deterministic and inspectable.

**Strength:** predictable control flow.

**Weakness:** less suitable when the task topology is genuinely uncertain.

### 5.2 Structured development methodology

**Candidate:** Superpowers.

**Use when:** planning, staged implementation, testing, review and verification benefit from explicit process.

**Strength:** makes process expectations concrete.

**Weakness:** can add overhead to small/simple tasks.

**EOKS rule:** select workflow depth according to task risk and uncertainty rather than applying one methodology universally.

### 5.3 Adaptive planning/reasoning

**Candidates:** planning systems and reusable reasoning strategies informed by planning research, including Brafman's work.

**Use when:** the task requires decomposing an uncertain objective, considering alternatives or adapting the sequence of actions.

**EOKS note:** planning is a strategy layer, not a replacement for evidence providers. A planner still needs reliable information on which to base decisions.

---

## 6. Review and evaluation

### 6.1 AI code review

**Candidates:** CodeRabbit and agent-based reviewers.

**Use when:** a second reasoning pass can discover defects, omissions or maintainability problems.

**Best practice:** separate authoring and review roles where practical.

**Caution:** an LLM review is evidence, not authoritative proof. Combine it with deterministic checks when the claim is mechanically verifiable.

### 6.2 Prompt/model evaluation

**Candidate:** Promptfoo and comparable evaluation frameworks.

**Use when:** comparing prompts, models, agents or configurations against a repeatable test set.

**EOKS role:** evaluation evidence can eventually feed the provider-selection loop.

### 6.3 End-to-end coding benchmarks

**Candidates:** Aider, OpenHands and comparable coding-agent benchmark suites.

**Use when:** the question is whether an agent/tool configuration actually improves coding-task outcomes.

**Caution:** benchmark results are workload-specific. Do not turn a benchmark score into a universal tool ranking.

---

## 7. Human knowledge and working environments

### Obsidian

**Best use:** human research, architecture exploration and cross-cutting notes before promotion into reviewed project knowledge.

**Not necessarily:** an agent-runtime dependency.

### OKF

**Best use:** portable durable structured knowledge with provenance/lifecycle semantics.

**EOKS relationship:** candidate representation/interchange layer, not the EOKS control plane itself. EOKS should consume OKF where useful without requiring every evidence source to be converted into it.

---

## 8. Practical selection recipes

### "I need to understand an unfamiliar repository"

Start with:

```text
search
  ↓
language/AST information
  ↓ if relationships matter
code graph
  ↓ if semantic/domain questions remain
LLM-assisted understanding
  ↓ if system boundaries matter
organizational/system context
```

Do not build a graph merely because the repository is large.

### "I need to know whether a value can reach a dangerous sink"

```text
compiler/types
  ↓ if insufficient
Semgrep / lightweight dataflow
  ↓ if insufficient
CodeQL / deep dataflow
  ↓ if static evidence remains incomplete
tests / runtime evidence
  ↓ when useful
independent review
```

The actual escalation depends on the required precision and consequence of error.

### "My coding agent repeatedly wastes context"

Compare:

```text
search/RAG
vs
reactive graph retrieval
vs
proactive context compilation
vs
hybrid
```

Measure task outcome, evidence coverage, tokens, tool calls, latency and failure modes. Do not optimize tokens in isolation.

### "Agents keep forgetting important repository decisions"

Compare:

```text
project docs / ADRs
        +
repository-local memory
        +
structured durable knowledge
```

Use memory for things worth remembering, but promote authoritative knowledge only after validation and with provenance/freshness.

### "I need a reliable development process"

First determine whether the process is:

```text
known and deterministic → explicit orchestration
known methodology       → structured workflow
uncertain/adaptive      → planning + adaptive execution
```

Then add verification gates appropriate to the consequences of failure.

---

## 9. Evidence-aware shortlist

The following is a **starting point for investigation**, not a ranking. Evidence strength refers to the independent-evidence classification in the prior-art ledger, not product quality.

| Problem | First candidates | Useful alternative | Main selection discriminator |
|---|---|---|---|
| Known repository fact | Search/RAG | Cody-style context | Retrieval precision/cost |
| Code relationships | Graphify / GitNexus / CodeGraph | Language tooling | Relationship coverage/freshness |
| Semantic code understanding | Understand Anything / CodeSight | LLM + source | Provenance vs semantic breadth |
| Proactive agent context | GrapeRoot | Hybrid retrieval | Context benefit vs pre-injection cost |
| Organizational context | Xirp | Portal/docs + search | System-context coverage |
| Type invariant | Compiler/type system | ESLint | Can the invariant be encoded in types? |
| Local code rule | ESLint / Semgrep | Custom AST tooling | Rule expressiveness and speed |
| Deep dataflow | CodeQL | Semgrep | Required depth vs setup/latency |
| Architecture/spec guard | TrueCourse / modularity | Custom tests | Executability of the intended constraint |
| Runtime behavior | Tests / observability | Static analysis | Coverage vs observed evidence |
| Persistent factual memory | Mem0 / Zep / Hindsight | Structured project knowledge | Freshness, temporal validity, provenance |
| Procedural memory | LangMem / Tencent-style memory | Hermes-style approaches | Promotion and evaluation |
| Deterministic workflow | Conductor | Explicit CI/workflow | Required control-flow guarantees |
| Structured agent workflow | Superpowers | Custom workflow | Process benefit vs overhead |
| Adaptive planning | Planning/reasoning strategies | Fixed workflow | Uncertainty and task topology |
| AI review | CodeRabbit / agent reviewer | Human review | Defect-finding value + independence |
| Agent evaluation | Promptfoo | Benchmark suites | Repeatability and workload coverage |
| End-to-end coding outcome | Aider / OpenHands benchmarks | Custom task set | Task realism and reproducibility |
| Durable knowledge format | OKF | Markdown/ADRs | Portability/provenance needs |

## 10. How to use this matrix correctly

Do **not** read a row as:

> "This is the best tool for this category."

Read it as:

> "These are plausible providers. Now derive the evidence requirement and compare them against the actual workload."

A good human selection process is:

```text
1. Describe the question.
2. Describe what must be established.
3. Identify the evidence kind/scope/depth required.
4. Start with the cheapest plausible provider.
5. Check whether its evidence satisfies the requirement.
6. Escalate or combine providers if it does not.
7. Record why the final provider(s) were chosen.
8. Measure the outcome and update the evidence base.
```

This is intentionally compatible with the future EOKS control plane: **the human-facing matrix is a discovery aid; structured capability profiles and evidence requirements remain the canonical selection mechanism.**
