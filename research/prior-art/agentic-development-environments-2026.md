# Agentic development environments: implementation prior art (2026)

This note examines concrete implementations of **agentic development environments (ADEs)** rather than treating ADE as another name for AI-assisted IDEs or AI-native SDLC.

The useful distinction is:

- **AI-native / agentic SDLC** — the lifecycle and operating model for software development when agents perform substantial work.
- **ADE** — the developer-facing environment that coordinates tasks, workspaces, agents, sessions, artifacts and human control.
- **EOKS** — the broader semantic/control layer hypothesized to coordinate resources, context, policy, execution, evidence and learning across replaceable environments and runtimes.

The terminology is still inconsistent. This note therefore focuses on recurring implementation boundaries rather than claiming a settled industry ontology.

## Executive synthesis

Concrete ADEs converge on a surprisingly small set of durable concepts:

```text
Work / Task
    |
    +-- Workspace
    |
    +-- Context / Loadout
    |
    +-- Agent / Provider
    |
    +-- Run / Session
    |
    +-- Artifacts / Diff / Tests
    |
    +-- Review / Human control
    |
    +-- Outcome
```

The most important observation is that **the agent process is not the durable unit of work**.

Emdash, OpenADE and Warp each approach this differently, but the same separation appears:

- work has a durable identity;
- execution happens in a replaceable runtime;
- provider-specific sessions are continuation handles, not the identity of the work;
- workspaces are managed execution resources;
- artifacts and verification provide evidence;
- the developer environment remains a control surface.

This is strong external support for EOKS's existing distinction between **Workload, Run, Agent, Execution Environment and logical execution state**. It does not require new EOKS runtime primitives.

## 1. Emdash

**Source:** https://github.com/generalaction/emdash

Emdash is the strongest implementation reference in this pass because its architecture is explicit and its object model is durable rather than being primarily a UI abstraction.

A task is associated with a workspace. The workspace may be a repository root or Git worktree and may be local or remote. Conversations have provider-specific session identifiers that allow continuation with an agent provider.

The resulting model is approximately:

```text
Project
  |
  +-- Task
       |
       +-- Workspace
       |     +-- local / remote
       |     +-- repository / worktree
       |     +-- desired/provisioned state
       |
       +-- Conversation
       |     +-- provider
       |     +-- provider session handle
       |
       +-- artifacts / Git / PR state
       |
       +-- automation runs
```

### Important implementation boundary

Emdash distinguishes:

- **Task** — durable user-level work;
- **Workspace** — durable execution/filesystem resource;
- **Conversation** — durable interaction record;
- **Provider session** — provider-specific continuation state;
- **Automation run** — one execution of scheduled automation;
- **Runtime** — concrete local/remote process infrastructure.

This is particularly useful for EOKS because it prevents several identities from collapsing into one "agent session".

### Workspace as a managed resource

Emdash goes further than simply creating a worktree.

Its workspace model retains configuration/provenance and observed state such as presence, Git state, creation outcomes, removal outcomes and runtime information. Provisioning can replay a stored creation specification after failure.

The pattern is therefore:

```text
desired workspace
       |
    provision
       |
    observe
       |
 actual state
       |
 reconcile / replay
```

This is directly relevant to EOKS's control-loop thesis. The useful analogy is not "an agent is a pod"; it is:

> **AI work can have desired state, observed state, evidence and reconciliation.**

### EOKS implication

Do not make the agent process the identity of a workload.

Keep these distinct:

```text
Workload / Task
      |
      +-- Run
      |    +-- Agent
      |    +-- Model/provider
      |    +-- Context snapshot
      |    +-- Policy
      |    +-- execution state
      |
      +-- Workspace
      +-- artifacts
      +-- evidence
      +-- outcome
```

A provider session can disappear or be replaced without changing the identity of the work.

## 2. OpenADE

**Source:** https://github.com/bearlyai/OpenADE

OpenADE is smaller than Emdash, but it exposes another useful boundary: an explicit runtime/protocol layer between the environment and agent provider.

Its concepts include task ownership, agent runtime state and an explicit goal/objective associated with a provider/thread.

A useful abstraction is:

```text
Task
  |
  +-- Goal / objective
  |
  +-- Runtime
        |
        +-- provider
        +-- thread/session
        +-- runtime status
```

The important point is again that:

- the **goal** is semantic;
- the **runtime** is execution infrastructure;
- the **provider thread** is a provider-specific mechanism.

This reinforces EOKS's preference for provider-neutral workload/run semantics with adapters around existing agent runtimes.

## 3. Warp

**Sources:**
- https://www.warp.dev/articles/what-is-an-agentic-development-environment
- https://www.warp.dev/newsroom/2026/4/28/warp-open-sources-its-agentic-development-environment
- https://www.warp.dev/articles/the-block-model-behind-warps-agentic-development-environment

Warp's definition is useful because it makes the ADE boundary explicit: the terminal/IDE/CLI is built around coding agents as first-class participants.

Its open-source ADE is paired with **Oz**, Warp's cloud agent orchestration system. The architectural signal is therefore not simply "terminal with AI":

```text
ADE / developer surface
        |
        v
agent orchestration
        |
        v
execution environments
        |
        v
code / tools / infrastructure
```

Warp's block model also illustrates an important human-control property: agent plans, shell commands and other execution events can coexist in a shared chronological surface.

### EOKS implication

An ADE can be a **consumer of a control/runtime layer**, rather than the complete control plane.

This supports treating the ADE as an environment/interface composition rather than adding "ADE" as another EOKS subsystem alongside Context, Knowledge, Policy and Execution.

## 4. ctx

ctx is useful as a complementary implementation because it emphasizes containerized workspaces, agent sessions, artifacts/diffs and review around arbitrary coding agents.

The recurring pattern is:

```text
task
  -> isolated workspace
  -> agent execution
  -> durable transcript/artifacts
  -> review
```

The implementation direction matters more than any particular UI feature: **isolation, bounded autonomy and inspectable artifacts are becoming environment-level concerns**.

This supports EOKS treating execution environment and policy as separable concerns:

- environment determines where/how execution occurs;
- policy determines what that execution may do;
- evaluation determines whether the result is acceptable.

## 5. codemcp / ade

**Source:** https://github.com/codemcp/ade

This project is useful less as an execution runtime and more as a harness/context boundary.

It distinguishes different categories of information supplied to an agent, including project conventions, process knowledge and reference/documentation knowledge.

That maps well onto EOKS's existing distinction between:

- Knowledge;
- Skills/procedures;
- Context;
- Execution state.

The important conclusion is not to copy its information architecture, but to preserve the distinction between **durable information categories** and the **task-specific context actually delivered to a model**.

## 6. Academic evidence and adjacent research

The implementation landscape is increasingly backed by academic work, although the terminology is not yet standardized. The papers fall into three useful groups: **intelligent development environments**, **AI-native/agentic SDLC**, and **long-horizon agent evaluation**.

### Intelligent development environments

**Marron, "A New Generation of Intelligent Development Environments" (2024).**

This is the closest direct academic precursor to the current ADE terminology. It argues that the IDE should evolve from a tool centered on human code editing into an environment where the developer directs AI agents and automated tools. The proposed environment coordinates communication with agents, organizes workflow from requirements through validation/deployment, and closes the loop with information from the running application.

This is strong evidence for treating the development environment as more than a code editor or chat UI. It also supports the EOKS distinction between the environment/control surface and the underlying agent runtime. citeturn0academia6

**Hassan et al., "Towards AI-Native Software Engineering (SE 3.0)" (2024/2026).**

This vision describes AI-native software engineering as intent-centric and conversation-oriented, with separate concerns for the developer environment, AI teammates, synthesis and runtime. The proposed stack includes Teammate.next, IDE.next, Compiler.next and Runtime.next.

The useful EOKS signal is the separation of the **development environment from execution/runtime capabilities**, rather than treating the IDE as the whole system. citeturn0search0

**Hu, Kula & Treude, "The Future of Development Environments with AI Foundation Models" (2025).**

A Shonan Meeting report bringing together 33 experts from software engineering, AI and HCI examines how foundation models change human-AI interaction in IDEs. It is useful as broader research-community evidence that the development-environment boundary itself is an active research problem, rather than only an industry product category. citeturn0search2

### AI-native / agentic SDLC

**Bhati, "Agentic AI in the Software Development Lifecycle" (2026).**

This work explicitly frames the transition as a move from code-generation assistance toward delegated repository/feature-level execution under human supervision. It proposes a reference architecture and identifies evaluation, governance, technical debt and human attention as major open problems.

This supports the existing EOKS distinction:

> AI-native SDLC describes the lifecycle and operating model; ADE describes the environment in which the work is performed.

It also strengthens the case for evaluation and governance as first-class parts of the system rather than post-hoc tooling. citeturn0academia8

**Martin & Schwander, "A Lean and Spec-Driven AI-Assisted Software Development Lifecycle" (2026).**

This work studies a lightweight, governed lifecycle using specifications, repository-local guidance, phase-specific skills, bounded autonomy and explicit review/test-oriented handoffs. It provides a concrete academic example of the emerging pattern of **durable engineering artifacts + bounded agent execution + human authority**. citeturn0academia16

### Long-horizon execution and evaluation

**Chen et al., "SWE-CI" (2026).**

SWE-CI evaluates agents over repository histories spanning an average of 233 days and 71 commits, requiring repeated analysis and coding iterations rather than one-shot repair. Its central argument is that static functional-correctness benchmarks miss long-term maintainability.

This is particularly relevant to EOKS's Work → Run → Evidence → Outcome model: the meaningful object of evaluation increasingly includes **sequences of runs and repository evolution**, not only the final response from one agent session. citeturn0academia9

**Huang et al., "DeepSWE" (2026).**

DeepSWE introduces original long-horizon engineering tasks across 91 repositories and uses independent hand-written verifiers rather than relying solely on tests associated with historical reference fixes. The work highlights weaknesses in benchmark setups where an agent can appear successful because the evaluation is too closely coupled to one expected implementation.

This reinforces the distinction between **agent-reported completion, implementation artifacts and independent evidence of outcome**. citeturn0academia11

**Hong et al., "SWE Refactor Bench" (2026).**

This work separates migration completeness from behavioural correctness and adds independent agentic verification. Its results show that passing behavioural checks does not establish that the requested engineering transformation actually happened.

This is a particularly clean example of the EOKS principle that **verification should establish the relevant obligation, not merely produce a passing test result**. citeturn0academia10

### Academic synthesis

Taken together, the literature strengthens three parts of the ADE/EOKS model:

1. **The environment is becoming a distinct architectural layer.** The development environment increasingly coordinates agents, tools, workflows, validation and human interaction rather than merely editing files.
2. **Agentic development is moving toward delegated, long-running work.** Evaluation therefore needs durable work/run identity, repository state and history across iterations.
3. **Outcome evidence must be independent of agent claims.** Tests, audits, static analysis, traces and other evidence need to be related to the obligation being evaluated.

The literature does **not** establish that a separate EOKS-like control plane is necessary. It does, however, provide independent research support for the problem decomposition already used by EOKS.


## 7. Cross-implementation comparison

| Concept | Emdash | OpenADE | Warp | EOKS interpretation |
|---|---|---|---|---|
| Work identity | Task | Task | work/task | Workload / Task |
| Workspace | explicit, local/remote | runtime environment | execution environment | Execution resource |
| Agent | provider-backed conversation | runtime/provider | coding agent | Agent |
| Provider session | explicit resume handle | thread/session | runtime-specific | Runtime detail |
| Run | automation/session lifecycle | runtime state | orchestrated execution | Run |
| Context | project/provider configuration | goal/runtime inputs | agent environment | Context + Loadout |
| Policy | approvals/settings/runtime controls | runtime controls | agent permissions | Policy |
| Artifacts | Git/worktree/PR | runtime outputs | code/workspace | Artifacts |
| Evidence | diff/tests/CI | runtime state | execution output | Evaluation/Evidence |
| Human control | task/session/diff/PR UI | environment UI | shared execution surface | Human interaction |

The mapping is not exact. That is useful: EOKS should remain a semantic model rather than trying to force every implementation into identical objects.

## 8. What ADE implementations add to EOKS

Most of the required concepts already exist in EOKS.

### Already well represented

- Workload / Task
- Run
- Execution Environment
- Agent
- Context
- Loadout
- Policy
- Artifacts
- Evaluation / Evidence
- Outcome
- Reconciliation
- replaceable execution infrastructure

### Worth making more explicit

**1. Work vs Run**

A Task/Work item can have multiple runs:

```text
Task
  +-- Run A -> failed validation
  +-- Run B -> agent changed
  +-- Run C -> accepted
```

A run is therefore an attempt under a particular context, policy and resource configuration, consistent with the existing EOKS terminology.

**2. Provider session vs Run**

Claude thread IDs, Codex rollout IDs, terminal sessions and containers should remain runtime handles. They should not become EOKS identity.

**3. Workspace as a resource with desired/observed state**

This is the clearest implementation-derived reinforcement of the control-loop model.

**4. Human interaction as a capability**

ADEs make approval, steering, review, takeover and inspection part of the execution environment. This strengthens EOKS's existing Human Interaction capability without requiring a new core primitive.

**5. Observability as a control surface**

As multiple runs execute concurrently, the ADE becomes an operational surface for inspecting status, diffs, failures and evidence. Observability therefore serves both debugging and active human control.

## 9. ADE is not AI-native SDLC

The distinction should remain explicit.

### AI-native SDLC

Defines the lifecycle:

```text
intent
 -> specification
 -> plan
 -> implementation
 -> verification
 -> review
 -> deploy
 -> operate
 -> learn
```

### ADE

Provides the environment in which humans and agents perform that work:

```text
task
 -> workspace
 -> agent
 -> run
 -> artifacts
 -> review
 -> outcome
```

An ADE can implement only part of an AI-native SDLC. Conversely, an AI-native SDLC can be implemented across several tools/environments.

### EOKS

Coordinates the resources and control loops underneath/across them:

```text
AI-native SDLC
       |
      ADE
       |
     EOKS
       |
+------+------+------+------+
context knowledge policy execution
       |
 agents / models / workspaces / tools
```

This is a useful conceptual stack, not a mandatory deployment architecture.

## 10. Updated EOKS hypothesis

The implementation evidence strengthens the existing EOKS model without requiring an ADE subsystem.

A concise formulation is:

> **An ADE is a user-facing composition of workloads, workspaces, agents, runs, context, artifacts and human-control capabilities. EOKS is concerned with the provider-neutral control and resource semantics that can coordinate those capabilities across replaceable environments and execution runtimes.**

The strongest common primitive is therefore not "ADE". It is the separation between:

```text
semantic work
     |
     +-- execution attempt (Run)
     |
     +-- execution resources
     |
     +-- provider/runtime handles
     |
     +-- evidence/artifacts
     |
     +-- outcome
```

## 11. What remains unproven

These implementations provide architecture evidence, not outcome evidence.

Open questions remain:

1. Does a provider-neutral control layer improve engineering outcomes over a well-designed ADE?
2. How much semantic state should live above an ADE versus inside it?
3. Can Work/Run/Resource semantics remain simple as execution becomes distributed?
4. What workspace state needs durable reconciliation versus ordinary lifecycle management?
5. How should evidence from IDE/ADE, CI, runtime and production be combined?
6. When should a provider session be resumable, replaceable or intentionally discarded?
7. Does explicit environment composition improve reliability, cost or velocity enough to justify its coordination overhead?

These should remain empirical questions.

## Working conclusion

Concrete ADE implementations make the EOKS boundary clearer:

> **ADE is an environment for performing agentic software work; AI-native SDLC is the lifecycle around that work; EOKS is a candidate control/resource layer for coordinating the underlying capabilities and evidence.**

The strongest new evidence is not a new primitive. It is the repeated separation of **Work, Run, Workspace, Agent, Provider Session and Outcome**, plus the emergence of workspace desired/observed state and reconciliation.

That supports sharpening existing EOKS concepts rather than expanding the ontology.
