# Current state of adjacent agent tooling

This note records externally verifiable facts about the Claude/developer-tooling discussion. It deliberately separates **product facts** from the EOKS architectural interpretation so that volatile implementation details and adoption signals do not become part of the core model.

## Understand Anything

Repository: https://github.com/Egonex-AI/Understand-Anything

Understand Anything describes itself as an interactive knowledge-graph system for codebases, knowledge bases and documentation. Its repository describes a multi-agent analysis pipeline that builds a graph of files, functions, classes and dependencies and provides structural exploration, semantic/domain views, guided exploration and change-impact capabilities. It supports multiple coding-agent and IDE environments rather than being limited to Claude Code.

The project has substantial visible community and contribution activity. Exact stars, forks and issue counts are intentionally not copied here because those numbers change continuously.

### EOKS interpretation

Treat Understand Anything as a **repository knowledge/evidence provider**. Its graph is a reusable source of evidence, not automatically the working context for an agent. EOKS should decide which relationships are relevant, distinguish observed structural facts from inferred explanations, account for graph freshness relative to the requested revision, and select how much evidence belongs in the context package.

## TrueCourse

Repository: https://github.com/truecourse-ai/truecourse

TrueCourse describes two related capabilities: code analysis and a specification-to-guard workflow. The analysis side combines deterministic rules with optional LLM-powered analysis for areas including architecture, security, bugs, performance, reliability and code quality. The guard workflow works from repository specifications such as PRDs, ADRs and READMEs, turns them into scenario-style checks, and executes those checks to detect implementation/specification drift.

The project provides CLI/dashboard workflows and stores repo-local artifacts under `.truecourse/`. Its current documented language coverage includes JavaScript/TypeScript, Python and C#, with additional languages planned. LLM-assisted stages can use Claude Code or model-provider APIs; deterministic analysis does not inherently require an LLM.

### EOKS interpretation

Treat TrueCourse primarily as an **assurance/evaluation/policy provider**. Its most interesting architectural pattern is the separation between LLM-assisted authoring of checks and inspectable, executable verification:

```text
LLM-assisted authoring
        |
        v
committed / inspectable checks
        |
        v
deterministic verification
```

That is stronger than relying on an LLM review as the only source of truth. It also illustrates an EOKS principle: some constraints should be executable and evaluated independently of the model that performed the work.

## Community and maturity

Community adoption is useful evidence for tool selection, but it is not evidence that an architectural abstraction is correct.

- Understand Anything currently has substantially stronger visible community traction and contribution activity.
- TrueCourse has a smaller community footprint but explores a distinctive architecture/specification-governance problem.
- Neither project's popularity should become an EOKS dependency, quality metric or architectural assumption.
- EOKS experiments can record adoption, release cadence, issue/PR activity and maintenance signals as **tool-selection metadata**.

## Alternatives discussed

| Tool / approach | Primary capability | EOKS role |
| --- | --- | --- |
| Understand Anything | repository knowledge graph / code understanding | context + evidence |
| TrueCourse | architecture analysis + executable specification guards | assurance + evaluation + policy |
| CodeRabbit | AI-assisted PR review | evaluation / feedback |
| Sourcegraph Cody | code retrieval and coding assistance | context retrieval + execution |
| Aider | coding-agent workflow | execution |
| Claude Code + custom checks | coding-agent runtime + project policy | execution + policy integration |
| Semgrep | deterministic structural/security analysis | evidence / verification |
| CodeQL | deeper semantic/dataflow analysis | high-cost evidence / verification |
| Graphify | code relationship graph extraction | structural evidence |

These tools should not be treated as a leaderboard: they answer different questions and can be composed.

## Evidence Provider contract

The common EOKS abstraction suggested by this landscape is an **Evidence Provider**, not a product-specific integration:

```text
EvidenceProvider
  input:
    workload/task scope
    repository/data revision
    evidence requirements
  output:
    evidence items
    provenance
    validation/confidence
    scope
    freshness
    cost/latency
```

An EOKS scheduler can then select the **minimum sufficient evidence** for a workload. A high-risk code change may justify tests, static analysis and architecture guards; a simple documentation change may require little or none of that. This makes evidence selection part of the control loop rather than an unconditional preprocessing step.

## Sources

- Understand Anything: https://github.com/Egonex-AI/Understand-Anything
- Understand Anything pull requests: https://github.com/Egonex-AI/Understand-Anything/pulls
- TrueCourse: https://github.com/truecourse-ai/truecourse
