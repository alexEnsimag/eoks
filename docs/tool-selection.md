# Tool selection

This document describes how EOKS should compare and select tools. The canonical capability model is in [`tool-capability-model.md`](tool-capability-model.md).

## The problem

A list of tools with categories and an overall score is useful for discovery but insufficient for selection.

The relevant question is not:

> Which tool has the highest score?

It is:

> Which provider can produce the evidence this question requires, with sufficient reliability and acceptable cost and latency?

## Selection pipeline

```text
Task
  ↓
Question
  ↓
Evidence Requirement
  ↓
Candidate Providers
  ↓
Capability filtering
  ↓
Reliability / cost / latency trade-off
  ↓
Minimum sufficient evidence
  ↓
Selection + rationale
```

## Example

Question:

> Can a user-controlled value reach a database sink through several calls?

Required evidence might be:

```yaml
kind: dataflow
scope: repository
depth: interprocedural
precision: high
deterministic: required
```

The candidate set might include:

| Provider | Likely fit | Why |
|---|---|---|
| Type checker | Low | Establishes types, not arbitrary dataflow |
| Pattern analyzer | Medium/low | Useful for local patterns, limited for deep flow |
| Repository graph | Medium | Strong structural evidence but may not prove value flow |
| Dataflow analyzer | High | Designed to establish interprocedural flows |
| Tests/runtime traces | Complementary | Strong observed evidence but may not cover all paths |
| LLM | Supporting | Useful for interpretation/hypothesis generation, not authoritative proof |

The selection is therefore not a universal ranking. It is a consequence of the evidence requirement.

## Selection factors

EOKS should consider:

### 1. Capability fit

Can the provider answer the required kind of question at the required scope and depth?

### 2. Evidence reliability

How trustworthy is the result for this particular claim? Consider precision, recall, determinism, provenance and known failure modes.

### 3. Existing evidence

Do not run a provider if already-available evidence is sufficient.

### 4. Cost and latency

Prefer cheaper/faster providers when they meet the requirement. Escalate only when additional evidence is justified.

### 5. Consequence of error

High-impact decisions can justify stronger or independent evidence even when a cheaper provider appears adequate.

### 6. Independence

Two providers based on the same underlying mechanism may not provide genuinely independent confirmation.

## Evidence ladders

A selection policy can define an escalation path, but it should remain requirement-specific.

For a simple software question, a ladder might be:

```text
language tooling
      ↓
lightweight static rule
      ↓
deep static/dataflow analysis
      ↓
test/runtime evidence
      ↓
independent review
```

For another question, the order could be completely different.

The important abstraction is therefore not the ladder itself but the rule:

> **Escalate when current evidence does not satisfy the requirement.**

## Pairwise tool comparisons

Pairwise comparisons are useful when several providers satisfy the same requirement.

The comparison should focus on the dimensions that actually differentiate them:

```text
Provider A
  better: speed, setup
  weaker: deep dataflow

Provider B
  better: dataflow depth, coverage
  weaker: latency, operational cost

Prefer A when:
  local/simple evidence is sufficient

Prefer B when:
  interprocedural/dataflow evidence is required
```

Pairwise pages should be derived from canonical profiles where possible. Avoid creating a permanent document for every pair.

## Selection rationale

A future EOKS run should be able to explain a selection in a compact form:

```yaml
question: "..."
requirement:
  kind: dataflow
  scope: repository
  depth: interprocedural
selected:
  provider: <provider>
  reason:
    - satisfies required evidence kind
    - meets required depth
    - deterministic evidence required
rejected:
  - provider: <provider>
    reason: insufficient depth
  - provider: <provider>
    reason: redundant with existing evidence
```

This makes tool selection inspectable and evaluable rather than an opaque ranking decision.

## Future research

The model should be tested empirically rather than expanded indefinitely.

Important experiments include:

- whether evidence requirements can be classified reliably from natural-language tasks;
- whether capability profiles predict useful tool choices;
- whether selecting the minimum sufficient evidence reduces cost without reducing correctness;
- whether escalation policies improve outcomes over fixed tool stacks;
- how to measure evidence independence and conflicting results;
- whether selection decisions transfer across repositories, languages and agent runtimes.
