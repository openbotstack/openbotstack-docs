# ADR-004: Wasm Skill Runtime

**Status:** Accepted  
**Date:** 2026-02-05

## Context

Skills need sandboxed execution with:
- No filesystem or syscall access
- Explicit Host API whitelist
- CPU/memory limits
- Module caching for performance

## Decision

### Runtime: wazero

**Why wazero over alternatives:**
- Pure Go (no CGO, no external deps)
- Zero-config compilation
- Excellent security defaults
- Active maintenance

**Not chosen:**
- wasmtime-go: CGO dependency
- wasmer-go: Less active, CGO

### SDK: TinyGo

Skills compile with TinyGo to Wasm:
```bash
tinygo build -o skill.wasm -target wasi ./skill.go
```

### Execution Model

```
Request → Agent decides Skill → Load module (cached)
  → Instantiate (sandbox) → Inject Host APIs
  → Execute entrypoint → Return result
```

### Host API Whitelist

| API | Function | Description |
|-----|----------|-------------|
| `LLM` | `generate(prompt) → text` | Model generation |
| `KV` | `get/set/delete(key, value)` | Key-value storage |
| `HTTP` | `fetch(req) → resp` | Controlled HTTP |
| `Log` | `log(level, msg)` | Structured logging |

**Explicitly forbidden:**
- File system access
- Network sockets (raw)
- Process spawning
- Environment variables

### Resource Limits

```go
type SkillLimits struct {
    MaxMemoryBytes   int64         // 128MB default
    MaxCPUMs         int64         // 5000ms default
    MaxExecutionTime time.Duration // 30s default
}
```

### Module Caching

```
skill_id + version → compiled module (in-memory LRU)
```

Cache invalidation on:
- Version change
- Manual reload
- Memory pressure

## Consequences

**Positive:**
- Secure by default
- Predictable resource usage
- Fast cold start (wazero compiles fast)
- Pure Go deployment

**Negative:**
- TinyGo has some Go limitations
- Custom ABI means no WASI ecosystem reuse
- Debug experience less mature than native

## Implementation

Location: `openbotstack-runtime/wasm/`
```
wasm/
├── runtime.go       # wazero wrapper
├── hostapi.go       # Host function bindings
├── cache.go         # Module cache
├── limits.go        # Resource enforcement
└── runtime_test.go  # Tests
```
