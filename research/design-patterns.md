# Recurring EOKS design patterns

## 1. Compile context, do not concatenate history

Raw conversation history is a poor universal context representation. A context compiler should select, normalize, deduplicate and order information for a particular workload.

## 2. Separate discovery from working-set composition

The system may discover hundreds of relevant artifacts while only a subset belongs in the model's immediate working context.

```text
available knowledge != working context
```

## 3. Make provenance first-class

Every important piece of generated context should ideally be traceable to an origin: source file, commit, decision, tool output, previous task, or external source.

## 4. Preserve uncertainty

A hypothesis should not silently become a fact because it was retrieved from memory. Represent claims, evidence and confidence separately where useful.

## 5. Use the cheapest sufficient analysis

Do not invoke the most sophisticated tool by default. Let workload requirements determine whether simple search, graph analysis, static analysis or deeper model reasoning is justified.

## 6. Treat models as replaceable execution resources

A model is an implementation choice subject to policy and evaluation. Context and task representations should be portable enough to test alternatives.

## 7. Close the loop

Execution without evaluation creates no reliable learning signal. Evaluation without policy change is merely reporting.

The useful loop is:

```text
intent -> plan -> execute -> observe -> evaluate -> update policy/memory -> next task
```

## 8. Human interaction should target the right abstraction

If humans need to intervene, expose context blocks, evidence, decisions and policies rather than forcing them to edit raw prompts.

## 9. Prefer layered evidence

LLM reasoning, deterministic analysis, tests and source provenance should reinforce each other. The system should know which layer produced each claim.

## 10. Keep representations replaceable

YAML, Markdown, JSON, graph databases, vector stores and relational databases are implementation choices. EOKS should define semantics and contracts before locking storage.

## 11. Measure tradeoffs, not only quality

Every EOKS intervention can have cost:

- tokens;
- latency;
- compute;
- tool execution;
- engineering complexity;
- human attention.

A better result is not automatically a better system if it costs an order of magnitude more.

## 12. Preserve failed hypotheses

Because the project is research-oriented, rejected ideas are valuable. An architecture decision should record why an alternative was rejected and what evidence could reopen it.