# ADR-007: Skill Categories

**Status:** Accepted  
**Date:** 2026-02-07

## Context

OpenBotStack must explicitly support multiple types of capabilities with different execution models, security properties, and governance requirements. This ADR formalizes the THREE distinct skill categories.

---

## Decision: Three Skill Categories

### 1. Declarative Skill (Prompt-Based)

**Definition:**  
A capability expressed entirely in natural language, executed by the LLM without code.

**Execution Model:**
```
Input → System Prompt + User Input → LLM → Output
```

**Properties:**
| Property | Value |
|----------|-------|
| Execution | LLM-only, no code |
| Determinism | Non-deterministic |
| Sandbox | Not applicable |
| Audit | Prompt + response logged |
| Permissions | None (no external access) |

**When to Use:**
- Cognitive tasks (summarization, translation, classification)
- Conversational responses
- Format transformation

**When NOT to Use:**
- External API calls required
- Deterministic output required
- Side effects needed

**Example:**
```yaml
id: core/summarize
type: declarative
prompt: |
  Summarize the following text in 3 bullet points.
  Be concise and factual.
```

---

### 2. LLM-Assisted Skill (Code + LLM)

**Definition:**  
A Wasm module that contains business logic AND calls the LLM via Host API.

**Execution Model:**
```
Input → Wasm Code → [Host API: LLM.Generate] → Wasm Code → Output
```

**Properties:**
| Property | Value |
|----------|-------|
| Execution | Wasm sandbox + LLM calls |
| Determinism | Partially deterministic (code is, LLM isn't) |
| Sandbox | Wasm with resource limits |
| Audit | Code execution + LLM calls logged |
| Permissions | Declared in manifest |

**When to Use:**
- Business logic with AI reasoning
- Structured output with validation
- Multi-step workflows

**When NOT to Use:**
- Pure cognitive tasks (use Declarative)
- No AI reasoning needed (use Deterministic)

**Example:**
```go
func Execute() {
    input := getInput()
    
    // Business logic
    if input.Amount > 10000 {
        response := hostLLM.Generate("Explain high-value transaction...")
        setOutput(response)
    } else {
        setOutput("Approved automatically")
    }
}
```

---

### 3. Deterministic Skill (Pure Code)

**Definition:**  
A Wasm module with pure code logic, no LLM calls.

**Execution Model:**
```
Input → Wasm Code → Output
```

**Properties:**
| Property | Value |
|----------|-------|
| Execution | Wasm sandbox only |
| Determinism | Fully deterministic |
| Sandbox | Wasm with resource limits |
| Audit | Code execution logged |
| Permissions | Declared in manifest |

**When to Use:**
- Calculations, validations
- Data transformations
- Security-critical operations
- Governance/compliance checks

**When NOT to Use:**
- Natural language understanding needed
- Flexible reasoning required

**Example:**
```go
func Execute() {
    input := getInput()
    
    // Pure calculation
    tax := input.Amount * 0.15
    setOutput(map[string]float64{"tax": tax})
}
```

---

## Selection Flow

```
     ┌─────────────────────────────────────────┐
     │         Agent receives request          │
     └─────────────────┬───────────────────────┘
                       │
          ┌────────────▼────────────┐
          │   Needs external data   │──Yes──► LLM-Assisted
          │   or side effects?      │         or Deterministic
          └────────────┬────────────┘
                       │ No
          ┌────────────▼────────────┐
          │   Needs LLM reasoning?  │──Yes──► Declarative
          └────────────┬────────────┘         or LLM-Assisted
                       │ No
          ┌────────────▼────────────┐
          │   Pure computation?     │──Yes──► Deterministic
          └────────────┬────────────┘
                       │ No
                       ▼
              Agent direct response
```

---

## What is NOT a Skill

The following are explicitly NOT Skills:

| Concept | Why Not |
|---------|---------|
| **Prompt** | A prompt is input to a Declarative Skill, not a Skill itself |
| **Tool** | A low-level operation (HTTP, DB); Skills compose tools |
| **MCP Server** | External service protocol; Skills may use MCP internally |
| **Workflow** | Orchestration of multiple Skills; higher abstraction |
| **Agent** | Stateful orchestrator; uses Skills, is not a Skill |

---

## Consequences

1. All capabilities must be classified into one of three categories
2. Manifest must declare skill type
3. Runtime applies appropriate execution model per type
4. Audit logs include skill type for governance
