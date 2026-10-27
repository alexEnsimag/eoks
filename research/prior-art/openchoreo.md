# OpenChoreo: platform control, agent operations, and context engineering

**Reviewed:** 2026-10-24  
**Repository:** [openchoreo/openchoreo](https://github.com/openchoreo/openchoreo)  
**Primary docs:** [Architecture](https://openchoreo.dev/docs/overview/architecture/), [Working with AI](https://openchoreo.dev/docs/ai/overview/), [MCP servers](https://openchoreo.dev/docs/reference/mcp-servers/), [v1.3 release](https://openchoreo.dev/blog/openchoreo-v1.3-release/)

## Why this matters to EOKS

OpenChoreo is more than a Kubernetes developer portal or an example of MCP integration. It is an implemented, multi-plane internal developer platform that combines declarative APIs, reconciliation, workload execution, observability, access control, audit trails, resource provisioning, and built-in operational agents.

It is useful prior art for EOKS across **control loops, capability composition, context/evidence acquisition, governed agent actions, execution environments, and outcome feedback**. It is not a general AI-engineering control plane: its primary domain is delivering and operating applications and infrastructure on Kubernetes.

## System model

OpenChoreo separates the Experience, Control, Data, Workflow, and Observability planes. The Control Plane translates developer/platform intent expressed through high-level APIs into Kubernetes resources and continuously reflects observed runtime state. Data and Workflow planes connect to the control plane through authenticated outbound WebSocket connections; the data plane can keep running workloads during disconnection, while desired-state changes wait for reconnection. The Observability Plane exposes a domain-centric API/MCP surface directly, avoiding a need to proxy all telemetry through the control plane.

Its domain model includes Components, Projects, Environments, Endpoints, Dependencies, ComponentTypes, Traits, Workflows, Resources/ResourceTypes, and deployment bindings. Platform teams configure reusable abstractions and golden paths; developers express intent without hand-authoring every low-level resource. Cells provide policy and observability boundaries around a project/environment's workloads. See the [architecture documentation](https://openchoreo.dev/docs/overview/architecture/) and [repository README](https://github.com/openchoreo/openchoreo).

## Findings across the implementation

### 1. Declarative intent plus continuous reconciliation

OpenChoreo is a concrete reference for keeping desired state separate from actual state. It translates high-level developer resources using ComponentTypes and Traits, then delegates resource reconciliation to the relevant Kubernetes API server while monitoring deployed state and taking corrective action.

**EOKS mapping:** INTENT, STATE, POLICY and the conductor/control loop already cover this. The useful nuance is the division between *semantic intent* (what a component means), *platform-specific realization* (how a Trait/ComponentType renders it), and *runtime reconciliation*. EOKS should preserve this separation without adopting Kubernetes CRDs as a universal storage format.

**Question to test:** For an EOKS workload, can the system distinguish a desired outcome, the current realization plan, observed evidence, and the acceptance decision without conflating them?

### 2. Programmable abstractions and capability composition

ComponentTypes, Traits, Workflows, ResourceTypes, and modules are extension points. A platform team can shape supported developer operations through configuration and reusable contracts rather than requiring bespoke controllers for every path. The Crossplane integration is a concrete example: a ResourceType exposes self-service infrastructure while the data plane decides how the resource is provisioned.

**EOKS mapping:** CAPABILITIES, POLICY, providers, resources and workflow composition. This supports EOKS's provider-agnostic approach: describe a capability by its contract and constraints, then let an implementation realize it. Do not turn every adapter, skill, module, or tool into a new semantic primitive.

**Risk:** Abstractions can become a second platform-specific language. Evaluate how well custom extensions remain discoverable, versionable, testable, and debuggable.

Sources: [Crossplane integration](https://openchoreo.dev/blog/provisioning-resources-with-crossplane/), [Architecture](https://openchoreo.dev/docs/overview/architecture/).

### 3. Context is an operational evidence pipeline, not only prompt text

The two MCP surfaces split control-plane resources from observability data. An agent can discover a component/project/environment, then query logs, metrics, traces, alerts, incidents, audit events, or cost insights using the same domain identifiers. Collection enriches telemetry with platform metadata, making evidence queryable in application concepts rather than only pod/namespace terms.

The Portal Assistant also uses task-specific tool sets, scoped prompt inputs, existing frontend-fetched evidence, skill loading, summarization, and tool-error/loop guards. The code explicitly avoids redundant log retrieval when suitable data is already present. The SRE agent combines telemetry with workload dependencies to investigate root causes.

**EOKS mapping:** evidence providers, working sets, context acquisition/compilation, provenance/freshness, and bounded reasoning steps. OpenChoreo supplies a concrete source of live operational evidence that can sit beside repository code, tests, architecture knowledge, and execution state.

**Important boundary:** this is useful evidence acquisition and task-aware assembly, but it is not evidence that OpenChoreo implements a general-purpose context compiler with EOKS's proposed working-set, provenance, budget, dependency-invalidation, and inspectability contracts.

Sources: [AI overview](https://openchoreo.dev/docs/ai/overview/), [MCP reference](https://openchoreo.dev/docs/reference/mcp-servers/), [Portal Assistant builder](https://github.com/openchoreo/openchoreo/blob/main/agents/portal-assistant/src/agent/builder.py), [Portal Assistant orchestrator](https://github.com/openchoreo/openchoreo/blob/main/agents/portal-assistant/src/agent/orchestrator.py), [SRE agent](https://github.com/openchoreo/openchoreo/blob/main/agents/sre-agent/src/agent/agent.py), [SRE skills extension](https://github.com/openchoreo/openchoreo/blob/main/agents/sre-agent/src/extensions/skills.py).

### 4. Built-in agents are operational workflows, not just chat

OpenChoreo has distinct agent applications:
- **Portal Assistant:** read-only natural-language access to platform and observability APIs.
- **SRE Agent:** investigates alerts and runtime evidence, produces structured root-cause analysis, and has remediation-related flows.
- **FinOps Agent:** analyzes resource usage/cost, generates rightsizing recommendations, and can support remediation.
- **Architecture agent:** listed in the platform direction/documentation as another built-in operational agent; validate its current implementation and maturity separately before relying on it.

The SRE implementation uses typed output models, middleware, request-scoped authentication, an explicit tool registry, optional summarization, and report backends. The Portal Assistant's orchestrator documents a read-only contract and uses a write guard so mutating tools are refused even if they appear in the catalog. The FinOps flow includes structured remediation/action models and a configurable remediation mode.

**EOKS mapping:** bounded workload types, capabilities, workflow roles, evidence, evaluation, policy and graduated autonomy. The useful pattern is to separate read-only diagnosis from consequential mutation and to represent reports/actions as structured artifacts rather than treating a fluent answer as completion.

**Open question:** How are proposed remediations independently verified, approved according to risk, made idempotent, and rolled back? The existence of a remediation model or guard does not by itself establish a complete assurance protocol.

Sources: [AI overview](https://openchoreo.dev/docs/ai/overview/), [FinOps Agent docs](https://openchoreo.dev/docs/ai/finops-agent/), [budget-triggered cost analysis tutorial](https://openchoreo.dev/docs/tutorials/configure-budget-alert-and-cost-analysis/), and the agent source paths above.

### 5. Authorization and audit are part of the agent surface

OpenChoreo exposes consistent APIs/MCPs under platform authorization. Its documented ABAC extension can constrain access by attributes such as environment. The v1.3 audit-log feature records state-changing operations through REST, portal, CLI, and MCP surfaces, including actor, action, resource, result, source surface, and correlation identifiers. Audit records have a separate stream/retention path from ordinary application logs.

**EOKS mapping:** POLICY and the boundary between observation, evidence, decision, and authorized state transition. Tool availability is not sufficient authorization; the action path itself must enforce policy. Audit evidence is useful for reconstructing agent behavior and evaluating policy compliance.

**Caveat:** the current audit-log documentation marks the feature beta and states that not every read or unauthenticated request is audited. Do not assume complete action/trajectory provenance from audit logs alone.

Sources: [ABAC in v1.1](https://openchoreo.dev/blog/openchoreo-v1-1-release/), [Audit logging](https://openchoreo.dev/docs/platform-engineer-guide/audit-logging/), [v1.3 release](https://openchoreo.dev/blog/openchoreo-v1.3-release/).

### 6. Operational feedback includes cost and delivery outcomes

The v1.3 release adds cost insights (OpenCost/Prometheus-backed) and DORA delivery/stability metrics, accessible through dashboards and MCPs. It also adds platform logs and audit logs. Cost recommendations can be surfaced in response to budget alerts and include actions that a user can apply.

**EOKS mapping:** evaluation and outcome should include operational consequences, not only model output or token consumption. This supports EOKS's focus on velocity, quality, trust/autonomy, and cost/scale. DORA and cloud cost are valuable workload/environment signals, but they are not direct measures of AI-generated change correctness.

**Experiment idea:** correlate agent-assisted changes with verified outcomes, rework, deployment lead time, failure rate, recovery time, and infrastructure cost. Avoid attributing changes causally to agents without a suitable baseline.

Source: [OpenChoreo v1.3 release](https://openchoreo.dev/blog/openchoreo-v1.3-release/).

### 7. Execution topology, isolation, and lifecycle are platform concerns

Separate planes have their own deployment/scaling/security boundaries. Workflow execution can use Argo Workflows by default while keeping a CRD-based engine replacement possible; external CI systems can also remain in place. Agent workloads can use platform component abstractions and integrations such as Agent Sandbox. The local dependency-tunneling workflow reduces the build/deploy cycle by letting local code access dependencies already deployed in an environment.

**EOKS mapping:** execution as a replaceable capability; Work/Run state separate from the current process, workspace, sandbox, or backend; policy and environment/loadout selection; and the principle of integrating existing execution providers rather than replacing them.

**What not to infer:** a multi-plane Kubernetes platform is not itself a durable agent execution substrate. Durable suspend/resume, checkpointing, replay safety, and portable execution state need to be verified in the specific execution backend.

Sources: [Architecture](https://openchoreo.dev/docs/overview/architecture/), [v1.1 release](https://openchoreo.dev/blog/openchoreo-v1-1-release/), [v1.3 release](https://openchoreo.dev/blog/openchoreo-v1.3-release/).

## EOKS mapping summary

| OpenChoreo pattern | Existing EOKS concept | Research value |
|---|---|---|
| Developer/Platform API and CRDs | INTENT, resources, STATE | Separate intent from realization and observation |
| ComponentTypes, Traits, ResourceTypes | CAPABILITIES, providers, POLICY | Stable contracts over interchangeable implementations |
| Control/data/workflow/observability planes | Control loop, execution provider, evidence provider | Separate control, execution, and evidence paths |
| Domain-enriched telemetry + MCP | Evidence providers, working set, context compiler | Context can be grounded in live operational reality |
| Read-only assistant + write guard | POLICY, assurance boundary | Enforce authority outside the model's prompt |
| SRE/FinOps structured reports/actions | WORKFLOW, evaluation, bounded decisions | Separate diagnosis, recommendation, action, and verification |
| Audit log stream | STATE/evidence/provenance | Reconstruct actions, with explicit coverage limitations |
| Cost/DORA metrics | Outcome/evaluation | Evaluate downstream impact, not just token/latency proxies |
| Cells, ABAC, sandbox modules | Isolation, policy, execution environment | Make boundaries explicit and enforceable |
| Swappable modules and external CI | Provider-agnostic architecture | Integrate existing tools without forcing replacement |

## What this changes — and does not change — in EOKS

OpenChoreo strengthens several existing EOKS hypotheses:
1. A platform's semantic domain model can be the common interface across UI, CLI, GitOps, APIs, and agents.
2. Context quality depends on the evidence model and the path used to obtain evidence, not only on prompt construction.
3. Read-only diagnosis, proposed actions, authorization, execution, and verification should be separate stages.
4. Observability and audit should be designed into the platform boundary, not retrofitted as agent logs.
5. Execution backends, infrastructure provisioners, CI systems, and observability stores should remain replaceable behind stable contracts.

It does **not** justify adding a new EOKS dimension, runtime primitive, or general-purpose platform service. These patterns map to existing INTENT / WORKFLOW / CAPABILITIES / KNOWLEDGE / STATE / POLICY and the current resource, control-loop, evidence, evaluation, and assurance models.

## Experiments worth borrowing

1. **Live operational context:** compare raw logs/metrics access with domain-enriched, resource-scoped evidence bundles. Measure task success, omitted evidence, tokens, latency, and rework.
2. **Read/act separation:** compare a single agent with a staged flow of diagnosis → structured proposal → policy check/approval → action → independent verification. Include denied, partial-failure, retry, and rollback cases.
3. **Audit completeness:** compare agent traces with platform audit records; identify actions and reads that cannot be reconstructed and specify the minimum useful provenance contract.
4. **Tool/skill progressive disclosure:** compare full tool catalogs, task allowlists, and on-demand skills on representative operational tasks. Measure correctness and total cost, not just initial prompt size.
5. **Outcome metrics:** link agent-supported operations to verified service outcomes, deployment metrics, recovery, and spend, while controlling for task difficulty and environment changes.

These are candidate experiments, not current architecture changes. Use EOKS's existing evaluation methodology and retain raw traces and task-level outcomes.

## Source register

- [OpenChoreo repository](https://github.com/openchoreo/openchoreo)
- [Architecture docs](https://openchoreo.dev/docs/overview/architecture/)
- [AI overview](https://openchoreo.dev/docs/ai/overview/)
- [MCP servers](https://openchoreo.dev/docs/reference/mcp-servers/)
- [v1.1 release: ABAC, skills, FinOps, agent sandbox](https://openchoreo.dev/blog/openchoreo-v1-1-release/)
- [v1.3 release: cost insights, DORA, audit/platform logs, local dependency tunnel](https://openchoreo.dev/blog/openchoreo-v1.3-release/)
- [Audit logging](https://openchoreo.dev/docs/platform-engineer-guide/audit-logging/)
- [Crossplane resource provisioning](https://openchoreo.dev/blog/provisioning-resources-with-crossplane/)
- [FinOps Agent](https://openchoreo.dev/docs/ai/finops-agent/)
- [SRE agent implementation](https://github.com/openchoreo/openchoreo/tree/main/agents/sre-agent)
- [FinOps agent implementation](https://github.com/openchoreo/openchoreo/tree/main/agents/finops-agent)
- [Portal Assistant implementation](https://github.com/openchoreo/openchoreo/tree/main/agents/portal-assistant)
