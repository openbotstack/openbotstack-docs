# OpenBotStack TODO Registry

> Generated: 2026-03-11
> Scope: `openbotstack-core`, `openbotstack-runtime`

---

## 1. Stub Implementations (In-Memory → Production)

These components have working APIs but use in-memory storage instead of real backends.

| # | Component | File | Current | Production Target |
|---|-----------|------|---------|-------------------|
| S1 | **PGAuditLogger** | `runtime/logging/execution_logs/pg_logger.go` | In-memory `[]AuditEntry` | PostgreSQL |
| S2 | **RedisLimiter** | `runtime/ratelimit/redis_limiter.go` | In-memory token buckets | Redis |
| S3 | **MilvusStore** | `runtime/memory/milvus_store.go` | In-memory `map[string]Document` | Milvus / pgvector |
| S4 | **RedisMemoryStore** | `runtime/memory/redis_store.go` | In-memory `map[string]*Entry` | Redis |
| S5 | **LocalEmbedder** | `runtime/memory/embedder.go` | FNV hash-based 384-dim vectors | OpenAI / sentence-transformers |
| S6 | **LLMSummarizer** | `runtime/memory/embedder.go` | String concatenation (first 50 chars) | Actual LLM summarization call |

---

## 2. Stub Providers (Interface-Compliant Shells)

LLM providers that compile but do not make real API calls.

| # | Provider | File | TODO Comment |
|---|----------|------|--------------|
| P1 | **ClaudeProvider** | `core/ai/providers/providers.go:37` | `TODO: Implement actual Claude API call` |
| P2 | **OpenAICompatibleProvider** | `core/ai/providers/providers.go:77` | `TODO: Implement actual OpenAI API call` |
| P3 | **VLLMProvider** | `core/ai/providers/providers.go:86` | `TODO: Implement` |

---

## 3. Stub Host APIs (Wasm Sandbox)

Host APIs exposed to Wasm skills that return placeholder responses.

| # | API | File | TODO Comment |
|---|-----|------|--------------|
| H1 | **LLMGenerate** | `core/registry/skills/hostapi.go:61` | `TODO: Integrate with model router` |
| H2 | **HTTPFetch** | `core/registry/skills/hostapi.go:105` | `TODO: Implement actual HTTP client with sandboxing` |
| H3 | **WasmModule.Execute** | `core/registry/skills/wasm.go:64` | `TODO: Use wasmtime/wasmer to actually load` |

---

## 4. Interface-Only Definitions (No Implementation)

Interfaces defined in `openbotstack-core` without concrete implementations.

| # | Interface | File | Purpose |
|---|-----------|------|---------|
| I1 | **PolicyEnforcer** | `core/control/policies/policy.go` | Pre-execution policy checks |
| I2 | **AuditEmitter** | `core/audit/emitter.go` | Publish audit events |
| I3 | **ContextAssembler** | `core/context/assembler.go` | Build LLM prompts (persona + memory + request) |
| I4 | **MemoryManager** | `core/memory/abstraction/manager.go` | Unified memory operations |
| I5 | **QuotaStore** | `core/access/ratelimit/limiter.go` | Quota persistence |

---

## 5. Inline TODO Comments

| # | File | Line | Comment |
|---|------|------|---------|
| T1 | `runtime/api/router.go` | 150 | `TODO: Retrieve actual history` |
| T2 | `runtime/api/handler.go` | 57 | `TODO: Integrate with agent execution and model providers` |
| T3 | `runtime/agent/agent.go` | 80 | `TODO: inject from session` (ConversationHistory) |
| T4 | `runtime/memory/milvus_store.go` | 70 | `TODO: Actual vector similarity` |
| T5 | `runtime/ratelimit/redis_limiter.go` | 27 | `TODO: Replace with actual Redis client` |

---

## 6. Placeholder Structs (No Behavior)

| # | Struct | File | Notes |
|---|--------|------|-------|
| PH1 | **User** | `core/access/auth/user.go` | ID, TenantID, Name — no methods |
| PH2 | **Tenant** | `core/access/tenant/tenant.go` | ID, Name — no methods |
| PH3 | **JSONSchema** | `core/registry/skills/skill.go:14` | `placeholder type; use proper JSON Schema library` |

---

## 7. Empty Placeholder Directories

Directories that exist for planned features but contain no code.

| # | Directory | Planned Feature |
|---|-----------|-----------------|
| D1 | `core/access/http/` | HTTP middleware (auth, CORS, validation) |
| D2 | `core/ai/embeddings/` | Embedding model abstraction |
| D3 | `core/ai/retrieval/` | RAG / retrieval pipeline |
| D4 | `core/ai/safety/` | AI safety / content filtering |
| D5 | `core/memory/session/` | Session-scoped memory abstraction |
| D6 | `core/memory/vector/` | Vector store abstraction |
| D7 | `runtime/toolrunner/tool_invocation/` | Tool invocation pipeline |

---

## Summary

| Category | Count |
|----------|-------|
| Stub implementations | 6 |
| Stub providers | 3 |
| Stub host APIs | 3 |
| Interface-only definitions | 5 |
| Inline TODOs | 5 |
| Placeholder structs | 3 |
| Empty directories | 7 |
| **Total items** | **32** |
