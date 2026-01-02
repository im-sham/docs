# Deployment Guide

This document covers deploying OpsOrchestra to Railway.

## Architecture

OpsOrchestra uses a multi-process architecture in production:

```
nginx (:$PORT) ──┬── Static files (React UI)
                 ├── /health (immediate 200)
                 └── /api/* ──► uvicorn (:8000) ──► FastAPI
```

### Port Configuration

| Port | Service | Purpose |
|------|---------|---------|
| `$PORT` | nginx | **Public entry point** - Railway routes here |
| 8000 | uvicorn | FastAPI backend (internal) |

> [!IMPORTANT]
> Railway must route to `$PORT` (nginx), NOT port 8000 (uvicorn directly).

## Railway Setup

### 1. Environment Variables

Set these in Railway's service variables:

```
ANTHROPIC_API_KEY=sk-ant-...
DEBUG=false
```

Optional (for Langfuse observability):
```
LANGFUSE_PUBLIC_KEY=pk-lf-...
LANGFUSE_SECRET_KEY=sk-lf-...
LANGFUSE_HOST=https://your-langfuse-instance
```

### 2. Networking Configuration

After deployment, verify in **Settings → Networking → Public Networking**:

- Domain should point to the port shown in logs as "Railway assigned PORT: XXXX"
- This is typically **8080** (or whatever Railway assigns via `$PORT`)
- Do NOT point to port 8000 (you'll get JSON API responses instead of the UI)

### 3. Health Checks

The `railway.toml` configures:
```toml
[deploy]
healthcheckPath = "/health"
healthcheckTimeout = 300
```

nginx responds immediately to `/health` with 200, allowing Railway to pass health checks while uvicorn loads ML models (~12-30 seconds).

## Troubleshooting

### 502 Bad Gateway

**Symptom:** Deployment succeeds, but app returns 502.

**Check 1: Railway Edge vs nginx**
```bash
curl -v https://your-app.up.railway.app/health
```

Look at the `server` header in the response:
- `server: railway-edge` + 502 → Railway can't reach container (port mismatch)
- `server: nginx` + 502 → nginx can't reach uvicorn

**Check 2: Port Configuration**
1. Go to Railway dashboard → Service → Settings → Networking
2. Verify the domain points to the correct port (match "Railway assigned PORT" in logs)
3. If incorrect, delete and recreate the domain

### JSON Response Instead of UI

**Symptom:** Visiting the app shows JSON like:
```json
{"app":"OpsOrchestra","description":"AI-powered Business Operations Analyst"...}
```

**Cause:** Railway is routing to port 8000 (uvicorn) instead of nginx.

**Fix:** Change the port mapping to `$PORT` (typically 8080).

## Build Process

The Dockerfile uses multi-stage build:

1. **Stage 1 (node):** Builds React frontend → `/web/dist`
2. **Stage 2 (python):** 
   - Installs Python dependencies + PyTorch CPU
   - Copies built React files to `/app/web/dist`
   - nginx serves these static files

## Local Testing

To test the Docker build locally:

```bash
docker build -t opsorchestra .
docker run -p 8080:8080 -e PORT=8080 -e ANTHROPIC_API_KEY=sk-ant-... opsorchestra
```

Then visit `http://localhost:8080`
