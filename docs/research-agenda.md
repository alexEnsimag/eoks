# Research agenda

## 1. Context quality

Build a benchmark where the same task is given different context constructions. Measure correctness, completeness, hallucination/error rate, latency and token cost. Test whether structured context, retrieval, graphs and context splitting produce measurable gains.

## 2. Context observability

Prototype a UI that shows context as inspectable blocks/clusters: source, relevance, provenance, freshness, confidence and token cost. Allow humans to understand what the model saw and what it did not see.

## 3. Memory lifecycle

Compare transcript memory, curated Markdown/files, structured records and graph memory. Measure retrieval usefulness, stale-memory failures and maintenance cost.

## 4. Model routing

Construct a workload with heterogeneous task types. Compare always-strongest-model against capability/cost-aware routing. Include model upgrades and regression testing.

## 5. Deterministic evidence

For software engineering, compare LLM-only reasoning against hybrid workflows using static analysis, code graphs, tests and dataflow/taint analysis.

## 6. Control-plane prototype

Implement the smallest scheduler that accepts a task, chooses context sources/tools/model, executes, evaluates and records the decision. Do not begin with a distributed platform; prove the control loop locally first.

## 7. Continuous assurance

Explore whether evaluation results can automatically alter future routing/context policies. The interesting system is not one that merely measures failures, but one that becomes better at choosing how to work.
