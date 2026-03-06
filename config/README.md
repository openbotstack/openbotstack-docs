# Configuration Reference

> Environment variables and schema for configuring OpenBotStack

## Environment Variables

These environment variables override settings in `config.yaml`.

| Variable | Description | Default |
|----------|-------------|---------|
| `OBS_ADDR` | The address and port for the API server. | `:8080` |
| `OBS_REDIS_URL` | Redis connection for sessions and rate limiting. | `redis://localhost:6379` |
| `OBS_MILVUS_URL` | Milvus connection for long-term vector memory. | (none) |
| `OBS_LLM_PROVIDER` | The LLM provider (e.g., `openai`, `claude`). | `openai` |
| `OBS_LLM_API_KEY` | Your API key for the selected provider. | (required) |
| `OBS_LOG_LEVEL` | Application logging level. | `info` |

## `config.yaml` Schema

Alternatively, configure via `config.yaml`:

```yaml
# Server settings
server:
  addr: ":8080"
  
# State storage
redis:
  url: "redis://localhost:6379"

# LLM Providers Configuration
providers:
  llm:
    default: openai
    
    openai:
      api_key: "sk-..." 
      model: "gpt-4o"
      
    claude:
      api_key: "sk-ant-..."
      model: "claude-3-sonnet-20240229"

# Wasm Sandbox Limits
sandbox:
  max_memory_mb: 128
  default_timeout_ms: 30000

# Rate Limiting & Quotas
quotas:
  tenant_tokens_per_minute: 1000
  user_requests_per_minute: 100
```

## Rate Limiting

OpenBotStack implements hierarchical rate limiting via Token Buckets in Redis:
- **Tenant:** Enforces hard limits for billing and scaling (`tenant_tokens_per_minute`). 
- **User:** Enforces soft limits for fairness (`user_requests_per_minute`).
