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

Repeated trajectories reveal patterns that are difficult to express as ordinary facts. The goal is not to imitate every historical action, but to discover **repeatable, outcome-linked procedures**.

## Second-brain use cases

A useful developer second brain can support several layers:

1. **Personal engineering knowledge** — solutions, architecture decisions, debugging discoveries, code patterns and lessons learned.
2. **Project memory** — constraints, conventions, decisions, known failures, goals and historical rationale.
3. **Debugging memory** — reusable investigation episodes: `symptom -> hypotheses -> evidence -> root cause -> fix -> verification`.
4. **Session learning** — recurring task decomposition, evidence-gathering, tool usage, verification habits, dead ends and human corrections.
5. **Career and organizational memory** — durable records of projects, impact, incidents and technical decisions that support handoffs, reviews and onboarding.

The useful distinction is that the system should retain both the **answer** and the trajectory that produced it.

## Three useful memory classes

### Semantic memory

Facts and durable knowledge: architecture, domain concepts, conventions, constraints and known failure modes.

### Episodic memory

Specific past experiences: incidents, difficult debugging sessions, successful implementations, failed approaches and corrections. Episodes are evidence, not automatically universal rules.

### Procedural / behavioral memory

Generalized ways of working: investigation procedures, planning patterns, implementation heuristics, review checklists, verification sequences, reusable workflows and escalation preferences.

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

A single session is usually insufficient evidence. A behavior may have happened because of an unusual constraint, a mistaken assumption, or a one-off incident.

Patterns should carry provenance, confidence, scope, prerequisites, supporting sessions, outcomes and counterexamples.

## Learning from outcomes

Frequency alone is not enough. A repeated behavior may simply be a habit. A stronger candidate connects behavior to outcome:

```text
procedure P
   |
   +--> task class A --> successful outcome
   +--> task class B --> successful outcome
   +--> task class C --> failure
```

EOKS should distinguish:

- **observed** — this happened;
- **repeated** — this happened several times;
- **successful** — this was associated with good outcomes;
- **validated** — evidence supports recommending it;
- **deprecated** — later evidence shows it should no longer be used.

## Candidate skills and policies

Validated procedural memory does not have to remain prose. It may become a skill, workflow, checklist, planner heuristic, tool-selection policy, verification policy or escalation rule.

For example:

```text
Candidate:
"When changing the telemetry pipeline, inspect the consumer topology
before modifying batching configuration."

Evidence:
- related sessions
- successful and exceptional outcomes
- commits / tests / reviews

Promotion:
"telemetry-change-investigation" skill
```

The generated artifact should retain evidence links and remain reversible.

## Hot path vs background learning

### Hot path

The executing agent records important observations while working. This gives immediate availability and explicit intent, but adds latency and cognitive/tool overhead.

### Background path

A separate process analyzes completed sessions and extracts/consolidates candidate memories. This can compare many sessions and is particularly attractive for behavioral learning because useful patterns usually require multiple episodes.

The background path should therefore be the default research direction for discovering general procedures, while high-confidence facts can still be captured during execution.

## Session traces as learning evidence

A coding session should be represented as a trace rather than only a transcript:

```text
Goal
  -> plan
  -> observations / evidence
  -> files inspected
  -> commands and tools used
  -> hypotheses
  -> edits
  -> failures / corrections
  -> tests / verification
  -> human feedback
  -> final outcome
```

Useful events include task start/completion, plan revisions, tool calls, artifacts inspected, hypotheses, tests, failures, corrections, human intervention, acceptance/rejection and cost/latency/model information.

A transcript preserves what was said. A trace makes the **decision process and outcome** queryable.

Sensitive data should be filtered before persistence, with explicit retention and promotion policies.

## From traces to learned patterns

```text
session trace
    -> episode
    -> recurring pattern
    -> validation
    -> Learning Record
    -> candidate skill / policy
    -> controlled rollout
    -> evaluation
```

The output should be proposals with evidence, not silent prompt mutation.

## Learning Record

A useful primitive is:

```text
LearningRecord
  situation
  action / strategy
  evidence
  outcome
  evaluation
  provenance
  confidence
  scope
  validity
  status: candidate | validated | promoted | deprecated
```

A memory says what is known; a Learning Record captures **what behavior was tried in what situation and what happened**. This can produce reusable skills, project policies, routing policies, verification policies and context-selection heuristics.

## Why transcript RAG is insufficient

A vector store containing historical conversations can answer:

> Have I seen this problem before?

Behavioral learning needs to answer:

> What did I tend to do in situations like this, did it work, and should the agent do something similar now?

That requires structured episodes, outcome/evaluation signals, provenance, temporal validity, confidence, promotion rules and regression evaluation. Historical transcripts remain valuable evidence, but they should not themselves become the learned policy.

## Continuous improvement loop

```text
observe
  -> extract
  -> validate
  -> store
  -> retrieve
  -> execute
  -> evaluate
  -> compare against previous outcomes
  -> update candidate policy / skill
  -> evaluate again
```

This turns the second brain from a passive archive into a **learning substrate for the control loop**.

## Reflection questions

A periodic reflection process can ask:

- Where did work repeatedly waste time?
- Which mistakes recur?
- Which developer corrections recur?
- Which evidence sources consistently resolve a class of problem?
- Which tools are used together?
- Which verification steps correlate with successful outcomes?
- Which behaviors should become reusable skills?
- Which instructions are redundant or contradictory?
- Which routing decisions should change?
- What new evaluation should be added?

## Learning plane

This suggests a learning plane alongside the execution/control plane:

```text
                  EOKS CONTROL PLANE
          scheduling · routing · policies
                         |
        +----------------+----------------+
        |                |                |
     CONTEXT           MEMORY          EXECUTION
        |                |                |
        +----------------+----------------+
                         |
                    EVALUATION
                         |
                    OBSERVABILITY
                         |
                    LEARNING PLANE
                         |
          trace analysis / reflection /
          pattern extraction / proposals
                         |
                  versioned knowledge
                         |
                    back to control
```

The learning plane should not be another autonomous agent runtime. Its job is to transform accumulated evidence into candidate improvements that can be evaluated and versioned.

## Existing tools as capability references

- **LangMem** — particularly relevant prior art for semantic, episodic and procedural memory plus background extraction/reflection and behavior optimization.
- **Mem0** — persistent memory and memory extraction/retrieval infrastructure.
- **Zep** — long-term memory and graph-oriented context infrastructure.
- **OpenHands** — primarily an execution runtime/SDK and a useful source of agent execution-trace prior art.

These are capability references, not EOKS dependencies. EOKS remains broader because it coordinates memory with evidence providers, context assembly, execution policy, scheduling and evaluation.

## Research questions

- What is the minimum useful session trace?
- How should successful outcomes be defined for software-engineering tasks?
- How many examples are needed before a pattern is promoted?
- How can the system avoid learning accidental habits?
- How should contradictory procedures coexist?
- How should developer-specific behavior differ from project-specific policy?
- Can a learned procedure be evaluated offline against historical sessions?
- When should a learned pattern become a skill versus remain evidence?
- How can model changes affect the apparent success of a learned procedure?
- How should humans approve, correct or delete learned behavior?

The key falsifiable question is empirical: **does learning procedural patterns from real development traces measurably improve future software-engineering task outcomes enough to justify the added complexity?**
