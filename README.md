# OpenClaw Docker Setup with Local Ollama (qwen3.5:9b)

Containerized [OpenClaw](https://openclaw.ai) personal AI assistant running on a local [Ollama](https://ollama.ai) instance with the **qwen3.5:9b** model on an **NVIDIA RTX 4060 Ti** GPU. Fully self-hosted — no cloud API keys required.

## Prerequisites

| Requirement | Notes |
|-------------|-------|
| **Docker Desktop** | With **WSL 2** backend enabled |
| **NVIDIA GPU drivers** | Installed on the Windows host |
| **NVIDIA Container Toolkit** | [Install guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) |
| **Ollama** | Running on the Windows host ([ollama.ai](https://ollama.ai)) with `qwen3.5:9b` pulled |

> [!IMPORTANT]
> **GPU Requirements:** 16 GB VRAM recommended for `qwen3.5:9b`. The RTX 4060 Ti (16 GB) handles this model well. For GPUs with less VRAM, consider a smaller model like `qwen3.5:4b`.

## Setup Instructions

### 1. Clone this repository

```bash
git clone <your-repo-url>
cd openclaw-docker
```

### 2. Pull the qwen3.5:9b model in Ollama (host)

Make sure Ollama is running on your Windows host, then pull the model:

```bash
ollama pull qwen3.5:9b
```

### 3. Clone the official OpenClaw repo (for Docker build)

```bash
git clone --depth 1 https://github.com/openclaw/openclaw.git /tmp/openclaw
```

### 4. Copy Docker files into the cloned repo

```bash
cp docker-compose.yml Dockerfile docker-setup.sh /tmp/openclaw/
```

### 5. Run the official setup wizard

```bash
cd /tmp/openclaw
./docker-setup.sh
```

During the onboarding wizard:
- **Onboarding mode:** QuickStart
- **Model/auth provider:** Skip (or choose Ollama)
- **Filter by provider:** `ollama`
- **Default model:** `ollama/qwen3.5:9b`
- **Channel:** Skip for now

### 6. Configure Ollama provider

After onboarding, set the Ollama base URL so the container can reach your host Ollama:

```bash
docker compose -f /tmp/openclaw/docker-compose.yml exec openclaw-gateway \
  node dist/index.js config set models.providers.ollama.baseUrl "http://host.docker.internal:11434"
```

Set the LLM timeout (local models can be slower):

```bash
docker compose -f /tmp/openclaw/docker-compose.yml exec openclaw-gateway \
  node dist/index.js config set agents.defaults.timeoutSeconds 300
```

Restart the gateway:

```bash
docker compose -f /tmp/openclaw/docker-compose.yml restart openclaw-gateway
```

### 7. Access the Dashboard

Get the dashboard URL with token:

```bash
docker compose -f /tmp/openclaw/docker-compose.yml exec openclaw-gateway \
  node dist/index.js dashboard --no-open
```

> [!IMPORTANT]
> **Dashboard URL format:** `http://localhost:18789/#token=YOUR_TOKEN`
>
> Replace `YOUR_TOKEN` with your actual gateway token. Use the session **`agent:main:main?token=your_token`** — NOT "Main Session".

### 8. Approve your browser device

If you see "pairing required", approve your device:

```bash
# List pending devices
docker compose -f /tmp/openclaw/docker-compose.yml exec openclaw-gateway \
  node dist/index.js devices list

# Approve a pending device
docker compose -f /tmp/openclaw/docker-compose.yml exec openclaw-gateway \
  node dist/index.js devices approve <requestId>
```

## Environment Variables

Copy `.env.example` to `.env` and fill in your values:

```bash
cp .env.example .env
```

| Variable | Description | Default |
|----------|-------------|---------|
| `OPENCLAW_CONFIG_DIR` | Host path to OpenClaw config | `~/.openclaw` |
| `OPENCLAW_WORKSPACE_DIR` | Host path to agent workspace | `~/.openclaw/workspace` |
| `OPENCLAW_GATEWAY_PORT` | Dashboard port | `18789` |
| `OPENCLAW_GATEWAY_BIND` | Network bind mode (`lan` / `loopback`) | `lan` |
| `OPENCLAW_GATEWAY_TOKEN` | Gateway auth token (auto-generated) | — |
| `OPENCLAW_IMAGE` | Docker image to use | `openclaw:local` |

See [.env.example](.env.example) for the full list.

## Architecture

```
┌──────────────────────┐      ┌──────────────────────┐
│  OpenClaw Gateway    │      │  Ollama (host)       │
│  (Docker container)  │─────▶│  qwen3.5:9b          │
│  :18789              │      │  :11434              │
│                      │      │  RTX 4060 Ti GPU     │
└──────────────────────┘      └──────────────────────┘
   host.docker.internal:11434
```

## Useful Links

- [OpenClaw Documentation](https://docs.openclaw.ai)
- [OpenClaw Ollama Provider](https://docs.openclaw.ai/providers/ollama)
- [OpenClaw Docker Guide](https://docs.openclaw.ai/install/docker)
- [OpenClaw Configuration Reference](https://docs.openclaw.ai/gateway/configuration-reference)
- [Ollama Model Library](https://ollama.ai/library)
