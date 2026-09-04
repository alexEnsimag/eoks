# Probably, Approximately, Shipped

## Why this matters to EOKS

This article argues that AI-assisted software systems accumulate instructions, skills, prompts, benchmarks and evaluation practices that can become less useful as the underlying model and environment change. Its concrete examples are model- and tool-specific, but the broader observation fits an existing EOKS concern:

> A mechanism that was useful under one set of dependencies and conditions cannot be assumed to remain useful after those dependencies or conditions change.

This should **not** become a new EOKS primitive by itself. It is evidence for the existing lifecycle ideas around context, memory, evaluation and model migration.

## Main observations

### 1. Useful instructions can become liabilities

The article describes "prompt debt": instructions accumulated to compensate for limitations of an earlier model can persist after the model changes. It cites Anthropic's observation that removing most of the Claude Code system prompt did not measurably reduce coding-evaluation performance for newer models, and a SWE-skills-bench study in which most evaluated skills produced no improvement while some increased token cost or degraded performance.

**EOKS interpretation:** information and policies should not be treated as permanently beneficial merely because they were once useful. Their value is workload- and environment-dependent and should be evaluated through outcomes.

### 2. Verification can be more durable than generation instructions

The article contrasts fragile instructions with strong external checks such as tests and deterministic validators. It uses bug reproduction and compiler-style verification as examples where the verifier constrains what counts as an acceptable result.

This reinforces the existing EOKS distinction between model-generated confidence and external evidence. It does **not** imply that every verifier is durable: the verifier itself can become stale, gameable or insufficient as capabilities and requirements change.

### 3. Evaluation itself has a lifecycle

The article connects several observations that are useful together:

- benchmarks can saturate;
- evaluation criteria can drift as evaluators encounter new outputs;
- solved evaluations may need to become regression suites;
- production edge cases can become new evaluation cases.

Recent benchmark-saturation research provides broader evidence: a systematic study of 60 LLM benchmarks found substantial saturation, with older benchmarks more likely to have lost discriminatory power. This supports treating evaluation suites as maintained artifacts rather than permanent ground truth.

**EOKS interpretation:** evaluation is part of the feedback loop, but evaluation mechanisms are also subject to change. A high score is evidence relative to a particular task set, criterion and environment—not a timeless measure of system quality.

### 4. Model changes are dependency changes

The article's examples of prompt behavior, unsupported mechanisms, benchmark movement and changing coding-agent performance all point to the same operational lesson: changing a model can change the behavior of surrounding mechanisms even when those mechanisms themselves are unchanged.

This fits the existing EOKS model-migration loop: compare candidate and production configurations on representative workloads, inspect regressions and interaction effects, then stage, promote or roll back.

### 5. Aggregate success can hide responsibility and failure

The article distinguishes generated output volume from delegated responsibility and notes that test-passing patches do not necessarily correspond to patches maintainers would accept. It also discusses changing evidence around developer productivity as AI tooling evolves.

**EOKS interpretation:** evaluation should remain workload- and outcome-oriented. A proxy such as generated lines, benchmark score or test pass rate should not silently become the definition of success.

## Simulation: environment change invalidates a useful mechanism

A simple EOKS simulation can express the common structure without depending on any particular product:

```text
initial environment
      |
      v
mechanism M is introduced
      |
      v
measured improvement
      |
      v
environment/dependency changes
      |
      v
re-run representative evaluation
      |
      +---- improvement preserved ----> continue
      |
      +---- neutral -------------------> simplify / retire
      |
      +---- regression ----------------> revise / invalidate
```

Possible instantiations include:

- a model change alters the value of previously accumulated context;
- a tool/API change makes an old instruction ineffective;
- a workload distribution changes and exposes an unmeasured failure mode;
- an evaluator becomes too easy to discriminate among stronger systems;
- a previously useful verifier no longer captures an important requirement.

The important abstraction is the transition from **assumed validity** to **observed validity under the current environment**.

## Relationship to existing EOKS concepts

This research does not require a new subsystem. It strengthens several existing ones:

| Existing EOKS concept | Contribution from this research |
|---|---|
| Context lifecycle | Context usefulness can change when its dependencies change. |
| Memory lifecycle | Promoted knowledge needs freshness, supersession and invalidation. |
| Evaluation | Evaluation suites and criteria require maintenance and regression use. |
| Model migration | A model upgrade can alter surrounding context, policy and verifier behavior. |
| Control loop | Observed outcomes should trigger revision rather than assuming permanent usefulness. |
| Computed/derived representations | Reuse should depend on whether the representation's relevant dependencies and validity conditions still hold. |

The last point is particularly relevant to the emerging idea of reusing computed representations: caching a derived result can save future computation only when the dependencies that make that result valid remain compatible with the new workload/environment.

## Evidence and related work

### Primary article

André Bergholz, **"Probably, Approximately, Shipped"**, AI Advances, September 2026. The article is the source for the concrete observations above, including prompt debt, skill overhead, verification, benchmark/evaluation lifecycle, and the distinction between generated output and responsibility.

### SWE-Skills-Bench

Han et al., **SWE-Skills-Bench** (arXiv:2603.15401, 2026). Evaluates 49 public skills over roughly 565 SWE task instances. The study reports that most skills did not improve pass rate, while a small subset produced meaningful gains and some degraded performance. It is useful evidence that reusable instructions/capabilities need workload- and version-specific evaluation.

### Catastrophic Remembering in Agentic Coding

**"Why Does CLAUDE.md Keep Growing? Catastrophic Remembering in Agentic Coding"** (arXiv:2608.11095, 2026). An empirical study of instruction lifetimes in coding repositories. It is particularly relevant to the persistence of instructions and the difficulty of removing obsolete ones. Treat as recent/preprint evidence rather than an established EOKS principle.

### The Verification Horizon

**"The Verification Horizon: No Silver Bullet for Coding Agent Rewards"** (arXiv:2606.26300, 2026). Argues that verification becomes increasingly important as agent capability increases and that verifiers must themselves evolve in response to stronger policies and changing failure modes. This complements the article's verifier discussion and prevents an overly simple "tests solve it" interpretation.

### Benchmark saturation

**"When AI Benchmarks Plateau: A Systematic Study of Benchmark Saturation"** (arXiv:2602.16763, 2026). Studies 60 LLM benchmarks and finds substantial saturation, with benchmark age associated with reduced discriminatory power. Useful evidence for an evaluation lifecycle.

### Criteria drift / EvalGen

**"Who Validates the Validators?" / EvalGen** (arXiv:2404.12272, 2024). Shows a feedback relationship between observed outputs and the criteria used to evaluate them. Relevant to the article's warning that evaluation criteria are not necessarily independent or permanent.

### Google bug reproduction

**"Dynamic Cogeneration of Bug Reproduction Test in Agentic Program Repair"** (FSE 2026). Demonstrates the value of generating a reproduction test alongside a repair for real bugs. Relevant to the distinction between a plausible generated fix and externally grounded evidence of correctness.

## What this does *not* establish

- It does not establish that prompts or persistent instructions are inherently bad.
- It does not establish that benchmarks are useless once they age.
- It does not establish that deterministic tests are sufficient for software correctness.
- It does not justify a universal freshness threshold or validity score.
- It does not justify making any particular tool, file convention or benchmark an EOKS architectural dependency.

The durable conclusion is narrower: **EOKS mechanisms should be evaluated in the environment in which they are currently used, and changes in dependencies or environment are reasons to reconsider previously validated assumptions.**
