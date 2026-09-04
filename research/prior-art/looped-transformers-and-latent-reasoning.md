# Looped Transformers and latent reasoning

This note records research relevant to EOKS's existing **reasoning strategy / computation-loop** concepts. It does not introduce a new EOKS runtime primitive.

## Why this matters to EOKS

Reasoning can be represented at different levels. Explicit chain-of-thought represents intermediate computation as generated tokens; an agent loop represents it through successive actions and observations; latent/recurrent reasoning represents intermediate computation as successive hidden states inside the model.

The useful EOKS abstraction is therefore not "looped Transformer" but:

> **Iterative computation transforms a state until a sufficient stopping condition is reached. The state and loop may live inside or outside the model.**

This preserves the distinction between representations without creating a separate architectural layer for each implementation.

```text
             iterative computation
                     |
          state -> transform -> state
                     |
                  enough?
                 /       \
               no         yes
               |           |
             repeat      output
```

### Where the loop lives

| Loop location | Intermediate state | Example |
|---|---|---|
| Explicit generation | reasoning tokens | chain-of-thought |
| Agent/workflow | observations, artifacts, decisions | act -> observe -> evaluate -> retry |
| Latent model computation | hidden representations | recurrent/looped Transformer |
| Longer-lived control | durable workload state | reconciliation across runs |

These are complementary representations. EOKS should not imply that one is universally superior.

## Research evidence

### Universal Transformers (Dehghani et al., 2018)

Universal Transformers apply recurrent computation to Transformer representations and introduce dynamic per-position halting. This is foundational prior art for treating depth as a recurrent computation budget rather than a fixed stack of independently parameterized layers.

- Dehghani et al., *Universal Transformers* (2018): https://arxiv.org/abs/1807.03819

### Looped Transformers / latent reasoning

Fan, Svete and Lee's 2026 LOTUS work directly connects looped Transformers to latent chain-of-thought. It uses repeated shared computation over latent blocks and supervises latent positions using gold CoT-step tokens. The paper reports a bridge between latent and explicit reasoning at the 3B scale and substantial thought-phase latency reductions in its experiments.

- Fan, Svete & Lee, *Bridging the Gap Between Latent and Explicit Reasoning with Looped Transformers* (2026): https://arxiv.org/abs/2606.31779

This is especially relevant to EOKS because it demonstrates that **the representation of intermediate computation can change while the reasoning objective remains recognizable**.

### Latent Recurrent Transformer

Huang et al. explore a different recurrent pathway: a high-level hidden state from the previous token becomes recurrent memory for the next token. Their results show another way to add recurrence without simply inserting explicit pause/reasoning tokens or repeatedly unrolling the whole model.

- Huang et al., *Latent Recurrent Transformer: Architecture Exploration, Training Strategies, and Scaling Behavior* (2026): https://arxiv.org/abs/2605.18694

This is useful evidence that "latent computation" is broader than one looped-Transformer recipe.

### The Recurrent Transformer

Oncescu et al. study recurrent computation across Transformer layers and report parameter-matched improvements in their smaller-scale pretraining experiments, framing recurrence as a way to trade depth and width while preserving useful Transformer properties.

- Oncescu et al., *The Recurrent Transformer: Greater Effective Depth and Efficient Decoding* (2026): https://arxiv.org/abs/2604.21215

### Latent recurrent thoughts

Chen and Fu's 2026 work explores a small recurrent reasoner that repeatedly refines continuous latent thoughts while using a frozen LLM as decoder. This provides a useful counterpoint to architectures that put recurrence directly into the Transformer: iterative reasoning can also be factored into a separate latent computation component.

- Chen & Fu, *Latent Recurrent Thoughts: Recurrent Refinement of Proposed Latents for Reasoning with Frozen LLMs* (2026): https://arxiv.org/abs/2609.01117

### Compute-matched evaluation

Wang et al.'s SMELT work is particularly useful as an evaluation caution. It compares looped and unlooped MoE Transformers while matching FLOPs, parameter count and KV-cache budgets, and reports improvements under those matched constraints. This helps distinguish an architectural advantage from simply spending more compute.

- Wang et al., *SMELT: Scaling Laws for Compute-Matched MoE Looped Transformers* (2026): https://arxiv.org/abs/2609.01343

## EOKS implications

### 1. Reasoning is not synonymous with visible CoT

Intermediate computation may be represented explicitly, latently, or through external workflow state. EOKS should preserve these as representations/modalities rather than make "chain-of-thought" the definition of reasoning.

### 2. Computation budget can be an internal control variable

A reasoning step may have an adaptive depth or iteration count. The relevant control question is not simply "which model?" but also **how much computation should this step receive, and what evidence justifies stopping?**

### 3. Halting belongs with evaluation/control semantics

Universal Transformers already demonstrate dynamic halting. More recent latent-reasoning work makes the stopping/computation-allocation problem increasingly relevant. EOKS should treat halting as a control/evaluation concern, not as a new primitive.

Potential signals include convergence, evaluator evidence, confidence calibrated to outcomes, budget exhaustion, or policy-defined limits. None should be assumed sufficient without workload-specific evidence.

### 4. Monitorability is a trade-off, not a verdict

Latent reasoning can make intermediate computation less directly inspectable than explicit reasoning tokens. That is an important assurance consideration, but it does not imply that latent computation is inherently unsafe or that explicit CoT is a faithful account of internal computation. EOKS should evaluate **observable evidence and outcome assurance**, not equate a readable trace with complete introspection.

## Relationship to EOKS architecture

This research fits existing concepts:

```text
Reasoning strategy
       |
       +--> explicit token-based reasoning
       +--> latent/recurrent computation
       +--> external iterative workflow
       |
       v
   computation budget
       |
       v
 evaluation / stopping evidence
       |
       v
 conductor reconciliation
```

No new runtime primitive is required. A model's internal recurrence is a capability of a model resource; a latent representation is a representation; adaptive depth is a computation-allocation strategy; and stopping is part of evaluation/control.

## What this research does not establish

- It does not establish that looped Transformers universally outperform conventional Transformers.
- It does not establish that latent reasoning is universally better than explicit reasoning.
- Parameter savings do not imply free computation: additional recurrent passes still consume inference compute.
- A model's explicit reasoning trace should not be treated as a complete window into its internal computation.
- OpenAI-specific claims should be kept separate from the peer-reviewed/preprint evidence above; media reports about unreleased systems are not themselves research evidence for the architectural claims.

The practical EOKS research question is therefore narrower and more durable:

> **Can adaptive iterative computation—whether explicit, latent, or external—improve end-to-end engineering outcomes enough to justify its additional compute, latency, complexity and assurance cost?**
