# EOKS synthesis: asymmetric assurance and challenge

> Assurance pattern derived from recent research on self-correction, verification, counterexamples, adversarial critique and correlated failure. This document extends the execution-graph synthesis without introducing new EOKS primitives.

## Scope

The execution-graph synthesis established that verification can be a distinct branch of an execution topology. Further research sharpens this into a more general assurance pattern:

```text
support / construct
        |
      artifact
        |
   +----+----+
   |         |
 attack   verification
   |         |
   +----+----+
        |
     evaluation
        |
 accept / revise / reject / escalate
```

The important distinction is **asymmetry**. A support run tries to construct the best artifact or claim. An attack run deliberately searches for reasons that the artifact should fail. A verification run tries to establish specific properties using a mechanism capable of doing so. Evaluation decides whether the combined evidence is sufficient for the intended transition.

These are roles/objectives over existing EOKS Runs, not new primitives.

## 1. Reflection is not the same as attack

Reflection asks a trajectory or worker to inspect what happened and determine what should change.

Attack asks a separate evidence-producing path:

> **Assume the artifact may be wrong. What would demonstrate that?**

This matters because research on intrinsic LLM self-correction repeatedly finds that models can struggle to locate their own reasoning errors and can sometimes make correct answers worse during self-correction. Stronger results tend to involve reliable external feedback or explicit verification mechanisms.

Therefore EOKS should not equate:

```text
reflection -> assurance
```

Instead:

```text
reflection -> possible improvement
attack     -> falsification evidence
verification -> property evidence
```

The resulting evidence can then enter Evaluation.

## 2. Support and attack are asymmetric evidence objectives

A worker is optimized for constructing a useful answer. A challenger is optimized for finding failure conditions.

```text
WORKER / SUPPORT
  "Build the best solution and support its claims."

CHALLENGER / ATTACK
  "Find a credible way this solution could be wrong, incomplete or invalid."
```

The challenger should ideally produce an **attack**, not simply disagreement:

- a missing requirement;
- an invalid assumption;
- a contradiction;
- a concrete counterexample;
- a violated invariant;
- an unavailable dependency;
- evidence that an acceptance criterion is insufficient;
- an operational or boundary failure.

This makes attack evidence amenable to the same provenance and evaluation mechanisms as other EOKS evidence.

## 3. Verification is a different role again

A challenger can discover a suspected failure. Verification should determine whether the suspected failure or desired property actually holds.

```text
challenger
   |
   +--> "I found a possible counterexample."
                 |
                 v
             verifier
                 |
                 +--> confirmed
                 +--> refuted
                 +--> unresolved
```

The verifier does not have to be an LLM. Depending on the property it may be:

- a unit/integration test;
- a property-based test;
- a compiler/type checker;
- a static analyzer;
- a database constraint;
- an executable specification;
- an authoritative source;
- a proof or formal checker;
- an independent model-based check;
- human review.

This supports a general EOKS rule:

> **Use the strongest affordable mechanism that can actually establish the property at stake.**

An LLM saying that a property appears true is not equivalent to a mechanism that tests the property.

## 4. Assurance topology

The execution graph can therefore contain an assurance topology:

```text
                         artifact
                            |
                    +-------+-------+
                    |               |
                 SUPPORT          ATTACK
                    |               |
                    +-------+-------+
                            |
                     substantiation
                            |
                       VERIFICATION
                            |
                        EVALUATION
                            |
                  +---------+---------+
                  |         |         |
                ACCEPT    REVISE    REJECT
```

Policy determines when this topology is warranted.

A low-consequence workload might use:

```text
support -> deterministic checks -> evaluation
```

A more consequential workload might use:

```text
support -> independent attack -> verification -> evaluation
```

The point is not that the second topology is always better. It must earn its cost through fewer escaped defects, better assurance, or better outcomes.

This is the assurance counterpart of the earlier graph principle:

> **Use the minimum coordination structure that provides the required outcome and assurance.**

## 5. Independence matters more than agent count

Adding more reviewers does not necessarily create more independent evidence.

Evidence paths can be correlated because they share:

- model family or checkpoint;
- prompt strategy;
- context;
- retrieved sources;
- artifact decomposition;
- assumptions;
- acceptance criteria;
- upstream failures;
- verification mechanism.

Thus:

```text
3 agents
  !=
3 independent evidence paths
```

EOKS should preserve provenance and relevant dependence information when it matters to an assurance decision.

A particularly useful experimental variable is the degree of independence between worker and challenger:

```text
same model + same context
same model + different context
same model + different attack strategy
different model + same evidence
different model + different evidence
LLM attack + deterministic verification
```

The purpose is not to prescribe heterogeneous models. It is to determine empirically when additional independence creates enough new evidence to justify its cost.

## 6. Challenger context should be deliberate

A challenger does not necessarily need the worker's reasoning trajectory.

Giving the challenger only:

```text
requirements + artifact + relevant evidence
```

may reduce anchoring on the worker's decomposition and assumptions.

Giving it the worker rationale may instead be useful when the rationale itself is the object of the attack.

Therefore context selection is an **assurance policy decision**, not a fixed rule.

A useful EOKS experiment is to compare artifact-only challenge with artifact-plus-rationale challenge and measure unique validated defect discovery and false-positive rate.

## 7. Attack protocol

A practical challenge protocol can be expressed without adding a new EOKS object:

### Construct

The support run produces:

- artifact;
- claims;
- assumptions;
- acceptance criteria;
- supporting evidence.

### Attack

The challenge run attempts to falsify or invalidate:

- requirements;
- assumptions;
- dependencies;
- properties;
- evidence;
- verification criteria;
- operational behavior.

### Substantiate

Each meaningful attack should become a checkable claim where possible.

### Verify

Use an appropriate independent mechanism to confirm, refute or leave the attack unresolved.

### Evaluate

Combine support, attack and verification evidence while preserving provenance and uncertainty.

### Decide

Accept, revise, reject or escalate according to Policy.

### Preserve

Keep the resulting assurance evidence attached to the artifact/state transition so later reuse does not erase why it was trusted.

## 8. Challenger taxonomy

The following categories are useful as a challenge protocol, not as EOKS ontology:

| Attack | Question |
|---|---|
| Requirement | What important requirement is missing, ambiguous or contradictory? |
| Assumption | Which assumption could be false? |
| Dependency | What hidden coupling or unavailable capability could break this? |
| Counterexample | What concrete state/input/scenario violates the claim? |
| Boundary | What happens at scale, limits, concurrency or failure? |
| Evidence | Is the supporting evidence stale, weak, circular or non-independent? |
| Verification | Do the stated checks actually establish the desired property? |
| Operational | How could deployment, recovery, observability, security or cost invalidate the design? |

A challenge need not cover every category. Policy should select the attack surface appropriate to the workload.

## 9. Relationship to synthesis

This changes how Synthesis should be understood in an execution graph.

Avoid:

```text
workers -> giant prompt -> synthesizer
```

Prefer:

```text
             +-> support A --+
Task -> fan-out              |
             +-> support B --+\
                                +-> reduce -> attack -> verify -> synthesize
             +-> alternative -+/             |
                                           evidence
```

The synthesizer should reason over a reduced, provenance-preserving evidence state that includes both supporting and adversarial findings.

This makes synthesis less vulnerable to a common failure mode: treating a large quantity of mutually consistent worker output as if consistency were independent confirmation.

## 10. Assurance evidence should survive reuse

The validated/reusable-computation synthesis implies that assurance is part of the evidence attached to a reusable artifact.

If an artifact is reused, the system should know:

- what was challenged;
- what was verified;
- which evidence supported the decision;
- when the evidence was produced;
- which dependencies/versions it covered;
- which assumptions it relied on;
- whether later changes invalidate the assurance.

Material changes should therefore be able to trigger **targeted re-assurance** rather than blindly re-running every check.

This connects assurance to temporal lineage and dependency-aware recomputation already present in EOKS.

## 11. Assurance is itself evaluated

A critical control-loop consequence is:

> **Do not trust an assurance mechanism merely because it exists. Evaluate whether it predicts or prevents real failures.**

For example:

```text
artifact accepted
      |
      v
real-world / blind evaluation
      |
      +--> defect escaped
      |
      +--> assurance should learn
```

Candidate measurements include:

- attack yield;
- unique attack yield;
- attack precision;
- severity-weighted challenge value;
- false reassurance;
- escaped defect rate;
- verification coverage;
- revision efficiency;
- effective assurance independence.

These are candidate EOKS metrics, not established standards.

## 12. Validation agenda

The most useful initial experiment is a topology comparison over the same task distribution:

```text
A  worker
B  worker + verifier
C  worker + challenger
D  worker + challenger + verifier
E  worker + independent challenger + verifier
```

For each condition measure:

- final correctness/completeness;
- unique defects discovered;
- false-positive attacks;
- escaped defects;
- revision cycles;
- assurance cost;
- latency;
- context/tool/model usage;
- human intervention.

Then vary independence and context boundaries.

The central hypotheses are:

1. **Asymmetric attack finds defects that reflection/support misses.**
2. **Independent attack produces more unique validated defects than correlated attack.**
3. **Verification converts attack findings into substantially stronger evidence than critique alone.**
4. **Additional assurance has diminishing returns and should be selected by Policy.**
5. **Preserving assurance provenance improves safe reuse and targeted re-assurance.**

None should be treated as an EOKS invariant until empirical work supports it.

## 13. Canonical EOKS loop

The canonical loop remains unchanged:

```text
intent
  -> desired state / outcome
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

The assurance synthesis clarifies the execution-selection and evaluation stages:

```text
execution selection
        |
        +--> support topology
        +--> attack topology
        +--> verification mechanism
                 |
                 v
              evaluation
                 |
                 v
              decision
```

No new foundational object is required.

## 14. Design principle

The current synthesis can be summarized as:

> **Construct with support, search for failure with attack, establish properties with verification, and make decisions from provenance-preserving evidence. Use only as much assurance topology as the consequence of being wrong justifies.**

This is a stronger and more precise formulation than "add a challenger agent." It makes the pattern about evidence and control rather than agent count.
