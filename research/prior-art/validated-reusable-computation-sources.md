# Validated and reusable computation — source register

This source register preserves prior art that should remain associated with the `validated-reusable-computation.md` synthesis. It separates foundational research from recent evaluation/optimization tooling and practitioner evidence.

## Foundational research and standards

- Madaan et al., **Self-Refine: Iterative Refinement with Self-Feedback**, NeurIPS 2023 — https://arxiv.org/abs/2303.17651
- Shinn et al., **Reflexion: Language Agents with Verbal Reinforcement Learning**, NeurIPS 2023 — https://arxiv.org/abs/2303.11366
- Gou et al., **CRITIC: Large Language Models Can Self-Correct with Tool-Interactive Critiquing**, ICLR 2024 — https://arxiv.org/abs/2305.11738
- Findings of EMNLP 2024, **Multi-step Problem Solving Through a Verifier: An Empirical Analysis on Model-induced Process Supervision** — https://aclanthology.org/2024.findings-emnlp.429/
- Jia et al., **Do We Need to Verify Step by Step? Rethinking Process Supervision from a Theoretical Perspective**, ICML 2025 — https://proceedings.mlr.press/v267/jia25f.html
- Morgan, **The Refinement Calculus**, South African Computer Journal — https://ir.unisa.ac.za/bitstream/handle/10500/24169/1995_SACJ_13_Morgan.pdf
- **Towards Large Language Model Aided Program Refinement** — https://arxiv.org/abs/2406.18616
- W3C, **PROV-DM / PROV Primer** — https://www.w3.org/TR/prov-primer/
- Acar et al., work on **Self-Adjusting Computation** and dynamic dependence graphs — https://doi.org/10.1145/2076021.2048101
- Bazel documentation on **hermeticity and incremental builds** — https://bazel.build/versions/8.6.0/basics/hermeticity
- Reproducible Builds, **Definition** — https://reproducible-builds.org/docs/definition/
- **PROV-AGENT** — https://arxiv.org/abs/2508.02866

## Evaluation and optimization tooling

- Opik documentation — https://www.comet.com/docs/opik/
- Opik Agent Optimizer documentation — https://www.comet.com/docs/opik/development/optimization-runs/overview
- Opik agent evaluation best practices — https://www.comet.com/docs/opik/evaluation/evaluate_agents
- Opik MCP — https://github.com/comet-ml/opik-mcp
- GEPA — https://github.com/gepa-ai/gepa
- GEPA FAQ — https://github.com/gepa-ai/gepa/blob/main/docs/docs/guides/faq.md
- GEPA use cases — https://github.com/gepa-ai/gepa/blob/main/docs/docs/guides/use-cases.md

## Recent evidence / research leads

- **Optimize Cheap, Deploy Strong: Cost-Aware Cross-Tier Transfer for Evolutionary Optimization** — https://arxiv.org/abs/2608.10694
- **Rethinking Self-Evolving Agents: Do We Still Need Prescribed Optimization Pipelines?** — https://arxiv.org/abs/2608.09629
- **ESPO: evolutionary prompt optimization with diagnosis, diversification and stabilization** — https://arxiv.org/abs/2609.04197

## How this register is used

The source register is intentionally separate from the synthesis. A source can provide evidence for a pattern without implying that EOKS should adopt the source's implementation, API or terminology. In particular, Opik, GEPA and HRPO are evidence about the empirical execution → trace → evaluation → candidate-refinement loop; they are not EOKS primitives.
