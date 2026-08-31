# OS and computer-architecture lens

This note records a design lens for EOKS: established operating-system, computer-architecture and systems-optimization concepts can be translated into candidate EOKS interventions. The analogy is deliberately **generative, not normative**. A technique should enter the EOKS architecture only when workload evidence shows that the translated mechanism is useful.

## Why this lens matters

EOKS has increasingly used Kubernetes as a reference model for reconciliation and control loops. Operating systems and computer architecture provide a complementary lower-level lens:

- OS concepts explain how workloads obtain, share, isolate, schedule and maintain resources.
- Computer architecture explains locality, caching, working sets, hierarchy, movement and capacity pressure.
- Kubernetes/control-loop concepts explain how desired and observed workload state are continuously reconciled.

The useful synthesis is therefore not “EOKS is an OS” or “EOKS is Kubernetes”. It is:

> **OS/computer architecture provide resource-management mechanisms and optimization hypotheses; Kubernetes provides a reference model for continuously reconciling workload state.**

The resulting EOKS abstraction is a workload control loop operating over finite, heterogeneous reasoning and information resources.

## The three-layer interpretation

```text
                         EOKS
                           |
              +------------+------------+
              |                         |
        RESOURCE LAYER             CONTROL LAYER
              |                         |
      OS / architecture          reconciliation / feedback
              |                         |
      memory hierarchy             desired state
      working sets                 observed state
      locality                     control loops
      caching                      scheduling
      paging                       lifecycle
      I/O                          policy
      isolation                    events
              |                         |
              +------------+------------+
                           |
                       WORKLOAD
                           |
          execution + working set + desired outcome
```

The workload is the center. The resource layer answers “what is available and how can it be efficiently accessed?” The control layer answers “what should happen next, given desired state, observations, policy and evidence?”

This is compatible with the existing EOKS architecture, where reconciliation is already the organizing control abstraction and Resource/Provider/Representation/Loadout/Context remain distinct vocabulary. The OS lens should reduce ambiguity between those concepts rather than create another ontology layer.

## Context as managed cache

Context should be treated as a **resident, task-specific working representation**, not as the memory store itself.

```text
resource / evidence universe
          |
    eligibility + policy
          |
          v
   workload working set
          |
 context acquisition/compilation
          |
          v
      model context
```

The working set is semantic: it is the subset of eligible resources/evidence currently useful for making progress. Model context is one materialization of that working set.

This distinction explains why durable knowledge, representations, execution state and model context should remain separate. A single source can have multiple representations, and a working-set item can be materialized as exact source, a structural slice, a verified fact, a summary, or a pointer to authoritative evidence.

A 2026 position paper explicitly frames multi-agent memory as a computer-architecture problem, distinguishing shared/distributed memory and proposing an I/O/cache/memory hierarchy while identifying cache sharing, access control and memory consistency as open challenges. See: https://arxiv.org/abs/2603.10062

A separate 2026 paper applies demand paging, working-set theory, eviction and fault-driven retrieval to LLM context windows, reporting production and replay experiments and explicitly discussing context thrashing. See: https://arxiv.org/abs/2603.09023

A 2026 survey on model-native computing likewise maps cache reuse, context capacity, agent scheduling and permission control to classical computer-systems problems, while emphasizing that the analogy has boundaries. See: https://arxiv.org/abs/2606.00288

## Ten concepts and their EOKS fit

### 1. Working set — promote to core vocabulary

Operating systems use working-set theory to reason about the pages actively needed by a process. EOKS can define:

> **Working set: the workload's currently useful subset of eligible resources and evidence.**

The working set can change by task phase:

```text
understand -> architecture + invariants + relevant code
implement  -> changed symbols + tests + interfaces
verify     -> changed artifacts + failures + invariants + checks
```

This is stronger than treating the entire context window as the working state. It creates a state variable that a control loop can observe and adapt.

### 2. Memory hierarchy — promote the principle, not fixed tiers

Computer systems exploit heterogeneous layers with different latency, capacity, cost and persistence. EOKS resources also differ in freshness, authority, fidelity, retrieval cost and transformation cost.

The EOKS principle should therefore be:

> **Representations form a hierarchy of access cost, capacity, freshness, authority and fidelity; materialize the least expensive representation that is sufficient for the current need.**

Do not hard-code an EOKS L1/L2/L3 taxonomy. The hierarchy is workload- and provider-dependent.

### 3. Locality — promote to core research vocabulary

Classical temporal and spatial locality suggest EOKS-specific forms:

- **temporal locality** — recently useful evidence is likely to be reused;
- **structural locality** — related symbols/files/dependencies are likely to be accessed together;
- **semantic locality** — related invariants, decisions and evidence are likely to co-occur;
- **workflow locality** — the result of one step predicts evidence needed by the next.

Locality provides a principled basis for prefetching, clustering, pinning and retention decisions.

### 4. Paging / demand retrieval — promote as a candidate mechanism

Translate a page fault into a **context miss**:

```text
required evidence absent from working set
        |
     context miss
        |
   acquire evidence
        |
 update working set / context
```

A context miss is not necessarily an error. It is a signal that the current working set did not contain sufficient evidence for the next operation.

Repeated misses should feed the control loop and may cause promotion, prefetching, representation changes, workload decomposition or a budget change.

### 5. Cache replacement — research intervention family

Classical LRU/LFU/FIFO/CLOCK-style policies are candidate baselines, not EOKS architecture. EOKS replacement must consider semantic value as well as recency/frequency.

Candidate policies include:

- LRU;
- LFU;
- relevance-weighted;
- authority/freshness-weighted;
- dependency-aware;
- retrieval-cost-aware;
- hybrid/adaptive policies.

Test whether simple policies already perform well before introducing learned replacement.

### 6. Thrashing — promote as an observable condition

Context thrashing occurs when acquisition and eviction dominate useful reasoning:

```text
retrieve A -> evict B -> retrieve B -> evict C
          -> retrieve C -> evict A -> ...
```

Candidate signals include context churn, repeated retrievals, repeated re-expansion, acquisition/token ratio, low progress despite high acquisition and repeated eviction/retrieval cycles.

Possible control responses include enlarging the working set, pinning critical evidence, changing representation, changing retrieval policy, restructuring the task or switching execution modality.

Thrashing should be an **observable workload condition**, not a mandatory runtime primitive.

### 7. Virtual resource addressing — strengthen the Resource Model

Operating systems hide physical placement behind virtual addressing. EOKS should similarly allow a workload to request a logical evidence/resource requirement without knowing which provider or storage system will satisfy it.

```text
logical requirement
       |
 provider resolution
       |
 representation
       |
 evidence
       |
 context
```

Navigation/resolution itself may be cached. This is particularly relevant to the existing distinction between navigation and knowledge.

### 8. Protection/isolation — consolidate into Loadout and Policy

A workload should have a scoped resource namespace describing what is readable, writable, executable, derived, restricted or approval-gated.

This strengthens the existing Loadout model:

> **Loadout is a workload-scoped resource namespace and eligibility boundary, not merely a prompt package.**

Do not introduce “memory protection” as a separate EOKS layer.

### 9. Copy-on-write / shared state — design principle for multi-agent work

Multiple agents/workflow phases should be able to share authoritative evidence while keeping derived or tentative state local until validated.

```text
canonical resource
      |
  shared read
      |
 local derived state
      |
 candidate -> validate -> promote/update/invalidate
```

This aligns with EOKS's evidence-first knowledge lifecycle and avoids uncontrolled shared mutable memory. It is a useful implementation principle, not currently a runtime primitive.

### 10. Interrupts/events — consolidate into control loops

External or internal events can trigger reconciliation instead of requiring continuous polling:

- test failure;
- dependency/source change;
- invariant violation;
- new commit;
- resource becoming stale;
- human input;
- verification result;
- external system change.

The architectural consequence is event-driven reconciliation, not an “interrupt subsystem”.

## Optimization techniques worth importing

The analogy becomes useful when it generates concrete optimization hypotheses.

### Cache optimization

Candidate techniques:

- temporal/structural/semantic locality;
- prefetching;
- cache admission;
- eviction/replacement;
- pinning;
- clustering;
- adaptive replacement;
- compression/demotion.

EOKS-specific objective:

> maximize expected reasoning benefit while accounting for token, latency, retrieval, staleness and attention-interference costs.

A cache hit is not sufficient evidence of success. The primary metric remains useful, verified workload outcome.

### Working-set optimization

Estimate the amount and composition of information required by the current workload phase rather than maximizing context capacity.

Possible feedback:

```text
high miss rate       -> increase or change working set
high irrelevant load -> decrease/change working set
thrashing             -> change policy/representation/budget
high hit rate + poor progress -> working set may be semantically wrong
```

The last condition is critical: EOKS must optimize task progress, not cache statistics.

### Paging optimization

Candidate interventions:

- demand acquisition;
- prefetching;
- page/evidence clustering;
- representation compression;
- promotion/demotion between representations;
- fault-history-based pinning.

Prefetching should be evaluated against its false-positive acquisition cost because evidence acquisition is more expensive than a typical hardware cache prefetch.

### I/O optimization

Agent/tool interactions are a form of I/O. Import:

- batching;
- buffering;
- asynchronous acquisition;
- request coalescing;
- locality-aware ordering;
- caching;
- prefetching.

Batching can also improve reasoning coherence by reducing fragmented evidence delivery.

### Scheduling optimization

CPU scheduling suggests:

- priorities;
- fairness/anti-starvation;
- aging;
- shortest-job-first as a candidate heuristic;
- work stealing;
- load balancing;
- preemption where appropriate.

For EOKS, competing work includes reasoning, retrieval, testing, review, verification, memory maintenance and deterministic execution. The conductor should schedule the cheapest sufficient modality rather than spending a model call when deterministic execution can satisfy the requirement.

### Deterministic-first execution

This connects directly to the existing EOKS deterministic-execution model:

> **Do not replace a task merely because an LLM can do it. Replace probabilistic execution when the behavior is sufficiently understood, the deterministic capability is simpler to maintain, and the engineering outcome is better.**

The scheduler can therefore treat deterministic tools as sensors/actuators/capabilities rather than agent types.

### Distributed-memory techniques

For shared multi-agent resources, candidate mechanisms include:

- versioned state;
- explicit consistency boundaries;
- ownership/leases;
- invalidation;
- replication where useful;
- event-driven synchronization;
- copy-on-write derived state.

Do not assume strong consistency is universally desirable. EOKS should measure the cost of consistency against stale-state failures and workload requirements.

## What should be measured

Every translated systems technique should be treated as an intervention.

```text
known systems technique
        |
 translate to EOKS mechanism
        |
 define workload-specific intervention
        |
 controlled benchmark
        |
 outcome / cost / latency / reliability
        |
   +----+----+
   |         |
 transfer   reject/refine
```

At minimum record:

- task correctness/completeness;
- verification quality and regressions;
- model and tool tokens/calls;
- latency and total cost;
- context size and growth;
- working-set hit/miss/churn statistics;
- acquisition and prefetch work;
- repeated retrievals;
- stale/contradictory evidence;
- recovery and session resets;
- deterministic execution ratio and amortization where relevant;
- tail cost/latency, not only averages.

The existing EOKS methodology remains the authority for cross-model/repository/task evaluation.

## What not to import literally

Avoid turning analogies into mandatory architecture:

- no fixed EOKS L1/L2/L3 hierarchy;
- no “page” runtime primitive;
- no mandatory “page fault” terminology;
- no MMU-equivalent component;
- no requirement that a workload be a Unix-like process;
- no assumption that LRU or another classical replacement algorithm is optimal;
- no assumption that context cache and inference KV cache are the same thing;
- no assumption that multi-agent shared memory requires one consistency model.

The analogy generates hypotheses and reusable terminology; experiments decide which mechanisms belong in EOKS.

## Context cache versus inference cache

Keep two caches conceptually separate:

1. **Semantic/context cache** — reusable knowledge, evidence, representations and navigation resolutions managed by EOKS.
2. **Inference/KV cache** — model-serving state used to avoid recomputing attention representations.

EOKS may influence the second indirectly by keeping repeated context structures stable, but it should not collapse the two into one memory abstraction.

## Architectural synthesis

The combined OS/Kubernetes view is:

```text
                 desired workload state
                          |
                          v
                    +-----------+
                    | reconcile |
                    +-----+-----+
                          |
          +---------------+----------------+
          |               |                |
       resources       working set      execution
          |               |                |
          +---------------+----------------+
                          |
                       schedule
                          |
                    execute/observe
                          |
                    updated state
                          |
                     reconcile
```

OS/computer architecture tells EOKS how to think about resource locality, hierarchy, capacity, movement and isolation. Kubernetes/control loops tell EOKS how to continuously reconcile desired and observed workload state. Neither analogy is the architecture by itself.

## Research priority

The first candidate interventions should be:

1. working-set estimation;
2. locality-aware acquisition;
3. simple replacement baselines (LRU/LFU/relevance-aware);
4. demand retrieval/context misses;
5. prefetching;
6. evidence clustering/batching;
7. representation compression/demotion;
8. context-thrashing detection;
9. priority scheduling;
10. deterministic-first scheduling;
11. cache admission and pinning;
12. navigation-resolution caching;
13. async acquisition;
14. versioned shared state/copy-on-write;
15. adaptive or learned policies only after simpler baselines.

The point is not to implement all of these. The point is to turn decades of known systems optimizations into a structured EOKS hypothesis backlog and test which ones transfer to probabilistic reasoning workloads.

## References

- Yu et al., **Multi-Agent Memory from a Computer Architecture Perspective: Visions and Challenges Ahead** (2026): https://arxiv.org/abs/2603.10062
- Mason, **The Missing Memory Hierarchy: Demand Paging for LLM Context Windows** (2026): https://arxiv.org/abs/2603.09023
- Lin et al., **Model-Native Computing Architecture: Envisioning Future System Architecture Through the Lens of Computer Architecture** (2026): https://arxiv.org/abs/2606.00288
- INFERCEPT, **Efficient Intercept Support for Augmented LLM Inference** (2024): https://arxiv.org/abs/2402.01869
- Denning, **The Working Set Model for Program Behavior** (1968), foundational working-set theory.

The recent AI-specific papers are evidence that these mappings are independently emerging; they are not evidence that any particular EOKS mechanism is already validated. EOKS should preserve that distinction.