# OpenBotStack V1 Implementation Stories

> TDD Approach: RED → GREEN → REFACTOR

## Status Legend
- ✅ Done (interface defined, tests pass)

---

## Epic 1: Core Interfaces [16h] — ✅ Complete

| ID | Story | Status | Est | AC |
|----|-------|--------|-----|-----|
| E1.1 | Skill interface | ✅ | 2h | Interface compiles, no Execute() |
| E1.2 | SkillRegistry interface | ✅ | 2h | Register/Get/List methods |
| E1.3 | AgentStateMachine interface | ✅ | 2h | 7 states, bounded reflection |
| E1.4 | MemoryManager interface | ✅ | 2h | Short/Long term, vector search |
| E1.5 | State transition tests | ✅ | 2h | 100% transition coverage |
| E1.6 | ContextAssembler interface | ✅ | 2h | Profile + memory → context |
| E1.7 | SkillExecutor interface (in core) | ✅ | 2h | Defined, impl in runtime |
| E1.8 | CircuitBreaker interface | ✅ | 2h | Open/Half/Closed states |

---

## Epic 2: Model Abstraction [12h] — ✅ Complete

| ID | Story | Status | Est | AC |
|----|-------|--------|-----|-----|
| E2.1 | CapabilityType enum | ✅ | 1h | 5 capabilities defined |
| E2.2 | ModelProvider interface | ✅ | 2h | ID, Capabilities, Generate |
| E2.3 | ModelRouter interface | ✅ | 2h | Route by capability + constraints |
| E2.4 | ModelScope provider impl | ✅ | 3h | Qwen models, OpenAI-compatible |
| E2.5 | Provider registry | ✅ | 2h | Multi-provider registration |

---

## Epic 3: Wasm Skill Loader [10h] — ✅ Complete

| ID | Story | Status | Est | AC |
|----|-------|--------|-----|-----|
| E3.1 | SkillManifest struct | ✅ | 2h | YAML parsing, validation |
| E3.2 | WasmModule interface | ✅ | 2h | Load, Execute, Close |
| E3.3 | Host API: LLM.Generate | ✅ | 3h | Call model from Wasm |
| E3.4 | Host API: KV.Get/Set | ✅ | 2h | In-memory adapter |
| E3.5 | Host API: HTTP.Fetch | ✅ | 2h | Sandboxed HTTP with URL allowlist |

---

## Epic 4: Rate Limiting [6h] — ✅ Complete

| ID | Story | Status | Est | AC |
|----|-------|--------|-----|-----|
| E4.1 | RateLimiter interface | ✅ | 2h | Allow, Consume, Remaining |
| E4.2 | QuotaConfig struct | ✅ | 1h | Tenant + User limits |
| E4.3 | Token bucket impl | ✅ | 3h | In-memory token bucket |

---

## Epic 5: Runtime Foundation [14h] — ✅ Complete

Repo: `openbotstack-runtime`

| ID | Story | Status | Est | AC |
|----|-------|--------|-----|-----|
| E5.1 | Runtime go.mod + structure | ✅ | 2h | Module initialized |
| E5.2 | SkillExecutor impl | ✅ | 3h | Timeout, cancellation |
| E5.3 | Wasm runtime (wazero) | ✅ | 3h | Load and execute |
| E5.4 | Audit logging | ✅ | 2h | In-memory structured logs |
| E5.5 | Async job queue | ✅ | 2h | Worker-based execution |
| E5.6 | SSE response streaming | ✅ | 2h | Partial results |

---

## Epic 6: Memory Subsystem [10h] — ✅ Complete

| ID | Story | Status | Est | AC |
|----|-------|--------|-----|-----|
| E6.1 | In-memory short-term impl | ✅ | 2h | Session-scoped |
| E6.2 | Milvus long-term stub | ✅ | 3h | Interface + stub |
| E6.3 | Embedding integration | ✅ | 2h | LocalEmbedder (hash-based) |
| E6.4 | Memory summarization | ✅ | 3h | LLMSummarizer (concatenation) |

---

## Epic 7: Web Channel MVP [10h] — ✅ Complete

Repo: `openbotstack-runtime`

| ID | Story | Status | Est | AC |
|----|-------|--------|-----|-----|
| E7.1 | REST API scaffold | ✅ | 2h | POST /v1/chat, GET /v1/sessions/{id}/history |
| E7.2 | Chat message handler | ✅ | 3h | Request → Agent → Response flow |
| E7.3 | React chat UI | ✅ | 3h | Input, messages, skill tag, execution status |
| E7.4 | E2E integration test | ✅ | 2h | Health, Chat, Session, Validation |

---

## Summary

| Epic | Total | Done |
|------|-------|------|
| E1: Core | 8 | 8 |
| E2: Model | 5 | 5 |
| E3: Wasm | 5 | 5 |
| E4: Rate Limit | 3 | 3 |
| E5: Runtime | 6 | 6 |
| E6: Memory | 4 | 4 |
| E7: Channel | 4 | 4 |
| **V1 Total** | **35** | **35** |
