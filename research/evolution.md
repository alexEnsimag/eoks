# Evolution of the EOKS hypothesis

The EOKS idea did not start as a fully formed AI operating system. The progression matters because it reveals which abstractions survived repeated questioning.

## 1. Context engineering

The initial focus was on giving models better information: how to select, structure and compress context, and how to deal with the limitations of large context windows.

The first major insight was that **more context is not necessarily better context**.

## 2. Context quality

The discussion then shifted toward measuring context itself. Ideas included entropy/noise, relevance, redundancy, contradictions, freshness, provenance and token cost.

This produced the idea of a visual/interactive context workbench where a human can see the blocks being supplied to a model and understand why they were selected.

## 3. Context versus memory

Persistent knowledge then became unavoidable. If the system must repeatedly reconstruct the same information, the information belongs somewhere outside the current context.

This created a distinction between persistent memory/knowledge and the model's working context.

## 4. Knowledge graphs and specialized analysis

Code and other structured domains showed that useful context can be generated from relationships rather than documents alone. Graph extraction, static analysis and dataflow analysis became potential evidence providers.

This shifted the question from “how do we retrieve documents?” to “how do we construct the right evidence for this workload?”

## 5. Evaluation and confidence

Once context and tools became dynamic, we needed to know whether they actually improved outcomes. Confidence, benchmarks, observability and continuous evaluation entered the design.

This was the point where evaluation stopped being an afterthought and became part of execution.

## 6. Model switching

Real model changes demonstrated that models cannot be treated as interchangeable strings. Different models can have materially different behavior, and a new model needs workload-level validation.

This suggested that model selection belongs in a policy/scheduling layer.

## 7. Control plane

The Kubernetes analogy emerged: perhaps AI workloads need a control plane that schedules tasks, selects resources, reconciles execution and learns from outcomes.

At this point EOKS became larger than context engineering.

## 8. Current hypothesis

The current hypothesis is approximately:

> Reliable AI workloads need a coordinating layer that manages context, memory, models, tools, execution and evaluation as resources, using observed outcomes to improve future decisions.

This is deliberately a hypothesis. The repository should make it possible to disprove it.

## Ideas that repeatedly survived

Across the evolution, several ideas kept returning:

- context should be a managed working set;
- persistent knowledge should have provenance and lifecycle;
- graphs are useful representations of relationships, but not necessarily the primary user abstraction;
- deterministic tools and LLM reasoning complement each other;
- model selection should depend on workload characteristics;
- evaluation should feed back into orchestration;
- context quality needs measurable dimensions;
- architecture should not assume one model, one agent or one representation.

## What remains uncertain

We still do not know whether these ideas require a unified EOKS product/runtime or whether a set of well-designed independent components is sufficient.

That is one of the most important architectural questions for future experiments.