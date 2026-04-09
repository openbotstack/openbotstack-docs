# ADR-012: Dual Bounded Loop Kernel

**Status:** Accepted  
**Date:** 2026-04-09

## Context

The current execution model in OpenBotStack is a **single-pass pipeline**:

```
Agent.HandleMessage → Planner.Plan → Executor.ExecutePlan → Response
```

The `WorkflowRunner.Run` iterates steps sequentially without re-planning, observation-based feedback, or stop-condition evaluation. This architecture has fundamental limitations:

1. **No iterative reasoning.** The agent plans once and executes. It cannot observe intermediate results and adjust.
2. **No multi-task orchestration.** A single user request maps to a single plan. Complex tasks requiring decomposition are not supported.
3. **No context compaction.** As execution progresses, the context window grows unbounded within a session.
4. **No memory checkpoints.** There is no mechanism to persist intermediate state during execution.
5. **No policy gates between steps.** Policy is checked before execution, not during.

Modern agentic systems (e.g., Claude Code) use a dual-loop architecture that addresses all of these limitations while maintaining bounded, auditable execution.

---

## Decision

Introduce a **Dual Bounded Loop Kernel** in `openbotstack-runtime/loop/` that replaces the flat pipeline with two nested, bounded loops.

### Outer Loop — Task/Workflow Loop

Orchestrates the high-level workflow:

```
INIT → TASK_SELECT → TASK_EXECUTE → CHECKPOINT → NEXT_TASK → DONE
```

| Responsibility | Description |
|---------------|-------------|
| Workflow orchestration | Sequenced task execution |
| Task decomposition | Receives pre-decomposed `[]TaskInput` from caller |
| Memory checkpoints | Persists intermediate state via `Checkpoint` interface |
| Policy checkpoints | Enforces governance gates via `PolicyCheckpoint` interface |
| Stop conditions | Max workflow steps (5), max session runtime (30s) |

### Inner Loop — Reasoning Turn Loop

Executes a single task through iterative reasoning:

```
TURN_INIT → PLAN → ACT → OBSERVE → CHECK_STOP → NEXT_TURN → DONE
```

| Responsibility | Description |
|---------------|-------------|
| LLM planning | Invokes `planner.ExecutionPlanner` each turn |
| Tool execution | Delegates to `toolrunner.ToolRunner` |
| Observation | Collects results, builds context for re-planning |
| Context compaction | Truncates turn history to keep context bounded |
| Stop conditions | Max turns (8), max tool calls (20), max turn runtime (15s) |

### Architecture Diagram

```
┌───────────────────────────────────────────────────────────────┐
│                    OUTER LOOP (Task/Workflow)                  │
│                                                               │
│  INIT → TASK_SELECT → TASK_EXECUTE → CHECKPOINT →             │
│                        NEXT_TASK → DONE                       │
│                                                               │
│  Bounds: max_workflow_steps=5, max_session_runtime=30s        │
│                                                               │
│  ┌───────────────────────────────────────────────────────────┐│
│  │              INNER LOOP (Reasoning Turn)                  ││
│  │                                                           ││
│  │  TURN_INIT → PLAN → ACT → OBSERVE →                      ││
│  │              CHECK_STOP → NEXT_TURN → DONE                ││
│  │                                                           ││
│  │  Bounds: max_turns=8, max_tool_calls=20,                  ││
│  │          max_turn_runtime=15s                             ││
│  └───────────────────────────────────────────────────────────┘│
└───────────────────────────────────────────────────────────────┘
```

### Interface Reuse

| Existing Interface | Used By | Purpose |
|-------------------|---------|---------|
| `planner.ExecutionPlanner` | Inner Loop (PLAN) | LLM-based plan generation per turn |
| `toolrunner.ToolRunner` | Inner Loop (ACT) | Tool execution |
| `execution.ExecutionContext` | Both loops | Request-scoped state tracking |
| `execution.ExecutionLogger` | Both loops | Structured audit events |

### State Boundedness

All loops are bounded by:
- **Step/turn counters** (hard limits, not configurable at runtime)
- **Wall-clock timeouts** (enforced via `context.WithTimeout`)
- **Context cancellation** (propagated from caller)
- **Explicit stop signals** (planner returns empty plan = stop)

There are NO infinite loops, NO unbounded reflection, and NO self-modifying behavior.

---

## Alternatives Considered

### 1. Extend existing `WorkflowRunner`

Add re-planning and observation to the existing flat loop. Rejected because:
- Mixing two levels of abstraction (task orchestration + reasoning) in one loop creates complexity.
- The existing `WorkflowRunner` uses `SkillExecutor.ExecutePlan`, which is a batch executor, not a turn-based reasoner.
- Stop conditions for tasks vs. turns have different semantics.

### 2. Single loop with reflection step

Add a REFLECTING state to the existing 7-state agent machine (ADR-010). Rejected because:
- A reflection step within the same loop cannot distinguish between "I need to adjust within this task" and "I need to move to the next task."
- Context compaction and memory checkpoints operate at different granularities.

### 3. External orchestrator (e.g., LangGraph)

Use an external orchestration framework. Rejected because:
- Violates the stateless, self-contained execution model.
- Introduces a heavyweight dependency.
- Reduces auditability.

---

## Consequences

1. The system gains iterative reasoning capability — the agent can plan, act, observe, and re-plan within bounded turns.
2. Multi-task workflows are first-class — the outer loop orchestrates sequences of tasks with checkpoints.
3. All execution remains bounded, auditable, and deterministic at the control-plane level.
4. Existing `workflow/` and `executor/` packages are NOT modified — the new `loop/` package is an opt-in alternative.
5. A future task decomposer (LLM-based) can be layered on top of the outer loop in V2.
6. The `loop/` package introduces new interfaces (`InnerLoop`, `OuterLoop`, `Checkpoint`, `PolicyCheckpoint`, `ContextCompactor`) that may be promoted to `openbotstack-core` in a future ADR.

---

## References

- [ADR-010: Agent Planning Model](./ADR-010-AGENT-PLANNING-MODEL.md)
- [SYSTEM_DESIGN.md](./SYSTEM_DESIGN.md) — D9 section
- [AI_CONTRACT.md (runtime)](../../openbotstack-runtime/AI_CONTRACT.md)
