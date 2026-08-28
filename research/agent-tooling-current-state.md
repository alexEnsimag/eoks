# Current state of adjacent agent tooling

This note records externally verifiable facts about the two tools discussed in the agent-code-understanding and architecture-governance research. It deliberately separates **product facts** from the **EOKS architectural interpretation** so that volatile adoption numbers or implementation details do not become part of the core model.

## Understand Anything

Repository: https://github.com/Egonex-AI/Understand-Anything

The project describes itself as an interactive knowledge-graph system for codebases, knowledge bases and documentation. Its repository describes a multi-agent analysis pipeline that builds a graph of files, functions, classes and dependencies, with structural exploration, business/domain views, guided exploration and change-impact capabilities. It also supports several coding-agent and IDE environments rather than being limited to Claude Code.

The repository currently shows substantial community activity: a large public star/fork base, active issues, discussions and a significant stream of pull requests. Those numbers are intentionally not copied into the EOKS model because they change continuously.

### EOKS interpretation

Treat Understand Anything as a **repository knowledge/evidence provider**. Its graph is a reusable source of evidence, not automatically the working context for an agent. EOKS should still decide:

- which relationships are relevant to a workload;
- which facts are deterministic versus inferred;
- how stale a graph is relative to the requested revision;
- how much of the graph belongs in the context package;
- whether the evidence is sufficient for the task.

## TrueCourse

Repository: https://github.com/truecourse-ai/truecourse

TrueCourse describes two related capabilities: code analysis and a specification-to-guard workflow. Its code-analysis side combines deterministic rules with optional LLM-powered analysis for categories including architecture, security, bugs, performance, reliability and code quality. Its guard side curates repository specifications such as PRDs, ADRs and READMEs, generates scenario tests, and executes those scenarios deterministically to detect implementation/specification drift.

The project stores repo-local artifacts under `.truecourse/` and exposes both CLI and dashboard workflows. Its current implementation supports JavaScript/TypeScript, Python and C#, with other languages listed as planned. LLM-powered stages can use Claude Code or provider APIs; deterministic analysis does not require an LLM.

### EOKS interpretation

Treat TrueCourse primarily as an **assurance/evaluation/policy provider**. Its most interesting architectural contribution is the separation between:

```text
LLM-assisted authoring of checks
        |
        v
committed / inspectable scenarios
        |
        v
deterministic verification
```

This is stronger than treating an LLM PR review as the sole source of truth. It also illustrates a broader EOKS principle: some constraints should be executable and evaluated independently of the model that performed the work.

## Community and maturity

Community adoption is useful evidence, but it should not be confused with architectural validity.

- Understand Anything currently has much stronger visible community traction and contribution activity.
- TrueCourse has a smaller community footprint but explores a less-common architectural-governance problem.
- Neither project's GitHub popularity should be encoded as an EOKS dependency or as a quality metric.
- For EOKS experiments, record adoption, release activity, issue/PR activity and maintenance signals as **tool-selection metadata**, separate from the capability contract.

## Alternatives discussed

The earlier comparison also considered:

| Tool / approach | Best viewed as | EOKS role |
| --- | --- | --- |
| Understand Anything | repository knowledge graph and code understanding | context/evidence |
| TrueCourse | architecture analysis and executable specification guards | assurance/evaluation/policy |
| CodeRabbit | AI-assisted PR review | evaluation/feedback |
| Sourcegraph Cody | code retrieval and coding assistance | context retrieval + execution |
| Aider | coding agent/workflow | execution |
| Claude Code + custom checks | general coding-agent runtime plus project policy | execution + policy |
| Semgrep | deterministic structural/security analysis | evidence |
| CodeQL | deeper semantic/dataflow analysis | high-cost evidence |
| Graphify | relationship/graph extraction | evidence |

The important comparison is therefore not a leaderboard. These systems answer different questions and can be composed.

## Recommended EOKS abstraction

The common interface should be an **Evidence Provider** rather than a product-specific integration:

```text
EvidenceProvider
  query/task scope
  repository/data revision
  -> evidence items
     provenance
     validation/confidence
     scope
     freshness
     cost/latency
```

An EOKS scheduler can then select the minimum sufficient evidence for a workload. A high-risk change may justify tests plus static analysis plus architecture guards; a simple documentation change may require none of those. This is the connection between these tools and the EOKS control loop.

## Sources

- Understand Anything repository: https://github.com/Egonex-AI/Understand-Anything
- Understand Anything pull requests/community activity: https://github.com/Egonex-AI/Understand-Anything/pulls
- TrueCourse repository: https://github.com/truecourse-ai/truecourse
