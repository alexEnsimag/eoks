# Prior art and adjacent systems

EOKS is a synthesis, not a claim that these ideas are new. Several projects and approaches explored in the surrounding research inform different layers.

## GrapeRoot

GrapeRoot was discussed as a context-aware CLI around Claude. It is relevant to task execution, persistent project context and agent workflows. EOKS asks what happens when those capabilities become explicit platform resources rather than remaining inside one CLI.

## CodeSight

CodeSight is relevant to codebase context and repository understanding. It illustrates the value of preparing structured knowledge for coding agents.

## EKOS

EKOS and related enterprise-knowledge systems demonstrate another interpretation of an AI/knowledge operating system. EOKS should remain distinct by focusing on the broader control loop around reasoning, context and execution rather than enterprise knowledge alone.

## TencentDB Agent Memory

The TencentDB work on agent memory reinforces a central EOKS idea: a larger context window does not eliminate the need for useful persistent memory. Memory needs lifecycle, retrieval and relevance semantics.

## XIRP

Spotify's XIRP was investigated as an example of infrastructure thinking around AI systems. It is useful prior art for understanding how an organization can build reusable AI infrastructure rather than isolated prompts.

## OKF

OKF was discussed as a structured file-oriented knowledge/context convention. An important EOKS conclusion is that a protocol or conceptual contract can be implemented with ordinary local files; hosting infrastructure is not what makes a representation meaningful.

## Graphify, Semgrep and CodeQL

These tools demonstrate why deterministic program analysis belongs beside LLM reasoning. Graph and taint/dataflow analysis can expose relationships that are difficult or unreliable to infer from text alone.

## Observability and evaluation tooling

The modern AI tooling ecosystem increasingly provides tracing, evaluations, prompt/version tracking and model comparisons. EOKS treats these as ingredients of a larger feedback loop rather than isolated dashboards.

## Positioning

The important question is not "which tool replaces EOKS?" but "which EOKS layer does this tool implement, and what contract would allow it to participate in the control loop?"
