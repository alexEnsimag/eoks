# Control-loop consolidation

EOKS treats **reconciliation** as the unifying control abstraction for automated engineering work. This document records how existing concepts fit that model; it does not introduce new runtime primitives.

## The consolidated model

```text
intent + policy
      |
      v
 desired outcome/state
      |
      v
 observe current state + evidence
      |
      v
 identify gap / uncertainty
      |
      v
 conductor chooses minimum sufficient action
      |
      v
 execute -> observe -> evaluate
      |
      v
 update actual state/evidence
      |
      +------ reconcile ------+
                             |
                  continue / repair / replan
                  verify / stop / escalate
```

The key distinction is **desired state versus plan**. Intent and policy define what should become true. A plan is an action proposal produced by the controller. If observations invalidate its assumptions, the plan can be discarded and a new action derived from current state.

## Consolidated responsibilities

| Existing concept | Control-loop interpretation |
|---|---|
| Task | Bounded workload / desired objective |
| Policy | Constraints on reconciliation and acceptable actions |
| Conductor | Controller/reconciler |
| Scheduler | Resource/action selection function of the conductor |
| Router | Capability-selection function of the conductor |
| Orchestrator | Workflow-coordination function of the conductor |
| Planner | Produces an action proposal when planning is useful |
| Executor | Actuator that changes workload state |
| Retriever/provider | Sensor/evidence acquisition |
| Reviewer/validator | Observation and evaluation resources |
| Evaluation | Determines whether evidence supports convergence |
| Outcome | Recorded result of an execution attempt |
| Decision | Control-plane choice; lifecycle remains empirical |
| Context compiler | Projection of relevant state/evidence for reasoning |
| Continuous knowledge maintenance | Reconciliation of derived representations |
| Deterministic tool | Reliable sensor or actuator when deterministic behavior is sufficient |
| Escalation | A control action when the current path is insufficient |
| Learning | Slower reconciliation over outcomes to improve candidate policies/skills |

These are responsibilities and mechanisms, not additional architecture layers or agent types.

## Nested loops

EOKS can contain loops operating at different timescales:

```text
workload loop
  goal -> act -> verify -> reconcile
       |
       +--> knowledge loop
       |      authoritative source
       |          -> representation
       |          -> freshness/validity
       |          -> update/invalidate/rebuild
       |
       +--> learning loop
              outcomes
                -> candidate skill/policy
                -> validation
                -> controlled promotion
```

The workload loop may depend on knowledge representations maintained by another loop. Learning must remain controlled: an observed behavior is evidence, not automatically a policy change.

## Durable state and reconstructability

The control loop should not depend on hidden state inside one agent session. Important workload state, evidence, decisions and artifacts should be durable and reconstructable. Contexts, indexes, caches and execution sessions are derived or disposable where practical.

This is a storage-agnostic architectural requirement. It does not prescribe S3, a database or another particular implementation. The important distinction is between **authoritative state** and **derived state that can be invalidated or rebuilt**.

## Deterministic capabilities

Deterministic tools are valuable when they provide cheap, reliable observations or controlled actions. A test runner can establish whether an acceptance criterion holds; a static analyzer can establish a structural fact; a formatter can perform a constrained transformation.

Their role is therefore not to replace LLM reasoning generally, but to improve the controller's observations and actuations. The conductor can choose them when they are the minimum sufficient modality.

## Cards: hypothesis, not primitive

A Card could be useful as an inspectable representation of workload state:

```text
SPEC
  objective / constraints / acceptance criteria

STATUS
  current state / observations / evidence / artifacts

CONTROL
  current decision / next action / verification

HISTORY
  runs / decisions / evaluations / provenance
```

This resembles the useful separation between desired specification and observed status in Kubernetes. However, EOKS should first test whether Task, Policy, Run, Decision, Evaluation and Outcome already provide these semantics. If Cards prove useful, they should initially be a representation, protocol or UI over existing state—not a new source of truth or runtime primitive.

## Design rule

For any proposed EOKS concept, ask:

1. What desired or actual state does it represent?
2. What does it observe?
3. What can it change?
4. How is the result verified?
5. What invalidates or rebuilds it?
6. Is it already a role, resource, provider, representation, context artifact, workflow construct, policy or conductor function?
7. What implementation or evaluation evidence justifies keeping it as a separate abstraction?

**Prefer consolidation until evidence demonstrates that a new abstraction is necessary.**
