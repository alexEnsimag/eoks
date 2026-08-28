# Agent workflows and reasoning strategies

EOKS should distinguish **knowledge**, **workflow**, and **reasoning strategy**. They are related but not interchangeable.

- Knowledge answers: **What do I know?**
- Workflow answers: **What should happen next?**
- Reasoning strategy answers: **How should I approach this step?**

## Workflow is an execution layer

A workflow is an explicit sequence or graph of actions, decisions and validation steps:

```text
                 goal
                   |
                discover
                   |
             plan / design
               /       \
          implement   review
               \       /
                 verify
                   |
                 result
```

The workflow should not contain the entire project knowledge base. Each node requests the context it needs from the knowledge/context planes.

```text
workflow node
     |
     v
context selection
     |
     v
relevant evidence
     |
     v
reasoning strategy
     |
     v
model + tools
```

This separation allows the same knowledge to support multiple workflows and the same workflow to run over different models.

## Reasoning strategies are another reusable layer

Recent experiments around so-called "ADHD" skills/stacking are useful not because of the label, but because they expose a more general idea: **cognitive strategies can be reusable execution components**.

Examples:

- divergent exploration before convergence;
- adversarial review;
- hypothesis generation and falsification;
- threat modeling;
- architecture review;
- performance investigation;
- test-first verification.

A workflow can therefore select a strategy per node:

```text
architecture task
  -> divergent design
  -> tradeoff analysis
  -> adversarial review
  -> decision
```

This is more durable than treating a single prompt or "mode" as the strategy.

## Workflows are not a replacement for knowledge

A workflow can say:

```text
understand -> design -> implement -> verify
```

It does not know the architecture itself. The context/knowledge system supplies the evidence needed by each step.

Conversely, a knowledge base can contain excellent architectural facts without defining how an agent should act on them.

## Relationship to natural-language programming

Agentic workflows suggest a higher-level programming abstraction: humans increasingly specify goals, constraints and process, while the system expands them into tool calls and code changes.

This does not imply that programming languages disappear. A useful layered model is:

```text
human intent / policy
        |
workflow specification
        |
agent execution
        |
programming languages / tools
        |
software artifacts
```

Natural language is therefore better viewed as a new control/specification layer than as an immediate replacement for deterministic programming languages.

## Workflow quality and confidence

A mature workflow should expose checkpoints and evidence requirements rather than trusting the agent's self-reported confidence.

For example:

```text
plan
 |
 +-- architecture evidence
 +-- relevant tests
 +-- historical decisions
 |
implement
 |
verify
 |
 +-- tests pass
 +-- static analysis pass
 +-- expected behavior observed
```

Confidence can be assembled from evidence, not only from the model's subjective probability.

## Continuous learning

Workflows are also one of the cleanest sources of learning signals. A completed run produces:

- decisions;
- tool traces;
- failures;
- corrections;
- test outcomes;
- review feedback;
- final artifacts.

These can produce **candidate** procedures or knowledge, but repeated behavior alone is not enough to promote a rule. Outcome-linked evidence, scope and counterexamples should be considered before promotion.

## EOKS placement

A useful high-level split is:

```text
KNOWLEDGE PLANE
  canonical knowledge
  structural graph
  semantic indexes
  history
  runtime evidence

EXECUTION PLANE
  workflows
  reasoning strategies
  agents
  tools

CONTEXT PLANE
  task-specific selection and assembly

EVALUATION
  evidence, quality, confidence, outcomes
```

The control plane coordinates all four.
