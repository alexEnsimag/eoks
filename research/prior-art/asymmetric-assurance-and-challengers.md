# Asymmetric assurance and challenger evidence

## Scope

This note investigates a specific reliability pattern emerging from the EOKS synthesis: a **support** path constructs an artifact or claim, while an **attack** path tries to falsify, invalidate or expose omissions in it. The goal is not to introduce a `Challenger` primitive. The question is whether asymmetric evidence generation should become an explicit assurance pattern over existing EOKS Tasks, Runs, Decisions, Policies, Evaluations and Outcomes.

The key distinction is between:

- **reflection** — inspect and improve a trajectory or artifact;
- **support** — construct evidence for a claim or artifact;
- **attack** — deliberately seek evidence that would make the claim or artifact fail;
- **verification** — use a mechanism capable of establishing a particular property;
- **evaluation** — decide whether the available evidence is sufficient for the intended decision.

## 1. Main finding

The literature does not support a simple rule that adding more critics, judges or agents makes a system more reliable.

A stronger conclusion is:

> **Assurance comes from useful evidence-producing paths whose failure modes are sufficiently different, followed by mechanisms capable of adjudicating the evidence.**

This has three consequences for EOKS.

First, a challenger should have an **asymmetric objective**. Asking another worker to solve the same problem again is not equivalent to asking it to find ways the current artifact could be wrong.

Second, nominal agent count is a poor assurance measure. Multiple agents using the same model, context, evidence and assumptions may make correlated errors.

Third, attacks should ideally terminate in **checkable evidence**: a counterexample, contradiction, failed test, violated invariant, missing requirement, authoritative conflict, or other observable reason for rejecting or revising the artifact. Negative prose without an adjudication path is weaker evidence.

## 2. Self-correction is not a sufficient model

Kamoi et al. survey self-correction research and find that prompted self-correction does not reliably work in general; the stronger positive cases are tasks with reliable external feedback. They also emphasize careful experimental design because self-correction evaluations can overstate benefits.

Source: Kamoi et al., *When Can LLMs Actually Correct Their Own Mistakes? A Critical Survey of Self-Correction of LLMs*, TACL 2024.
https://aclanthology.org/2024.tacl-1.78/

Tyen et al. isolate an important failure mode: LLMs often struggle to **find** reasoning mistakes even when they can correct a mistake once its location is provided. This separates error localization from error repair.

Source: Tyen et al., *LLMs cannot find reasoning errors, but can correct them given the error location*, Findings of ACL 2024.
https://aclanthology.org/2024.findings-acl.826/

Yang et al. further decompose self-correction into confidence and critique capability, finding that prompting changes can improve one while degrading the other. This is another reason not to treat a model's willingness to revise as evidence that it found a real defect.

Source: Yang et al., *Confidence v.s. Critique: A Decomposition of Self-Correction Capability for LLMs*, ACL 2025.
https://aclanthology.org/2025.acl-long.203/

**EOKS implication:** reflection is useful as a control mechanism, but a reflection step should not automatically be treated as independent assurance.

## 3. External or structured verification changes the picture

Several lines of work improve correction by introducing an explicit verification mechanism rather than asking the model to judge itself in an unconstrained way.

Wu et al. construct a verify-then-correct process around explicit key-condition verification and report improvements over a self-correction baseline on several reasoning datasets.

Source: Wu et al., *Large Language Models Can Self-Correct with Key Condition Verification*, EMNLP 2024.
https://aclanthology.org/2024.emnlp-main.714/

Zhang et al. use a divide-verify-refine pattern for complex instructions, explicitly relying on tools that can rigorously check individual constraints and provide feedback.

Source: Zhang et al., *Divide-Verify-Refine: Can LLMs Self-align with Complex Instructions?*, Findings of ACL 2025.
https://aclanthology.org/2025.findings-acl.709/

Song et al. similarly use program-driven verification and refinement, with executable verification logic supplying feedback to refinement.

Source: Song et al., *ProgCo: Program Helps Self-Correction of Large Language Models*, ACL 2025.
https://aclanthology.org/2025.acl-short.73/

These results support a useful hierarchy:

```text
unconstrained self-critique
        <
structured critique
        <
external / executable verification
```

This is not a universal performance ordering, but it is a useful assurance hypothesis: **the more directly a feedback mechanism can establish or falsify a property, the less it depends on the generator's own judgment.**

## 4. Attack should seek falsification, not disagreement

A weak challenger is simply an opposing opinion:

```text
worker:    "This design is correct."
challenger:"I disagree."
```

A stronger challenger attempts to produce an attack:

```text
claim
  -> identify assumption/property
  -> construct failure condition
  -> produce counterexample or contradictory evidence
  -> make attack checkable
```

Recent work on LLM-generated counterexamples provides useful evidence for this formulation. Balestra et al. use LLMs to generate tests intended specifically to invalidate inferred program assertions; their experiments report that generated counterexamples can discard invalid assertions and improve specification precision.

Source: Balestra et al., *Improving Dynamic Specification Inference with LLM-Generated Counterexamples*, 2026.
https://arxiv.org/abs/2604.10761

Related property-based testing work explicitly positions properties and invariants as a way to avoid a self-reinforcing cycle in which generated tests merely reproduce assumptions of generated code. One proposed architecture separates code generation from property-based testing and uses property violations as feedback.

Source: He et al., *Use Property-Based Testing to Bridge LLM Code Generation and Validation*, 2025.
https://arxiv.org/abs/2506.18315

**EOKS implication:** the preferred output of an attack is not “critique text”; it is **attack evidence** that can enter the same evidence/evaluation machinery as supporting evidence.

## 5. Correlated failure is the critical limitation

Independent-looking reviewers are not necessarily independent.

Recent work studying LLM judge ensembles reports substantial correlation between judges' errors. The practical lesson is that adding nominally separate judges can provide far less independent evidence than their count suggests.

This connects directly to EOKS's existing emphasis on provenance and evidence lineage: an assurance record should preserve not only *what* evidence was produced, but enough metadata to reason about likely dependence between evidence paths.

Potential dependence factors include:

- same model family or model checkpoint;
- same prompt or attack strategy;
- same context and source material;
- same retrieved evidence;
- same generated artifact or reasoning trajectory;
- same deterministic checker;
- same assumptions or acceptance criteria;
- common upstream failures.

Therefore:

```text
3 agents
    !=
3 independent evidence paths
```

The relevant quantity is closer to **effective assurance independence** than raw reviewer count.

This should remain a research hypothesis rather than a new EOKS primitive until measured in the EOKS evaluation agenda.

## 6. Debate is not automatically assurance

Debate-style architectures provide an important negative result. Models can remain highly confident while disagreeing, including cases where both sides assign themselves high probabilities of winning. This indicates that adversarial dialogue can amplify confidence without producing reliable calibration.

Source: Nguyen and Prasad, *Two LLMs debate, both are certain they've won*, 2025.
https://arxiv.org/abs/2505.19184

The EOKS distinction is therefore:

```text
opposition
    !=
attack
    !=
verification
```

A useful attack topology needs a downstream mechanism that can adjudicate whether an alleged defect is real.

## 7. Worker, challenger and verifier have different objectives

A useful role separation is:

| Role | Primary objective | Typical output |
|---|---|---|
| Worker / support | Construct the best artifact or claim | artifact + supporting evidence |
| Challenger / attack | Find ways the artifact could fail | attack + counterexample / contradiction / missing requirement |
| Verifier | Establish particular properties | test result, proof, authoritative check, analyzer result |
| Evaluator | Decide whether evidence is sufficient | evaluation / decision recommendation |

The roles are not necessarily different agents. A single model can perform more than one role, but **reusing the same model or trajectory weakens the independence claim**.

A verifier can be deterministic. A challenger is often probabilistic. An evaluator may combine both.

## 8. What a challenger should attack

A practical attack protocol can cover several dimensions without turning them into EOKS primitives:

1. **Requirement attack** — identify missing, ambiguous or contradictory requirements.
2. **Assumption attack** — find assumptions that are unjustified or likely to fail.
3. **Dependency attack** — expose hidden coupling, unavailable capabilities or ordering assumptions.
4. **Counterexample attack** — construct a concrete input, state or scenario that violates a claim.
5. **Boundary attack** — test edge cases, scale, concurrency, failure and unusual states.
6. **Evidence attack** — question stale, weak, circular or non-independent evidence.
7. **Verification attack** — ask whether the stated checks actually establish the desired property.
8. **Operational attack** — expose deployment, observability, recovery, security or cost failures.

This taxonomy is a protocol aid, not a new ontology.

## 9. Information boundary matters

A challenger should normally receive the artifact, requirements and relevant evidence needed to attack it, but need not receive the worker's private reasoning trajectory.

The reason is not that hidden reasoning is intrinsically better. It is that exposing the worker's reasoning can create an **anchoring dependency**: the challenger may inherit the same decomposition and assumptions instead of independently searching for failures.

A useful experimental variable is therefore:

```text
challenger sees artifact only
          vs
challenger sees artifact + worker rationale
```

The EOKS evaluation agenda should measure whether withholding the worker trajectory increases unique defect discovery without increasing false positives.

## 10. Assurance protocol

The strongest current candidate protocol is:

```text
1. Construct
   worker produces artifact + claims + assumptions + acceptance criteria

2. Attack
   challenger attempts to falsify or invalidate the artifact

3. Substantiate
   attack findings are converted into checkable claims/counterexamples

4. Verify
   deterministic, authoritative or independent mechanisms test the claims

5. Evaluate
   combine support, attack and verification evidence

6. Decide
   accept / revise / reject / escalate

7. Preserve
   retain assurance evidence and provenance with the resulting artifact
```

For a revision, the changed claims should be identifiable so that re-assurance can be incremental where safe rather than forcing a full restart.

## 11. Assurance topology

The execution-graph synthesis in EOKS can represent this without introducing a new graph primitive:

```text
                         artifact
                            |
                    +-------+-------+
                    |               |
                 SUPPORT          ATTACK
                    |               |
                    +-------+-------+
                            |
                       substantiation
                            |
                       verification
                            |
                        evaluation
                            |
                  +---------+---------+
                  |         |         |
                accept    revise    reject
```

The conductor can select an assurance topology according to policy, consequence/risk, historical performance and available verification mechanisms.

The important design rule is:

> **Use the minimum assurance topology sufficient for the consequence of the decision.**

A low-consequence task may need only support plus deterministic checks. A high-consequence design may justify an independent attack path and stronger verification. The exact mapping is policy-dependent and must be validated empirically.

## 12. Metrics and falsifiable hypotheses

Candidate metrics include:

- **Attack yield:** valid defects discovered per challenge execution.
- **Unique attack yield:** defects discovered by the attack path that the support path did not discover.
- **Attack precision:** validated defects / reported attacks.
- **Challenge value:** severity-weighted validated defects / challenge cost.
- **False reassurance:** accepted artifacts later found defective.
- **Assurance independence:** effective independent evidence relative to nominal evidence paths.
- **Verification coverage:** relevant claims for which an independent verification mechanism exists.
- **Revision efficiency:** validated defects removed per revision/re-assurance cycle.
- **Escaped defect rate:** defects surviving the configured assurance topology.

These are **candidate EOKS metrics**, not established industry standards.

The most important experiment is a controlled topology comparison:

```text
A: worker
B: worker + verifier
C: worker + challenger
D: worker + challenger + verifier
E: worker + independent challenger + verifier
```

Use blind evaluation after acceptance and measure defect discovery, unique defects, false positives, escaped defects, cost, latency and revision count.

Vary independence deliberately:

```text
same model / same context
same model / different context
same model / different attack strategy
other model / same evidence
other model / different evidence
LLM attack / deterministic verification
```

This would test whether EOKS gains come from **asymmetry**, **independence**, **verification**, or merely additional compute.

## 13. Relationship to existing EOKS concepts

This research strengthens rather than expands the EOKS ontology.

### Task
Defines the workload and desired result that assurance applies to.

### Policy
Determines the required assurance level/topology based on consequence, risk, authority and available evidence.

### Conductor
Selects and coordinates support, attack and verification runs.

### Context
Provides the working set for each run. Challenger context may intentionally differ from worker context to reduce correlated failure.

### Run
Executes worker, challenger or verifier work. Role is an execution objective, not a new primitive.

### Evidence
Support and attack produce evidence with provenance. Verification produces evidence about specific properties.

### Evaluation
Adjudicates evidence and determines whether the assurance requirement has been met.

### Decision
Accepts, revises, rejects or escalates the artifact/state transition.

### Outcome
Provides the eventual ground truth against which the assurance topology can itself be evaluated.

This is important: **the assurance mechanism is itself subject to evaluation.** A challenger that rarely finds real defects, or a verifier that creates false reassurance, should not become trusted merely because it exists.

## 14. Relationship to reflection, graph engineering and agentic evaluation

```text
reflection
  = inspect/change a local trajectory

support / attack
  = generate asymmetric evidence about an artifact or claim

verification
  = establish particular properties

graph engineering
  = coordinate these activities and their dependencies

agentic evaluation
  = measure whether the resulting process/outcome works

synthesis
  = generalize from reduced evidence and trajectories
```

This gives the concepts different jobs instead of collapsing them into a generic "multi-agent" abstraction.

## 15. Design constraints for EOKS

The current evidence supports the following constraints:

1. Do not introduce `Challenger`, `Verifier` or `Assurance` as foundational EOKS primitives.
2. Treat support and attack as evidence-generation objectives/roles over existing Runs.
3. Prefer attacks that produce checkable evidence over unconstrained negative commentary.
4. Do not equate agent count with assurance strength.
5. Preserve provenance and likely dependence between evidence paths.
6. Prefer independent context, strategy, evidence or mechanism where the consequence justifies it.
7. Prefer deterministic or authoritative verification where the property permits it.
8. Make unresolved attacks visible; do not hide missing or contradictory evidence.
9. Let Policy determine when additional assurance is worth its cost.
10. Evaluate the assurance topology against escaped defects and real outcomes, not only local agreement.

## Sources

- Kamoi, Zhang, Zhang, Han & Zhang. *When Can LLMs Actually Correct Their Own Mistakes? A Critical Survey of Self-Correction of LLMs*. TACL 2024. https://aclanthology.org/2024.tacl-1.78/
- Tyen, Mansoor, Carbune, Chen & Mak. *LLMs cannot find reasoning errors, but can correct them given the error location*. Findings of ACL 2024. https://aclanthology.org/2024.findings-acl.826/
- Wu, Zeng, Zhang, Tan, Shen & Jiang. *Large Language Models Can Self-Correct with Key Condition Verification*. EMNLP 2024. https://aclanthology.org/2024.emnlp-main.714/
- Zhang et al. *Self-Contrast: Better Reflection Through Inconsistent Solving Perspectives*. ACL 2024. https://aclanthology.org/2024.acl-long.197/
- Yang et al. *Confidence v.s. Critique: A Decomposition of Self-Correction Capability for LLMs*. ACL 2025. https://aclanthology.org/2025.acl-long.203/
- Zhang et al. *Divide-Verify-Refine: Can LLMs Self-align with Complex Instructions?* Findings of ACL 2025. https://aclanthology.org/2025.findings-acl.709/
- Song et al. *ProgCo: Program Helps Self-Correction of Large Language Models*. ACL 2025. https://aclanthology.org/2025.acl-short.73/
- Nguyen & Prasad. *Two LLMs debate, both are certain they've won*. 2025. https://arxiv.org/abs/2505.19184
- Balestra et al. *Improving Dynamic Specification Inference with LLM-Generated Counterexamples*. 2026. https://arxiv.org/abs/2604.10761
- He et al. *Use Property-Based Testing to Bridge LLM Code Generation and Validation*. 2025. https://arxiv.org/abs/2506.18315
- Haroon, Khan & Gulzar. *Evaluating LLM-Based Test Generation Under Software Evolution*. 2026. https://arxiv.org/abs/2603.23443
