# Context evolution prior art

This note records prior art relevant to the EOKS hypothesis that context should evolve as the work changes rather than preserving an ever-growing session transcript.

## Synthesis

The research landscape increasingly treats agent memory as a managed lifecycle rather than a passive store. A 2026 survey frames memory as a **write–manage–read** loop coupled to perception and action and identifies continual consolidation, causally grounded retrieval, trustworthy reflection and learned forgetting as open problems.

- Pengfei Du, *Memory for Autonomous LLM Agents: Mechanisms, Evaluation, and Emerging Frontiers* (2026): https://arxiv.org/abs/2603.07670

MemoryOS provides an explicit operating-system analogy: hierarchical short-, mid- and long-term memory with Storage, Updating, Retrieval and Generation modules. Its dynamic updates between levels support the EOKS hypothesis that persistence should have different horizons rather than being a single transcript.

- Jiazheng Kang et al., *Memory OS of AI Agent* (EMNLP 2025): https://aclanthology.org/2025.emnlp-main.1318/

Reflective Memory Management (RMM) is especially close to the evolution hypothesis. It separates prospective reflection—deciding what to retain from interactions across utterance, turn and session granularities—from retrospective reflection, which revises retrieval behavior using evidence from later interactions. This supports the EOKS distinction between forward retention and later correction.

- Zhen Tan et al., *In Prospect and Retrospect: Reflective Memory Management for Long-term Personalized Dialogue Agents* (ACL 2025): https://aclanthology.org/2025.acl-long.413/

LycheeMemory V2 provides recent evidence for **semantic-segment-level consolidation**. Rather than invoking memory construction after every interaction, it batches exchanges into semantically coherent segments and consolidates completed segments into typed records. This is directly relevant to EOKS's proposed semantic-boundary trigger: context should evolve on meaningful changes, not mechanically after every message.

- Dongfang Li et al., *LycheeMemory V2: Efficient Long-Term Memory for LLM Agents via Semantic Segment-Level Consolidation* (2026): https://arxiv.org/abs/2608.12990

## Implications for EOKS

These systems validate several pieces already present in EOKS:

1. memory has a lifecycle rather than being an ever-growing transcript;
2. multiple temporal/resolution levels are useful;
3. reflection can update or reorganize retained information;
4. consolidation granularity affects both quality and cost;
5. forgetting and invalidation are first-class research problems.

EOKS adds a different architectural boundary. The object being evolved is not only durable memory. It is the **active workload state** from which task-specific context is compiled. The proposed separation is:

```text
Durable resources / memory / evidence
              |
              v
       Context state evolution
              |
              v
       Context compilation
              |
              v
          model run
              |
              v
       outcome / evaluation
              |
              +----> context-state update
              +----> memory / knowledge candidates
```

The important EOKS hypothesis is therefore:

> **Remember the work, not the transcript.**

The active state should retain current direction, decisions, constraints, open questions, high-value evidence and a causal spine explaining important transitions. Detailed historical evidence can remain dormant and be reacquired on demand.

## Open research questions

- How accurately can semantic boundaries be detected in software-engineering sessions?
- What information belongs in active state versus dormant history?
- Can causal-spine representations preserve decision continuity better than generic summaries?
- When should a context change trigger synthesis or consolidation?
- How should contradictory evidence alter active state?
- Can learned retention/forgetting improve end-to-end task outcomes without increasing stale-context errors?
- How should context evolution interact with task loadouts, model changes and workflow stage changes?
- What is the minimum state needed to reconstruct a high-quality context after compaction or session restart?

## Proposed experiment

Compare three strategies over long software-engineering tasks:

```text
A. full transcript
B. periodic generic summaries
C. evolving context state + causal spine + on-demand authoritative evidence
```

Measure not only token cost and retrieval metrics, but decision/rationale retention, stale-context rate, context churn, reconstruction quality, verification cost, task quality, reliability and completion outcomes.

This is prior-art-informed research, not a claim that EOKS should adopt any of the cited implementations or reproduce their ontologies.
