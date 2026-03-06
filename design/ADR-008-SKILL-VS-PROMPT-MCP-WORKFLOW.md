# ADR-008: Skill vs Prompt vs MCP vs Workflow

**Status:** Accepted  
**Date:** 2026-02-07

## Context

OpenBotStack needs precise terminology to prevent conceptual drift. This ADR establishes clear boundaries between Skill, Prompt, MCP, and Workflow.

---

## Definitions

### 1. Skill

**Definition:**  
A governed, self-contained unit of capability with declared inputs, outputs, permissions, and execution constraints.

**Properties:**
- Has a manifest (metadata, schema, permissions)
- Executed in sandbox (Wasm or LLM)
- Produces auditable execution trace
- Registered in SkillRegistry

**Examples:**
- `core/summarize` (declarative)
- `core/sentiment` (llm-assisted)
- `core/tax-calculator` (deterministic)

---

### 2. Prompt

**Definition:**  
Input text sent to an LLM. A prompt is NOT a Skill — it is an argument to a Skill or ModelProvider.

**Properties:**
- No standalone execution
- No permissions model
- Not registered in registry
- Part of a Declarative Skill's implementation

**When Confused:**
| Incorrect | Correct |
|-----------|---------|
| "Run this prompt" | "Execute this Declarative Skill" |
| "Register a prompt" | "Register a Declarative Skill with this prompt" |

---

### 3. MCP (Model Context Protocol)

**Definition:**  
A protocol for LLMs to communicate with external services. MCP servers are NOT Skills.

**Relationship to Skills:**
```
Skill MAY use MCP internally
Skill is NOT an MCP server
Agent MAY call MCP servers directly
```

**Key Differences:**
| Aspect | Skill | MCP |
|--------|-------|-----|
| Governance | Manifest, permissions | None built-in |
| Sandbox | Wasm/LLM | External process |
| Audit | Built-in | External |
| Registry | SkillRegistry | Tool definition |

---

### 4. Workflow

**Definition:**  
Orchestration of multiple Skills in sequence or parallel, with branching logic.

**Relationship to Skills:**
```
Workflow orchestrates → Skills execute
```

**Properties:**
- Higher-level abstraction
- References Skills by ID
- May include human approval gates
- V2 scope (not V1)

**Example:**
```yaml
workflow: expense-approval
steps:
  - skill: core/extract-receipt-data
  - skill: core/validate-expense
  - if: amount > 1000
      skill: core/request-manager-approval
  - skill: core/submit-to-finance
```

---

## Decision Matrix

| Concept | Definition | Registered | Sandboxed | Audited | V1 |
|---------|------------|------------|-----------|---------|-----|
| **Skill** | Unit of capability | ✅ | ✅ | ✅ | ✅ |
| **Prompt** | LLM input text | ❌ | N/A | partial | N/A |
| **MCP** | Service protocol | ❌ | ❌ | external | ✅ |
| **Workflow** | Orchestration | ✅ | N/A | ✅ | ❌ V2 |

---

## Consequences

1. Never call a prompt a "Skill"
2. MCP servers are external tools, not Skills
3. Workflows are V2 — build Skills first
4. All capabilities must be wrapped as Skills for governance
