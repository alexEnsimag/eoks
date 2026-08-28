# EOKS architecture

EOKS is best understood as a set of cooperating planes rather than a single agent.

```text
                         EOKS CONTROL PLANE
              scheduling · policies · resource selection
                                  |
        +-------------------------+-------------------------+
        |                         |                         |
  KNOWLEDGE PLANE          CONTEXT PLANE            EXECUTION PLANE
 canonical knowledge       selection · assembly      workflows · agents
 graphs · semantic         ranking · compression     reasoning strategies
 history · runtime         progressive disclosure    tools · artifacts
 evidence                  context workbench
        |                         |                         |
        +-------------------------+-------------------------+
                                  |
                         EVALUATION / FEEDBACK
                  evidence · quality · confidence · outcomes
                                  |
                           OBSERVABILITY
                decisions · cost · latency · provenance
```

## Control plane

The control plane decides **what should happen next**. It can schedule tasks, select models, evidence providers and tools, impose policies, and react to evaluation signals.

The Kubernetes analogy is useful here: a desired task state can be reconciled against observed execution. AI workloads differ because reasoning quality is probabilistic, context is mutable, and resource requirements are semantic rather than only CPU/memory based.

## Knowledge plane

The knowledge plane maintains durable project information and the representations used to navigate it. It should not assume that a single graph is the canonical form.

Possible representations include:

- hierarchical `CLAUDE.md` / Markdown as human-reviewable canonical project knowledge;
- ADRs and cross-cutting architecture documents;
- deterministic structural graphs and symbol indexes;
- semantic indexes and concept clusters;
- historical timelines and decision records;
- runtime observations;
- episodic and procedural memory;
- provenance and confidence metadata.

The key distinction is **canonical knowledge versus derived evidence providers**. Humans can maintain a concise mental model while machines maintain graphs/indexes/caches that make authoritative evidence easier to locate.

See [Engineering knowledge as a multi-representation system](knowledge-representations.md) and [Knowledge base and persistent project knowledge](knowledge-base.md).

## Context plane

The context plane constructs the information supplied to a reasoning step. It should support retrieval, ranking, compression, deduplication, conflict detection, provenance, progressive disclosure and explicit context budgets.

The context plane is not a storage system. It is a **compiler from task + workflow + available evidence into task-specific model context**.

A graph, semantic index or memory store should normally be queried by this plane rather than dumped into the model context.

A useful mental model is a **context budget**, not a context window: every item has relevance, freshness, reliability, cost and interaction effects.

### Context Workbench

The Context Workbench is the proposed interactive observability/control surface for this plane. Context should be represented conceptually as inspectable blocks or clusters rather than an opaque concatenated prompt. A block can represent knowledge, a decision, a dependency slice, raw evidence, a test result, a Git event, a procedure or working memory.

The workbench should let the system automatically assemble context while allowing humans to inspect and, when useful, edit it:

- include, exclude or pin blocks;
- inspect provenance, freshness, confidence and token cost;
- explain why a block was selected or omitted;
- enforce a token/latency budget;
- compare automatic and optimized context compositions;
- view relationships among task, knowledge, code and evidence as a graph;
- save successful context recipes;
- feed manual edits and task outcomes back into future selection policies.

This is **context observability and control**, not a requirement that humans manually curate every prompt. The default should remain automatic compilation, with intervention available when the system is uncertain or the user wants to inspect the decision.

See [Context Workbench](context-workbench.md).

## Context quality

Context quality should be treated as multidimensional rather than reduced immediately to one scalar. Useful dimensions include relevance, coverage, redundancy, reliability, uncertainty, freshness, dependency completeness, provenance, contradiction risk, ordering/structure and token/latency cost.

A promising experimental metric is **marginal context value**: the change in task quality associated with adding a block relative to its resource cost. This is a benchmark framing, not a claim that online task-quality probabilities can be estimated exactly.

The objective is not maximum information. It is maximum useful evidence per unit of context and reasoning cost.

## Context layers

Context can be decomposed into layers so different workflow nodes can request different information budgets:

```text
L0 task
L1 constraints
L2 persistent knowledge
L3 structural context
L4 evidence
L5 working memory
L6 reasoning state
```

This is an information-architecture boundary, not a prescription for how the model must reason. If a layer distinction does not improve quality, cost or observability, it should not be imposed merely for structure.

## Execution plane

The execution plane runs workflows, reasoning strategies, agents and tools; obtains artifacts; modifies repositories; executes tests; and records outcomes.

Workflows answer **what should happen next**. Reasoning strategies answer **how a reasoning step should approach its problem**. These are distinct from knowledge, which answers **what the system knows**.

See [Agent workflows and reasoning strategies](agent-workflows.md).

## Evaluation and feedback

Evaluation closes the loop. Results should feed back into model selection, context construction, memory/knowledge updates and task scheduling.

Confidence should be evidence-oriented rather than only model-reported. Examples include:

- deterministic extraction versus LLM inference;
- test and static-analysis results;
- review outcomes;
- runtime observations;
- freshness and provenance of the underlying source.

Context evaluation should additionally record what was included, omitted and manually changed, so context-selection policies can be compared against outcomes.

## Continuous knowledge updates

Knowledge maintenance should be incremental, like a modern build system rather than a full rebuild after every change.

```text
code/doc/event change
        |
   impact detection
        |
  +-----+-----+-----+-----+
  |           |           |
 graph     semantic    knowledge
 update     update     candidate
  |           |           |
  +-----+-----+-----+-----+
        |
 context-cache invalidation
```

Cheap deterministic updates should happen frequently. LLM-heavy reasoning should be reserved for higher-signal events such as merged PRs, incidents, architecture changes or completed workflows.

Hooks are event boundaries into this lifecycle. They should not imply that the entire knowledge system is recomputed after every tool call.

## Context invalidation

Context caches and derived context blocks must be revision-aware. A useful dependency is:

```text
source revision
      |
knowledge/evidence representation
      |
context block
      |
compiled context
```

When an authoritative source changes, affected derived representations and cached context should be invalidated or marked stale according to their dependency scope. This is another reason provenance and freshness belong in the context plane rather than being added only as UI metadata.

## Subagents

Subagents provide isolation but can also create repeated repository-discovery cost. EOKS should distinguish **isolated reasoning** from **isolated knowledge**.

Where available, a subagent should receive a context contract containing the task, known facts, relevant nodes, scope and unresolved questions. The subagent remains free to retrieve missing evidence, but does not have to reconstruct the entire repository from zero.

This should be benchmarked rather than assumed to improve outcomes.

## Compaction and session boundaries

Conversation compaction is one mechanism for continuing a long session; it is not equivalent to persistent knowledge or task-specific context compilation. EOKS should make it possible to clear or replace a conversation while reconstructing the minimum sufficient context from durable knowledge and authoritative evidence.

## Model routing

Model routing is orthogonal to context engineering. Routing chooses a model; context compilation chooses the information supplied to that model.

A useful control flow is:

```text
task
 |
context compilation
 |
optimized context
 |
capability / complexity policy
 |
model selection
 |
model
```

This allows context optimization to be evaluated independently before introducing routing as a cost optimization. A router cannot eliminate waste caused by a strong model repeatedly rediscovering the same repository.

## Observability

Observability should expose not only latency and token counts, but **why the system made information and execution decisions**:

- retrieved and omitted context;
- context blocks and their relationships;
- evidence providers queried;
- model/reasoning strategy selected;
- tools invoked;
- evidence considered;
- evaluation outcome;
- confidence and provenance signals;
- knowledge updates and invalidations;
- manual context edits;
- context policy/version used.
