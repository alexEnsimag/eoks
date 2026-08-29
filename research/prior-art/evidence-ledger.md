# EOKS prior-art evidence ledger

This ledger makes the systematic review auditable. It separates **what a project provides** from **what independent evidence establishes** and from **the EOKS inference**. It is deliberately conservative: absence of a source means that a claim was not independently established in this pass, not that no source exists.

## Evidence scale

| Level | Meaning | How EOKS should use it |
|---|---|---|
| **A** | Independent controlled experiment, peer-reviewed research, or reproducible external benchmark | Suitable for an architectural hypothesis, subject to workload scope |
| **B** | Independent technical review / hands-on evaluation with concrete observations | Strong evidence for strengths and failure modes; not a universal benchmark |
| **C** | Multiple independent practitioner/community reports or issue discussions | Good for discovering failure modes and hypotheses |
| **D** | Single practitioner report | Anecdotal; never enough by itself for a core architectural claim |
| **P** | Primary/project/vendor evidence only | Prior art, but not independent validation |
| **I** | EOKS interpretation/inference | Must remain explicitly labeled as our conclusion |

A row can have multiple levels because different claims about the same project have different evidence quality.

## Evidence map

| Project / concept | What is independently established | Evidence | What remains unproven | EOKS implication |
|---|---|---|---|---|
| GrapeRoot | Proactive context packing plus reactive graph/MCP exploration is a coherent architecture | P/C | Whether it improves real coding outcomes versus strong search baselines; proprietary internals | Treat as a context-provider architecture; benchmark proactive/reactive/hybrid strategies |
| TencentDB Agent Memory | Multi-resolution memory and reusable Skills/Wiki/CodeGraph can coexist as distinct resources | P | Broad independent field validation and long-term memory quality | Keep memory/procedure/knowledge/evidence semantically separate |
| OKF | Portable Markdown/YAML knowledge bundles with lifecycle/provenance concepts are practical | P | Whether OKF is superior to other representations for agent outcomes | Representation layer, not control plane |
| Graphify | Structural graphs can provide strong retrieval/code-intelligence evidence; results vary by workload | A/B/C | Causal improvement in real SWE task success across diverse repos | Select graph retrieval when its evidence contract beats alternatives |
| CodeGraph | Agent-facing repository graph/querying is a viable representation strategy | P/C | Comparative task-level advantage | Alternative structural provider |
| GitNexus | Precomputed graph plus higher-level process/impact/trace views can support agents | P/B | General advantage over simpler graph/search systems | Treat views as provider capabilities, not an EOKS ontology |
| CodeSight | Repository context/understanding can be packaged for coding agents | P | Strong independent performance evidence | Keep as prior art; do not base architecture on vendor claims |
| Understand Anything | Combining deterministic structure with LLM semantic interpretation creates a useful interactive code model; provenance/freshness can be ambiguous | B/C | General task-level superiority and robust incremental economics | Derived facts need source revision, derivation type and provenance |
| Semgrep | Custom deterministic pattern/dataflow rules are practical and useful | A/B/C | Best depth/cost tradeoff for every invariant class | First targeted evidence provider for suitable questions |
| CodeQL | Deep interprocedural/path-sensitive analysis can establish properties beyond simple structure | A/B | Cost/complexity is workload-dependent | Escalation provider for questions lightweight analysis cannot establish |
| TypeScript compiler | Type-level constraints prevent whole classes of invalid states | A/B/C | Expressiveness for every project-specific invariant | Prefer compile-time enforcement when possible |
| ESLint | Fast deterministic local policy is highly usable | B/C | Deep semantic coverage | First-line local policy provider |
| ts-morph/compiler APIs | Narrow custom semantic checks are feasible without building a full analyzer | P/B | Maintenance economics at scale | Experimental targeted provider |
| TrueCourse | Specifications can be connected to executable scenarios/guards | P/B | General adoption and comparative outcome benefit | Strong prior art for knowledge → policy → executable evidence |
| Superpowers | Explicit planning, staged implementation, TDD and review can improve workflow discipline; overhead is real | B/C | General causal benefit across task classes | Workflow is a selectable policy resource |
| modularity | A concrete coupling model can generate architecture evidence/design artifacts | P/B | General accuracy versus human architecture review | Specialized architecture evidence provider |
| Conductor | Known workflow topology can be executed deterministically without LLM routing overhead | P/B | Whether deterministic topology beats adaptive routing for a given task | Select deterministic vs adaptive orchestration by workload |
| Langroid | Explicit agent/task/tool abstractions support multi-agent delegation | P/B | Comparative advantage over other orchestration frameworks | Execution substrate |
| Plano | Routing, orchestration, traces, guardrails and memory can form a runtime data plane | P | Independent broad outcome evidence | Runtime infrastructure, not EOKS's evidence-aware selection loop |
| LangMem | Semantic/episodic/procedural memory and consolidation can be implemented as explicit operations | P/research | Long-term quality across domains | Memory provider with promotion/validation policy |
| Mem0 | Persistent memory is benchmarkable; update/deletion/staleness matter | A/B | Universal superiority of any memory architecture | Memory eval must test contradiction, deletion, temporal validity and leakage |
| Zep | Temporal memory/context assembly can optimize accuracy, latency and token use together | A/B | Generality beyond published workloads | Temporal validity is a first-class memory property |
| memsearch | Lightweight local memory/search is a viable implementation point | P/D | Competitive quality at scale | Optional provider, not architectural basis |
| Hindsight | Distinguishing facts, experiences and beliefs plus reflection produces richer memory semantics | A/B/C | Whether reflection reliably improves future task outcomes | Memory types and promotion lifecycle should be explicit |
| Xirp / Spotify | Organizational/system context, ownership, services, dependencies and session continuity matter operationally at large scale | P/B | Generality beyond Spotify and beta product constraints | System/institutional context is first-class provider input |
| LangSmith/Langfuse | Tracing and evaluation infrastructure can make agent behavior observable and experimentally comparable | B/C | Traces alone establish neither truth nor causal reliability | Keep observability, evaluation and control separate |
| TransformerLab | Integrated model experimentation/evaluation workbenches are useful | P | Independent comparative evidence | Model-experiment infrastructure |
| Promptfoo | Provider-agnostic eval/red-team regression can be automated and put in CI | B/C | Whether YAML/config approach is optimal for complex workflows | Evaluation harness feeding control decisions |
| Aider | End-to-end coding-agent benchmarks can measure actual software tasks | A/B/C | Benchmark generalization across models/harnesses | Use task-level outcomes as the relevant context metric |
| OpenHands | Open/self-hostable coding-agent execution can be benchmarked reproducibly | A/B/C | Stable ranking across model/runtime changes | Execution baseline/provider |
| OpenAI Evals-style frameworks | Workload-specific evaluation can be codified and rerun | A/B/C | Best evaluation architecture for all EOKS workloads | Evaluation is an input to control |
| CodeRabbit | AI review can find real defects, but reviewer behavior/noise varies | A/B/C | Reliability without human/deterministic evidence | Review is evidence, not truth; use separation of duties for risk |
| Sourcegraph Cody | Combining keyword/search/code-graph context is a practical enterprise strategy | B/C/P | Universal benefit of every component | Strong evidence for heterogeneous context providers |
| Claude Code / `CLAUDE.md` | Project policy/context files are operationally useful; a small 2026 ablation found no measurable correctness improvement on its tested task set | A/C | General effect across tasks and repositories | Measure causal task benefit; never equate more context with better outcome |
| OpenWiki | Generated repository knowledge can improve agent-facing documentation/retrieval workflows | B/C | Fidelity/freshness versus source-grounded alternatives | Derived knowledge must carry provenance and source revision |
| Obsidian | Human knowledge work benefits from durable linked notes | C | Agent-task outcome benefit | Optional human-side workspace |
| OpenWolf | Interaction-derived repository summaries can persist context across sessions | P/C | Robustness and task-level improvement | Candidate memory/evidence provider with freshness controls |
| Liza | Multi-agent review/documentation/audit workflow is feasible | P/D | Independent comparative evidence | Prior art, not architectural evidence |
| Reusable reasoning strategies | Divergence, alternatives, adversarial critique and convergence can be treated as distinct reasoning steps | C/I | Which strategy helps which workload | Model strategy as a selectable resource |
| Hermes | Turning successful work into reusable behavior is a plausible learning mechanism | P/B | Reliability of self-improvement and avoidance of bad policy promotion | Candidate learning provider; require validation before promotion |
| Faraday / Replica | Outer judgment can improve use of a stronger executor under rubric/budget constraints | A | Generality outside studied settings | Strong prior art for separate judgment/execution/evaluation |
| Ronen Brafman planning research | Explicit state, beliefs, action semantics, uncertainty and replanning remain useful agent-control primitives | A | When model-based planning beats simpler LLM workflows | Planning is an optional control/reasoning strategy |

## Coverage status

### High-confidence prior art

These areas have enough external evidence that EOKS should treat them as **existing capabilities**, not research gaps:

- repository graphs and structural representations;
- static/dataflow analysis;
- persistent memory;
- multi-agent orchestration;
- workflow gates;
- tracing/evaluation infrastructure;
- coding-agent execution;
- AI-assisted code review;
- model-based planning as a mature research area.

### Emerging but important

These areas have meaningful evidence but still need workload-specific validation:

- institutional/system context;
- proactive context compilation;
- executable architecture/specification guards;
- memory reflection and self-improvement;
- adaptive judgment over agent trajectories;
- hybrid deterministic/adaptive orchestration.

### Still a plausible EOKS gap

The research did **not** establish a mature general-purpose system that does all of the following as one provider-neutral control loop:

```text
workload + policy + constraints
          ↓
select among heterogeneous
knowledge / evidence / reasoning /
execution providers
          ↓
execute with explicit budget
          ↓
independent verification
          ↓
attribute outcome evidence
          ↓
update future provider selection
```

This is an **EOKS research hypothesis, not a novelty claim**. The evidence only shows that the pieces are fragmented across existing systems. A future search could still uncover a system that covers more of this loop.

## Synthesis matrix

| EOKS capability | Prior-art maturity | Representative systems | What remains interesting |
|---|---|---|---|
| Durable knowledge representation | High | OKF, Markdown/ADR systems | Selection and authority across representations |
| Repository structural knowledge | High | Graphify, GitNexus, CodeGraph, Understand Anything | Choosing graph vs search vs direct source evidence |
| Semantic/context compilation | High/emerging | GrapeRoot, Cody, OpenWiki | Outcome-driven provider composition |
| Memory | High | Hindsight, Mem0, Zep, LangMem, TencentDB | Promotion, invalidation, contradiction and scope policies |
| Workflow execution | High | Superpowers, Conductor, Langroid | Adaptive depth and deterministic/adaptive topology selection |
| Architecture assurance | Emerging | TrueCourse, modularity | Cross-provider assurance policies |
| Deterministic evidence | High | compiler, ESLint, Semgrep, CodeQL | Least-sufficient-provider escalation |
| AI judgment/review | High/emerging | CodeRabbit, Faraday/Replica | Separation of duties and evidence fusion |
| Observability/evaluation | High | LangSmith, Langfuse, Promptfoo, Evals | Turning outcomes into control decisions |
| Planning | High research maturity | Brafman and broader planning literature | Selecting planning depth versus simpler execution |
| Institutional context | Emerging | Xirp | Provider-neutral representation and freshness |
| Provider selection | Fragmented | Pieces in routers/orchestrators | General evidence-aware selection |
| Outcome → future selection | Fragmented | Memory/learning systems provide pieces | Closed-loop resource selection and credit assignment |

## Research standard going forward

Before adding a new EOKS architectural claim, record:

1. the workload/task family;
2. the baseline;
3. the exact provider/model/version;
4. the evidence contract being tested;
5. correctness/task success;
6. evidence coverage/recall where meaningful;
7. cost and latency;
8. failure modes;
9. provenance/freshness behavior;
10. whether the result was independently reproduced.

A token reduction, benchmark score, graph size, memory recall score or agent self-report is **not** by itself evidence that a provider improves EOKS-level outcomes.

## Source trail

The detailed web/repository source URLs used during the review are maintained in the surrounding research notes and tool-specific sections. Where this ledger says `P`, the project itself is the evidence and the claim is intentionally not promoted beyond that. Where it says `A/B/C`, the corresponding independent source should be retained with the relevant tool note when that evidence materially affects the architecture.

This ledger is intentionally not a permanent ranking. Its purpose is to make future updates additive and falsifiable: when a new independent benchmark contradicts an existing conclusion, update the evidence level and the EOKS implication rather than merely adding another tool to a list.
