# Working representations

EOKS already treats Representation as a form optimized for a question or operation. Recent investigation of Excalidraw, Claude Artifacts and MCP Apps exposes a related but distinct capability: a representation can itself be an editable working surface shared by human and agent.

This should remain a lightweight concept, not a new runtime primitive.

## Working representation

A **working representation** is an external, editable representation of a problem, system, plan, hypothesis or work product that helps a human or agent reason about it and can be revisited or modified over time.

Key properties:

- externalized — reasoning is not confined to the model conversation;
- structured enough to manipulate — more than an opaque screenshot or transcript;
- persistent or revisitable — can survive a reasoning step or session;
- editable — can evolve as understanding changes;
- shared or inspectable — human and/or agent can use it as a common object;
- purposeful — supports reasoning, communication, planning or investigation.

It does not have to be visual. Examples include architecture/deployment diagrams, flow and sequence diagrams, state machines, data/process models, investigation maps, design sketches, plans, specifications and structured decision documents.

## Boundary with existing EOKS concepts

| Concept | Primary question |
|---|---|
| Knowledge | What durable information do we have? |
| Representation | In what form is information/evidence optimized for a question? |
| Working representation | What external representation are we actively using to reason, communicate or shape the work? |
| Context | What information is compiled for this reasoning step? |
| Execution state | What has the workload attempted, observed, changed or verified? |
| Outcome | What actually happened? |
| Audit/trace | What happened during execution, and can it be reconstructed? |

A working representation can contain knowledge, hypotheses or execution state, but it is not identical to any of them.

**Execution traces are not working representations by default.** A trace records execution for observability, reconstruction or evaluation. A diagram used by an engineer to reason about the system is a working representation even if it is never executed.

Likewise, an artifact is not automatically a working representation. The useful distinction is whether the object is an editable external surface for reasoning/work, rather than merely a produced output, cache, trace or computed result.

## Excalidraw

Excalidraw is useful prior art because its core object is a structured scene rather than a rendered image. Its file format contains elements and application state as JSON, making drawings persistable and programmatically manipulable.

Its primary product use is visual thinking and communication: diagrams, sketches, brainstorming and collaborative visual work. The EOKS signal is therefore not drawing and not hooks. It is:

> A reasoning workspace can be an external structured representation that both humans and software can inspect and modify.

Recent Excalidraw+ capabilities strengthen this signal: its Public API/MCP integration supports programmatic scene management, while real-time edits, scene-content syncing and collaboration features make the representation a live, externally manipulable object. These are implementation mechanisms around the working representation, not the representation itself.

## Claude Artifacts

Claude Artifacts demonstrate a closely related agent-side pattern. Anthropic describes Artifacts as a dedicated space where generated content such as code, documents, graphics, diagrams and website designs can be viewed, edited and built alongside the conversation.

The emphasis differs:

- Artifacts are commonly introduced as a generated work product that becomes interactive/editable.
- Excalidraw is fundamentally a persistent canvas/workspace that can exist before an agent contributes to it.

Both demonstrate that useful agent interaction can happen through a structured object alongside the conversation rather than only through chat text.

EOKS should therefore avoid treating artifact as the architectural answer. The narrower capability is the working representation; different systems can realize it as an artifact, canvas, document, graph editor or another surface.

## MCP Apps

MCP Apps add an important interface-level observation.

An MCP App links an MCP tool to a UI resource. The host renders the UI, passes tool results into it, and the UI can call tools back through the host. The same capability can therefore expose a model-facing interface and a human-facing interactive surface, with the host mediating their interaction and tool/resource state.

This is relevant when the working representation is interactive:

    capability / resource
          /        \
    model-facing  human-facing
      tool/data        UI
          \        /
           shared state

The architectural lesson is not that EOKS needs MCP Apps. It is that capability and representation interface can be separated.

MCP Apps also supports progressive enhancement: a tool can retain a useful text/structured-data fallback when a host does not support the UI. Interactive presentation should therefore not become the only source of semantic information.

## Representation lifecycle

A working representation can move through:

    authoritative sources / intent / observations
                       |
                       v
                create or select
                       |
                       v
              working representation
                 /             \
              inspect          edit
                 \             /
                    iterate
                       |
                +------+------+
                |             |
                v             v
             discard      retain/promote
                              |
                              v
                    durable knowledge /
                    reusable representation

Promotion should not be automatic. An editable diagram, plan or hypothesis can contain tentative or incorrect information. If it becomes durable knowledge, it needs provenance, validation, freshness and authority treatment like other knowledge.

## Working representation versus context

A representation may be persistent and rich while context compilation exposes only a task-specific projection:

    persistent working representation
                    |
              query / selection
                    |
             context compilation
                    |
              model context

For example, a large architecture diagram might be the shared working representation while a reasoning step receives only the authentication subsystem and relevant annotations.

This avoids turning every representation into prompt text.

## Working representation versus learning

A working representation is not itself a learning mechanism.

Learning can use representations as an input/output surface:

    execution + feedback
            |
            v
    candidate insight / change
            |
            v
    working representation
            |
      validation / review
            |
            v
    durable knowledge / policy / Skill

Hooks can trigger this lifecycle, and learning-oriented hook systems can use hooks to capture or promote information. But **hooks are lifecycle mechanisms; the representation is the object being maintained**.

This distinction explains why Excalidraw belongs in the representation category rather than the learning category.

## Representations as thinking and learning surfaces

The Visual PKM work around Obsidian-Excalidraw adds an important refinement: a representation is not necessarily an output of reasoning. Constructing and manipulating the representation can itself be part of the reasoning process.

Zsolt Viczián describes Excalidraw drawings as more than illustrations: they can become standalone, interlinked documents in the knowledge graph; he also uses them for brainstorming and problem-solving, arguing that the slower visual process creates room for reflection and internalization. His later Visual PKM material describes text and visuals as complementary thinking modes, spatial rather than purely linear connections, reusable visual vocabulary, and iterative construction of visual maps.

This suggests several distinct roles for a representation:

- **Capture** — externalize an idea, observation or hypothesis.
- **Think** — manipulate, reorganize, connect and inspect the representation to understand the problem.
- **Communicate** — use the representation to explain an understanding to another human or agent.
- **Navigate** — use the representation as an index or map through related knowledge.
- **Retain** — keep a useful representation as a durable knowledge object.
- **Learn** — use representation construction, transformation or comparison as part of learning.

These roles can occur in the same object. A sketch can begin as a temporary thinking surface, become a navigation map, and later be retained as a durable knowledge object. The transition is a lifecycle decision, not a change of primitive.

### Representation transformation

The Visual PKM examples also expose a useful operation that is easy to miss if representation is treated as formatting: **transforming information between representations can itself be a reasoning/learning operation**.

For example:

    source material
          |
       summarize
          |
     textual notes
          |
       visualize
          |
     multiple sketches
          |
       distill
          |
   compact visual model

Zsolt's "Book on a Page" workflow follows this kind of progressive transformation from source material through textual summarization and section sketches into a distilled visual summary.

The EOKS implication is not that visual representations are inherently better. It is that **representation choice and transformation are potentially active operations in reasoning and learning**, with different representations exposing different structure.

### Structured relationships, not only pictures

ExcaliBrain provides a related example: relationships between notes can be inferred, explicitly declared, customized and eventually expressed semantically; the resulting visual interface is used to aggregate and contextualize a topic and to explore how it fits into the rest of the vault.

This is important because a working representation need not be a free-form drawing. It can expose or manipulate an underlying relational model. Visual presentation is one interface to the representation; the semantic relationships are part of the represented structure.

### Working representation and learning

This strengthens the distinction between **representation** and **learning mechanism**:

> A representation is not a learning mechanism, but constructing, transforming, navigating and revising representations can be part of a learning loop.

That is compatible with the existing EOKS learning model. Hooks, feedback, evaluation and promotion mechanisms can operate around these changes, while the working representation provides an external surface on which understanding can be developed and made inspectable.

The evidence here is primarily community/practice evidence rather than a controlled demonstration of engineering outcomes. EOKS should therefore treat the mechanism as a research hypothesis to test, not as a proven productivity or learning improvement.

## EOKS architectural status

Current conclusion:

- Working representation is a useful architectural concept.
- It is a specialization/role of existing Representation + Asset concepts, not a new runtime primitive.
- It should not be confused with traces, execution state, context, computed artifacts or durable knowledge.
- A working representation can be visual, textual, structured or code-based.
- Representation construction and transformation can themselves participate in reasoning and learning; this does not make Representation a learning mechanism.
- Human/agent shared manipulation is useful but not mandatory.
- The implementation can be a file, canvas, graph editor, document, UI-backed resource or another structured surface.
- MCP Apps, Excalidraw and Claude Artifacts demonstrate different parts of the pattern.
- Visual PKM and ExcaliBrain provide community evidence for representations as thinking, navigation and knowledge-maintenance surfaces.

This keeps the EOKS ontology small while making an important capability explicit.

## Research gaps

The evidence supports the concept but leaves empirical questions:

1. Does a persistent working representation improve engineering outcomes? Measure task success, rework, context cost, handoff quality and time rather than assuming a diagram or canvas helps.
2. When is visual representation better than text or structured data?
3. What persistence granularity is useful? Per-step, per-workload, per-repository or long-lived persistence can have different staleness costs.
4. How should edits be attributed and validated? Human, agent and derived edits may have different authority.
5. How should large representations be queried into context? Selective projections are likely preferable to wholesale serialization.
6. What is the relationship to learning? Test whether maintained working representations improve knowledge promotion, skill extraction or future retrieval.
7. What protocol is sufficient? MCP Apps demonstrates one tool/UI protocol, but EOKS should test whether simpler file/resource interfaces are enough before adopting a richer UI protocol.

These are experiments, not reasons to add another runtime subsystem now.

## Design principle

> **A useful agent environment can externalize reasoning into persistent, editable representations that humans and agents can revisit — not only into prompts, traces or final outputs.**

The representation should remain subordinate to the workload: create it when it reduces reconstruction or communication cost, maintain it only when its value exceeds its staleness and maintenance cost, and compile only the needed portion into model context.


## References

- Excalidraw developer documentation: https://docs.excalidraw.com/
- Excalidraw JSON schema: https://docs.excalidraw.com/docs/codebase/json-schema/
- Excalidraw+ changelog (Public API, MCP, real-time edits and scene syncing): https://plus.excalidraw.com/changelog
- Anthropic, “Collaborate with Claude on Projects”: https://www.anthropic.com/news/projects
- MCP Apps: https://apps.extensions.modelcontextprotocol.io/
- Zsolt Viczián, “Sketchnoting for PKM”: https://www.zsolt.blog/2021/07/sketchnoting-for-pkm.html
- Zsolt Viczián, “Sketchnoting a Book in Obsidian”: https://www.zsolt.blog/2021/07/sketchnoting-book-in-obsidian.html
- Zsolt Viczián, “Mind mapping with Excalidraw in Obsidian”: https://www.zsolt.blog/2021/09/mind-mapping-with-excalidraw-in-obsidian.html
- Nicole van der Hoeven, “How to use the ExcaliBrain Obsidian plugin”: https://notes.nicolevanderhoeven.com/system/cards/How%2Bto%2Buse%2Bthe%2BExcaliBrain%2BObsidian%2Bplugin
- MCP Apps overview and lifecycle: https://apps.extensions.modelcontextprotocol.io/api/documents/overview.html
