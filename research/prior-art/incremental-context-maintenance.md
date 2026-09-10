# Incremental maintenance of derived context

This note follows the recent context-caching research from the perspective of **context evolution** rather than model-serving optimization.

The key question is:

> **When authoritative state changes, can EOKS update only the derived context that is affected instead of rebuilding everything?**

This is a narrower and more operational question than context evolution as a whole. It connects EOKS's existing context-evolution and validated-reusable-computation work to established research on self-adjusting computation, incremental view maintenance, differential dataflow, dependency-aware build systems, and current LLM context-caching systems.

## From cache invalidation to incremental maintenance

A conventional cache asks whether a previous result can be reused:

```text
inputs -> result
          |
       cache hit
```

Dependency-aware incremental computation adds the missing structure:

```text
inputs
  |
  v
computation / dependency graph
  |
  +--> derived A
  +--> derived B
  +--> derived C
```

When an input changes, the system can identify affected computation and reuse unaffected state:

```text
change
  |
  v
identify affected dependencies
  |
  +--> invalidate/recompute A
  +--> reuse B
  +--> update C
```

This is the important bridge to EOKS. The goal is not simply a larger cache. It is a **maintainable derived-state graph** whose nodes have explicit dependencies and whose update cost can be smaller than full recomputation.

## Current LLM caching systems: useful evidence, but not the whole model

The current model-serving ecosystem provides concrete evidence for several layers of reuse that EOKS needs to distinguish. The detailed tool-by-tool evidence is preserved in [`llm-context-caching.md`](llm-context-caching.md), covering vLLM, Hugging Face, Anthropic, Gemini, OpenAI and Redis.

The recurring pattern is:

```text
prefix/KV cache
    -> reuse model computation

prompt/context cache
    -> reuse expensive input processing

semantic result cache
    -> retrieve a similar prior result
    -> validate before reuse

EOKS derived context
    -> preserve a validated semantic representation
    -> track dependencies as evidence changes
```

The production systems therefore give EOKS an important empirical anchor: **cache identity, context ordering, TTLs and scope all affect reuse, but none alone provide a general model for evolving semantic context.**

## Self-adjusting computation

Acar and collaborators developed self-adjusting computation as a model in which computations respond to changes in their inputs using dependency tracking, change propagation and memoization. The work explicitly combines memoization with a dynamic dependence graph so that unaffected computation can be reused while affected computation is re-executed. The experimental literature also shows an important caveat: change propagation is not automatically faster than recomputation; its value depends on the structure and size of changes. [1][2]

This maps closely to a potential EOKS lifecycle:

```text
source state
    |
    v
computation trace / dependencies
    |
    v
derived context
    |
    +---- source changes ----+
                              |
                              v
                    change propagation
                              |
                   +----------+----------+
                   |                     |
               unaffected            affected
                   |                     |
                 reuse             update/recompute
```

The important lesson is not to import self-adjusting computation as an EOKS runtime design. It provides a **theoretical model for selective recomputation**.

## Materialized views

Database research frames a similar problem as incremental maintenance of materialized views. A view is a derived representation of base data; incremental view maintenance computes changes to the view in response to insertions, deletions or updates rather than necessarily recomputing the whole view. Classic work also distinguishes maintenance strategies according to the available information and the kinds of modifications being handled. [3][4]

The analogy is useful:

| Database | EOKS |
| --- | --- |
| base relation | authoritative source/evidence |
| view definition | synthesis/transformation procedure |
| materialized view | derived context/representation |
| update | source/evidence change |
| view maintenance | context evolution / derived-state update |
| stale view | stale context artifact |
| view dependency | provenance/dependency relation |

The analogy should not be taken too literally. LLM-generated synthesis is generally probabilistic and semantically richer than a relational query. The useful transfer is the **maintenance problem**, not the database implementation.

### DBSP and modern incremental view maintenance

DBSP revisits incremental view maintenance for rich query programs and frames incremental computation as maintaining a result as input changes arrive, rather than repeatedly evaluating the complete query. [17]

This strengthens the EOKS analogy because the relevant unit is not necessarily a database row or an entire context document. It can be an intermediate derived representation with its own dependencies and incremental update rule.

For EOKS, this suggests a useful research direction: make the **maintenance strategy itself** part of the provenance of a derived context artifact, rather than treating every update as an opaque LLM call.

## Differential dataflow and repeated change

Differential dataflow extends incremental computation to computations over changing data and nested iteration. Its central operational idea is that a computation can maintain outputs as inputs change instead of rerunning the complete computation. The framework also makes time and change accumulation explicit, allowing multiple changes to be processed together. [5][6]

This is relevant to EOKS because context evolution is unlikely to receive one clean update at a time. A software-engineering workload can produce a burst of changes:

```text
commit
+ test result
+ tool result
+ design decision
+ rejected hypothesis
+ new requirement
```

An eventual context-maintenance system may benefit from treating these as a **change set** and determining the smallest useful update after the changes have been observed, rather than synthesizing after every event.

That connects directly to EOKS's existing idea of **semantic boundaries**: incremental maintenance provides a computational reason to batch changes, while context evolution provides the semantic reason to decide when a new representation is worth materializing.

## Build systems: a practical analogy

Bazel provides a particularly concrete example of dependency-aware reuse. Build actions declare inputs and outputs; action identity and dependency information allow previous outputs to be reused when the relevant inputs have not changed. Hermeticity is important because cache correctness depends on making the relevant inputs observable and reproducible. [7][8]

Nix makes a related distinction through derivations and content-addressed outputs: a build step is defined in terms of inputs, and content-addressing can make identical resulting objects independently reusable. Its documentation also illustrates why naive input identity can cause unnecessary rebuilds when a change in how an input is referenced does not change the resulting content. [9][10]

The EOKS lesson is not that context should become a build system. It is that **reuse needs an identity boundary**.

For a derived context artifact, the relevant identity may include:

```text
source revisions
relevant evidence
synthesis procedure/version
context compiler/version
model/configuration where semantically relevant
policy/permission scope
external dependencies
```

The exact set is workload-dependent. The research question is which dependencies are actually sufficient to establish validity.

## A more precise EOKS model

The combined evidence suggests distinguishing four related operations:

```text
reuse
  = use an existing result without recomputation

invalidation
  = determine that an existing result is no longer eligible

incremental maintenance
  = update a derived result from changes to its dependencies

context evolution
  = decide which derived state should remain active/useful as the workload changes
```

This gives a useful relationship:

```text
                 context evolution
                         |
              decides what should change
                         |
             +-----------+-----------+
             |                       |
        reuse/invalidate       incremental update
             |                       |
             +-----------+-----------+
                         |
                  derived context
                         |
                  context compilation
```

This is stronger than treating context evolution as a specialized cache policy.

## Where LLM systems differ

The classical systems above generally have stronger assumptions than AI workloads.

A build action can often have a deterministic output for a declared input set. A database view has an explicit query definition. A self-adjusting program has a defined computation and dependency semantics.

An LLM synthesis can be affected by model behavior, stochasticity, hidden provider behavior, changing retrieval, external state and underspecified semantic dependencies. Therefore an EOKS system should not assume that a syntactically unchanged dependency set proves semantic equivalence.

This reinforces the existing EOKS distinction between **reuse eligibility** and **correctness/authority**:

```text
candidate reusable artifact
          |
          v
 dependency/identity check
          |
          v
 provenance + validity evidence
          |
          v
 workload-specific acceptance
```

A cache hit is therefore not equivalent to a trustworthy result.

## New synthesis hypothesis

The strongest synthesis from this prior art is:

> **Derived context should be treated as materialized computation with explicit dependencies, provenance and validity conditions. When authoritative state changes, EOKS should prefer selective invalidation or incremental maintenance when the expected savings and reliability justify the additional complexity.**

This extends the existing validated-reusable-computation synthesis without introducing a new EOKS primitive.

It also sharpens context evolution:

```text
observe change
    |
    v
identify affected derived state
    |
    +--> keep valid state
    +--> invalidate stale state
    +--> incrementally update where possible
    +--> fully recompute when cheaper/safer
    |
    v
new context state
    |
    v
compile next reasoning context
```

## Research questions

1. **Dependency granularity:** how fine-grained can dependencies become before tracking overhead outweighs reuse?
2. **Semantic dependencies:** which dependencies matter for LLM-generated representations even when byte-level inputs are unchanged?
3. **Change sets:** when should multiple observations be coalesced before maintenance?
4. **Selective recomputation:** when does partial recomputation actually beat fresh synthesis once model-call and validation costs are included?
5. **Contradiction handling:** how should new evidence retract or downgrade derived context rather than merely append to it?
6. **Probabilistic outputs:** what evidence is needed to treat a reused probabilistic artifact as equivalent enough to a fresh computation?
7. **Maintenance drift:** can repeated incremental updates accumulate error or stale assumptions compared with periodic full recomputation?
8. **Evaluation:** should the primary metric be saved computation, total cost, task quality, stale-context rate, or a workload-specific combination?

## Relationship to existing EOKS work

This note is intentionally a **prior-art and synthesis refinement**, not a new runtime component.

It connects directly to:

- [`docs/context-evolution.md`](../../docs/context-evolution.md) — context as evolving state rather than transcript retention;
- [`research/validated-reusable-computation.md`](../validated-reusable-computation.md) — provenance, dependency-aware reuse, invalidation and incremental recomputation;
- [`research/prior-art/incremental-semantic-computation.md`](incremental-semantic-computation.md) — lower-level reusable intermediate computation;
- [`research/prior-art/llm-context-caching.md`](llm-context-caching.md) — current LLM caching evidence and the computational-vs-semantic reuse boundary;
- [`docs/synthesis-execution-graphs.md`](../../docs/synthesis-execution-graphs.md) — dependency topology and selective coordination;
- [`research/prior-art/knowledge-memory-context-synthesis.md`](knowledge-memory-context-synthesis.md) — context as a task-time projection of knowledge, experience and live state.

The main addition is the explicit bridge from **incremental computation/materialized-view maintenance** to **evolving derived context**, grounded both in classical incremental systems and in the current production practice of LLM prefix/KV/context caching and semantic result caching.

## Sources

### Incremental computation and derived-state maintenance

1. Umut A. Acar, Guy E. Blelloch and Robert Harper, *Adaptive Functional Programming* / self-adjusting computation research. CMU research bibliography: https://www.cs.cmu.edu/~rwh/papers.html
2. Umut A. Acar, Matthias Blume and Jacob Donham, *A Consistent Semantics of Self-Adjusting Computation*, 2011: https://arxiv.org/abs/1106.0478
3. Ashish Gupta, Inderpal Singh Mumick and V. S. Subrahmanian, *Maintaining views incrementally*, SIGMOD Record, 1993: https://doi.org/10.1145/170036.170066
4. Ashish Gupta and Inderpal Singh Mumick, *Maintenance of Materialized Views: Problems, Techniques, and Applications*, IEEE Data Engineering Bulletin, 1995: https://www.vldb.org/dblp/db/journals/debu/GuptaM95.html
5. Frank McSherry, Derek Murray, Rebecca Isaacs and Michael Isard, *Differential Dataflow*, CIDR 2013: https://www.microsoft.com/en-us/research/publication/differential-dataflow/
6. Derek G. Murray et al., *Incremental, iterative data processing with timely dataflow*, Communications of the ACM, 2016: https://research.google/pubs/incremental-iterative-data-processing-with-timely-dataflow/
7. Bazel, *Hermeticity*: https://bazel.build/concepts/hermeticity
8. Bazel, *Remote Caching*: https://docs.bazel.build/versions/0.26.0/remote-caching.html
9. Nix, *Store Derivation and Deriving Path*: https://nix.dev/manual/nix/2.35/store/derivation/
10. Nix, *Content-addressing derivation outputs*: https://nix.dev/manual/nix/2.35/store/derivation/outputs/content-address.html
17. Felice Bacciu et al., *DBSP: Automatic Incremental View Maintenance for Rich Query Languages*, 2023: https://arxiv.org/abs/2307.05551

The current LLM caching sources are maintained separately in [`llm-context-caching.md`](llm-context-caching.md), so that the production/tool-level evidence remains directly inspectable rather than being reduced to a few citations here.
