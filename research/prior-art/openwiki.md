# OpenWiki

[OpenWiki](https://github.com/AnswerLayer/OpenWiki) is useful prior art for treating generated codebase documentation as an **operated memory pipeline**, rather than as a one-shot documentation generator. The production-oriented setup described by Udaykiran Estari emphasizes a git-native, reviewable workflow around OpenWiki's Code Brain output. The article is useful primarily for the operational pattern; exact implementation details should be verified against the current OpenWiki project before becoming EOKS requirements.

## What matters for EOKS

### The product is the memory pipeline

OpenWiki's useful artifact is not merely prose. Its Code Brain mode generates linked, version-controlled Markdown under an `openwiki/` directory and adds small discovery pointers in files such as `AGENTS.md` and `CLAUDE.md`. This makes the generated knowledge discoverable by coding agents while keeping it in the repository's normal review/version-control workflow.

This reinforces an EOKS distinction:

```text
source repository
      |
 documentation / analysis generator
      |
 versioned knowledge artifacts
      |
 review / validation / freshness checks
      |
 agent discovery
      |
 task-specific context compilation
```

The generated wiki is therefore a **derived knowledge representation**, not the source of truth. EOKS should preserve provenance back to source revisions and make generated artifacts replaceable or invalidatable when their inputs change.

### Git is part of the trust boundary

For repository-local engineering knowledge, storing generated artifacts in Git provides several useful properties at once:

- explicit version history;
- human review through ordinary diffs and pull requests;
- reproducibility against a repository revision;
- straightforward rollback when generated knowledge is wrong;
- a natural freshness signal through source/documentation commits.

This fits EOKS's existing preference for human-reviewable canonical knowledge and controlled promotion of learned artifacts. It also suggests that **knowledge lifecycle should include review state and change history**, not just retrieval metadata.

Git-native storage is not universally appropriate: volatile runtime observations, private operational data and high-volume derived indexes may need other stores. The architectural principle is to choose a persistence mechanism whose provenance, reviewability, freshness and access properties match the semantic role of the resource.

### Generated knowledge needs an operational loop

A production documentation generator is itself an agentic or computational workload. Its output should therefore be evaluated like other EOKS-produced artifacts rather than trusted merely because a generator produced it.

A useful lifecycle is:

```text
source revision
      |
 generation / analysis
      |
 candidate knowledge
      |
 structural + semantic validation
      |
 diff / review
      |
 promote
      |
 published knowledge
      |
 freshness / drift detection
      |
 regenerate or invalidate
```

This connects OpenWiki to EOKS's existing **candidate extraction → validation → promotion** memory lifecycle. It adds an important operational perspective: documentation generation has cost, can drift, and should have explicit triggers and acceptance criteria.

### Discovery pointers are part of context infrastructure

Small pointers in `AGENTS.md`, `CLAUDE.md` or equivalent agent instruction files are different from the generated knowledge itself. They form a lightweight **discovery/index layer** telling an agent where durable repository knowledge lives.

EOKS should model this as a navigation mechanism rather than duplicate the generated documentation in every agent prompt:

```text
agent bootstrap
      |
      +--> pointer / index
              |
              +--> targeted wiki page
              +--> source file
              +--> ADR / decision
              +--> analyzer result
              |
              v
       compiled task context
```

This is consistent with EOKS's progressive-disclosure model and with CodeSight's compact-index → targeted-query pattern. The important generalization is that a pointer can lead to any authoritative or derived evidence provider, not only a wiki page.

## What not to copy into EOKS

OpenWiki should not become a required EOKS component or imply that repository Markdown is the universal knowledge substrate. Its value here is as a concrete example of:

1. generated knowledge as a versioned artifact;
2. discovery pointers separated from the knowledge payload;
3. generation followed by review/validation;
4. source-revision and freshness-aware regeneration;
5. documentation treated as part of an agent memory supply chain.

These ideas complement, rather than replace, the EOKS resource model, context compiler, knowledge representations, deterministic repository evidence and controlled learning lifecycle.

## Source

- Udaykiran Estari, [OpenWiki Setup Guide: Build AI-Ready Codebase Documentation You Can Trust](https://medium.com/@UdaykiranEstari/openwiki-in-production-a-realistic-setup-review-guide-23edce255b09).
