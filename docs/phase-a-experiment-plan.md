# Phase A experiment plan

This plan updates the practical Phase 1 direction after the latest tool and workflow review. It is intentionally mechanism-first: EOKS should learn from tools without making the harness depend on any one product.

## Phase A goals

Phase A should answer three related questions in parallel:

1. **Can EOKS control the context lifecycle around a real coding-agent harness?**
2. **Can we build a useful end-to-end representation of a real software system, rather than only a repository-local code graph?**
3. **Can useful context be captured and evolved from work itself, without making the human the integration layer?**

The three experiments share measurements and artifacts, but remain separable so one does not mask failures in another.

## A1 — Harness-native context mechanisms

Start from mechanisms observed in OpenWolf and Spotify Portal/Shunt rather than adopting either wholesale.

### OpenWolf-derived mechanisms

- session/lifecycle hooks
- durable local session state
- pre-compaction state preservation
- context/session artifacts
- token and operation measurements
- session handoff

### Shunt / Spotify Portal-derived mechanisms

- intercept expensive context-producing operations before they reach the frontier model
- distinguish cheap/read-oriented work from operations that need the main model
- transform or delegate selected operations
- keep the policy decision explicit and measurable

### Capture / observation gap

The second-brain review adds an important upstream question. Notion, Obsidian, Capacities, Mem, Tana and Roam all assume some deliberate capture/workspace interaction, although they differ substantially in structure and automation. Soda represents the opposite hypothesis: passively observe work context and turn it into durable, searchable knowledge without requiring the user to stop and record it.

EOKS should treat this as a **capture/observation mechanism to investigate**, not as a reason to build a second-brain application. The key experiment is whether work-coupled observation can produce useful, permissioned evidence/artifacts with acceptable privacy, provenance and noise characteristics.

Tana is particularly relevant as a bridge case: its current products combine structured graph memory with meeting/context capture and agentic actions. It is useful prior art for the hypothesis that capture should happen as a side effect of work, while still requiring us to test whether EOKS needs a simpler, harness-native mechanism.

### EOKS harness boundary

The implementation should expose a small stable event interface and keep harness-specific hook implementations replaceable. Avoid coupling the experiment to an external database, MCP server, or hosted observability service.

Initial feature flags should remain small and independently disableable:

```text
EOKS_READ_INTERCEPT
EOKS_CONTEXT_STATE
EOKS_SESSION_MEMORY
EOKS_TOKEN_METRICS
EOKS_WORK_CAPTURE
```

`EOKS_WORK_CAPTURE` is deliberately an experiment flag, not a commitment to passive surveillance. Capture must be explicit about scope, provenance, permissions and pause/exclusion behavior.

### A1 success signals

Measure at minimum:

- input/output/cache tokens
- number and type of context-producing operations
- intercepted/delegated operations
- latency added by policy
- task outcome / correctness
- context size before and after transformation
- failures attributable to missing or transformed context
- captured observations per unit of work
- useful-vs-noisy captured observations
- provenance and freshness of captured information
- human intervention required to make captured information usable

The first experiment is not "build the perfect context engine" or "build a second brain". It is to establish controlled places where context acquisition, transformation and capture can change and their consequences can be measured.

## A2 — Repository and system representation

**Understand Anything is promoted to a Phase A experiment**, specifically as a representation provider, not as the EOKS runtime.

The key hypothesis is broader than "does its graph look useful?":

> Can a representation provider expose enough structural and semantic information to navigate a real software system across languages and repository boundaries, and can both agents and humans use that representation effectively?

### Candidate provider composition

Do not force Understand Anything to solve the entire problem. The research now explicitly includes complementary providers:

- **Understand Anything** — semantic/code representation and navigation.
- **Nx** — project/dependency and potential synthetic-monorepo cross-repository representation.
- **Sourcegraph** — cross-repository code-intelligence baseline.
- **Structurizr** — system/architecture representation and human-readable C4 views.
- **CodeSee** — codebase/dependency visualization comparison.
- **Graphify / CodeQL / Semgrep** — targeted structural or evidence providers where useful.

These are candidates, not dependencies. The missing capability remains the composition boundary between code, project, system, runtime and evidence representations.

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
7. project/service/API/message/database relationships
8. architecture/system relationships
9. runtime evidence where available
10. incremental changes and re-analysis
11. machine-readable export/provenance
12. interactive visualization and navigation

### Important distinction

A repository-local semantic graph is **not** an end-to-end system representation.

Nx can potentially cover project/dependency boundaries; Sourcegraph provides a strong cross-repository code-intelligence baseline; Structurizr models system architecture; Understand Anything supplies a richer code/semantic layer. None should automatically be treated as the canonical EOKS graph.

The desired end state is closer to:

```text
system
  ├── repository / project
  │    ├── module / symbol
  │    └── semantic/domain concept
  ├── service
  ├── API / RPC
  ├── queue / event
  ├── datastore
  ├── deployment/runtime relationship
  └── evidence (tests / traces / configs)
```

A provider does not have to supply every layer. The experiment should identify which layers it supplies, which it can consume from other providers, and which remain missing.

## A3 — Human-facing research surface

**Obsidian remains in Phase A**, but only as a human-facing inspection, curation and visualization layer. It should consume ordinary EOKS Markdown/artifacts rather than become part of the agent execution path.

The other second-brain tools from the review are comparison points, not runtime dependencies:

- **Notion** — structured workspace/databases/collaboration; useful comparison for organization and operational documentation.
- **Capacities** — object-based structured knowledge with links/backlinks; useful comparison for typed representations and low-friction organization.
- **Tana** — typed graph, structured capture and agent-facing knowledge; useful comparison for evolving work-coupled knowledge.
- **Roam Research** — linked/daily-note model; useful comparison for deliberately curated temporal knowledge.
- **Mem** — AI-assisted capture, recall and evolving workspace; primarily Phase B memory prior art.

The question is not which of these should become EOKS. The question is which mechanisms are worth reproducing in a simpler, work-coupled EOKS artifact flow.

```text
agent / human work
       ↓
EOKS harness + capture/observation
       ↓
measurements + representations + decisions + evidence
       ↓
Markdown / structured artifacts
       ↓
Obsidian or other inspection surfaces
```

## Explicit Phase B boundary

Phase B begins once Phase A has established that the current-session lifecycle, harness intervention, representation and capture mechanisms are useful enough to support persistent evolution.

### Phase B — persistent/evolving memory and richer observability

- **Hindsight / LangMem-style memory** — durable semantic/episodic/procedural memory and learning.
- **Mem / Mem0 / Zep-style systems** — persistent recall, automatic organization and memory retrieval.
- **Tana** — especially relevant as prior art for typed knowledge graphs fed by meetings/work and exposed to agents; useful for testing whether structured evolving memory beats simpler EOKS artifacts.
- **Opik / LangSmith / Langfuse-style observability** — richer traces, evaluations and operational analysis once minimal local measurements are understood.
- **Notion / Capacities / Roam / Obsidian** — remain human-facing knowledge-management comparison points unless experiments show a concrete mechanism EOKS should adopt.
- **Soda-style passive work capture** — Phase B candidate if Phase A establishes that observation quality, privacy, provenance and usefulness justify persistent background capture. The current evidence is insufficient to make it an implementation dependency.

The Phase B question is not "how do we store more memory?" It is:

> Can EOKS preserve and evolve the parts of prior work that materially improve future work, while invalidating stale knowledge, retaining provenance, detecting contradictions and avoiding accumulation of irrelevant history?

## Deferred / reference tools

- **GrapeRoot:** reference/prior art for proactive context compilation. Its critical graph engine is proprietary, so it is not a strong first implementation substrate.
- **Graphify / CodeQL / Semgrep:** targeted structural/evidence providers; not the Phase A system representation by themselves.
- **CodeSee:** visualization/reference candidate; useful for comparison but not currently a required runtime component.
- **Nx:** high-value cross-repository/project-graph candidate; test as a provider rather than adopting its broader platform surface.
- **Sourcegraph:** strong cross-repository intelligence baseline; reference and comparison rather than an immediate EOKS dependency.
- **Structurizr:** system-level architecture-model reference; complementary to code-level representation rather than a replacement for it.

## Phase A decision gates

Do not promote a tool because its UI is impressive. Promote it when an experiment establishes that it provides a missing capability with acceptable operational cost.

### Harness gate

Can we change context acquisition/transformation at the harness boundary and attribute resulting changes in tokens, latency, and task outcomes?

### Capture gate

Can EOKS capture useful work-derived context as a side effect of work, without requiring the human to become the integration layer, while preserving permissions, provenance, freshness and acceptable noise?

### Representation gate

Can we represent and navigate a polyglot, multi-repository system well enough to answer real engineering questions faster or more accurately than native repository search alone?

### System-model gap

If no current provider or provider composition passes the cross-repository/system-level requirement, that is a useful Phase A result. It becomes an explicit EOKS research gap rather than a reason to force one repository graph into the role of a system model.
