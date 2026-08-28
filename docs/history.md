# Project evolution

EOKS emerged from a series of questions about AI-assisted software engineering and context management.

## Starting point: better context

The initial concern was that agents often fail not because the model lacks capability, but because the model receives the wrong information: too much noise, missing dependencies, stale assumptions or an unstructured mixture of unrelated material.

## Context quality

This led to exploring context splitting, context entropy, interactive context inspection, structured Markdown/file conventions, graphs and retrieval. The key shift was from **context size** to **context quality**.

## Memory

The next step was separating durable memory from current context. Work on agent memory reinforced that a large context window does not remove the need for persistent, curated knowledge.

## Code intelligence

Software-engineering experiments exposed a second limitation: some questions are fundamentally structural. Code graphs, taint/dataflow analysis, CodeQL, Semgrep and similar tools can provide evidence that should be fed into LLM reasoning instead of asking the model to rediscover it from source text.

## Evaluation

The discussion then moved toward monitoring confidence, evaluating context, and safely switching between model versions. This introduced the idea that evaluation must be part of the system itself.

## Control plane

Finally, these pieces suggested a broader abstraction: something analogous to an operating system/control plane for AI workloads. A scheduler could select the task strategy, context, tools and model, then evaluate the result and feed evidence back into future decisions.

## Current position

EOKS is therefore a hypothesis about a missing systems layer. The purpose of this repository is to test that hypothesis incrementally, not to assume the final architecture in advance.
