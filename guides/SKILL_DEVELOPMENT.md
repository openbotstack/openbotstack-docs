# Skill Development Guide

> Build custom skills for OpenBotStack using WebAssembly

## Quick Start

### 1. Create Skill Manifest

```yaml
# manifest.yaml
name: hello-world
version: 1.0.0
description: A simple greeting skill
entrypoint: main.wasm

triggers:
  - pattern: "hello"
  - pattern: "greet"

permissions:
  - llm:generate
  - log:write
```

### 2. Write Skill Code (TinyGo)

```go
// main.go
package main

import "github.com/openbotstack/skill-sdk-go"

func main() {}

//export execute
func execute() {
    input := skill.GetInput()
    
    // Use LLM host API
    response, _ := skill.LLM.Generate("Say hello to: " + input.Message)
    
    skill.Log.Info("Generated greeting")
    skill.SetOutput(response)
}
```

### 3. Build

```bash
tinygo build -o main.wasm -target wasi main.go
```

### 4. Package

```
hello-world/
├── manifest.yaml
└── main.wasm
```

### 5. Deploy

```bash
curl -X POST http://localhost:8080/admin/skills \
  -F "skill=@hello-world.zip"
```

## Host APIs

### LLM API
```go
// Generate text with LLM
response, err := skill.LLM.Generate(prompt)
response, err := skill.LLM.GenerateWithModel(prompt, "claude-3-sonnet")
```

### KV Store API
```go
// Get/Set key-value data
value, err := skill.KV.Get(key)
err := skill.KV.Set(key, value)
err := skill.KV.Delete(key)
```

### Log API
```go
skill.Log.Debug("message")
skill.Log.Info("message")
skill.Log.Warn("message")
skill.Log.Error("message")
```

### HTTP API
```go
resp, err := skill.HTTP.Get(url)
resp, err := skill.HTTP.Post(url, body)
```

## Resource Limits

| Resource | Default | Configurable |
|----------|---------|--------------|
| Memory | 128 MB | Yes |
| Execution time | 30s | Yes |
| KV storage | 1 MB | Per tenant |

## Local Testing

```bash
# Run skill locally with mock host
openbotstack skill test ./hello-world

# With custom input
openbotstack skill test ./hello-world --input "Hello world"
```

## Debugging

```bash
# Enable debug logging
openbotstack skill test ./hello-world --debug

# View execution trace
openbotstack skill test ./hello-world --trace
```

## Best Practices

1. **Keep skills small** — One skill, one responsibility
2. **Use structured logging** — Helps debugging in production
3. **Handle errors gracefully** — Return meaningful error messages
4. **Test locally first** — Use the skill test command
5. **Version your skills** — Use semantic versioning
