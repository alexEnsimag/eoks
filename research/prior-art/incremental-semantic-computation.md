# Incremental semantic computation and reusable computed artifacts

This note extends EOKS's existing context, artifact, provenance and reasoning concepts with research on **reusing derived computation** rather than recomputing a workload from raw inputs. It does not introduce a mandatory runtime component.

## Why this matters to EOKS

Context caching answers a narrow question: **is the information needed for this reasoning step already available?** A broader systems question is:

> **Has this computation already been performed, and is its result still valid for the current inputs and dependencies?**

For example, an EOKS workload might derive:

```text
source revision
    -> repository representation
    -> API graph
    -> request flow
    -> authentication flow
    -> checkout flow
    -> security analysis
```

A later task should not necessarily repeat every derivation. If the relevant inputs and dependencies have not changed, a previously verified derived artifact can be reused. If only part of the dependency graph changed, only affected computations need to be reconsidered.

This is better understood as **incremental semantic computation** than as ordinary response caching.

## Research foundations

### Memoization and dynamic dependency tracking

Classical memoization reuses a function result for the same inputs. More powerful incremental systems additionally track which computations depend on which inputs so that changes can propagate selectively.

### Self-adjusting computation

Acar et al.'s work on self-adjusting computation makes dynamic dependencies and change propagation first-class. A computation records dependencies while running; after an input change, the system reuses unaffected work and propagates changes through the dependency structure instead of restarting the entire computation.

- Umut A. Acar, Guy E. Blelloch, Robert Harper, *Adaptive Functional Programming* (2002): https://www.cs.cmu.edu/~guyb/papers/acar-thesis.pdf
- Umut A. Acar, Guy E. Blelloch, Robert Harper, *Adaptive Functional Programming* / self-adjusting computation line of work: https://www.cs.cmu.edu/~guyb/papers/

The key EOKS lesson is the combination of **memoization + dependency graph + change propagation**, not any particular programming-language implementation.

### Adapton

Adapton applies demand-driven incremental computation to programs with dynamic dependencies. It separates cached computation from the demand for results and can reuse unaffected subcomputations while allowing the dependency structure to evolve.

- Hammer et al., *Adapton: Composable, Demand-Driven Incremental Computation* (2014): https://matthewhammer.org/adapton/

This is especially relevant to EOKS because an agent need not eagerly refresh every stale derived artifact. A potentially stale computation can be revisited when a workload actually demands it.

### Differential Dataflow

McSherry, Murray, Isaacs and Isard's Differential Dataflow maintains incremental results over changing and iterative data. Rather than treating each new input as a reason to rerun the whole dataflow, it propagates differences through operators and supports nested iteration.

- Frank McSherry, Derek G. Murray, Rebecca Isaacs, Derek G. Murray, *Differential Dataflow* (2013): https://www.microsoft.com/en-us/research/publication/differential-dataflow/
- Frank McSherry et al., *Differential Dataflow* (2013), arXiv: https://arxiv.org/abs/1309.7685

For EOKS, the useful principle is **retain prior computation and update only the affected result**, including when computations themselves iterate.

## Build systems and materialized computation

Build systems provide a practical engineering precedent. A build step transforms inputs into an artifact while recording enough dependency information to decide whether that artifact can be reused. Content-addressed caches and dependency-aware invalidation make this more precise than a simple key/value cache.

This is the closest established systems analogy to the proposed EOKS pattern:

```text
inputs + transformation + dependencies
                |
                v
             artifact
                |
             cache
                |
     valid? ----+---- invalidated?
       |                    |
     reuse             recompute/update
```

Database **materialized views** provide another established analogy: an expensive derived relation can be persisted and incrementally maintained when base data changes. The important distinction is that EOKS derived artifacts may be semantic and probabilistic rather than exact database relations, so freshness and validity need explicit evidence.

## LLM-specific evidence

Recent LLM systems research explores increasingly higher levels of reuse:

### KV and intermediate-state reuse

LLM serving systems cache inference intermediates such as KV states to avoid repeating model computation. Work on partial KV recomputation and semantic-aware KV reuse demonstrates that reuse can cross request boundaries when contexts are sufficiently related.

- Kwon et al., *KVPR: Efficient LLM Inference with KV Cache Reuse* (2025): https://aclanthology.org/2025.findings-acl.997/
- *AdapShot: Adaptive KV Cache Reuse for Efficient LLM Inference* (2026): https://aclanthology.org/2026.acl-long.1990/
- *SpecCache: Efficient LLM Inference via KV Cache Reuse for RAG* (2026): https://aclanthology.org/2026.acl-long.859/

These systems are lower-level than EOKS: they primarily reuse model execution state, not durable semantic knowledge. They nevertheless establish that **intermediate computation is itself a reusable resource**.

### Intermediate semantic artifacts

SemanticALLI explores caching structured intermediate results in LLM pipelines rather than treating the final response as the only cacheable output. Its experiments report substantially higher reuse than monolithic response caching in the studied workloads.

- *SemanticALLI: Semantic-Aware Caching for LLM Applications* (2026): https://arxiv.org/abs/2601.16286

MiniCache similarly explores reusable, parameterized computation for program-of-thought workloads rather than requiring exact query/result matches.

- *MiniCache: ...* (2026): https://arxiv.org/abs/2607.20507

These works are particularly relevant because they move the cache boundary from **final answer** toward **intermediate computation**.

### Reusable reasoning trajectories

Thought-Action Graphs represent successful reasoning as reusable structured operators/trajectories. This is not the same as incremental computation, but it supports the related idea that a completed reasoning process can become a reusable computational asset rather than disappearing with the session.

- *Thought-Action Graph: ...* (2026): https://aclanthology.org/2026.findings-acl.1572/

## The EOKS distinction

The research suggests a useful hierarchy:

```text
                    computation reuse
                           |
          +----------------+----------------+
          |                |                |
     model state      reasoning state   knowledge state
          |                |                |
       KV cache       intermediate IR   computed artifact
          |                |                |
      transient        task/session       durable
```

These layers should not be conflated.

- **Context cache** stores or resolves information needed for a reasoning step.
- **Inference cache** stores model execution state such as KV representations.
- **Reasoning cache** may preserve intermediate task computation or reusable trajectories.
- **Computed artifact** preserves a derived semantic representation together with its provenance and dependencies.

The last category is the most directly aligned with EOKS.

## Proposed EOKS representation

A durable computed artifact can remain an ordinary EOKS artifact rather than becoming a new top-level knowledge primitive:

```text
ComputedArtifact
├── input evidence / source revisions
├── dependency set
├── transformation / computation method
├── representation + result
├── provenance
├── verification evidence
├── validity / freshness
└── invalidation conditions
```

The computation method may be deterministic code, a static analyzer, a graph transformation, an LLM-assisted derivation, or a workflow of several steps. The artifact should record enough provenance to distinguish **reproducible derived evidence** from an unsupported model assertion.

## Demand-driven reuse

Adapton suggests an important control policy for EOKS: staleness need not imply immediate recomputation.

```text
source changes
     |
mark affected artifacts potentially stale
     |
     +---- no demand ----> defer
     |
     +---- demand -------> validate dependencies
                              |
                       reusable / recompute
```

This avoids turning incremental computation into a new source of background work and aligns with EOKS's principle of using the cheapest provider that can reliably answer the current requirement.

## Invalidation is the hard part

A semantic cache is only useful if EOKS can answer:

1. What inputs and dependencies produced this artifact?
2. Which changes can invalidate it?
3. Can the artifact be reused exactly, partially, or only as a starting point?
4. What verification evidence establishes that it remains trustworthy?
5. Does the current model/tool/compiler version affect validity?

For deterministic transformations, content hashes and dependency graphs can provide strong invalidation semantics. For LLM-derived results, model version, prompt/strategy, context, evaluator, source scope and stochasticity may also matter.

Therefore EOKS should avoid pretending that semantic equivalence is exact when it is not. A cached artifact can instead have an explicit validity class, for example:

```text
exactly reusable
conditionally reusable
stale but useful as a hypothesis
invalid
```

The labels are illustrative; policy should define the actual acceptance semantics.

## Relationship to context engineering

This concept extends the existing context-cache model rather than replacing it:

```text
persistent knowledge/evidence
          |
   computed artifacts
          |
   working-set selection
          |
   context compilation
          |
       reasoning
```

A computed artifact can reduce future **context acquisition** as well as future computation. Instead of rediscovering an API flow from source files, the context compiler may select a verified API-flow artifact and then drill into its authoritative evidence when necessary.

This creates a useful distinction:

> **Context caching reuses what was made available. Incremental semantic computation reuses what was derived.**

## Evaluation hypothesis

The goal should not be maximum cache-hit rate. Evaluate end-to-end outcomes:

```text
recompute baseline
      vs
incremental reuse
      |
      +--> task quality
      +--> evidence coverage
      +--> stale-artifact errors
      +--> invalidation precision
      +--> recomputation avoided
      +--> latency
      +--> cost
      +--> verification effort
```

A cache hit that causes a stale or incorrect engineering decision is a failure even if token and latency metrics look excellent.

## What this research does not establish

- LLM intermediate-state reuse is not evidence that arbitrary hidden states are safely portable across models, versions or contexts.
- Semantic caching does not by itself solve invalidation or provenance.
- Incremental computation techniques for deterministic programs do not automatically transfer to probabilistic LLM transformations.
- A reusable reasoning trajectory is not necessarily a reusable semantic fact.
- Higher cache hit rate is not equivalent to better engineering outcomes.

The durable EOKS research question is therefore:

> **Can provenance-aware, dependency-sensitive reuse of derived semantic computation reduce repeated discovery and reasoning work while preserving evidence quality and correctness?**
