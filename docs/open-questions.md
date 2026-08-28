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
