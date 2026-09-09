# EOKS synthesis: execution graphs and coordination

> Focused follow-up to the conceptual synthesis. This document makes execution topology explicit without introducing a new EOKS runtime primitive or making graph engineering synonymous with EOKS.

## Scope

Recent work on graph engineering exposes a dimension that is present but under-specified in the current EOKS synthesis: **how work is coordinated across runs, agents, tools, verification steps and human gates**.

The useful conclusion is not that EOKS needs another graph implementation. It is that EOKS should distinguish three graph roles:

```text
knowledge / structural graph  -> what exists and how things relate
context graph / acquisition   -> what evidence may be useful
execution graph                -> what work happens, in what dependency order
```

These representations may interact, but they have different semantics and lifecycles.

## 1. Execution graph as a control representation

An execution graph describes work dependencies and control flow:

```text
                         Task
                           |
                         Plan
                           |
              +------------+------------+
              |            |            |
           Research      Analyze      Verify
              |            |            |
              +------------+------------+
                           |
                         Reduce
                           |
                       Synthesize
                           |
                        Evaluate
                       /        \
                   sufficient  insufficient
                       |             |
                      stop      acquire / verify /
                                retry / branch / escalate
```

A node can represent deterministic computation, a model call, an agent run, a verification step, a reducer, a human approval step, or another existing execution mechanism. The graph is a **coordination representation**, not necessarily a new runtime object.

The canonical EOKS `Run` remains the useful execution unit. A run may participate in a larger execution graph and may itself contain loops or subagents.

## 2. Edges should express real dependencies

A useful design test is:

> **What exact state, artifact or evidence crosses this edge, and why does the downstream work depend on it?**

If the downstream node does not require the upstream result, the dependency may be artificial and the work can potentially execute independently.

This yields a practical distinction:

```text
fake dependency
A -> B

real dependency
A --[typed artifact/evidence/state]--> B
```

The second form is more useful for scheduling, observability and optimization because it makes the reason for sequencing inspectable.

An execution edge may carry or constrain:

- artifact/evidence identity;
- provenance and source/version;
- readiness condition;
- validity/freshness;
- authority/permission;
- resource requirements;
- failure semantics;
- downstream assurance requirements.

These remain properties of existing Tasks, Contexts, Runs, Decisions, Policies, Evaluations and Outcomes rather than new primitives.

## 3. Topology is workload-dependent

Graph engineering should not imply that more agents, more nodes or more parallelism is better.

The topology should follow the dependency structure of the workload:

```text
inherently sequential work
A -> B -> C

independent work
A ─┬─> B
   └─> C

mixed work
      ┌-> B ─┐
A ----┤      ├-> D
      └-> C ─┘
```

Multi-agent evidence is increasingly consistent with this caution: parallelization can help tasks with genuine independence, while unnecessary decomposition can add coordination cost and error propagation. Therefore the EOKS objective should be **minimum sufficient coordination**, not maximum orchestration.

The relevant comparison is usually against a simpler topology:

```text
single run
    vs
minimal multi-step workflow
    vs
parallel / multi-agent workflow
```

A more complex graph should earn its additional cost through measurable improvement in the task outcome or assurance.

## 4. Fan-out -> reduce -> verify -> synthesize

The strongest synthesis implication is for the existing Synthesis concept.

Avoid treating synthesis as simply another model call over raw worker transcripts:

```text
workers -> giant prompt -> synthesizer
```

Prefer an evidence-oriented pipeline:

```text
                 +-> worker A --+
Task -> fan-out -+-> worker B --+-> reduce -> verify -> synthesize
                 +-> worker C --+
```

### Fan-out

Generate independent evidence or analyses only where the work can genuinely proceed independently.

### Reduce

Normalize, deduplicate, reconcile obvious overlap, preserve provenance, and turn raw observations into an inspectable evidence set.

### Verify

Use the cheapest independent verification that is sufficient for the consequence of the decision. Deterministic checks, tests, analyzers, additional evidence, another model, or human review are alternative mechanisms rather than one universal verifier.

### Synthesize

Reason over the reduced evidence state rather than over every raw trajectory. The synthesizer should be able to distinguish evidence, claims, conflicts and uncertainty.

This reinforces the existing EOKS distinction between evidence and context: the model should normally receive the evidence it needs, not the internal graph, index or storage system itself.

## 5. Graph engineering and the EOKS conductor

The conductor already selects resources, working sets and execution strategies. The execution-graph perspective clarifies that this may include **topology selection**:

```text
                         CONDUCTOR
                             |
            +----------------+----------------+
            |                |                |
         context         resources         topology
            |                |                |
       working set      model/tool      sequential/parallel
                                          fan-out/reduce
                                          verify/escalate
```

This does not require a graph scheduler as a new EOKS subsystem. It means that topology is one possible control decision when the workload and evidence justify it.

A topology decision should consider at least:

- dependency structure;
- expected information gain;
- resource capability;
- latency and cost;
- failure correlation;
- verification requirements;
- consequence/risk;
- authority and human gates;
- historical outcome quality.

## 6. Three graphs, three questions

### Knowledge / structural graph

**Question:** What exists and what relationships hold in the represented domain?

Examples include repository structure, imports, calls, symbols, dependencies and architecture relationships.

### Context / acquisition graph

**Question:** What information could be useful for the current task, and how can it be acquired?

This can connect a task to candidate representations, evidence providers and relevant evidence slices.

### Execution graph

**Question:** What work should happen, in what dependency order, with what control conditions?

Keeping these separate prevents an important category error:

```text
repository relationship
        !=
execution dependency
        !=
context relevance
```

A repository call edge does not mean two reasoning steps must be sequential. A context-relevance relationship does not imply execution dependency. An execution dependency may exist even when no corresponding domain relationship exists.

## 7. Graph-level evaluation

The existing EOKS evaluation model should extend naturally to topology-level measurements.

| Signal | What it measures | Interpretation |
|---|---|---|
| Critical-path latency | Time along the dependency bottleneck | Whether sequencing dominates execution |
| Topology width | Concurrent work at each stage | Whether available independence is being exploited |
| Dependency/edge count | Coordination complexity | Whether the graph is over-connected |
| Fan-out efficiency | Unique useful evidence per parallel branch | Whether branches justify their cost |
| Retry amplification | Additional work caused by failures | Whether failures propagate through topology |
| Verifier rejection rate | Fraction of outputs rejected by verification | Whether generation or decomposition is poorly scoped |
| Reduction/compression ratio | Raw worker material versus retained evidence | Whether reduction removes redundancy without losing useful information |
| Human intervention rate | Frequency of approval/escalation | Where autonomy or assurance remains insufficient |
| Evidence yield | Validated useful evidence per acquisition/computation cost | Whether topology produces sufficient information economically |
| Dependency efficiency | Real information dependencies relative to execution edges | Whether sequencing is justified |

The last two are **candidate EOKS metrics**, not established industry standards. They should be validated empirically before becoming normative.

## 8. Topology should be evaluated end-to-end

A graph is not successful merely because it completes or because every node succeeds.

Evaluation should compare:

```text
                 topology
                    |
       +------------+------------+
       |            |            |
    outcome      assurance      cost
       |            |            |
       +------------+------------+
                    |
                end-to-end
                 utility
```

For a coding task, a useful evaluation record can include:

- task correctness/completeness;
- relevant evidence coverage;
- context tokens and churn;
- tool/model calls;
- critical-path latency;
- total cost;
- retries and failures;
- verification effort;
- escaped defects;
- human intervention.

This makes it possible to discover that a more elaborate graph improved a local metric while making the overall workload worse.

## 9. Relationship to reflection, loops and synthesis

The concepts now fit together cleanly:

```text
SYSTEM
  |
  +-- execution graph
       |
       +-- loop
       |    |
       |    +-- act -> observe -> reflect
       |
       +-- parallel workers
       |
       +-- reduce / verify
       |
       `-- synthesize
```

- **Reflection** asks what happened, what explains it, and what should change in a local trajectory.
- **Loop engineering** governs repeated execution of a worker.
- **Graph engineering** governs dependencies and coordination among workers/loops.
- **Synthesis** generalizes across reduced evidence and trajectories.
- **Promotion** asks whether the resulting knowledge, procedure or artifact has enough evidence to become reusable.

This preserves the existing lifecycle distinctions rather than turning every concept into a graph node.

## 10. Relationship to agentic evaluation and observability

Graph-level evaluation is a natural extension of trajectory-level evaluation.

```text
node / run
   -> trajectory evidence
   -> outcome

whole graph
   -> topology evidence
   -> aggregate outcome
```

Telemetry systems can provide execution observations, while evaluation interprets those observations for a task or control decision. A trace is therefore not automatically an evaluation, and topology metrics should not replace task-level outcome measurement.

This also gives tools such as Opik a clear place in the EOKS landscape: they can provide execution/evaluation instrumentation without becoming the EOKS control model.

## 11. Design principle: minimum sufficient coordination

The combined synthesis can be stated as:

> **Use the minimum coordination structure that provides the evidence, assurance and outcome required by the workload.**

This is the graph-level counterpart to minimum sufficient evidence and minimum sufficient context.

```text
minimum sufficient context
            +
minimum sufficient evidence
            +
minimum sufficient computation
            +
minimum sufficient coordination
            |
            v
      acceptable outcome
      at acceptable cost/risk
```

The principle is deliberately asymmetric: additional context, computation, verification or coordination must justify itself through measurable benefit.

## 12. What this does not introduce

This synthesis does **not** introduce:

- an `ExecutionGraph` runtime primitive;
- a universal graph representation;
- a graph database requirement;
- a mandatory multi-agent architecture;
- a universal topology optimizer;
- a universal graph-quality score.

Existing EOKS primitives remain sufficient to represent the current hypothesis:

```text
Task · Context · Run · Decision · Policy · Evaluation · Outcome
```

Execution topology is currently a cross-cutting control representation expressed through those primitives.

## 13. Validation agenda

The highest-value experiments are small and falsifiable.

### A. Single versus parallel topology

For workloads with known independent and sequential components, compare a single run, a minimal sequential workflow and a parallel workflow.

Measure outcome quality, critical-path latency, total cost, retries, evidence coverage and human intervention.

### B. Fan-out width

Increase parallel branches while holding the task fixed. Determine where marginal evidence yield falls below coordination cost.

### C. Raw transcript versus reduced evidence synthesis

Compare synthesis over raw worker outputs with synthesis over a provenance-preserving reduced evidence set.

Measure correctness, contradiction handling, context size, latency and cost.

### D. Dependency pruning

Remove candidate edges that do not carry required state/evidence and measure whether the simplified graph preserves outcome and assurance while reducing latency or failure propagation.

### E. Topology adaptation

Use prior run outcomes to choose between simple and complex topologies. Test whether adaptive selection beats a fixed topology on a workload distribution without creating unacceptable selection failures.

These experiments should be connected to the existing EOKS evaluation methodology and research agenda. They are not evidence that a particular topology is universally superior.

## 14. Relationship to the canonical EOKS architecture

The canonical loop remains:

```text
intent
  -> desired state / outcome
  -> policy
  -> conductor / reconciliation
  -> resource + working-set + execution selection
  -> run
  -> observe / verify
  -> outcome
  -> evaluation / evidence
  -> actual state
  -> reconcile
```

The execution-graph synthesis adds one clarification:

```text
execution selection
        |
        v
  choose / adapt topology
        |
        v
       runs
```

Topology therefore belongs to the **control decision space**, not to a new foundational ontology layer.

The strongest current hypothesis is that EOKS should eventually be able to observe enough of this topology to learn whether a particular coordination structure is actually useful. Until that is demonstrated, graph engineering remains a synthesis lens and experimental target rather than an architectural commitment.
