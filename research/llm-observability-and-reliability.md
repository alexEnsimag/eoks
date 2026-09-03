# LLM observability and reliability signals

This note captures research on **observability as an evidence source for EOKS reliability and control**. The canonical EOKS evaluation model lives in [`evaluation-and-model-switching.md`](evaluation-and-model-switching.md) and [`docs/evaluation.md`](../docs/evaluation.md); this note should not duplicate those models.

## Observability is not evaluation or control

Tracing systems can record model calls, prompts, responses, token usage, latency, tool calls, retrieval and memory activity. These observations are valuable inputs to EOKS, but they do not by themselves establish correctness, explain causality, or determine what should happen next.

The existing EOKS boundary remains:

```text
observability -> execution evidence
                    |
                    v
evaluation -> outcome / quality evidence
                    |
                    v
control -> next action
```

A model self-rating, entropy measure or trace attribute should therefore be treated as **evidence**, not as a universal confidence oracle. Reliability signals need calibration against the workload and decision in which they are used. See [`evaluation-and-model-switching.md`](evaluation-and-model-switching.md) for the canonical treatment of calibration, reliability and control.

## OpenTelemetry as an interoperability substrate

OpenTelemetry (OTel) GenAI semantic conventions are useful prior art because they provide a common vocabulary for observing GenAI execution across providers and frameworks. Current conventions cover model inference, agents/workflows, tools, retrieval and memory, including model identity, token usage and evaluation results.

EOKS should treat OTel as an **interoperability substrate for execution evidence**, not as an EOKS ontology. In particular, EOKS should not create a parallel telemetry schema merely to rename concepts that already exist in the ecosystem.

The useful mapping is approximate:

| EOKS meaning | Typical telemetry evidence |
|---|---|
| Run / workflow | trace or workflow span |
| Agent invocation | agent/workflow span |
| Model inference | GenAI inference span |
| Tool execution | tool span |
| Retrieval / memory | retrieval/memory span |
| Evaluation | evaluation result/event |
| Outcome | run/workflow state and artifacts |

OTel spans are appropriate for operations with duration and hierarchy; events are appropriate for point-in-time occurrences and state transitions. Evaluation results can be attached to the operation or run being evaluated. These are implementation choices, not additional EOKS primitives.

## Telemetry attributes and attribution

The important question for harness and workload evaluation is not whether every concept gets an OTel attribute. It is whether telemetry preserves enough **identity and lineage** to connect an intervention with its execution and outcome.

At minimum, useful evidence needs to make it possible to relate:

```text
task / workload
   -> run / workflow
      -> decision / role
         -> context / acquisition
         -> model inference
         -> tool / retrieval / memory
      -> outcome
   -> evaluation
```

This supports attribution without creating a separate `Harness` object. A harness change is simply a configuration/execution intervention whose effect can be evaluated against the same task contract.

### Raw observations versus derived measurements

Token counts, latency, cache usage, tool calls and evaluation events are **observations**. Cost shares, per-agent totals, intervention deltas and cost-per-correct-result are **derived measurements**.

The distinction matters because nested agent/workflow telemetry can repeat child inference totals and accidentally double-count usage. Derived attribution therefore needs an explicit scope and accounting rule rather than being treated as another raw span attribute.

This is the right place to interpret "tokenomics": as one cost/evaluation view over the broader resource economics of a run, alongside latency, infrastructure cost, context acquisition, tools, memory, evaluation and retries. The optimization target is useful verified outcome per total resource cost, not minimum token count.

## Context occupancy is not token usage

Per-call input/output tokens describe what a provider processed for an individual call. They do not necessarily describe the context state that led to the decision. Recent OTel discussion around agent context occupancy is therefore relevant to EOKS: **context size/occupancy is a different measurement from per-call token usage**.

For EOKS, context composition and acquisition provenance are more useful than an additional undifferentiated token counter when evaluating context or harness interventions. They can explain why a call was expensive and whether the consumed context was relevant to the task.

## Traces do not automatically provide causal explanation

A rich trace can establish what executed and what was observed. It does not automatically establish why a consequential decision occurred or which context item caused it.

This distinction has empirical support. *TelemetrySuffBench: Is Agent Telemetry Sufficient for Failure-Origin Diagnosis?* (2026) reports a large gap between detecting failures from telemetry and correctly localizing their origin when decision content and provenance are removed. The result is directly relevant to EOKS: **failure detection and causal attribution are different evaluation capabilities**.

Therefore, where attribution matters, EOKS should preserve explicit provenance from consequential decisions to relevant workload state, policy and context/evidence selection. Telemetry should not be assumed to provide causal explanation merely because the trace is detailed.

## Harness evaluation

Recent harness-engineering research reinforces the existing EOKS evaluation model rather than requiring a new architectural layer. In particular, *Agentic Harness Engineering: Observability-Driven Automatic Evolution of Coding-Agent Harnesses* (Lin et al., 2026) makes harness edits explicit, distills execution experience into inspectable evidence and links each edit to a prediction that is later checked against task-level outcomes.

Its useful EOKS contribution is the **falsifiable intervention**: a change to the execution environment or policy should be represented together with its execution evidence and subsequent outcome. The appropriate home for this idea is the existing evaluation/reconciliation model, not a new harness ontology. See the dedicated entry in [`evaluation-and-model-switching.md`](evaluation-and-model-switching.md).

## Research questions

The remaining OTel-specific questions are narrower than the general EOKS evaluation questions:

1. Which stable identifiers and lineage relationships are sufficient to attribute workload outcomes to context, model, tool and execution-policy interventions?
2. Which OTel GenAI signals remain stable enough across providers and frameworks to serve as interoperability evidence?
3. How should derived cost attribution avoid double-counting nested agent/workflow aggregates?
4. When does context occupancy add explanatory value beyond provider-reported token usage?
5. What minimum decision/context provenance is required for reliable failure localization without capturing unnecessary sensitive content?
6. How should OTel evaluation events relate to richer EOKS evaluation records without creating two competing evaluation models?

These should remain empirical interoperability questions. They should not become new EOKS semantic objects unless repeated experiments demonstrate a genuine architectural gap.
