# Tool capability model

EOKS needs to compare tools in a way that supports **selection**, not just ranking.

An overall score is useful for a quick impression, but it is not sufficient for choosing a tool. A tool can be excellent overall and still be the wrong provider for a particular question.

The canonical representation should therefore be a **capability profile**. Comparison tables and selection views should be derived from these profiles rather than becoming independent sources of truth.

## Core distinction

```text
Tool quality
  = how good the tool is in general

Tool capability
  = what kind of evidence the tool can produce

Evidence requirement
  = what evidence a particular question needs

Tool selection
  = which provider can satisfy the requirement with sufficient reliability and acceptable cost
```

This distinction prevents a global score from becoming an automatic routing mechanism.

## Capability dimensions

Tool profiles should describe the dimensions that materially affect selection. The exact scale can evolve as experiments provide evidence.

| Dimension | Meaning |
|---|---|
| Evidence kind | Pattern, type, structural, dataflow, semantic, runtime, behavioral, architectural, etc. |
| Scope | File, package, repository, organization, runtime/system, or broader |
| Depth | Local matching through interprocedural/system-level reasoning |
| Precision | How reliably a positive result means the claimed property holds |
| Recall | How much of the relevant population the tool can find |
| Determinism | Whether the same input normally produces the same result |
| Freshness | How quickly results reflect current source/runtime state |
| Latency | Time required to obtain useful evidence |
| Cost | Compute, licensing, infrastructure and operational cost |
| Setup | Integration and maintenance burden |
| Explainability | How directly a result can be inspected and justified |
| Provenance | Ability to identify the exact source and method behind evidence |

These should be treated as **properties**, not collapsed into one score.

## Canonical tool profile

A significant tool should have a compact profile containing:

```text
Identity
Category
Capabilities
Strengths
Weaknesses
Best-fit questions
Poor-fit questions
Evidence characteristics
Operational characteristics
Dependencies
Complements
Alternatives / overlaps
Known limitations
Overall score (optional, for coarse browsing)
```

The profile should answer both:

> What is this tool good at?

and:

> When should I prefer it over another provider?

## Strengths and weaknesses are first-class information

Scores hide important trade-offs. Profiles should explicitly state them.

For example, a deep dataflow analyzer may have excellent precision and coverage but high setup and latency. A pattern matcher may be much faster and easier to author while being fundamentally unable to establish some interprocedural properties.

That distinction is more useful to EOKS than saying one tool is `8.7/10` and another is `8.2/10`.

## Tool relationships

Tools should also record important relationships:

- **Complement** — using both provides materially different evidence.
- **Overlap** — both address substantially similar questions.
- **Alternative** — either can reasonably satisfy the same requirement.
- **Escalation** — one is appropriate when a cheaper provider is insufficient.
- **Specialization** — one is better for a narrower class of questions.
- **Dependency** — one relies on another's output or environment.

These relationships are especially useful for constructing an evidence ladder.

Example:

```text
compiler/type checker
        ↓ if insufficient
lightweight pattern analysis
        ↓ if invariant requires deeper reasoning
dataflow/security analysis
        ↓ if static evidence is insufficient
runtime evidence / tests
```

The ladder is not universal. EOKS should derive it from the evidence requirement and tool capabilities.

## Evidence requirement and provider selection

The missing intermediate abstraction between a task and a tool is an **Evidence Requirement**.

Instead of:

```text
Task → choose tool
```

use:

```text
Task
  ↓
Question
  ↓
Evidence Requirement
  ↓
Candidate providers
  ↓
Capability comparison
  ↓
Minimum sufficient evidence
  ↓
Selected provider(s) + rationale
```

An evidence requirement describes what must be established, not which tool should establish it.

Example:

```yaml
question: "Can this user-controlled value reach SQL execution?"
evidence:
  kind: dataflow
  scope: repository
  depth: interprocedural
  precision: high
  deterministic: required
```

The selection layer can then compare CodeQL, Semgrep, a repository graph, tests, runtime traces and an LLM according to their actual capabilities.

## Minimum sufficient evidence

EOKS should prefer the **cheapest sufficiently reliable provider** that satisfies the requirement.

This is deliberately different from selecting the deepest or most powerful tool.

For example:

```text
Question: "Does this function return a string?"

Type checker → sufficient
CodeQL       → unnecessary
LLM          → unnecessary
```

Whereas:

```text
Question: "Can request input reach this database sink through several calls?"

Type checker → insufficient
Semgrep      → possibly insufficient
CodeQL       → potentially sufficient
LLM          → useful for interpretation, not authoritative for the invariant
```

The selected provider should be accompanied by a rationale that records why alternatives were rejected or considered unnecessary.

## Selection pipeline

Tool selection is a **control-plane decision**, not a separate architectural layer.

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

The conductor should consider:

1. capability fit for the question and required scope/depth;
2. reliability for the particular claim, including precision, recall, determinism and known failure modes;
3. evidence already available from the current run;
4. cost and latency;
5. consequence of being wrong;
6. independence when multiple providers are used for confirmation.

Do not run a provider when existing evidence is already sufficient. Escalate only when current evidence does not satisfy the requirement.

For example:

```text
Question: "Can request input reach a database sink?"
        |
  evidence requirement
        |
 existing evidence?
    |          |
 sufficient   insufficient
    |          |
 continue   select provider
               |
          collect evidence
               |
            evaluate
            /      \
       sufficient  insufficient
          |             |
       continue    escalate/combine/revise
```

## Evidence ladders

A policy can define an escalation path, but it should remain requirement-specific.

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

For another question, the order could be completely different. The important abstraction is therefore not the ladder itself but the rule:

> **Escalate when current evidence does not satisfy the requirement.**

## Comparison views

The repository can expose three related views from the same canonical profiles.

### Capability matrix

Answers:

> What can these tools do relative to each other?

Example:

| Provider | Pattern | Types | Structure | Dataflow | Semantic | Runtime |
|---|---:|---:|---:|---:|---:|---:|
| Pattern analyzer | High | Low | Medium | Low | Low | None |
| Dataflow analyzer | Medium | Medium | High | High | Low | None |
| Repository graph | Low | Medium | High | Medium | Medium | None |
| Tests | Low | Medium | Medium | Medium | Low | High |
| LLM | Medium | Medium | Medium | Variable | High | Variable |

The exact values should eventually come from structured profiles rather than manually maintained prose.

### Pairwise comparison

Answers:

> Why choose A rather than B?

A pairwise view should emphasize meaningful differences, not repeat every field.

```text
A is preferable when:
  - requirement X matters
  - latency must be low

B is preferable when:
  - requirement Y matters
  - deep dataflow is required

They overlap on:
  - requirement Z

Use both when:
  - X and Y must be established independently
```

Pairwise pages should be derived from canonical profiles where possible. Avoid creating a permanent document for every pair.

### Selection rationale

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

This makes provider selection inspectable and evaluable rather than an opaque ranking decision.

## What not to do

Do not create a manually maintained matrix containing every possible pair of tools. That becomes quadratic, stale and difficult to trust.

Do not make the overall score the routing mechanism.

Do not assume that a more powerful analyzer is always better.

Do not treat an LLM's plausible explanation as equivalent to deterministic evidence when the question requires a mechanically verifiable invariant.

The canonical source should remain **structured tool capabilities plus evidence requirements**. Comparison tables, pairwise views and selection recommendations should be derived from that model.

## Future research

The model should be tested empirically rather than expanded indefinitely.

Important experiments include:

- whether evidence requirements can be classified reliably from natural-language tasks;
- whether capability profiles predict useful tool choices;
- whether selecting the minimum sufficient evidence reduces cost without reducing correctness;
- whether escalation policies improve outcomes over fixed tool stacks;
- how to measure evidence independence and conflicting results;
- whether selection decisions transfer across repositories, languages and agent runtimes.
