# Agent codebase-context and repository-intelligence tooling landscape

Research date: 2026-09-29

This note expands EOKS prior-art research on tools that help coding agents understand repositories, select context, navigate code, maintain project memory, compress agent traffic, or evaluate code intelligence.

It is a landscape, not an adoption list. EOKS should not become a bundle of these tools. The purpose is to extract reusable concepts, understand boundaries, and identify experiments.

## 1. Executive synthesis

The tools in this landscape are often described with the same words — context, code intelligence, knowledge graph, memory, token savings — but they solve different problems.

A useful decomposition is:

engineering reality
  code · docs · git · tests · decisions · runtime
                         |
             deterministic / learned
                  representations
                         |
      +------------------+------------------+
      |                  |                  |
 context packaging  structural code   semantic/
                    representation     explanatory
                         |
                    graph / symbols
                         |
                   retrieval / analysis
                         |
                task-shaped evidence
                         |
                  context compilation
                         |
                       agent
                         |
                  execution/validation
                         |
                  outcomes/feedback

Main families:

1. Context packaging: Repomix, Gitingest, code2prompt, Riflet.
2. Structural code intelligence: Aider Repo Map, Serena, Sourcegraph, grepai.
3. Code knowledge graphs: GitNexus, CodeGraph, Graphify, OpenVisio, code-review-graph, codebase-memory-mcp, GraphMind, GitGalaxy.
4. Persistent explanatory/project knowledge: DeepWiki, Mex, parts of Repowise and GraphMind.
5. Agent-facing repository intelligence: Greptile, Repowise, CodeGraph, GitNexus.
6. Incremental/fresh context infrastructure: CocoIndex.
7. Context/token compression: Headroom, LLMLingua/LLMLingua2, Selective Context, AIFW.
8. General token/output optimization mentioned by the benchmark discussion: RTK, Caveman.

The strongest EOKS lesson is not "build a graph":

Maintain multiple representations of engineering reality, then select the cheapest reliable representation/evidence sufficient for the current workload and compile it into task-specific context.

This reinforces the existing EOKS distinction between knowledge, experience, context and evaluation.

## 2. The September 2026 LLMDevs benchmark

A September 2026 r/LLMDevs benchmark compared Repowise, CodeGraph, Serena, Graphify, code-review-graph, CocoIndex and codebase-memory-mcp across Codex, Claude Code and a local model.

Source:
https://www.reddit.com/r/LLMDevs/comments/1wp1szf/i_benchmarked_repowise_codegraph_serena_graphify/

The author used 48 Django questions derived from SWE-bench, five question types, the same agent/prompt/repository commit/tool access, fresh indexes, a no-tool baseline, 261 runs, and 43 paired questions after an API cap. It also added a deterministic retrieval benchmark and compiler-graded call-graph correctness tests.

Reported Codex output-token changes versus baseline:

| Tool | Output tokens | Tool calls |
| --- | ---: | ---: |
| Repowise | -31.6% | 3.8 |
| CodeGraph | -24.4% | 4.0 |
| Serena | -14.8% | 10.1 |
| Graphify | -8.9% | 7.4 |
| code-review-graph | -6.0% | 7.2 |
| No tools | baseline | 7.2 |

The author explicitly concludes that none of the tested tools produced the commonly advertised 60–90% full-session savings.

This does not mean the tools are ineffective. It means the measurement unit matters. A single compressed payload can show dramatic savings while the complete agent session saves much less because agents re-read, backtrack and re-plan.

Important caveats:

- The benchmark author disclosed working on Repowise, so its result is not independent validation.
- Claude Code tool-use results were unstable and strongly affected by harness/tool-discovery behavior.
- The quality judge found no meaningful quality winner; reported differences were smaller than benchmark rerun noise.
- The deterministic retrieval benchmark and compiler-graded call graph are useful because they reduce dependence on an LLM judge.

For EOKS, measure complete sessions rather than one tool response:

- input/output tokens;
- tool calls;
- wall-clock time;
- index/build time;
- index memory/storage;
- retrieval precision/recall;
- task quality;
- validation outcome;
- stale-index failures;
- harness/tool discovery;
- cache effects;
- model/agent version.

## 3. Individual tools

### Repomix
Repository: https://github.com/yamadashy/repomix

Core abstraction: repository to portable LLM-readable context.

It packages repository contents with filtering, token counting and compression, and supports MCP. Tree-sitter-based compression can produce a smaller structural representation.

Distinctive idea: multi-resolution source representation.

EOKS: context-materialization provider, not a knowledge layer. The reusable idea is that one resource can have full, compressed, structural and selected-file representations.

### Gitingest
Repository: https://github.com/coderamp-labs/gitingest

Core abstraction: repository to deterministic textual digest.

Distinctive idea: simple ingestion with little semantic machinery.

EOKS: useful baseline and experimental control. It reminds us that sophisticated retrieval should beat a simple deterministic digest on a workload before its extra complexity is justified.

### code2prompt
Repository: https://github.com/mufeedvh/code2prompt

Core abstraction: repository selection/filtering to templated context artifact.

Distinctive idea: context templates for different tasks such as debugging, implementation, review and explanation.

EOKS: strong prior art for context policy and context compilation.

### Aider Repo Map
Documentation: https://aider.chat/docs/repomap.html

Core abstraction: repository to ranked structural skeleton.

It uses Tree-sitter, definitions/references and dependency relationships, with relevance ranking under a token budget.

Distinctive idea: global structural awareness without loading all source.

EOKS: very strong prior art for multi-resolution structural context. Global skeleton plus detailed local source is a particularly useful pattern.

### Serena
Repository: https://github.com/oraios/serena

Core abstraction: IDE/LSP semantic capabilities exposed to agents.

Typical operations include symbol lookup, references, definitions and semantic edits.

Distinctive idea: code understanding as a capability rather than merely retrieval.

EOKS: primarily CAPABILITIES, not KNOWLEDGE. It is complementary to repository graphs: an LSP can answer precise symbol questions while a graph can answer broader relational questions.

### grepai
Repository: https://github.com/yoanbernabeu/grepai

Core abstraction: local semantic code search plus call graphs.

Distinctive idea: semantic retrieval and structural traversal are separate complementary query modes.

EOKS: supports a retrieval-provider family: lexical, semantic, symbolic, dependency and call/execution graph.

### Sourcegraph
Website: https://sourcegraph.com/

Core abstraction: large-scale code search, navigation and code intelligence.

Its SCIP ecosystem provides language-neutral code-intelligence data.

Distinctive idea: language-specific analysis can feed a language-neutral intermediate representation.

EOKS: strong architectural prior art for separating extraction, normalized structural representation, search/navigation and agent context.

### DeepWiki
Website: https://deepwiki.com/

Core abstraction: repository to generated explanatory knowledge.

Distinctive idea: explanatory knowledge is different from structural knowledge. A graph can say A calls B; documentation can explain why A exists and how the subsystem works.

EOKS: durable explanatory/project knowledge provider.

### GitNexus
Repository: https://github.com/digitalapplied/gitnexus

Core abstraction: resolved code knowledge graph plus precomputed relational intelligence and agent tools.

It builds symbols, relationships, functional clusters and execution processes, then exposes higher-level queries such as context, impact and trace.

Distinctive ideas:

1. Precomputed relational intelligence. The agent receives an impact/process/trace result rather than traversing raw edges through many calls.
2. Community detection to functional areas and generated agent skills.
3. Hybrid search and graph-aware hooks.

EOKS: extremely strong prior art for structural representation, derived evidence views, capability generation and context compilation. The graph should still be treated as one representation/provider rather than the EOKS ontology.

### CodeGraph
Repository: https://github.com/codegraph-ai/CodeGraph

Core abstraction: cross-language semantic code graph plus agent-facing analysis and optional persistent memory.

The current project describes Tree-sitter parsing across 38 languages, a large MCP surface and IDE integrations.

Distinctive ideas:

1. Intent-aware context: explain, modify and debug can require different evidence.
2. Capability-surface management: exposing many tools is itself a context/selection problem.

EOKS: very strong prior art for CAPABILITIES + POLICY + context compilation.

### Graphify
MCP documentation: https://graphiffy.com/mcp

Core abstraction: local knowledge graph exposed through graph-native MCP tools.

Distinctive idea: explicit provenance/confidence on relationships, including distinctions such as extracted, inferred and ambiguous in the broader project.

EOKS: strong prior art for derived evidence provenance. Derived relationships should not all look equally authoritative.

### OpenVisio
Website: https://openvisio.io/

Core abstraction: deterministic repository map shared by humans and agents.

Distinctive idea: one underlying representation can support agent queries, navigation and human visualization.

EOKS: supports the idea of multiple projections over the same derived evidence rather than separate knowledge systems.

### Riflet
Website: https://riflet.com/

Core abstraction: human-controlled context selection and packaging.

Distinctive idea: human selection remains a valid context strategy.

EOKS: context selection should support automatic, human and hybrid modes.

## 4. Additional tools from the Reddit discussion

### Repowise
Repository: https://github.com/repowise-dev/repowise

Core abstraction: multi-layer repository intelligence.

It indexes dependency graph, git history, documentation, architectural decisions, code health, change risk and test intelligence, and exposes task-shaped MCP tools.

Distinctive ideas:

- task-shaped evidence rather than many primitive calls;
- combining structural, historical, documentation and quality evidence;
- proactive agent integration.

EOKS: very strong prior art for a multi-provider evidence plane.

### code-review-graph
Repository: https://github.com/tirth8205/code-review-graph

Core abstraction: incremental structural graph optimized for code-review context.

Distinctive idea: start from a change and identify the minimum relevant structural evidence.

EOKS: useful prior art for change-aware context compilation.

### CocoIndex / CocoIndex Code
Repositories:
https://github.com/cocoindex-io/cocoindex
https://github.com/cocoindex-io/cocoindex-code

Core abstraction: incrementally maintained, continuously fresh context indexes. CocoIndex Code provides AST-aware semantic code search as CLI/MCP/agent skill.

Distinctive idea: freshness is part of representation validity.

EOKS: highly relevant to incremental context maintenance. A derived representation needs source revision/freshness state, not just existence.

### codebase-memory-mcp
Repository: https://github.com/DeusData/codebase-memory-mcp

Core abstraction: persistent structural knowledge graph for fast agent queries.

It uses Tree-sitter plus hybrid LSP resolution for selected languages and exposes structural relationships over MCP.

Distinctive idea: fast persistent structural memory as an agent substrate.

EOKS: prior art for persistent structural representation and hybrid AST/LSP analysis. Its benchmark claims should be treated as project claims; the Reddit benchmark provides additional external measurements.

### GraphMind
Documentation: https://getgraphmind.com/docs/

Core abstraction: structural code graph plus semantic project memory plus proactive agent hooks.

Distinctive ideas:

- explicit separation of graph and semantic memory;
- hooks that can augment search and inject context;
- cross-project links;
- durable conventions and decisions.

EOKS: strong prior art for separating structural evidence, semantic memory and proactive context delivery.

### Mex
Repository: https://github.com/mex-memory/mex

Core abstraction: persistent project memory represented as structured Markdown connected to a deterministic code graph.

Distinctive idea: important learned knowledge remains reviewable and versionable instead of living only in a hidden database.

EOKS: useful for the boundary between derived structural evidence, durable explanatory knowledge and drift detection.

### GitGalaxy
Repository: https://github.com/squid-protocol/gitgalaxy

Core abstraction: deep repository intelligence and static analysis designed to work even without successful compilation.

Distinctive idea: structural intelligence need not depend on one parser strategy; the project author explicitly contrasts its methodology with Tree-sitter approaches.

EOKS: keep provider contracts open to AST parsers, compiler indexes, LSPs, static analyzers and other extraction methods. Tree-sitter should not become a required EOKS dependency.

### Headroom
Repository: https://github.com/headroomlabs-ai/headroom

Core abstraction: context/output compression layer between an application or agent and an LLM.

It can compress tool outputs, file reads, logs and search results, and supports library/proxy/agent-wrapper/MCP modes. It also describes reversible caching and cross-agent memory.

Distinctive ideas:

- compression after retrieval;
- reversible compression;
- output as well as input optimization.

EOKS: context transformation/transport optimization, not canonical knowledge. Reversibility is particularly useful for provenance.

### LLMLingua / LLMLingua-2
Paper: https://arxiv.org/abs/2310.05736

Core abstraction: learned prompt/context compression.

Distinctive idea: token/span pruning as an explicit learned transformation.

EOKS: context compiler transformation after selection. It should be evaluated for information loss, not only compression ratio.

### Selective Context
Paper: https://arxiv.org/abs/2303.06004

Core abstraction: information/surprisal-based context pruning.

Distinctive idea: domain-agnostic information-density selection.

EOKS: useful as a generic transformation that can compose with domain-aware retrieval.

### AIFW
Website: https://aifw.io/

Core abstraction: LLM traffic/context optimization.

The Reddit discussion cites vendor/customer observations of smaller coding-session savings than the commonly advertised 60–90% range.

EOKS: treat this as evidence about optimization interventions, not as independently validated performance.

### RTK
RTK is mentioned in the Reddit benchmark as an earlier token-saving system whose headline savings did not reproduce under the author's workload.

EOKS lesson: per-payload savings can overstate full-session savings.

### Caveman
Caveman is likewise mentioned as a token-saving system whose earlier headline savings were much larger than a later JetBrains rerun.

EOKS lesson: complete-workload measurement is essential.

## 5. Why these tools are complementary

A capability-oriented comparison is more meaningful than one overall ranking:

| Capability | Representative tools |
| --- | --- |
| Repo serialization | Repomix, Gitingest |
| Context templating | code2prompt |
| Human context selection | Riflet |
| Global repo map | Aider Repo Map |
| Symbol/LSP navigation | Serena |
| Semantic code search | grepai, CocoIndex |
| Structural graph | CodeGraph, GitNexus, Graphify, GraphMind |
| Precomputed impact/process analysis | GitNexus, CodeGraph, Repowise |
| Change/review context | code-review-graph, Repowise |
| Historical/project evidence | Repowise |
| Generated explanatory knowledge | DeepWiki, Mex, GitNexus wiki |
| Persistent semantic memory | GraphMind, Mex, CodeGraph |
| Incremental freshness | CocoIndex, code-review-graph |
| Context compression | Headroom, LLMLingua2, Selective Context |
| Visualization | OpenVisio, Graphify, GitNexus |
| Proactive delivery/hooks | GitNexus, GraphMind, Repowise |
| Deep/static analysis | GitGalaxy and specialized analyzers |
| Cross-project intelligence | Repowise, GraphMind, Sourcegraph |

The useful question is therefore not "which tool is best?" but:

Which capabilities are required for this workload, and which provider can supply them with adequate reliability, cost and freshness?

## 6. Three different optimization strategies

These tools reduce agent work in three fundamentally different ways.

### A. Retrieve less

Aider Repo Map, grepai, GitNexus, CodeGraph, Graphify, code-review-graph and CocoIndex.

The intervention is better retrieval:

large repository -> relevant evidence set.

### B. Precompute more

GitNexus, Repowise, CodeGraph, Sourcegraph and codebase-memory-mcp.

The intervention is moving deterministic work from query time into index time:

expensive reasoning -> index -> cheap query-time evidence.

### C. Compress what was retrieved

Repomix, Headroom, LLMLingua2 and Selective Context.

The intervention is transforming an already selected evidence set:

retrieved evidence -> compressed representation.

These are complementary, not mutually exclusive. A useful EOKS experiment is:

structural retrieval -> task-shaped evidence -> reversible compression -> agent.

## 7. Where intelligence lives

The tools distribute intelligence across different layers.

Agent-side:
the model chooses what to search/read.

Provider-side:
the indexer computes structure/relevance.

Harness-side:
hooks proactively intercept or augment agent behavior.

Human-side:
the developer chooses context.

Transformation-side:
a compressor reformats selected evidence.

EOKS should remain neutral about where intelligence lives. The correct placement depends on determinism, latency, freshness, cost, observability, risk and reversibility.

## 8. Provenance and confidence

Several projects independently point toward provenance:

- Graphify: confidence/provenance on edges;
- Repowise: resolution/trust information;
- GitNexus: confidence in relational analysis;
- Sourcegraph: precise code-intelligence indexes;
- Mex: knowledge tied to implementation;
- CocoIndex: incremental freshness.

A useful research-level metadata model for derived evidence is:

Evidence
  source
  source_revision
  representation
  extractor
  extractor_version
  derivation_method
  location
  confidence
  freshness
  scope

This should remain a hypothesis, not automatically become a new EOKS ontology object.

## 9. The code graph is not the knowledge base

A structural graph contains facts such as:

- symbol A calls B;
- file X imports Y;
- module M contains N;
- process P crosses A/B/C.

It does not automatically contain:

- why the architecture exists;
- which decision was intentional;
- which convention the team follows;
- what happened during a failed deployment;
- what the user wants from the current task.

Therefore:

structural graph != durable knowledge != experience != task context

A mature EOKS system can use all four.

## 10. Stronger context-compiler model

The combined landscape suggests:

WORKLOAD
  |
intent + risk + policy
  |
LOADOUT
  |
eligible resources/providers
  |
  +-- graph
  +-- memory
  +-- documents
  +-- history
  +-- analyzers
  |
evidence candidates
  |
relevance / sufficiency
  |
task-shaped evidence
  |
optional compression
  |
provenance + budget
  |
CONTEXT ARTIFACT
  |
AGENT
  |
execution
  |
validation
  |
outcome
  |
evidence / learning

The graph is therefore one evidence provider in the compiler, not the compiler itself.

## 11. Research questions

1. Retrieval versus compression: compare raw exploration, structural retrieval, semantic retrieval, combined retrieval and retrieval plus compression on complete sessions.
2. Precomputation tradeoff: measure index cost + query cost + stale-index cost rather than query tokens alone.
3. Task-shaped versus primitive tools: compare many small graph calls with one precomputed task-shaped evidence call.
4. Proactive versus reactive context: compare agent-discovered tools, hooks and hybrid hint-then-retrieve.
5. Graph correctness: use compiler/reference analyses where available and measure precision, recall, confidence and stale-edge rate.
6. Knowledge lifecycle: compare generated wikis, human-maintained Markdown, graph-derived summaries and session-learned decisions.
7. Minimum sufficient evidence: determine the smallest evidence set that preserves task outcome.
8. Capability exposure: measure whether exposing 5, 15 or 40 tools changes discovery, tool calls, success and context overhead.
9. Provider selection: learn when a structural graph is sufficient and when deeper analysis is worth its cost.
10. Freshness: quantify how stale derived representations become during realistic coding sessions and whether incremental maintenance is sufficient.

## 12. EOKS positioning

The landscape does not suggest that EOKS should build another code graph, semantic code search engine, repository wiki, prompt packer or token compressor.

It strengthens the existing EOKS boundary:

EOKS coordinates heterogeneous representations, evidence providers, context transformations, execution resources and evaluation under workload-specific policy.

A graph provider may be excellent for impact analysis. An LSP provider may be best for exact symbol navigation. A wiki may be best for architectural explanation. Git history may be best for change-risk evidence. A static analyzer may be required for a security invariant. A compressor may reduce transport/context cost after evidence has been selected.

The EOKS problem is deciding which combination is appropriate, what evidence was used, how trustworthy/fresh it is, and whether the intervention improved the workload outcome.

## 13. Provider-contract research direction

A useful research direction is to standardize a provider contract conceptually without standardizing one universal EOKS graph schema:

Provider
  capabilities()
  representations()
  query(workload, constraints)
  evidence()
  freshness()
  provenance()
  cost()

A provider could be Serena, GitNexus, CodeGraph, Repowise, CocoIndex, a static analyzer, a memory store, a wiki or a runtime telemetry system.

The EOKS layer then performs provider selection and context compilation.

This keeps the EOKS semantic model small.

## 14. Final conclusions

1. Codebase understanding is not one problem. Mapping, navigation, retrieval, explanation, impact analysis, memory and compression are distinct capabilities.
2. Graphs are representations, not necessarily knowledge bases.
3. Precomputation can move deterministic reasoning out of the agent loop.
4. Task-shaped evidence can be more useful than exposing primitive graph edges.
5. Semantic navigation and graph intelligence are complementary.
6. Freshness is part of evidence validity.
7. Compression and retrieval are different interventions and should be evaluated separately.
8. Human-selected context should coexist with automated selection.
9. Provenance/confidence should accompany derived structural claims.
10. Agent/harness behavior is an experimental variable.
11. Token savings must be measured over complete workloads.
12. Quality and reliability must be measured alongside tokens.
13. The most promising EOKS abstraction remains a context/evidence control plane over heterogeneous providers, not another code-intelligence engine.

Strongest concepts to carry forward:

- Aider: ranked multi-resolution repository maps.
- Serena: semantic IDE capabilities as agent tools.
- GitNexus: precomputed relational intelligence and graph-derived skills.
- CodeGraph: intent-aware context and capability-surface management.
- Repowise: joining structural, historical, documentation and quality evidence.
- CocoIndex: incremental freshness.
- Graphify: explicit evidence provenance.
- GraphMind/Mex: separation of structural graph from durable project memory.
- Headroom/LLMLingua2: reversible or measurable context transformation.
- Reddit benchmark: evaluate the complete agent workload, including indexing and harness behavior.

## Sources

Primary project sources:

- Repomix: https://github.com/yamadashy/repomix
- Gitingest: https://github.com/coderamp-labs/gitingest
- code2prompt: https://github.com/mufeedvh/code2prompt
- Aider Repo Map: https://aider.chat/docs/repomap.html
- Serena: https://github.com/oraios/serena
- grepai: https://github.com/yoanbernabeu/grepai
- Sourcegraph: https://sourcegraph.com/
- DeepWiki: https://deepwiki.com/
- GitNexus: https://github.com/digitalapplied/gitnexus
- CodeGraph: https://github.com/codegraph-ai/CodeGraph
- Graphify: https://graphiffy.com/mcp
- OpenVisio: https://openvisio.io/
- Riflet: https://riflet.com/
- Repowise: https://github.com/repowise-dev/repowise
- code-review-graph: https://github.com/tirth8205/code-review-graph
- CocoIndex: https://github.com/cocoindex-io/cocoindex
- CocoIndex Code: https://github.com/cocoindex-io/cocoindex-code
- codebase-memory-mcp: https://github.com/DeusData/codebase-memory-mcp
- GraphMind: https://getgraphmind.com/docs/
- Mex: https://github.com/mex-memory/mex
- GitGalaxy: https://github.com/squid-protocol/gitgalaxy
- Headroom: https://github.com/headroomlabs-ai/headroom
- LLMLingua: https://arxiv.org/abs/2310.05736
- Selective Context: https://arxiv.org/abs/2303.06004
- AIFW: https://aifw.io/

Community evidence:

- r/LLMDevs benchmark: https://www.reddit.com/r/LLMDevs/comments/1wp1szf/i_benchmarked_repowise_codegraph_serena_graphify/

Vendor claims and community reports are recorded as evidence to investigate, not as independently validated EOKS conclusions.
