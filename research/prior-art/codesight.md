# CodeSight: deterministic repository context compilation prior art

[CodeSight](https://github.com/Houseofmvps/codesight) is a useful adjacent system for the **repository-understanding and context-preparation layer** of EOKS. It is not a replacement for the EOKS control plane. Its strongest contribution is showing how deterministic code analysis, persistent knowledge views and MCP can turn repeated repository exploration into a reusable evidence service.

## What CodeSight provides

The current project describes itself as a universal AI context generator. It combines:

- AST-based extraction for TypeScript and framework/ORM-specific detectors;
- regex-based fallback detection for other supported languages/frameworks;
- routes, schemas, components, libraries, configuration, middleware, events and dependency relationships;
- dependency-graph and blast-radius queries;
- persistent wiki articles generated from extracted code structure;
- a knowledge mode that maps Markdown notes such as ADRs, meeting notes, retrospectives, specifications and research into a compact knowledge map;
- MCP tools for requesting targeted repository evidence rather than loading the complete map;
- generated configuration for several coding-agent environments;
- watch/hook modes for keeping derived context current.

The project explicitly reports that its wiki approach starts with a small index and then retrieves targeted articles, rather than loading the full repository map into every session. These are the most EOKS-relevant design choices; the project's benchmark token-savings figures should be treated as project-reported measurements rather than general guarantees.

## The key architectural idea

CodeSight demonstrates a useful two-stage optimization:

```text
repository
    |
    | deterministic extraction
    v
structured repository knowledge
    |
    +--> compact index
    |
    +--> targeted articles / queries
    |
    v
agent context
```

This is better understood as **knowledge/evidence compilation** than as simply generating a large context file. The generated artifacts are derived representations: they help the agent navigate authoritative source material without repeatedly rediscovering the same structure.

The important EOKS principle is therefore:

> **Do not make the model rediscover deterministic repository structure when it can be compiled once and queried selectively.**

## CodeSight's wiki and knowledge modes

The wiki is particularly interesting because it separates a small catalog from deeper topic-specific material. The index can act as a bootstrap pointer, while an agent retrieves an article such as authentication, database or payments only when the task requires it.

The newer knowledge mode extends the idea beyond source code. It classifies Markdown material into categories such as decisions, meeting notes, retrospectives, specifications and research and produces a compact knowledge map.

This creates an instructive boundary for EOKS:

```text
code                         human knowledge
  |                                |
  v                                v
CodeSight code scan          CodeSight knowledge scan
  |                                |
  +------------+-------------------+
               |
        derived navigation
        / knowledge maps
               |
               v
       EOKS context compiler
               |
               v
             agent
```

However, CodeSight's classification of Markdown is still a **derived index**, not automatically authoritative organizational truth. EOKS should preserve the distinction between canonical human-reviewed knowledge and generated navigation summaries.

## EOKS placement

CodeSight spans two related capabilities:

1. **Structural evidence provider** — deterministic facts about repository structure, dependencies, routes, schemas, configuration and impact.
2. **Context navigation/compilation aid** — compact indexes and targeted views that reduce unnecessary exploration.

It is therefore primarily a **knowledge/evidence + context layer** component.

It is not, from the public project architecture, the EOKS control plane because it does not attempt to coordinate the complete workload lifecycle across heterogeneous resources, policies, models, workflows, evaluations and outcomes.

### A useful composition

```text
                     EOKS control plane
                              |
                task / policy / loadout / budget
                              |
                    context compilation
                              |
             +----------------+----------------+
             |                                 |
       CodeSight provider                other providers
             |                         |       |       |
       repository structure         OKF    Graphify   tests
       wiki / knowledge map         ADRs   analyzers  runtime
             |                         |       |       |
             +-------------------------+-------+-------+
                              |
                       selected evidence
                              |
                            agent
                              |
                         evaluation
```

This makes CodeSight a plausible **provider implementation**, rather than something EOKS should reproduce wholesale.

## CodeSight versus GrapeRoot

These projects overlap around reducing repository exploration, but they optimize different layers.

| | CodeSight | GrapeRoot-like architecture |
|---|---|---|
| Primary contribution | deterministic repository knowledge + targeted retrieval | proactive context optimization around an existing coding agent |
| Code understanding | central | input to context optimization |
| Persistent repository representation | wiki / maps | graph and session-oriented context |
| MCP | specialized evidence queries | graph/context exploration |
| Context delivery | index + targeted article/tool query | proactive packing plus on-demand exploration |
| Session/task optimization | secondary | central |
| Control/evaluation loop | outside scope | outside the broader EOKS scope as well |

The distinction is useful: **CodeSight can prepare and expose evidence; GrapeRoot-like systems can decide how to inject or retrieve it around an agent lifecycle.** An EOKS implementation could combine both patterns without making either one the canonical architecture.

## CodeSight versus Graphify and general code graphs

Graphify-like systems emphasize the graph as a first-class structural representation. CodeSight packages similar structural facts into an agent-oriented context product with additional specialized detectors, wiki views and MCP queries.

The important commonality is the representation boundary:

```text
code
  |
  +--> graph / AST / symbol representation
  |
  +--> routes / schema / events / config facts
  |
  +--> impact relationships
  |
  v
context/evidence provider
```

Neither a graph nor a CodeSight map should be treated as the complete semantic truth of the software system. Deep dataflow, runtime behavior, architecture compliance and business invariants may require specialized analyzers, tests or runtime evidence.

## CodeSight versus canonical knowledge

CodeSight's knowledge mode is useful because it recognizes that decisions, specifications and research matter alongside source code. But EOKS should keep three concepts separate:

```text
canonical knowledge
  ADRs / reviewed docs / scoped CLAUDE.md
           |
           v
      authoritative truth

CodeSight-generated maps
  indexes / classifications / summaries
           |
           v
      navigation evidence

source / tests / runtime
           |
           v
      direct evidence
```

A generated map can point to an ADR without becoming the ADR. Likewise, a generated description of a function should normally point to the implementation rather than become a second mutable copy of it.

This follows the broader EOKS principle of **pointers before duplication**: derived representations should locate and rank authoritative evidence where possible, while synthetic insights that have no authoritative source can be promoted to durable knowledge through a governed lifecycle.

## Incremental maintenance

CodeSight's watch and hook modes reinforce another EOKS idea: derived context should be updated incrementally and kept fresh close to source changes.

A useful generalized lifecycle is:

```text
source change
    |
    v
affected representations
    |
    +--> structural map refresh
    +--> impacted context/wiki entries
    +--> stale-cache detection
    +--> candidate knowledge review
```

A dependency graph can help determine which derived artifacts need regeneration. EOKS should avoid rebuilding unrelated representations after every change.

Freshness still needs to be explicit. A generated map should carry at least a source revision or equivalent provenance so an agent can distinguish current evidence from stale context.

## What CodeSight validates for EOKS

CodeSight strengthens several EOKS hypotheses:

- **Deterministic extraction belongs below context compilation.** AST and framework facts should not consume model tokens merely to rediscover them.
- **Progressive disclosure beats repository-wide prompt stuffing.** A small index plus targeted retrieval can be a better bootstrap than a large static context map.
- **MCP can be an evidence interface, not only an action interface.** Specialized read-only queries are useful context primitives.
- **Repository knowledge benefits from persistent derived representations.** The same structure should not be reconstructed independently in every agent session.
- **Code and organizational knowledge are related but distinct.** Decisions, retrospectives and specs can be indexed alongside code while retaining their different authority and provenance.
- **A context compiler should compose providers.** CodeSight can answer repository-structure questions while other providers answer runtime, security, historical or organizational questions.

## What CodeSight does not establish

The project should not be used as evidence that:

- generated context is always correct or sufficient;
- token savings necessarily imply better task outcomes;
- static analysis can establish arbitrary semantic invariants;
- a generated knowledge map should replace canonical documentation;
- an MCP interface by itself constitutes a context-control plane;
- one proactive or reactive delivery strategy is universally optimal.

These remain EOKS evaluation questions.

## EOKS research questions opened by CodeSight

1. Can CodeSight-like providers expose a common EOKS `EvidenceProvider` contract with provenance, revision, confidence, cost and freshness metadata?
2. When should a context compiler request a summary/article versus direct source evidence?
3. Can blast-radius information be used to narrow the set of knowledge, tests and analyzers considered after a proposed change?
4. How should generated wiki/knowledge artifacts be invalidated when their source evidence changes?
5. Can multiple providers share a common progressive-disclosure protocol without forcing their internal representations into one graph?
6. Does a 200-token bootstrap plus targeted evidence outperform both full-map injection and purely reactive exploration on representative coding tasks?
7. Which generated facts are useful enough to promote into canonical knowledge, and what validation should be required first?

## Bottom line

CodeSight is best viewed as **a practical implementation of the repository-context/evidence-provider layer** that EOKS needs, not as an alternative definition of EOKS.

Its strongest architectural lesson is the separation between **compiled knowledge for navigation** and **task-specific context selection**. CodeSight largely solves the former. EOKS can own the latter: selecting among CodeSight, canonical documents, graphs, memory, history, analyzers, tests and runtime evidence according to the task, policy and budget, then evaluating whether that context actually improved the outcome.
