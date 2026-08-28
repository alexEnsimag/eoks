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

For LLM execution, distinguish:

```text
model uncertainty != evidence strength != probability of correctness
```

Where available, retain model-native uncertainty signals such as logprob-derived entropy, but calibrate them against actual outcomes before using them as decision thresholds.

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

For probabilistic workflows, evaluation should be an input to the next control decision, not merely a post-run metric.

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

## 13. Engineer termination conditions, not iteration counts

A workflow should prefer measurable stop conditions over arbitrary loop limits when practical:

```text
state
  -> action
  -> observation
  -> evaluation
  -> stop / continue / verify / branch
```

Candidate termination evidence includes validator success, evidence coverage, calibrated uncertainty, independent agreement, lack of new information, marginal information gain and expected value versus cost.

## 14. Treat graph edges as policies

A graph edge should represent a decision policy, not merely “the next prompt”. The policy can use evaluation state to select among retrieval, verification, repair, replanning, escalation or termination.

## 15. Preserve experiment versus explanation

An observed improvement is evidence about an intervention, not automatically an explanation of its mechanism or a general law. Record:

```text
hypothesis
  -> intervention
  -> measured effect
  -> failure analysis
  -> scope / generality
```

This avoids turning prompt folklore into architecture assumptions.
