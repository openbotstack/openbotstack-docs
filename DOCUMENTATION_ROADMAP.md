# OpenBotStack Documentation Roadmap

## Current Documentation

| Document | Status |
|----------|--------|
| [SYSTEM_DESIGN.md](design/SYSTEM_DESIGN.md) | ✅ Complete |
| [V1_STORIES.md](design/V1_STORIES.md) | ✅ Complete |
| [ADR-004-WASM-RUNTIME.md](design/ADR-004-WASM-RUNTIME.md) | ✅ Complete |
| [ADR-005-CHAT-UI.md](design/ADR-005-CHAT-UI.md) | ✅ Complete |
| [ADR-006-WEBUI-LOCATION.md](design/ADR-006-WEBUI-LOCATION.md) | ✅ Complete |

## Documentation Gaps (Priority Order)

### 1. Skill Development Guide (HIGH) - ✅ Complete
**Target:** First-time skill developers
```
docs/guides/SKILL_DEVELOPMENT.md
├── What is a Skill?
├── Skill Manifest (manifest.yaml)
├── Host APIs (LLM, KV, Log)
├── Build with TinyGo
├── Local testing
└── Examples
```

### 2. Deployment Guide (HIGH) - ✅ Complete
**Target:** DevOps / SRE
```
docs/guides/DEPLOYMENT.md
├── Prerequisites (Redis, Milvus)
├── Configuration (config.yaml)
├── Docker deployment
├── Kubernetes deployment
├── Health checks & monitoring
├── Scaling recommendations
└── Troubleshooting
```

### 3. API Reference (MEDIUM) - ✅ Complete
**Target:** Integration developers
```
docs/api/README.md
├── Authentication
├── POST /v1/chat
├── GET /v1/sessions/{id}/history
├── SSE streaming
└── Error codes
```

### 4. Configuration Reference (MEDIUM) - ✅ Complete
```
docs/config/README.md
├── Environment variables
├── config.yaml schema
├── Provider configuration
└── Rate limiting
```

### 5. Examples (HIGH) - ✅ Complete

Located in `openbotstack-runtime/examples/skills/`:
```
examples/skills/
├── hello-world/          # Minimal deterministic skill
├── math-add/             # Pure Go calculation
├── meeting-summarize/    # LLM-assisted meeting notes
├── sentiment/            # Sentiment analysis
├── summarize/            # Declarative text summarizer
├── tax-calculator/       # Deterministic tax calculation
└── wordcount/            # Wasm word counter
```

Deployment examples are in `openbotstack-runtime/` root:
- `docker-compose.yml` — Full stack with Redis

## Recommended Order

1. **examples/skills/hello-world/** — Get people building fast
2. **SKILL_DEVELOPMENT.md** — Reference for skill authors
3. **docker-compose.yml** (in runtime repo) — Easy local stack
4. **DEPLOYMENT.md** — Production deployment
5. **API Reference** — Integration details

