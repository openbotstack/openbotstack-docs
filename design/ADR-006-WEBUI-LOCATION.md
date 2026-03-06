# ADR-006: webui Handler Location

**Status:** Accepted  
**Date:** 2026-02-07

## Decision

The `webui` Go package (embed handler) is located at `web/webui/` inside the frontend source directory.

## Rationale

```
web/
├── src/           # React source
├── webui/         # Go embed handler
│   ├── handler.go
│   ├── handler_test.go
│   └── dist/      # Vite build output (configured via vite outDir)
```

**Pros:**
1. **Embed path works**: Go embed requires paths relative to package dir
2. **Co-location**: Frontend source + Go handler in one directory
3. **Single build target**: `npm run build` → `dist/` is adjacent

**Alternative considered:**
- `internal/server/ui.go` embedding `../web/dist` — rejected because Go embed doesn't support parent paths

## Consequences

- Vite `outDir` is configured to output directly to `webui/dist/`
- No manual copy step required

## Build Process

```makefile
web-build:
	cd web && npm run build
```
