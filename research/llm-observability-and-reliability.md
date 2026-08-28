# LLM observability and reliability signals

The recent AI-engineering tooling discussion sharpened an important EOKS boundary: **LLM observability is necessary for the control loop, but it is not the control loop itself**.

Tools such as LangSmith and Langfuse primarily provide tracing, evaluation, analytics and monitoring around model/agent execution. The EOKS question is one layer further: can those observations become reliable signals that help the control plane decide whether to continue, verify, retry, branch, switch models, or stop?

The distinction is:

```text
observability
  -> record what happened

reliability / uncertainty estimation
  -> estimate how much the result should be trusted

control
  -> use evidence and reliability signals to decide what happens next
```

## Why this matters

An agent can complete successfully at the infrastructure level while producing a bad result. A useful control plane therefore needs signals about **task reliability**, not just request health.

Operational signals remain important:

- latency;
- token usage and cost;
- retries and failures;
- tool-call success/failure;
- retrieval activity;
- model/version;
- prompt/context version;
- evaluator scores;
- human feedback.

But EOKS should additionally investigate signals that describe uncertainty or disagreement in the model computation and in the surrounding evidence.

## Observability tools are the substrate

The tools discussed in the AI-engineering article occupy useful but different layers:

| Capability | EOKS interpretation |
|---|---|
| LangSmith-style tracing | execution observations and evaluation traces |
| Langfuse-style tracing/analytics | open observability/evaluation substrate |
| Langroid-style multi-agent execution | execution/orchestration evidence |
| Plano-style routing/governance | infrastructure/control-plane prior art |
| TransformerLab-style experimentation | offline model/evaluation experimentation |

The article's five projects therefore do not constitute a single EOKS layer. They are evidence that modern AI engineering is becoming a stack of observability, evaluation, orchestration, governance and experimentation capabilities.

In particular, tracing systems can capture prompts, model responses, token usage, latency and tool/retrieval steps. That makes them good sources for EOKS traces and downstream evaluation. They do **not**, by themselves, establish that a model's answer is correct or provide a universally meaningful probability of correctness.

## Do not call one number "confidence"

A model's self-reported confidence should be treated as evidence, not ground truth. Likewise, a raw entropy or log-probability statistic should not automatically be interpreted as "the answer is 92% likely to be correct."

A better abstraction is a **reliability signal** assembled from multiple evidence sources.

```text
                         execution trace
                               |
          +--------------------+--------------------+
          |                    |                    |
     model signals        evidence signals     outcome signals
          |                    |                    |
   logprobs / entropy      retrieval agreement   tests
   token uncertainty       source authority      static analysis
   answer instability      contradiction         tool outcomes
   self-assessment         freshness             human review
          |                    |                    |
          +--------------------+--------------------+
                               |
                       reliability evidence
                               |
                    calibrated over outcomes
                               |
                     control-plane decision
```

The useful output may be a vector rather than a scalar:

```text
ReliabilityEvidence {
  model_uncertainty,
  answer_agreement,
  evidence_agreement,
  evidence_quality,
  execution_success,
  historical_task_reliability,
  evaluator_scores,
  provenance,
  calibration_state,
}
```

A scalar score can be derived for a particular policy, but the underlying evidence should remain inspectable.

## Signals from the model computation

### Token probabilities / logprobs

When an inference provider exposes token probabilities, EOKS can derive measures such as token-level uncertainty and entropy.

For example, a generated answer may contain a small number of positions where the model's probability mass is much more diffuse than elsewhere. Those positions can become candidates for verification or additional evidence retrieval.

This is useful because it is closer to the model's actual token-selection computation than a post-hoc self-rating. It is still not a correctness oracle: a model can be confidently wrong.

### Entropy and related information metrics

Entropy can quantify how concentrated or diffuse a token distribution is. Higher uncertainty can be a useful trigger for further work, but its meaning depends on the task, model, tokenization and aggregation method.

EOKS should therefore preserve the raw/derived metric and its calculation metadata rather than pretending that one universal threshold exists.

### Answer instability / semantic agreement

For black-box models or cases where logprobs are unavailable, EOKS can compare multiple independent generations.

```text
sample 1 -> auth.go
sample 2 -> auth.go
sample 3 -> middleware.go
sample 4 -> auth.go
sample 5 -> middleware.go
                    |
                    v
             disagreement
                    |
              verify / branch
```

The important measure is semantic agreement, not merely string equality. Several differently worded answers can express the same conclusion, while identical wording can still be consistently wrong.

Repeated sampling is an evaluation cost, so it should be selectively enabled for workloads where the expected value of additional evidence justifies it.

## External reliability signals

Model-internal uncertainty is only one source of evidence. For software engineering, deterministic and executable evidence can often be stronger:

```text
LLM claim
   |
   +--> type checking
   +--> static analysis
   +--> tests
   +--> repository graph
   +--> runtime/tool result
   +--> authoritative documentation
   +--> human review
```

This connects directly to EOKS's evidence-provider model. A high-uncertainty answer can trigger a deeper analyzer; a high-uncertainty answer that is independently confirmed by strong evidence may be safe to accept; a high-confidence answer contradicted by deterministic evidence should be treated as suspicious.

## Calibration

A raw uncertainty metric is not enough. EOKS should measure whether a reliability signal predicts actual correctness on the workload where it is used.

A simplified calibration loop is:

```text
prediction / reliability signal
              |
              v
       task actually evaluated
              |
              v
        correct / incorrect
              |
              v
       calibration dataset
              |
              v
       reliability mapping
```

Useful evaluation families include calibration error and proper scoring rules such as Brier-style scores. The exact method should depend on whether EOKS is predicting binary correctness, ranking candidates, selecting between actions, or estimating expected utility.

The key principle is:

> **Reliability is workload- and decision-dependent; it must be calibrated against outcomes rather than assumed from a model's raw score.**

## Reliability as a control signal

The most interesting EOKS use is not displaying a dashboard. It is changing execution policy.

```text
                         task
                           |
                    context compilation
                           |
                     model execution
                           |
                reliability evidence
                           |
              +------------+------------+
              |            |            |
             high        medium         low
              |            |            |
             stop       verify        branch
                         /retry       /retrieve
                           |
                      new evidence
                           |
                       evaluate
                           |
                      final outcome
```

The actions should be policy-driven rather than hard-coded around one metric. Possible policies include:

- stop when independent evidence is sufficiently strong;
- run a deterministic check when uncertainty concerns a verifiable invariant;
- retrieve additional context when evidence coverage is poor;
- sample another answer when instability is the main concern;
- route to another model when the workload/model combination has poor historical reliability;
- ask for human review when the expected cost of an error exceeds the value of additional automated work.

This is where observability becomes part of an EOKS control loop.

## Reliability evidence graph

The existing EOKS evaluation model proposed a confidence evidence graph rather than a single confidence number. This research strengthens that idea.

A result should be represented together with the evidence that supports or contradicts it:

```text
                    candidate conclusion
                           |
          +----------------+----------------+
          |                |                |
      model output     retrieved facts   execution result
          |                |                |
     uncertainty       authority/age       tests
     agreement        contradictions       static analysis
          |                |                |
          +----------------+----------------+
                           |
                    reliability state
                           |
                   decision / outcome
```

This allows EOKS to answer not only **"how confident are we?"**, but also **"why should we trust this, what contradicts it, and what evidence would reduce the remaining uncertainty?"**

## Relationship to evaluation

Evaluation and observability should remain distinct but connected:

```text
observability -> captures execution evidence
      |
      v
evaluation -> determines quality/outcome
      |
      v
calibration -> learns how signals relate to outcomes
      |
      v
control -> changes future execution policy
```

This is stronger than a monitoring dashboard because it creates a feedback path from production behavior to future decisions.

## Research questions

1. Which token-level uncertainty measures correlate with correctness for software-engineering tasks?
2. When are multiple sampled answers a better black-box uncertainty signal than model self-assessment?
3. Can semantic answer agreement be computed cheaply enough to use selectively at runtime?
4. Which external evidence sources reduce uncertainty most efficiently for different task classes?
5. Can reliability signals predict the value of another tool call or model invocation?
6. How should uncertainty thresholds change by task type, model, repository and evidence provider?
7. Can reliability be calibrated well enough to drive automated stop/verify/branch decisions?
8. Does a vector of evidence outperform a single scalar confidence score for control decisions?
9. How much additional latency/cost does reliability estimation introduce, and when does it pay for itself?
10. Can the control plane learn which uncertainty signals are predictive for each workload class?

## Proposed first experiment

Use a small software-engineering benchmark with known outcomes.

For every task, record:

- context manifest and evidence providers;
- model/version;
- token-level uncertainty where available;
- model self-assessment if requested;
- answer agreement from a small number of samples on selected tasks;
- retrieval/evidence agreement;
- tests/static-analysis results;
- execution cost and latency;
- final correctness.

Then compare:

1. model self-confidence alone;
2. token uncertainty alone;
3. answer agreement alone;
4. external evidence alone;
5. a combined reliability model.

Evaluate both **prediction quality** and **decision utility**: does using the signal actually reduce incorrect actions, unnecessary retries and unnecessary expensive analysis?

The experiment should begin offline. Only after calibration is demonstrated should reliability signals be allowed to alter the live control loop.

## Architectural conclusion

The strongest EOKS formulation is therefore:

> **Observability supplies the sensors; evaluation supplies outcome labels; calibration turns raw signals into workload-specific reliability evidence; the control plane uses that evidence to choose the next action.**

This makes LangSmith/Langfuse-like systems useful EOKS components without making them the EOKS control plane. It also leaves room for model-level signals, deterministic analyzers, tests, retrieval systems and human feedback to contribute to the same reliability model.
