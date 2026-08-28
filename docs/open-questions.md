# Open questions

These are deliberately unresolved. They should become experiments, ADRs or implementation work rather than being silently turned into assumptions.

## Context

- Can context quality be measured independently of task success?
- Can we estimate marginal value of another context item before spending inference cost?
- What is the right representation for context blocks: files, structured objects, graphs, or a hybrid?
- Can a UI expose context as clusters/blocks/graphs without adding cognitive overhead?
- How should contradictory sources be represented?

## Memory

- What deserves promotion from episodic history into durable memory?
- How should memories decay or be invalidated?
- Can provenance and confidence be first-class properties?
- When is a graph materially better than structured files or relational storage?

## Control plane

- What is the correct scheduling unit: task, agent, reasoning step or workflow?
- How should model selection learn from historical outcomes?
- How much autonomy should a scheduler have?
- Can the control plane reconcile desired and observed reasoning state in a Kubernetes-like way?

## Evaluation

- What benchmark isolates context quality from raw model capability?
- How should confidence be calibrated against external evidence?
- How do we evaluate model upgrades on a user's real workload rather than generic benchmarks?
- What signals predict that an agent is going down an unproductive path?

## Architecture

- Which interfaces should be stable across implementations?
- What should be a protocol versus an implementation detail?
- Is EOKS best implemented as a local runtime, service, SDK, or a combination?
- What minimum useful EOKS can exist without hosting infrastructure?

## Falsification

EOKS should be considered unsuccessful if a simpler combination of existing tools consistently achieves the same reliability, inspectability and cost without requiring a coordinating control layer.
