# Behavioral memory and learning how developers work

## Motivation

A persistent knowledge base answers questions such as **what is true about the project?** A second-brain system for software engineering can go further: it can learn **how work gets done**.

The distinction matters because a coding agent does not only need facts. It also needs to choose useful investigation strategies, recognize recurring failure modes, follow effective review habits, and know when a particular workflow has historically worked well.

This suggests a useful separation:

```text
semantic memory   -> what is true / known
episodic memory   -> what happened / what worked
procedural memory -> how work should be performed
```

For EOKS, procedural memory is especially interesting because it can become an input to execution policy rather than merely another retrieved document.

## From second brain to behavioral memory

A conventional developer second brain might contain:

- solutions to problems already solved;
- architecture decisions and their rationale;
- debugging discoveries;
- reusable code patterns;
- learning notes;
- project history.

Those are valuable, but an agent can also learn from the **trajectory** that produced them:

```text
problem
  -> hypothesis
  -> evidence gathering
  -> failed attempt
  -> correction
  -> implementation
  -> verification
  -> review
  -> outcome
```

Repeated trajectories reveal patterns that are difficult to express as ordinary facts. For example, a developer may consistently inspect a particular subsystem before changing code, prefer a minimal change before considering a redesign, or use a specific verification sequence after modifying a risky component.

The goal is not to imitate every historical action. It is to discover **repeatable, outcome-linked procedures**.

## Three useful memory classes

### Semantic memory

Facts and durable knowledge:

- architecture;
- domain concepts;
- conventions;
- constraints;
- known failure modes.

This is closely related to the existing EOKS knowledge-base model.

### Episodic memory

Specific past experiences:

- a production incident;
- a difficult debugging session;
- a successful implementation;
- a failed approach and its correction;
- a previous task with similar shape.

Episodes are useful as evidence and examples. They should not automatically become universal rules.

### Procedural / behavioral memory

Generalized ways of working:

- investigation procedures;
- planning patterns;
- implementation heuristics;
- review checklists;
- verification sequences;
- reusable agent workflows;
- preferences about when to escalate or ask for review.

Procedural memory is the most interesting class for continuous agent improvement because it can directly influence future execution.

## Observation is not learning

A critical EOKS boundary is:

```text
observed behavior
      |
      v
candidate pattern
      |
      v
supporting evidence
      |
      v
validated procedure
      |
      v
execution policy / skill
```

A single session is usually insufficient evidence that a behavior is desirable. A behavior may have happened because of an unusual constraint, a mistaken assumption, or a one-off incident.

Patterns should therefore carry provenance and confidence, including where they were observed, what outcomes followed, and whether they have been validated across multiple examples.

## Learning from outcomes

Frequency alone is not enough. A repeated behavior may simply be a habit.

A stronger candidate pattern connects behavior to outcome:

```text
procedure P
   |
   +--> task class A --> successful outcome
   +--> task class B --> successful outcome
   +--> task class C --> failure
```

EOKS should therefore distinguish:

- **observed** — this happened;
- **repeated** — this happened several times;
- **successful** — this was associated with good outcomes;
- **validated** — there is enough evidence to recommend it;
- **deprecated** — later evidence shows it should no longer be used.

This is analogous to knowledge freshness and invalidation, but applied to behavior.

## Candidate skills and policies

A useful destination for validated procedural memory is not necessarily a prose note. It may become a reusable:

- skill;
- workflow;
- checklist;
- planner heuristic;
- tool-selection policy;
- verification policy;
- escalation rule.

For example:

```text
Candidate:
"When changing the telemetry pipeline, inspect the consumer topology
before modifying batching configuration."

Evidence:
- 7 related sessions
- 6 successful outcomes
- 1 exception caused by unrelated infrastructure failure

Promotion:
"telemetry-change-investigation" skill
```

The generated artifact should retain links to its supporting evidence and remain reversible.

## Hot-path vs background learning

Learning can happen during a task or after it.

### Hot path

The agent explicitly records important observations while working.

Advantages:

- immediate availability;
- explicit intent;
- useful for high-confidence facts.

Costs:

- adds latency and cognitive/tool overhead;
- asks the executing agent to reason about its own memory while solving the task.

### Background path

A separate process analyzes completed sessions and extracts/consolidates candidate memories.

Advantages:

- no impact on task latency;
- can compare many sessions;
- better suited to pattern discovery and generalization.

Costs:

- delayed learning;
- requires careful provenance and validation;
- extraction quality becomes an important evaluation problem.

For behavioral learning, background analysis is particularly attractive because patterns usually require multiple episodes rather than a single observation.

## Continuous learning loop

The resulting EOKS loop is:

```text
                 +-------------------+
                 |   development     |
                 |      activity     |
                 +---------+---------+
                           |
                           v
                      observation
                           |
                           v
                       session
                        traces
                           |
                           v
                   pattern extraction
                           |
                 +---------+---------+
                 |                   |
          episodic memory     candidate procedure
                 |                   |
                 +---------+---------+
                           |
                           v
                     validation
                           |
                           v
                 skill / policy / rule
                           |
                           v
                    future execution
                           |
                           +-------> outcome
                                         |
                                         +--> new observation
```

This turns the second brain from a passive archive into a **learning substrate for the control loop**.

## Architectural implication for EOKS

EOKS should not become a monolithic "developer clone" or automatically rewrite agent prompts after every session. The stronger abstraction is a lifecycle for behavioral knowledge:

1. observe work;
2. preserve the trace and outcome;
3. extract candidate patterns;
4. group similar episodes;
5. test whether the pattern correlates with successful outcomes;
6. validate against source evidence and counterexamples;
7. promote into a skill, policy or reusable workflow;
8. retrieve it selectively for future tasks;
9. measure whether it actually improves outcomes;
10. invalidate or revise it when evidence changes.

This keeps learning compatible with the existing EOKS principles of provenance, progressive disclosure, evaluation and controlled promotion.

## Open questions

- What is the minimum useful session trace?
- How should successful outcomes be defined for software-engineering tasks?
- How many examples are needed before a pattern is promoted?
- How can the system avoid learning accidental habits?
- How should contradictory procedures coexist?
- How should developer-specific behavior differ from project-specific policy?
- Can a learned procedure be evaluated offline against historical sessions?
- When should a learned pattern become a skill versus remain retrievable evidence?
- How can model changes affect the apparent success of a learned procedure?
- How should humans approve, correct or delete learned behavior?

The key research question is empirical: **does learning procedural patterns from development traces measurably improve future task outcomes enough to justify the additional complexity?**
