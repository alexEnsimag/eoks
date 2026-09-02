# System harnesses, feedback loops and EOKS

## Why this matters to EOKS

Gorinova et al., *Position: Coding Benchmarks Are Misaligned with Agentic Software Engineering* (2026), argues that a practical coding agent is not just a model. It distinguishes an **agent harness**—a model interacting with tools for a task—from a **system harness**, which operates at a larger orchestration scope: turning higher-level goals into tasks, dispatching agent harnesses, managing their environment and routing feedback over time. The paper identifies five recurring system-harness components: tasks, agent harnesses, environment, context and feedback signals.

[Paper: arXiv:2606.17799](https://arxiv.org/abs/2606.17799)

This is useful prior art for EOKS because it independently reinforces several distinctions already present in the architecture: the model is only one execution resource; context is a workload-specific materialization rather than the whole knowledge system; evaluation/feedback is part of execution control; and long-running software work must be evaluated at the system/workload level rather than by a model-only score.

## Agent harness versus system harness

The paper's distinction is useful, but EOKS should not turn it into another architectural layer:

```text
system harness
  |
  +-- tasks / work decomposition
  +-- agent harnesses / executors
  +-- environment
  +-- context
  +-- feedback
```

In EOKS terms, these are primarily workload, execution, resource, context and evaluation concerns coordinated by reconciliation.

The important boundary is:

> **The system harness is execution/orchestration machinery; EOKS is the broader control layer that selects resources, context, execution modality and verification, observes outcomes, and reconciles the workload.**

A system harness can therefore be an execution resource or integration boundary for EOKS. EOKS does not require ownership of the underlying coding-agent runtime.

This is consistent with the existing EOKS integration direction: coordinate existing coding agents and tools rather than replacing them, while keeping the control responsibility around the workload.

## Feedback as control evidence

The paper distinguishes feedback by scope and latency:

```text
inner loop   -> tests / types / lint / compile
middle loop  -> review / maintenance / broader quality signals
outer loop   -> PR acceptance / reverts / incidents / user outcomes
```

The paper also distinguishes strict verifiers that can block progress from broader feedback that informs decisions without being a hard gate. Inner-loop signals are fast and narrow; middle-loop signals expose recurring quality or maintenance problems; outer-loop signals are delayed and closer to deployment reality. The paper argues that all three can participate in improvement of the system harness.

This maps directly onto the existing EOKS control loop rather than requiring a new feedback abstraction:

```text
execution
   |
   v
observation / evidence
   |
   v
evaluation
   |
   +--> continue / verify / repair / re-plan / escalate
   |
   v
actual workload state
   |
   v
reconcile
```

Inner, middle and outer feedback can be understood as **nested or differently scoped observations of the same workload control problem**. Their different latency, scope and trust characteristics are useful metadata for deciding how strongly a signal should influence control.

The existing EOKS principle remains important: an observation is not automatically a reliable control signal. Signals should be calibrated against actual workload outcomes before they are allowed to drive consequential automatic decisions.

## Component-level attribution

The paper's central benchmark criticism is also relevant to EOKS experiments. A single end-to-end score can hide whether a result changed because of the model, context, tools, environment, orchestration or feedback policy. Its proposed direction is to expose component-level signal so the system can be iterated rather than treating the whole harness as an opaque number.

For EOKS this reinforces the existing experiment requirement to record, where practical:

- model and version;
- context/loadout and acquisition decisions;
- provider/resource selection;
- tool and execution traces;
- verification and evaluation evidence;
- retries, branches and escalations;
- cost and latency;
- final and delayed outcomes.

The goal is not to maximize attribution granularity. It is to make a workload outcome diagnosable enough to determine which intervention is worth changing.

## Loop engineering as a convergent direction

LoopsBench, *From Harness Engineering to Loop Engineering in Coding Agent Evaluation* (2026), is useful follow-on evidence for this direction. It evaluates sustained long-horizon coding through dependency-aware development units and explicitly studies loop implementations rather than only localized agent behavior.

[LoopsBench: arXiv:2608.00267](https://arxiv.org/abs/2608.00267)

This supports a useful progression in the research vocabulary:

```text
model
  -> agent harness
  -> system harness
  -> loop engineering
  -> workload control / reconciliation
```

This should be read as a convergence of research concerns, not as a claim that these are universally accepted architectural layers. In particular, **loop engineering does not replace the EOKS control-loop concept**; it provides a concrete software-engineering domain in which the control-loop hypotheses can be measured.

## EOKS boundary

EOKS should not introduce:

- a `SystemHarness` runtime primitive;
- a separate feedback-loop subsystem;
- a benchmark-specific harness ontology;
- a mandatory multi-agent orchestration layer;
- a requirement to replace existing coding-agent runtimes.

Instead, EOKS should treat harnesses, agents, models, tools, environments and feedback mechanisms as resources, execution mechanisms or evidence sources whose selection and coordination can be controlled and evaluated.

The useful EOKS question is therefore not **"which harness wins?"** but:

> **Given a workload, policy and available execution resources, which combination of context, harness behavior, verification and feedback produces the best trustworthy engineering outcome for the cost?**

## Research implications

The paper strengthens several existing EOKS experiments:

1. **Evaluate components as well as end-to-end outcomes.** Record enough trace data to identify which resource or policy changed the result.
2. **Evaluate nested feedback loops.** Test whether cheap inner-loop evidence improves outcomes and whether middle/outer signals improve the calibration of those inner signals.
3. **Separate verification from broader feedback.** A blocking test result and a qualitative review comment should not be treated as equivalent evidence.
4. **Measure delayed outcomes.** PR acceptance, reverts, incidents and maintenance effects can validate or invalidate shorter-horizon proxies.
5. **Test loop policies, not only agent capabilities.** Compare continuation, verification, repair, re-planning and escalation policies under the same workload and execution resources.
6. **Keep the benchmark boundary explicit.** A benchmark can measure a model, an agent harness, a system harness or a sustained workload loop; the reported construct should match what is actually being measured.

The paper therefore strengthens the existing EOKS evidence model without adding another architectural abstraction.
