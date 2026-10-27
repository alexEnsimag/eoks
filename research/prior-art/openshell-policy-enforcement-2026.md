# NVIDIA OpenShell: policy enforcement and verification at the execution boundary

## Why this matters to EOKS

NVIDIA OpenShell is useful prior art for a part of EOKS that is already present in the architecture: **policy and capability intent must be enforced by a trusted execution substrate, not by the agent being governed**.

This is not a proposal for a new EOKS dimension or a new sandbox abstraction. EOKS already separates the workload/control loop from a replaceable execution plane, distinguishes logical Work state from the process or machine realizing it, and recognizes that filesystem, network, credential, resource and isolation constraints need substrate-level enforcement. See the [EOKS architecture](../../docs/architecture.md), [resource model](../../docs/resource-model.md), [terminology](../../docs/terminology.md), [Google AX / Agent Executor](agent-executor-ax.md), and [DeepSeek Elastic Compute](deepseek-elastic-compute-2026.md).

OpenShell is valuable because it turns those boundaries into a concrete implementation and exposes a few questions worth making more precise in EOKS.

## OpenShell's implementation boundary

OpenShell describes three stable runtime components—CLI, Gateway and Supervisor—with sandbox workloads and infrastructure adapters around them:

- **Gateway:** owns control-plane state, authorization, sandbox lifecycle, policy/settings delivery, provider records and coordination.
- **Supervisor:** sits at the local enforcement boundary, launches the agent as a restricted process, applies or refreshes supported controls, mediates network access and credentials, and emits logs.
- **Sandbox / compute runtime:** supplies the isolation boundary and workload environment. Compute drivers can integrate with different infrastructure without moving infrastructure-specific behavior into the semantic model.
- **Policy Prover:** checks proposed policy changes against an allowed access boundary and identifies risky permission expansion before approval.

The key split is **desired policy and durable control-plane state on one side; local enforcement close to the workload on the other**. OpenShell's documented controls include filesystem and process restrictions, network egress rules, credential mediation, and inference routing. Some controls are static for a sandbox lifetime and require recreation; supported network/credential/inference controls can be updated while it is running.

Sources: [How OpenShell Works](https://docs.nvidia.com/openshell/latest/about/how-it-works), [OpenShell repository](https://github.com/NVIDIA/OpenShell), and [Security Best Practices](https://docs.nvidia.com/openshell/latest/security/best-practices.html).

## What this adds to the existing EOKS model

| Concern | Existing EOKS position | Concrete OpenShell prior art / question |
|---|---|---|
| Execution isolation | Execution is a replaceable substrate; isolation requirements belong at the policy/environment boundary. | A supervisor outside the agent process mediates access; the runtime establishes filesystem, process and network boundaries. |
| Policy vs enforcement | EOKS distinguishes semantic policy from constraints that the substrate must physically enforce. | The gateway distributes policy, while the supervisor enforces it where process identity, filesystem and network activity are visible. |
| Credentials | Credentials and capabilities are governed resources; a model's claim that it will protect a secret is not an enforcement mechanism. | Providers map logical service access to stored credentials, which are supplied through controlled request paths rather than handed to the agent as general-purpose secrets. |
| Permission changes | Decisions and consequential state transitions remain subject to policy and assurance. | Proposed policy expansion can be checked against a defined boundary; supported controls can be updated during a run, while static controls require recreation. |
| Security evidence | Observations, evidence, evaluation and authority are distinct; an agent's self-report is not authoritative by itself. | Explicit allow/deny and lifecycle logs can provide independent evidence, but their coverage and retention limits must be represented. |
| Portability | Work and execution state should not be conflated with the currently allocated runtime. | Gateway/supervisor and compute-driver boundaries show how runtime-specific isolation can sit behind a stable control contract. |

The main lesson is not “EOKS needs sandboxing.” It is that **the contracts between policy, authorization, enforcement, and evidence should be precise enough to survive changes of runtime**.

## The most useful refinement: policy changes are governed artifacts

EOKS already has the control-loop shape needed to govern a policy change:

```
observed need / proposed permission
             |
             v
       candidate policy
             |
             v
  verify against allowed boundary
             |
             v
  authorization / human gate
             |
             v
  apply through trusted runtime
             |
             v
  observe effective state + events
             |
             v
        reconcile again
```

OpenShell's Policy Prover formally checks proposed policy changes for risky expansion relative to a configured boundary. The documentation describes cases such as reaching a new host with credentials or enabling additional HTTP methods. A finding can block automatic approval and leave the change for review; a clean result should be interpreted only within the features and boundary the prover actually models.

This is a useful **policy-verification capability/provider** to investigate, not a reason to add a core ontology object. EOKS should preserve the difference between:

- **verified within stated coverage** — the checker established its defined property for the supported policy features;
- **violated** — the checker found a counterexample or forbidden permission expansion;
- **unsupported / inconclusive** — the requested property or policy construct was not fully checked;
- **authorized** — the responsible policy/approval mechanism permits the change;
- **enforced** — the target runtime actually applied the authorized policy.

These are proposed result distinctions for an adapter contract, not a claim that OpenShell exposes exactly this normalized API. In particular, formal policy checking does not prove that the whole workload is safe, that every possible policy construct was modeled, or that the runtime successfully applied the resulting policy.

Sources: [Policy Prover documentation](https://docs.nvidia.com/openshell/latest/how-it-works/policies/prover), [OpenShell architecture](https://docs.nvidia.com/openshell/latest/about/how-it-works).

## Delegation and runtime replacement

Two boundary cases deserve explicit tests against EOKS's existing model.

### Child executions

A child agent or delegated run should not gain authority merely because it is created by a permitted parent. The system needs a defined relationship between the parent's authorized boundary, the child's requested permissions, and the effective policy enforced by the child runtime.

A useful invariant to test is:

> A delegated execution cannot obtain effective permissions outside the authority explicitly delegated to it, except through a separately authorized policy transition.

This is a candidate invariant for policy/evaluation tests, not a claim that EOKS already has a complete permission-delegation algorithm.

### Resume or backend migration

A logical Work can outlive its process, container or VM. Resuming it on a different execution backend should not automatically inherit trust from the previous environment. The new backend must demonstrate that it satisfies the Work's required isolation and policy constraints, and the run trace should retain which policy revision and environment were effective.

This builds on the existing EOKS execution-state and loadout model. It does not require EOKS to implement the isolation mechanism itself.

## Runtime telemetry as evidence

OpenShell's allow/deny and lifecycle events can be treated as evidence providers for EOKS evaluation and reconciliation. They may help answer whether an execution attempted a forbidden action, which policy revision was active, or whether a permission was granted.

However, **absence of a log event is not proof that an action did not occur**. OpenShell documents local logging and log push, but audit completeness still depends on collection, retention, delivery and failure behavior. EOKS should preserve telemetry coverage, provenance and known gaps rather than collapse runtime events into an unqualified “secure” verdict.

Potential evidence fields for a provider-neutral adapter include:

- workload, run, sandbox and environment identity;
- policy revision/hash and enforcement point;
- event time and collection time;
- allow/deny/violation result and the operation it refers to;
- source/runtime and schema version;
- coverage, delivery/retention status and known gaps;
- correlation to the decision or policy transition that caused the event.

This is a candidate mapping onto EOKS's existing Evidence/Provider model, not a recommendation to create a new security-specific evidence primitive.

Sources: [How OpenShell Works](https://docs.nvidia.com/openshell/latest/about/how-it-works), [Network Rules](https://docs.nvidia.com/openshell/dev/how-it-works/policies/network-rules).

## What should not be imported into EOKS

OpenShell does not justify:

- a new EOKS dimension for sandboxing or security;
- making OpenShell, OPA, a specific kernel mechanism or a particular VM technology a semantic dependency;
- assuming all execution providers support the same policy features or live-update semantics;
- treating a policy prover's success as authorization, runtime enforcement, or proof of whole-task safety;
- treating missing telemetry as proof that nothing happened;
- conflating the policy desired by the control plane with the policy actually active in a runtime.

OpenShell is one provider that can realize some execution and policy-enforcement capabilities. Other providers may expose different guarantees and must declare their limitations.

## Focused follow-up questions

These should be checked against the existing EOKS open questions and answered through contracts or experiments before changing the core model.

1. **Policy verification:** Can a policy/evaluation provider state its verified property, supported constructs, boundary, and result provenance? How are unsupported or inconclusive results prevented from being treated as success?
2. **Delegation:** How is authority constrained when Work spawns a child execution, parallel agent or remote worker?
3. **Effective policy:** Can each Run record the requested policy, authorized revision, and effective policy confirmed by the runtime?
4. **Runtime replacement:** What evidence is required before a resumed/migrated Work is allowed to continue on a different backend?
5. **Evidence quality:** How do missing, delayed, dropped or incomplete security events affect assurance and automated decisions?
6. **Portability:** What is the smallest provider-neutral contract for policy application, verification, runtime identity and enforcement evidence without pretending that every backend offers equivalent guarantees?

## Proposed experiment

Use one bounded software-engineering Work and compare two execution providers or configurations, one with a runtime-enforced boundary and one with a simpler local execution path. Keep task and acceptance criteria constant. Test at least:

- an allowed filesystem operation;
- a denied filesystem operation;
- an allowed network request;
- a denied network request;
- a proposed permission expansion;
- a child execution with a narrower delegated boundary;
- interruption/resume or backend replacement, if supported;
- missing or delayed telemetry.

Record requested/authorized/effective policy, checker result and coverage, enforcement result, runtime identity, events collected, evidence gaps, task outcome, setup cost and human interventions. The experiment should establish which guarantees are observable and portable—not assume one backend wins.

## Sources

- NVIDIA, [How OpenShell Works](https://docs.nvidia.com/openshell/latest/about/how-it-works).
- NVIDIA, [Policy Prover](https://docs.nvidia.com/openshell/latest/how-it-works/policies/prover).
- NVIDIA, [Security Best Practices](https://docs.nvidia.com/openshell/latest/security/best-practices.html).
- NVIDIA, [OpenShell repository](https://github.com/NVIDIA/OpenShell).
- NVIDIA, [Network Rules](https://docs.nvidia.com/openshell/dev/how-it-works/policies/network-rules).

This note records implementation prior art and questions to test against EOKS's existing model. It does not make OpenShell an EOKS dependency or propose a new core primitive.
