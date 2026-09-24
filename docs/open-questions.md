# Open questions

These are deliberately unresolved. They should become experiments, ADRs or implementation work rather than being silently turned into assumptions.

## Context

- Can context quality be measured independently of task success?
- Can we estimate marginal value of another context item before spending inference cost?
- What is the right representation for context blocks: files, structured objects, graphs, or a hybrid?
- Can a UI expose context as clusters/blocks/graphs without adding cognitive overhead?
- How should contradictory sources be represented?
- Which context selections should be reusable as recipes, and how should those recipes become stale?

## Memory and knowledge

- What deserves promotion from episodic history into durable memory?
- How should memories decay or be invalidated?
- Can provenance and confidence be first-class properties without becoming an opaque trust score?
- When is a graph materially better than structured files or relational storage?
- Which knowledge representations should be canonical, and which should remain derived evidence providers?
- How should delayed outcomes change the trust or applicability of previously promoted knowledge?

## Control plane

- What is the correct scheduling unit: task, run, reasoning step or workflow?
- How should model selection learn from historical outcomes?
- How much autonomy should a scheduler have?
- Can the control plane reconcile desired and observed reasoning state in a Kubernetes-like way?
- What is the minimum intervention point that provides value: before a run, during a run, after verification, or only between runs?
- Which provider-selection decisions need to be persisted as part of the run trace?

## Evidence and software analysis

- How should EOKS express an evidence-provider contract across graphs, type systems, static analyzers, tests and organizational/system context?
- How can the system determine the **minimum sufficient evidence** for a question rather than simply choosing the deepest analyzer?
- When should a project-specific invariant be encoded preventively in types/API design versus detected by an analyzer?
- How should conflicting evidence from different providers be reconciled and explained?

## Evaluation and reliability

- What benchmark isolates context quality from raw model capability?
- How should reliability signals be calibrated against external evidence and actual outcomes?
- Which uncertainty signals remain predictive after controlling for context composition and task difficulty?
- How do we evaluate model upgrades on a user's real workload rather than generic benchmarks?
- What signals predict that an agent is going down an unproductive path early enough to justify intervention?
- Which control decisions benefit enough from reliability estimation to justify its measurement and calibration cost?

## Probabilistic decisions and control

- Can the existing **Decision** primitive represent typed alternatives, ordered scores and yes/no semantic judgments without introducing provider-specific concepts?
- When does a probabilistic decision signal add enough control value to justify its calibration and measurement cost?
- Can Jev, token probabilities, semantic-uncertainty estimators and deterministic validators share one provider-neutral decision-evidence contract?
- How should probability, model confidence, evidence strength and execution authority remain explicitly separate?
- How should step-level calibrated signals be related to trajectory-level outcomes when decisions are correlated?
- What calibration metrics and labelled workloads are sufficient before a threshold is allowed to drive automatic routing, stopping, escalation or execution?
- How sensitive are decision distributions to context wording, option order, irrelevant evidence and prompt injection?
- How should decision calibration be versioned across model/provider changes?
- Can a cheap decision layer reduce frontier-model calls without reducing task-level outcome quality?

## Execution

- What is the minimum portable **Execution** abstraction that can represent a Claude Code session, deterministic process, workflow/graph, remote agent, or agent fleet?
- Which execution semantics should EOKS understand versus delegate to an execution substrate: topology, state, checkpoint, interrupt, resume, retry/recovery, handoff, human gates, observability and placement?
- How should a Claude Code session and its harness tools relate to a higher-level execution runtime without collapsing the two layers?
- Can one Work span multiple executions, sessions, machines, containers, CI jobs or human interventions while retaining coherent provenance and evidence?
- How should execution topology be selected independently from the runtime that realizes it?
- When does a richer execution runtime materially improve durability, inspectability, reliability or recovery enough to justify its abstraction and operational cost?
- Can a simple agent loop remain the default while workflows, graphs and fleets are selected only when their additional coordination semantics are justified?
- What is the correct scheduling unit: Work, execution, session, task, reasoning step or workflow?
- Can checkpoint/interrupt/resume become portable EOKS semantics while storage and implementation remain runtime-specific?
- How should execution events, artifacts, evidence and evaluations be normalized across different substrates?

### Workflow runtimes as execution substrates

- Which workflow/runtime interface is sufficient for EOKS to observe and control execution without coupling the architecture to one runtime?
- Which features are genuinely portable execution semantics versus runtime-specific conveniences?
- Can the same Work be executed through a simple agent loop, deterministic workflow, graph runtime and multi-agent topology while preserving comparable Run/evidence/evaluation records?
- Can EOKS use a runtime without making that runtime an EOKS dependency or architectural center?
- Which capabilities are worth adopting from LangGraph, CrewAI, Microsoft Agent Framework, Google ADK, OpenAI Agents SDK and AutoGen, and which are better left behind their provider boundary?

## Architecture and interoperability

- Which interfaces should be stable across implementations?
- What should be a protocol versus an implementation detail?
- Is EOKS best implemented as a local runtime, service, SDK, or a combination?
- What minimum useful EOKS can exist without hosting infrastructure?
- Which parts of the seven-primitive model survive contact with real run traces?
- How much of an external agent's execution must EOKS observe to make useful control decisions?
- Can existing coding-agent CLIs be integrated non-invasively while preserving useful provenance and outcome data?

## Falsification

EOKS should be considered unsuccessful if a simpler combination of existing tools consistently achieves the same reliability, inspectability and cost without requiring a coordinating control layer.
