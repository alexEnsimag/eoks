# OpenWolf

OpenWolf is an interesting adjacent system for EOKS because it targets a concrete failure mode of coding-agent workflows: repeated reconstruction of repository context across sessions.

## Why it is relevant

The problem is not simply that an agent lacks a sufficiently large context window. A new session has to rediscover facts about the repository, often by repeatedly traversing files and reconstructing relationships that were already established in previous work.

OpenWolf approaches this through local, persistent repository-oriented state and hooks around agent file interactions. The important idea for EOKS is the **persistence of derived project knowledge close to the development environment**.

## What OpenWolf should be considered

For EOKS purposes, OpenWolf is best understood as a combination of:

- repository indexing / structural summarization;
- persistent project notes or memory;
- interaction-aware context optimization;
- automatic maintenance driven by agent activity.

It is therefore closer to a **knowledge extraction and context-optimization layer** than to a complete knowledge base or control plane.

## What it does not solve by itself

A persistent summary is not automatically trustworthy knowledge. Several problems remain:

- summaries can become stale after code changes;
- generated notes can contain incorrect inferences;
- there may be no explicit distinction between observation and interpretation;
- provenance may be weaker than the source evidence;
- retrieval policy is still needed to decide what enters a task's context;
- knowledge should remain useful if the agent or model is replaced.

Consequently, EOKS should not make an OpenWolf-like artifact the unquestioned source of truth.

## Security considerations

A local, file-oriented implementation can have a relatively small data-exfiltration surface, but "local" should not be equated with "automatically safe". Any tool installed into an agent workflow should be reviewed for:

- network access;
- commands executed by hooks;
- files read and written;
- permissions required;
- generated state that may contain source code, secrets, or sensitive project information;
- behavior changes caused by agent lifecycle hooks.

The right EOKS abstraction is therefore a **policy-governed evidence provider**, not an implicitly trusted hook.

## Community / maturity signal

OpenWolf is useful as emerging prior art, but it should be treated as an early ecosystem signal rather than an established standard. Its value for EOKS is primarily architectural: it demonstrates that automatically maintained local project state can attack the repeated-context problem without requiring a large external knowledge service.

Before depending on it operationally, EOKS should measure it rather than relying on claimed token savings or anecdotal reports.

## Evaluation questions

A useful experiment would compare Claude Code workflows with and without an OpenWolf-like layer:

1. tokens spent on repository discovery;
2. time to first useful action;
3. repeated file reads;
4. retrieval precision and recall;
5. task success rate;
6. regression rate caused by stale or incorrect memory;
7. maintenance cost after substantial repository changes;
8. performance when switching models or agents.

The last point is particularly important: a knowledge layer is more valuable if it survives model changes and improves multiple agents rather than becoming optimized for one CLI's behavior.

## EOKS interpretation

OpenWolf suggests a useful EOKS resource boundary:

```text
repository + agent activity
            |
        observation
            |
     derived knowledge
       /          \
   candidate      evidence
       \          /
        validation
             |
      durable knowledge
             |
        task retrieval
             |
          context
```

The strongest idea to carry forward is not OpenWolf's exact file format or hook mechanism. It is the idea that **project knowledge should be incrementally maintained instead of reconstructed from zero for every reasoning session**.
