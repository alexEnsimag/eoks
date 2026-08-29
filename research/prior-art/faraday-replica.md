# Faraday / Replica: learned scientific judgment over coding agents

**Source:** Damon Falck et al., [*Training AI Scientists to Replicate Research*](https://arxiv.org/abs/2608.13331), August 2026.

## Why this matters to EOKS

This work is relevant to EOKS primarily as **evaluation/control-loop and reasoning-strategy prior art**, not as a new knowledge-store or agent-runtime category.

The central architectural observation is that a relatively small outer model can be trained to provide **scientific judgment** while delegating implementation work to a much larger coding agent. Faraday is a 27B-parameter agent that uses coding agents as tools; the paper reports stronger replication performance than the evaluated Claude Opus 4.8 and GPT-5.5 baselines on its benchmark. The important EOKS question is therefore not whether the 27B model is intrinsically “smarter”, but whether a learned decision layer can improve the use of stronger execution tools.

## Replica as an executable knowledge/evaluation environment

Replica contains 310 figure-replication tasks derived from 100 ML and AI-for-science papers. A task hides a target result and asks the agent to reconstruct it under constrained resources. The task is intentionally underspecified: papers omit many implementation details, and the agent must decide which details and experiments matter for the scientific claim.

The resulting structure is close to an executable knowledge loop:

```text
paper / claim
     |
     v
replication task + constraints
     |
     v
agent decisions / experiments
     |
     v
artifacts + execution trace
     |
     v
rubric-based evaluation
     |
     v
reward / policy evidence
     |
     +--> improve future decisions
```

This is a useful concrete example of the EOKS principle that **knowledge becomes more operational when it is connected to tasks, evidence, evaluation and outcomes**. A paper is not merely retrieved context: its claims can define an environment in which an agent must produce evidence.

## Scientific judgment versus execution capability

The architecture suggests a useful separation:

```text
                    workload / research goal
                              |
                              v
                  judgment / planning layer
               what matters? what to try next?
                              |
                              v
                  coding / execution agent
                    implementation + tools
                              |
                              v
                     experiments / artifacts
                              |
                              v
                         evaluation
                              |
                              +----> next decision
```

For EOKS this maps naturally to **reasoning strategies and workflow/control resources** rather than to the context layer itself. The outer layer chooses actions; execution resources perform them; evaluation produces evidence that feeds the next control decision.

This also reinforces the practical EOKS preference for a small topology of logical roles rather than an uncontrolled agent swarm: a conductor/judgment role can coordinate an executor and an independent evaluator/reviewer when there is a concrete benefit from separating responsibilities.

## Resource constraints are part of the task semantics

Replica deliberately constrains time and compute. The agent cannot simply reproduce every experimental detail at full scale. It must determine what can be reduced without destroying the mechanism underlying the claim.

This is important for EOKS context compilation and control: **budget is not merely an operational metric; it can change which action is rational.** A control plane should therefore be able to expose workload budgets and let reasoning/workflow policies trade evidence quality against time, tokens and compute.

This connects to the existing EOKS concepts of marginal context value and expected value of another step. The relevant optimization target is not “maximum context” or “maximum exploration”, but sufficient evidence for the workload at acceptable cost.

## Evaluation and the judge boundary

The paper introduces an auto-generated rubric-based judge to provide a scalable reward signal for replication quality and reports human-validation experiments. This is particularly relevant to EOKS because it demonstrates a path for evaluating **underspecified, long-horizon work** where a simple exact-match oracle does not exist.

However, EOKS should preserve an important caveat: the same rubric/judging machinery is central to both training and headline evaluation. Independent review notes have therefore raised a potential circularity risk, and the external human validation should be treated as supporting evidence rather than proof that the judge is an objective oracle.

The EOKS boundary should be:

```text
agent output
    |
    +--> deterministic validators where possible
    +--> independent evidence
    +--> calibrated judge / rubric
    +--> human review for consequential cases
             |
             v
       evaluation evidence
             |
             v
           policy
```

A learned or LLM-based evaluator is therefore an **evidence provider**, not automatically the source of truth.

## Turn-level credit assignment

The training work also highlights a control/evaluation problem that is relevant beyond scientific agents: long-horizon trajectories contain many actions, but the final outcome does not make every action equally responsible for success or failure. The paper uses turn-level credit assignment in its reinforcement-learning setup.

For EOKS, this suggests a broader research direction: evaluation should retain enough trace structure to identify which decisions materially contributed to an outcome. This can support better policy learning, context-selection updates and workflow optimization instead of assigning all credit/blame to the final response.

This should remain a research hypothesis, not a new EOKS primitive until real traces demonstrate that it is useful.

## EOKS mapping

| Replica / Faraday concept | EOKS interpretation |
|---|---|
| Paper / scientific claim | durable knowledge + authoritative source |
| Replication task | workload / task contract |
| Time and compute budget | workload constraints / policy inputs |
| Faraday | reasoning/judgment strategy or conductor-like role |
| Coding agent | execution resource |
| Experiments / workspace | execution artifacts and evidence |
| Rubric judge | evaluation/evidence provider |
| Turn-level credit | trajectory-level evaluation signal |
| Replication score | task outcome/evaluation evidence |
| Learned behavior | candidate policy/reasoning capability, subject to validation |

## What not to infer

- Replica does **not** imply that EOKS needs a scientific-research agent.
- Faraday does **not** replace the context compiler, knowledge providers or deterministic analyzers.
- A rubric judge should not replace deterministic evidence where deterministic evidence is available.
- The reported benchmark result should not be generalized into a claim that a 27B model universally outperforms frontier models; it is a result on this particular replication task and evaluation setup.
- Turn-level credit assignment is a training technique here, not evidence that EOKS needs a new persistent resource type.

## Proposed EOKS experiments

1. **Judgment/execution split:** compare one model performing both planning and coding with a smaller planning model delegating to a stronger coding agent.
2. **Evidence-aware planning:** give the planner explicit provenance, freshness, deterministic-check and evaluation signals and measure whether it makes better next-step decisions.
3. **Budget-aware context:** test whether a planner can choose between additional context, repository exploration, deterministic analysis and execution under a fixed token/time budget.
4. **Trajectory attribution:** retain decision-level traces and test whether attributing outcome value to important turns improves workflow or policy learning.
5. **Evaluator independence:** compare self/LLM judging with deterministic checks and independent evaluators; measure calibration against human-reviewed outcomes.

These experiments belong primarily in EOKS **evaluation, control loop and reasoning-strategy research**, with a secondary connection to executable knowledge.