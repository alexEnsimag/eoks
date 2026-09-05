# EOKS conceptual synthesis

> Cross-cutting synthesis of the concepts currently composing the EOKS hypothesis. This document maps ideas across the canonical architecture and research corpus; it is not a competing architecture or a new source of truth.

## 1. Why this layer exists

EOKS now has a strong canonical architecture and a rich research corpus. The main risk is no longer lack of concepts; it is losing information when related ideas are consolidated into a smaller vocabulary before their relationship is understood.

This document is the bridge:

```text
                 CANONICAL ARCHITECTURE
                         ^       |
                         |       v
                  conceptual synthesis
                         ^       |
                         |       v
                   RESEARCH CORPUS
```

The synthesis layer should answer questions that neither side answers alone:

- Which ideas are actually the same concept?
- Which are different representations of one underlying thing?
- Which are complementary mechanisms that should remain separate?
- Which ideas only look similar because they occur at different lifecycle stages?
- Which emerging concepts are strong enough to influence architecture?
- Which hypotheses need experiments before they become design claims?

**Consolidation should reduce conceptual ambiguity, not conceptual richness.**

When ideas overlap, first classify the relationship as **same concept**, **different representation**, **complementary mechanism**, or **genuinely distinct concept**. Only then decide whether anything should be merged.

---

## 2. Current canonical model

The current architecture remains the reference model:

```text
intent
  -> desired state / outcome
  -> policy
  -> conductor / reconciliation
  -> resource + working-set + execution selection
  -> run
  -> observe / verify
  -> outcome
  -> evaluation / evidence
  -> actual state
  -> reconcile
```

The seven provisional runtime primitives remain:

```text
Task · Context · Run · Decision · Policy · Evaluation · Outcome
```

Nothing in this synthesis promotes a new primitive. In particular, **Computation**, **Evidence**, **State**, **Memory**, **Knowledge**, and **Working Set** are currently treated as cross-cutting concepts, resources, representations, lifecycle structures, or existing architectural constructs until evidence demonstrates that they need independent runtime identity.

---

## 3. The integrated EOKS map

The strongest synthesis from the accumulated work is a single control loop viewed through several dimensions:

```text
                              INTENT
                                |
                         desired state/outcome
                                |
                              POLICY
                                |
                                v
                         CONTROL / RECONCILE
                                |
             +------------------+------------------+
             |                  |                  |
           STATE          INFORMATION          COMPUTATION
             |          / REPRESENTATION           |
             |                  |                  |
             +------------ RESOURCE --------------+
                                |
                         working set / context
                                |
                           execution / run
                                |
                         observe + verify
                                |
                 +--------------+--------------+
                 |              |              |
              outcome       trajectory    intermediate
                 |                           evidence
                 +--------------+--------------+
                                |
                         EVALUATION / ASSURANCE
                                |
                         actual state / evidence
                                |
                             RECONCILE
                                |
              +-----------------+------------------+
              |                 |                  |
           LEARNING       REUSE / INVALIDATE   KNOWLEDGE
              |                 |                  |
              +-----------------+------------------+
                                |
                         future control
```

This is not a replacement architecture. It is a map showing that many apparently separate EOKS ideas are dimensions of the same workload-control system.

---

## 4. Five cross-cutting dimensions

### 4.1 State

EOKS repeatedly encounters different state domains:

- desired workload state;
- actual workload state;
- execution/run state;
- working-set state;
- knowledge state;
- representation freshness/validity state;
- computation state;
- evaluation/assurance state;
- learned-policy or procedure state;
- relevant external/world state.

These should not automatically be collapsed into one `State` object. The useful synthesis is that the **conductor reconciles across several state domains**.

A useful question for future work is:

```text
What state is being observed?
What state can this action change?
What state becomes stale when a dependency changes?
What state must be durable to reconstruct the workload?
```

### 4.2 Information and representation

EOKS manages transformations between representations rather than a single universal knowledge store:

```text
authoritative source
       |
       v
representation(s)
       |
       v
evidence / derived facts
       |
       v
working set
       |
       v
context materialization
```

Graphs, indexes, timelines, source slices, summaries, memory records, procedural knowledge, runtime observations and computed semantic artifacts can all be useful representations. A graph is therefore a representation, not the ontology of EOKS.

The compiler analogy is useful: several intermediate representations can coexist because different representations optimize different operations. The EOKS question is which transformations preserve provenance, freshness and meaning sufficiently for the intended use.

### 4.3 Computation

The research corpus increasingly describes computation at several levels:

- deterministic tool execution;
- model inference;
- iterative/latent reasoning;
- context compilation;
- sampled alternative reasoning;
- validation and analysis;
- derived computation producing reusable artifacts;
- long-lived procedures promoted from successful reasoning.

The synthesis hypothesis is:

> EOKS may eventually need to reason about computation as a resource with cost, latency, dependencies, state, outputs, evidence, reproducibility and reuse potential.

This is intentionally a **hypothesis**, not a sixth/eighth primitive. Existing architecture can already represent computation through Run, Resource, Provider, Context, Outcome and Evaluation.

### 4.4 Evidence and assurance

Evidence now spans several layers:

```text
model-native signals
        +
intermediate results
        +
trajectory/process evidence
        +
external observations
        +
deterministic validation
        +
human review
        +
historical calibration
        +
economic evidence
        |
        v
   evaluation / assurance
        |
        v
 control decision
```

Evidence is therefore best understood as a cross-cutting semantic property of observations and derived claims, not automatically as a runtime primitive.

A central distinction remains:

```text
model uncertainty != evidence strength != probability of correctness
```

Observability is a sensor/substrate. It becomes control-relevant only when evidence is interpreted and evaluated for the decision at hand.

### 4.5 Learning and adaptation

Several apparently different mechanisms share a lifecycle:

```text
observe
  -> extract
  -> validate
  -> promote
  -> reuse
  -> evaluate
  -> update / invalidate
```

This appears in:

- episodic/behavioral learning;
- Learning Records and Skills;
- knowledge maintenance;
- deterministic procedure promotion;
- routing/policy improvement;
- derived representation maintenance;
- computation reuse.

They are not identical mechanisms, but they share a controlled adaptation pattern: **a past observation becomes reusable state only after validation and with explicit scope/validity/provenance**.

---

## 5. Concept convergence matrix

| Observed idea | Surface form | Deeper mechanism | EOKS relationship | Status |
|---|---|---|---|---|
| Context cache | cached context | reusable information | Context / working set | established distinction |
| Inference/KV cache | inference optimization | reusable computation state | Execution/provider mechanism | distinct, related |
| Computed artifact | saved derived result | reusable computation output | Resource + provenance/dependencies | promising |
| Memory | stored experience/knowledge | persistent reusable state | Resource/representation + learning lifecycle | established, multi-form |
| Working set | active useful subset | locality-aware active state | Resource/context boundary | strong |
| Context miss | missing evidence | insufficiency signal for control | Context + reconciliation | promising |
| Graph | relational representation | navigation/structural view | Representation | established |
| Intermediate evidence | evidence during execution | process/result/model signal | Evaluation/control input | strong |
| Trajectory | execution trace | process evidence | Run/Evaluation | strong |
| OTel/telemetry | operational trace | execution sensor | Observation substrate | complementary |
| Logprobs/entropy | model signal | model-native uncertainty evidence | Evaluation input | research |
| Hidden states | internal activation | model-native intermediate representation | Provider-specific evidence source | research |
| LLM judge | semantic evaluator | derived evidence | Evaluation provider | research/calibration required |
| Tool selection | provider routing | minimum sufficient capability | Conductor/Policy | strong |
| Verification | tests/review/analyzers | uncertainty reduction | Evaluation/resource | complementary |
| Deterministic promotion | compile reasoning into procedure | reuse + assurance | Learning + execution modality | promising |
| Knowledge maintenance | update derived knowledge | dependency-aware reconciliation | Knowledge/control | strong |
| Session learning | trace -> skill | validated behavioral adaptation | Learning | strong |
| Latent/iterative reasoning | longer internal computation | adaptive computation budget | Execution/Computation hypothesis | research |
| Chain-of-Draft | compact reasoning format | intervention on computation/token budget | Reasoning strategy | workload-dependent |
| Model routing | switch model | resource selection under uncertainty | Conductor/Policy | strong |
| Graduated autonomy | increase delegation | assurance-driven control | Outcome/evaluation/policy | strong |

The matrix is deliberately not a taxonomy of primitives. Its purpose is to reveal convergence without erasing mechanism differences.

---

## 6. Same, different representation, complementary, or distinct?

### Context cache vs inference KV cache vs computed artifact vs memory

These all support reuse, but at different layers:

```text
memory                 -> reusable semantic/persistent state
context cache          -> reusable task information/materialization
inference KV cache     -> reusable model inference state
computed artifact      -> reusable derived computation/result
```

The common property is **reuse under validity, scope and cost constraints**. The mechanisms should remain separate until implementation evidence shows that a common protocol is useful.

### Trajectory vs intermediate evidence vs execution trace vs observability

These overlap but answer different questions:

- **trajectory**: what actions/steps the agent took;
- **intermediate evidence**: what evidence became available before the final outcome;
- **execution trace**: a recorded temporal account of execution;
- **observability**: instrumentation and telemetry exposing system behavior.

A trace can contain trajectory and intermediate evidence; OTel can transport execution observations; none is automatically synonymous with evaluation.

### Retrieve vs verify vs sample vs switch model vs analyzer vs human

These are different actions that can serve a common control objective:

> reduce uncertainty or close an evidence gap at acceptable cost/risk.

They should therefore be compared as **alternative interventions selected by the conductor**, not collapsed into one mechanism.

### Observe / validate / promote / reuse / invalidate

These are lifecycle stages rather than necessarily separate concepts. The same lifecycle appears in knowledge, memory, computation and deterministic procedure promotion.

---

## 7. The computation/reuse convergence

The recent computation work provides a particularly important bridge:

```text
probabilistic / latent computation
              |
              v
       intermediate state
              |
              v
       intermediate evidence
              |
              v
       validated derived artifact
              |
              v
      reusable computation/state
              |
       dependency changes
          /         \
      still valid   stale
          |           |
        reuse     invalidate/recompute
```

This connects the work on latent reasoning, intermediate evidence, computed artifacts, context caches, deterministic promotion and knowledge maintenance.

The deeper hypothesis is not that all of these should become one `Computation` object. It is that **reuse may be a cross-cutting control problem over derived state and computation**.

This should be tested with simulations before changing the ontology.

---

## 8. The uncertainty/evidence/control convergence

Another strong convergence is:

```text
uncertainty / risk
       |
       v
 evidence requirement
       |
       v
 candidate providers / actions
       |
       v
 capability + reliability + cost + latency
       |
       v
 minimum sufficient evidence/action
       |
       v
 evaluate
   /      \
sufficient  insufficient
   |             |
  stop      retrieve / verify / sample /
            analyzer / model switch / human
                  |
                  v
               reconcile
```

This unifies the work on reliability signals, tool capability selection, context acquisition, evaluation, routing and graph branching without requiring a universal confidence scalar.

---

## 9. The reasoning-to-procedure loop

A complementary convergence is:

```text
uncertain workload
       |
   probabilistic reasoning
       |
   repeated successful behavior
       |
   observed trajectory + outcome
       |
   candidate procedure / learning record
       |
   validation + dependency capture
       |
   deterministic capability
       |
   monitor / evaluate
       |
   drift or invalidation?
      /          \
    no            yes
    |              |
  reuse       return to reasoning
```

This is the common shape behind behavioral learning, deterministic promotion, knowledge maintenance and adaptive execution.

The important boundary is that **successful execution does not automatically create policy**. Promotion is a controlled state transition requiring evidence, scope and rollback/invalidation semantics.

---

## 10. Hidden states: what the synthesis actually claims

The hidden-state discussion should remain precise.

A model's internal representation can be:

```text
physically present
      != automatically API-observable
      != human-interpretable
      != stable across versions
      != reusable as computation
```

When an instrumentable runtime exposes activations, probes can potentially derive intermediate evidence from them. EOKS should treat that as a **provider/model-specific evidence source**, not as a universal architectural assumption.

This preserves the useful research direction without turning “hidden state” into an ontology object merely because it is technically interesting.

---

## 11. Concept provenance

For important concepts, future research should record a small provenance chain:

```text
concept
  -> original observation / motivation
  -> research references
  -> independent/convergent evidence
  -> related EOKS concepts
  -> contradicting evidence / limitations
  -> current hypothesis
  -> experiment needed
  -> architecture impact, if any
```

This makes it possible to understand *why* a concept exists and prevents a later consolidation from silently deleting the reasoning that justified it.

A concept should carry a status such as:

- **observed** — seen in implementation or workload behavior;
- **supported** — backed by multiple relevant sources or experiments;
- **hypothesis** — plausible but not established;
- **mixed** — evidence depends strongly on workload or assumptions;
- **rejected** — evidence argues against the proposed mechanism;
- **canonical** — adopted into the current architectural model.

“Canonical” should be a conclusion, not a synonym for “interesting.”

---

## 12. Validation matrix

Every deeper synthesis hypothesis should eventually be expressible as:

| EOKS claim | Observable prediction | Baseline | Intervention | Metrics | Failure modes | Result |
|---|---|---|---|---|---|---|
| reusable computation helps | repeated work decreases without quality loss | recompute | reuse with invalidation | quality, cost, latency, reuse rate | stale artifact, hidden dependency | open |
| intermediate evidence improves control | earlier evidence improves stop/verify choices | outcome-only | trajectory/intermediate signals | quality, false stops, cost | correlated signals | open |
| minimum sufficient evidence is better | weaker sufficient action matches stronger action at lower cost | strongest resource | capability selection | outcome, cost, latency | underpowered provider | open |
| learned procedures improve future runs | validated promotion reduces repeated reasoning | no promotion | controlled promotion | success, regressions, maintenance cost | accidental habit, drift | open |
| working-set control helps | less irrelevant context with equal/better outcomes | broad context | working-set/context policy | quality, tokens, latency, misses | over-pruning, thrashing | open |

The existing evidence-ledger and research-agenda practices remain the place to record actual results. This table is a synthesis scaffold, not an experimental result.

---

## 13. Conceptual simulations

Before promoting a cross-cutting hypothesis into architecture, use small simulations or thought experiments.

### A. Minimum sufficient action

Given an uncertain task, compare retrieval, deterministic verification, a stronger model, another sample, a specialized analyzer and human review. Measure whether the controller can reach the same acceptance criterion with lower expected cost/risk.

### B. Computation reuse

Model a computation with dependencies, save a derived artifact, change one dependency, and test whether only the affected computation is invalidated. Compare full recomputation with dependency-aware reuse.

### C. Evidence composition

Combine model-native uncertainty, retrieval, deterministic checks and trajectory evidence. Inject correlated or misleading signals and test whether a naive aggregate becomes overconfident. This directly tests the need to preserve provenance and evidence independence.

### D. Learning promotion

Generate successful traces, extract a candidate procedure, validate it, promote it, and evaluate it on a new workload distribution. Include regression and drift cases to test rollback/invalidation.

### E. Deterministic compilation

Compare repeated probabilistic reasoning with a validated deterministic procedure. Introduce a dependency or requirement change and measure whether the system detects drift and returns to reasoning instead of silently reusing stale behavior.

These simulations should be small enough to falsify an idea. They are not demonstrations designed to prove the architecture.

---

## 14. What this changes — and what it does not

### It changes

- EOKS now has an explicit place to map concepts across documents and research.
- New research should be evaluated against existing concepts before new terminology is introduced.
- Similar mechanisms can be compared without forcing them into one abstraction.
- Historical richness can be preserved as concept provenance rather than duplicated architecture prose.
- Cross-cutting hypotheses such as computation-as-resource and reusable derived state can be tested before promotion.

### It does not change

- the seven provisional runtime primitives;
- the reconciliation/control-loop architecture;
- the resource/context/working-set distinctions;
- the model-agnostic boundary;
- the exploratory status of research notes;
- the principle that evidence must be calibrated before driving consequential automatic control.

In particular, this document **does not add `Computation`, `Evidence`, `State`, `HiddenState`, or `Card` as runtime primitives**.

---

## 15. Preservation audit from the recent consolidation history

The synthesis should explicitly remember where earlier consolidation compressed useful material.

### Behavioral memory / workflow consolidation

The deleted behavioral-memory material contained useful dimensions that should remain represented in the research/synthesis layer: second-brain use cases; personal/project/debugging/session/organizational memory; answer-versus-trajectory distinction; hot-path versus background learning; reflection questions; concrete Skill examples; detailed Learning Records; learning-plane structure; human approval/deletion; model-change effects; accidental habits; and contradictory procedures.

The current memory lifecycle remains valid; the richer dimensions should not be lost simply because the canonical model is smaller.

### Tool-selection consolidation

The capability-selection model is strong, but the deleted tool-selection material also contained scenario-oriented mappings: repository understanding; graph versus search; proactive versus reactive context; invariant verification; memory; deterministic workflows; adaptive planning; review; end-to-end benchmarks; and organizational context.

These should remain available as scenario evidence rather than forcing them back into a second normative tool-selection document.

### Control-plane consolidation

The canonical conductor/control responsibility is clearer than a standalone control-plane document, but the richer material around workload state, temporal/concurrent activities, multi-agent state, policy, model selection and topology remains useful as research context.

### Chain-of-Draft

The final CoD treatment is intentionally compact: reasoning-format changes belong on the intervention/reasoning-strategy axis, and token savings are workload-dependent. The richer conceptual point is preserved by the broader computation and adaptive-reasoning synthesis: **changing reasoning format is one intervention on computation, not evidence that a new EOKS primitive is needed**.

### Recent additions

The work on operational evidence, evaluator validity, latent/iterative computation, computed artifacts, trajectory evaluation and intermediate/model-native evidence should be retained as a connected research progression. In particular:

```text
latent computation
  -> intermediate evidence
  -> validated derived artifact
  -> reusable computation
```

is now a first-class synthesis hypothesis.

---

## 16. How to avoid repeating the consolidation mistake

The project can become richer without becoming structurally chaotic by changing the research-to-architecture workflow:

1. **Every new idea gets a relationship check.** Before adding or deleting a concept, identify nearby concepts and classify the relationship: same, representation, complementary mechanism, or distinct.
2. **Preserve provenance when consolidating.** If material is removed from a canonical document, record where its useful ideas went and why. A consolidation should be reversible at the conceptual level even when the old file is deleted.
3. **Keep three layers explicit.** `docs/` defines the current model; `research/` preserves exploration and competing evidence; the synthesis layer explains convergence and unresolved boundaries.
4. **End research PRs with integration.** Each research addition should state: relevant existing concepts, relationship to them, evidence status, contradictions, and possible architecture impact.
5. **Promote claims, not vocabulary.** A new name is not evidence for a new abstraction. Promotion should require implementation traces, experiments, or strong independent evidence showing that the distinction matters.
6. **Use scenario matrices and simulations to preserve richness.** When many mechanisms serve the same goal, keep the mechanisms and compare them under concrete workloads instead of collapsing them into one generic term.
7. **Run a deletion audit.** Before removing a document, enumerate its concepts, examples, scenarios, hypotheses and references; verify each has a destination or is explicitly rejected.
8. **Treat the repository as an epistemic control loop.** `hypothesis -> research -> evidence -> synthesis -> confirm/refine/split/reject -> architecture` should be the normal lifecycle.

The key rule is simple:

> **Do not ask only “where does this concept fit?” Ask “what relationship does it have to the concepts already here, what evidence supports that relationship, and what would falsify it?”**

---

## 17. Open synthesis questions

These remain intentionally unresolved:

1. Is computation eventually a first-class resource/capability model, or is Run + Provider + Outcome sufficient?
2. Is evidence best represented as a semantic type/property over observations, or does some evidence need independent lifecycle/identity?
3. Can context cache, computed artifacts, memory and inference cache share a useful abstract reuse protocol without hiding important layer differences?
4. Which state domains require independent durable identity for reconstruction and reconciliation?
5. How much model-native evidence is actually useful compared with deterministic external validation?
6. Can learned procedures be promoted safely enough to justify a more explicit learning plane?
7. Does `Decision` require independent identity/lifecycle, or can run/event state eventually represent it?
8. Which cross-cutting concepts survive adversarial simulations across different workloads?

Until those questions have stronger evidence, the conservative seven-primitive architecture remains appropriate.

---

## 18. Working rule for future PRs

A useful final template for future research/consolidation PRs is:

```text
NEW IDEA
  |
  +--> what does it represent?
  |
  +--> related existing concepts?
  |
  +--> same / representation / complementary / distinct?
  |
  +--> supporting + contradicting evidence?
  |
  +--> scenario / simulation that could distinguish them?
  |
  +--> synthesis entry + provenance
  |
  +--> architecture impact only if evidence warrants it
```

This lets EOKS accumulate knowledge without requiring the canonical architecture to grow at the same rate as the research corpus.