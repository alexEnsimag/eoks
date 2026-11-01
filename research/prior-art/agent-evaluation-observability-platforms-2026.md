# Agent evaluation and observability platforms (2026)

**Reviewed:** 2026-11-01  
**Status:** research / provider comparison, not an architectural decision

This note compares external tooling that can help EOKS observe agent executions, evaluate coding-agent behavior, and turn real failures into reproducible regression cases. It is deliberately a provider landscape, not a proposal for a new EOKS subsystem.

## Executive synthesis

The tools fall into three overlapping but distinct roles:

1. **Execution observability** records what happened: sessions, model calls, tool calls, subagents, timing, tokens and cost.
2. **Task and trajectory evaluation** tests whether an agent achieved a task and whether its path was sound, efficient and safe.
3. **Production feedback / dataset management** turns observed executions into scored examples, datasets and future regression tests.

No single product should be assumed to cover all three well. In particular, a trace UI is not a benchmark, and a benchmark runner is not necessarily a useful historical trace store.

### Shortlist for EOKS

| Candidate | Primary role | Existing Claude Code path | Deployment / operating shape | EOKS fit |
|---|---|---|---|---|
| **Opik** | Observability, experiments and evaluation | Dedicated Claude Code plugin; native OTel path is a separate beta option | Open-source platform; cloud or self-hosted | Strong first tracing experiment |
| **Langfuse** | Observability, datasets and evaluation | Dedicated Claude Code hooks/plugin | Cloud or self-hosted; production self-hosting has several storage/worker dependencies | Strong tracing alternative |
| **Inspect AI + Inspect SWE** | Task/agent evaluation | Runs actual Claude Code CLI, Codex CLI and Gemini CLI in a sandbox through the separate Inspect SWE package | Open-source, local evaluation framework | Strongest fit for reproducible CLI-agent benchmarks |
| **Promptfoo** | Prompt/agent evaluation and red teaming | Claude Agent SDK provider is documented; do not assume this is identical to driving the standalone Claude Code CLI | Local-first evaluation CLI; optional hosted workflow/UI | Strong for regression assertions and security testing |
| **DeepEval** | Programmatic evaluation, including trajectories | Instrumented application/agent traces; no direct standalone Claude Code CLI integration established in this review | Python / pytest-oriented, local-first core | Useful for code-owned eval suites |
| **Braintrust** | Tracing, scoring, datasets and production feedback | Broad SDK / OTel integrations; no dedicated Claude Code CLI capture path established here | Primarily managed platform | Interesting trace-to-eval workflow |
| **MLflow GenAI** | Trace storage and evaluation | SDK / OTel instrumentation; no turnkey Claude Code CLI capture path established here | Open-source, self-hostable, broader ML lifecycle platform | Useful if already using MLflow |
| **Arize Phoenix** | OTel-based tracing and evaluation | Generic OTel / OpenInference instrumentation; no dedicated CLI hook verified here | Open-source, local SQLite by default; PostgreSQL for production | Lightweight OTel-oriented alternative |

Fit is an EOKS-specific assessment, not a general product ranking. “Not established” means this review did not verify a first-party direct integration; it does not mean a custom adapter is impossible.

## 1. Execution observability

### Opik

Opik currently documents a dedicated Claude Code plugin that captures turns, tool calls and subagent spans, with session grouping and token/cost information. It also provides an MCP server so a coding agent can inspect traces, score outputs and run evaluations. The plugin and native Claude Code OpenTelemetry export are distinct paths: the plugin is the documented default; the OTel path depends on Claude Code's enhanced telemetry beta and has different content/redaction behavior.

**Why it is interesting for EOKS**
- Directly instruments the existing Claude Code harness rather than requiring a move to an agent SDK.
- Exposes nested tool/subagent execution and a trace UI.
- Can be queried by the agent through MCP, making trace inspection part of a development workflow.
- Supports cloud and self-hosted deployments.

**Limitations and checks**
- The plugin captures sensitive prompt and tool content by default; decide explicitly what may leave the machine and who can access it.
- A trace is not a complete context manifest. EOKS still needs to record which instructions, skills, hooks, policies and knowledge/context artifacts were selected, with versions and provenance.
- Verify that plugin instrumentation is fail-open and does not materially change agent behavior or latency.
- Native OTel capture is not equivalent to the plugin: content is redacted by default and some detailed trace fields remain beta-gated.

**Assessment:** best first candidate for a low-friction, real Claude Code session-tracing experiment.

Sources: [Claude Code integration](https://www.comet.com/docs/opik/integrations/claude-code), [Opik MCP server](https://www.comet.com/docs/opik/mcp-server).

### Langfuse

Langfuse has a dedicated Claude Code observability plugin based on Claude Code hooks. Its integration reads the session transcript and emits turns, generations, tool calls, inputs/outputs, timing and session grouping. It is opt-in per project.

**Why it is interesting for EOKS**
- Direct integration with the existing CLI harness.
- Open-source and self-hostable.
- Covers tracing, scoring, datasets and evaluation rather than tracing alone.

**Limitations and checks**
- The integration documentation explicitly notes that assembled context files such as \`CLAUDE.md\` and loaded skills are not captured as the effective context. Skill invocations may be visible as tool calls, but that is not the same as knowing the complete context presented to the model.
- Hook-based capture is telemetry, not enforcement: developers can disable local hooks.
- The documented hook currently relies on Langfuse SDK 4.x internals for historical timestamps. Pinning the SDK and validating upgrades is part of operating this integration.
- Self-hosting is materially heavier than a single local service: the documented architecture includes web and worker containers, PostgreSQL, ClickHouse, Redis/Valkey and blob storage.

**Assessment:** credible alternative to Opik. Compare both against the same sessions; do not run both permanently unless the comparison demonstrates distinct value.

Sources: [Claude Code integration](https://langfuse.com/integrations/developer-tools/claude-code), [coding-agent tracing limitations](https://langfuse.com/resources/engineering/coding-agent-tracing), [self-hosting](https://langfuse.com/self-hosting).

### OpenTelemetry as the interoperability boundary

OTel is useful as a transport and common telemetry vocabulary, not as EOKS's semantic model. It can make instrumentation portable across trace backends and agent runtimes, but it cannot recover information that the harness never emits.

For Claude Code specifically, distinguish:
- native OTel emitted by Claude Code;
- a hook/plugin that reads the local session transcript;
- EOKS-owned events that describe context selection, policy, workflow state and outcomes.

These paths have different coverage, privacy and reliability properties. An OTel-compatible backend should not be described as having complete Claude Code visibility merely because it accepts spans.

Source: [OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/).

## 2. Task and trajectory evaluation

### Inspect AI + Inspect SWE

Inspect is an open-source evaluation framework built around datasets, solvers/agents, tools and scorers. Its separate Inspect SWE package provides integrations for real coding agents including Claude Code CLI, Codex CLI and Gemini CLI, running them inside a sandbox.

**Why it is especially relevant**
- Evaluates the actual CLI agent rather than requiring EOKS to rewrite its workflow around an SDK.
- Provides a task/dataset/scorer structure suitable for repeatable repository tasks.
- Supports sandboxed execution, limits, checkpointing and intervention.
- Makes it practical to evaluate task completion and inspect the execution log.

**Limitations**
- It is an evaluation harness, not a general-purpose always-on observability backend.
- The coding-agent integration is a separate package and should be assessed for version compatibility and sandbox requirements.
- A model-graded scorer is still an evaluator whose validity must be checked; use deterministic repository checks wherever possible.

**Assessment:** strongest candidate in this shortlist for a controlled benchmark of EOKS workflows that currently run through Claude Code CLI.

Sources: [Inspect overview](https://inspect.aisi.org.uk/), [coding-agent tutorial](https://inspect.aisi.org.uk/tutorial.html), [agent evaluation capabilities](https://inspect.aisi.org.uk/agents.html).

### Promptfoo

Promptfoo is an evaluation and red-team framework. Its coding-agent guide documents providers including the Claude Agent SDK, Codex SDK/app-server and OpenCode SDK. It supports structured assertions, repeated runs, trace/metadata assertions, and cost/latency checks. Its coding-agent red-team collection covers repository prompt injection, terminal-output injection, secret access, sandbox boundaries, network egress and verifier sabotage.

**Why it is interesting**
- Strong evaluation ergonomics for task assertions and CI regression checks.
- Explicit support for trace-aware agent evaluation.
- Particularly useful for testing security boundaries and whether an agent weakens the tests or policies meant to verify its work.

**Important distinction**
The documented Claude integration is the Claude Agent SDK provider. That is not automatically the same as evaluating a developer's ordinary standalone Claude Code CLI session. Use Inspect SWE for the latter unless a Promptfoo provider/adapter is separately verified.

**Assessment:** use Promptfoo for agent regression and red-team experiments, especially security and verifier-integrity cases; do not select it as the primary Claude Code session recorder.

Sources: [Evaluate coding agents](https://www.promptfoo.dev/docs/guides/evaluate-coding-agents/), [red-team coding agents](https://www.promptfoo.dev/docs/red-team/coding-agents/).

### DeepEval

DeepEval provides Python/pytest-oriented LLM evaluation and supports trajectory-based evaluation over ordered agent steps, tool calls, handoffs and intermediate spans. Its trajectory metrics include task completion, step efficiency and plan adherence, with custom metrics and CI assertions.

**Why it is interesting**
- Makes evaluation feel like ordinary automated testing.
- Can score both complete trajectories and individual spans.
- Useful when EOKS owns a Python-based agent or can instrument an application-level execution.

**Limitations**
- It is not, based on this review, a turnkey recorder for standalone Claude Code CLI sessions.
- Trajectory evaluation depends on instrumentation that actually captures the steps being scored.
- LLM-based metrics require evaluator validation and should complement deterministic checks.

**Assessment:** a good alternative when the workload is already represented as a programmatic Python agent; lower priority for a CLI-first EOKS experiment.

Source: [Trajectory-based evaluation](https://deepeval.com/docs/evaluation-trajectory-based-llm-evals).

## 3. Production trace-to-evaluation platforms

### Braintrust

Braintrust emphasizes tracing prompts, completions, tool calls and spans, scoring live traces, and turning failures into evaluation datasets. This production-trace-to-regression-dataset loop is attractive for EOKS's governed learning direction.

**Assessment:** worth revisiting if production feedback and dataset curation become a priority. This review did not establish a first-party Claude Code CLI hook comparable to Opik or Langfuse, and its managed-platform orientation should be weighed against data-governance requirements.

Source: [Braintrust tracing](https://www.braintrust.dev/learn/tracing/v0).

### MLflow GenAI

MLflow's GenAI tooling supports trace search and evaluation of stored production traces using built-in or custom scorers. Scorers can inspect intermediate spans, including tool trajectories and routing, rather than only final outputs.

**Assessment:** technically capable and attractive where MLflow is already part of the environment. It is broader than EOKS's immediate need and does not remove the need to instrument the actual CLI harness.

Source: [Evaluating production traces](https://mlflow.org/docs/latest/genai/eval-monitor/running-evaluation/traces/).

### Arize Phoenix

Phoenix is an open-source observability platform centered on OpenTelemetry/OpenInference tracing and evaluation. Its self-hosted default can use local SQLite, with PostgreSQL available for production deployments.

**Assessment:** a useful OTel-oriented alternative when portability and local deployment matter. The direct Claude Code CLI capture gap remains: EOKS or another integration must emit suitable spans.

Sources: [Phoenix self-hosting](https://arize.com/docs/phoenix/self-hosting/deploying-phoenix), [Phoenix endpoints](https://arize.com/docs/phoenix/learn/resources/faqs/what-is-my-phoenix-endpoint).

## 4. What EOKS should own

The existing EOKS model already distinguishes:
- **observability** — records what happened;
- **evaluation** — assesses intermediate or final outcomes;
- **reliability evidence** — estimates how much a result should be trusted for a particular decision;
- **policy/control** — decides what should happen next;
- **learning** — extracts candidate improvements and governs their promotion.

External products should implement or support provider capabilities within those boundaries. They should not become the EOKS ontology, canonical run record or control plane.

### Minimum EOKS run/evaluation contract

A provider-neutral record should correlate, where available:

| Information | Why it matters |
|---|---|
| Task contract and dataset version | Defines what success means |
| Run/session/trace identifiers | Joins EOKS state to external traces |
| Repository revision and workspace/sandbox identity | Makes the run reproducible and isolates artifacts |
| Model, harness/runtime and version | Attributes behavior to the actual execution system |
| Configuration, skills, hooks and policy versions | Describes the intervention being tested |
| Context manifest and provenance | Records what EOKS selected; do not assume a trace backend captures it |
| Tool/subagent events and timestamps | Supports trajectory and efficiency evaluation |
| Changed files, artifacts and validation results | Grounds task outcomes in executable evidence |
| Evaluator/rubric version and scores | Makes evaluation reproducible and recalibratable |
| Tokens, cost, latency, retries and human intervention | Measures run economics, not just model tokens |

The record should link to raw traces rather than copy every transcript into canonical EOKS state. Preserve content only when needed and authorized; otherwise store identifiers, metadata, hashes and provenance.

### Data and trust boundaries

Tracing can expose prompts, source code, tool outputs, repository contents, environment details and secrets accidentally printed by commands. Before enabling any integration:
- choose local/self-hosted versus managed ingestion deliberately;
- define content capture, redaction, retention and access controls;
- use synthetic secrets in adversarial tests;
- keep credentials out of repository settings and traces;
- make instrumentation fail-open where possible, so telemetry outages do not break engineering work;
- never treat telemetry capture as a security enforcement boundary.

A hook can be disabled or bypassed. Use sandboxing, permissions and policy enforcement for security guarantees; use traces to observe and evaluate behavior.

## 5. Recommended EOKS experiment

Do not adopt multiple platforms or build a generic tracing service first. Run a small, controlled comparison.

### Experiment A — CLI observability

1. Use the same short, representative Claude Code sessions with a few tool calls and at least one subagent.
2. Compare Opik's plugin with Langfuse's plugin.
3. Check capture completeness: prompt/response, tool input/output, subagent nesting, session grouping, timing and token/cost metadata.
4. Separately record the EOKS context manifest, configuration and policy versions; verify that these are not falsely assumed to be present in the vendor trace.
5. Test instrumentation failure, opt-out, redaction, retention and local/self-hosted operation.

Select one trace provider only if it materially reduces integration effort while preserving the evidence EOKS needs.

### Experiment B — task and trajectory evaluation

1. Start with 20–30 representative repository tasks and explicit success contracts, following the existing [context-evaluation methodology](../context-evaluation.md).
2. Use Inspect SWE to run the actual Claude Code CLI in a disposable sandbox.
3. Use deterministic checks for tests, changed files, protected-file hashes, expected findings and policy invariants.
4. Add model-graded criteria only for semantic qualities that deterministic checks cannot establish; validate those graders against independently labelled examples.
5. Repeat stochastic tasks and report distributions, not only the best run.
6. Use Promptfoo for a separate coding-agent red-team pass, especially prompt injection, sandbox and verifier-integrity cases.

### Decision criteria

Choose based on demonstrated fit, not feature count:

- direct compatibility with the existing agent/harness;
- completeness and fidelity of execution evidence;
- ability to correlate traces with EOKS context/configuration/outcomes;
- dataset, scorer and regression ergonomics;
- deterministic checks and trajectory visibility;
- data residency, privacy, retention and access controls;
- self-hosting and operational burden;
- instrumentation overhead and failure behavior;
- exportability and avoidance of backend lock-in.

## 6. Recommendation

For the current EOKS direction:

1. **Opik** — first observability experiment for existing Claude Code sessions.
2. **Langfuse** — direct alternative; compare against Opik rather than deploying both by default.
3. **Inspect SWE** — first choice for repeatable evaluation of actual Claude Code CLI tasks.
4. **Promptfoo** — complementary coding-agent regression and red-team testing.
5. **DeepEval** — consider for Python-owned agents and pytest-native trajectory tests.
6. **Braintrust / MLflow / Phoenix** — revisit when production feedback, existing infrastructure or OTel portability makes their particular trade-offs valuable.

The key architectural conclusion is not “EOKS should use Opik” or “EOKS needs a new evaluation subsystem.” It is:

> **EOKS should own the task contract, context/configuration provenance, outcome semantics, evaluation policy and governed learning. External tools can provide trace capture, benchmark execution, scorers and dataset interfaces behind replaceable provider boundaries.**

This is consistent with the existing [Evaluation](../../docs/evaluation.md), [Context evaluation](../context-evaluation.md) and [LLM observability and reliability](../llm-observability-and-reliability.md) documents. The next step is an empirical integration experiment, not an architectural promotion.
