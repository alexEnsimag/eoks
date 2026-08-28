# Learning from development sessions

## The session as a learning unit

A coding-agent session contains more than a transcript. It can expose a trajectory through a software-engineering task:

```text
intent
  -> plan
  -> repository discovery
  -> evidence gathering
  -> tool calls
  -> hypotheses
  -> implementation attempts
  -> failures / corrections
  -> tests and verification
  -> review
  -> final outcome
```

This trajectory is potentially more useful for learning than the final answer alone.

## What to capture

A practical session trace can combine several evidence sources:

### Agent interaction

- user goal;
- agent plan and decisions;
- prompts and responses where useful;
- tool calls and results;
- files inspected or changed;
- commands executed;
- tests and verification performed.

### Development artifacts

- commits;
- pull requests;
- review comments;
- CI results;
- issue changes;
- runtime or observability signals where available.

### Corrections

Human corrections are especially valuable because they expose a mismatch between the current behavior and desired behavior:

- user rejected an approach;
- reviewer requested a change;
- test exposed a wrong assumption;
- implementation was reverted;
- an unnecessary step was identified.

### Outcome

The trace should connect behavior to an outcome when possible:

- task completed or not;
- tests passed or failed;
- review accepted or required changes;
- number of retries/corrections;
- time or tool cost;
- regressions discovered later.

The exact schema should remain experimental. EOKS should first establish which signals are predictive before committing to a heavyweight event model.

## Pattern mining

A reflection process can analyze individual sessions and then compare patterns across sessions.

A useful pipeline is:

```text
session traces
     |
     v
normalize events
     |
     v
extract episodes
     |
     v
cluster similar tasks / trajectories
     |
     v
identify recurring procedures
     |
     v
compare outcomes
     |
     v
candidate behavioral knowledge
```

The analyzer should look for patterns such as:

- repeated investigation sequences;
- common successful tool combinations;
- recurring dead ends;
- steps that reliably prevent later failures;
- situations where human intervention is repeatedly required;
- workflows that differ by task class;
- unnecessary work that appears repeatedly.

## Example

Suppose several database migration sessions show:

```text
inspect schema
  -> inspect indexes
  -> estimate affected rows
  -> run targeted query
  -> change migration
  -> test on representative data
```

and sessions that skipped the query-estimation step repeatedly caused performance problems.

A reflection agent might produce:

```text
Candidate procedure
-------------------
For large database migrations, inspect the expected affected-row volume
and relevant indexes before finalizing the migration.

Observed in: 8 sessions
Successful: 7
Exceptions: 1
Evidence: migration commits + CI + review comments
Confidence: candidate
```

The candidate can then be tested against historical sessions or future tasks before becoming a trusted workflow rule.

## Anti-pattern: cloning the transcript

Simply embedding all previous conversations is not behavioral learning.

It has several problems:

- retrieval may surface irrelevant history;
- repeated behavior does not imply good behavior;
- contradictions accumulate;
- context cost grows;
- the system cannot distinguish causes from incidental actions.

The goal is to transform traces into **small, evidence-backed abstractions** while retaining the original episodes as provenance.

## Anti-pattern: learning every correction as policy

Human feedback is valuable but not every correction should become a global rule.

For example:

```text
"Don't use approach X here"
```

may mean:

- X is always bad;
- X is bad for this project;
- X is bad for this task;
- X was correct but too expensive;
- X was correct but violated a temporary constraint.

The learning system must preserve scope and conditions rather than generalizing blindly.

## Evaluation loop

Behavioral learning needs its own evaluation layer.

A learned procedure should be evaluated on tasks with similar characteristics, ideally with a baseline that does not use the procedure:

```text
historical tasks
      |
      +---- baseline execution
      |
      +---- learned-procedure execution
      |
      v
compare outcomes
```

Useful measures may include:

- task success;
- regression rate;
- correction count;
- time to completion;
- tool/token cost;
- review acceptance;
- quality of tests;
- unnecessary exploration.

This prevents EOKS from confusing **more agent activity** with **better engineering**.

## Human control

A continuous-learning system should make learned behavior inspectable.

For every promoted procedure, a developer should be able to answer:

- What is the procedure?
- Why does the system believe it works?
- Which sessions support it?
- Which outcomes support it?
- What are its scope and prerequisites?
- What evidence contradicts it?
- When was it last validated?
- How can it be corrected or removed?

This is consistent with EOKS's broader preference for provenance and controlled promotion over opaque adaptation.

## Minimal viable experiment

The first experiment does not require a sophisticated memory database.

Start with:

```text
session JSON/JSONL
        |
        v
reflection script / agent
        |
        v
candidate-pattern Markdown/JSON
        |
        v
human review
        |
        v
small skill/policy set
```

Then replay historical tasks to test whether the learned procedures help.

This provides a falsifiable experiment before building a generalized behavioral-memory service.

## Relationship to EOKS resources

Session learning introduces a useful resource flow:

```text
Trace -> Episode -> Pattern -> Procedure -> Policy -> Context
```

Each stage has different trust and lifecycle semantics. A raw trace should not have the same retrieval weight as a validated procedure, and a project policy should not be silently replaced by an inferred pattern.

This is another example of why EOKS is better modeled as a control plane than as a single memory store.
