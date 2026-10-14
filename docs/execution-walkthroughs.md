# Execution walkthroughs: small examples of who does what

This is a practical companion to [Agent workflows](agent-workflows.md), [Agent roles](agent-roles.md), and [EOKS architecture](architecture.md). It shows how the same task changes when we change the execution mechanism.

The examples are intentionally small. They are **prompt fragments**, not universal templates: use only the instruction that solves a real problem. The workflow, policy, tools and evidence around a prompt matter just as much as its wording.

## One task, several execution designs

Example task: add an optional timeout to a Go HTTP client, preserving the current default behavior.

### 1. One agent: make the change

~~~yaml
role: implementer
goal: Add an optional timeout to the Go HTTP client.
context:
  - Inspect the existing client, callers, and tests before editing.
constraints:
  - Preserve current behavior when the timeout is unset.
  - Make the smallest maintainable change.
steps:
  - Implement the change and add focused tests.
  - Run relevant checks.
return:
  - Summary of the change
  - Checks run and their results
  - Remaining uncertainty or follow-up
done_when: Required tests pass, or a concrete blocker is reported.
~~~

**Mechanism:** one agent inspects, edits and checks its work.

**Use when:** the task is clear, bounded and easy to verify.

**Limitation:** the agent may miss an assumption it made while implementing.

### 2. Add an independent reviewer

Keep the implementation prompt above. Give a second run the patch and the original task:

~~~yaml
role: independent reviewer
goal: Find defects in the patch relative to the original task.
inputs: Original task, diff, relevant code and test results.
focus:
  - Default behavior and compatibility
  - Edge cases and failure paths
constraints:
  - Do not edit or rewrite the patch.
  - Report actionable findings only.
return:
  - Finding, evidence, and impact for each confirmed defect
  - Open questions separately
done_when: The patch has been reviewed; findings are evidence-backed.
~~~

**What changed:** not a longer implementer prompt, but a second perspective with a different job and fresh context.

**Use when:** missed defects are costly enough to justify another run.

**Important:** a reviewer is not a validator. A reviewer reasons about possible defects; tests and other checks produce evidence about the properties they cover.

### 3. Investigate before implementing

Instead of asking several agents to implement the same feature, give them separate, bounded questions:

~~~yaml
role: investigator
question: How do request timeouts and cancellation work today?
scope:
  - Trace the relevant call path and inspect related tests/callers.
constraints:
  - Do not edit files.
  - Separate observed facts from assumptions.
return:
  - Relevant files and call path
  - Evidence for current behavior
  - Uncertainties that could change the implementation
done_when: The question is answered with references to evidence, or the remaining gap is explicit.
~~~

Another investigator might inspect tests and callers. A conductor combines the findings, resolves contradictions, then gives the implementation run a short evidence-backed brief.

**Use when:** the main uncertainty is *what is true about the system*, not how to write the code.

**Why not always parallelize?** Parallel work costs time and tokens and creates a synthesis task. Split only when questions can genuinely be investigated independently.

### 4. Make completion explicit for unattended work

A task can span multiple turns, CI runs or background jobs. A final-sounding progress message must not be confused with a verified outcome.

~~~yaml
role: unattended execution agent
goal: Complete the task against its acceptance criteria.
state:
  - Read the persisted checklist before acting.
  - Update each item from observed tool or check results.
continue_while:
  - Required criteria remain unmet.
  - No blocking dependency or configured limit has been reached.
stop_when:
  - All required criteria have evidence, or
  - A blocker, approval requirement, or limit prevents progress.
return:
  - Checklist with status and evidence
  - Blocker and next required action, if blocked
constraint: Do not report completion based only on a progress message or turn ending.
~~~

This prompt helps communicate desired behavior, but it does **not** enforce it by itself. The harness must preserve task state, inspect tool/job results, apply retry and time limits, and decide whether completion criteria are satisfied.

**Use when:** the system must continue without a person prompting each next step.

### 5. Add a hard boundary around actions

A prompt can state the boundary, but permissions should enforce it:

~~~yaml
role: read-only investigator
goal: Propose a change and gather evidence for a human decision.
authority:
  allowed: Read files and run non-mutating inspection commands.
  forbidden: Modify files or perform external side effects.
return:
  - Proposed patch or concrete change outline
  - Supporting evidence
  - Decisions a human must make
enforcement: Use read-only tools or a sandbox that prevents writes.
~~~

**Mechanism:** pair the instruction with read-only tools or sandbox permissions. Do not rely on the model remembering a sentence when the tool itself can enforce the restriction.

**Use when:** a step should investigate, review or propose without performing side effects.

## The flow is more than the prompt

~~~mermaid
flowchart TD
    T[Task + success conditions] --> C[Context assembly]
    C --> P[Prompt / role instructions]
    P --> A[Agent proposes or performs action]
    A --> X[Tools / CI / external system]
    X --> E[Artifacts and observed evidence]
    E --> D{Controller evaluates state and policy}
    D -- Continue / repair / re-plan --> C
    D -- Accepted --> O[Record outcome]
    D -- Blocked / limit / approval needed --> H[Escalate]
~~~

The prompt influences the agent's behavior, but the surrounding system decides what information it sees, what it can do, what counts as evidence, and what happens next.

A role name alone is not a handoff contract. At minimum, the next step needs:

- **Goal:** what question or result is this step responsible for?
- **Inputs:** which task, artifacts and revisions are authoritative?
- **Authority:** what may it inspect or change?
- **Output:** what artifact or evidence must it return?
- **Exit condition:** what observable result means this step is finished?
- **Failure path:** what happens if it cannot finish?

For a small workflow, these can be a few lines in a prompt. For a durable system, keep the task state, permissions, artifacts and completion decision outside the prompt.

## What the Opus 5.5 prompting guidance adds

Anthropic's [Opus 5.5 prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5) reinforces several useful lessons for EOKS. These are model-specific observations, not universal laws.

### Use fewer instructions, but make the important ones precise

Anthropic reports that newer models can need less scaffolding, and recommends testing whether older instructions are still useful. Conflicting, repeated rules can make intent harder to interpret.

**EOKS implication:** do not make every role prompt a mini policy manual. Put stable knowledge in the context system, permissions in policy/tool configuration, and task-specific instructions in the prompt. Keep instructions that address observed failure modes; remove redundant rules only after testing.

### Express intent once; compile it for the model and task

A durable task definition might say:

~~~yaml
goal: "Add an optional HTTP request timeout"
must_preserve:
  - "Unset timeout preserves existing behavior"
evidence_required:
  - "Tests cover configured and default behavior"
completion: "Required checks pass; remaining limitations are recorded"
~~~

A prompt turns that intent into model-facing language. The controller and validator turn it into executable checks and a completion decision. These representations should agree, but they serve different purposes.

**EOKS implication:** prompting is one *realization* of intent and context, not a new architectural dimension. Keep task meaning and evidence requirements independent of any one model's preferred prompt style.

### Completion is a state decision, not a phrase

Anthropic warns that an unattended agent can finish a turn with a progress update while work remains. A harness must distinguish a turn ending from the task being complete, preserve a checklist or equivalent durable state, and limit automatic continuations.

**EOKS implication:** model output is an event or observation. The controller evaluates it against durable task state and completion criteria. A prompt can ask the agent to continue, but cannot replace that control logic.

### Calibrate effort and time budgets separately

The guide recommends measuring effort settings against your own tasks, and describes elapsed-time signals as a way to help agent teams pace work. A time budget is advisory unless the harness enforces a hard timeout.

**EOKS implication:** model effort, run budget, wall-clock deadline, retry limit and quality threshold are distinct controls. Don't encode them all as prose in a prompt.

### Context selection is part of execution design

For multi-tool workflows, the guide recommends looking across relevant sources before acting when task information may live in places the request did not explicitly name.

**EOKS implication:** an agent cannot use information it was never given or could not retrieve. Context compilation should select relevant sources, preserve provenance and mark untrusted content; a generic instruction to “be thorough” is not a substitute for those mechanisms.

## A simple rule for choosing what to add

| Observed problem | First mechanism to consider |
|---|---|
| Agent misunderstands the goal | Clarify intent and acceptance criteria |
| Agent lacks necessary facts | Improve retrieval and compiled context |
| Agent repeatedly misses one failure mode | Add a targeted instruction and test it |
| Agent claims success too early | Durable checklist + controller-side completion check |
| Agent takes an unauthorized action | Tool permissions / sandbox, not just stronger wording |
| Agent repeats expensive work | Persist state and artifacts; inspect the control loop |
| Parallel agents disagree | Evidence-based synthesis and explicit adjudication |
| Runs are too slow or costly | Measure effort, context size, delegation and budgets |

## What to test

Treat prompt changes as hypotheses. Compare a small baseline against one targeted change on representative tasks. Measure outcome quality, missed defects, unnecessary actions, latency and cost—not just whether the answer sounds better. Keep a rule when it fixes a repeatable problem without creating a larger one.

This document is an execution walkthrough, not a new EOKS primitive. It uses existing concepts—intent, context, roles, workflow, capabilities, policy, state, evaluation and evidence—to show how different mechanisms produce different execution flows.
