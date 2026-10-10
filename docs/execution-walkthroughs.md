# Execution walkthroughs: concrete flows and prompt blocks

This is an implementation-oriented companion to [Agent workflows](agent-workflows.md), [Agent roles](agent-roles.md), and the [EOKS domain model](domain-model.md).

Those documents explain the concepts. This one walks through **what a system could actually do**, which role or mechanism does each part, what gets passed between steps, and when to keep work in one agent versus splitting it across roles or runs.

The prompt blocks below are examples to adapt, not canonical EOKS prompts or a requirement to use a particular agent framework. EOKS models responsibilities and control semantics; existing agents, tools, hooks, CI and workflow engines can implement them.

## Start with one concrete task

Suppose the task is:

> Add an optional request timeout to a Go HTTP client, preserve current behavior by default, and add tests.

The system needs to answer five practical questions:

1. Who decides what work is needed?
2. Who changes the repository?
3. Who checks whether the change is correct?
4. Who decides whether the evidence is sufficient to finish?
5. What happens if something fails?

These are separate responsibilities. They do not automatically require five agents.

## Flow 1 — One agent, one run

Use this for small, well-scoped work when the same context can support implementation and self-checking.

~~~mermaid
flowchart TD
    T[Task + acceptance criteria] --> A[One coding agent]
    A --> C[Inspect code and plan]
    C --> I[Implement]
    I --> V[Run tests and checks]
    V --> E{Acceptance criteria met?}
    E -- Yes --> O[Record artifacts and outcome]
    E -- No --> R[Repair within budget]
    R --> V
    R -- Budget exhausted / ambiguity --> H[Escalate]
~~~

**Who does what**

- The control logic supplies the objective, constraints, permissions and stopping conditions.
- The agent plans, edits and runs available checks.
- Deterministic tools (tests, formatter, type checker, static analysis) produce evidence.
- The controller records the changed files, check results and final status; it does not treat the agent's claim of success as sufficient evidence.

**Example prompt block**

~~~text
ROLE: Implementer

OBJECTIVE
Add an optional request timeout to the Go HTTP client. Preserve existing
behavior when the option is unset.

ACCEPTANCE CRITERIA
- The timeout is configurable through the existing public API style.
- The default preserves current behavior.
- Tests cover configured and default behavior.
- Existing relevant tests pass.

OPERATING RULES
- Inspect the existing client and nearby tests before editing.
- Make the smallest coherent change.
- Do not change unrelated files or public behavior.
- Run the relevant tests and report the exact commands and results.
- If requirements conflict with the existing design, stop and explain the conflict.

RETURN
- Summary of changes
- Files changed
- Tests/checks run and their observed results
- Remaining uncertainty or known gaps
~~~

This is the cheapest topology. Its weakness is correlated error: the same agent that made an assumption may fail to notice that assumption is wrong. Self-review can still help, but it is not independent assurance.

## Flow 2 — Implementer, independent reviewer, deterministic validator

Use this when correctness matters enough to separate construction from challenge and verification.

~~~mermaid
flowchart TD
    T[Task + policy] --> C[Conductor / workflow]
    C --> I[Implementer run]
    I --> A[Patch + implementation report]
    A --> R[Reviewer with fresh context]
    A --> V[Deterministic validation]
    R --> F[Review findings]
    V --> E[Test / analysis evidence]
    F --> D{Action decision}
    E --> D
    D -- Findings or failed checks --> P[Repair run]
    P --> V
    P --> R
    D -- Evidence sufficient --> O[Accept outcome]
    D -- Ambiguous / unsafe --> H[Human escalation]
~~~

The reviewer and validator have different jobs:

- **Implementer:** makes the change.
- **Reviewer:** tries to find defects, missing requirements, compatibility problems and unjustified assumptions.
- **Validator:** runs checks that establish specific properties.
- **Conductor:** correlates the findings, applies policy and decides whether to repair, accept or escalate.

The reviewer can be another agent/session or a human. The validator may be ordinary shell commands and CI; it does not need to be an LLM.

### Prompt block: independent reviewer

Give the reviewer the task, acceptance criteria, patch, and relevant source/tests. Do not require it to trust the implementer's summary. Provide the summary only as optional navigation.

~~~text
ROLE: Independent reviewer

GOAL
Find concrete reasons this patch may be incorrect or incomplete.

INPUTS
- Original task and acceptance criteria
- Patch and changed files
- Relevant source, tests, and project conventions
- Available validation results (if already run)

CHECK
- Requirement coverage and default/backward-compatible behavior
- Edge cases and error handling
- Concurrency/resource lifecycle where relevant
- API and data compatibility
- Missing, weak, or misleading tests
- Unintended scope expansion

RULES
- Do not rewrite the patch.
- Do not report style preferences as correctness defects.
- For each finding, identify the affected code and explain the failure condition.
- Distinguish confirmed defects from risks or unanswered questions.
- If no actionable finding is supported by evidence, say so.

RETURN
- Findings ordered by severity, with file/line references where possible
- Evidence or a minimal scenario that demonstrates each finding
- Unresolved questions
- Review status: findings / no actionable findings / unable to assess
~~~

### Prompt block: validator (when an LLM is involved at all)

Usually this block is an executable check manifest, not a model prompt.

~~~yaml
checks:
  - id: unit-tests
    command: go test ./path/to/client/...
    establishes: relevant client behavior passes the unit tests
    required: true
  - id: formatting
    command: gofmt -w <changed-go-files>
    establishes: changed Go files are formatted
    required: true
  - id: broader-tests
    command: go test ./...
    establishes: broader repository tests pass
    required: policy-dependent
~~~

The commands are illustrative; substitute the repository's real packages and check policy. A successful test run is evidence for the behavior covered by those tests, not proof that every acceptance criterion is satisfied.

### How the controller decides

| Observation | Next action |
|---|---|
| Required check failed | Repair or diagnose; then rerun affected checks |
| Reviewer reports a substantiated defect | Send the finding and evidence to repair |
| Reviewer offers an unsupported preference | Ask for evidence or disregard under the review policy |
| Checks pass but a requirement is untested | Add a check, gather other evidence, or escalate |
| Evidence conflicts | Resolve the conflict; do not average away the disagreement |
| Required evidence is present and policy permits completion | Record outcome and stop |
| Retry/budget limit reached, or a consequential ambiguity remains | Escalate |

The controller owns the decision. Neither the implementer nor reviewer should be able to silently declare the workflow complete.

## Flow 3 — Parallel investigation, one implementation

Use this when there are genuinely independent questions, such as tracing a bug through separate subsystems or comparing multiple plausible causes. Do not parallelize merely because subagents are available.

~~~mermaid
flowchart TD
    T[Task / question] --> C[Conductor defines bounded investigations]
    C --> A[Investigator A: call path]
    C --> B[Investigator B: tests and history]
    C --> D[Investigator C: competing hypothesis]
    A --> M[Reduce and reconcile evidence]
    B --> M
    D --> M
    M --> Q{Enough evidence to choose?}
    Q -- No --> X[Targeted follow-up or escalate]
    Q -- Yes --> I[One implementation run]
    I --> V[Validate and review]
    V --> O[Outcome]
~~~

### Prompt block: bounded investigator

~~~text
ROLE: Investigator

QUESTION
Determine whether the HTTP client timeout can be added without changing
the default request lifecycle.

SCOPE
Inspect the client implementation, its callers, tests, and relevant history.
Do not edit files.

DELIVER
- Findings backed by file paths, code references, tests, or commits
- A concrete answer to the question
- Alternative explanations and contradictory evidence
- What you did not inspect
- The next smallest investigation if the answer remains uncertain

BOUNDARY
Do not propose a broad redesign unless the evidence shows the existing design
cannot satisfy the requirement.
~~~

The conductor should combine the evidence, preserve disagreements and provenance, then decide whether another investigation is worth its cost. Investigators should return findings, not long transcripts. The implementation run should receive the reduced evidence set and authoritative source references, not every worker's raw conversation.

## Flow 4 — Plan, execute, observe, re-plan

Use this when the task is long-running, has uncertain intermediate results, or depends on external activities such as CI, a deployment, a remote job or a human response.

~~~mermaid
flowchart TD
    S[Current durable state + policy] --> P[Planner proposes next steps]
    P --> G[Controller checks permissions, preconditions and budget]
    G --> X[Execute one action / start activity]
    X --> O[Observe result or progress]
    O --> U[Update durable run state and evidence]
    U --> E{Progress and evidence sufficient?}
    E -- Continue as planned --> P
    E -- Assumptions invalidated --> R[Re-plan]
    R --> G
    E -- Retryable failure --> F[Retry / alternate mechanism]
    F --> G
    E -- Accepted --> D[Record outcome]
    E -- Unsafe, blocked, or budget exhausted --> H[Escalate]
~~~

Here the planner and controller are explicitly different:

- **Planner:** proposes a plan from the current information.
- **Planner/conductor boundary:** the controller decides whether the proposed action is allowed and appropriate now.
- **Executor:** performs the action.
- **Observer:** obtains the actual result (often a tool, CI system, or event).
- **Evaluator:** checks whether the result meets the required condition.

The plan is disposable. If a test failure invalidates the next planned step, the controller updates state and asks for a new plan rather than continuing from stale assumptions.

A durable run record needs only enough information to reconstruct control: current step, inputs/revisions, decisions, started/completed activities, artifacts, observations, retries, approvals, budget and outstanding conditions. Provider session memory is not the source of truth.

## Flow 5 — Construct, attack, verify, decide

Use this for high-consequence changes, architecture decisions or security-sensitive work where a normal review may share too many assumptions with the implementation.

~~~mermaid
flowchart TD
    A[Artifact + claims + acceptance criteria] --> S[Support path: construct solution]
    A --> C[Challenge path: try to falsify claims]
    S --> V[Adjudicate concrete claims]
    C --> V
    V --> D[Deterministic / authoritative verification]
    D --> E[Evaluate evidence against policy]
    E --> R{Sufficient assurance?}
    R -- Yes --> O[Accept]
    R -- No, repairable --> P[Revise artifact and repeat relevant checks]
    P --> A
    R -- No, consequential uncertainty --> H[Human / stronger authority]
~~~

The challenge path should search for counterexamples, missing invariants, failure conditions and contradictions. It should not simply be asked whether it agrees. Whenever possible, validate the challenge with tests, static analysis, authoritative sources or a reproducible scenario.

**Important:** two agents are not automatically two independent evidence paths. Shared prompts, context, assumptions and models can correlate their errors. The system should evaluate whether the added challenge improves outcomes enough to justify its cost.

## Choosing a flow

| Work characteristics | Start with | Add separation when… |
|---|---|---|
| Small, clear, reversible | One agent + checks | Failures repeatedly escape self-checks |
| Moderate change with meaningful correctness risk | Implementer + reviewer + validator | Existing review is insufficient or risks are more consequential |
| Independent unknowns | Parallel investigation, then one implementation | Evidence tasks can truly proceed independently |
| Long-running / external side effects | Durable plan-execute-observe loop | Progress cannot be reconstructed from a single session |
| High-consequence or hard-to-test claims | Construct + challenge + authoritative verification | The cost of a missed defect justifies extra assurance |

Start simple. Add a role, run, checkpoint, approval gate or parallel branch only when it changes who can decide, what evidence is available, what can safely happen, or how recovery works.

## The handoff contract

A role name alone is not enough to make a workflow executable. Every handoff should define a compact contract. This can be structured data, a task card, a message or a prompt section; EOKS does not require one format.

~~~yaml
handoff:
  task: "Add optional request timeout"
  role: "reviewer"
  objective: "Find correctness and compatibility defects"
  inputs:
    - artifact: "git diff"
      revision: "working-tree revision or commit"
    - artifact: "acceptance criteria"
  scope:
    inspect:
      - client implementation
      - callers
      - relevant tests
    may_edit: false
  constraints:
    - "Do not trust implementer claims without checking evidence"
  output:
    - "actionable findings with evidence"
    - "unresolved questions"
  completion_condition: "review findings or explicit unable-to-assess result"
~~~

A useful handoff answers:

- **Objective:** what question or result belongs to this step?
- **Inputs:** which artifacts, revisions and evidence are authoritative?
- **Scope:** what should be inspected or changed?
- **Authority:** which tools and side effects are allowed?
- **Output:** what artifact, evidence or decision must be returned?
- **Completion:** what observable condition means this step is finished?
- **Failure:** what happens if it cannot finish or its assumptions are false?

This makes the flow portable across different agents and execution providers. A framework can translate the contract into system prompts, tool permissions, job payloads, CI steps or workflow state.

## What should be recorded between steps?

Avoid copying the whole transcript from one agent to the next. Preserve the information needed for the next decision and for later reconstruction.

| Record | Example | Why it matters |
|---|---|---|
| Work identity and acceptance criteria | Task ID, required behavior | Keeps all runs aimed at the same outcome |
| Run/step state | Started, blocked, failed, complete | Enables resume and control decisions |
| Context manifest | Selected sources and revisions | Explains what the agent could know |
| Artifact | Patch, report, test output | Gives the next role something concrete to inspect |
| Evidence | Exact check result, reproduction, source reference | Supports evaluation beyond self-report |
| Decision | Retry, repair, accept, escalate + rationale | Makes control choices inspectable |
| Policy/budget | Allowed commands, retry and cost limits | Constrains autonomy |
| Outcome | Accepted, incomplete, rejected, escalated | Enables end-to-end evaluation and learning |

A summary can be useful, but it is a derived navigation aid, not a substitute for source artifacts and evidence when those are available.

## The implementation boundary

A minimal implementation does not need a new agent platform. It can be assembled from an existing coding agent, prompts or role contracts, shell/CI checks, a small durable run record, and a controller that interprets results.

~~~text
Task + Policy
     |
     v
Controller / workflow state
     |
     +--> assemble role-specific context
     |
     +--> invoke existing agent or deterministic tool
     |
     +--> collect artifact + evidence
     |
     +--> evaluate against acceptance criteria
     |
     +--> continue / repair / re-plan / stop / escalate
~~~

The implementation can begin as a script or lightweight workflow. Introduce durable workflow infrastructure, multiple concurrent agents, sandbox orchestration or learned topology selection only when the workload needs those capabilities.

## How this fits the EOKS model

These walkthroughs instantiate existing concepts rather than add new primitives:

- **Task:** durable identity and objective of the work.
- **Run:** one attempt or bounded execution of a task/step.
- **Context:** the task-specific information supplied to a reasoning step.
- **Workflow and roles:** sequence/dependencies and responsibilities.
- **Resources/capabilities:** agents, models, tools, CI and evidence providers.
- **Policy:** permissions, constraints, required assurance and budgets.
- **Decision:** continue, retry, branch, re-plan, accept or escalate.
- **Evaluation and evidence:** whether the result meets the required conditions.
- **Outcome:** what happened, including artifacts and unresolved gaps.

The practical design question is therefore not “Which agent framework should EOKS copy?” It is:

> Given a task, what is the smallest execution flow that assigns each responsibility, passes the right artifacts and evidence between steps, enforces authority, and knows when to continue, recover, stop or ask for help?

That is the point at which EOKS's conceptual model becomes an executable design.
