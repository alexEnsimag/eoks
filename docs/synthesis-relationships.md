# EOKS synthesis relationships

> Focused follow-up to the conceptual synthesis. This document makes a small set of relationships explicit without introducing new runtime primitives or collapsing distinct mechanisms.

## Scope

The recent synthesis pass does **not** reveal another large missing body of concepts. The current architecture already contains the major concepts needed to describe workload control, resources, working sets and context, execution, evaluation, evidence, knowledge maintenance, learning, deterministic execution and graduated autonomy.

The remaining opportunity is primarily relational: make explicit how existing concepts interact across lifecycle boundaries.

The four relationships below are the main synthesis improvements. They should be treated as architectural clarifications and hypotheses to validate, not as new ontology objects.

## 1. Reuse is a cross-cutting control concern

EOKS contains several forms of reuse that should remain technically distinct:

```text
semantic memory
       |
context / working-set reuse
       |
inference / KV cache
       |
computed artifact
       |
validated deterministic procedure
       |
derived knowledge representation
```

These mechanisms operate at different layers and have different invalidation semantics. They should not be merged into one cache or memory abstraction.

What they share is a control question:

> **Can an existing state, representation, computation or procedure be reused safely instead of being reacquired or recomputed?**

Reuse is therefore governed by several properties:

- **scope** — where and for which workload the result applies;
- **validity** — whether its assumptions and dependencies still hold;
- **freshness** — whether it reflects the required source/version/time;
- **provenance** — how it was produced and what evidence supports it;
- **dependency lineage** — what changes should invalidate it;
- **economics** — whether reuse is cheaper or faster than recomputation/acquisition;
- **fidelity** — whether the reused representation is sufficiently faithful for the current decision.

This gives a common lens without requiring a `Reuse` runtime primitive.

```text
candidate reusable result
          |
   scope / validity / lineage
          |
      still usable?
       /        \
     yes         no
      |           |
    reuse     invalidate
                  |
              recompute
```

The existing distinction between semantic/context caching and inference/KV caching should remain explicit. A common reuse protocol may eventually emerge, but that is an implementation hypothesis requiring evidence.

## 2. Objective -> risk -> assurance -> authority/autonomy

EOKS already has objectives, policies, risk, evidence, evaluation and graduated autonomy. The useful synthesis is to make their decision relationship explicit:

```text
objective / intended outcome
            |
            v
     consequence / risk
            |
            v
    required assurance
            |
            v
 evidence + verification
            |
            v
 permitted authority / autonomy
            |
            v
      control decision
```

The point is not to create a universal risk score or confidence scalar. Different workloads can require different evidence and different forms of authority.

A higher-risk objective can require stronger or more independent assurance before the same execution modality is allowed to act autonomously. Conversely, well-understood low-risk work can justify more automation with less expensive verification.

This makes **autonomy an outcome of policy and assurance**, rather than a property of the agent/model itself.

The distinction should remain:

```text
model confidence
      != evidence strength
      != probability of correctness
      != permitted authority
```

The conductor can use evaluated evidence and policy to choose among continued execution, additional verification, escalation, human approval or autonomous completion.

## 3. Workload control loop vs. derived-state lifecycle

EOKS already describes reconciliation and knowledge maintenance as loops. A broader distinction is useful because many reusable artifacts are neither authoritative workload state nor merely ephemeral execution data.

### Workload control

```text
goal / policy
    -> observe workload
    -> identify gap
    -> choose action
    -> execute
    -> verify / evaluate
    -> update actual state
    -> reconcile
```

### Derived-state lifecycle

```text
authoritative source/state
    -> derive representation / computation
    -> validate
    -> publish for use
    -> reuse
    -> dependency changes
    -> validate again
       /          \
    still valid   stale
       |             |
     reuse      invalidate/rebuild
```

The two loops interact:

```text
             WORKLOAD CONTROL
                    |
             requires information
                    |
                    v
             DERIVED-STATE LOOP
                    |
          valid representation/result
                    |
                    v
             WORKLOAD CONTROL
```

This applies to graphs and indexes, summaries, computed artifacts, context materializations, learned procedures and other derived resources.

The architectural implication is a clear distinction between:

- **authoritative state** — the source of truth for a claim or workload condition;
- **derived state** — materialized from other state and therefore potentially stale;
- **execution state** — what a run has attempted, observed or changed;
- **evidence** — information supporting a claim or decision about any of the above.

Derived state should carry enough provenance and dependency information to determine when reuse is safe and when invalidation/recomputation is required.

This does **not** imply a new `DerivedState` primitive or a separate orchestration plane. It is a lifecycle distinction that clarifies existing knowledge-maintenance and reuse semantics.

## 4. Authority, representation and temporal lineage

Three smaller clarifications follow naturally from the first three relationships.

### Authority is not the same as evidence

A source can be authoritative without being sufficient evidence for a particular claim, while an observation can be useful evidence without becoming authoritative state.

```text
authoritative source
       |
       +--> representation
       |
       +--> observation / evidence
                         |
                         v
                    evaluation
                         |
                         v
                     decision
```

For important state and claims, EOKS should be able to distinguish at least:

- who/what is authoritative;
- who/what produced the observation;
- what evidence supports it;
- how fresh it is;
- what dependencies constrain its validity.

### Representation is not state

A representation is a materialization or expression of information; it is not automatically the thing being represented.

For example:

```text
authoritative repository state
          |
      graph / index
          |
      working set
          |
       context
```

The graph, working set and context may all describe or select aspects of the same underlying information while having different lifecycles and validity conditions.

Likewise, a computed artifact can be a reusable derived result without becoming authoritative state.

### Temporal lineage makes reuse and assurance inspectable

For consequential decisions, it is useful to preserve the chain:

```text
source/version
     |
 derived at T1
     |
representation/version
     |
computation at T2
     |
evidence observed at T3
     |
decision at T4
     |
outcome at T5
```

This answers a practical question that simple provenance does not fully capture:

> **What did the controller know, from which representation and source version, when it made this decision?**

Temporal lineage should remain a property of state, evidence, representations, runs and decisions rather than a new primitive.

## 5. What this synthesis does not introduce

The following should remain explicitly **non-primitives** unless implementation or evaluation evidence demonstrates otherwise:

- `Reuse`;
- `Risk`;
- `Assurance`;
- `Authority`;
- `Lineage`;
- `DerivedState`;
- a universal confidence score;
- a universal cache/memory abstraction;
- a separate assurance or reuse control plane.

They are dimensions, properties, lifecycle relationships or control criteria spanning the existing EOKS model.

## 6. Validation implications

These relationships should be validated through small, falsifiable simulations rather than immediately promoted into ontology.

### Reuse

Compare recomputation against dependency-aware reuse while changing one dependency. Measure quality, latency, cost, reuse rate and stale-result failures.

### Assurance/autonomy

Hold the underlying task constant while varying risk and available evidence. Test whether an explicit assurance policy selects appropriately different verification and autonomy levels without relying on a single confidence number.

### Derived-state lifecycle

Create several dependent representations, change an upstream source, and test whether only affected artifacts are invalidated and whether the controller avoids consuming stale results.

### Temporal lineage

Reconstruct a past decision from recorded source versions, representations, evidence, runs and evaluations. Test whether the system can explain which information was available at decision time.

A successful simulation should increase confidence in the relationship; it should not automatically make the relationship a runtime primitive.

## 7. Relationship to the canonical architecture

These clarifications reinforce the existing EOKS architecture rather than replacing it:

```text
intent
  -> desired outcome/state
  -> policy
  -> conductor / reconciliation
  -> resource + working-set + execution selection
  -> run
  -> observe / verify
  -> outcome
  -> evaluation / evidence
  -> actual state
  -> reconcile
```

The additions are cross-cutting lenses on this loop:

- **reuse** governs when prior state/computation can substitute for new work;
- **risk and assurance** constrain how much authority the controller can exercise;
- **derived-state lifecycle** governs the validity of reusable representations/results;
- **authority and temporal lineage** make state, evidence and decisions interpretable and reconstructable.

No new runtime primitive is required by this synthesis.
