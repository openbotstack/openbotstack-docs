# Deployment Guide

> Deploy OpenBotStack to production

OpenBotStack can be deployed via **Docker** (recommended), **Bare Metal** (Linux/macOS), or **Kubernetes**.

## Prerequisites

Regardless of the deployment method, you need:

1.  **Redis**: Required for session state and short-term memory.
2.  **Milvus** (Optional): For long-term vector memory.
3.  **LLM API Key**: DeepSeek, OpenAI, Anthropic, etc.

---

## Option 1: Docker Deployment (Recommended)

The easiest way to get started.

### Step 1: Create Project Directory

```bash
mkdir openbotstack
cd openbotstack
```

### Step 2: Create Configuration

Create a `docker-compose.yml` file:

```yaml
services:
  openbotstack:
    image: ghcr.io/openbotstack/openbotstack:latest
    ports:
      - "8080:8080"
    environment:
      - OBS_ADDR=:8080
      - OBS_REDIS_URL=redis://redis:6379
      - OBS_LLM_PROVIDER=openai
      # Or use secret management
      - OBS_LLM_API_KEY=${OPENAI_API_KEY}
    depends_on:
      - redis
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
    restart: unless-stopped

volumes:
  redis-data:
```

### Step 3: Configure Environment

Create a `.env` file for secrets:

```bash
OPENAI_API_KEY=sk-...
```

### Step 4: Start Services

```bash
docker-compose up -d
```

### Step 5: Verify

Check logs:
```bash
docker-compose logs -f
```

Visit UI:
http://localhost:8080/ui/

---

## Option 2: Bare Metal Deployment

Deploy directly on Linux (AMD64/ARM64) or macOS (Apple Silicon).

### Step 1: Build or Download Binary

**Build from source:**

```bash
# Clone repository
git clone https://github.com/openbotstack/openbotstack-runtime.git
cd openbotstack-runtime

# Build for your platform
make build              # Current OS
# OR
make build-linux-amd64  # Linux x64
make build-linux-arm64  # Linux ARM
```

The binary will be in `./build/`.

### Step 2: Run External Dependencies

You must provide a running Redis instance.

```bash
# Example: Run Redis via Docker
docker run -d -p 6379:6379 redis:7-alpine
```

### Step 3: Create Configuration

Create `config.yaml` in the same directory as the binary:

```yaml
server:
  addr: ":8080"

redis:
  url: "redis://localhost:6379"

providers:
  llm:
    default: openai
    openai:
      api_key: "sk-..." 
      model: "gpt-4o"
```

### Step 4: Run Application

```bash
# Run with config file
./build/openbotstack --config config.yaml

# OR override with environment variables
export OBS_LLM_API_KEY=sk-...
./build/openbotstack
```

### Step 5: Setup Systemd (Linux Production)

Create `/etc/systemd/system/openbotstack.service`:

```ini
[Unit]
Description=OpenBotStack Runtime
After=network.target

[Service]
Type=simple
User=obs
ExecStart=/opt/openbotstack/openbotstack
WorkingDirectory=/opt/openbotstack
Environment="OBS_REDIS_URL=redis://localhost:6379"
Environment="OBS_LLM_API_KEY=sk-..."
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable and start:
```bash
systemctl enable --now openbotstack
```

---

## Option 3: Kubernetes Deployment

For high-availability clusters.

### Step 1: Create Secrets

```bash
kubectl create secret generic openbotstack-secrets \
  --from-literal=openai-api-key=sk-... \
  --from-literal=redis-url=redis://redis-service:6379
```

### Step 2: Apply Manifest

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: openbotstack
spec:
  replicas: 3
  selector:
    matchLabels:
      app: openbotstack
  template:
    metadata:
      labels:
        app: openbotstack
    spec:
      containers:
      - name: openbotstack
        image: ghcr.io/openbotstack/openbotstack:latest
        ports:
        - containerPort: 8080
        env:
        - name: OBS_LLM_API_KEY
          valueFrom:
            secretKeyRef:
              name: openbotstack-secrets
              key: openai-api-key
        resources:
          limits:
            memory: "1Gi"
            cpu: "1000m"
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: openbotstack
spec:
  selector:
    app: openbotstack
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
```

### Step 3: Deploy

```bash
kubectl apply -f deployment.yaml
```

---

## Configuration Reference

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `OBS_ADDR` | Listen address | `:8080` |
| `OBS_REDIS_URL` | Redis connection | `redis://localhost:6379` |
| `OBS_MILVUS_URL` | Milvus connection | (optional) |
| `OBS_LLM_PROVIDER` | LLM provider | `openai` |
| `OBS_LLM_API_KEY` | LLM API key | (required) |
| `OBS_LOG_LEVEL` | Log level | `info` |

### Health Checks

| Endpoint | Description |
|----------|-------------|
| `GET /health` | Liveness check |
| `GET /ready` | Readiness check |
| `GET /metrics` | Prometheus metrics |

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **Connection refused** | Check if Redis is running and reachable. |
| **LLM errors** | Verify API key and network access to LLM provider. |
| **Wasm errors** | Check skill logs using `OBS_LOG_LEVEL=debug`. |
| **Memory limit** | Bare metal: Check system RAM. Docker: Adjust memory limit. |
