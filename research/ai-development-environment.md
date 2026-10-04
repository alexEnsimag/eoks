# AI development environment

## Thesis

EOKS is not primarily an agent runtime or an AI-SDLC workflow engine. A better north star is a **programmable, observable and learning AI development environment**: a control layer that coordinates the capabilities and resources required to make agentic software engineering reliable.

The distinction matters:

- **AI-SDLC / AI-DLC** describes how the software-development lifecycle changes when AI performs more of the work.
- **An agent harness** provides the runtime scaffolding around a model: tools, instructions, permissions, hooks, context delivery, execution and feedback.
- **An AI development environment** is the broader environment in which those capabilities are composed, governed, observed, evaluated and improved.
- **EOKS** is hypothesized to sit at that environment/control layer, without replacing the underlying IDE, agent, harness or execution runtime.

This is a refinement of the existing EOKS model, not a proposal to add a new runtime primitive.

## Environment model

A useful decomposition is:

```text
                    AI DEVELOPMENT ENVIRONMENT
     +----------------------------------------------------+
     |                                                    |
     |  Intent / Working representation                   |
     |              |                                     |
     |  Context + Knowledge + Memory                      |
     |              |                                     |
     |  Policy / Permissions / Governance                 |
     |              |                                     |
     |  EXECUTION: local | remote | sandbox | fleet       |
     |              |                                     |
     |  Evaluation / Validation                            |
     |              |                                     |
     |  Observability / Evidence                          |
     |              |                                     |
     |  Learning / Adaptation                              |
     |              |                                     |
     |              +---------------------> improvement   |
     |                                                    |
     +----------------------------------------------------+
```

These are **environment capabilities**, not necessarily EOKS runtime primitives.

The existing EOKS runtime vocabulary remains intentionally small:

```text
Task -> Context -> Run -> Decision -> Evaluation -> Outcome
                                      |
                                      v
                              Memory / Policy update
```

The environment provides the resources, mechanisms and evidence around that loop.

## Tiles as an environment vocabulary

The word **tile** is useful as a compositional vocabulary for major environment capabilities:

> **A tile is a bounded capability of the AI development environment, with its own interface, state, policy, evidence and lifecycle.**

Possible tiles include:

- **Execution** — local, remote, sandboxed or fleet execution; checkpoint/resume; placement and recovery.
- **Context** — acquisition, compilation, delivery, inspection and budget management.
- **Knowledge / Memory** — durable project knowledge, experience and governed reusable procedures.
- **Policy** — permissions, guardrails, approval and autonomy constraints.
- **Evaluation** — tests, reviewers, deterministic analyzers and outcome measurement.
- **Observability** — traces, cost, latency, provenance and execution evidence.
- **Learning** — extraction of candidate improvements and controlled promotion into reusable behavior.
- **Representation** — plans, specifications, diagrams, code maps and other external working representations.
- **Human interaction** — objectives, approvals, corrections, exceptions and accountability.
- **Capabilities / Tools** — tools and external systems made available to a workload.

A tile is therefore **above** resources/assets/loadouts and **around** the execution runtime. It should not become another semantic or runtime primitive simply because a capability has a convenient name.

### Why the tile vocabulary is useful

It gives EOKS a way to reason about the environment compositionally:

1. What capabilities does a workload need?
2. Which provider/resource implements each capability?
3. What policy governs it?
4. What state does it maintain?
5. What evidence does it produce?
6. How is the capability evaluated?
7. How can the environment improve it over time?

This is compatible with the existing Resource/Asset/Provider/Loadout/Context model.

## Relationship to the existing EOKS model

The environment framing does **not** replace the six EOKS dimensions:

| EOKS dimension | Environment interpretation |
|---|---|
| INTENT | What the human/workload is trying to achieve |
| WORKFLOW | How work is organized and sequenced |
| CAPABILITIES | What the environment can provide |
| KNOWLEDGE | What the environment knows and can reuse |
| STATE | What persists across executions |
| POLICY | What is allowed, required or gated |

Likewise, tiles should not be confused with:

- **Resources** — reusable things that provide capabilities.
- **Assets** — governed lifecycle objects.
- **Loadouts** — workload-scoped eligibility.
- **Working set** — currently useful candidates.
- **Context** — the materialized information available to a reasoning step.
- **Agents** — execution loops.
- **Workflows** — explicit sequences/graphs of actions and decisions.
- **Runs** — individual attempts under a particular configuration.

A single resource can participate in several tiles, and a tile can be implemented by several resources.

## Execution is one tile, not the environment

This is an important correction to workflow-centric interpretations of EOKS.

A Claude Code session, deterministic process, workflow runtime, remote agent or agent fleet can all be an **execution**. Workflow runtimes are therefore one class of execution substrate rather than the definition of the environment.

The environment also decides what the execution can access, what context it receives, how it is observed, how its output is evaluated, and what is learned from the result.

This makes the control-loop view more natural than a workflow-engine view:

```text
intent
  -> select loadout/resources
  -> compile context
  -> execute
  -> observe
  -> evaluate
  -> produce outcome
  -> extract learning candidates
  -> govern/promote
  -> improve future runs
```

## Relationship to current industry terminology

Recent industry and research work increasingly uses adjacent concepts such as **agentic development environments**, **harness engineering**, **agentic engineering**, and **AI-DLC**.

The terminology is fragmented, but the convergence is useful:

- The **model alone is insufficient**; the surrounding environment strongly affects performance.
- Context, tools, permissions, feedback and execution are becoming explicit engineering concerns.
- Developer environments are evolving from editors with assistants toward persistent agent workspaces.
- Evaluation and feedback are needed to determine whether an environment intervention actually improves outcomes.
- Some research is beginning to treat the **environment itself as an object that can be optimized or learned**, rather than optimizing only the model.

EOKS should absorb these ideas without becoming tied to one vendor's definition of a harness or development environment.

## Architectural boundary

The strongest current boundary is:

```text
+----------------------------------------------------------+
|                 AI DEVELOPMENT ENVIRONMENT               |
|                                                          |
|  EOKS semantic/control layer                             |
|  intent · resource selection · context · policy         |
|  evaluation · evidence · learning · provenance           |
|                                                          |
+----------------------------------------------------------+
| replaceable infrastructure                               |
|                                                          |
| IDE/workspace · agent protocol · agent/harness runtime   |
| execution/sandbox · model · tools · code intelligence    |
+----------------------------------------------------------+
```

EOKS should **consume and coordinate** these lower layers rather than reimplement them.

In particular, this argues against making EOKS:

- another coding-agent loop;
- another terminal/workspace runtime;
- another generic session store;
- another editor/agent protocol;
- a mandatory graph database;
- a mandatory workflow engine;
- a monolithic memory system.

The environment abstraction is valuable precisely because it can compose these components.

## Research implications

The environment framing suggests several concrete research questions:

1. **Tile interfaces:** what minimum interface should a capability expose for state, policy, evidence and lifecycle?
2. **Tile composition:** how are multiple providers composed without introducing a second orchestration language?
3. **Loadout selection:** how should EOKS select the capabilities/resources needed for a workload?
4. **Evaluation:** how do we measure whether a tile or tile combination improved the end-to-end outcome?
5. **Learning:** what evidence is sufficient to change a capability's configuration, loadout or policy?
6. **Autonomy:** which environment capabilities should be allowed to adapt automatically, and which require human promotion?
7. **Interoperability:** which existing agent/harness protocols can serve as stable boundaries?
8. **Representation:** when should a problem be represented as text, code structure, graph, diagram, plan or another external working representation?

These should be answered experimentally rather than by expanding the EOKS ontology prematurely.

## Working conclusion

The useful synthesis is:

> **EOKS is a programmable, observable and learning AI development environment/control layer. Execution is one composable capability within that environment, not the environment itself.**

This gives the project a stronger north star than either "AI-SDLC" or "agent runtime":

- **AI-SDLC** is the lifecycle EOKS helps enable.
- **Agent harnesses** are execution infrastructure EOKS can integrate with.
- **Tiles** are a useful vocabulary for composing environment capabilities.
- **EOKS** coordinates those capabilities through a small semantic/control model and learns from outcomes.

This remains a research hypothesis. It should not become a new architectural constraint until experiments show that the environment/tile boundary improves implementation clarity, interoperability or measurable engineering outcomes.
