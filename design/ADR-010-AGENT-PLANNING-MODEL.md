# ADR-010: Agent Planning Model

**Status:** Accepted  
**Date:** 2026-02-07

## Context

The Agent is the orchestrator that combines capabilities to fulfill user requests. This ADR defines how Agents plan and execute using the three skill categories.

---

## Agent Architecture

```
┌─────────────────────────────────────────────────────┐
│                      AGENT                           │
│  ┌───────────────┐  ┌───────────────┐  ┌─────────┐  │
│  │ State Machine │  │ Skill Selector│  │ Memory  │  │
│  └───────────────┘  └───────────────┘  └─────────┘  │
└───────────▲────────────────▲────────────────▲───────┘
            │                │                │
     7-State Model    SkillRegistry     Context
```

---

## Planning Flow

```
User Input
    │
    ▼
┌──────────────────────────────────────────────────┐
│  1. INIT: Parse input, establish context         │
└──────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────┐
│  2. PLANNING: Determine required capabilities    │
│     - List available Skills from Registry        │
│     - Match Skills to intent                     │
│     - Order by priority                          │
└──────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────┐
│  3. SKILL_SELECTION: Choose skill category       │
│     - Deterministic if pure calculation          │
│     - Declarative if cognitive task              │
│     - LLM-Assisted if code + reasoning needed    │
└──────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────┐
│  4. EXECUTION: Execute via SkillExecutor         │
│     - Sandbox execution                          │
│     - Timeout enforcement                        │
│     - Resource limits                            │
└──────────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────────┐
│  5. RESPONSE: Format and return result           │
└──────────────────────────────────────────────────┘
```

---

## Skill Selection Logic

```go
func selectSkillType(intent Intent, availableSkills []Skill) Skill {
    // Priority 1: Deterministic for safety-critical operations
    if intent.RequiresDeterminism || intent.IsCompliance {
        return findDeterministicSkill(availableSkills)
    }
    
    // Priority 2: LLM-Assisted for business logic + reasoning
    if intent.RequiresBusinessLogic && intent.RequiresReasoning {
        return findLLMAssistedSkill(availableSkills)
    }
    
    // Priority 3: Declarative for pure cognitive tasks
    if intent.IsCognitive {
        return findDeclarativeSkill(availableSkills)
    }
    
    // Fallback: Direct LLM response
    return nil
}
```

---

## Combining Skill Types

Single request may require multiple skills:

### Example: Expense Report Analysis

```
User: "Analyze this receipt and calculate tax"

Agent Plan:
1. core/extract-receipt (LLM-Assisted)
   - Vision + text extraction
   - Business logic for field identification

2. core/tax-calculator (Deterministic)
   - Pure calculation
   - Auditable, reproducible

3. core/summarize (Declarative)
   - Natural language summary
   - No external access
```

---

## Decision Constraints

### Must Always:
- Check permissions before execution
- Log all skill invocations
- Enforce timeouts
- Validate outputs against schema

### Must Never:
- Execute unapproved skills
- Skip audit logging
- Allow unbounded execution
- Trust skill output without validation

### Bounded Reflection (V2):
- Maximum 3 reflection loops
- Each loop reduces remaining budget
- Explicit stop conditions

---

## State Machine Integration

| State | Skill Activity |
|-------|---------------|
| INIT | Load context, no skills |
| PLANNING | Query SkillRegistry |
| THINKING | Optional reasoning (no execution) |
| EXECUTING | SkillExecutor.Execute() |
| REFLECTING | Review output, decide next (V2) |
| RESPONDING | Format final response |
| COMPLETE | Cleanup, audit finalization |

---

## Consequences

1. Agent ALWAYS consults SkillRegistry before execution
2. Skill selection is auditable and deterministic
3. Multi-skill plans follow PLANNING → EXECUTION pattern
4. Reflection is bounded to prevent infinite loops
