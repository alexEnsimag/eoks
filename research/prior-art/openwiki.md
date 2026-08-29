# OpenWiki

[OpenWiki](https://github.com/AnswerLayer/OpenWiki) is useful prior art for treating generated codebase documentation as a **derived evidence representation** rather than as the canonical source of truth.

## EOKS-relevant pattern

OpenWiki's Code Brain workflow generates repository knowledge as Markdown that can live in Git, while lightweight pointers such as `AGENTS.md` and `CLAUDE.md` can make that knowledge discoverable to coding agents. The important architectural distinction is:

```text
source of truth
    |
    +--> generated representation
    |
    +--> discovery / navigation pointer
    |
    +--> context compiler
```

The generated representation is useful because it is reviewable, diffable and versioned. The discovery pointer is not the knowledge itself; it is a bootstrap/navigation mechanism. EOKS can generalize this beyond Markdown: a pointer may lead to a source file, ADR, graph query, analyzer result, test, runtime trace or generated document.

Git is particularly useful when provenance, human review, rollback and alignment with a source revision matter. It should not be treated as the universal storage layer for volatile or high-volume evidence.

## Lifecycle implication

OpenWiki reinforces a general EOKS lifecycle for **derived representations**:

```text
source / observation
        |
        v
derived candidate representation
        |
provenance + validation
        |
review / promotion
        |
published evidence / knowledge
        |
freshness + drift detection
        |
update / regenerate / invalidate
```

The generator itself is part of the workload: it has cost, can introduce errors, and should be observable and evaluated. A generated page should therefore carry enough provenance to determine what source revision produced it and whether it is still trusted.

This is a general principle, not an OpenWiki-specific requirement. The same lifecycle applies to code maps, generated documentation, semantic indexes and other derived knowledge/evidence representations.

## What OpenWiki does *not* imply

OpenWiki should not become an EOKS dependency, nor should repository Markdown become the universal knowledge substrate. Its contribution is the concrete production example of **versioned derived knowledge + lightweight discovery + validation/review + freshness management**.

## Source

- Udaykiran Estari, [OpenWiki in Production: A Realistic Setup & Review Guide](https://medium.com/@UdaykiranEstari/openwiki-in-production-a-realistic-setup-review-guide-23edce255b09).
