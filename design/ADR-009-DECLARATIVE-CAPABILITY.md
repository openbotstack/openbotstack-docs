# ADR-009: Declarative Capability Design

**Status:** Accepted  
**Date:** 2026-02-07

## Context

Declarative capabilities (prompt-based) are executed by the LLM without code. This ADR defines how they are represented, why they are distinct from full Skills, and their governance implications.

---

## Decision

### Declarative Skills ARE Skills

Declarative capabilities are represented as Skills with `type: declarative` in their manifest. They are NOT treated as "lesser" Skills.

```yaml
id: core/summarize
type: declarative  # Key distinction
prompt: |
  ...
```

### Structure

```
declarative-skill/
├── manifest.yaml    # Metadata + prompt template
└── README.md        # Documentation
```

No `main.go` or `.wasm` file needed.

---

## Why Declarative Skills Are Not "Just Prompts"

| Aspect | Raw Prompt | Declarative Skill |
|--------|------------|-------------------|
| Registry | ❌ | ✅ Registered |
| Schema | ❌ | ✅ Input/Output defined |
| Governance | ❌ | ✅ Audit logged |
| Discovery | ❌ | ✅ Agent can select |
| Versioning | ❌ | ✅ manifest.version |

---

## Execution Model

```
┌─────────────────────────────────────────────────┐
│               Agent                              │
│   ┌─────────────────────────────────────────┐   │
│   │  1. Select Skill from Registry          │   │
│   │  2. Load manifest (prompt template)     │   │
│   │  3. Template prompt with input          │   │
│   │  4. Call ModelProvider.Generate()       │   │
│   │  5. Parse output against schema         │   │
│   │  6. Log audit event                     │   │
│   └─────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

---

## Governance Properties

### Permissions
Declarative Skills have no external access. They may NOT:
- Call HTTP endpoints
- Access KV store
- Read secrets
- Modify state

They only receive input and produce output via LLM.

### Audit
Every execution logs:
- Skill ID and version
- Input (potentially redacted)
- Model used
- Output
- Duration

### Safety
- Prompt injection mitigation via template isolation
- Output validation against schema
- No side effects possible

---

## V2 Considerations

| Feature | V1 | V2 |
|---------|----|----|
| Prompt template | Static | Dynamic (from memory) |
| Schema validation | Optional | Required |
| Prompt versioning | Manual | Automatic |
| A/B testing | ❌ | ✅ |

---

## Consequences

1. Declarative Skills are first-class citizens
2. Agent discovers them like any other Skill
3. Governance is enforced at Skill level
4. Prompt templates are encapsulated, not exposed directly
