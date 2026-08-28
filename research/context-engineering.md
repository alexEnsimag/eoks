# Context engineering research

## Starting point

A recurring question was whether the limiting factor in AI-assisted engineering is the model itself or the quality of the information presented to it. The discussion moved quickly away from “put more text in the prompt” toward treating context as a resource that can be selected, structured, measured and maintained.

The central working definition became:

> Context quality is not how much context exists; it is how much task-relevant, trustworthy, non-redundant information survives into the model's effective working set.

That distinction matters because a larger context window does not automatically produce better reasoning. More information can introduce redundancy, stale facts, irrelevant branches, contradictory instructions and attention competition.

## Context as a system

We explored a pipeline roughly like:

```text
repository / history / knowledge / traces
                    |
                    v
            candidate context
                    |
                    v
             context workbench
          inspect / edit / score
                    |
                    v
             context compiler
                    |
                    v
                 model
                    |
                    v
               outcome
                    |
                    +------> evaluation / memory
```

This suggests that context has a lifecycle rather than being an opaque string:

1. discover candidate information;
2. classify and normalize it;
3. establish provenance and freshness;
4. rank it for the task;
5. compose a working set;
6. compile it into the model-specific representation;
7. observe the result;
8. retain useful information and revise the policy.

## Context blocks versus graphs

We discussed a UI in which context is visible as blocks or clusters. A graph is useful for discovery, dependency visualization and explanation, but raw graph nodes are not necessarily the right human abstraction for editing a context.

A **context block** is a better primary unit for deliberate inclusion/exclusion. A block might represent code, a decision, evidence, a hypothesis, a prior task result, an API contract or a piece of project knowledge.

Possible metadata:

```yaml
id: slack-auth
type: code | knowledge | decision | evidence | hypothesis
sources: [...]
relations: [...]
confidence: 0.86
freshness: ...
relevance: ...
token_cost: ...
inclusion_reason: ...
```

This is a hypothesis, not a proposed mandatory file format. The important idea is that context should have **explainable composition**.

## Interactive context control

A proposed “context workbench” would let a human inspect and influence the working set without pretending to edit the model's hidden reasoning. Useful operations include:

- include / exclude;
- pin important information;
- inspect provenance;
- inspect dependencies;
- see estimated token cost;
- compare automatic and manually constrained context;
- save a context package for reuse;
- understand why a block was selected.

The interesting question is whether this improves results enough to justify the interaction cost. It should be tested, not assumed.

## Context quality and entropy

“Entropy” was used as an intuition for the amount of uncertainty, noise or competing information in a context. It should not be confused with a single mathematically established metric for prompt quality.

Candidate measurable dimensions discussed include:

- task relevance;
- redundancy;
- contradiction rate;
- freshness/staleness;
- source reliability;
- provenance completeness;
- token cost;
- coverage of required facts;
- number and strength of unresolved ambiguities;
- retrieval precision/recall;
- downstream task success.

A useful future metric would ideally predict whether adding/removing a context block improves a task outcome, rather than rewarding an aesthetically “clean” prompt.

## Splitting context

We repeatedly returned to **context splitting**: instead of creating one maximal context, maintain several specialized contexts or views. Examples include a task context, architecture context, code context, historical decisions and external evidence.

The split should be based on task boundaries and information dependencies, not arbitrary file sizes.

This raises an important EOKS question: should the system select one context, compose several contexts, or schedule multiple model calls over specialized contexts?

## Representation matters

We questioned why YAML-like structures appeared repeatedly in designs. The conclusion was not that YAML is intrinsically better. Structured representations are useful because they expose relationships and metadata, but the optimal representation may be JSON, a database, a graph, Markdown, typed objects or a compiled model-specific prompt.

The underlying requirement is **structured semantics**, not a particular serialization.

## Progressive disclosure

Another recurring idea was to avoid sending everything at once. Store rich information externally and disclose only what the current task needs, with mechanisms for the model or orchestrator to request deeper information.

This resembles how operating systems avoid putting every resource into a process's immediate working set: availability and working-set membership are different concepts.

## The deeper hypothesis

The important transition is:

```text
prompt engineering
        ->
context engineering
        ->
context management
        ->
knowledge + memory management
        ->
AI workload control
```

EOKS should therefore not become “a better prompt builder.” Context management is one subsystem of a broader control architecture.