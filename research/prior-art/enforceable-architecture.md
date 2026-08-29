# Enforceable architecture, ADRs, fitness functions and spec-driven development

## Why this matters to EOKS

A recurring weakness in agent-assisted software development is the gap between **architectural intent** and **mechanical enforcement**. An ADR can explain why a boundary exists, a specification can describe what a change must achieve, and an agent can implement it—but none of those artifacts necessarily prevents the implementation from drifting later.

Recent work around spec-driven development (SDD), evolutionary architecture, architecture fitness functions and policy-as-code points toward a stronger artifact chain:

```text
intent / outcome
      |
      +--> ADR: why this decision exists
      |
      +--> specification / contract: what must be true
      |
      +--> acceptance criteria / invariants
      |
      +--> fitness functions / policies: how conformance is checked
      |
      +--> implementation
      |
      +--> evidence + evaluation
      |
      +--> decision / repair / supersession
```

This is highly relevant to EOKS because it gives the control plane something concrete to coordinate: not just context and agents, but **the evidence and policies that determine whether a proposed action is acceptable**.

## The important distinctions

### ADR is not the specification

An ADR records a consequential decision: context, alternatives, decision, consequences and often the assumptions behind it. It is primarily an **intent/rationale artifact**.

A specification or contract describes behavior, interfaces, constraints and acceptance criteria. It is primarily a **what-must-be-true artifact**.

The two should be linked rather than collapsed:

```text
ADR
  why / trade-offs / assumptions
       |
       v
specification / contract
  required behavior / constraints
       |
       v
acceptance + invariants
  observable conditions
```

A single ADR may produce several specifications, and a specification may depend on several ADRs.

### Documentation is not enforcement

A statement such as "domain code must not depend on infrastructure" becomes much stronger when paired with an architecture test or dependency rule. Similarly, "all infrastructure must satisfy this security policy" becomes enforceable when represented as policy-as-code in the delivery pipeline.

EOKS should therefore distinguish:

- **intent** — what people decided and why;
- **specification** — what the system is required to do;
- **invariant/policy** — what must remain true;
- **fitness function** — an executable measurement/check of an architectural characteristic;
- **evidence** — the observed result of a check or experiment;
- **decision** — what the control loop does with that evidence.

A decision may be intentionally unenforceable. In that case the architecture should say so and use another control such as review, monitoring or an explicit re-evaluation date rather than pretending prose is a hard guardrail.

## Fitness functions as the enforcement bridge

Evolutionary architecture treats a fitness function as an automated mechanism for measuring whether an implementation remains within desired architectural characteristics. In practice, a fitness function can be very small: a dependency-direction test, contract test, schema check, performance threshold, security scan or custom analyzer.

The useful EOKS abstraction is not a particular framework. It is the relationship:

```text
architectural intent
        |
        v
named invariant / characteristic
        |
        v
fitness function(s)
        |
        v
structured evidence
        |
        v
policy / workflow decision
```

The enforcement mechanism should remain independent of the invariant. The same invariant might be enforced by a compiler/type system, architecture test, static analyzer, policy engine, contract test or runtime evaluation depending on what is actually measurable.

### Hard gates versus advisory signals

Not all checks should have the same authority.

**Deterministic gates** are appropriate when a violation is mechanically decidable: failed tests, incompatible schemas, forbidden dependencies, invalid configuration, security policy violations or exceeded hard thresholds.

**Probabilistic/agentic judges** are useful for evidence-bound concerns that require interpretation: boundary fidelity, semantic contract drift, workflow coupling or whether ADR assumptions still match reality. They should normally begin as advisory signals.

A safe hierarchy is:

```text
deterministic invariant
       |
       +--> hard gate when violation is objective
       |
       +--> advisory evidence when interpretation is required
                         |
                         +--> calibrated confidence
                         +--> human escalation when ambiguous
                         +--> deterministic rule candidate when repeated
```

An LLM judge should not silently replace a reliable deterministic check.

## Agentic fitness functions

Recent work describes an **agentic fitness function** as a governed architecture check in which an AI evaluator applies a versioned analytic rubric to a bounded evidence pack and returns a structured verdict.

A useful contract is:

```text
FitnessFunction
  intent: named architectural concern
  evidence_contract: allowed evidence
  rubric: versioned evaluation criteria
  evaluator: deterministic check or agentic judge
  verdict: structured result
```

A verdict should preserve at least:

```yaml
fitness_function: checkout-boundary-fidelity
rubric_version: 2026-07-01
score: 0.68
confidence: 0.74
decision: advisory_warn
violated_criteria:
  - semantic coupling
evidence:
  - ADR-014
  - changed contract
  - dependency analysis
recommended_action: review boundary ownership
deterministic_rule_candidate: null
```

The exact schema is experimental; the important property is that the result is machine-readable, evidence-linked and auditable rather than an unstructured review paragraph.

### Calibration is part of the artifact

Agentic fitness functions should be calibrated against previously classified changes before becoming consequential. Useful controls include:

- a versioned calibration set;
- explicit rubrics rather than a generic "review the architecture" prompt;
- evidence scoping rather than unrestricted repository access;
- model, prompt, rubric and tool versions recorded with results;
- confidence and missing-evidence reporting;
- repeated-run variance / judge disagreement checks;
- escalation for low confidence, disagreement or high blast radius;
- periodic recalibration after evaluator changes.

Repeated agentic findings can eventually be converted into deterministic rules. This gives the system a learning path without allowing an uncalibrated model to become the architecture authority.

## Policy-as-code

Policy-as-code is the complementary enforcement mechanism for constraints that are naturally expressed as rules over structured input. OPA/Rego and Conftest are representative examples; architecture linters and dependency-rule tools provide a related capability at the code level.

EOKS should treat these as **policy/evidence providers**, not as part of the semantic core. A policy engine can answer questions such as:

- Is this dependency allowed?
- Does this deployment configuration satisfy the policy?
- Is a required artifact present?
- Does a contract or report meet a threshold?
- Does a change violate an organizational constraint?

The control plane can then decide whether a failed policy blocks a run, triggers repair, requests human approval, or merely records a warning.

## Spec-driven development and the artifact chain

Modern SDD tools such as GitHub Spec Kit, OpenSpec and AWS Kiro demonstrate a common pattern: structure work into artifacts such as constitution/principles, requirements, design, tasks and acceptance criteria rather than relying on a single prompt.

EOKS should not become a competing SDD framework. The useful architectural insight is the **artifact graph**:

```text
principles / constitution
        |
        v
ADR / decision
        |
        v
requirements / specification
        |
        v
acceptance criteria + invariants
        |
        v
plan / tasks
        |
        v
implementation
        |
        v
validation + fitness functions
        |
        v
outcome / evidence
```

The artifacts should remain versioned and traceable. A change should be able to answer which requirement it implements, which decisions constrain it, which checks validate it, and what evidence justified acceptance.

## One canonical fact, multiple views

A key design lesson from the broader SDD/software-factory direction is to avoid maintaining equivalent facts independently in prose, diagrams, rules and generated artifacts.

Prefer:

```text
canonical semantic fact
        |
        +--> human-readable explanation
        +--> machine-readable contract
        +--> executable fitness/policy check
        +--> derived visualization
```

For example, a boundary declaration can be represented as a policy and rendered as a dependency graph. An API contract can drive validation and also generate documentation. The human view should explain the machine-enforced fact rather than becoming a second, silently divergent source of truth.

This aligns with EOKS's existing distinction between assets, representations, providers and context: a representation is useful because it answers a question; it should not automatically become the canonical authority.

## Where this belongs in EOKS

This research reinforces, rather than replaces, the current EOKS model:

- **ADR / Decision** is durable architectural intent and rationale.
- **Specification / contract / invariant** is a governed description of what must be true.
- **Policy / fitness function** is an executable enforcement or measurement mechanism.
- **Provider** supplies evidence from checks, analyzers, tests and policy engines.
- **Evaluation** measures the quality and reliability of the evidence and the resulting workload outcome.
- **Task / Run / Decision / Outcome** provide the control-loop state around the change.
- **Context compilation** selects the relevant decisions, specifications, policies and evidence for an agent rather than dumping the whole repository into context.

This suggests that EOKS does **not** need to make `FitnessFunction`, `Specification` or `PolicyEngine` new core runtime primitives yet. They can initially be governed resource types / capabilities and evidence providers associated with the existing `Policy`, `Evaluation`, `Decision` and `Asset` concepts.

## A stronger software-engineering control loop

For coding-agent workloads, the practical loop becomes:

```text
change request
     |
     v
identify relevant decisions/specs/policies
     |
     v
compile scoped context + evidence contract
     |
     v
agent implements
     |
     v
run deterministic fitness functions / policies
     |
     +---- hard violation ----> repair / reject
     |
     v
run independent review / agentic fitness where useful
     |
     +---- low confidence / disagreement ----> human
     |
     v
behavioral validation
     |
     v
Outcome + Evaluation
     |
     +---- repeated finding ----> candidate deterministic rule
     |
     +---- decision invalid ----> supersede ADR/spec/policy
     |
     +---- accepted ----> record evidence and learn
```

The important shift is from **"agent writes code and a human reviews it"** toward **"intent is represented by artifacts, conformance is continuously tested, and agents operate inside an evidence-and-policy loop."**

## Tools and prior art to track

These are examples of capabilities, not EOKS dependencies:

| Area | Examples | EOKS interpretation |
|---|---|---|
| SDD | GitHub Spec Kit, OpenSpec, AWS Kiro | structured intent/requirements/work planning |
| ADRs | MADR, adr-tools, OpenSpec ADR schemas | durable decision/rationale |
| Architecture fitness | ArchUnit-style tests, dependency-cruiser, archlint | executable architecture invariants |
| Policy-as-code | OPA/Rego, Conftest | executable policy over structured evidence/configuration |
| Agentic fitness | agentic fitness-function patterns / ADK reference implementations | calibrated judgment over bounded evidence |
| Agent governance | hooks, CI gates, CODEOWNERS and review workflows | enforcement and escalation boundaries |
| Contract validation | OpenAPI/AsyncAPI validators and contract tests | executable interface specifications |

The list should evolve as experiments establish which abstractions are useful rather than becoming a catalog of tools.

## Open questions

- Should EOKS define a portable representation for an invariant and its enforcement bindings?
- How should a policy reference the evidence provider(s) required to evaluate it?
- How should a fitness function declare authority: hard gate, advisory, or human-review trigger?
- How should superseding an ADR update or retire its enforcement automatically?
- Can context compilation select the minimal evidence pack required by a particular policy or fitness function?
- How should policy/fitness results become durable evaluation evidence without being mistaken for ground truth?
- When does an agentic finding have enough calibration evidence to become a hard gate?
- Can the control plane learn which validation path is the cheapest reliable one for a workload?

## References

- [GitHub Spec Kit](https://github.com/github/spec-kit)
- [OpenSpec](https://github.com/Fission-AI/OpenSpec)
- [Open Policy Agent](https://www.openpolicyagent.org/)
- [Conftest](https://www.conftest.dev/)
- [ArchUnit](https://www.archunit.org/)
- [Evolutionary Architecture](https://www.buildingevolutionaryarchitectures.com/)
