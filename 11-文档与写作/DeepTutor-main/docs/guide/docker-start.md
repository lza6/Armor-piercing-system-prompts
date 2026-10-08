# Docker Deployment

Docker deployment requires no Python or Node.js setup — everything runs in containers.

## Prerequisites

- **Docker** — [Install Docker](https://docs.docker.com/get-docker/)
- **Docker Compose** — [Install Docker Compose](https://docs.docker.com/compose/install/)

## Quick Start Options

Choose your preferred deployment method:

<div class="method-tabs">

### Option A: Build from Source

Build and run DeepTutor locally from the source code.

```bash
# Clone repository (if not done)
git clone https://github.com/HKUDS/DeepTutor.git
cd DeepTutor

# Create and configure .env file
cp .env.example .env
# Edit .env with your API keys

# Build and start
docker compose up

# First run takes ~11 minutes on Mac Mini M4
```

#### Common Commands

```bash
# Start in background
docker compose up -d

# Stop services
docker compose down

# View logs
docker compose logs -f

# Rebuild after code changes
docker compose up --build

# Clear cache and rebuild
docker compose build --no-cache
```

</div>

<div class="method-tabs">

### Option B: Pull Pre-built Image (Faster)

Use our official pre-built images from GitHub Container Registry.

```bash
# Works on all platforms - Docker auto-detects your architecture
docker run -d --name deeptutor \
  -p 8001:8001 -p 3782:3782 \
  --env-file .env \
  -v $(pwd)/data:/app/data \
  -v $(pwd)/config:/app/config:ro \
  ghcr.io/hkuds/deeptutor:latest
```

::: warning Windows PowerShell
Use `${PWD}` instead of `$(pwd)`:

```powershell
docker run -d --name deeptutor `
  -p 8001:8001 -p 3782:3782 `
  --env-file .env `
  -v ${PWD}/data:/app/data `
  -v ${PWD}/config:/app/config:ro `
  ghcr.io/hkuds/deeptutor:latest
```
:::

</div>

## Available Image Tags

| Tag | Architectures | Description |
|:----|:--------------|:------------|
| `:latest` | AMD64 + ARM64 | Latest stable release (auto-detects architecture) |
| `:v0.5.x` | AMD64 + ARM64 | Specific version (auto-detects architecture) |
| `:v0.5.x-amd64` | AMD64 only | Explicit AMD64 image |
| `:v0.5.x-arm64` | ARM64 only | Explicit ARM64 image |

::: tip Multi-Architecture Support
The `:latest` tag is a **multi-architecture image** — Docker automatically pulls the correct version for your system (Intel/AMD or Apple Silicon/ARM).
:::

## Cloud Deployment

When deploying to a cloud server, you must set the external API URL:

```bash
docker run -d --name deeptutor \
  -p 8001:8001 -p 3782:3782 \
  -e NEXT_PUBLIC_API_BASE_EXTERNAL=https://your-server.com:8001 \
  --env-file .env \
  -v $(pwd)/data:/app/data \
  ghcr.io/hkuds/deeptutor:latest
```

::: warning Why is this needed?
The default API URL is `localhost:8001`, which points to the user's local machine in the browser, not your server. Setting `NEXT_PUBLIC_API_BASE_EXTERNAL` ensures the frontend connects to your server's public address.
:::

## Custom Ports

To use different ports:

```bash
docker run -d --name deeptutor \
  -p 9001:9001 -p 4000:4000 \
  -e BACKEND_PORT=9001 \
  -e FRONTEND_PORT=4000 \
  -e NEXT_PUBLIC_API_BASE_EXTERNAL=http://localhost:9001 \
  --env-file .env \
  -v $(pwd)/data:/app/data \
  ghcr.io/hkuds/deeptutor:latest
```

::: warning Important
The `-p` port mapping must match the `BACKEND_PORT`/`FRONTEND_PORT` values.
:::

## Access URLs

| Service | URL | Description |
|:---:|:---|:---|
| **Frontend** | http://localhost:3782 | Main web interface |
| **API Docs** | http://localhost:8001/docs | Interactive API documentation |

## Container Management

### View Running Containers

```bash
docker ps
```

### Stop Container

```bash
docker stop deeptutor
```

### Remove Container

```bash
docker rm deeptutor
```

### View Logs

```bash
# All logs
docker logs deeptutor

# Follow logs in real-time
docker logs -f deeptutor

# Last 100 lines
docker logs --tail 100 deeptutor
```

## Troubleshooting

### Frontend cannot connect in cloud deployment

**Problem:** Frontend shows "Failed to fetch" or "NEXT_PUBLIC_API_BASE is not configured"

**Solution:** Set `NEXT_PUBLIC_API_BASE_EXTERNAL` to your server's public URL:

```bash
-e NEXT_PUBLIC_API_BASE_EXTERNAL=https://your-server.com:8001
```

### Settings page shows "Error loading data" with HTTPS reverse proxy

**Problem:** HTTPS requests are being redirected to HTTP (307 redirect)

**Solution:** This issue has been fixed in v0.5.0+. Update to the latest version.

**nginx Configuration:**

```nginx
# Frontend
location / {
    proxy_pass http://localhost:3782;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
}

# Backend API
location /api/ {
    proxy_pass http://localhost:8001;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-Proto $scheme;
}

# WebSocket support
location /api/v1/ {
    proxy_pass http://localhost:8001;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

### Data not persisting after restart

**Solution:** Ensure you're mounting the data directory:

```bash
-v $(pwd)/data:/app/data
```

### HuggingFace models fail to download (restricted networks)

**Solution:** Add HuggingFace configuration to your `.env`:

```bash
# Use a mirror endpoint
HF_ENDPOINT=https://your-hf-mirror.example.com

# Or force offline mode (requires pre-cached models)
HF_HUB_OFFLINE=1
```

---

**Next Step:** [Make Contribution →](/contribution)

<style>
.method-tabs {
  background: var(--vp-c-bg-soft);
  border: 1px solid var(--vp-c-border);
  border-radius: 12px;
  padding: 20px;
  margin: 16px 0;
}

.method-tabs h3 {
  margin-top: 0;
  padding-top: 0;
  border-top: none;
}
</style>
