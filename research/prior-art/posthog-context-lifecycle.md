# PostHog context lifecycle — prior art

Source: Jina Yoon, **Read this before deleting your AGENTS.md**, PostHog, Aug 31 2026: https://newsletter.posthog.com/p/your-agentsmd-is-holding-you-back

This is practitioner evidence, not a new EOKS concept. The useful question is which existing EOKS hypotheses it validates or sharpens.

## What PostHog demonstrates

PostHog describes a production workflow in which context is actively inspected, trimmed, evaluated and updated rather than treated as static documentation. The article reports:

- inspection of resident context/token cost and unused capabilities;
- an example where always-loaded `AGENTS.md` guidance imposed measurable resident cost;
- a stale `AGENTS.md` instruction whose relevant dependency lived in external GitHub state;
- saving agent failures as regression cases when context changes are made to address them;
- automated evaluation of coding-agent runs after context changes;
- collecting agent feedback about missing or misleading guidance;
- clustering and independently checking feedback before changing durable context.

These observations reinforce existing EOKS concepts rather than requiring another architectural layer.

## Mapping to EOKS

### Context as a managed working set

The resident-context example is practical evidence for the existing EOKS model of context as a capacity- and cost-constrained materialization of a workload working set. A context item can have a carrying cost even when it is irrelevant to the current task.

The relevant optimization target is therefore not “more context” or “fewer tokens” in isolation, but useful verified work per unit of total reasoning cost.

### Scope and progressive disclosure

PostHog's argument against accumulating instructions in `AGENTS.md` supports the existing EOKS distinction between durable project guidance and task-specific context. Persistent guidance should be concise, scoped and high-value; detailed knowledge can remain available through progressive disclosure.

This is consistent with EOKS's existing context locality, loadout and context-compilation model. It does **not** imply that `AGENTS.md` should disappear.

### Freshness and external dependencies

The external GitHub-state example strengthens an existing maintenance principle: context validity cannot be inferred from context text alone. A context artifact may depend on repository state, tooling configuration, organizational policy or external service state.

Such dependencies are candidates for provenance, freshness checks and targeted invalidation. Consequential changes still require validation rather than assuming that a freshness signal proves semantic correctness.

### Evaluation and regression

PostHog treats a context change much like a code change: if a context edit fixes a concrete agent failure, the failure can become a regression case and be rerun after later context changes.

This is a useful operationalization of EOKS's existing evaluation loop:

```text
observed failure
      -> context hypothesis
      -> context revision
      -> regression evaluation
      -> keep / revise / reject
```

It strengthens the case for evaluating context interventions on end-to-end outcomes, not only resident tokens, retrieval counts or latency.

### Feedback and promotion

PostHog's feedback loop treats agent reports as observations. Feedback is clustered and checked before becoming a durable context change.

This fits EOKS's existing distinction between observation, candidate knowledge, validation and promotion. Agent feedback is evidence about the system; it is not automatically authoritative truth.

## Relationship to GrapeRoot

The two examples illuminate different parts of the same existing EOKS model:

```text
resources / evidence
        |
        v
working-set selection
        |
context compilation  <---- GrapeRoot-like systems
        |
     agent run
        |
 outcome / observations
        |
 evaluation + maintenance <---- PostHog provides practitioner evidence
        |
 refresh / invalidate / revise
        |
        +-------------------------->
```

GrapeRoot is primarily useful prior art for repository understanding, working-set selection, context packing and agent integration. PostHog is useful prior art for observing the cost and effectiveness of persistent context and feeding validated findings back into its maintenance.

Neither is an EOKS replacement. The EOKS hypothesis is the composition: **context acquisition/compilation inside a broader workload control and maintenance loop**.

## What should *not* be added to EOKS

The PostHog article does not justify new canonical concepts such as “AGENTS.md optimization” or a separate “context control loop.” EOKS already has:

- working sets, context compilation and context misses;
- locality, progressive disclosure and cache-management hypotheses;
- knowledge/context lifecycle and invalidation;
- continuous knowledge maintenance;
- observation → validation → promotion;
- end-to-end context evaluation;
- workload reconciliation/control loops.

Likewise, “selection / shaping / evolution” can be a useful analysis of context operations, but should not become another required EOKS taxonomy. Those responsibilities already exist across selection, compilation, representation transformation, maintenance and invalidation.

## Research implications

PostHog supplies concrete hypotheses for the existing benchmark program:

1. Compare always-loaded guidance with concise routing plus progressive disclosure.
2. Measure resident context cost alongside end-to-end outcome and discovery cost.
3. Test how external-state freshness affects context reliability.
4. Preserve context-related failures as regression cases and test subsequent context revisions against them.
5. Compare manual maintenance with feedback-driven maintenance plus independent validation.
6. Measure whether lazy loading reduces total cost without increasing harmful discovery misses.

These remain experiments, not design conclusions. Results should be evaluated across model × repository × task × intervention × budget.

## Evidence boundary

This is a single practitioner's production account. It is valuable because it demonstrates concrete lifecycle mechanisms and failure modes, but it does not establish that trimming, lazy loading, proactive context, or any other policy is universally better. EOKS should reproduce the relevant claims under controlled evaluation.
