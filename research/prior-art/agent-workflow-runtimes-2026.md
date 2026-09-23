# Agent workflow runtimes and EOKS

## Why this matters

Recent investigation of "AI agent engineer" tooling clarifies a part of EOKS that was already present but not named precisely enough: **agent workflow runtimes are execution substrates for explicit loops, graphs, state and tool use**.

Frameworks such as LangGraph, CrewAI, Microsoft Agent Framework, Google ADK and the OpenAI Agents SDK are not model-training infrastructure. They sit above model APIs and below the application/control-plane concerns of a production agent system. They make it easier to express and run workflows in which nodes may be model calls, deterministic functions, tools, other agents, verification steps or human gates.

This is directly relevant to the EOKS concepts of **workflow, loop engineering, graph engineering, execution, Run, conductor and minimum sufficient coordination**.

## Current ecosystem signal

The 2026 framework landscape is converging around several execution models:

- **Graph/state-machine orchestration** — explicit nodes, edges and state; LangGraph is a prominent example.
- **Role/task orchestration** — agents are assigned responsibilities and collaborate; CrewAI is a prominent example.
- **Agent-loop / handoff orchestration** — a lighter loop where the model chooses tools or hands off to another agent; OpenAI Agents SDK is an example.
- **Event/workflow-oriented runtimes** — workflows combine agent steps with deterministic event-driven execution; Google ADK and Microsoft Agent Framework expose this broader direction.

AutoGen is important prior art for conversational multi-agent execution, but Microsoft has moved new development toward Microsoft Agent Framework. EOKS should therefore study AutoGen for its execution patterns without treating it as the preferred current dependency.

The ecosystem evidence also reinforces that a framework is not the whole production stack. Frameworks address application-level orchestration; observability/evaluation, security, persistence, deployment and infrastructure remain separate concerns.

## EOKS interpretation

The key mapping is:

```
LLM / model API
      |
      v
agent workflow runtime
      |
      +-- state
      +-- loops
      +-- branches
      +-- tools
      +-- handoffs
      +-- checkpoints / resume
      +-- human gates
      |
      v
agent application
      |
      v
production platform / infrastructure
```

EOKS sits **across and above this execution substrate**, because its proposed control loop selects and coordinates resources, context, evidence, execution modality and topology based on workload state and policy.

Therefore:

> **An agent workflow runtime is an EOKS resource/execution substrate, not the EOKS control plane itself.**

EOKS should be able to use one, wrap one, observe one, or run a simpler deterministic workflow without one.

## Relationship to workflow and graph engineering

The framework landscape gives concrete implementation evidence for concepts already present in EOKS:

- **Loop engineering** — repeated act/observe/evaluate/retry cycles.
- **Graph engineering** — explicit dependencies, branches, parallelism and joins.
- **Workflow** — the reusable execution topology and control conditions.
- **Run** — one execution of a workload/task or subgraph.
- **Role** — a responsibility implemented by a node, agent, deterministic function or human.
- **Conductor** — decides which topology/resource/execution modality is appropriate; it does not have to be implemented by the framework.
- **Evaluation** — determines whether the execution produced sufficient evidence/outcome; framework traces are observations, not acceptance by themselves.

This validates the existing EOKS distinction between **execution graph** and **control plane** rather than requiring a new EOKS primitive.

## Frameworks versus EOKS primitives

| Concern | Agent workflow runtime | EOKS |
|---|---|---|
| Define workflow topology | Yes | Select/adapt topology |
| Execute graph/loop | Yes | Coordinate/observe execution |
| Tool calling | Yes | Treat tools as resources/providers |
| State/checkpointing | Often | Govern run/workload state |
| Human approval | Often | Policy/assurance decision |
| Context construction | Varies | First-class context compilation |
| Knowledge/memory lifecycle | Varies | Resource/knowledge lifecycle |
| Evidence-provider selection | Usually not the central concern | First-class control decision |
| Cross-run learning/promotion | Limited/varies | Core research concern |
| Outcome/evidence-based reconciliation | Usually application-specific | Core EOKS loop |
| Production observability/evaluation | Often integrated or adjacent | Consumes evaluation/observability as control evidence |

The distinction is deliberately porous: a runtime can provide some context, memory, evaluation or observability features. EOKS should model those as capabilities rather than assuming the framework boundary is universal.

## Relationship to existing EOKS tooling

This also clarifies why agent workflow runtimes belong beside, rather than inside, the harness/tooling already studied:

```
                 EOKS environment
                       |
       +---------------+----------------+
       |               |                |
   context/          harness          execution
   knowledge         hooks/skills      workflows
       |               |                |
 OpenWolf/Portal   Claude Code       LangGraph/
 Understand        cmux/Herdr        CrewAI/MAF/
 Anything          MCP              ADK/etc.
       |               |                |
       +---------------+----------------+
                       |
                 observe/evaluate
```

These are complementary mechanisms:

- **Harness mechanisms** shape the agent's local operating environment and information flow.
- **Workflow runtimes** define/execute multi-step agentic work.
- **Execution surfaces/runtimes** such as cmux or Herdr manage live processes, sessions and human attention.
- **Knowledge/context systems** determine what information is available and how it is compiled.
- **Evaluation/observability systems** record and assess what happened.
- **EOKS** is the proposed coordinating/control layer that decides how these resources should be combined for a workload.

A single product can span several of these categories; the categories describe capabilities, not vendor boundaries.

## Important design consequence

EOKS should **not build a graph runtime merely because graph runtimes exist**.

Instead, the research question is:

> Can an EOKS conductor select among a simple agent loop, deterministic workflow, graph runtime, coding-agent session, or richer multi-agent topology based on workload requirements and evidence?

That is consistent with the existing principle:

> **Use the minimum coordination structure that provides the evidence, assurance and outcome required by the workload.**

A framework should earn its place by reducing implementation cost or improving durability, inspectability, reliability or operational behavior—not by making a workflow look more agentic.

## Research / experiment implications

The highest-value next experiment is not a framework bake-off. It is a **runtime substitution experiment**:

1. Express the same small software-engineering workflow as:
   - one coding-agent run;
   - a deterministic workflow;
   - a graph-based agent runtime;
   - optionally a role-based multi-agent workflow.
2. Capture the same EOKS Run-level observations.
3. Compare outcome quality, evidence coverage, context cost, latency, retries, coordination overhead, human intervention and reproducibility.
4. Test whether the EOKS-level control decisions remain portable across runtimes.

This would test the hypothesis that **workflow topology is an EOKS control concern while the mechanism used to execute that topology is replaceable infrastructure**.

## Sources / prior art

- LangGraph / LangChain framework landscape: https://www.langchain.com/resources/ai-agent-frameworks
- CrewAI: https://docs.crewai.com/
- Microsoft Agent Framework: https://learn.microsoft.com/en-us/agent-framework/
- Google Agent Development Kit: https://google.github.io/adk-docs/
- OpenAI Agents SDK: https://openai.github.io/openai-agents-python/
- AutoGen: https://microsoft.github.io/autogen/

These sources are ecosystem evidence, not EOKS requirements.
