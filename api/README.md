# API Reference

> REST API integration guide for OpenBotStack V1

## Authentication

Currently, OpenBotStack V1 runs in a single-tenant demo mode and does not enforce strict authentication on the API layer. However, standard bearer token validation is planned for V2.

*Note: For production deployments, it's highly recommended to place OpenBotStack behind an API gateway.*

## Endpoints

### `POST /v1/chat`

Creates a new message in a chat session and returns the agent's response. Supports SSE streaming.

**Request Body:**
```json
{
  "message": "Calculate tax for 500",
  "session_id": "optional-session-id",
  "stream": true
}
```

**Response (Non-Streaming):**
```json
{
  "id": "msg_123",
  "content": "The tax calculation is 50.00",
  "skills_used": ["core/tax.calculate"]
}
```

### SSE Streaming (if `stream: true`)
Returns `text/event-stream`.
```text
event: message
data: {"chunk": "The tax "}

event: message
data: {"chunk": "calculation is "}

event: message
data: {"chunk": "50.00"}

event: done
data: "[DONE]"
```

### `GET /v1/sessions/{id}/history`

Retrieves the conversation history for a given session.

**Response:**
```json
{
  "session_id": "sess_456",
  "messages": [
    {
      "role": "user",
      "content": "Calculate tax for 500",
      "timestamp": "2026-02-15T12:00:00Z"
    },
    {
      "role": "assistant",
      "content": "The tax calculation is 50.00",
      "timestamp": "2026-02-15T12:00:05Z"
    }
  ]
}
```

### `GET /v1/skills`

Lists all skills registered with the executor.

**Response:**
```json
{
  "skills": [
    {
      "id": "core/math.add",
      "type": "code",
      "version": "1.0.0",
      "enabled": true
    }
  ]
}
```

### `GET /v1/executions`

Retrieves the history of skill executions from the audit logger.

**Response:**
```json
{
  "executions": [
    {
      "execution_id": "exec_789",
      "skill_id": "core/math.add",
      "status": "success",
      "duration_ms": 120
    }
  ]
}
```

## Error Codes

The API returns standard HTTP status codes along with JSON error bodies.

| Status | Code | Description |
|--------|------|-------------|
| `400` | `invalid_request` | The request body is malformed. |
| `404` | `session_not_found` | The requested session ID does not exist. |
| `429` | `rate_limit_exceeded`| Token bucket limit reached. |
| `500` | `internal_error` | An unexpected runtime error occurred. |
| `504` | `skill_timeout` | A skill exceeded its resource limits. |

**Error Response Example:**
```json
{
  "error": {
    "code": "invalid_request",
    "message": "Message content cannot be empty"
  }
}
```
