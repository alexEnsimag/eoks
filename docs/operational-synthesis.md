# EOKS operational synthesis

> Translate the conceptual synthesis into decisions about what to use, build, investigate, and measure. This document is an operational companion to [`conceptual-synthesis.md`](./conceptual-synthesis.md), not a replacement architecture.

## 1. Why this layer exists

The conceptual synthesis established relationships among the ideas accumulated in EOKS. The next question is practical:

> **Given what we currently believe, what should we use now, what should we prototype, what should we test, and what should remain research?**

Without this layer, synthesis can become an increasingly elegant description of possibilities without changing engineering practice. EOKS should instead use its synthesis as a decision system:

```text
research / observation
        |
        v
conceptual synthesis
        |
        v
operational triage
   /       |        \
use now  hypothesis  research
   |         |          |
workflow   experiment  investigate
   |         |          |
   +---------+----------+
             |
        evidence / outcome
             |
             v
       update synthesis
```

This document therefore adds **action status** to the existing conceptual status vocabulary. It deliberately does not add runtime primitives.

## 2. Two different questions: epistemic status and action status

A concept can be well supported but not worth implementing, or uncertain but highly valuable to investigate.

Keep these dimensions separate.

### Epistemic status

- **Observed** — seen in an implementation, workload, or controlled observation.
- **Supported** — backed by multiple relevant sources or experiments.
- **Mixed** — evidence depends materially on workload, model, repository, or assumptions.
- **Hypothesis** — plausible synthesis claim that still needs validation.
- **Research** — interesting but not sufficiently formulated or evidenced for an EOKS claim.
- **Rejected** — evidence argues against the proposed mechanism or abstraction.
- **Canonical** — adopted into the current EOKS architecture.

### Action status

- **Use now** — sufficiently understood to affect an existing workflow without requiring an EOKS-specific implementation.
- **Prototype** — useful enough to build a small capability and learn from it.
- **Experiment** — formulate a controlled test before committing to implementation or architecture.
- **Research first** — investigate existing evidence or prior art before designing an experiment.
- **Watch** — retain because it may become relevant, but no immediate action is justified.
- **Do not pursue** — current evidence or cost/benefit does not justify investment.

The combination is more informative than either status alone.

For example:

```text
hidden-state signals       = research + research first
trajectory capture         = supported + use now
computed-artifact reuse    = hypothesis + prototype
model routing              = strong + experiment
Task / Run / Outcome       = canonical + use now
```

## 3. Operational decision record

For each important concept or capability, record the smallest useful decision chain:

```text
concept
  -> evidence
  -> current interpretation
  -> epistemic status
  -> action status
  -> candidate mechanisms/tools
  -> missing capability
  -> experiment / workflow trial
  -> measurable outcome
  -> EOKS impact
```

This should preserve the reasoning behind decisions without turning every research note into a formal specification.

When a decision changes, record the new evidence and the reason for the change. Do not silently rewrite the current interpretation.

## 4. Operational map

The current synthesis can be triaged into six practical zones:

```text
                         EOKS synthesis
                               |
          +--------------------+--------------------+
          |                    |                    |
      use now              prototype            investigate
          |                    |                    |
   existing workflow    small capability      research / test
          |                    |                    |
          +--------------------+--------------------+
                               |
                         evidence/results
                               |
                         architecture only
                       when evidence warrants it
```

The default direction is **not** from an interesting concept to an architectural primitive. The default direction is from a workload problem to the minimum intervention that can test or solve it.

## 5. Current operational triage

| Area | Current interpretation | Epistemic status | Action | Useful now | Candidate mechanisms / tools | Missing capability to investigate | First useful test | Potential EOKS impact |
|---|---|---|---|---|---|---|---|---|
| Context acquisition | Different providers can obtain different evidence slices | Supported | Use now + experiment | Yes | search, structural search, graph/index, semantic retrieval, agentic exploration | provider policy and acquisition attribution | compare acquisition strategies on one repository task | Context / Decision / Policy |
| Working-set management | Active evidence should be selected rather than equated with the full context window | Hypothesis with strong systems analogy | Experiment | Yes | context blocks, pinning, relevance/freshness filters, retrieval | explicit working-set policy | broad context vs selected working set | Context / Policy |
| Context contracts | Typed task/context boundaries may reduce subagent rediscovery | Hypothesis | Experiment | Prototype | structured handoff, evidence slice, scope/exclusion hints | contract format and validation | fresh subagent with/without contract | Context / Run |
| Context observability | Inspectable context can diagnose misses and pollution | Supported as an observability need; benefit of editing remains open | Prototype | Yes | context manifest, provenance, freshness, relevance, diffs | lightweight context workbench | inspect and compare successful/failed runs | Context / Evaluation |
| Tool/provider selection | Select the cheapest sufficient capability for the evidence requirement | Strong | Use now + experiment | Yes | retrieval, graph queries, type checking, tests, static analysis, runtime evidence, LLM reasoning | adaptive provider policy | fixed vs evidence-driven provider selection | Decision / Policy |
| Deterministic verification | Independent deterministic checks provide complementary evidence | Strong | Use now | Yes | compiler/type checks, tests, static analysis, analyzers | evidence-aware escalation | LLM-only vs hybrid verification | Evaluation |
| Trajectory capture | Process evidence complements final outcome | Strong | Use now | Yes | OTel, execution trace, decision records | consistent semantic trace schema | outcome-only vs trajectory-aware review | Run / Evaluation |
| Intermediate evidence | Evidence available before completion can support earlier control decisions | Supported / promising | Experiment | Partly | intermediate artifacts, tool results, model-native signals | evidence composition and provenance | early-stop/verify policy | Evaluation / Decision |
| Durable execution state | Explicit state can improve long-horizon recovery beyond transcript history | Hypothesis | Experiment | Prototype | task ledger, attempts, observations, invalidations | state model and stale-state handling | reset/recovery benchmark | Run / Context |
| Durable memory | Persistent knowledge/experience should be validated and scoped | Supported | Use now + experiment | Yes | curated files, structured records, learning records | controlled promotion/invalidation | fresh-session reconstruction | Learning / Resource |
| Procedural learning | Repeated successful behavior may be promoted into reusable procedures | Promising | Experiment | Prototype | workflows, scripts, skills, validated procedures | promotion, rollback, drift detection | repeated task distribution shift | Learning / Execution |
| Computed artifacts | Expensive computation may be worth persisting with dependencies | Hypothesis | Prototype | Partly | artifact store, derived files, cached analyses | dependency-aware invalidation | reuse vs recompute simulation | Resource / Run |
| General reuse | Memory, context caches, inference state and computed artifacts share reuse concerns but remain different mechanisms | Synthesis hypothesis | Experiment | Conceptually | caches, artifacts, durable state | common reuse metadata/protocol | classify reuse by validity/scope/cost | Cross-cutting, no primitive yet |
| Model routing | Model choice can be treated as a policy/resource decision | Strong | Experiment | Yes | model router, historical outcome data | calibrated decision policy | fixed model vs adaptive selection | Decision / Policy |
| Adaptive orchestration | Extra orchestration should earn its cost through measurable benefit | Supported hypothesis | Experiment | Partly | executor/reviewer/verification, conductor | attribution and intervention policy | single agent vs minimal topology | Policy / Run |
| Graduated autonomy | Allowed autonomy should depend on evidence, consequence, reliability and authority | Strong synthesis hypothesis | Experiment | Partly | approval gates, verification, escalation | assurance-to-authority policy | low/high consequence paired tasks | Policy / Evaluation |
| Risk/consequence | The same uncertainty can justify different actions depending on consequences | Strong conceptual clarification | Prototype + experiment | Yes | risk classes, action gates, verification tiers | explicit mapping to assurance | same uncertainty, different consequences | Policy / Evaluation |
| Authority/governance | Capability and authorization are separate selection constraints | Strong conceptual clarification | Research first + prototype | Partly | permissions, scoped providers, approval | consistent authority model | authorized vs merely capable provider | Policy / Resource |
| Hidden/model-native signals | Internal model signals may provide useful evidence but are provider-specific | Research | Research first | No | activations/probes where available | stable access, calibration, interpretation | signal vs outcome calibration | Evaluation, if validated |
| Latent/iterative reasoning | Additional computation may improve outcomes in workload-dependent ways | Research / mixed | Experiment | No direct implementation assumption | adaptive compute, repeated sampling, reasoning budgets | observable stopping/economic policy | fixed vs adaptive compute budget | Run / Evaluation |
| Knowledge representations | Different representations optimize different acquisition/use operations | Supported | Use now + experiment | Yes | Markdown, ADRs, structured records, graphs, indexes | representation selection/maintenance | representation vs raw exploration | Resource / Context |
| Knowledge maintenance | Derived knowledge needs dependency-aware update/invalidation | Strong | Use now + prototype | Yes | source links, dependency graphs, freshness checks | impact detection and update policy | mutate source and trace affected artifacts | Resource / Policy |
| OS/cache analogies | Classical systems ideas provide testable optimization hypotheses, not architecture requirements | Supported as research lens | Experiment selectively | Yes | working sets, locality, admission, prefetch, scheduling | semantic metrics and workload adaptation | transfer one mechanism at a time | Systems optimization research |
| AI-native SDLC artifacts | Durable specs/plans/reviews can be treated as workload state/evidence | Emerging / promising | Experiment | Yes | versioned specs, plans, review artifacts | artifact-driven handoff policy | artifact vs transcript handoff | Context / Run / Evaluation |

## 6. What is already useful without building EOKS infrastructure?

Several ideas are mature enough to influence the user's current LLM workflows immediately.

### 6.1 Treat context acquisition as a decision

Do not ask only “how do I give the model more context?” Ask:

```text
What evidence is required?
Which provider can obtain it?
What is the cheapest sufficient acquisition path?
What was actually acquired?
What was missed?
```

This suggests using existing search, graph, structural and semantic tools as **different acquisition mechanisms**, not as competing universal context systems.

### 6.2 Keep a context/working-set manifest

For expensive or long-running tasks, record:

- sources included;
- sources deliberately excluded when material;
- provenance;
- freshness/version;
- authority;
- reason for inclusion;
- cost of acquisition;
- important context misses.

This is useful even before an automated context compiler exists.

### 6.3 Capture trajectories, not just answers

A final answer is insufficient for diagnosing why an agent succeeded or failed. Capture meaningful tool calls, evidence acquired, decisions, verification and outcome.

This also creates the raw material needed for later learning, provider-selection evaluation and deterministic promotion.

### 6.4 Separate verification from generation

When a deterministic or independent check is cheap, use it as an evidence source rather than asking the same model to judge its own output.

The goal is not “always verify with a tool.” The goal is to select the minimum sufficient independent evidence for the consequence of the action.

### 6.5 Preserve reusable artifacts

If an investigation produces a structured result that will plausibly be useful again, record its provenance and dependencies rather than leaving it only in the transcript.

This is a practical first step toward testing the computed-artifact hypothesis.

## 7. Candidate tools: evaluate mechanisms, not brands

Existing tools should be mapped to the mechanism they implement. EOKS should not canonize a tool merely because it is popular or convenient.

A useful record is:

| Problem | Mechanism | Candidate implementation | What it actually provides | What remains EOKS-level |
|---|---|---|---|---|
| repository navigation | structural/relational representation | graph/index tooling | navigation over known repository structure | deciding when this representation is sufficient |
| repository search | lexical/semantic acquisition | search/retrieval tooling | candidate evidence | evidence requirement and sufficiency |
| code structure | deterministic analysis | compiler/LSP/static analysis | precise structural evidence | provider selection and interpretation |
| runtime behavior | observability | OTel/telemetry | execution observations | semantic evaluation and control |
| context reuse | caching | context/artifact caches | retained information or results | validity, scope and reuse policy |
| model choice | routing | model routers | provider switching | outcome-aware policy |
| durable knowledge | structured persistence | Markdown/records/knowledge stores | stored representations | validation, promotion and invalidation |
| workflow enforcement | deterministic execution | scripts/workflows | repeatable procedure | promotion criteria and drift handling |

The existing subcategory-based tool-selection matrix remains valuable here. It answers **how a human or agent should compare tools for a mechanism**. It should not be collapsed into the evidence ledger.

## 8. Where EOKS likely needs to build something

The strongest gaps are not generic “AI infrastructure” gaps. They are coordination gaps between mechanisms that existing tools usually expose separately.

### 8.1 Evidence-aware provider selection

Potential capability:

```text
engineering question
        |
 evidence requirement
        |
 candidate providers
        |
 capability + reliability + cost + latency + freshness
        |
 minimum sufficient provider set
        |
 evidence
        |
 evaluation
```

This is a good candidate for a small EOKS prototype because the mechanism is already well understood enough to instrument.

### 8.2 Context compiler / working-set policy

Potential capability:

```text
intent + task state
        |
 evidence requirements
        |
 candidate representations
        |
 relevance + authority + freshness + dependency + cost
        |
 working set
        |
 context materialization
```

The important research question is whether this actually beats strong agentic exploration, not whether the compiler architecture looks elegant.

### 8.3 Computed-artifact registry

Potential capability:

```text
computation
   |
output + dependencies + provenance + validity
   |
artifact registry
   |
reuse / invalidate / recompute
```

This is especially interesting for repeated repository analysis, API-flow reasoning, architecture maps and other expensive structured investigations.

### 8.4 Assurance-to-autonomy policy

Potential capability:

```text
uncertainty + evidence + consequence + authority + historical reliability
                         |
                    assurance level
                         |
              permitted action/autonomy
                         |
            verify / escalate / execute
```

This should remain a policy experiment before becoming an architectural construct.

## 9. Highest-value experiments

Do not start all experiments in the research agenda. The first experiments should maximize learning about EOKS's central claims while solving real workflow pain.

### Experiment A — context / working-set selection

**Question:** Can explicit working-set selection reduce context/acquisition cost without reducing engineering outcome?

**Baseline:** strong agentic repository exploration with the existing workflow.

**Intervention:** acquisition providers produce a candidate working set with provenance, relevance and freshness metadata.

**Compare:** broad exploration, retrieval-assisted exploration, graph/structural-assisted exploration, and a hybrid.

**Metrics:**

- task correctness/completeness;
- relevant evidence coverage;
- discovery/tool calls;
- repeated exploration;
- context tokens;
- context growth/churn;
- latency;
- total cost;
- context misses;
- verification effort.

**Failure modes:** over-pruning, stale representation, representation bias, retrieval thrashing, loss of useful exploratory behavior.

**What it teaches EOKS:** whether working-set management is an actual control capability or merely a useful analogy.

### Experiment B — minimum sufficient evidence/provider selection

**Question:** Can an evidence-aware controller reach the same acceptance criterion with less cost than always using the strongest available provider?

**Baseline:** strongest practical analysis/tool/model path.

**Intervention:** select providers according to evidence requirements, capability, reliability, cost, latency and freshness.

**Candidate providers:** retrieval, graph query, compiler/type checks, tests, static analysis, runtime evidence, LLM reasoning and human review where appropriate.

**Metrics:** outcome, evidence quality, cost, latency, false stops, unnecessary escalations and defect escape.

**Failure modes:** provider blind spots, correlated evidence, premature stopping, stale evidence, selection-policy drift.

**What it teaches EOKS:** whether “minimum sufficient evidence” can become a measurable control objective.

### Experiment C — computation/artifact reuse

**Question:** Can expensive derived computations be reused safely when their dependencies have not changed?

**Baseline:** recompute.

**Intervention:** persist structured artifacts with dependency/provenance metadata and perform dependency-aware invalidation.

**Scenarios:** repository analysis, architectural maps, API-flow analysis, semantic summaries or other repeated investigations.

**Metrics:** reuse rate, recomputation avoided, latency, cost, correctness, stale-artifact rate, invalidation precision and maintenance overhead.

**Failure modes:** hidden dependencies, false validity, excessive invalidation, artifact pollution, changing model behavior.

**What it teaches EOKS:** whether reusable computation deserves a common protocol or remains a collection of specialized mechanisms.

## 10. Second-wave experiments

Once the first vertical slice is instrumented, prioritize:

1. **context contracts for fresh subagents** — test whether explicit contracts reduce rediscovery;
2. **execution-state continuity** — compare transcript history with explicit state and recovery;
3. **procedural promotion** — validate whether repeated successful traces can safely become deterministic procedures;
4. **model × infrastructure** — test whether infrastructure gains transfer across model capability levels;
5. **adaptive orchestration** — determine when reviewer/verification workers justify coordination cost;
6. **assurance-driven autonomy** — vary consequence and evidence while holding uncertainty approximately constant;
7. **knowledge maintenance** — mutate authoritative sources and measure dependency-aware repair;
8. **AI-native SDLC artifacts** — test durable artifacts as context/evidence across workflow stages.

Do not promote any of these to architectural requirements based solely on intuition or analogy.

## 11. Research-first topics

Some ideas should remain deliberately outside immediate implementation.

### Hidden/model-native states

Interesting because they may expose evidence unavailable through ordinary APIs. Not yet suitable as an EOKS abstraction because observability, interpretation, stability and reuse are provider-dependent.

### Adaptive latent computation

Interesting because more internal computation may change quality/cost tradeoffs. Needs workload-specific evidence and a measurable stopping policy before influencing architecture.

### Learned cache/replacement policies

Classical cache policies provide strong baselines, but semantic evidence has different costs and validity conditions. Test simple policies first.

### General reusable-state abstraction

The reuse lens is useful, but a common runtime protocol should only emerge if experiments show enough shared semantics across context caches, artifacts, memory and execution state.

## 12. Decision criteria for architectural promotion

A capability should influence the canonical architecture only when at least one of the following is demonstrated:

1. the current model cannot express an important workload behavior;
2. repeated implementations expose a stable missing relationship;
3. the concept has independent/convergent evidence and clear operational semantics;
4. a controlled experiment demonstrates a material engineering benefit;
5. treating it as a first-class concept substantially improves correctness, observability or control.

Conversely, do **not** promote a concept merely because:

- it has an attractive analogy;
- it appears in several papers under different names;
- a tool exposes it as an API object;
- it is technically interesting;
- it makes the diagram look more complete.

## 13. Measurement discipline

Operationalization should inherit the existing research-agenda methodology rather than inventing a new benchmark vocabulary.

At minimum, experiments should report:

**Outcome** — correctness, completeness, regressions, evidence quality.

**Efficiency** — model tokens, tool calls, latency, total cost.

**Context health** — relevance, coverage, redundancy, contradiction, freshness, growth/churn.

**Acquisition** — discovery work, repeated searches, missed evidence.

**Autonomy** — retries, recovery, intervention frequency and successful completion without intervention.

**Cost distribution** — median and useful tail percentiles, variance, success-conditioned cost and retry/tail cost where practical.

Always compare against the strongest practical baseline, not a deliberately weak baseline.

## 14. The first vertical slice

The preferred implementation path is a narrow, instrumented software-engineering workload rather than a general EOKS platform.

```text
Task
  |
  v
intent / acceptance criterion
  |
  v
acquisition + provider selection
  |
  v
working set / context
  |
  v
agent execution
  |
  +---- trajectory / observations
  |
  v
verification
  |
  v
Outcome
  |
  v
Evaluation
  |
  +---- reusable artifact candidate
  |
  +---- learning candidate
  |
  v
next Decision
```

Record enough information to answer:

- What did the controller know?
- What evidence did it request?
- Why was a provider selected?
- What context was actually supplied?
- What computation occurred?
- What evidence appeared during execution?
- What verification happened?
- What was the outcome?
- What was reused later?
- What became stale?
- Where did human intervention occur?
- What did the next decision learn from the run?

This is the minimum substrate needed to move from conceptual claims to attributable evidence.

## 15. What this PR does not do

This operational synthesis intentionally does **not**:

- change the seven provisional runtime primitives;
- introduce `Evidence`, `Computation`, `Risk`, `Autonomy`, `Authority`, `Reuse`, `Lineage` or `DerivedState` as primitives;
- replace the conceptual synthesis;
- replace the research agenda;
- replace the tool-selection matrix;
- declare a specific graph, retrieval system, cache, model router or agent framework to be the EOKS implementation;
- claim that a proposed experiment has already demonstrated a benefit.

The purpose is to make the existing body of work actionable while keeping uncertainty explicit.

## 16. How the synthesis should evolve

Future research PRs should preferably end with a short operational disposition:

```text
Concepts affected:
  ...

Evidence status changed:
  ...

Action status changed:
  ...

Existing workflow/tool implication:
  ...

Experiment added or completed:
  ...

Architecture impact:
  none / clarify / change

Why:
  ...
```

This creates a durable decision history:

```text
idea
  -> evidence
  -> synthesis
  -> operational decision
  -> implementation / experiment
  -> result
  -> revised synthesis
```

That history is itself part of EOKS's epistemic discipline. It prevents future consolidation from preserving only the final vocabulary while losing the evidence and reasoning that produced it.

## 17. Current priority

The immediate priority should be **one instrumented vertical slice**, not another broad architectural rewrite.

The recommended order is:

```text
1. instrument the workload
        |
2. measure baseline
        |
3. context / working-set intervention
        |
4. evidence-provider selection intervention
        |
5. computation/artifact reuse intervention
        |
6. compare outcomes and costs
        |
7. update synthesis
        |
8. only then consider architectural promotion
```

The central operational principle is:

> **EOKS should earn abstractions through measurable control problems, not create control problems to justify abstractions.**
