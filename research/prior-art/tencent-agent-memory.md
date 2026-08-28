# TencentDB Agent Memory

TencentDB Agent Memory is particularly relevant to EOKS because it treats agent continuity as more than transcript recall. Its current v2 architecture organizes reusable resources around memory, skills and knowledge, with a MemoryCore metadata/data service, a knowledge layer for Wiki and CodeGraph, and a Memory Proxy that adapts the system to coding-agent clients.

This note records the EOKS-relevant architecture and the boundaries that matter. It is prior art, not a proposal to make TencentDB Agent Memory an EOKS dependency.

## Why it matters

The project makes a useful distinction that is easy to miss when talking about "agent memory": there are several kinds of reusable state with different lifecycles and access patterns.

The current project describes four reusable resource families:

- **Chat Memory** — conversation-derived memory, refined from raw conversations into multiple levels of abstraction;
- **Skill** — reusable procedures extracted from successful work, with versions, resources, trigger boundaries, execution steps and validation rules;
- **LLM-Wiki** — structured documentation/knowledge pages with links and retrieval;
- **CodeGraph** — indexed code structure, symbols, relationships and impact paths.

These are better treated in EOKS as **different semantic resource types**, not collapsed into one memory primitive. EOKS uses the generic term **Asset** only for their shared lifecycle/governance boundary; see [Resource model](../../docs/resource-model.md).

A useful decomposition is therefore:

```text
                    reusable resources
                             |
          +------------------+------------------+
          |                  |                  |
     conversational       procedural         knowledge /
        memory             skills          representations
          |                  |             /           \
     L0-L3 memory      versioned SOPs     Wiki       CodeGraph
          |                  |               |            |
          +------------------+---------------+------------+
                             |
                      context compilation
                             |
                           Run
```

## Layered memory

TencentDB Agent Memory's Chat Memory is explicitly hierarchical:

```text
L0  conversation
      |
L1  atomic memory / facts
      |
L2  scenario memory
      |
L3  core profile / durable cognition
```

The important EOKS lesson is not the exact four-level schema. It is **multi-resolution memory**.

Raw conversations are useful for provenance and exact reconstruction, while higher-level memories are cheaper to recall for context bootstrapping. Retrieval can then drill down to lower-level evidence when the summary is insufficient.

This is a stronger model than treating memory as a flat vector collection or an ever-growing transcript.

EOKS should therefore distinguish at least:

```text
raw evidence
  -> extracted fact / event
  -> scenario/project knowledge
  -> durable pattern/profile
```

while retaining links back to the evidence that supports each abstraction.

## Short-term context is also hierarchical

Tencent's current Memory Proxy distinguishes different delivery modes. Short-term conversation memory can be written back each turn; session-level key memory can be recalled through tools; agent/team/global memory can be injected into the prompt; Skills can be surfaced as relevant summaries and exposed through skill tooling; Wiki and CodeGraph can be discovered and queried through knowledge tools.

This corrects an overly simple mental model of the system as "inject all four resources into the prompt every turn." **It is a hybrid architecture:** some durable state is proactively injected, while larger knowledge sources are made available through on-demand discovery and retrieval.

That is directly relevant to EOKS's progressive-disclosure model:

```text
cheap bootstrap context
        |
        +--> on-demand memory recall
        +--> skill retrieval/execution
        +--> Wiki drill-down
        +--> CodeGraph queries
        |
        v
minimum sufficient task context
```

## Skills are more than prompt snippets

Tencent's Skill model is useful prior art for EOKS's distinction between procedural memory and a reusable execution resource.

A useful Skill can carry:

- version;
- resources/files;
- trigger or applicability boundaries;
- execution steps;
- validation rules;
- ownership/visibility;
- lifecycle state.

This fits EOKS's existing promotion model:

```text
execution trace
      |
      v
candidate procedure
      |
validation + outcome evidence
      |
Learning Record
      |
promoted Skill / policy
```

The important boundary remains: **a repeated action is not automatically a good Skill**. EOKS should require outcome evidence, scope, provenance and a way to revise or retire the procedure.

## Resource governance and Agent Loadout

Tencent's Memory Hub treats reusable resources as governed entities. The current architecture includes users, teams, agents, tasks, skills and knowledge resources, with ownership/membership relationships and visibility controls. It also supports an Agent Loadout concept: different agents can be equipped with different resources and priorities.

This maps well to the EOKS distinction between:

- the **resource universe** available to an organization/project;
- an **agent/workload loadout** containing the assets that may be used for a task;
- the **compiled context** actually supplied to a reasoning step.

```text
resource universe
      |
 governance / access / scope
      |
   agent/task loadout
      |
 task + policy + budget
      |
 context compilation
      |
 compiled context
```

This is a useful refinement of the EOKS control-plane model: access and applicability should constrain retrieval **before** relevance ranking becomes the only filter.

## CodeGraph versus GrapeRoot

Tencent's CodeGraph and GrapeRoot overlap, but they occupy different architectural positions.

| | TencentDB Agent Memory CodeGraph | GrapeRoot |
|---|---|---|
| Main role | reusable code representation/resource | proactive context/runtime sidecar |
| Representation | indexed code symbols/relationships/impact paths | local graph used by a context packer plus agent integration |
| Primary delivery | knowledge discovery/query | proactive context packing, with exploration escape hatches |
| Memory/resource governance | explicit team/agent resource model | not its primary abstraction |
| Execution integration | Memory Proxy/adapters | launcher/hooks/MCP around an existing agent |
| EOKS placement | knowledge/evidence provider | context/execution provider |

The important conclusion is that these are complementary. EOKS can consume a CodeGraph-like representation without adopting GrapeRoot's runtime, and can use a GrapeRoot-like context engine without making CodeGraph the canonical knowledge representation.

## TencentDB Agent Memory versus EOKS

TencentDB Agent Memory is closer to an implementation of the **memory/knowledge resource infrastructure** that EOKS needs than GrapeRoot is. It demonstrates:

- persistent multi-resolution memory;
- reusable procedural Skills;
- structured Wiki knowledge;
- code-graph knowledge;
- resource ownership and access control;
- agent-specific loadouts;
- client adapters/proxy integration;
- hybrid proactive/on-demand context delivery.

EOKS still proposes a broader coordination boundary:

```text
                    EOKS control plane
        task / policy / scheduling / evaluation
                         |
             +-----------+-----------+
             |           |           |
          resources   evidence    execution
             |           |           |
     Tencent-like     analyzers   agents/tools
       memory/       graphs/tests
       skills/wiki
       codegraph
             \           |           /
              +----------+----------+
                         |
                 context compiler
                         |
                       Run
                         |
                   Evaluation
                         |
                      Outcome
                         |
                      feedback
```

The main additional EOKS concern is therefore not another memory store. It is **coordination and evaluation**: which resources and evidence providers should be used for this workload, what context should be compiled, what assurance is required, whether the run succeeded, and what should change next time.

## Generic Asset boundary

Tencent's architecture is useful evidence for a generic EOKS **Asset** lifecycle boundary, but the four Tencent families should not become the EOKS ontology.

An EOKS Asset is simply a reusable, governed resource that can contribute information, procedures or evidence to future workloads. Possible examples include:

```text
Memory
Skill
Wiki page
Decision / ADR
CodeGraph
Structural index
Runtime observation
Test/evaluation result
Incident
Task history
```

The semantic types remain distinct. Asset-level metadata can cover identity, provenance, scope/applicability, freshness, revision, trust/verification state, relationships, ownership/access, version and cost characteristics.

The value of the abstraction is **lifecycle interoperability**, not forcing every resource into the same representation or storage system.

## What this adds to the EOKS architecture

The Tencent work suggests a useful three-stage distinction:

```text
1. Resource universe
   What reusable knowledge, memory, procedures and evidence exist?

2. Agent/workload loadout
   Which resources is this workload allowed and expected to use?

3. Context compilation
   Which subset is actually worth placing into the current reasoning context?
```

This prevents a common mistake: treating retrieval as the only selection problem. Authorization, scope, applicability, freshness and lifecycle should constrain the candidate set before context quality is optimized.

It also suggests that context should be evaluated not only at the block level, but at the **asset → loadout → compiled-context** boundary.

## What EOKS should not copy blindly

- **Four resource types are not a universal ontology.** They are a useful implementation decomposition and prior-art vocabulary.
- **Layered memory levels are not automatically semantic truth.** Higher-level summaries must retain provenance and be revisable when source evidence changes.
- **Asset governance is not context quality.** ACLs can determine what may be used; they do not establish that the information is correct or useful.
- **Retrieval success is not task success.** Memory recall, skill selection and code-graph relevance still need end-to-end evaluation.
- **A larger memory system is not automatically better.** More retained state can increase contradiction, staleness and context cost.

## EOKS experiments suggested by this prior art

1. **Flat vs hierarchical memory** — compare raw transcript/vector retrieval with L0-L3-style multi-resolution memory on continuity, stale-memory failures, provenance and cost.
2. **Asset loadout ablation** — compare no loadout, fixed loadout and task-specific loadout; measure leakage, irrelevant context and task outcomes.
3. **Proactive vs on-demand knowledge** — compare GrapeRoot-like pre-injection, Tencent-like hybrid delivery and reactive tool-only exploration.
4. **Skill promotion** — measure whether outcome-linked Skill extraction improves repeat tasks without accumulating harmful/stale procedures.
5. **Asset freshness** — change the underlying code/docs and measure whether derived Wiki/CodeGraph/memory assets are invalidated or refreshed before they mislead the agent.
6. **Cross-agent portability** — keep the asset set and compiled context constant while changing the agent/model/harness.
7. **Asset-level evaluation** — measure which assets were selected, actually used, necessary, misleading or redundant, and connect those labels to downstream outcomes.

## Sources

- [TencentDB Agent Memory](https://github.com/TencentCloud/TencentDB-Agent-Memory) — current project overview and architecture.
- [MemoryCore](https://github.com/TencentCloud/TencentDB-Agent-Memory/blob/feat/server_team/MemoryCore/README.md) — memory layers, knowledge metadata and resource management.
- [MemoryProxy](https://github.com/TencentCloud/TencentDB-Agent-Memory/blob/feat/server_team/MemoryProxy/README.md) — delivery modes, Skill/knowledge discovery and agent integration.
- [v2.0.0 / v2.0.1-beta.1 releases](https://github.com/TencentCloud/TencentDB-Agent-Memory/releases) — current asset and proxy capabilities.

These links are references to upstream prior art, not EOKS dependencies.
