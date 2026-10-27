# Agent execution, sandboxing and policy enforcement: cross-project comparison (2026)

## Purpose

This note compares the agent-execution and isolation projects discussed alongside NVIDIA OpenShell, with the question EOKS can usefully answer:

**Can EOKS describe a Work's requirements and desired outcome independently of the execution provider, while retaining enough evidence to know what actually ran, what it could access, and what guarantees were enforced?**

This is comparative prior art, not a new EOKS architecture proposal. EOKS already separates workload reconciliation, policy, resources/context, evidence/evaluation, and replaceable execution. The aim is to pressure-test those boundaries against real implementations—not to create a sandbox dimension, standardize every provider prematurely, or add vendor dependencies.

Scope and status are based on public documentation available on 2026-10-27. Product capabilities and preview status change quickly; links below are the primary references to re-check before implementation.

## Executive synthesis

The projects are related but not interchangeable:

- **Google Agent Compute Engine (ACE)** is principally an agent-oriented compute substrate: fast-starting, hardware-isolated microVMs and lifecycle optimized for agents that start, pause, and resume.
- **Google Agent Executor (AX)** is principally a declarative agent-execution/orchestration runtime: Task, Workspace, Gateway and Model over a stateful execution substrate. EOKS already has a separate [AX prior-art note](agent-executor-ax.md).
- **NVIDIA OpenShell** emphasizes a trusted gateway/supervisor boundary, credential mediation, policy delivery/enforcement, and a policy prover for proposed policy changes.
- **AWS Strands Box + Dogwood** combines OS-level containment with action-level policies that can consider shared event history across supported interpreters and gateways.
- **Microsoft MXC** exposes a common typed/configuration model over multiple OS- and VM-based containment backends. This is a strong portability example, but common configuration is not proof of equivalent enforcement.
- **OpenSandbox** focuses on a common sandbox API/SDK and integrations across execution backends and agent workflows.
- **OpenAI Agents API / Sandbox Agents** separates the agent harness/session from its execution environment, with both managed and self-hosted options depending on the integration.
- **Anthropic's Claude/Cowork isolation patterns** show different trade-offs for ephemeral cloud sandboxes, local execution, egress restrictions, and server-side connector/credential handling.
- **Docker and Cloudflare sandbox patterns** are useful implementation references: Docker for practical agent workspaces, Cloudflare for keeping credentials in a trusted broker rather than handing secrets to an agent process.

These are different layers of the problem, not eight competing implementations of the same abstraction.

## Comparison matrix

| Project | Main layer / contribution | What is concretely interesting | Important boundary or caveat |
|---|---|---|---|
| Google ACE | Compute substrate | Agent-oriented microVM lifecycle; isolation; rapid start/pause/resume at scale | A compute product is not, by itself, EOKS's workload control loop or canonical policy authority |
| Google AX | Execution runtime/orchestration | Task, Workspace, Gateway, Model; durable state and suspend/resume | Keep separate from ACE: lifecycle/API and underlying compute are related but distinct |
| NVIDIA OpenShell | Trusted runtime and policy boundary | Gateway/Supervisor split, credential mediation, policy updates, Policy Prover, audit events | Prover coverage is bounded; verification is not authorization, enforcement, or proof of total safety |
| AWS Strands Box / Dogwood | Containment + semantic action policy | Default-deny action checks; policy can use prior events/time; common policy engine across supported gateways/interpreters | Only operations routed through its enforcement points get those checks; direct OS grants have different visibility |
| Microsoft MXC | Portable containment SDK | Typed policy/configuration over OS and VM backends; lifecycle and diagnostics | Backend-dependent features and strength must remain explicit |
| OpenSandbox | Provider API / sandbox platform | Unified API/SDK and adapters for Docker/Kubernetes-style environments and agent workflows | API compatibility alone does not establish a security-equivalence contract |
| OpenAI Agents API | Managed harness + sessions + optional environment | Durable sessions, orchestration/recovery, events; environment can be OpenAI-hosted or self-hosted | Harness ownership and compute ownership are separate deployment choices |
| OpenAI Sandbox Agents | Environment abstraction for application-owned harness | Container-like filesystem, commands, packages, ports, snapshots and resumable state | Do not conflate with the managed-harness Agents API; they put ownership in different places |
| Anthropic Cowork / Claude isolation | Product-level cloud/local execution security | Temporary cloud sandbox, outbound proxy, private-address restrictions, short-lived credentials; local VM pattern | Product-specific controls; local integrations and observability differ by execution mode |
| Docker Sandboxes | Practical isolated developer/agent environments | Familiar workspace/container workflows and integrations with coding agents | The exact isolation boundary depends on product/runtime configuration; don't infer VM-grade isolation from the word “sandbox” |
| Cloudflare sandbox/credential pattern | Brokered access and secret boundary | Trusted server-side component can keep credentials out of the agent environment and mediate requests | A credential broker does not replace filesystem/process isolation or workload-level authorization |

## Project notes

### Google: Agent Compute Engine and Agent Executor are complementary, not duplicates

**ACE** is the infrastructure-level story. Google's announcement describes a purpose-built compute service for agents, using hardware-isolated microVMs and lifecycle behavior designed around agents that repeatedly start, pause, and resume. Its relevance to EOKS is the compute boundary and the operational cost of durable, high-churn execution.

**AX** is the runtime/API-level story. Its Task, Workspace, Gateway and Model primitives describe the agent workload, environment composition, access/network boundary and model configuration, while Agent Substrate handles stateful sandboxed execution and checkpoint/resume.

Together, these suggest a useful distinction already compatible with EOKS:

- A logical Work/Run has an identity, requirements, policy and acceptance criteria.
- An execution provider realizes it using a particular runtime and compute backend.
- A workspace/loadout composes the repository, tools, skills and other environment resources.
- Suspend/resume and migration are lifecycle transitions that must preserve logical identity without blindly assuming that a new runtime has the same effective policy.

EOKS already documents AX in [Google AX / Agent Executor](agent-executor-ax.md). This note should link to that analysis rather than repeat its full architecture review.

Sources: [Google ACE announcement](https://cloud.google.com/blog/topics/ai-infrastructure/introducing-agent-compute-engine-and-expanded-agent-governance), [ACE technical deep dive](https://cloud.google.com/blog/topics/ai-infrastructure/agent-compute-engine-deep-dive), [Google AX](https://agentexecutor.io/).

### NVIDIA OpenShell: policy, enforcement and verification

OpenShell is the focal point of the companion note, [NVIDIA OpenShell: policy enforcement and verification](openshell-policy-enforcement-2026.md). The most relevant concepts are the split between gateway/control-plane state and local trusted enforcement, credential mediation, policy updates, and checking proposed policy changes before approval.

For EOKS, preserve separate claims for policy verification, authorization, effective policy, runtime enforcement and evidence. A successful prover result only supports the property it checked over the constructs it understands.

Sources: [OpenShell architecture](https://docs.nvidia.com/openshell/latest/about/how-it-works), [Policy Prover](https://docs.nvidia.com/openshell/latest/how-it-works/policies/prover), [OpenShell repository](https://github.com/NVIDIA/OpenShell).

### AWS Strands Box and Dogwood: policies can depend on action history

Strands Box combines OS-level isolation with a semantic policy engine. The policy layer can inspect an operation and shared history across supported enforcement points; rules may depend on earlier operations, order or accumulated rates. For example, an earlier file read can influence whether a later outbound request is permitted.

This is a distinct idea from static filesystem/network allowlists: **authorization may depend on the trajectory of the workload**, not only on the current operation's arguments. It is relevant to EOKS's event/evidence and policy boundaries, especially when the same logical activity crosses shell, Python, HTTP and MCP.

The caveat is coverage. The Box documentation distinguishes direct OS grants from operations routed through its interpreters and gateways. An EOKS evaluation must record which operations actually pass through policy enforcement; it must not assume every action is visible to the semantic policy engine.

Sources: [AWS Open Source announcement](https://aws.amazon.com/blogs/opensource/introducing-strands-box-ai-agent-sandboxes-powered-by-dogwood/), [Strands Box repository](https://github.com/strands-agents/box), [Dogwood design decisions](https://github.com/strands-agents/box/blob/main/docs/design/decisions.md).

### Microsoft MXC: one configuration, multiple containment backends

MXC offers a typed SDK and policy/configuration model across platform-appropriate backends, including OS-native process sandboxes and VM-oriented options. Its value for EOKS is the concrete provider-adapter problem: a caller can request a containment configuration without hand-writing each backend's launch details.

The key lesson is **normalize the request, not away the guarantees**. If backend A supports a network restriction and backend B cannot enforce it, EOKS must see that as an unsupported or weaker guarantee—not a successful application of identical policy. Backend-specific behavior, experimental status and diagnostics should remain inspectable.

Sources: [Microsoft MXC repository](https://github.com/microsoft/mxc).

### OpenSandbox: common interface, not automatic security equivalence

OpenSandbox is an open sandbox platform with APIs/SDKs and integrations intended to make isolated execution easier to use across environments and agent workflows. It is useful prior art for provider integration and lifecycle APIs.

It is not a replacement for EOKS's policy/evidence semantics. A common sandbox create/execute API tells a caller how to ask for execution; EOKS still needs to know what security properties the chosen backend actually provides, which policy features are supported, how secrets are mediated, and what evidence can be collected.

Sources: [OpenSandbox project](https://github.com/opensandbox-group/OpenSandbox), [project website](https://open-sandbox.ai/), [Agentic AI Foundation announcement](https://www.linuxfoundation.org/press/agentic-ai-foundation-announces-four-new-projects-agent-skills-open-responses-openenv-and-opensandbox).

### OpenAI: separate harness ownership from environment ownership

OpenAI documents two related but distinct options:

- **Agents API:** an OpenAI-managed Codex harness that handles sessions, orchestration, context compaction and recovery; the application supplies the task, tools and execution-environment choice.
- **Sandbox Agents:** a sandbox/environment model for applications that run their own harness, including files, commands, packages, ports, snapshots and resumable state.

This is useful because it makes ownership boundaries explicit. A system may delegate harness/session management to a vendor while retaining control of the environment, or own the harness while using a managed sandbox. EOKS should describe the chosen boundary rather than assume “agent provider” and “compute provider” are one thing.

Sources: [Agents API overview](https://developers.openai.com/api/docs/guides/agents-api/overview), [Sandbox Agents](https://developers.openai.com/api/docs/guides/agents/sandboxes).

### Anthropic: isolation depends on execution mode

Anthropic's public material distinguishes cloud execution from local execution. Its Cowork cloud sessions use temporary isolated sandboxes, outbound traffic is mediated by a proxy, private/internal destinations are restricted, and connector credentials are handled server-side rather than placed directly in the sandbox. Local execution has a different trust and observability boundary, including VM-based code execution in the documented desktop architecture.

This is useful not because EOKS should copy a product architecture, but because **execution mode changes the threat model**. “Local” versus “cloud” is not just a latency/cost choice; it changes where data is processed, which network paths exist, which local tools can be reached, and what monitoring can observe.

Sources: [Claude Cowork architecture overview](https://support.claude.com/en/articles/14479288-claude-cowork-architecture-overview), [How Anthropic contains Claude](https://www.anthropic.com/research/how-we-contain-claude).

### Docker and Cloudflare: implementation patterns, not complete control planes

Docker's sandbox tooling is useful as a practical environment option for coding agents. Cloudflare's sandbox and brokered-credential patterns are useful for understanding how to keep secrets in a trusted component and mediate external requests rather than expose long-lived credentials to an agent.

These patterns should be compared at the capability level. A brokered credential path does not establish process isolation; a container does not automatically provide all the policy, audit, or tenant-isolation guarantees EOKS might require.

Sources: [Docker Sandboxes](https://docs.docker.com/ai/sandboxes/), [Cloudflare Sandbox SDK](https://developers.cloudflare.com/sandbox/).

## Mapping to existing EOKS concepts

| EOKS concern | Evidence from these projects | What EOKS should preserve |
|---|---|---|
| Work/Run identity and lifecycle | AX tasks; ACE start/pause/resume; OpenAI sessions/snapshots | Logical workload identity should outlive an individual process/container/VM |
| Environment/loadout | AX Workspace; OpenAI environment; Docker/OpenSandbox integrations | Compose repository, tools, skills, credentials access and runtime configuration without conflating them with the agent |
| Replaceable execution | ACE, MXC, OpenSandbox, OpenAI environment options | Provider adapters should expose capabilities and limitations; don't promise identical guarantees |
| Policy vs enforcement | OpenShell, Dogwood, MXC | Policy intent/authorization and actual runtime enforcement are distinct |
| Stateful/trajectory policy | Dogwood shared event history | If supported, action history and policy-state dependencies must be explicit and testable |
| Credential authority | OpenShell, Strands Box, Anthropic, Cloudflare | Prefer mediated, scoped credentials; distinguish logical access from secret material |
| Evidence and assurance | OpenShell logs, Dogwood decision records, product telemetry | Record provenance, enforcement point, policy revision, coverage, delivery gaps and retention limits |
| Resume/migration | ACE/AX/OpenAI durable execution patterns | Revalidate the target runtime's effective policy before resuming under required guarantees |
| Acceptance and learning | Not the primary focus of these execution projects | EOKS still owns workload-level evaluation, acceptance criteria, reconciliation and learning from outcomes |

## Proposed provider-neutral contract questions

These are questions for existing EOKS policy, execution, resource and evidence contracts—not a proposal to create new core primitives.

1. **Capability declaration:** Can a provider report supported controls, unsupported controls, enforcement location, strength/limitations, update semantics and backend/runtime identity?
2. **Requested vs effective policy:** Can a Run distinguish what was requested, what was authorized, what the provider claims to have applied, and what independent evidence confirms?
3. **Fail-closed behavior:** If a required control is unsupported, unavailable, or cannot be verified, does execution stop, degrade only with explicit authorization, or continue under a recorded weaker assurance level?
4. **Policy history:** For stateful policies, what event history is authoritative, how is it ordered, and how do missing events affect decisions?
5. **Delegation:** Does a child Run's effective authority remain within the parent's explicitly delegated boundary unless a separate authorization changes it?
6. **Resume/migration:** Before resuming on a different backend, which isolation and policy requirements must be re-established, and what evidence is sufficient?
7. **Telemetry quality:** How are missing, delayed, dropped, duplicated or schema-incompatible events represented without treating absence as proof of absence?
8. **Credential lifecycle:** Are credentials scoped to the run/session, kept outside model-visible context where possible, and revoked or rotated on completion, interruption or migration?

A provider may answer “unsupported” for a control. That is useful information; silently claiming uniform support is not.

## A small experiment that discriminates between the approaches

Do not begin with a broad benchmark of every vendor. Choose two available execution providers with materially different isolation/ownership models, and run the same bounded repository task with identical acceptance criteria.

### Test cases

- Read/write only within an allowed workspace; attempt access outside it.
- Allow a declared network destination; deny an undeclared destination.
- Attempt to read an injected credential and to use the credential through an approved broker.
- Propose a permission expansion and verify that it requires the intended authorization.
- Spawn a child execution and test whether it can exceed the delegated boundary.
- Interrupt and resume the same logical Work; if possible, resume on a different backend.
- Simulate missing/delayed runtime events and determine whether EOKS overstates assurance.
- Run an action through each relevant interface (shell, language runtime, HTTP gateway, MCP) to identify enforcement coverage gaps.

### Record

For each attempt, capture:

- logical Work/Run ID and execution provider/runtime identity;
- requested, authorized and effective policy revisions;
- control requested, provider support status and enforcement point;
- attempted operation and allow/deny result;
- independent verification result and its coverage;
- event provenance, collection/retention status and known gaps;
- resume/migration transitions;
- task outcome, latency/setup cost, human interventions and recovery effort.

### Success criterion

The experiment succeeds if EOKS can distinguish **“task completed”**, **“policy was requested”**, **“policy was applied”**, and **“the available evidence supports that the policy was enforced”**. It should expose unsupported controls and evidence gaps rather than turn them into a generic secure/unsafe boolean.

Do not compare vendors on marketing labels like “sandbox” or assume that the strongest isolation backend is always the best choice. Measure guarantees, usability, task success, cost and operational recovery separately.

## Conclusions

1. The existing EOKS separation between workload/control loop and execution substrate is consistent with the direction of these implementations.
2. The most useful new pressure is on the **provider capability/evidence contract**: policy feature support, enforcement coverage, effective-policy confirmation, credential mediation and assurance after resume/migration.
3. Google ACE and AX should be read together but not collapsed into one project: compute substrate and durable execution API are distinct concerns.
4. Strands Box adds a particularly interesting trajectory-aware policy model; MXC and OpenSandbox sharpen the portability question; OpenAI and Anthropic illustrate different ownership and execution-mode boundaries.
5. None of this justifies a new EOKS dimension or a hard dependency on a particular vendor. Promote a contract change into normative docs only after an experiment or a concrete implementation need demonstrates it.

## Primary sources

- Google, [Agent Compute Engine announcement](https://cloud.google.com/blog/topics/ai-infrastructure/introducing-agent-compute-engine-and-expanded-agent-governance).
- Google, [Agent Compute Engine technical deep dive](https://cloud.google.com/blog/topics/ai-infrastructure/agent-compute-engine-deep-dive).
- Google, [Agent Executor](https://agentexecutor.io/).
- NVIDIA, [OpenShell architecture](https://docs.nvidia.com/openshell/latest/about/how-it-works), [Policy Prover](https://docs.nvidia.com/openshell/latest/how-it-works/policies/prover), [repository](https://github.com/NVIDIA/OpenShell).
- AWS, [Strands Box announcement](https://aws.amazon.com/blogs/opensource/introducing-strands-box-ai-agent-sandboxes-powered-by-dogwood/), [repository](https://github.com/strands-agents/box).
- Microsoft, [MXC repository](https://github.com/microsoft/mxc).
- Linux Foundation / AAIF, [OpenSandbox announcement](https://www.linuxfoundation.org/press/agentic-ai-foundation-announces-four-new-projects-agent-skills-open-responses-openenv-and-opensandbox), [repository](https://github.com/opensandbox-group/OpenSandbox).
- OpenAI, [Agents API overview](https://developers.openai.com/api/docs/guides/agents-api/overview), [Sandbox Agents](https://developers.openai.com/api/docs/guides/agents/sandboxes).
- Anthropic, [Cowork architecture overview](https://support.claude.com/en/articles/14479288-claude-cowork-architecture-overview), [containment patterns](https://www.anthropic.com/research/how-we-contain-claude).
- Docker, [Sandboxes](https://docs.docker.com/ai/sandboxes/).
- Cloudflare, [Sandbox SDK](https://developers.cloudflare.com/sandbox/).

This is research/prior art. It does not establish a normative EOKS contract, a vendor ranking, or a guarantee that the listed products support identical controls.
