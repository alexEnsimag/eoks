# EOKS domain model and architecture diagrams

This document is the visual companion to [EOKS architecture](architecture.md).

The diagrams deliberately use different zoom levels. They are not competing architectures:

1. **System map** — where EOKS sits relative to ADEs, agents, models and execution substrates.
2. **Six dimensions** — how the existing conceptual vocabulary maps onto the runtime model without turning each dimension into a subsystem.
3. **Core domain model** — the durable relationships between Task, Run, Context, Decision, Policy, Evaluation and Outcome.
4. **Run zoom-in** — what a Run binds together: context, policy, workflow state, resources, workspace, execution and evidence.
5. **Context/resource zoom-in** — how governed resources become a task-specific context.
6. **Control loop** — how desired state, decisions, execution, evidence and evaluation reconcile.
7. **Learning and maintenance loop** — how outcomes can improve resources and policy without silently rewriting canonical knowledge.

## 1. System map

EOKS is not the ADE/IDE, agent runtime, model provider or sandbox. Those are replaceable surfaces/resources that EOKS can coordinate.

~~~mermaid
flowchart TB
    H["Human / developer"]
    ADE["ADE / IDE / CLI / API<br/>work surface"]

    subgraph E["EOKS — workload control & resource coordination"]
        I["Intent / Task"]
        C["Conductor / reconciliation"]
        R["Resources"]
        CTX["Context compilation"]
        RUN["Runs / execution state"]
        EV["Evaluation / evidence"]
        OUT["Outcome"]
        I --> C
        C --> R
        C --> CTX
        C --> RUN
        RUN --> EV
        EV --> OUT
        OUT --> C
    end

    subgraph X["Replaceable execution & evidence resources"]
        AG["Agents / sessions"]
        M["Models"]
        T["Tools / deterministic capabilities"]
        W["Workspace / execution environment"]
        K["Knowledge / memory / representations"]
        P["Evidence providers / analyzers / tests"]
    end

    H --> ADE
    ADE --> E
    E --> AG
    E --> M
    E --> T
    E --> W
    E --> K
    E --> P
~~~

**Boundary:** EOKS coordinates these things; it does not need to own their implementation.

## 2. Six dimensions mapped to the runtime model

The six dimensions remain useful as architectural lenses, not six separate services.

~~~mermaid
flowchart LR
    INT["INTENT<br/>What outcome is wanted?"] --> TASK["Task"]
    WF["WORKFLOW<br/>How does work progress?"] --> WORK["Workflow / roles"]
    CAP["CAPABILITIES<br/>What can act or provide evidence?"] --> RES["Resources / providers"]
    KNOW["KNOWLEDGE<br/>What durable information exists?"] --> ASSET["Assets / representations"]
    STATE["STATE<br/>What has happened / is true now?"] --> RUN["Run / execution state"]
    POL["POLICY<br/>What is allowed / required?"] --> POLICY["Policy"]

    TASK --> RUN
    WORK --> RUN
    RES --> RUN
    ASSET --> RUN
    POLICY --> RUN
~~~

This preserves the useful six-dimensional vocabulary while keeping the runtime semantic core small.

## 3. Core domain model — UML

This is the most important diagram. It makes ownership and cardinality explicit.

~~~mermaid
classDiagram
    direction TB

    class Task {
        <<runtime primitive>>
        +id
        +objective
        +constraints
        +acceptanceCriteria
        +status
    }

    class Run {
        <<runtime primitive>>
        +id
        +attempt
        +status
        +startedAt
        +endedAt
    }

    class Context {
        <<runtime primitive>>
        +revision
        +provenance
        +selectionRationale
        +budget
    }

    class Decision {
        <<runtime primitive>>
        +id
        +kind
        +rationale
        +authority
    }

    class Policy {
        <<runtime primitive>>
        +id
        +revision
        +constraints
        +requiredEvidence
    }

    class Evaluation {
        <<runtime primitive>>
        +id
        +kind
        +result
        +assurance
    }

    class Outcome {
        <<runtime primitive>>
        +status
        +artifacts
        +verification
        +delayedResults
    }

    class Workflow {
        <<execution structure>>
        +steps
        +dependencies
        +handoffs
    }

    class Resource {
        <<resource>>
        +id
        +kind
        +availability
    }

    class Asset {
        <<governed resource>>
        +revision
        +provenance
        +scope
        +freshness
        +authority
    }

    class Representation {
        <<resource representation>>
        +kind
        +sourceRevision
    }

    class Provider {
        <<resource mechanism>>
        +capabilities
        +cost
        +latency
    }

    class Workspace {
        <<resource>>
        +identity
        +desiredState
        +observedState
    }

    class Agent {
        <<resource>>
        +runtime
        +session
    }

    class Model {
        <<resource>>
        +provider
        +version
    }

    class Tool {
        <<resource>>
        +capability
        +sideEffects
    }

    class Evidence {
        <<supporting record>>
        +source
        +provenance
        +revision
        +authority
    }

    class Artifact {
        <<supporting record>>
        +kind
        +revision
        +location
    }

    Task "1" *-- "1..*" Run : has attempts
    Task "1" --> "0..*" Workflow : uses
    Task "1" --> "0..*" Policy : governed by

    Run "1" --> "0..*" Context : uses
    Run "1" --> "0..*" Decision : records
    Run "1" --> "0..*" Evaluation : evaluated by
    Run "1" --> "0..1" Outcome : produces
    Run "1" --> "1" Workflow : executes
    Run "1" --> "1..*" Policy : operates under

    Run "1" --> "0..*" Resource : uses
    Run "1" --> "0..*" Evidence : observes
    Run "1" --> "0..*" Artifact : produces

    Context "0..*" --> "0..*" Asset : selects
    Context "0..*" --> "0..*" Representation : materializes from
    Context "0..*" --> "0..*" Evidence : includes

    Asset --|> Resource
    Representation --|> Resource
    Provider --|> Resource
    Workspace --|> Resource
    Agent --|> Resource
    Model --|> Resource
    Tool --|> Resource

    Provider --> Evidence : produces
    Tool --> Evidence : produces
    Workspace --> Artifact : contains
~~~

### The key relationship

The central lifecycle is:

**Task → Run → Outcome**

A Task is the durable identity of the work. A Run is one execution attempt. Therefore:

- one Task can have many Runs;
- Runs can fail, be retried, resumed or superseded;
- each Run has its own context/policy/resource configuration;
- the final Outcome is about the work, but is supported by evidence accumulated across Runs.

Context is deliberately still one of the seven primitives. A **context snapshot** is a versioned/reconstructable instance of Context for a particular reasoning step; it does not need to become an eighth primitive.

## 4. Run zoom-in

A Run is where the otherwise separate EOKS concerns meet.

~~~mermaid
flowchart TB
    TASK["Task<br/>objective + acceptance criteria"]
    POLICY["Policy<br/>constraints + required assurance"]
    LOAD["Loadout<br/>eligible resources"]
    WF["Workflow<br/>steps + roles"]

    RUN["RUN<br/>one execution attempt"]

    CTX["Context<br/>versioned materialization"]
    WS["Workspace / Environment<br/>desired + observed state"]
    EXEC["Execution resources<br/>agent · model · tools"]
    STATE["Execution state<br/>events · checkpoints · progress"]
    ART["Artifacts<br/>changes · outputs · generated files"]
    EVID["Evidence<br/>tests · review · observations"]
    EVAL["Evaluation<br/>quality / assurance"]

    TASK --> RUN
    POLICY --> RUN
    LOAD --> RUN
    WF --> RUN

    RUN --> CTX
    RUN --> WS
    RUN --> EXEC
    RUN --> STATE
    RUN --> ART
    RUN --> EVID
    EVID --> EVAL
    ART --> EVAL
    EVAL --> RUN
~~~

The important distinction is:

~~~text
Run identity
    ≠
agent/provider session identity
    ≠
execution environment identity
~~~

A provider session, container or VM can disappear and be replaced while the logical Run remains the same.

## 5. Context and resource zoom-in

This is the resource-management side of EOKS.

~~~mermaid
flowchart LR
    U["Resource universe"]

    G["Governance<br/>scope · access · freshness · authority"]
    L["Loadout<br/>workload eligibility"]
    WS["Working set<br/>currently useful resources/evidence"]
    ACQ["Context acquisition<br/>retrieve · query · analyze"]
    CC["Context compilation<br/>select · rank · transform<br/>order · compress · budget"]
    C["Context<br/>reasoning-step materialization"]
    R["Reason / act"]
    S["Execution state + evidence"]

    U --> G --> L --> WS
    WS --> ACQ --> CC --> C --> R
    R --> S
    S --> WS

    MISS["Context miss"]
    MISS -.-> ACQ
    THRASH["Context pressure / thrashing"]
    WS -.-> THRASH
    THRASH -.-> CC
~~~

The distinctions are intentional:

~~~text
Loadout    = what is eligible
Working set = what is currently useful
Context    = what is actually materialized for a reasoning step
~~~

A context snapshot should preserve enough identity/provenance to reconstruct **what was presented and from which resource revisions**, without requiring a giant copy of every underlying source.

## 6. Control-loop zoom-in

The control plane is best understood as reconciliation rather than as a collection of scheduler/router/orchestrator boxes.

~~~mermaid
flowchart TB
    D["Desired state<br/>Task objective + Policy"]
    O["Observed state<br/>Run state + Workspace + Evidence"]

    C["CONDUCTOR<br/>reconciliation responsibility"]

    DEC["Decision<br/>next action / resource / modality"]
    ACT["Action<br/>retrieve · plan · execute · verify · retry · stop · escalate"]
    RUN["Run / execution"]
    OBS["Observation<br/>events · artifacts · tool results"]
    EVAL["Evaluation<br/>is evidence sufficient?"]

    D --> C
    O --> C
    C --> DEC
    DEC --> ACT
    ACT --> RUN
    RUN --> OBS
    OBS --> EVAL
    EVAL --> O
    O --> C

    STOP["Acceptance satisfied"]
    ESC["Escalation / human gate"]
    EVAL --> STOP
    EVAL --> ESC
~~~

The same pattern works for context selection, resource/provider selection, execution, verification, retry/repair, model selection and workspace provisioning.

The **conductor is a responsibility**, not necessarily a separate agent or service.

## 7. Evaluation, learning and maintenance

The fast execution loop and slower improvement loop should be distinguished.

~~~mermaid
flowchart LR
    RUN["Run"]
    OBS["Observations"]
    OUT["Outcome"]
    EVAL["Evaluation"]
    DEC["Control decision"]

    RUN --> OBS --> OUT --> EVAL --> DEC
    DEC --> RUN

    EVAL --> CAND["Candidate learning<br/>pattern · memory · skill · representation · policy change"]
    CAND --> VALID["Validate / calibrate"]
    VALID --> PROM["Promote / update / supersede"]
    PROM --> RES["Durable resources / policy"]
    RES --> RUN
~~~

Promotion is deliberately explicit. A run should not silently rewrite canonical knowledge or policy merely because an agent produced a plausible suggestion.

This gives EOKS three useful timescales:

~~~text
step loop       act → observe → evaluate
work loop       reconcile until acceptance / escalation
system loop     aggregate outcomes → evaluate interventions → update resources/policy
~~~

These are control-loop views, not three separate EOKS subsystems.

## 8. Evidence and authority

A final architectural zoom-in is useful because it explains why EOKS does not equate agent completion with success.

~~~mermaid
flowchart TB
    CLAIM["Agent claim / proposed result"]

    E1["Deterministic checks"]
    E2["Tests / static analysis"]
    E3["Independent review"]
    E4["Runtime observations"]
    E5["Model / agent assessment"]
    E6["Human judgment"]

    EV["Evidence set<br/>with provenance + authority"]
    EVAL["Evaluation"]
    POL["Policy"]
    DEC["Decision"]
    OUT["Authorized outcome"]

    CLAIM --> EV
    E1 --> EV
    E2 --> EV
    E3 --> EV
    E4 --> EV
    E5 --> EV
    E6 --> EV

    EV --> EVAL --> POL --> DEC --> OUT
~~~

The principle is:

**observation → evidence → evaluation → policy/decision → authorized state transition**

rather than:

**agent says done → system accepts**.

## 9. What is canonical

The diagrams imply a deliberately small semantic core.

### Runtime primitives

- **Task** — durable bounded work.
- **Context** — task/step-specific compiled information.
- **Run** — one execution attempt.
- **Decision** — a control-plane choice.
- **Policy** — constraints/requirements.
- **Evaluation** — measurement/assurance.
- **Outcome** — what happened.

### Important surrounding concepts

These are architectural concepts, but not automatically additional runtime primitives:

- Workflow
- Role
- Resource
- Asset
- Provider
- Representation
- Loadout
- Working set
- Workspace / execution environment
- Agent
- Model
- Tool
- Evidence
- Artifact
- Execution state
- Conductor

This distinction is important. EOKS should only promote one of these to a new primitive when implementation/evaluation evidence shows that it needs an independent identity, lifecycle and semantics.

## 10. The compact mental model

When the whole architecture needs to fit in one picture:

~~~text
                 INTENT
                   │
                   ▼
                 TASK
                   │
          ┌────────┴────────┐
          │                 │
       POLICY           WORKFLOW
          │                 │
          └────────┬────────┘
                   ▼
                 RUN
          ┌────────┼─────────┐
          │        │         │
       CONTEXT  RESOURCES  WORKSPACE
          │        │         │
          └────────┼─────────┘
                   ▼
              EXECUTION
                   │
          observations / artifacts
                   │
                   ▼
                EVIDENCE
                   │
                   ▼
              EVALUATION
                   │
                   ▼
                OUTCOME
                   │
                   ▼
             RECONCILIATION
                   │
                   └──────────► next Run / stop / escalate

          slower feedback:
       outcomes → learning → governed resource/policy updates
~~~

This is the **canonical relationship view**. The other diagrams are zoom-ins of it.

## Design rule

> **EOKS coordinates Work, Runs, resources, context, policy, decisions, evidence and outcomes through reconciliation; it does not need to own the agents, models, tools, knowledge stores or execution substrates it coordinates.**
