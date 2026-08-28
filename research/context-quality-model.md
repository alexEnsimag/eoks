# Context quality model

“Context quality” became one of the most important ideas in the EOKS discussions. It should not become a vague synonym for “good prompt”. This note proposes a research model that can be tested.

## Context as a constrained working set

Let a task have a set of potentially useful information items `I`. The model sees a selected working set `C ⊂ I`, subject to a budget.

The selection problem is not simply:

```text
maximize |C|
```

It is closer to:

```text
maximize expected task utility(C)
subject to cost(C) <= budget
```

where utility includes correctness, coverage and decision quality, while cost includes tokens, latency and retrieval/tool work.

## Dimensions

A candidate context can be evaluated along several dimensions.

### Relevance

Does the information help the current task?

### Coverage

Does the context contain the facts required to solve the task?

### Redundancy

How much information repeats other information without adding useful evidence?

### Contradiction

Does the context contain mutually inconsistent claims or instructions?

### Freshness

Is the information current for the task's time horizon?

### Authority

How trustworthy is the source?

### Provenance

Can the system explain where the information came from?

### Structure

Can the model distinguish facts, decisions, hypotheses, instructions and evidence?

### Cost

How much context budget is consumed?

These dimensions can conflict. More coverage can increase redundancy; more provenance can increase tokens; newer information can conflict with historical information.

## Why “entropy” is not yet a metric

We used entropy as an intuition for context becoming noisy or uncertain. That is useful, but it should not be promoted to a formal metric without specifying a probability model and demonstrating predictive value.

Instead, experiments should measure observable proxies such as contradiction density, semantic redundancy, uncertainty and unresolved references.

## Context quality versus context size

A useful benchmark should deliberately hold token count approximately constant while varying composition. Otherwise a result may only show that one prompt was longer.

Likewise, a quality metric should be tested against outcomes. Human preference for a context is interesting but insufficient.

## Selection signals

Possible signals for selecting a block:

```text
relevance
+ dependency proximity
+ source authority
+ freshness
+ task history
+ prior usefulness
- redundancy
- contradiction risk
- token cost
```

This is a conceptual scoring model, not a recommendation to use a linear weighted sum. Learned or rule-based policies may perform better.

## Context compilation

Selection and compilation should be separate:

```text
knowledge
   |
   v
candidate blocks
   |
   v
selection policy
   |
   v
semantic context package
   |
   v
model-specific compiler
   |
   v
actual invocation
```

This permits the same semantic working set to be rendered differently for different models.

## Context diffing

A powerful evaluation primitive is to compare two context packages:

```text
C1 = baseline
C2 = C1 + block X
```

If repeated experiments show that X improves outcomes for a task class, the system has evidence that X is valuable. If X increases errors or cost, its inclusion policy can change.

This is a path toward **context policies learned from outcomes**, rather than hand-tuned prompt templates.

## Research questions

- Can context quality predict task outcome before execution?
- Which dimensions generalize across workload types?
- Can useful context be selected without exposing all candidate information to the model?
- How stable are context-quality signals across models?
- Does human context editing provide enough improvement to justify the interface?
- Can context policies learn from successful and failed runs?

The answer to these questions determines whether “context quality” deserves to be a first-class EOKS subsystem.