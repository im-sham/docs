# Design Partner Onboarding & Isolation Guide

> **Status:** Updated for Multi-Tenancy (Week 11)
> **Goal:** Secure, isolated testing environments for Design Partners via key-based multi-tenancy.

## Architecture: Key-Based Multi-Tenancy

OpsOrchestra now supports **application-level multi-tenancy** where partners access a single deployment with data isolation.

### How It Works

1. **Landing**: Partner visits the OpsOrchestra URL
2. **Gate**: A simple "Enter Access Key" screen blocks the main UI
3. **Active Session**: Upon entering a valid key (e.g., `PARTNER-ALPHA`), the UI stores this in the session
4. **Isolation**:
   - Uploaded documents go to a tenant-specific collection
   - Chat history is stored in `{tenant_id}_traces.jsonl`
   - The Knowledge Graph is loaded from `{tenant_id}_graph.json`

## Deployment Options

### Option A: Multi-Tenant (Recommended)

Use the built-in tenant isolation for simpler deployment:

```bash
# Set valid tenant keys in environment
export VALID_TENANT_KEYS="PARTNER-ALPHA,PARTNER-BETA,PARTNER-GAMMA"

# Start single instance
docker run -d \
  -e VALID_TENANT_KEYS="$VALID_TENANT_KEYS" \
  -e ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY" \
  -v ./data:/app/.agent \
  -p 8000:8000 \
  opsorchestra:latest
```

### Option B: Single-Tenant Instances (Legacy)

For maximum isolation, deploy separate instances per partner:

```bash
# Example: Running for Partner A
docker run -d \
  -v ./partner_a_data/chroma:/app/.chroma \
  -v ./partner_a_data/agent:/app/.agent \
  -p 8001:8000 \
  --name opsorchestra-partner-a \
  opsorchestra:latest
```
## Onboarding Checklist

1. **Generate Access Key**: Create a unique key for the partner (alphanumeric, no special chars)
2. **Add to Allowlist**: Add key to `VALID_TENANT_KEYS` environment variable
3. **Share Credentials**: Provide partner with URL and access key
4. **Data Ingestion**: Partner uploads documents via the UI
5. **Validation**: Run test query to verify isolation

## Security Considerations

- **Trust Model**: Designed for semi-trusted Design Partners, not public SaaS
- **Key Validation**: Keys are validated against an allowlist
- **Path Traversal Prevention**: Keys cannot contain `/` or `..`
- **Rate Limiting**: Consider adding rate limiting for production

## Knowledge Graph Strategy

The system uses a hybrid graph approach:

1. **Tenant Graph (Read/Write)**: Private documents, co-retrieval patterns, sensitive entities
2. **Global Seed Graph (Read-Only)**: Curated benchmarks, vendor categories, best practices

This ensures tenant data isolation while sharing universal knowledge.
