# Agent-loop mediation spike

## Purpose

Test the new EOKS hypothesis that a semantic control loop can mediate an existing coding-agent runtime directly through its programmatic control surface, without becoming another agent runtime, terminal harness, knowledge store or generic SDK.

This is the first implementation experiment following the agent-loop mediation research. It should be deliberately small: prove the boundary before building infrastructure around it.

## Hypothesis

> A thin EOKS controller can observe an agent turn, supply a task-specific working set, make one semantic control decision, and feed the resulting outcome back into the next turn without requiring terminal/process control or a separate persistent knowledge system.

A successful spike should demonstrate that the useful EOKS responsibility is **semantic mediation**, while agent execution remains owned by the underlying runtime.

## Scope

Start with one repository-engineering task and one agent runtime. Implement the smallest adapter needed to:

1. establish a workload objective and policy;
2. construct a minimal context/working-set manifest;
3. start or resume an agent run;
4. observe agent-loop events and outcomes;
5. record the decision and evidence needed to explain the next action;
6. make one controller decision after an observable transition;
7. continue, verify, or stop according to that decision;
8. capture what changed in the working set/context and what should survive the run.

Then repeat the same experiment against the second vendor runtime if the first adapter validates the boundary.

### First runtime

Use the runtime with the cleanest programmatic control surface available at implementation time. Codex app-server is the preferred first candidate because it exposes explicit thread/turn lifecycle and event-driven control. Claude Agent SDK is the comparison implementation.

Do not require both adapters before testing the hypothesis.

## Deliberately out of scope

The spike must not introduce:

- a new agent runtime;
- terminal or pane management;
- process supervision;
- a general-purpose agent SDK;
- a graph, wiki or database for persistence;
- a new generic model router;
- a replacement for vendor sessions, compaction, handoffs or subagents;
- autonomous long-running scheduling;
- a large vendor-neutral protocol.

Vendor-specific capabilities should remain behind the adapter until the experiment demonstrates a stable common semantic boundary.

## Minimal EOKS-side contract

The experiment should use only concepts already present in the architecture:

```text
Workload
  objective
  policy
  working set / context
  acceptance criteria

Agent adapter
  start / resume
  send or continue
  observe events
  interrupt when supported
  collect outcome

Conductor decision
  continue
  acquire / change context
  verify
  delegate / escalate
  stop

Outcome
  observations
  evidence
  artifacts
  cost / usage where available
  next-state signals
```

The adapter is an execution resource/provider boundary. It translates EOKS decisions into the native runtime API; it does not own the decision semantics.

## Event observation

Do not normalize every vendor event. Preserve the native event/provenance and extract only the minimum semantic observations required by the controller:

- turn/session started or resumed;
- model output available;
- tool/action requested or completed;
- waiting/blocked state when exposed;
- verification result;
- turn/session completed or failed;
- usage/cost when exposed;
- explicit human intervention.

The experiment should prove whether these observations are sufficient to drive a useful reconciliation step. If not, record the missing signal rather than inventing a broad event ontology.

## First control loop

Use a task where the agent must inspect, modify and verify a small repository change.

```text
intent + policy
      |
      v
working-set construction
      |
      v
agent start/resume
      |
      v
observe agent-loop events
      |
      v
identify semantic transition
      |
      v
Conductor decision
      |
      +---- continue with current context
      +---- acquire/compile additional evidence
      +---- request verification/review
      +---- stop/escalate
      |
      v
agent continues / verification runs
      |
      v
Outcome + evidence
      |
      v
context-evolution candidate extraction
```

The first experiment should intentionally include at least one point where the controller has a reason to make a different choice than simply allowing the agent to continue unchanged. Otherwise it does not test mediation.

## Assisted versus autonomous mode

Test the boundary in two modes, but not necessarily in the same implementation step.

### Autonomous mode

EOKS owns the agent session through the runtime's programmatic interface.

This is the cleanest test of semantic mediation.

### Assisted mode

A normal Claude Code or Codex CLI workflow remains user-facing while EOKS observes/augments it through supported hooks or integration surfaces.

This tests whether useful EOKS semantics can be added without replacing the user's existing harness.

The experiment should not assume that both vendors expose equivalent assisted-mode control.

## Measurements

Use the existing EOKS evaluation discipline. Record:

- task success / acceptance outcome;
- verification evidence;
- context supplied and context changes;
- agent turns and tool/action count where available;
- latency;
- token/usage/cost where available;
- controller interventions;
- interventions that were unnecessary or harmful;
- information retained for the next turn/run;
- failure modes caused by insufficient control visibility.

The key question is not whether the adapter reduces tokens. It is:

> Does semantic mediation improve the end-to-end engineering outcome or assurance enough to justify its control complexity?

## Baselines

At minimum compare:

1. normal agent workflow with its native context/session behavior;
2. EOKS mediation with the same model/runtime and an equivalent initial task;
3. where practical, EOKS mediation with the intervention disabled after observation, so the value of the controller decision can be isolated.

Do not compare different models or runtimes until the within-runtime baseline is understood.

## Success criteria

The spike is successful if it demonstrates all of the following:

- EOKS can observe enough agent-loop state to identify a meaningful control point;
- the Conductor can make a semantic decision without owning agent execution;
- the decision can be expressed through the native runtime API;
- the resulting outcome can be evaluated and fed back into reconciliation;
- the experiment can identify what context should remain transient versus become a candidate for durable evolution;
- the adapter remains thin enough that vendor-specific runtime behavior is not leaking into EOKS architecture.

A failure is also useful if it identifies a missing runtime capability, insufficient event visibility, or a boundary that cannot be shared between vendors.

## Expected artifacts

The implementation should initially produce:

- one small adapter;
- one controller experiment;
- a reproducible task/workload definition;
- structured execution traces sufficient to inspect decisions and outcomes;
- an experiment report using the standard EOKS experiment record.

Do not create a persistent store specifically for the spike unless the experiment demonstrates that transient trace handling is itself the limiting factor.

## Follow-up decisions

After the first runtime:

- **Use / integrate** if the native API is sufficient and the boundary is thin;
- **Prototype further** if the control boundary is useful but a missing semantic capability is identified;
- **Compare vendors** if the first runtime validates the model;
- **Reject / narrow** if EOKS mediation adds complexity without measurable control or assurance value.

Only after this experiment should EOKS consider a reusable agent adapter abstraction. The implementation should follow the evidence rather than establish a generic interface in advance.
