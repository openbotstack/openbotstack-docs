# OpenBotStack Architecture Overview

OpenBotStack is an enterprise AI execution infrastructure designed to provide assistant identity, memory abstraction, skills, policy governance, and controlled execution. It does **not** implement business applications directly. Instead, it offers a robust platform upon which industry applications are built.

To guarantee auditable, bounded, and deterministic execution at the control-plane level, OpenBotStack strictly enforces a 7-Plane Architecture.

## 7-Plane Architecture

```
┌─────────────────────────────────────────┐
│        Application Plane (External)     │
│  Industry Apps (Nursing / Finance / etc) │
└───────────────▲─────────────────────────┘
                │
┌───────────────┴─────────────────────────┐
│              Client Plane               │
│  Web UI / Mobile / SDK / API Clients    │
└───────────────▲─────────────────────────┘
                │
┌───────────────┴─────────────────────────┐
│              Access Plane               │
│  API Gateway / Auth / Tenant / RateLimit│
└───────────────▲─────────────────────────┘
                │
┌───────────────┴─────────────────────────┐
│             Control Plane               │
│  Assistant / Memory / Policy / Registry │
└───────────────▲─────────────────────────┘
                │
┌───────────────┴─────────────────────────┐
│            Execution Plane              │
│  Skill Executor / Wasm Sandbox / Tool   │
└───────────────▲─────────────────────────┘
                │
┌───────────────┴─────────────────────────┐
│           AI Service Plane              │
│  LLM Router / Providers / Embeddings    │
└───────────────▲─────────────────────────┘
                │
┌───────────────┴─────────────────────────┐
│          Infrastructure Plane           │
│  Redis / Milvus / Observability / LLMs  │
└─────────────────────────────────────────┘
```

---

### 1. Application Plane

**Location:** `openbotstack-apps/`

Industry applications built on top of OpenBotStack. This plane contains domain-specific skills, tools, workflows, and example applications (e.g., nursing management, clinical risk analysis). It depends on `openbotstack-core` interfaces but contains no control-plane or runtime logic.

**Responsibilities:**
- Domain skills (e.g., nursing, clinical, operations)
- Tool adapters (e.g., EHR, analytics, database)
- Workflows (e.g., shift handover, patient summary)
- Example applications

### 2. Client Plane

**Location:** External or Edge Interface

The external interfaces that connect users or external systems to OpenBotStack.

**Responsibilities:**
- Web UI
- Mobile apps
- SDK clients
- API integrations

### 3. Access Plane

**Location:** `openbotstack-core`

Responsible for system entry, governance, and traffic control.

**Suggested Module Structure (`core/access/`):**
- `http/`
- `auth/`
- `tenant/`
- `ratelimit/`
- `middleware/`

**Responsibilities:**
- API gateway
- Authentication and RBAC
- Tenant isolation
- Rate limiting
- Request audit logging

### 4. Control Plane

**Location:** `openbotstack-core`

The brain of the infrastructure that coordinates constraints, policies, and assistant behavior. It MUST be deterministic, with no autonomous reasoning loops.

**Suggested Module Structure:**
- `core/control/`
- `core/memory/`
- `core/policy/`
- `core/registry/`

**Responsibilities:**
- Assistant profiles
- Memory abstraction
- Skill registry
- Policy enforcement
- Execution state machine
- Governance model

### 5. Execution Plane

**Location:** `openbotstack-runtime`

Responsible for bounded execution. This plane executes skills and tools in an isolated sandbox. It MUST NOT directly call model providers; all AI operations go through the AI Service Plane.

**Suggested Module Structure:**
- `runtime/executor/`
- `runtime/sandbox/`
- `runtime/wasm/`
- `runtime/toolrunner/`

**Responsibilities:**
- Skill execution
- Tool invocation
- WASM sandbox
- Execution timeouts
- Retries
- Execution logging
- Resource guards

### 6. AI Service Plane

**Location:** `openbotstack-core`

Responsible for all LLM interactions and AI capabilities. It isolates the rest of the system from external LLM providers and enforces safety filtering.

**Suggested Module Structure (`core/ai/`):**
- `router/`
- `providers/`
- `embeddings/`
- `retrieval/`
- `prompts/`
- `safety/`

**Responsibilities:**
- LLM router
- Provider adapters (OpenAI, vLLM, Ollama, local GPU)
- Prompt templates
- Embedding service
- Retrieval service (RAG)
- Safety filtering

### 7. Infrastructure Plane

**Location:** External Deployment Dependencies

The external infrastructure systems required to deploy and run OpenBotStack.

**Responsibilities:**
- Redis
- Vector database (Milvus)
- Object storage
- Event bus
- Observability stack
- External LLM providers
