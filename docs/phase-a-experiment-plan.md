# Phase A experiment plan

This plan updates the practical Phase 1 direction after the recent tool review. It is intentionally mechanism-first: we should learn from tools without making the EOKS harness depend on any one product.

## Phase A goals

Phase A should answer two questions in parallel:

1. **Can EOKS control the context lifecycle around a real coding-agent harness?**
2. **Can we build a useful end-to-end representation of a real software system, rather than only a repository-local code graph?**

The first is the harness experiment. The second is the representation experiment. They should share measurements and artifacts, but remain separable so that one does not mask failures in the other.

## A1 — Harness-native context mechanisms

Start from the mechanisms observed in OpenWolf and Spotify's Portal/Shunt rather than adopting either wholesale.

### OpenWolf-derived mechanisms

- session/lifecycle hooks
- durable local session state
- pre-compaction state preservation
- context/session artifacts
- token and operation measurements
- session handoff

### Shunt-derived mechanisms

- intercept expensive context-producing operations before they reach the frontier model
- distinguish cheap/read-oriented work from operations that need the main model
- transform or delegate selected operations
- keep the policy decision explicit and measurable

The EOKS implementation should expose a small stable event interface and keep harness-specific hook implementations replaceable. Avoid coupling the experiment to an external database, MCP server, or hosted observability service.

Initial feature flags should remain small and independently disableable:

```text
EOKS_READ_INTERCEPT
EOKS_CONTEXT_STATE
EOKS_SESSION_MEMORY
EOKS_TOKEN_METRICS
```

### A1 success signals

Measure at minimum:

- input/output/cache tokens
- number and type of context-producing operations
- intercepted/delegated operations
- latency added by policy
- task outcome / correctness
- context size before and after transformation
- failures attributable to missing or transformed context

The first experiment is not "build the perfect context engine". It is to establish a controlled place where context policy can change and its consequences can be measured.

## A2 — Repository and system representation

**Understand Anything is promoted to a Phase A experiment**, but specifically as a representation provider, not as the EOKS runtime.

The key hypothesis is broader than "does its graph look useful?":

> Can a representation provider expose enough structural and semantic information to navigate a real software system across languages and repository boundaries, and can both agents and humans use that representation effectively?

### Minimum test matrix

Use a deliberately polyglot system where possible:

- Go service/repository
- TypeScript service/repository
- Python service/repository
- more than one repository participating in the same system

Test progressively:

1. files/modules
2. functions/classes/symbols
3. imports/calls/dependencies
4. semantic/domain relationships
5. repository boundaries
6. cross-repository relationships
7. service/API/message/database relationships
8. runtime evidence where available
9. incremental changes and re-analysis
10. interactive visualization and navigation

### Important distinction

A repository-local semantic graph is **not** an end-to-end system representation.

EOKS should explicitly track this gap rather than treating a good repository graph as a solved system model.

The desired end state is closer to:

```text
system
  ├── repository
  │    ├── module
  │    │    ├── symbol
  │    │    └── dependency
  │    └── semantic/domain concept
  ├── repository
  ├── service
  ├── API / RPC
  ├── queue / event
  ├── datastore
  └── runtime relationship / evidence
```

A provider does not have to supply every layer. The experiment should identify which layers it supplies, which it can consume from other providers, and which remain missing.

## A3 — Human-facing research surface

**Obsidian remains in Phase A**, but only as a human-facing inspection and knowledge layer.

It should consume ordinary EOKS Markdown/artifacts rather than become part of the agent execution path.

```text
agent
  ↓
EOKS harness
  ↓
measurements + representations + decisions
  ↓
Markdown/artifacts
  ↓
Obsidian
```

This gives us visualization, curation, and durable research context without introducing another runtime dependency or conflating human knowledge management with agent memory.

## Explicitly deferred

- **Hindsight / LangMem-style memory:** Phase B; first establish current-session lifecycle and context behavior.
- **Opik / hosted-style observability:** Phase B; first establish minimal measurements we fully understand and control.
- **GrapeRoot:** reference/prior art for proactive context compilation. Its critical graph engine is proprietary, so it is not a strong first implementation substrate.
- **Graphify / CodeQL / Semgrep:** use as targeted structural/evidence providers where they answer a concrete representation or evidence question; they are not the Phase A system representation by themselves.

## Phase A decision gates

Do not promote a tool because its UI is impressive. Promote it when an experiment establishes that it provides a missing capability with acceptable operational cost.

### Harness gate

Can we change context acquisition/transformation at the harness boundary and attribute resulting changes in tokens, latency, and task outcomes?

### Representation gate

Can we represent and navigate a polyglot, multi-repository system well enough to answer real engineering questions faster or more accurately than native repository search alone?

### System-model gap

If no current tool passes the cross-repository/system-level requirement, that is a useful Phase A result. It becomes an explicit EOKS research gap rather than a reason to force one repository graph into the role of a system model.
