# Deterministic execution and minimizing unnecessary uncertainty

EOKS should not treat an LLM as the default implementation mechanism for every workflow step. A central design principle is to **minimize unnecessary uncertainty**: use probabilistic reasoning where ambiguity or open-ended judgment requires it, then transfer sufficiently specified work to the most deterministic mechanism that can satisfy the requirement.

This complements tool selection, planning, execution, validation and graduated autonomy; it is not a new runtime layer.

## Principle

> **Use probabilistic reasoning to resolve uncertainty. Once the required behavior is sufficiently specified, prefer deterministic execution and objective verification.**

Deterministic does not mean universally superior. Novel, ambiguous, poorly specified or highly state-dependent work can require an LLM or another reasoning mechanism. The objective is to avoid paying for probabilistic reasoning when the desired behavior is already determined.

```text
natural-language goal
        |
        v
   specification / plan
        |
   +----+------------------+
   |                       |
   v                       v
LLM reasoning        deterministic execution
when uncertainty     when behavior is specified
remains                    |
   |                       |
   +-----------+-----------+
               v
      objective verification
               |
             evidence
               |
      continue / replan /
         escalate
```

## Execution modality

A workflow step may be implemented by different modalities:

- deterministic computation or algorithm;
- deterministic tool, query, compiler, validator or script;
- retrieval / lookup;
- specialized model;
- general LLM reasoning;
- multi-agent reasoning;
- human judgment.

The conductor should select the smallest modality and topology that satisfy the step's requirements. A role name does not imply an agent, and an agent does not imply that every operation inside it should be model-mediated.

This is a per-step property, not a permanent classification of a provider. One workflow can legitimately use an LLM to produce a structured specification, a deterministic component to execute it, and an LLM again to interpret an unexpected result.

## LLM as specification compiler

A useful pattern is:

```text
natural-language goal
        |
       LLM
        |
validated structured specification / procedure
        |
 deterministic runtime
        |
 objective verification
```

The LLM supplies semantic interpretation or synthesis; the runtime supplies repeatable execution. Preconditions, effects, permissions and validation requirements can make the boundary explicit.

This generalizes the existing EOKS separation between state, context, reasoning and execution: context should support reasoning, but reasoning should not remain in the execution path when no further uncertainty needs to be resolved.

## Compile once, execute many

Repeated workloads create an additional opportunity. If an agent repeatedly discovers essentially the same procedure, EOKS should be able to consider compiling the procedure into a reusable deterministic artifact.

```text
first execution:
  goal -> reasoning -> validated procedure

subsequent executions:
  inputs/state -> deterministic procedure -> evidence
```

This can amortize model cost and improve latency, reproducibility, auditability and operational predictability. The procedure must not be assumed valid forever: changes in inputs, repository state, dependencies, policies or preconditions can invalidate it.

The safe pattern is therefore **compile, validate, monitor, invalidate/recompile when required**, rather than blindly caching agent behavior.

## Community evidence

This direction is also emerging independently in practitioner communities and agent frameworks. These reports are not controlled evidence of general superiority, but they are useful evidence that the same engineering problem is being encountered repeatedly.

### Deterministic subflows are becoming an explicit agent-framework discussion

LangGraph community discussions from May and June 2026 explicitly describe a split between steps that genuinely require model reasoning and predictable tool/data-transformation steps that should become deterministic, replayable subflows. The motivations listed include avoiding unnecessary model calls, improving replay/debugging, reducing variance and cost, and creating clearer failure boundaries. A follow-up discussion asks when repeated model-routed paths should be promoted to deterministic subflows after they stabilize.

See:
- https://github.com/langchain-ai/langgraph/issues/7855
- https://github.com/langchain-ai/langgraph/issues/8032

This is unusually close to the EOKS hypothesis and supports making **stabilization → compilation/consolidation** an explicit research direction rather than only a runtime optimization.

### Trustworthy runtimes are moving deterministic boundaries outward

A July 2026 Qwen Code proposal argues for deterministic tool-execution boundaries in which the language model remains outside the trust boundary while the runtime deterministically constrains, authorizes, observes and evaluates model-produced actions. This reinforces an important EOKS distinction: the model can propose, while a deterministic runtime owns enforceable execution policy.

See:
- https://github.com/QwenLM/qwen-code/issues/8102

### Replayability is a separate reason to reduce model-mediated execution

An AutoGen discussion from May 2026 highlights how difficult deterministic replay becomes when external APIs, MCP state, filesystems and checkpoints drift. The discussion points out that observability alone is insufficient for replay: the inputs that produced an execution also need to be preserved.

For EOKS this means deterministic execution is not only about cost. It can also reduce the amount of state and stochastic behavior that must be reconstructed to reproduce or audit a run.

See:
- https://github.com/microsoft/autogen/discussions/7695

### Practitioners are independently converging on workflow + bounded LLM decisions

Recent practitioner discussions describe architectures where deterministic code owns the workflow while the LLM is restricted to decisions inside that workflow. Other discussions use a simpler distinction: if the flowchart can be specified before execution, it is generally better treated as a workflow; if the next step depends on new observations and cannot be known ahead of time, an agent becomes more justified.

These are anecdotal reports, but they align closely with the EOKS distinction between deterministic execution and probabilistic reasoning.

Examples:
- https://www.reddit.com/r/AI_Agents/comments/1w035sb/i_stopped_letting_the_llm_control_the_workflow_my/
- https://www.reddit.com/r/AI_Agents/comments/1vy5dkh/are_ai_agents_actually_better_than_deterministic/

### Deterministic gates are already being used around coding agents

A recent practitioner discussion describes highly automated coding workflows that rely on deterministic tests, linting, typechecking and other gates after specification/planning. The important pattern is not that these gates replace reasoning; they prevent the agent from being the sole judge of whether its work succeeded.

See:
- https://www.reddit.com/r/LLMDevs/comments/49t0w1l6/what_ai_coding_workflow_did_you_eventually_settle/

### New research makes compilation from traces especially relevant

TraceCompiler (August 2026) mines repeated LLM-agent traces and compiles recurring procedures into mostly deterministic workflows. Importantly, it uses evidence about producer/consumer dependencies and refuses to compile an intent when an irreversible side effect remains under-determined. This suggests a concrete EOKS mechanism: **repetition alone should not trigger compilation; repetition plus evidence of stable dependencies and safe execution should.**

See: https://arxiv.org/abs/2608.02680

A separate August 2026 production study on deterministic executability gating reports that deterministic predicates can prune non-executable candidates before LLM skill selection. The study reports large reductions in skill-description context and demonstrates that deterministic state checks can prevent impossible candidates from influencing model selection at all.

See: https://arxiv.org/abs/2608.01050

## Escalation back to reasoning

Deterministic execution should be able to return control to a reasoning component when it encounters uncertainty that the current procedure cannot resolve:

```text
procedure
   |
   +--> preconditions satisfied -> execute -> verify -> done
   |
   `--> unexpected state / ambiguity / failed coverage
                    |
                    v
              diagnose / replan
                    |
                    v
              updated procedure
```

This makes determinism a control-flow decision rather than a rigid architecture. The system should escalate because evidence says the current mechanism is insufficient, not because an agent framework happens to provide another model call.

## Relationship to evidence and assurance

Deterministic execution and deterministic evidence are related but distinct.

- A deterministic tool can execute a known operation reliably.
- A deterministic check can establish a property with known semantics.
- A deterministic execution result is not automatically proof that the desired outcome was achieved.
- Conversely, a probabilistic component can be useful for interpreting objective evidence without being authoritative about the underlying property.

EOKS should therefore preserve provenance and distinguish:

**model confidence → evidence strength → outcome quality**.

The latter two should determine advancement and autonomy according to workload risk and policy.

## Relationship to tool selection

The tool-selection pipeline should consider execution modality alongside capability fit, reliability, cost, latency and consequence of error.

For example:

```yaml
requirement:
  kind: repository_transformation
  scope: repository
  repeatability: high
  execution_modality: deterministic_preferred
```

A natural-language request may first require an LLM to infer the intended transformation. Once the transformation is explicit, an AST transformation, compiler, formatter or script may be preferable to having an LLM edit every occurrence.

The same principle applies to verification: if a property can be established by a type checker, static analyzer, test, schema validator or other objective mechanism, it should not be replaced by an LLM assertion that the property holds.

## What not to optimize

EOKS should **not** maximize the percentage of deterministic execution. Determinism is a means, not an outcome.

A deterministic workflow that solves the wrong problem is worse than a more expensive reasoning workflow that solves the right one. Likewise, forcing novel tasks into deterministic procedures can increase engineering effort and reduce adaptability.

The relevant objective is:

> **maximize useful engineering work per unit of probabilistic reasoning without degrading correctness, safety, coverage or adaptability.**

## Measurement

Experiments comparing agentic and deterministic alternatives should measure at least:

- task success and correctness;
- critical regression / failure rate;
- model calls per successful outcome;
- tokens or model compute per outcome;
- fraction of steps executed deterministically;
- reasoning cost amortized across repeated executions;
- end-to-end latency;
- retry, repair and escalation rates;
- variance across repeated runs;
- human intervention and review effort;
- reproducibility and provenance.

A higher deterministic-execution ratio is useful only when it improves the engineering outcome or provides equivalent assurance at lower cost/latency.

## Prior art and research direction

Several recent research directions support this separation:

- **LLM + classical planning** demonstrates using an LLM for language-to-formal-problem translation and a classical planner for the actual planning problem, rather than expecting the LLM to perform every planning operation.
- **Compiled AI** explores generating a validated executable artifact and executing it without repeated model calls, directly studying the cost and predictability advantages of compilation for suitable workflows.
- **PlanCompiler** explores validating structured LLM-generated plans against typed capabilities and compiling them into deterministic execution.
- **Policy-as-code / deterministic guard approaches** similarly move enforceable constraints out of probabilistic instruction following and into executable checks.

These are evidence for a research hypothesis, not proof that every EOKS workload should be compiled or deterministic. EOKS should test the boundary empirically across task types, models and repositories.

## Experiments for EOKS

Useful experiments include:

1. Compare an LLM-mediated implementation with a deterministic implementation for the same repeated workload.
2. Measure whether an LLM can reliably produce a structured specification that a deterministic runtime can validate.
3. Measure amortization when a generated procedure is reused across many executions.
4. Inject repository/state changes and measure procedure invalidation and recovery.
5. Compare fixed agent loops against escalation policies that invoke reasoning only after deterministic mechanisms fail.
6. Mine repeated successful traces and test whether dependency evidence can safely identify procedures worth compiling.
7. Measure whether deterministic executability gates improve model selection and reduce unnecessary context/model work.
8. Measure the effect on correctness, trust, human review effort, latency and cost rather than optimizing any single metric.

The key empirical question is not **“Can an LLM do this?”** It is **“Does this step actually require probabilistic reasoning?”**
