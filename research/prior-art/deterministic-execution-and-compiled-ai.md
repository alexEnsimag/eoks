# Deterministic execution and compiled AI — prior art

This note records research relevant to EOKS's principle of minimizing unnecessary probabilistic execution.

## Research themes

### LLM + classical planning

LLM+P (2023) separates natural-language interpretation from formal planning: an LLM translates the request into a planning representation and a classical planner solves the resulting problem. This supports an architectural separation between semantic interpretation and deterministic/symbolic computation.

Reference: https://arxiv.org/abs/2304.11477

### Compiled AI

Compiled AI (2026) studies generating a validated executable artifact from an LLM-produced workflow and executing the artifact without repeated model calls. This is particularly relevant to the EOKS hypothesis that reasoning can sometimes be paid once and amortized over repeated deterministic execution.

Reference: https://arxiv.org/abs/2604.05150

### PlanCompiler

PlanCompiler (2026) explores structured plans, typed capability registries and compilation into executable procedures. The important EOKS-relevant idea is not the specific implementation but the boundary between model-generated intent/plan and validated execution semantics.

Reference: https://arxiv.org/abs/2604.13092

### Deterministic policy enforcement

Recent policy-as-code and agent-guard research explores enforcing policies through executable checks rather than relying exclusively on an LLM to remember and follow natural-language constraints. This reinforces the broader principle of moving enforceable properties into mechanisms with explicit semantics.

Reference example: https://aclanthology.org/2025.emnlp-industry.41/

## Why this matters for EOKS

These directions converge on a useful decomposition:

```text
ambiguous intent
      ↓
probabilistic interpretation / synthesis
      ↓
structured specification / plan
      ↓
validation / compilation
      ↓
deterministic execution
      ↓
objective verification
      ↓
replan only when observations invalidate assumptions
```

This should be treated as an empirical design hypothesis, not a universal rule. Deterministic execution is preferable only when it can satisfy the requirement without sacrificing correctness, safety, coverage or adaptability.

## Research gaps for EOKS

The most useful questions are workload-specific:

- How reliably can models produce specifications that are sufficiently precise for deterministic execution?
- How often can repeated agent procedures be compiled and safely reused?
- How quickly should compiled procedures be invalidated after state or policy changes?
- Does deterministic execution reduce cost and variance while maintaining task success?
- Does an escalation policy outperform fixed LLM-heavy agent loops?
- Which tasks benefit from deterministic execution, and which genuinely require continued reasoning?

EOKS should measure these questions rather than assume that either maximum agentic behavior or maximum determinism is optimal.
