# LLM context caching and computational reuse

This note records the current production/tooling evidence behind the EOKS distinction between **reusing computation** and **reusing semantic results**. It complements [`incremental-context-maintenance.md`](incremental-context-maintenance.md), which connects this evidence to incremental maintenance and context evolution.

## Why this matters to EOKS

Modern LLM systems expose several different reuse layers that are easy to collapse into the word "cache":

```text
prompt/KV caching
    -> reuse model computation

application/result caching
    -> reuse a prior result

semantic caching
    -> retrieve a similar prior result and decide whether it is valid

EOKS derived context
    -> preserve a validated representation whose dependencies can evolve
```

The first layer is mostly computational. The later layers increasingly depend on semantic validity, provenance, freshness and scope. This gives EOKS a useful empirical anchor for its broader context lifecycle model.

## vLLM Automatic Prefix Caching

vLLM's Automatic Prefix Caching reuses KV-cache blocks when requests share the same prefix. The system hashes prefix blocks and can reuse blocks whose prefix context matches. [1]

The EOKS-relevant observation is not the specific KV implementation. It is that **cache identity expresses a dependency boundary**. If a stable part of context remains identical, its expensive model-side computation can remain reusable even while the dynamic tail changes.

This supports a layered context layout such as:

```text
stable instructions / policies
stable project knowledge
------------------------- reusable computation boundary
current task state
live evidence
execution state
```

It also reinforces the distinction between a computed representation and the knowledge represented by that computation: a KV cache is not itself a trustworthy memory artifact.

## Hugging Face KV cache

Hugging Face's Transformers documentation describes KV caching as storing key/value states so they do not need to be recomputed for tokens that have already been processed. It documents dynamic, static, offloaded and quantized cache strategies. [2]

This is useful background for EOKS because it makes the computational/semantic distinction explicit: **cached model state is an execution optimization, not persistent knowledge**.

## Anthropic Prompt Caching

Anthropic's prompt-caching documentation exposes explicit cache breakpoints and cache lifetimes, allowing applications to mark reusable portions of prompts and choose an appropriate retention period. [3]

For EOKS this is evidence that context assembly itself affects reuse. Stable material should be grouped so that volatile changes do not unnecessarily invalidate expensive computation.

The important abstraction is therefore not "prompt cache" but:

```text
context compiler
    |
    +--> stable representation
    |       |
    |       +--> reusable computation
    |
    +--> volatile representation
            |
            +--> current execution
```

## Google Gemini Context Caching

Gemini's context-caching documentation targets repeated large context, including long documents and codebases, and exposes cached-token usage. Its long-context guidance also discusses structuring repeated context so that common prefixes can be reused. [4][5]

This is valuable EOKS evidence because it connects representation design to both execution cost and repeated-use economics. A derived representation can therefore have two kinds of value:

1. **semantic value** — it makes future reasoning more effective;
2. **computational value** — its stable representation makes future execution cheaper.

EOKS should keep those values conceptually separate even when one representation provides both.

## OpenAI prompt caching

OpenAI's prompt-caching guidance similarly treats exact prefix stability as important for cache reuse. Current guidance also calls out keeping request-level reasoning configuration stable when changing reasoning effort, because changes to the cached prefix can affect reuse. [6]

Across vLLM, Anthropic, Gemini and OpenAI, the same production pattern appears: **context layout is part of the computational reuse strategy**.

This strengthens the EOKS context-compiler hypothesis: deciding what belongs in stable versus volatile context is not only an information-selection problem; it is also an execution-cost problem.

## Redis semantic caching

Redis's semantic-caching documentation describes storing prompts, embeddings, responses and metadata, then using semantic similarity to find candidate responses. Metadata can encode application-specific constraints such as model/version, tenant and locale. [7]

This is qualitatively different from prefix/KV caching:

```text
prefix/KV cache
    -> exact computational identity
    -> reuse computation

semantic cache
    -> approximate semantic similarity
    -> candidate result
    -> validity decision required
```

For EOKS, the second path is particularly important. Similarity should be treated as **retrieval of a candidate**, not evidence that the candidate is safe to reuse. Dependency checks, freshness, authority, policy/scope and task requirements should determine eligibility.

This is consistent with the EOKS principle:

> **Reuse computation aggressively; reuse semantic results conservatively.**

## Cache boundaries, TTLs and context evolution

These systems also clarify the relationship between caching and context evolution:

| Mechanism | What persists | Main validity signal | Typical failure mode |
| --- | --- | --- | --- |
| KV/prefix cache | computed model state | exact prefix/cache identity | lost reuse |
| prompt/context cache | reusable input context | cache boundary + lifetime | lost reuse / extra cost |
| semantic result cache | prior response | similarity + metadata/freshness | stale or incorrect result |
| EOKS derived context | synthesized representation | dependencies + provenance + validation | stale/contradictory context |

TTL is therefore only one invalidation mechanism. It is useful for execution caches, but insufficient as the general validity model for derived context. EOKS needs to reason about **why** an artifact remains valid, not only **how long** it has existed.

## Dependency identity is the deeper commonality

The systems above suggest a hierarchy:

```text
exact identity
    |
    +--> KV/prefix computation reuse

versioned / metadata identity
    |
    +--> application cache reuse

semantic similarity + hard constraints
    |
    +--> semantic result reuse

explicit dependency graph + provenance + validation
    |
    +--> EOKS derived-context maintenance
```

The last layer is the research frontier relevant to EOKS. Existing caching systems demonstrate pieces of the mechanism, but generally do not solve the full problem of maintaining a useful semantic representation as its underlying evidence changes.

## Security and scope are part of cache validity

Semantic and shared caches also make a less obvious point relevant to EOKS: cache identity can be a **security boundary**. A result valid for one tenant, permission scope, user or policy context must not become reusable merely because its semantic content is similar to another request.

Therefore dependency metadata for EOKS-derived context should be able to express scope and authority, not just source versions.

## Implications for EOKS

The production evidence suggests several concrete hypotheses for the broader EOKS synthesis:

1. **Context should have computational structure.** Stable and volatile portions should be distinguishable when doing so improves reuse.
2. **Computed state is not knowledge.** KV/prefix caches demonstrate reusable execution state, not durable semantic memory.
3. **Similarity is not validity.** Semantic caches make candidate retrieval cheap, but correctness still requires explicit constraints.
4. **TTL is not dependency tracking.** Time-based expiration is useful but cannot express all reasons a derived context artifact becomes stale.
5. **Cache identity is an early form of provenance.** The more semantic the artifact, the richer its dependency identity needs to become.
6. **Cache locality is a context-compilation concern.** Ordering and partitioning context can affect the economics of execution.
7. **The natural next layer is maintenance.** Once derived context has explicit dependencies, EOKS can ask whether changes require invalidation, incremental update, or full recomputation.

This should be read together with [`incremental-context-maintenance.md`](incremental-context-maintenance.md): current LLM caching provides the production evidence for computational reuse, while incremental-computation research provides the stronger model for maintaining derived state as dependencies change.

## Sources

1. vLLM, *Automatic Prefix Caching*: https://docs.vllm.ai/en/latest/features/automatic_prefix_caching.html
2. Hugging Face Transformers, *Caching*: https://huggingface.co/docs/transformers/en/kv_cache
3. Anthropic, *Prompt caching*: https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching
4. Google Gemini API, *Context caching*: https://ai.google.dev/gemini-api/docs/caching
5. Google Gemini API, *Long context*: https://ai.google.dev/gemini-api/docs/long-context
6. OpenAI, *Prompt caching*: https://platform.openai.com/docs/guides/prompt-caching
7. Redis, *Semantic caching*: https://redis.io/docs/latest/develop/interact/search-and-query/advanced-concepts/vectors/semantic-caching/
