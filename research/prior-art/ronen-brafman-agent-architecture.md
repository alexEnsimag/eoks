# Ronen Brafman: model-based agents, planning and trustworthy execution

This note captures the parts of Ronen Brafman's recent work (roughly 2024–2026) that materially inform EOKS. It is **prior art, not an EOKS specification**. The goal is to connect model-based planning and autonomous-systems research with the LLM/context-engineering architecture explored elsewhere in this repository.

## Why this matters

Brafman's recent work is not primarily about LLM agent frameworks. Its value for EOKS is more foundational: it treats autonomous behavior as a system with an explicit state/model, action or capability descriptions, planning, execution, observation, uncertainty and replanning. That provides a useful counterweight to architectures that reduce agents to `LLM + memory + tools`.

A useful synthesis is:

```text
                         GOAL / POLICY
                              |
                         PLAN / DECIDE
                              |
                    +---------v---------+
                    |    WORLD MODEL    |
                    | state · history   |
                    | beliefs · time    |
                    | constraints       |
                    +---------+---------+
                              |
                           EXECUTE
                              |
                    capabilities / tools
                              |
                            WORLD
                              |
                         observations
                              |
                           re-plan
```

EOKS can retain this model while treating the LLM as one reasoning component rather than the owner of canonical agent state.

## Key work and concepts

### Regular Decision Processes (2024)

Brafman and De Giacomo's **Regular Decision Processes** extend decision-process models with history-dependent dynamics and rewards whose relevant history can be represented compactly using regular/temporal structures. The useful EOKS lesson is not to adopt RDPs wholesale, but to distinguish **current state from relevant historical state** instead of treating all history as undifferentiated context.

This suggests explicit categories such as:

- current state;
- historical/temporal state;
- observations;
- beliefs and uncertainty;
- policies and constraints.

For EOKS, context can then be understood as a task-specific projection/compilation of these representations rather than the canonical storage of state itself.

### Plug'n Play Task-Level Autonomy (2024)

The Plug'n Play work describes task-level autonomy for robotics using POMDPs and probabilistic programs. Its broader architectural idea is that capabilities can be described at an abstraction suitable for planning and then composed into higher-level behavior.

For EOKS this reinforces the distinction between:

```text
implementation -> capability model -> plan -> execution
```

A tool should not have to expose its entire implementation to an agent. It can expose a machine-readable capability/action description containing preconditions, effects, uncertainty, duration or other execution-relevant properties when these are known.

### Online Planning in MDPs with Stochastic Durative Actions (2025)

This work addresses actions that are **durative, stochastic and potentially concurrent**. That matters because real agent workflows are not always sequences of instantaneous tool calls. A repository scan, CI run, deployment, external API operation or review can be running while other activities proceed or while the system waits for an observation.

The EOKS implication is that an agent runtime may need:

- durable execution state;
- action start/completion/failure events;
- scheduling and temporal constraints;
- monitoring of long-running actions;
- concurrency where dependencies allow it;
- replanning after observations or unexpected outcomes.

This strengthens the control-plane analogy: the runtime is closer to a **reconciliation/scheduling loop** than a simple prompt loop.

### Model-based AI planning and execution platforms for robotics (2025)

This survey provides the clearest architectural connection. Model-based autonomy separates a domain/world model, planning and execution, with execution feeding observations back into the system.

For EOKS the useful pattern is:

```text
knowledge / evidence
        |
        v
     world model
        |
     planner
        |
   executable plan
        |
     executor
        |
   observations
        +-------> re-planning
```

This complements EOKS's existing separation between resources, context compilation, workflows and control. In particular, it argues for making the **model used for planning** explicit even when some reasoning is performed by an LLM.

### Factored planning in partially observable multi-agent domains (2026)

The multi-agent work is relevant because cooperating agents do not necessarily share the same observations or beliefs. A global plan can coexist with agent-local information, and communication/signaling can itself be part of coordinated behavior.

This motivates an EOKS distinction between:

```text
shared knowledge/state
        +
agent-local observations/beliefs
        +
communication history
        +
coordination state
```

"Shared context" should therefore not be assumed to mean "every agent has identical knowledge". EOKS should preserve provenance and scope so that an agent can distinguish what is globally established from what is locally observed or believed.

### Solving Dec-POMDPs as POMDPs Using Imitation Learning (2026)

This work explores deriving distributed behavior from centralized planning and learning local policies/beliefs. The relevant EOKS architectural idea is a separation between **global coordination and local execution/belief**:

```text
             global coordinator
                    |
              global plan/belief
          +---------+---------+
          |         |         |
        agent A   agent B   agent C
          |         |         |
       local      local      local
       belief     belief     belief
```

This is useful prior art for future multi-agent EOKS designs, but it does not imply that EOKS should default to centralized planning or agent swarms.

### TRUST-CUA (2026 workshop)

TRUST-CUA is directly relevant to trustworthy computer-using agents. Its focus on predictability, steerability, auditability, recovery, policy compliance and human oversight complements EOKS's control/evaluation model.

The important EOKS lesson is that task success is not enough. A trustworthy agent workload should expose evidence about:

- whether actions followed policy;
- what happened and why;
- whether the system can recover from failure;
- how much human oversight was required;
- whether the execution was predictable;
- whether risky actions were reversible or appropriately gated.

These concerns belong around the execution/control loop rather than inside the knowledge layer.

## Consolidated EOKS implications

The research supports four additions/refinements to EOKS.

### 1. Explicit agent/world state

EOKS should distinguish the canonical state/model used by a workload from the context compiled for a particular reasoning step.

```text
representations / evidence
          |
          v
     workload state
       /   |   \
 history beliefs policies
       \   |   /
          v
   context compilation
          |
          v
       reasoning
```

The LLM can propose a state transition or action, but canonical state should be maintained and validated by the surrounding system where practical.

### 2. Capability/action models

Resources used for execution can expose action-level semantics when available: inputs, preconditions, effects, duration, side effects, uncertainty, required permissions and validation requirements. This is richer than a tool schema but does not require a universal formal planning language.

EOKS should adopt the **capability-model concept**, not mandate that every tool become a formal planner operator.

### 3. Temporal and observation-driven execution

The control loop should support long-running and asynchronous activities:

```text
plan
  -> start action
  -> observe progress/outcome
  -> update state
  -> verify
  -> continue / retry / branch / re-plan / escalate
```

A plan is therefore an executable hypothesis, not a guarantee that all subsequent actions remain valid.

### 4. Scoped knowledge and multi-agent beliefs

For multi-agent workloads, knowledge should carry scope/provenance such as `global`, `agent-local`, `observed`, `inferred`, `communicated` and `policy-mandated` where useful. This prevents context assembly from accidentally turning an agent's uncertain local belief into shared canonical knowledge.

## Relationship to existing EOKS concepts

| Brafman-oriented concept | EOKS counterpart | Boundary |
| --- | --- | --- |
| world/domain model | knowledge representations + workload state | EOKS can use multiple representations rather than one world model |
| history-dependent state | historical/episodic/temporal state | do not put all history into prompt context |
| action/capability model | resource + capability metadata | formal planning semantics are optional |
| planner | workflow/control/reasoning strategy | EOKS does not require a single planning algorithm |
| execution monitor | observability + verification + control loop | traces are evidence, not canonical state |
| belief state | scoped uncertain knowledge | distinguish belief from fact |
| signaling/communication | agent coordination and provenance | communication can be an explicit workflow action |
| durative actions | long-running workflow activities | requires durable state and observation |
| recovery/replanning | reconciliation | advance based on evidence, not final messages |
| trustworthy CUA | policy/evaluation/human gates | trust is workload-specific and must be evidenced |

## What EOKS should *not* take from this work

- Do not turn EOKS into a classical planning framework.
- Do not require POMDP/MDP formalisms for ordinary software-engineering tasks.
- Do not assume every workload needs a global world model.
- Do not assume multi-agent execution is better than one agent with focused contexts.
- Do not replace context engineering with planning; context remains the task-specific evidence projection.
- Do not treat formal action models as a prerequisite for useful agent tooling; adopt them incrementally where they improve scheduling, safety, verification or composability.

The useful synthesis is **model-based control around probabilistic reasoning**, not a replacement of LLM agents with symbolic planners.

## References

- Brafman, R. I. & De Giacomo, G. — *Regular decision processes*, Artificial Intelligence, 2024.
- Wertheim, O., Suissa, D. R. & Brafman, R. I. — *Plug'n Play Task-Level Autonomy for Robotics Using POMDPs and Probabilistic Programs*, IEEE Robotics and Automation Letters, 2024.
- Berman, T., Brafman, R. I. & Karpas, E. — *Online Planning in MDPs with Stochastic Durative Actions*, IJCAI 2025.
- Wertheim, O. & Brafman, R. I. — *Model-based AI planning and execution platforms for robotics*, AI Magazine, 2025.
- Shekhar, S., Brafman, R. I. & Shani, G. — *Factored planning in partially observable and deterministic multi-agent domains*, Artificial Intelligence, 2026.
- Keller, R. & Brafman, R. I. — *Solving Dec-POMDPs as POMDPs Using Imitation Learning*, PRIMA 2025 proceedings, published 2026.
- Li, T. J. J., Shlomov, S., Deng, X., Brafman, R., Yaeli, A. & Wang, Z. — *TRUST-CUA: Trustworthy Computer-Using Generalist Agents for Intelligent User Interfaces*, IUI 2026 Companion.

See Brafman's BGU research profile for the publication record and current research projects. The active 2025–2028 project on task planning with concurrent durative actions and stochastic effects is particularly consistent with the temporal-execution direction above.
