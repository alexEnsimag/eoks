# Context quality model

“Context quality” is a measurable property of the information available to a model for a reasoning step. It is not a synonym for a good prompt, a large context window, or a particular retrieval technology.

## Context as a constrained working set

Let a task have potentially useful information items `I`. The model receives a selected working set `C ⊂ I`, subject to access, scope and resource budgets.

The objective is not:

```text
maximize |C|
```

It is closer to:

```text
maximize expected task utility(C)
subject to cost(C) <= budget
```

Task utility includes correctness, completeness, decision quality and evidence quality. Cost includes model tokens, latency, retrieval/tool work and infrastructure cost.

The same repository can therefore require very different working contexts for different tasks, models and workflow stages.

## Dimensions

### Relevance

Does the information help the current task or decision?

### Coverage

Does the context contain the facts or evidence required to solve the task?

### Precision

How much supplied information is unnecessary for the task?

### Redundancy

How much information repeats other information without adding useful evidence?

### Contradiction

Does the context contain mutually inconsistent claims, instructions or versions of the same fact?

### Freshness

Is the information current for the task's repository revision and time horizon?

### Authority

How trustworthy is the source for the claim being made?

### Provenance

Can the system explain where the information came from and which revision produced it?

### Structure

Can the model distinguish facts, constraints, decisions, hypotheses, evidence and generated suggestions?

### Dependency completeness

Does the context contain the surrounding relationships needed to interpret the selected item correctly?

### Cost

How much token, latency, tool and infrastructure budget does acquiring and presenting the context consume?

These dimensions can conflict. More coverage can increase redundancy; provenance can add tokens; fresh information can conflict with historical information; a smaller context can be cheaper but omit a critical dependency.

## Context quality is not context size

A larger context window does not imply that a larger working set is better. Academic long-context research and software-agent studies provide evidence that relevant information can become harder to use as contexts grow, while other work shows that agents can successfully process large repositories by navigating them through tools. EOKS therefore treats context size, composition and acquisition strategy as separate experimental variables.

Benchmarks should include both:

1. **fixed-budget composition tests** — vary what is supplied while approximately holding tokens constant;
2. **budget/frontier tests** — measure the best achievable outcome as the context budget changes.

## Context quality versus context acquisition

Context quality describes the resulting working set. **Context acquisition** describes the work required to obtain it.

```text
repository / knowledge / evidence
            |
       acquisition
   (search / graph / LSP /
    retrieval / exploration)
            |
       working context
            |
         model
```

An intervention can improve acquisition efficiency without changing the final context, or improve context quality while requiring expensive preprocessing. Both effects must be measured.

Raw agent exploration is therefore a valid baseline, not an error to be eliminated. Grep/read/edit/tool use may be part of semantic reasoning. Infrastructure should only be credited for replacing exploration when the experiment shows that the replacement preserves or improves outcomes.

## Working context versus execution state

Not everything useful for a long-running agent belongs in the current context.

**Working context** answers:

> What information should the model see for this reasoning step?

**Execution state** answers:

> What has the workflow already observed, established, changed, attempted, invalidated or verified?

Execution state can prevent redundant work without repeatedly injecting the entire history into the model. It should therefore be evaluated separately from context compression or memory retrieval.

## Selection signals

Potential signals for selecting a block include:

```text
relevance
+ dependency proximity
+ source authority
+ freshness
+ task/workflow history
+ prior usefulness
+ evidence coverage
- redundancy
- contradiction risk
- token/tool cost
```

This is a hypothesis space, not a recommendation to implement a linear weighted sum. Learned, rule-based, model-driven and hybrid policies should be compared empirically.

## Context compilation

Selection and compilation should remain separate concepts:

```text
knowledge / evidence / execution state
                |
                v
        candidate information
                |
                v
          selection policy
                |
                v
       semantic working set
                |
                v
       model-specific compiler
                |
                v
        actual model call
```

This permits the same semantic working set to be rendered differently for different models and budgets.

## Progressive disclosure

A useful strategy is to provide a compact pointer or structural answer and make deeper evidence available on demand:

```text
compact bootstrap
      |
      +--> source file
      +--> graph neighborhood
      +--> ADR / knowledge record
      +--> analyzer result
      +--> test / runtime evidence
```

But proactive, reactive and hybrid delivery are competing hypotheses. Measure them rather than assuming progressive disclosure always wins.

## Context diffing and marginal value

A useful evaluation primitive is:

```text
C1 = baseline
C2 = C1 + block X
```

Measure both the task outcome difference and the acquisition/presentation cost.

```text
marginal value(X) =
    Δ task utility / Δ total cost
```

The numerator should use an end-to-end task metric, not a retrieval score alone. Repeated experiments can reveal that a block is essential, useful, irrelevant or misleading for a workload class.

## Model dependence

Context policies should not be assumed to transfer unchanged between models. A tool may provide little value to a frontier model while substantially helping a cheaper model, or the reverse.

A core EOKS experiment is therefore:

```text
same task + repository + intervention
             |
       ┌─────┴─────┐
       v           v
   strong model  cheaper model
       |           |
       └─────┬─────┘
             v
       compare outcome,
       cost and gap
```

One important hypothesis is whether intelligence infrastructure can reduce the capability/cost gap between models. This must be measured rather than assumed.

## Repository dependence

The same intervention may behave differently in:

```text
AI-native / well-structured
          |
modern / mature
          |
legacy / poorly documented / heterogeneous
```

Legacy systems may contain more implicit knowledge, cross-language dependencies, generated code, missing tests and historical conventions. Benchmarks should therefore report repository characteristics rather than treating all codebases as equivalent.

## Why “entropy” is not yet a metric

Entropy is a useful intuition for noisy or uncertain context, but it should not become a formal metric without a defined probability model and evidence that it predicts outcomes. Prefer observable measures such as contradiction density, semantic redundancy, unresolved references, stale-source rate and context-selection errors until a stronger metric is validated.

## Research questions

- Can context-quality measures predict task outcome before execution?
- Which dimensions generalize across models and workload types?
- How much repository exploration is productive semantic reasoning versus avoidable discovery work?
- Can infrastructure reduce acquisition cost without reducing understanding?
- When do graphs, semantic indexes, retrieval, LSP or agentic search outperform raw exploration?
- Does execution state reduce redundant work without causing stale assumptions?
- Can context policies improve cheaper models more than frontier models?
- How different are results on legacy systems versus AI-native repositories?
- Can context policies learn from successful and failed runs without overfitting?

The answer to these questions determines whether particular context mechanisms deserve to become EOKS capabilities. The model itself remains a research variable, not a fixed assumption.