# Workflow Development Guide

While **Skills** are used to perform a single governed capability, **Workflows** orchestrate multiple skills deterministically to handle multi-step domain processes.

## 1. Concept

Workflows exist in the **Application Plane**. They solve the problem of complex orchestration without resorting to autonomous, non-deterministic agent loops.

In OpenBotStack, you never tell an LLM: *"Here is a goal, figure out the steps to achieve it."*

Instead, you use a Workflow to explicitly define:
1. State 1: Gather facts
2. State 2: Generate summary
3. State 3: Request human approval
4. State 4: Save to database

## 2. Structure 

Workflows are defined as files (often YAML or JSON) representing state machines.

```text
workflows/
├── nurse_shift_handover.yaml
└── sepsis_screening.yaml
```

### Example: State-Machine Driven Workflow

```yaml
name: nurse_shift_handover
version: 1.0.0
description: Orchestrates the shift handover process

triggers:
  - event: shift_end

states:
  gather_vitals:
    type: execute_skill
    skill: retrieve_patient_vitals
    next: gather_notes
    fallback: error_state

  gather_notes:
    type: execute_skill
    skill: summarize_shift_notes
    next: generate_handover

  generate_handover:
    type: execute_skill
    skill: build_sbar_report
    next: end

  error_state:
    type: terminate
    status: failed
```

## 3. Orchestration Patterns

### Sequential Execution
The simplest pattern. A workflow moves strictly from State A -> State B -> State C.

### Human-in-the-Loop (HITL)
Workflows can pause execution, yield state back to the UI, and wait for human review.

```yaml
  review_handover:
    type: wait_for_human
    timeout: 1h
    on_approve: save_record
    on_reject: regenerate_handover
```

### Parallel Execution (Scatter-Gather)
A workflow can launch multiple read-only deterministic tools or disjoint skills simultaneously and block until all results are gathered.

## 4. Why Use Workflows?

You might be tempted to use a large LLM context window to do all of this in one shot. However, OpenBotStack enforces workflows because:
- **Auditability:** We can log exactly which state failed and what the input/output of that state was.
- **Governance:** If the `gather_notes` skill violates a policy, the workflow stops safely.
- **Determinism:** The execution is bounded by the state machine at the **Control Plane**, guaranteeing no infinite loops.
