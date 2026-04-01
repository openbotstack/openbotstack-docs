# OpenBotStack Roadmap

> Updated: 2026-03-11
> Reference: [TODO_REGISTRY.md](./TODO_REGISTRY.md) | [V1_STORIES.md](./V1_STORIES.md)

---

## Current State: V1 MVP — Demo-Ready

All 35 V1 stories complete. System runs end-to-end with in-memory stubs.
See [v1-audit.md](../v1-audit.md) for details.

---

## Phase 2: Production Foundation

**Goal:** Replace stubs with real backends. System becomes deployable for internal use.

### 2.1 Authentication & Multi-Tenancy [HIGH — blocks all production use]

| ID | Task | Repo | Refs |
|----|------|------|------|
| 2.1.1 | Implement JWT middleware in `access/http/` | core | D1, PH1, PH2 |
| 2.1.2 | Add `Authenticate()` and `Authorize()` to User/Tenant | core | PH1, PH2 |
| 2.1.3 | Wire auth middleware into runtime HTTP router | runtime | T2 |
| 2.1.4 | Implement tenant-scoped data partitioning | runtime | — |

### 2.2 Persistence Layer [HIGH — blocks audit and compliance]

| ID | Task | Repo | Refs |
|----|------|------|------|
| 2.2.1 | Replace `PGAuditLogger` stub with PostgreSQL client | runtime | S1 |
| 2.2.2 | Replace `RedisLimiter` stub with Redis client | runtime | S2, T5 |
| 2.2.3 | Replace `RedisMemoryStore` stub with Redis client | runtime | S4 |
| 2.2.4 | Implement `QuotaStore` for persistent quota config | core | I5 |

### 2.3 AI Service Layer [HIGH — blocks real LLM usage]

| ID | Task | Repo | Refs |
|----|------|------|------|
| 2.3.1 | Implement ClaudeProvider with real Anthropic API | core | P1 |
| 2.3.2 | Implement OpenAICompatibleProvider with real API | core | P2 |
| 2.3.3 | Implement VLLMProvider for self-hosted models | core | P3 |
| 2.3.4 | Wire Wasm host API `LLMGenerate` to ModelRouter | core | H1 |

### 2.4 Operational Readiness [MEDIUM]

| ID | Task | Repo | Refs |
|----|------|------|------|
| 2.4.1 | Add `/healthz` and `/readyz` endpoints | runtime | — |
| 2.4.2 | Add TLS support (built-in or reverse proxy guide) | runtime | — |
| 2.4.3 | Structured logging with `slog` levels + correlation IDs | runtime | — |
| 2.4.4 | Prometheus metrics endpoint | runtime | — |

---

## Phase 3: Intelligence Layer

**Goal:** Replace placeholder AI components. System becomes useful for real conversations.

### 3.1 Vector Memory [HIGH]

| ID | Task | Repo | Refs |
|----|------|------|------|
| 3.1.1 | Replace `MilvusStore` stub with real Milvus/pgvector | runtime | S3, T4 |
| 3.1.2 | Replace `LocalEmbedder` with real embedding model | runtime | S5 |
| 3.1.3 | Implement cosine similarity search | runtime | T4 |
| 3.1.4 | Define vector store abstraction in core | core | D6 |

### 3.2 Context & Memory Pipeline [MEDIUM]

| ID | Task | Repo | Refs |
|----|------|------|------|
| 3.2.1 | Implement `ContextAssembler` (persona + memory → prompt) | core | I3 |
| 3.2.2 | Replace `LLMSummarizer` stub with real LLM call | runtime | S6 |
| 3.2.3 | Inject `ConversationHistory` from session into agent | runtime | T3 |
| 3.2.4 | Implement session history retrieval | runtime | T1 |
| 3.2.5 | Define session memory abstraction in core | core | D5 |

### 3.3 Wasm Sandbox Maturity [MEDIUM]

| ID | Task | Repo | Refs |
|----|------|------|------|
| 3.3.1 | Replace stub `WasmModule` loader with real wazero loader | core | H3 |
| 3.3.2 | Implement sandboxed HTTP client for Wasm skills | core | H2 |
| 3.3.3 | Implement tool invocation pipeline | runtime | D7 |

---

## Phase 4: Governance & Safety

**Goal:** Enterprise-grade policy enforcement and AI safety.

### 4.1 Policy Engine [HIGH for enterprise]

| ID | Task | Repo | Refs |
|----|------|------|------|
| 4.1.1 | Implement `PolicyEnforcer` rule engine | core | I1 |
| 4.1.2 | Implement `AuditEmitter` for real-time event publishing | core | I2 |
| 4.1.3 | Implement `MemoryManager` unified interface | core | I4 |

### 4.2 AI Safety [MEDIUM for enterprise]

| ID | Task | Repo | Refs |
|----|------|------|------|
| 4.2.1 | Content filtering / guardrails in `ai/safety/` | core | D4 |
| 4.2.2 | Implement embedding abstraction in `ai/embeddings/` | core | D2 |
| 4.2.3 | Implement RAG retrieval pipeline in `ai/retrieval/` | core | D3 |

---

## Phase App: Application Plane

**Goal:** Implement domain-level skills, tools, workflows, and example applications in `openbotstack-apps`.

| ID | Task | Repo | Notes |
|----|------|------|-------|
| A.1 | Repository scaffold + AI_CONTRACT.md | apps | go.mod, Makefile |
| A.2 | Tool interface and registry | apps | tools/tool.go |
| A.3 | EHR tools (patient, vitals, labs) | apps | Stub implementations |
| A.4 | Analytics tools (risk score) | apps | Deterministic calculation |
| A.5 | Nursing skills (query, summarize, SBAR) | apps | Implements core Skill interface |
| A.6 | Workflow engine + nursing workflows | apps | Builds ExecutionPlan |
| A.7 | Nursing dashboard CLI demo | apps | End-to-end demonstration |

---

## Phase 5: Scale & Extend

**Goal:** Multi-instance deployment, more skill types, external integrations.

| ID | Task | Notes |
|----|------|-------|
| 5.1 | Horizontal runtime scaling (stateless instances) | Already designed for this |
| 5.2 | Runtime metrics + distributed tracing | OpenTelemetry |
| 5.3 | Skill marketplace / external skill loading | Via Wasm |
| 5.4 | Multi-channel support (Slack, Teams, API) | Beyond web UI |
| 5.5 | JSON Schema validation for skill parameters | Replace PH3 |

---

## Phase Summary

| Phase | Focus | Items | Blocks |
|-------|-------|-------|--------|
| ~~V1~~ | ~~MVP~~ | ~~35/35~~ | ~~Done~~ |
| **2** | Production Foundation | 16 tasks | All production deployment |
| **3** | Intelligence Layer | 12 tasks | Real AI conversations |
| **4** | Governance & Safety | 6 tasks | Enterprise adoption |
| **App** | Application Plane | 7 tasks | Domain capabilities |
| **5** | Scale & Extend | 5 tasks | Growth |
| **Total remaining** | | **46 tasks** | |
