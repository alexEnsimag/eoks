# LLM observability and reliability signals

This note collects the **observability evidence and representations** that emerged from recent AI-engineering research. It deliberately keeps several representations: they answer different questions and can be useful at different evaluation/control boundaries.

The canonical EOKS distinction remains:

```text
observability
  -> record what happened

reliability / uncertainty estimation
  -> estimate how much the result should be trusted

control
  -> use evidence and reliability signals to decide what happens next
```

The purpose of this note is therefore not to define another EOKS layer. It documents useful telemetry, evidence structures and experiments that can feed the existing evaluation and control model.

## 1. Evidence taxonomy

The recent resources suggest four complementary views of execution evidence:

| View | Main question | Useful representation |
|---|---|---|
| **Operational** | What happened during execution? | traces, spans, events, metrics |
| **Reliability** | What evidence supports or contradicts the result? | decomposable evidence vector / evidence graph |
| **Causal / attribution** | Which intervention or decision contributed to the outcome? | lineage, provenance, intervention → outcome links |
| **Economic** | What did the run consume and what value did it produce? | run economics and marginal value |

These are not competing ontologies. A single run can have all four views.

## 2. Observability as execution evidence

Tools such as LangSmith and Langfuse primarily provide tracing, evaluation, analytics and monitoring around model/agent execution. OpenTelemetry (OTel) and OTel-compatible GenAI conventions provide a useful interoperability substrate for representing similar execution evidence across implementations.

A useful EOKS mapping is:

| EOKS activity | Telemetry representation |
|---|---|
| run / workflow | trace or workflow span |
| agent invocation | agent/workflow span |
| model inference | GenAI inference span |
| tool execution | tool span |
| retrieval / memory | retrieval or memory span/event |
| evaluation | evaluation result/event |
| outcome / artifact | workflow state and linked artifact |
| intervention | explicit configuration/version + lineage |

The important boundary is that telemetry records observations; it does not by itself determine correctness, reliability or policy.

### Spans versus events

Use spans for operations with duration and parent/child relationships. Use events for point-in-time facts or structured annotations. This distinction matters when reconstructing an execution trace and when avoiding accidental double-counting.

OTel should be treated as an **interoperability substrate**, not as the EOKS semantic model. EOKS can consume richer or poorer telemetry depending on the evidence required by the evaluation task.

## 3. Reliability evidence: keep multiple representations

A reliability assessment should remain inspectable rather than being reduced immediately to one confidence number.

A useful vector representation is:

```text
ReliabilityEvidence
├── model uncertainty
├── answer agreement
├── evidence agreement
├── evidence quality
├── execution validation
├── historical task reliability
├── evaluator results
├── provenance
└── calibration state
```

The same evidence can also be represented as a graph:

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

The vector is convenient for measurement and calibration. The graph is useful for provenance, contradiction analysis and deciding what evidence is missing. A policy may derive a scalar, ranking or expected-utility estimate from either representation, but the underlying evidence should remain available.

## 4. Model-level uncertainty signals

### Token probabilities / logprobs

When an inference provider exposes token probabilities, EOKS can preserve the raw values and derive measures such as token-level uncertainty, length-normalized log probability, top-token margins and predictive entropy.

These measures are closer to the model's token-selection computation than a post-hoc self-rating, but they are not correctness oracles. Their interpretation depends on the task, model, tokenization and aggregation method.

### Entropy and semantic uncertainty

Entropy measures how concentrated or diffuse a token distribution is. For black-box models or cases without logprobs, multiple sampled generations can provide a different signal: semantic agreement or instability.

```text
sample 1 -> conclusion A
sample 2 -> conclusion A
sample 3 -> conclusion B
sample 4 -> conclusion A
                    |
                    v
             disagreement
                    |
             possible verify
```

Semantic agreement is preferable to literal string equality when the question is whether conclusions agree. Repeated sampling is itself an evaluation cost, so it should be selectively enabled when its expected value exceeds that cost.

## 5. External evidence and verification

Model-internal uncertainty is only one source of reliability evidence. For software engineering, executable evidence can often be stronger:

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

This connects observability to EOKS's evidence-provider model. A reliability policy can choose evidence according to the uncertainty or failure mode it is trying to resolve.

Examples:

- uncertainty about a verifiable invariant → deterministic check;
- missing repository knowledge → targeted retrieval;
- disagreement between plausible conclusions → independent sample or evidence provider;
- historically unreliable model/task combination → model switch or escalation;
- high-value result without sufficient provenance → additional evidence or review.

## 6. Provenance and causal attribution

A trace tells us **what happened**, but it does not automatically tell us **why a decision happened** or which intervention caused an outcome.

This distinction was made especially clear by telemetry research such as TelemetrySuffBench: compact telemetry can retain strong failure-detection performance while losing the information needed for origin-step or causal attribution. In particular, decision content and provenance links can matter much more for attribution than for simply detecting that something failed.

The EOKS implication is not to demand maximal telemetry everywhere. Instead, evaluate the minimum evidence needed for the claim being made:

```text
claim
  |
  +--> detect failure?
  |       -> operational telemetry may suffice
  |
  +--> locate failing step?
  |       -> richer lineage needed
  |
  +--> attribute outcome to intervention?
  |       -> intervention + provenance + outcome linkage
  |
  +--> learn policy from the result?
          -> retain enough context to distinguish confounders
```

This creates an explicit **observability sufficiency** question: what telemetry is sufficient for a particular evaluation or control decision?

## 7. Run economics: tokens are one dimension

Token usage is important, but it should not become a new EOKS subsystem called "tokenomics" or "harness economics".

A broader representation is:

```text
Run economics
├── model tokens
├── context occupancy / composition
├── tool calls
├── retrieval / memory work
├── evaluation / verification
├── retries / replanning
├── latency
├── infrastructure cost
└── human intervention
          |
          v
     outcome value
          |
          v
   marginal run value
```

This matters because reducing model tokens can increase verification, retrieval or retry costs. Likewise, a larger context can be worthwhile if it prevents expensive exploration or errors.

### Context occupancy is not token usage

Per-call input tokens measure what a provider processed for that call. They do not necessarily describe the semantic composition or occupancy of the context available to the agent across a workflow.

EOKS should therefore distinguish:

```text
context composition / occupancy
          !=
provider-reported per-call token usage
```

Both can be useful evaluation dimensions, and neither is a universal efficiency score.

### Attribution and nested traces

Aggregating cost from nested spans can double-count the same model or tool work if parent spans already include child usage. Cost accounting therefore needs explicit ownership rules and should preserve raw observations separately from derived aggregates.

## 8. Harness evolution as an evaluation problem

Agentic Harness Engineering (AHE) provides a useful representation of iterative configuration improvement. It makes editable harness components explicit, distills execution experience into layered evidence, and associates an edit with a prediction that can later be checked against task-level outcomes. Its reported gains therefore provide evidence for an existing EOKS pattern:

```text
configuration intervention
          |
          v
       execution
          |
          v
    observed evidence
          |
          v
    task-level outcome
          |
          v
       evaluation
          |
          v
  retain / revert / evolve
```

The important contribution for EOKS is the **falsifiable intervention representation**, not a requirement for a new Harness object. AHE also illustrates why component-level, trajectory-level and decision-level observations can coexist without being collapsed into one trace field. citeturn0academia0

The broader harness literature and source-code studies reinforce the same evaluation boundary: behavior emerges from the model together with its runtime, tools, context management and execution policy. citeturn0academia2turn0academia3

LoopsBench extends this concern to sustained long-horizon execution, where dependency structure and regression obligations make trajectory/workflow evaluation important. citeturn0academia1

## 9. Evaluation and control

The evidence views above feed the existing EOKS evaluation model:

```text
execution
    |
    +--> operational observations
    +--> reliability evidence
    +--> provenance / lineage
    +--> run economics
    |
    v
 evaluation + calibration
    |
    v
 policy / control
    |
    +--> stop
    +--> verify
    +--> retrieve
    +--> retry / replan
    +--> change model
    +--> escalate
```

The policy should use the representation appropriate to the decision. There is no requirement that every decision use the same scalar reliability score or the same telemetry resolution.

## Appendix A — Evidence matrix for the resources discussed

| Resource / idea | Primary contribution | EOKS placement | What it adds |
|---|---|---|---|
| Agentic Harness Engineering | component / experience / decision observability; falsifiable edits | evaluation + execution evidence | explicit intervention/evidence/outcome linkage |
| LoopsBench | long-horizon loop evaluation | workflow evaluation | dependency-aware trajectory/regression evidence |
| Harness anatomy / source-code study | runtime as a platform around the model | execution resources + evaluation | richer component boundaries and empirical patterns |
| Code as Agent Harness | code as operational substrate | execution / feedback | alternative harness representation |
| OpenTelemetry GenAI | common telemetry representation | observability substrate | interoperable traces, events and attributes |
| TelemetrySuffBench | telemetry sufficiency and attribution limits | observability → evaluation | separates failure detection from causal attribution |
| Tokenomics / harness economics | resource consumption across an agent run | system evaluation | broadens cost from tokens to run economics |

This table is a **mapping aid**, not a taxonomy of EOKS subsystems.

## Appendix B — Simulation design: reliability evidence and control

A small simulation can test the EOKS control hypothesis without committing the architecture to a particular reliability formula.

Generate synthetic tasks with a known outcome and several imperfect evidence sources:

```text
                 true task outcome
                        |
        +---------------+---------------+
        |               |               |
   model signal    evidence signal   validator
        |               |               |
     noisy           noisy/strong     deterministic
        |               |               |
        +---------------+---------------+
                        |
              reliability representation
                / vector / graph
                        |
                 policy decision
                        |
          +-------------+-------------+
          |             |             |
         stop         verify        retrieve
          |             |             |
          +-------------+-------------+
                        |
                  final outcome
```

Compare policies using the same underlying tasks:

1. a single scalar confidence threshold;
2. a decomposable evidence policy;
3. a cost-aware policy using expected value of another action;
4. an oracle policy using the true outcome, only as an upper bound.

Measure:

- task correctness;
- false acceptance and unnecessary verification;
- additional tokens / tool calls;
- latency and cost;
- calibration;
- robustness when one evidence source becomes unavailable or misleading.

The point of the simulation is not to discover a universal formula. It is to test whether preserving heterogeneous evidence improves decisions under realistic cost and failure assumptions.

## Appendix C — Open questions

- What telemetry is sufficient for each class of evaluation claim?
- Which provenance links are necessary for intervention attribution?
- When does a reliability vector outperform a scalar for actual control decisions?
- Which evidence providers resolve different uncertainty modes most efficiently?
- How should run economics account for context, verification and retries rather than tokens alone?
- How should trajectory-level reliability be represented when individual steps are correlated?
- Can learned control policies remain calibrated when the model, harness configuration or workload distribution changes?
- What evidence should be retained long-term for regression analysis without retaining every raw token?
