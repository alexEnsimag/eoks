# Agent code understanding, architecture and governance tooling

This note captures the tooling discussion around Claude-oriented developer tools and places each capability in the EOKS model. These are **adjacent systems and evidence**, not proposed EOKS dependencies.

## Why this category matters

AI coding agents create a new systems problem: the agent can often make a locally plausible change without possessing a durable, explicit model of the repository's architecture, business rules, or prior decisions.

Two complementary needs emerge:

1. **Understand the system** — construct reusable structural and semantic evidence about the codebase.
2. **Preserve the system** — check proposed changes against architectural and behavioral constraints.

This suggests that repository understanding and architecture governance should be treated as separate capabilities in EOKS, connected through shared evidence and policy.

```text
                    EOKS control / policy
                            |
             +--------------+--------------+
             |                             |
       understanding                 assurance
             |                             |
   code/context resource          architecture/spec checks
             |                             |
   +---------+----------+          +-------+--------+
   |                    |          |                |
Understand Anything   static      TrueCourse    deterministic
   knowledge graph     analysis      analyze       rules/tests
   + semantic views       |          + guard          |
             +------------+--------------+-----------+
                          |
                    evidence layer
                          |
                  context assembly
                          |
                    agent execution
                          |
                     evaluation
```

## Understand Anything

[Understand Anything](https://github.com/Egonex-AI/Understand-Anything) turns a repository into an interactive knowledge graph. Its current implementation combines deterministic parsing (including Tree-sitter) with LLM analysis and produces a graph covering files, functions, classes and dependencies. It also exposes architectural/domain views, guided tours, semantic search, diff impact analysis and explanations. The project supports multiple agent/IDE environments rather than Claude Code alone.

### EOKS interpretation

This is best understood as a **context/evidence provider**, not as an agent runtime.

A useful EOKS contract would be something like:

```text
CodeKnowledgeProvider
  input: repository revision / scope
  output:
    - symbols
    - dependencies
    - architectural relationships
    - domain relationships
    - explanations
    - impact relationships
  provenance: revision + extraction method + confidence
```

The important idea is that an agent should be able to request *code understanding* as a resource instead of repeatedly exploring the same repository from scratch.

### Strengths

- Makes repository structure visible to humans and agents.
- Combines deterministic structural facts with LLM-generated explanations.
- Can persist the graph and reuse it between sessions.
- Diff impact analysis is particularly relevant to an EOKS execution/evaluation loop.
- The project has substantial community traction and broad platform support.

### Limitations / questions

- Initial analysis can be expensive in tokens on large repositories; incremental analysis reduces the ongoing cost.
- A graph is not automatically a good context. EOKS still needs retrieval, relevance and context-budget policies.
- LLM-derived relationships and summaries need provenance and validation if they become durable knowledge.
- A visually impressive graph should not become the definition of the system's architecture.

### EOKS placement

**Context + knowledge/evidence layer**, with a possible role in evaluation through impact analysis.

---

## TrueCourse

[TrueCourse](https://github.com/truecourse-ai/truecourse) takes a different approach. It combines deterministic code analysis with LLM rules and adds a specification-to-guard pipeline. Its analysis covers architectural/code defects such as circular dependencies, layer violations and dead modules, while the guard workflow turns documented behavior from PRDs/ADRs/READMEs and related specifications into scenario tests that can be executed to detect business-logic drift.

### EOKS interpretation

TrueCourse is primarily an **assurance / evaluation / policy enforcement capability**.

The especially interesting distinction is between:

```text
Understand the code       -> What does the system appear to be?
Guard the system          -> What is the system supposed to do?
Analyze a change          -> What changed and what constraints did it violate?
```

That distinction maps naturally to EOKS's evidence/evaluation/control loop.

### Strengths

- Uses deterministic analysis where deterministic analysis is appropriate.
- Architecture checks can catch structural drift that ordinary tests may miss.
- The spec -> scenario -> deterministic execution model is more interesting than simply asking an LLM to review a PR.
- Repo-local JSON artifacts make the results inspectable and versionable.
- Can use Claude Code for LLM-powered steps while retaining a deterministic path for non-LLM analysis.

### Limitations / questions

- Architecture rules and specifications still need to be maintained; stale contracts can produce noise.
- The hard problem is not merely detecting violations but deciding which constraints are authoritative.
- LLM-authored scenarios require validation and lifecycle management.
- EOKS should not assume that every repository needs maximal static/semantic analysis. Analysis depth should be selected according to the task and the required evidence.

### EOKS placement

**Evaluation + policy + assurance layer**, consuming context/evidence from code analysis and durable project knowledge.

---

## Community signal

These projects have different kinds of community evidence.

Understand Anything has very strong visible GitHub adoption and a broad ecosystem of integrations. This makes it useful prior art for the proposition that reusable code knowledge can become a shared artifact rather than being regenerated independently by each agent session.

TrueCourse is much smaller, but its architectural-governance thesis is strategically interesting. Its lower adoption should be treated as a maturity/adoption signal, not as evidence that the underlying problem is unimportant.

For EOKS, community size should therefore be one input to tool selection, not the architecture itself. A small project may contain the more important abstraction, while a large project may validate only one implementation approach.

---

## Other alternatives and where they fit

The earlier discussion considered several alternatives. They should not be collapsed into one category because they solve different problems.

| Tool / approach | Primary capability | EOKS interpretation |
|---|---|---|
| Understand Anything | Repository knowledge graph / code understanding | Context + evidence provider |
| TrueCourse | Architecture analysis + spec/behavior guard | Evaluation + policy |
| CodeRabbit | AI PR review | Evaluation / execution feedback |
| Sourcegraph Cody | Large-codebase retrieval and coding assistance | Context retrieval + execution |
| Aider | Agentic coding workflow | Execution layer |
| Claude Code + custom checks | General agent runtime + project-specific assurance | Execution + policy integration |
| Semgrep | Lightweight deterministic structural/security analysis | Evidence provider |
| CodeQL | Deep semantic/dataflow analysis | High-cost evidence provider |
| Graphify | Code relationship graph extraction | Evidence provider |

The architectural implication is important: **EOKS should not be another monolithic coding agent.** It should be able to compose capabilities like these according to workload requirements.

For example:

```text
Task: modify authentication flow

scheduler
  -> retrieve repository knowledge
  -> request targeted static/dataflow evidence
  -> assemble relevant context
  -> select model + execution strategy
  -> run agent
  -> run architecture/spec guards
  -> evaluate outcome
  -> persist useful observations
```

The scheduler should not blindly run every available analyzer. It should select the **minimum sufficient evidence** needed for the task, balancing correctness, latency and cost.

---

## A stronger EOKS abstraction: Evidence Providers

This tooling landscape suggests a useful abstraction missing from a simple context/memory split: **evidence providers**.

An evidence provider can answer a bounded question about a workload and return:

- facts or derived relationships;
- provenance;
- scope/revision;
- confidence or validation status;
- cost/latency characteristics.

Examples:

- repository graph -> dependency evidence;
- Semgrep -> structural/security evidence;
- CodeQL -> taint/dataflow evidence;
- Understand Anything -> synthesized code/domain understanding;
- TrueCourse -> architecture/spec compliance evidence;
- tests -> behavioral evidence;
- observability -> runtime evidence.

EOKS can then reason about **evidence sufficiency** rather than treating context as a bag of text.

This connects directly to the broader EOKS thesis: context quality is not just token count. A context can be large but weak if it contains redundant, stale, unproven, or irrelevant information. The control plane should select evidence and assemble context based on the task and its uncertainty.

---

## What I would prototype

A useful experimental vertical slice would combine the two strongest ideas rather than adopting either project wholesale:

1. Generate a deterministic repository graph.
2. Store graph facts with revision/provenance.
3. Allow an agent to request targeted code evidence.
4. Assemble only the relevant evidence into its working context.
5. Record which evidence was used.
6. Run deterministic architecture/behavior checks after the change.
7. Feed evaluation results back into the workload record.

This would test a central EOKS hypothesis:

> **The useful unit of context is not a document or token sequence; it is task-relevant, provenance-bearing evidence assembled under a policy.**

The existing tools can be treated as interchangeable implementations of individual capabilities while EOKS owns the orchestration contract.

## Open research questions

- How should EOKS represent provenance across deterministic and LLM-derived evidence?
- Can a knowledge graph be incrementally maintained cheaply enough to be a default repository service?
- How should confidence be represented when an LLM inferred a relationship rather than a parser observed it?
- How does the scheduler estimate the marginal value of running a deeper analyzer?
- Can architecture rules be learned from the repository and then promoted to explicit policy, without turning inferred conventions into accidental constraints?
- How should stale architecture/spec artifacts be detected?
- Can model switching be evaluated using the same evidence/evaluation pipeline rather than subjective impressions?
