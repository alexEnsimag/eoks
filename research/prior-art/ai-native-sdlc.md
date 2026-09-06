# AI-native SDLC and EOKS

## Why this matters to EOKS

AI-native SDLC is becoming a named software-engineering pattern rather than simply a description of using AI to write code. Anthropic's 2026 AI-Native SDLC Playbook defines it as a loop rather than a linear lifecycle, with automated handoffs, continuous evaluation, governance at execution time, and production signals feeding new intent back into the lifecycle. Each stage commits a durable artifact such as `intent.md`, `spec.md`, `plan.md`, implementation/test records, review findings or incident records; the next stage consumes that artifact. [Anthropic, *The AI-Native SDLC Playbook*](https://academy.claude.com/courses/ai-native-sdlc-playbook/introduction)

GitHub's Spec Kit and other recent SDD systems describe a related but narrower pattern: **intent/specification → plan → tasks → implementation**, with each phase producing an artifact that supplies structured context to the next phase. [GitHub Spec Kit](https://github.github.com/spec-kit/) [Spec-Driven Development](https://github.com/github/spec-kit/blob/main/docs/concepts/sdd.md)

These patterns are useful prior art for EOKS because they provide a concrete software-engineering workload over which the EOKS control-loop model can operate.

## The EOKS connection

> **AI-native SDLC is a concrete application of EOKS's workload control-loop model to software delivery: versioned intent, specifications, plans, implementation artifacts and evaluation evidence provide durable representations of workload state and handoffs, while reconciliation coordinates progress between desired and actual software state.**

The mapping is intentionally architectural rather than a claim that EOKS should implement a particular SDLC framework:

```text
AI-native SDLC                    EOKS
----------------                  -------------------------
intent / requirements       ->    intent + desired state/policy
spec / contract             ->    governed representation of
                                  desired behavior/constraints
plan                        ->    action proposal / execution state
code + tests + PR           ->    run artifacts + evidence
review / eval               ->    evaluation + evidence
production observations     ->    actual workload state
new intent                   ->    reconciliation / next workload
```

The SDLC stages themselves are therefore a **workflow over the control loop**, not additional EOKS primitives.

## AIDLC as a concrete adaptive-workflow case

AWS's current AI-DLC implementation is useful as a more developed case study of the same architectural pressures. The current implementation uses a deterministic engine for routing and state transitions while a conductor executes the selected work; its adaptive composer can propose an `EXECUTE`/`SKIP` plan from the task, workspace scan or running workflow, and it estimates several uncertainty/risk dimensions to justify the minimum sufficient workflow. This is concrete evidence for the EOKS hypothesis that workflow selection can itself be adaptive, rather than evidence that EOKS needs an AIDLC-specific workflow taxonomy. [AI-DLC Workflows](https://github.com/awslabs/aidlc-workflows/) [Adaptive composer](https://github.com/awslabs/aidlc-workflows/blob/main/docs/guide/12-cli-commands.md)

The current implementation also makes a useful state/history distinction: the per-intent state file is the durable cursor for current progress, while an append-only audit trail records decisions and events and can be used to reconstruct state. This reinforces EOKS's separation of current workload state, historical evidence and longer-lived knowledge; it does not imply that `aidlc-state.md` or an AIDLC audit format should become EOKS primitives. [AIDLC state and audit](https://github.com/awslabs/aidlc-workflows/blob/main/docs/guide/10-state-and-audit.md)

A particularly useful assurance lesson is the explicit deterministic boundary between the conductor and the engine: the agent's narrative about a transition should not by itself be treated as authoritative workflow state. AIDLC's implementation experience includes cases where execution and persisted state can diverge, making the broader EOKS requirement concrete: authoritative state transitions should be represented and verified outside an agent's self-report. This is prior-art evidence for deterministic state authority, not a reason to add a new EOKS primitive.

AIDLC also exposes a candidate heuristic for adaptive process selection by estimating implementation uncertainty across intent ambiguity, structural uncertainty, verification uncertainty, risk and unresolved assumptions. These estimates are useful as an **evaluation hypothesis**: can workload-aware uncertainty signals select a minimum sufficient process without sacrificing assurance? The heuristic itself should not be adopted as an EOKS metric without comparative evaluation.

The practitioner account *From OpenSpec to AIDLC* provides independent evidence for the motivation behind these mechanisms: fixed specification workflows can become burdensome, decisions and intent can be lost, and teams may bypass process when its overhead exceeds its perceived value. It therefore strengthens the motivation for adaptive workflow selection, but does not by itself establish that AIDLC improves software quality. [From OpenSpec to AIDLC](https://www.dataleadsfuture.com/from-openspec-to-aidlc-how-i-improved-my-teams-ai-code-quality/)

## Artifacts are durable control-plane representations

The important architectural lesson is not that EOKS needs an `Artifact` primitive. An artifact is a durable representation that can carry different semantics depending on the workload:

- intent or desired state;
- rationale or decision;
- specification or contract;
- execution state or plan;
- evidence and evaluation;
- outcome;
- audit/provenance information.

An artifact can therefore be consumed by the next reconciliation step, provide context for a reasoning step, establish an acceptance contract, or preserve evidence needed to reconstruct a workload. It should not automatically become a second source of truth merely because it is versioned or machine-readable.

This is consistent with EOKS's reconstructability requirement: important workload state, evidence, decisions and artifacts should survive beyond an agent session so another controller or run can resume without hidden agent memory.

## SDD versus AI-native SDLC versus EOKS

These should remain distinct:

```text
SDD
  structures development around durable specifications
  and a specification -> plan -> implementation workflow

AI-native SDLC
  extends that idea across the whole software lifecycle,
  including evaluation, governance, deployment and maintenance
  as a continuous agentic loop

EOKS
  provides a general control architecture for workloads,
  independent of whether the workload is software delivery
  or another domain
```

SDD is therefore useful prior art for artifact-driven execution, while AI-native SDLC is useful prior art for applying a closed-loop, agentic model to the whole SDLC. EOKS should compose with these approaches rather than compete with them.

## Context is the missing connective tissue

The artifact chain does not imply that one artifact is sufficient context for the next step. For example, implementing a `spec.md` may also require relevant ADRs, current repository state, architecture evidence, policies, prior incidents and authoritative source material.

This maps directly onto EOKS's distinction between the resource/evidence universe, workload working set and compiled model context:

```text
artifact + authoritative evidence + policy + current state
                         |
                         v
                    working set
                         |
                         v
                 context compilation
                         |
                         v
                      reasoning
```

Thus the useful EOKS contribution to an AI-native SDLC is not another artifact workflow. It is the control of **which evidence and capabilities are required for each transition**, how that working set is assembled, and how the result is evaluated and reconciled.

## Enforcement and evaluation

The AI-native SDLC pattern also reinforces the existing EOKS distinction between deterministic and agentic execution. Tests, type checks, policy checks, architecture fitness functions and other mechanically decidable invariants should remain deterministic where sufficient. Agentic review can handle evidence-bound questions requiring interpretation, with calibration and human escalation where appropriate.

This connects directly to the existing EOKS prior art on ADRs, specifications, invariants, fitness functions, policy-as-code, evidence and evaluation. The new terminology does not require new runtime primitives.

## Continuous maintenance

The strongest control-loop example is the maintenance stage: a production observation can become new intent and re-enter the same lifecycle. This is a nested workload loop rather than a special "maintenance agent" role:

```text
production state
      |
observation / evidence
      |
evaluation
      |
identified gap
      |
new intent / desired state
      |
reconciliation
      v
SDLC workflow
```

This reinforces EOKS's existing model of continuous knowledge/workload maintenance and nested control loops.

## EOKS boundary

EOKS should not prescribe:

- `intent.md`, `spec.md` or another specific file format;
- a particular SDD framework;
- a fixed Plan/Design/Build/Test/Deploy/Maintain workflow;
- an agent for every SDLC stage;
- artifacts as a new runtime ontology object;
- AIDLC's stage taxonomy, scopes, entropy heuristic or audit/state file formats.

Instead, EOKS provides the underlying control semantics: desired state and policy, working-set/context selection, capability selection, execution, observation, evaluation, durable evidence and reconciliation.

## Research implication

AI-native SDLC, SDD and AIDLC are useful proving grounds for EOKS because they make several hypotheses measurable: whether durable artifacts improve reconstructability, whether artifact-driven handoffs reduce repeated context work, whether adaptive workflow selection reduces process overhead without reducing assurance, whether deterministic state authority improves recovery and auditability, whether uncertainty signals can predict the minimum sufficient process, and how much of the lifecycle can safely move from human execution to deterministic or agentic automation.

The evidence should be evaluated at the workload level—correctness, assurance, recovery, latency, cost and human attention—not merely by artifact count, workflow size, or token reduction.
