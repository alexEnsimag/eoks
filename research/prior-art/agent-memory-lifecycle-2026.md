# Agent memory lifecycle — 2025–2026 research synthesis

This note records research prompted by the July 2026 article “Memory in AI Agents: Why Your Agent Forgets Everything (And How to Fix It)” and extends it with primary research and implementation prior art.

## Executive synthesis

The useful progression is not simply short-term memory → long-term memory. Recent work increasingly treats memory as a lifecycle:

```text
experience / evidence
        ↓
     storage
        ↓
     reflection
        ↓
     synthesis / abstraction
        ↓
 candidate knowledge / capability
        ↓
 validation / promotion
        ↓
 retrieval / context construction
        ↓
 action / outcome
        ↓
 evaluation / revision / forgetting
        ↺
```

Luo et al. (Findings of ACL 2026) explicitly frame the evolution of LLM-agent memory as **Storage → Reflection → Experience**: trajectory preservation, trajectory refinement, and trajectory abstraction. They identify long-range consistency, dynamic environments and continual learning as drivers, and highlight proactive exploration and cross-trajectory abstraction as frontier mechanisms.

For EOKS, this supports treating memory as a governed lifecycle spanning evidence, experience, knowledge and reusable capability rather than as a database or a single semantic category.

## Primary research

### A-MEM: Agentic Memory for LLM Agents

Wujiang Xu, Zujie Liang, Kai Mei, Hang Gao, Juntao Tan, Yongfeng Zhang. NeurIPS 2025.

A-MEM dynamically organizes memories into interconnected knowledge networks. New memories are represented with structured attributes, linked to relevant prior memories, and can trigger evolution of existing representations. It is useful evidence that memory can be an adaptive relational structure rather than flat vector retrieval.

EOKS implication: relationships and derived representations are valuable, but a graph remains a representation mechanism rather than the ontology of all knowledge. EOKS should preserve provenance and authority boundaries when representations evolve.

Source: https://proceedings.neurips.cc/paper_files/paper/2025/hash/19909c36f51abc4856b4560aff3d36d6-Abstract-Conference.html

### From Storage to Experience: A Survey on the Evolution of LLM Agent Memory Mechanisms

Jinghao Luo et al. Findings of ACL 2026, pp. 41622–41652.

The survey provides the strongest terminology match for EOKS: Storage preserves trajectories; Reflection refines trajectories; Experience abstracts trajectories into reusable knowledge. It explicitly positions continual learning as the longer-term objective.

EOKS implication: reflection should not be equated with ordinary chain-of-thought or “thinking first”. In this context it is post-/inter-experience processing that refines evidence and supports abstraction. Synthesis is the EOKS operation that decides what can become reusable knowledge.

Source: https://aclanthology.org/2026.findings-acl.2069/

### Agentic Memory (AgeMem): Learning Unified Long-Term and Short-Term Memory Management

Yi Yu et al. ACL 2026, pp. 21457–21483.

AgeMem integrates short- and long-term memory management into the agent policy and exposes store, retrieve, update, summarize and discard as tool-like operations. This demonstrates that memory management itself can become an agent capability rather than a fixed external heuristic.

EOKS implication: even when an agent can request memory operations, the control plane can still govern resources, policy, provenance, validation and promotion. Agent-directed memory operations do not eliminate the need for external governance.

Source: https://aclanthology.org/2026.acl-long.981/

### Mem2ActBench: A Benchmark for Evaluating Long-Term Memory Utilization in Task-Oriented Autonomous Agents

Yiting Shen, Kun Li, Wei Zhou, Songlin Hu. ACL 2026, pp. 8173–8190.

Mem2ActBench argues that passive fact recall is insufficient: memory should be evaluated by whether it is correctly applied to tool-based action, including tool choice and parameter grounding. Their benchmark contains 2,029 sessions and 400 tool-use tasks; experiments over seven memory frameworks found current systems inadequate at active memory utilization.

EOKS implication: memory evaluation should be downstream of retrieval. The relevant chain is memory/knowledge → context → reasoning → action → validation → outcome.

Source: https://aclanthology.org/2026.acl-long.370/

### MemoryAgentBench: Evaluating Memory in LLM Agents

This benchmark work evaluates memory beyond simple retrieval, including accurate retrieval, test-time learning, long-range understanding and selective forgetting.

EOKS implication: forgetting, updating and adaptation should be first-class lifecycle operations, and memory quality should be evaluated against workload behavior rather than recall alone.

Source: https://arxiv.org/abs/2507.05257

### LongMemEval

LongMemEval evaluates long-term interactive memory capabilities including information extraction, multi-session reasoning, temporal reasoning, knowledge updates and abstention.

EOKS implication: persistent knowledge must carry temporal validity and update semantics. A memory system should be able to avoid confidently using stale or contradicted knowledge.

Source: https://arxiv.org/abs/2410.10813

## Tool / framework prior art

### MemGPT / Letta

MemGPT established the operating-system-inspired framing in which finite working context is managed against external memory. It is important prior art for memory hierarchies and context management.

EOKS implication: context capacity and external persistence are resource-management problems, but EOKS extends the model by governing heterogeneous resources, evidence, policies and evaluation rather than treating all external storage as memory.

Source: https://arxiv.org/abs/2310.08560

### Mem0

Mem0 is a production-oriented memory layer emphasizing extraction, update/consolidation and retrieval. Its research evaluates persistent memory against baseline approaches and explores graph-based memory.

EOKS implication: useful capability reference for memory extraction and consolidation; not an EOKS dependency or canonical architecture.

Source: https://arxiv.org/abs/2504.19413

### LangMem

LangMem provides memory-management primitives for LangGraph-based agents, including extracting, updating and searching memories, with support for hot-path and background memory formation.

EOKS implication: supports separating critical-path interaction from background consolidation and learning. EOKS should preserve the broader distinction between memory, evidence, Skills, project knowledge and derived representations.

Source: https://github.com/langchain-ai/langmem

### TencentDB Agent Memory

Tencent's Agent Memory work is useful prior art for multi-resolution memory, including conversation, atomic, scenario and core/profile levels, alongside reusable Skills, Wiki and CodeGraph resources.

EOKS implication: multiple resolutions and resource families are useful patterns, but should not be treated as a universal ontology. EOKS already models shared governance metadata—provenance, scope, freshness, ownership/access, version and validation—across semantically distinct resources.

Source: https://github.com/Tencent/Angel/tree/main/agent-memory (repository/project reference; see EOKS prior-art note for the version researched)

## Cognitive-architecture context

Classical cognitive architectures distinguish working, episodic, semantic and procedural forms of memory. Soar provides explicit episodic and semantic memory mechanisms; ACT-R distinguishes declarative and procedural memory. These are useful conceptual precedents, not evidence that an EOKS resource literally models human memory.

Soar semantic memory: https://soar.eecs.umich.edu/soar_manual/06_SemanticMemory/
Soar episodic memory: https://soar.eecs.umich.edu/soar_manual/07_EpisodicMemory/

## EOKS synthesis

The research supports five distinctions:

1. **Evidence** — what actually happened: conversations, code, tool outputs, traces, tests, reviews and outcomes.
2. **Experience** — structured records of episodes and trajectories, retaining what was attempted and what happened.
3. **Knowledge** — generalized facts, relationships, constraints and project understanding derived from evidence/experience.
4. **Capability** — reusable procedures, Skills, workflows and policies describing how work can be performed.
5. **Context** — a workload-specific projection compiled from eligible evidence, knowledge and capabilities.

Memory should not collapse these categories. A graph, vector index, document, summary or database is an implementation/representation choice.

### Proposed EOKS lifecycle

```text
observe
  → retain evidence / episode
  → retrieve
  → reflect
  → synthesize
  → validate
  → promote
  → compile context
  → execute
  → evaluate outcome
  → revise / invalidate / forget
```

Promotion is important: an agent observation or successful run should not silently become canonical knowledge or policy. Candidates need evidence, scope, provenance, validation and lifecycle state.

### Evaluation implication

Memory evaluation should cover at least:

- retrieval accuracy and relevance;
- temporal/update correctness;
- contradiction handling and abstention;
- selective forgetting/invalidation;
- cross-session and cross-trajectory generalization;
- use in actual task/action execution;
- context/token efficiency;
- downstream quality, reliability, cost and other workload outcomes.

## Relationship to existing EOKS work

This research reinforces existing EOKS decisions rather than replacing them:

- knowledge is not a graph;
- context is not the knowledge layer;
- memory is not one flat store;
- derived representations should retain provenance;
- learning should be controlled rather than silently rewriting canonical knowledge;
- validation and outcomes matter more than self-reported agent success;
- context should be compiled from eligible resources rather than dumping all history into the prompt.

The new contribution is a sharper lifecycle vocabulary: **evidence → experience → reflection → synthesis → candidate knowledge/capability → validation/promotion → context → action → outcome → revision**.

## Research status

This note is a synthesis of primary research and implementation/framework documentation available through September 2026. It is intentionally non-normative: individual systems use overlapping terminology and should not be treated as interchangeable evidence for the EOKS model.
