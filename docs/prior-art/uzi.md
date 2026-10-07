# uzi

[uzi](https://github.com/vtmocanu/uzi) is an open-source "AI dark factory" for software engineering. Its stated direction is roughly **issues in, reviewed pull requests out**: durable work is executed through isolated workers and specialized agent roles, with planning, approval, implementation, verification and review around the resulting artifacts.

## Why it is relevant

uzi is useful prior art because it implements, in a concrete coding-agent system, many of the primitives and relationships already described by EOKS:

| EOKS concept | uzi implementation | Architectural signal |
|---|---|---|
| Intent / Work | issue-driven work item | intent can persist independently of a particular execution |
| Run | durable run with lifecycle, milestones and outcomes | one Work can lead to multiple execution attempts |
| Agent | lead/coder/reviewer/auditor/tester-style roles | agent roles are execution choices, not new ontology |
| Execution environment | isolated workers / containers with repository checkout | execution is separate from the logical agent |
| Capabilities | skills, tools and worker capabilities | capabilities constrain what an execution can do |
| Workflow | plan, approve, implement, verify, review | workflow coordinates existing primitives rather than replacing them |
| Policy / human control | approval gates and execution guards | authority can be constrained independently of model capability |
| Artifacts | branches, commits and pull requests | execution produces durable external artifacts |
| Evaluation | reviews, audits, tests and a final judge | outcome quality is evaluated separately from generation |
| Learning | judge recommendations / feedback | learning can produce advice without automatically mutating the system |
| Control loop | CI/review feedback can trigger further work | completed runs can feed subsequent reconciliation |

None of these requires a new EOKS primitive.

## The useful validation for EOKS

uzi provides concrete evidence for several architectural choices in EOKS.

### Work and Run should remain distinct

The same piece of requested work may require retries, rework or additional verification. Treating the durable request and an individual execution as the same object would make that lifecycle harder to represent.

```text
Work
  |
  +--> Run 1 -- failed verification
  |
  +--> Run 2 -- review requested changes
  |
  +--> Run 3 -- accepted PR
```

The implementation details differ from EOKS, but the distinction is directly useful.

### Execution is more than an agent

uzi separates the logical roles performed by agents from the worker/environment in which they execute. This reinforces the EOKS relationship:

```text
Run
 |
 +--> Agent
 +--> Execution Environment
 +--> Capabilities
```

A model/agent should not implicitly define the filesystem, isolation, tools, credentials or resource limits available to it.

### Workflow should not become the EOKS center

uzi can look workflow-centric because its visible lifecycle is plan → approve → code → review → test → PR. But the useful architectural decomposition underneath is:

```text
Work
  |
 Run
  |
  +--> agents
  +--> environment
  +--> capabilities
  +--> artifacts
  +--> observations/evaluation
```

The workflow describes coordination between these elements. It is not a replacement for them.

### Feedback crosses lifecycle boundaries

The important loop is not only internal workflow state:

```text
Run
 |
 v
artifact / PR
 |
 v
CI / review / external system
 |
 v
observation / evaluation
 |
 v
new decision or Run
```

This reinforces treating external feedback as part of the EOKS control loop rather than merely as execution telemetry.

### Learning should remain governed

uzi's judge/recommendation direction is consistent with an important EOKS boundary:

```text
Run
 |
 v
Evaluation
 |
 v
Recommendation
 |
 +--> human / policy
 |
 +--> future Work or execution
 |
 +--> candidate knowledge
```

A successful run should not automatically become canonical knowledge or change future behavior without an appropriate promotion and policy step.

## What uzi does not add

After mapping uzi against the current EOKS model, it does not reveal a missing top-level dimension or runtime primitive.

In particular, uzi does **not** require EOKS to add:

- a new "dark factory" primitive;
- a separate agent-fleet primitive;
- a workflow-centric architecture;
- a judge primitive;
- a worker primitive as a new semantic layer;
- automatic learning/mutation.

These are implementation mechanisms or compositions of existing EOKS concepts.

## EOKS interpretation

uzi is therefore best treated as **validation/evidence for the current EOKS architecture**, rather than as a new architecture to synthesize into it.

The strongest evidence to retain is:

```text
intent / Work
      |
    Run(s)
      |
agent + environment + capabilities
      |
artifacts + observations
      |
evaluation / external feedback
      |
policy / decision
      |
new Run / outcome / candidate learning
```

This is especially useful because uzi is a relatively concrete end-to-end implementation rather than a conceptual framework. It demonstrates that the separation of Work, Run, execution environment, capabilities, evaluation and feedback is practical for a real coding-agent system.

## Limits

uzi should not be treated as proof that its particular orchestration, role decomposition or approval flow is optimal. The useful evidence is the **separation of concerns and lifecycle relationships**, not the exact implementation.

It is also one implementation, so it should be compared with other execution systems before EOKS turns any of its design choices into normative guidance.