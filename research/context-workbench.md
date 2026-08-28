# Context workbench concept

One of the more concrete ideas from the context-quality discussions was an interactive UI for inspecting and controlling model context.

## Motivation

Today, context is often hidden inside a prompt template, agent framework or retrieval pipeline. That makes it difficult to answer basic questions:

- Why did the model receive this information?
- Why was an important fact omitted?
- Which source produced this claim?
- How much of the context is redundant?
- What changed between two runs?
- What does removing this block do to the outcome?

A workbench would make these questions observable.

## Primary abstraction: blocks

The proposed UI should expose **context blocks** rather than forcing users to manipulate a graph or raw token sequence.

A block could represent:

- a source file;
- a code symbol;
- a decision;
- an issue;
- a test result;
- a previous task result;
- external evidence;
- a hypothesis;
- a memory item.

Blocks can have relationships, but the human can reason about them as units.

## Views

Potential views include:

### Composition view

Which blocks are currently included, excluded or pinned?

### Provenance view

Where did each block come from?

### Relationship view

What dependencies or contradictions exist between blocks?

### Budget view

What is the estimated token/cost impact of each block?

### Diff view

What changed between two contexts or two model runs?

### Outcome view

How did context composition correlate with the result?

## Context package

A selected working set could become a reusable artifact:

```text
ContextPackage
  task
  blocks[]
  dependencies[]
  provenance[]
  budget
  compiler_version
  model_target
  selection_policy
```

The package should be model-portable where possible while allowing a model-specific compilation step.

## Automatic versus human selection

A particularly useful experiment is to compare automatic retrieval with human-assisted selection. The goal is not to make humans curate every prompt. It is to determine whether occasional intervention provides enough value to justify the interface.

## Graph + blocks

The UI can show a graph for discovery while retaining blocks for manipulation:

```text
             relationship graph
                    |
        +-----------+-----------+
        |           |           |
      block       block       block
        |           |           |
        +-----------+-----------+
                    |
             selected context
```

This reconciles two ideas that initially looked competing: graph-based context and human-friendly context blocks.

## EOKS significance

If successful, the workbench would be more than a prompt editor. It would be an observability and control surface for the context subsystem of EOKS.