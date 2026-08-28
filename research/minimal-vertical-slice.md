# Minimal EOKS vertical slice

A recurring risk in the EOKS work is designing an impressive architecture before demonstrating that the abstraction is useful. The first prototype should therefore be deliberately small.

## Proposed workload

Use one concrete software-engineering task on a real repository, such as diagnosing and implementing a non-trivial bug fix.

The workload should require enough repository understanding to make context selection meaningful and enough verification to produce an objective signal.

## Minimal system

```text
                task
                  |
                  v
             task planner
                  |
        +---------+---------+
        |                   |
        v                   v
 context compiler      model router
        |                   |
        +---------+---------+
                  |
                  v
               model
                  |
                  v
             code change
                  |
          +-------+-------+
          |               |
        tests        static analysis
          |               |
          +-------+-------+
                  |
                  v
              evaluator
                  |
                  v
               outcome
```

Memory can initially be implemented as a small versioned store of project decisions and previous validated findings. It does not need a sophisticated vector database for the first experiment.

## What to measure

For each run capture:

- task success;
- correctness;
- tests passed;
- verification findings;
- model used;
- context blocks selected;
- context token count;
- tool calls;
- latency;
- cost;
- retries;
- escalation;
- final evidence.

## Experiments

### Experiment 1: context ablation

Hold model and task constant. Remove or add individual context blocks and measure outcome differences.

### Experiment 2: model routing

Run the same task corpus against several models. Determine whether task characteristics predict a useful model choice.

### Experiment 3: deterministic evidence

Compare LLM-only execution against LLM + targeted analysis.

### Experiment 4: memory

Repeat related tasks with and without retained project knowledge.

### Experiment 5: control policy

Compare a fixed agent workflow with a policy that can decide when to retrieve, verify, retry or escalate.

## The key baseline

Every EOKS experiment needs a baseline. The strongest baseline is not “no AI”; it is a **well-engineered conventional agent runtime** using the same models and tools without the EOKS coordination layer.

If EOKS cannot beat that baseline on a meaningful metric, the additional architecture is not justified.

## What success would look like

A convincing first result could be modest:

> For a defined class of coding workloads, EOKS achieves the same correctness with lower cost, or higher correctness at comparable cost, by selecting context, models and verification dynamically.

That would justify building the next layer. A diagram or a sophisticated repository structure would not.