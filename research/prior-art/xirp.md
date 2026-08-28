# Xirp / Spotify

[Xirp](https://xirp.spotify.com/) is useful prior art for a part of EOKS that is easy to under-specify: **shared institutional context for AI coding agents**.

The public Xirp site describes it as an agentic development environment that connects an agent to services, ownership, dependencies, documentation and architectural decisions. It also describes a Portal-based Workspace for sharing work items, sessions and documentation, and a mechanism for turning coding-session knowledge into living documentation that can be reused by later engineers or agents. Xirp is presented as a beta and as a harness that can work with Claude, Gemini and Codex rather than being tied to one model provider.

The important point for EOKS is not the product packaging. It is the architectural observation behind it:

> **A coding agent needs context about the system and organization around the code, not just the files in front of it.**

## What problem Xirp identifies

Xirp frames a common failure mode of AI coding agents as a retrieval problem. The knowledge needed to make a technically plausible decision operationally correct may already exist in Slack discussions, service ownership, architectural decisions, documentation or the experience of the people who built a system. The difficulty is getting the right knowledge into the session at the right time.

This complements EOKS's existing context-engineering model. A useful decomposition is:

```text
organizational / engineering reality
             |
     multiple knowledge representations
             |
      retrieval + evidence selection
             |
       context compilation
             |
       task-specific context
             |
            agent
```

The implication is that **context engineering should include organizational and historical context when the workload requires it**, not only repository-local code context.

## Xirp's three useful capability areas

### 1. System-aware agent context

Xirp's Agent concept is positioned around understanding the system a file belongs to: upstream and downstream services, ownership and architectural rationale. This is broader than a code graph.

For EOKS, this belongs primarily to **knowledge retrieval and evidence provision**:

```text
service ownership      -> organizational evidence
service dependencies   -> structural evidence
architecture decisions -> historical/canonical evidence
documentation          -> canonical knowledge
code                   -> authoritative implementation evidence
```

A context compiler can then select the subset relevant to the current task.

This reinforces an existing EOKS distinction: a graph, document store or service catalog is not itself context. It is a representation or evidence source from which context can be compiled.

### 2. Shared Workspace / session continuity

The Xirp Workspace concept addresses a different boundary: useful context should survive the end of an individual coding session and be available to another engineer or agent working on the same system.

This maps closely to EOKS's distinction between **working context** and **durable knowledge**:

```text
session
  |
  +--> ephemeral working context
  |
  +--> observations / decisions / artifacts
             |
       candidate knowledge
             |
       validation / provenance
             |
       durable project or organizational knowledge
             |
       future context compilation
```

The key EOKS safeguard remains important: not everything observed in a session should become canonical knowledge. Session output is evidence and candidate knowledge until it has an appropriate scope, provenance, confidence and validation state.

### 3. Living documentation from development work

Xirp describes coding sessions as a source of knowledge that can be converted into documentation and fed into future sessions. This is strong prior art for the **knowledge feedback loop** already present in EOKS:

```text
execution
   |
trace + artifacts + decisions + corrections + outcome
   |
candidate extraction
   |
validation / promotion
   |
updated knowledge
   |
future context
   |
new execution
```

The interesting research question is not whether documentation can be generated automatically. It is whether automatically extracted knowledge remains **correct, useful and appropriately scoped** as the system evolves.

This is particularly important because generated summaries can become stale while source code, service ownership and architecture change.

## What Xirp adds to the EOKS model

Earlier EOKS discussions already separated:

- canonical project knowledge;
- structural code representations;
- semantic/dataflow evidence;
- historical/episodic memory;
- context compilation;
- execution workflows;
- evaluation and feedback.

Xirp makes another dimension explicit: **organizational/system context shared across people and agents**.

That context can include:

- service ownership;
- upstream/downstream relationships;
- system boundaries;
- architectural rationale;
- operational documentation;
- prior work on the same subsystem;
- session-derived knowledge that has been promoted for reuse.

This is broader than repository memory but narrower than an unconstrained enterprise knowledge graph. EOKS should treat it as another set of representations/evidence providers consumed by the context compiler.

## Xirp is not the EOKS control plane

It would be a mistake to turn Xirp into the definition of EOKS.

The public material emphasizes the agentic development environment, shared workspace and institutional context. EOKS is exploring a broader control-plane abstraction that can choose among knowledge, context, execution, models, tools and evaluation resources.

A useful relationship is therefore:

```text
                     EOKS control plane
                            |
          +-----------------+------------------+
          |                                    |
 context / knowledge                      execution
          |                                    |
    +-----+------+                       Xirp-like
    |            |                       agent harness
 project     organizational                    |
 knowledge      context                     model
    |            |
    +-----+------+ 
          |
   context compilation
          |
      task context
```

Xirp can be viewed as a **candidate implementation of part of the context + execution surface**, while EOKS asks how such capabilities should be selected, evaluated and composed with other providers.

## Xirp versus Graphify

These systems illustrate two different meanings of "understanding the system":

| | Xirp | Graphify |
|---|---|---|
| Primary focus | System/organizational context for coding agents | Structural code/repository representation |
| Ownership | Yes, as described publicly | Not the primary focus |
| Architectural decisions | Yes, as described publicly | Not inherently |
| Service context | Yes | Usually represented only insofar as encoded in the repository |
| Code relationships | Yes, as part of broader system understanding | Strong focus |
| Dataflow/invariants | Not the primary public positioning | Structural graph is not proof of deep semantic flow |
| Session continuity | Core part of the product story | Can support durable derived artifacts, but not its defining role |
| Context assembly | Central | Primarily provides evidence/navigation |

The useful conclusion is **complementarity rather than competition**:

```text
                    task
                     |
              context compiler
                     |
       +-------------+-------------+
       |                           |
  Xirp-like system            Graph/code graph
  context provider             evidence provider
       |                           |
       +-------------+-------------+
                     |
              task-specific context
```

Graphify can tell an agent where code relationships are. Xirp's model highlights why the surrounding system and organization matter. EOKS can potentially combine both.

## Xirp versus OpenWolf / session-memory systems

There is also overlap with the earlier EOKS research on OpenWolf and behavioral memory. The distinction is useful:

- **repository/project summarization** reduces repeated discovery;
- **session memory** preserves potentially useful experience across sessions;
- **institutional context** connects work to the broader system, ownership and architecture;
- **canonical knowledge** is the reviewed source of truth;
- **context compilation** selects what the agent actually receives.

These are related capabilities, not interchangeable concepts.

A strong EOKS architecture should be able to use all of them without requiring one product to own the whole stack.

## Important caveat: generated knowledge is not automatically truth

The strongest risk in the Xirp-style approach is also the strongest reason to make the EOKS knowledge lifecycle explicit.

A development session contains a mixture of:

```text
                    session
                       |
          +------------+------------+
          |            |            |
      durable       temporary     incorrect
       insight      reasoning     conclusion
          |            |            |
          +------------+------------+
                       |
                 candidate set
                       |
              validation / ranking
                       |
          +------------+------------+
          |                         |
    durable knowledge        ephemeral memory
```

Automatically persisting everything can create a feedback loop in which an incorrect generated summary is later retrieved as if it were an authoritative fact.

EOKS should therefore preserve:

- provenance;
- source revision/time;
- scope and applicability;
- confidence/validation state;
- contradictions;
- promotion history;
- invalidation/expiry semantics where appropriate.

The goal is **compounding context quality**, not simply accumulating more text.

## Research implications

Xirp suggests several concrete EOKS experiments:

1. **System-context ablation** — compare an agent with repository-only context against repository + ownership + service/dependency + architectural-rationale context.
2. **Session-continuity experiment** — compare fresh sessions with and without validated knowledge promoted from previous sessions.
3. **Institutional-context retrieval** — test whether the right service/owner/decision evidence can be selected automatically without flooding the context with irrelevant organizational information.
4. **Generated-documentation quality** — measure whether session-derived documentation remains accurate as code, architecture and ownership change.
5. **Human correction loop** — measure whether include/exclude/correct actions in a Context Workbench improve future selection without overfitting to one engineer or task.
6. **Cross-agent portability** — test whether durable context remains useful when switching between Claude, Gemini, Codex or another execution harness.

These experiments directly extend EOKS's existing context-quality, memory-lifecycle and control-plane research.

## Bottom line

Xirp strengthens an EOKS hypothesis that was already emerging from several other tools:

> **The valuable unit is not "a better prompt" or "a bigger memory store". It is a reliable mechanism for turning distributed engineering reality into the minimum sufficient, provenance-bearing context for the current task.**

Xirp is particularly interesting because it emphasizes **institutional memory and system awareness** rather than only repository structure. EOKS should incorporate that capability into its model without making Xirp, or any other single tool, the canonical architecture.
