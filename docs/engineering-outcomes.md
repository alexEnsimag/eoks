# Engineering outcomes and graduated autonomy

EOKS is ultimately useful to software engineers only if it changes measurable engineering outcomes. The current architecture should therefore be evaluated against four primary outcomes rather than against the sophistication of any individual EOKS component.

## Primary outcomes

| Outcome | Engineering question | What EOKS should improve |
|---|---|---|
| **Velocity** | How quickly can useful work reach an acceptable outcome? | Less repository rediscovery, faster planning, targeted context, fewer blocked/duplicated steps. |
| **Quality** | Did the change satisfy its intent and preserve required properties? | Better context and planning, explicit specifications/invariants, stronger and more appropriate verification. |
| **Trust / autonomy** | How much work can be delegated safely to an agent? | Evidence-backed decisions, independent verification, provenance, calibrated reliability and explicit escalation. |
| **Cost / scale** | How much engineering output can be produced for a given human + AI budget? | Less rework and review burden, minimum-sufficient evidence, efficient resource selection, and safe parallel/autonomous execution. |

**Quality** is the broader outcome; accuracy/correctness, reliability, consistency and resilience are important dimensions of it. **Scale** is related to cost but is not identical: the goal is not merely cheaper engineering, but increasing useful engineering throughput without proportional human involvement.

## Supporting dimensions

Several dimensions are useful diagnostics rather than additional top-level goals:

- **Cognitive load** — how much system knowledge an engineer or agent must reconstruct or retain manually.
- **Risk** — probability and impact of an undetected bad outcome.
- **Explainability** — whether the system can show why it selected context, tools, plans and verification steps.
- **Traceability** — whether requirements, decisions, changes, evidence and outcomes can be connected.
- **Consistency** — whether agents and engineers make compatible decisions under shared constraints.
- **Maintainability** — whether knowledge, specifications and derived representations remain useful as the system evolves.
- **Reproducibility** — whether a workflow and its evidence can be reconstructed sufficiently to understand or investigate an outcome.

These should be measured when they explain changes in the four primary outcomes rather than turned into an ever-growing EOKS ontology.

## Trust is earned by assurance, not by agent confidence

An agent's statement that it completed a task is not sufficient evidence of correctness. EOKS should instead combine multiple evidence sources appropriate to the workload:

```text
intent / requirements
        |
architecture + policies + invariants
        |
relevant system knowledge + repository evidence
        |
       plan
        |
     agent action
        |
 deterministic checks + tests + independent review
        |
  observed outcome + evaluation
        |
 reliability / assurance evidence
        |
   next control decision
```

The important distinction is between **model confidence**, **evidence strength**, and **outcome quality**. They are related signals, not interchangeable quantities. Automated authority should depend on calibrated evidence and policy, not on an LLM's self-assessment alone.

## Graduated autonomy

EOKS should not assume a binary choice between human work and fully autonomous agents. A workflow can operate at different autonomy levels according to task risk and available assurance.

A useful conceptual progression is:

1. **Human-led** — the agent assists; the human owns decisions and validation.
2. **Agent-executed, human-approved** — the agent performs the workflow but a human approves consequential outcomes.
3. **Evidence-gated autonomy** — deterministic and/or calibrated checks can authorize routine outcomes automatically; failures escalate.
4. **Continuous autonomous workflow** — low-risk, well-understood workloads execute end-to-end with monitoring and exception handling.

Moving upward requires stronger evidence, not merely a stronger model.

## The key EOKS loop

For automated software-engineering workflows, the desired loop is:

**Understand → specify → plan → act → verify → evaluate → learn → repeat**

EOKS should make each transition inspectable and should preserve the evidence needed to determine whether an intervention actually improved the outcome.

## Measurement implications

Experiments should report at least:

- task success/correctness and critical regressions;
- human intervention and review effort;
- end-to-end latency / cycle time;
- model/tool/context consumption and monetary or resource cost;
- verification coverage and evidence strength;
- escalation, retry and repair rates;
- failure modes and undetected-failure rate where measurable;
- reproducibility/provenance of the run.

Comparisons should use the same workload and baseline where possible. A system that is faster but less correct, or cheaper but less trustworthy, is not automatically an improvement.

## Architectural consequence

This framing reinforces the existing EOKS architecture rather than adding new runtime primitives. Knowledge representations, evidence providers, context compilation, planning, execution, evaluation, observability and control remain mechanisms. Their purpose is to improve engineering outcomes and progressively increase the amount of work that can be delegated safely.
