# Probabilistic decision and semantic control: Jev / Foreman prior art

## Why this matters to EOKS

Recent work by Josh Rosen around Jev and the Foreman software-factory supervisor provides a concrete example of a pattern already implicit in EOKS: probabilistic reasoning can be used as a bounded decision primitive inside a predominantly deterministic control system, rather than making an agent responsible for the entire workflow.

The important insight is not that EOKS should depend on Jev. It is that EOKS may need an explicit architectural concept for semantic/probabilistic decision points.

## Rosen's Jev pattern

Rosen's "Jev in the Wild: Early Architecture Patterns for System One Models" describes Jev being embedded in conventional software for bounded decisions such as routing, context selection, semantic tool gating, worker supervision, fast bounded control loops, and fuzzy predicates over data.

The common shape is deterministic state/evidence -> bounded semantic question -> probabilistic decision -> deterministic policy/action.

The probabilistic component does not need to own the workflow, durable state, authorization boundary or final execution.

## Foreman: probabilistic supervision

Foreman is a particularly useful software-engineering example. A coding worker performs the implementation while a separate supervisory loop evaluates evidence about progress, completeness, tests, drift and verification.

The useful architectural separation is: coding agent -> changes/evidence -> probabilistic assessment -> bounded decision -> deterministic policy -> continue/steer/retry/verify/stop/escalate.

This is not simply one agent watching another. The important property is that judgment and execution have different authority.

An agent's self-report can be an observation, but it does not automatically become acceptance evidence. Independent tests, analyzers, runtime observations, review and human approval can have different authority for different decisions.

## Relationship to the EOKS control loop

The existing EOKS model already contains Intent/desired outcome, Policy, Resources and capabilities, Working set and Context, State and Evidence, Run/execution, Decision, Evaluation, Outcome and Reconciliation.

Jev suggests that Decision should be understood as a control-plane boundary, not necessarily as an LLM agent step.

A decision may be produced by deterministic rules, a probabilistic model, an evaluator/judge, a planner, a human, or a composition of these. Jev is therefore an implementation example of one class of decision mechanism, not a new EOKS primitive.

## A refined architecture

The EOKS dimensions can be understood as the state/context of an AI engineering environment, while decision mechanisms operate over that environment:

Environment: what exists and what can be known or done.

Decision: what should happen now, given the relevant environment state and evidence.

Execution: what actually changes the environment.

This is a conceptual decomposition, not a proposal to create three new runtime layers.

## Context selection as a decision problem

One especially relevant Jev pattern is context filtering.

EOKS already distinguishes the resource/evidence universe, workload working set and model context. Probabilistic decision mechanisms add another useful interpretation: candidate evidence -> current intent/state/policy -> relevance or sufficiency decision -> working context.

The decision can be deterministic where possible and probabilistic where semantic interpretation is required.

This reinforces an existing EOKS principle: knowledge is not context. Knowledge can be large and durable; context is a task-specific projection selected for a particular reasoning step.

## Planning and control

Rosen's work also connects to EOKS's existing planning/control-loop research.

Planning can propose a sequence or policy from intent and current state. A controller still needs to decide what to do when observations invalidate assumptions.

A useful decomposition is Intent -> Plan/Policy proposal -> current State/Evidence -> Decision -> Action -> new State -> reconcile/re-plan.

This avoids equating planning with workflow execution. Plans can remain disposable proposals when observations make their assumptions invalid.

The user's earlier AI-planning research is relevant as prior art for the planning side of this separation; Jev is relevant to the bounded decision side.

## Execution lineage and durable artifacts

Rosen's earlier work on execution lineage and intermediate artifacts provides a complementary substrate.

The common theme is that important work should not exist only as ephemeral agent context. Intermediate artifacts, evidence and dependencies should be durable enough to inspect, evaluate and replay.

That aligns with EOKS's existing emphasis on durable Work/Run state, evidence provenance, inspectable context, artifacts and outcomes, reconstructability, and separating logical execution state from the process/container/VM currently realizing it.

The combined picture is: durable state/artifacts/evidence -> context compilation -> bounded decision -> deterministic policy -> execution -> new artifacts/state -> evaluation/reconciliation.

## What EOKS should take from this

1. Keep Decision as a first-class conceptual primitive. The current seven-primitives model already has it; Jev strengthens the rationale.
2. Do not make Jev a dependency or ontology object. It is a concrete implementation of probabilistic decision, not the semantic model itself.
3. Make decision authority explicit. A probabilistic judgment is evidence for a control decision; it is not automatically authorization.
4. Separate decision from execution. An agent can produce a decision, but deterministic code, policy or a human can own the consequential action.
5. Treat context selection as a control decision. Relevance and sufficiency are decisions over current workload state, not merely retrieval operations.
6. Prefer deterministic mechanisms where they are sufficient. Probabilistic judgment should fill semantic gaps rather than replace tests, schemas, type systems or explicit policy.
7. Calibrate consequential probabilistic decisions against actual workload outcomes before allowing them to drive stop/continue/repair/model-switch behavior.
8. Preserve evidence and provenance so decisions can be reconstructed from relevant state, evidence, policy and evaluator/version information.
9. Keep agents as replaceable execution resources. Supervision should be possible without making the worker agent the owner of the control loop.
10. Use Jev/Foreman as a reference pattern. They are evidence that this architecture is implementable, not evidence that EOKS should adopt their particular technology.

## Open questions

- Which decisions benefit enough from probabilistic judgment to justify its cost and variance?
- What is the minimum evidence contract for a semantic decision?
- How should deterministic and probabilistic decision mechanisms be composed?
- When should a probabilistic decision be advisory versus consequential?
- How should decision calibration be represented and monitored?
- Can repeated probabilistic judgments be promoted into deterministic rules?
- What decision/event schema is sufficient to reconstruct why an action occurred?
- How should context-selection decisions be evaluated against downstream outcomes?
- Where should human judgment sit when probabilistic and deterministic evidence disagree?
- Can a common decision interface span coding agents, workflow controllers, policy engines and human approvals without becoming another orchestration framework?

## References

- Josh Rosen, "Jev in the Wild: Early Architecture Patterns for System One Models" (2026).
- Josh Rosen, work on the Jev software-factory control plane and Foreman.
- Josh Rosen, "From Agent Loops to Deterministic Graphs: Execution Lineage for Reproducible AI-Native Work" (2026).
- Josh Rosen, "Intermediate Artifacts as First-Class Citizens" (2026).

These references are tracked as external prior art; EOKS should preserve its own abstractions independently of the particular Jev implementation.