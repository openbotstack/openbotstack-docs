# ADR-005: Embedded Frontend Architecture

**Status:** Accepted  
**Date:** 2026-02-05

## Decision

Frontend is built as a static React application and **embedded into the Go runtime binary** using `embed.FS`, producing a single-binary distribution for V1.

## Rationale

- Simplifies deployment (single binary)
- Reduces operational burden
- Fits internal enterprise usage
- Preserves future separation capability

## Structure

```
openbotstack-runtime/
├── cmd/openbotstack/main.go
├── api/
│   ├── router.go       # JSON / SSE APIs
│   └── router_test.go
├── web/
│   ├── package.json
│   ├── src/
│   │   ├── App.tsx
│   │   ├── components/
│   │   └── hooks/useSSE.ts
│   ├── dist/            # vite build output
│   └── webui/           # Go embed handler
│       ├── handler.go
│       └── handler_test.go
```

## Embedding

```go
//go:embed web/dist/*
var uiFS embed.FS
```

## Routing

```
/api/*   → JSON / SSE APIs
/ui/*    → embedded frontend
/        → redirect /ui/
```

## Vite Config

```ts
export default defineConfig({
  base: "/ui/",
})
```

## Escape Hatch Rules

To preserve future flexibility:

1. Frontend ONLY accesses backend via HTTP API
2. No server-side templates
3. No Go runtime-specific variables

This allows future:
- CDN deployment
- Separate admin UI
- SaaS multi-tenant

## Stack

- React 19 + TypeScript
- Vite (base: "/ui/")
- Vanilla CSS

## Not Suitable For

❌ High-concurrency public-facing UI  
❌ Multi-tenant theming  
❌ White-label / multi-brand

**Current V1 scope is internal enterprise → embedded is correct.**
