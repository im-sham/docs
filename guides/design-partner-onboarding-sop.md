# Design Partner Onboarding SOP

> Standard Operating Procedure for onboarding and managing OpsOrchestra design partners

---

## Overview

This document covers the end-to-end process for onboarding new design partners, monitoring their usage, and managing their access.

---

## Prerequisites

Before onboarding partners, ensure:

1. **ADMIN_SECRET configured** in production `.env`
2. **VALID_TENANT_KEYS** is empty or removed (we now use the tenants.json store)
3. **API and frontend deployed** to Railway

---

## 1. Creating a New Design Partner

### Option A: CLI (Recommended for automation)

```bash
cd /path/to/opsorchestra

# Create tenant with basic info
python -m scripts.admin create-tenant --name "Acme Corp" --email "ops@acme.com"

# Create with notes
python -m scripts.admin create-tenant --name "Beta Inc" --email "jane@beta.io" --notes "Referred by investor"
```

**Output:**
```
==================================================
TENANT CREATED SUCCESSFULLY
==================================================

Access Key: acme_corp_A1B2C3D4
Name:       Acme Corp
Email:      ops@acme.com
Created:    2026-01-02T15:30:00Z

--------------------------------------------------
IMPORTANT: Share this access key with the design partner.
They will use it to log into the OpsOrchestra UI.
--------------------------------------------------
```

### Option B: Admin UI

1. Navigate to the OpsOrchestra app
2. Click **Admin** (shield icon) in the sidebar
3. Enter your `ADMIN_SECRET` when prompted
4. Click **Add Partner**
5. Fill in partner details
6. Copy the generated access key

---

## 2. Sharing Access with Partners

### Email Template

```
Subject: Your OpsOrchestra Access Key

Hi [Partner Name],

Welcome to OpsOrchestra! Here's your access key to get started:

Access Key: [PASTE KEY HERE]
App URL: https://your-railway-app.up.railway.app

Getting Started:
1. Go to the app URL above
2. Enter your access key when prompted
3. Start chatting with the AI analyst

Your data is completely isolated from other users. Feel free to upload 
real documents - only you can see them.

Questions? Reply to this email.

Best,
[Your Name]
```

---

## 3. Monitoring Partner Engagement

### CLI Quick Check

```bash
# List all active tenants with usage stats
python -m scripts.admin list-tenants

# Output:
# ================================================================================
# KEY                       NAME                 STATUS     QUERIES  LAST ACTIVE
# ================================================================================
# acme_corp_A1B2C3D4       Acme Corp            active     47       2026-01-02
# beta_inc_E5F6G7H8        Beta Inc             active     3        2026-01-01
# --------------------------------------------------------------------------------
# Total: 2 tenant(s)
```

### Admin UI Dashboard

1. Go to Admin view
2. View at-a-glance stats:
   - **Active Partners**: Currently onboarded count
   - **Total Queries**: Aggregate usage across all partners
   - **Have Logged In**: Partners who have actually used the app

### Red Flags to Watch

| Signal | Action |
|--------|--------|
| Partner created but never logged in | Send reminder email after 3 days |
| Low query count (< 5 after 1 week) | Schedule check-in call |
| High query count (> 50/week) | Great engagement! Ask for feedback |
| No activity for 7+ days | Follow up to understand blockers |

---

## 4. Collecting Feedback

### During Usage

The app automatically logs decision traces. Review in Langfuse or the Traces view.

### Scheduled Check-ins

Recommended cadence:
- **Day 3**: Quick email - "How's it going?"
- **Week 1**: 15-min call - Walk through their experience
- **Week 2**: In-depth feedback session

### Feedback Questions

1. What was your first impression?
2. What questions did you ask? Did the answers help?
3. What documents did you upload?
4. What's missing that would make this useful for you?
5. Would you recommend this to a peer? Why/why not?

---

## 5. Revoking Access

### When to Revoke

- Partner requests removal
- Partnership ends
- Security concern
- Inactive for 30+ days (optional cleanup)

### CLI

```bash
python -m scripts.admin revoke-tenant acme_corp_A1B2C3D4

# With confirmation skip (for scripts)
python -m scripts.admin revoke-tenant acme_corp_A1B2C3D4 --force
```

### Admin UI

1. Go to Admin view
2. Find the tenant in the table
3. Click the trash icon
4. Confirm revocation

### Post-Revocation

- Partner's access key immediately stops working
- Their data (documents, traces, graph) remains in `.agent/{tenant_id}/`
- To fully delete data, manually remove the tenant folder

---

## 6. Troubleshooting

### "Access Key Not Working"

1. Verify key exists: `python -m scripts.admin show-tenant [key]`
2. Check status is "active" (not "revoked")
3. Ensure backend is running and reachable

### "Admin Page Shows Error"

1. Verify `ADMIN_SECRET` is set in production `.env`
2. Ensure you're using the correct secret
3. Check API logs for 401/403 errors

### "Partner Data Not Isolated"

This should never happen. If you suspect cross-tenant data leakage:
1. Stop the application immediately
2. Check tenant_id propagation in API logs
3. Verify `get_tenant_id` dependency is applied to all routes

---

## 7. Architecture Reference

```
.agent/
├── _admin/
│   └── tenants.json          # Tenant metadata store
├── _global/
│   └── seed_graph.json       # Shared SaaS benchmarks (read-only)
├── acme_corp_A1B2C3D4/       # Tenant-specific data
│   ├── graph_state.json
│   └── traces/
│       └── session_*.jsonl
└── beta_inc_E5F6G7H8/
    ├── graph_state.json
    └── traces/
```

---

## 8. Quick Reference Commands

| Task | Command |
|------|---------|
| Create tenant | `python -m scripts.admin create-tenant --name "Name" --email "email"` |
| List tenants | `python -m scripts.admin list-tenants` |
| Show tenant details | `python -m scripts.admin show-tenant [key]` |
| Revoke tenant | `python -m scripts.admin revoke-tenant [key]` |
| List including revoked | `python -m scripts.admin list-tenants --all` |

---

## Appendix: Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `ADMIN_SECRET` | Secret for admin API access | `your-secure-secret-here` |
| `ANTHROPIC_API_KEY` | Claude API key | `sk-ant-...` |
| `LANGFUSE_*` | Observability config | See .env.example |

---

*Last Updated: January 2, 2026*
