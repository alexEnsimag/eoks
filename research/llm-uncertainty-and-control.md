# LLM uncertainty, semantic entropy and control

This note captures an engineering-oriented view of LLM reliability that emerged from the EOKS research: prompting should not be treated as a mature predictive engineering discipline; LLM applications should instead expose measurable signals, calibrate them against outcomes and use them as control inputs for workflows and graphs.

## Why this belongs in EOKS

A language model is a probabilistic inference system. For a fixed model and decoding configuration, generation can be viewed as repeated evaluation of a conditional distribution:

```text
P(next token | context, previous tokens)
```

Training and inference should be kept conceptually separate:

```text
training
  -> data
  -> self-supervised / supervised objectives
  -> optimization
  -> parameters

inference
  -> fixed parameters
  -> context
  -> probability distributions
  -> generated outcome
```

Prompt or context design changes the input to the learned function; it does not normally retrain the model. This is why “prompt engineering” should not be confused with machine-learning engineering. Many prompting techniques are useful empirical interventions, but a measured improvement is not by itself an explanation of causality or a general engineering law.

EOKS should therefore prefer **evaluation-driven context and workflow engineering**: formulate a hypothesis, measure outcomes, inspect failure modes and retain evidence about when an intervention works.

## The useful mathematical substrate

LLM inference exposes a natural family of probabilistic signals.

### Token probabilities and log probabilities

If token probabilities are available, the model provides the probability assigned to generated tokens. Log probabilities are easier to aggregate numerically. Useful derived signals include:

- mean or length-normalized log probability;
- top-k probability margin;
- token-level predictive entropy;
- entropy trajectories over a generation;
- unusually low probability on critical spans such as names, numbers, tool arguments or structured fields.

These are **model uncertainty signals**, not automatically probabilities that the final answer is correct. A fluent but false statement can still have high token probability.

### Predictive entropy

For a distribution over possible next tokens, Shannon entropy is:

```text
H(Y | x) = -Σ_y P(y | x) log P(y | x)
```

Low entropy means probability mass is concentrated; high entropy means the model is distributing probability over several alternatives.

This makes entropy a natural candidate for an execution signal: a workflow can treat high uncertainty as a reason to retrieve more evidence, verify, branch, sample alternatives or escalate.

### Semantic entropy

Token entropy is not enough for free-form language because different token sequences can express the same meaning. For example, “Paris”, “The capital is Paris” and “France's capital is Paris” are lexically different but semantically equivalent.

Semantic entropy addresses this by sampling multiple generations, grouping meaning-equivalent answers and computing entropy over the resulting semantic clusters. Farquhar et al. demonstrated that semantic entropy can detect a useful class of hallucinations/confabulations and can support selective refusal or additional grounding. See the references below.

This is especially relevant to EOKS because the desired control variable is often **uncertainty about the task outcome**, not uncertainty about the exact wording.

### Self-consistency

Sampling several reasoning paths and selecting the most consistent answer is another probabilistic signal. Agreement is evidence, but it is not proof: correlated errors can make many samples agree on the same wrong answer.

Therefore:

```text
agreement != correctness
```

Agreement becomes much stronger when combined with independent evidence, deterministic validators or calibrated historical performance.

## Confidence must be calibrated

A raw model score should not be interpreted as a trustworthy probability of correctness without validation.

The engineering question is not:

> “Does this signal correlate with correctness?”

but:

> “For this workload and decision, does this signal produce a calibrated estimate of the probability/risk we care about?”

Useful calibration and evaluation concepts include:

- reliability diagrams;
- Expected Calibration Error (ECE);
- Brier score for probabilistic correctness decisions;
- AUROC when the signal is primarily used for ranking correct vs. incorrect outcomes;
- risk-coverage / rejection curves when the system can abstain or escalate;
- workload-specific post-hoc calibration such as Platt scaling or isotonic regression.

Calibration is empirical. A raw entropy or logprob threshold should be learned/evaluated on representative labelled outcomes and revisited after model, prompt/context, task-distribution or decoding changes.

## Three different quantities

EOKS should keep these distinct:

```text
model uncertainty
      !=
evidence strength
      !=
probability that the outcome is correct
```

They can be related, but they are not interchangeable.

Likewise:

```text
observability
  -> what happened?

reliability estimation
  -> how trustworthy is this outcome for this decision?

control policy
  -> what should happen next?
```

A useful control system combines model-native signals with external evidence instead of expecting one universal “confidence” number.

## From confidence to graph control

For an EOKS workflow/graph, each node should produce not only an output but an **evaluation state** that can drive the next transition.

Conceptually:

```text
state
  -> action / model / tool
  -> observation
  -> evaluation
  -> decision policy
       |       |       |       |
      stop   verify  retrieve  branch/escalate
```

A node contract should define at least:

1. **Goal** — what outcome or uncertainty is the node trying to resolve?
2. **Evidence** — what observations support the current state?
3. **Quality/reliability signals** — what measurable indicators are available?
4. **Acceptance criteria** — what must be true before the node can terminate?
5. **Continuation policy** — what evidence would justify another iteration?
6. **Budget and safety limits** — maximum cost, latency, iterations or other resources.

### Stop conditions

Avoid arbitrary rules such as “run five times” when a measurable criterion is available.

Potential stop conditions include:

```text
validator passes
required evidence coverage reached
semantic uncertainty below calibrated threshold
independent attempts converge
no new relevant evidence is discovered
marginal information gain falls below ε
expected value of another step is below its cost
```

No one condition is universally correct. The stop rule is part of the workload's policy.

### Branching

A graph can make the same evidence drive different actions:

```text
                    evaluation
                        |
          +-------------+-------------+
          |             |             |
       sufficient     uncertain      failed
          |             |             |
         stop       retrieve       repair/replan
                        |
                    evaluate again
```

This turns an agent graph into a control system rather than a fixed chain of prompts.

## Marginal information gain

A useful research hypothesis is to measure the value of another step rather than the number of steps taken.

For a state `s`, an action `a` can be evaluated by the expected change in task-relevant information or expected task utility relative to its cost:

```text
continue when:

E[value of new evidence | s, a] > cost(a)
```

In practice, “information gain” may need a workload-specific proxy: new entities discovered, unresolved claims reduced, evidence coverage increased, test failures removed, contradiction risk reduced, or evaluator score improved.

This is intentionally a research direction rather than a claim that one universal information-gain metric exists for all agent workloads.

## Long-horizon workflows

Single-turn uncertainty does not transfer directly to multi-step agents. A trajectory introduces uncertainty about:

- user intent;
- world state;
- retrieved evidence;
- tool arguments;
- action consequences;
- intermediate state;
- final outcome.

Naively multiplying step probabilities is only justified under strong assumptions such as a suitable definition of per-step success and conditional independence. Real agent steps are correlated, and later verification can compensate for earlier uncertainty.

Therefore EOKS should record both **step-level evidence** and **trajectory-level outcome** and empirically learn how signals propagate for each workload class.

## Semantics and information compression

LLMs are unusually effective at semantic transformation: summarization, paraphrase, extraction, classification and cross-formulation reasoning. A useful theoretical lens is representation learning and information compression rather than a claim that the model contains explicit dictionary-like semantic concepts.

Training rewards representations that preserve distinctions useful for predicting future tokens. Repeated contextual regularities can therefore produce latent geometry in which semantically related patterns become easier to manipulate. This explains why semantic structure can emerge from a next-token objective without an explicit symbolic “meaning” loss.

For EOKS, the practical consequence is important:

```text
surface form
   -> semantic representation
   -> task-specific transformation
   -> surface form
```

Context engineering should exploit this strength while using structural and deterministic evidence where exactness matters. Semantic similarity is not a substitute for source provenance, symbolic analysis or tests.

## Prompt engineering versus engineering

EOKS should avoid presenting prompt recipes as universal laws. A useful intervention should be treated like an experiment:

```text
hypothesis
  -> controlled intervention
  -> representative evaluation
  -> effect size + uncertainty
  -> failure analysis
  -> promotion / rejection
```

For example, “adding a delimiter improved benchmark accuracy by 15%” is evidence about that experiment. It is not yet an explanation of why delimiters work, nor evidence that the effect transfers to another model or workload.

This distinction is important for the EOKS research corpus: preserve empirical effects, proposed mechanisms and demonstrated generality as separate claims.

## Implications for EOKS observability

A future EOKS execution trace should be able to capture, where available:

```text
model / version / decoding configuration
prompt/context manifest
input and output token counts
logprob-derived signals
entropy or uncertainty trajectory
semantic-agreement / semantic-entropy signals
retrieval and evidence coverage
validator/test results
tool outcomes
cost and latency
policy decision
termination reason
final outcome
```

Hosted models may not expose all internal signals. The architecture should therefore support a hierarchy:

1. deterministic external evidence;
2. provider-exposed probability/logprob signals;
3. sampling-based semantic uncertainty;
4. agreement and consistency signals;
5. learned workload-specific reliability models.

The absence of logprobs must not make the control plane blind; it changes which estimator is available and its cost/limitations.

## Research questions

- Which token-level signals predict correctness for each EOKS workload class?
- When does semantic entropy outperform token-level entropy?
- How should step-level uncertainty be aggregated into trajectory-level risk?
- Can stopping policies be learned from calibrated uncertainty and marginal value?
- Which signals remain useful when the model does not expose logprobs?
- How should correlated agents/attempts be handled when estimating confidence?
- Can evidence coverage and semantic uncertainty be combined into calibrated selective prediction?
- How much do model/context changes alter calibration?
- Which representations preserve semantic information while losing the least task-critical detail?
- Can EOKS learn model × context × workload affinity from production outcomes?

## Prior art and references

- Farquhar, Kossen, Kuhn & Gal, **Detecting hallucinations in large language models using semantic entropy**, Nature 630 (2024): https://doi.org/10.1038/s41586-024-07421-0
- Wang et al., **Self-Consistency Improves Chain of Thought Reasoning in Language Models** (2022): https://arxiv.org/abs/2203.11171
- Sharma & Chopra, **Think Just Enough: Sequence-Level Entropy as a Confidence Signal for LLM Reasoning** (2025): https://arxiv.org/abs/2510.08146
- Huang, Lu & Zeng, **Calibrated Language Models and How to Find Them with Label Smoothing** (ICML 2025): https://proceedings.mlr.press/v267/huang25w.html
- Jenane et al., **From Entropy to Calibrated Uncertainty: Training Language Models to Reason About Uncertainty** (2026): https://arxiv.org/abs/2603.06317

These references demonstrate that probabilistic uncertainty, semantic entropy, calibration and uncertainty-aware stopping are active research areas. They should be treated as prior art and experimental evidence, not as settled engineering standards.
