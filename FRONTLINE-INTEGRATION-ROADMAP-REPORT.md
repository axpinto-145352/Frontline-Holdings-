# Frontline Holdings — Integration Roadmap & Options Report
## Payment Automation: Procore, QuickBooks Online, Excel (OneDrive), n8n

**Date:** February 28, 2026
**Prepared for:** Frontline Holdings Leadership Team
**Purpose:** Decision-ready summary of integration options, verified API capabilities, architecture paths, and implementation timeline

**How to read this report:** Sections 1-6, 9-11 are written for the leadership team. Sections 7-8 contain technical implementation details for the development team and can be skipped by non-technical readers.

**Companion files in this repository:**
- `PROPOSAL-FRONTLINE-PAYMENT-AUTOMATION.md` — Client-facing proposal with 2 engagement options and pricing
- `FRONTLINE-PAYMENT-AUTOMATION.md` — Full technical architecture document
- `n8n-workflows/` — Importable n8n workflow JSON templates (3 workflows)
- `notion-2nd-brain/SETUP-GUIDE.md` — Notion database setup guide (retained as reference material only; not included in current proposal options)
- `REVIEW-ASSESSMENT-REPORT.md` — Prior assessment and review findings

---

## 1. Situation Summary

Frontline Holdings lost **$150,000 in confirmed duplicate payment losses last year**. Based on that confirmed figure and industry benchmarks, annual exposure is **$150,000-$300,000+** as project volume grows.

The root cause: invoices enter through multiple channels (email, Procore, mail) with no cross-system deduplication. The current stack — Procore (project management), QuickBooks Online (accounting), and SmoothX (Procore-QBO sync middleware) — has no automated layer to catch duplicates, match invoices to commitments, notify PMs, or follow up on unpaid customer invoices.

This report covers:
1. What each system's API can actually do (verified against official documentation)
2. Two proposed architecture options with honest trade-offs (plus one documented but not proposed)
3. A phased implementation roadmap with timelines
4. Cost comparison across options (including development labor)

---

## 2. Verified API Capabilities — What We Can Actually Build Against

### 2.1 Procore API

| Capability | API Endpoint | Verified? |
|-----------|-------------|-----------|
| List all projects | `GET /rest/v1.0/projects` | Yes |
| Get subcontracts/POs | `GET /rest/v1.1/projects/{id}/commitments` | Yes |
| Get line items on a commitment | `GET /rest/v1.0/projects/{id}/commitments/{cid}/line_items` | Yes |
| Get subcontractor pay apps | `GET /rest/v1.0/projects/{id}/commitments/{cid}/requisitions` | Yes |
| Get direct costs (expenses, invoices) | `GET /rest/v1.1/projects/{id}/direct_costs` | Yes |
| Get prime contracts | `GET /rest/v1.0/prime_contracts?project_id={id}` | Yes |
| Get change orders | `GET /rest/v1.0/projects/{id}/change_orders` | Yes |
| Get vendor directory | `GET /rest/v1.0/companies/{cid}/vendors` | Yes |
| Get budget views | `GET /rest/v1.0/projects/{id}/budget/views` | Yes |
| Real-time webhooks | `POST /rest/v1.0/companies/{cid}/webhooks` | Yes |

**Authentication:** OAuth 2.0 with two grant types. **Client Credentials** using a Developer Managed Service Account (DMSA) is recommended for automated server-to-server workflows. Authorization Code grant is available for user-interactive flows but is not needed for this project. Procore does NOT use OAuth scopes — permissions are controlled via DMSA project-level settings. Tokens expire every 1.5 hours (5,400 seconds); n8n handles refresh automatically. All calls require `Procore-Company-Id` header.

**Rate limit:** 3,600 requests/hour (can request increase to 7,200 or 14,400 via apisupport@procore.com). Rate limit headers: `X-Rate-Limit-Remaining`, `X-Rate-Limit-Reset`. Throttled = HTTP 429.

**n8n connection:** No native Procore node exists. Use HTTP Request node with Generic OAuth2 credentials. Fully functional — the exact configuration parameters have been verified against Procore's developer documentation.

### 2.2 QuickBooks Online API

| Capability | API Endpoint | Verified? |
|-----------|-------------|-----------|
| Query existing bills (for dedup) | `GET /v3/company/{id}/query?query=SELECT * FROM Bill WHERE...` | Yes |
| Create vendor bills | `POST /v3/company/{id}/bill` | Yes |
| Create customer invoices | `POST /v3/company/{id}/invoice` | Yes |
| Record bill payments | `POST /v3/company/{id}/billpayment` | Yes |
| Check payment status | `GET /v3/company/{id}/query?query=SELECT * FROM Payment WHERE...` | Yes |
| Vendor CRUD | `GET/POST /v3/company/{id}/vendor` | Yes |
| Customer CRUD | `GET/POST /v3/company/{id}/customer` | Yes |

**n8n connection:** Native QBO node available in n8n. Supports full CRUD on invoices, bills, payments, vendors, and customers out of the box.

### 2.3 Microsoft Graph API (Excel on OneDrive)

| Capability | API Endpoint | Verified? |
|-----------|-------------|-----------|
| Create workbook session | `POST /v1.0/me/drive/items/{id}/workbook/createSession` | Yes |
| Read table rows | `GET /v1.0/me/drive/items/{id}/workbook/tables/{table}/rows` | Yes |
| Add table rows | `POST /v1.0/me/drive/items/{id}/workbook/tables/{table}/rows/add` | Yes |
| Read/write range | `GET/PATCH /v1.0/me/drive/items/{id}/workbook/worksheets/{sheet}/range` | Yes |
| List drive items | `GET /v1.0/me/drive/items/{id}/children` | Yes |

**Authentication:** Azure AD OAuth 2.0. Requires Azure app registration with `Files.ReadWrite` and `Sites.ReadWrite.All` permissions.

**Rate limit:** 10,000 requests per 10 minutes (more than adequate for invoice volumes).

**n8n connection:** HTTP Request node with OAuth2 credentials (Microsoft Graph). No native Excel/OneDrive node required — the HTTP Request node handles all Graph API calls.

### 2.4 Notion API (Reference Only)

> **Note:** Notion is **not included in current proposal options**. This section is retained as reference material only. The Excel approval dashboard (via Microsoft Graph API) replaces all Notion functionality in both proposed options.

| Capability | API Endpoint | Verified? |
|-----------|-------------|-----------|
| Create tracking databases | `POST /v1/databases` | Yes |
| Query with filters (dedup check) | `POST /v1/databases/{id}/query` | Yes |
| Create invoice records | `POST /v1/pages` | Yes |
| Update status/fields | `PATCH /v1/pages/{id}` | Yes |
| Full-text search | `POST /v1/search` | Yes |

**n8n connection:** Native Notion node. Full support for create, read, update, and query.

**Rate limit:** 3 requests/second.

### 2.5 n8n Workflow Automation

| Node | Purpose | Type |
|------|---------|------|
| QuickBooks | Full QBO API access | Native |
| HTTP Request | Procore API (via OAuth2) | Generic |
| HTTP Request | Microsoft Graph API for Excel dashboard (via OAuth2) | Generic |
| Gmail / SMTP | PM notifications, customer invoices | Native |
| Webhook | Receive Procore events in real-time | Native trigger |
| Code | Dedup logic, fingerprint hashing, normalization | JavaScript |
| AI Agent | Claude API for fuzzy matching, vendor normalization | Native |
| IF / Switch / Merge | Routing, decision logic, data combining | Built-in |

**What is n8n?** An open-source workflow automation platform (similar to Zapier but self-hostable and far more powerful). It connects systems via visual workflows without requiring full custom software development.

**Bottom line:** n8n can talk to every system in the stack. Procore and Microsoft Graph require HTTP Request nodes (no native nodes), but QBO has a native node. All integrations have been verified.

---

## 3. Three Architecture Options

> **Note:** These architecture options (A, B, C) map to the engagement options in the client-facing proposal (`PROPOSAL-FRONTLINE-PAYMENT-AUTOMATION.md`) as follows:
>
> | Roadmap Option | Proposal Option | Description |
> |---------------|----------------|-------------|
> | Option A | **Proposal Option 1 — Approval Gate + Gap-Filler ($12,000)** | n8n adds human approval gate + fills SmoothX gaps. Excel dashboard via Microsoft Graph API. |
> | Option B | **Proposal Option 2 — Full Replacement ($32,000)** | Complete n8n buildout replaces SmoothX. Excel dashboard via Microsoft Graph API. |
> | Option C | Not proposed | Procore's free connector has too many limitations (US-only, no retention, existing projects require paid Professional Services). Documented here for completeness but not recommended. |

### Option A: Hybrid — SmoothX + n8n (Recommended)

**Core concept: Human approval gate — nothing hits QBO without PM sign-off via Excel.**

```
SmoothX owns:                       n8n owns:
├── Procore → QBO bill sync         ├── Human approval gate (Excel dashboard)
├── QBO → Procore payment sync      ├── 5-layer dedup engine
├── Contact/vendor sync             ├── AI-powered invoice matching
├── Cost code mapping               ├── Excel approval dashboard (OneDrive)
├── Retention/retainage             ├── PM notifications & approval workflows
├── Progress claims                 ├── Customer invoice generation
└── API change absorption           ├── Follow-up & escalation
                                    └── Procore webhook event routing

                                    Dedup data: n8n internal datastore + Excel + QBO custom fields
                                    Audit trail: Excel audit log tab + n8n execution logs + QBO records
                                    Dashboards: Excel approval dashboard + Procore + QBO native reporting
```

**How it works:** SmoothX continues handling the Procore↔QBO financial sync (what it already does today). n8n sits alongside it, reading from both Procore and QBO to run dedup checks, AI matching, PM notifications, and customer invoicing. **n8n never writes vendor bills to QBO** — SmoothX owns that. n8n only writes customer invoices to QBO (which SmoothX doesn't handle). The Excel approval dashboard (via Microsoft Graph API) provides centralized visibility, PM approval workflows, duplicate alerts, and a complete audit trail.

| Pros | Cons |
|------|------|
| Fastest to production (10 weeks) | Monthly SmoothX subscription continues |
| No risk of breaking existing Procore↔QBO sync | Two systems to monitor instead of one |
| Retention, progress claims, cost codes handled by battle-tested software | Dependency on SmoothX for core financial sync |
| n8n effort focused entirely on the novel problem (dedup + automation) | |
| Excel dashboard provides centralized visibility and audit trail | |
| Human approval gate ensures nothing hits QBO without PM sign-off | |
| PMs already know Excel — minimal change management | |
| API changes absorbed by SmoothX | |

**Cost estimate:**
- SmoothX: ~$200-500/month (current cost, already in budget)
- n8n Cloud: ~$50-100/month or self-hosted (free)
- Claude API (AI matching): ~$20-30/month at expected volumes
- Development time: **10 weeks** to production

---

### Option B: n8n Only — Full DIY (Proposal Option 2: Full Replacement)

```
n8n owns everything:
├── Procore → QBO bill sync          (MUST BUILD — replaces SmoothX)
├── QBO → Procore payment sync       (MUST BUILD — replaces SmoothX)
├── Contact/vendor sync              (MUST BUILD — replaces SmoothX)
├── Cost code mapping                (MUST BUILD — replaces SmoothX)
├── Retention/retainage handling     (MUST BUILD — COMPLEX)
├── Progress claims                  (MUST BUILD — COMPLEX)
├── Human approval gate (Excel dashboard)
├── 5-layer dedup engine
├── AI-powered invoice matching
├── Excel approval dashboard (OneDrive)
├── PM dashboard & approval workflows
├── Customer invoice generation
├── Follow-up & escalation
├── Complete audit trail (Excel)
└── Procore webhook event routing
```

**How it works:** Drop SmoothX entirely. n8n handles all Procore↔QBO sync plus the dedup/automation layer. You build and maintain the financial sync workflows yourself using the Procore HTTP Request node and QBO native node. The Excel approval dashboard (via Microsoft Graph API) provides centralized visibility, PM approval workflows, and audit trail. In the proposal, this is structured as an 18-week engagement that completes the approval gate and gap-filler work first, then adds the SmoothX replacement work.

| Pros | Cons |
|------|------|
| No SmoothX subscription | 18 weeks to production (full engagement) |
| Full control over every data flow | You build and maintain retention/retainage logic |
| No vendor dependency | You build and maintain progress claim mapping |
| Single orchestration platform | You absorb Procore + QBO API changes forever |
| Excel dashboard for approval gate and audit trail | Higher risk of financial posting errors during first 3-6 months |
| | 2-4 hours/week ongoing maintenance minimum |
| | Cost code → Chart of Accounts mapping is non-trivial |
| | Maintenance retainer strongly recommended |

**Cost estimate:**
- SmoothX: $0 (eliminated)
- n8n Cloud: ~$50-100/month or self-hosted (free)
- Claude API: ~$20-30/month
- Development time: **18 weeks** to production
- Ongoing maintenance: **2-4 hours/week** (developer time)

**What you'd actually need to build for Procore↔QBO sync:**

| Workflow | Complexity | Estimated Build Time |
|----------|-----------|---------------------|
| Vendor sync (Procore → QBO) | Medium | 3-5 days |
| Cost code → Chart of Accounts mapping | Medium-High | 5-7 days |
| Commitment → Bill sync (one-way) | Medium | 5-7 days |
| Pay application → Bill payment sync | High | 7-10 days |
| Retention/retainage tracking | **Very High** | 10-15 days |
| Progress claim handling | **Very High** | 10-15 days |
| Bi-directional conflict resolution | High | 5-7 days |
| Negative invoices / credit memos | Medium | 3-5 days |
| Error handling / retry / recovery | Medium | 3-5 days |

**Key risks:**
- QBO has no native retention concept — you need synthetic line items and separate tracking accounts
- Progress claims require tracking cumulative amounts across billing periods
- Bi-directional sync needs conflict resolution rules (last-write-wins or lock-based)
- Both Procore and Intuit change their APIs — you own compatibility forever

---

### Option C: n8n + Procore's Free Built-In QBO Connector (Not Proposed)

> **Note:** This option is documented for completeness but was **not included in the client proposal** due to significant limitations — US-only payment import (Frontline operates in Canada), no retention/retainage, no progress claims, and existing projects require paid Procore Professional Services onboarding.

```
Procore's built-in QBO Connector:    n8n owns:
├── Basic Procore → QBO sync         ├── 5-layer dedup engine
├── Limited cost code mapping        ├── AI-powered invoice matching
└── One-way data flow                ├── Excel approval dashboard
                                     ├── PM notification emails
                                     ├── Customer invoice generation
                                     ├── Follow-up & escalation
                                     └── Custom approval workflows
```

**How it works:** Replace SmoothX with Procore's own free first-party QBO connector. n8n handles the custom automation layer.

| Pros | Cons |
|------|------|
| $0 for the sync layer | **New projects only** — existing in-progress projects require paid Procore Professional Services to onboard |
| First-party support from Procore | Limited to **1 QBO company** per Procore site |
| Fastest possible setup for new projects | **No negative invoice support** |
| | **US-only** payment import (problematic if Frontline operates in Canada) |
| | **No retention/retainage handling** |
| | **No progress claim sync** |
| | **No attachment sync** |
| | Less comprehensive than SmoothX overall |

**Cost estimate:**
- Connector: $0
- n8n Cloud: ~$50/month or self-hosted
- Claude API (AI matching): ~$20-30/month
- Procore Professional Services (for existing projects): Unknown — contact Procore for a quote before selecting this option
- Development time: **6-8 weeks** to production
- Gap-filling for missing features: **Additional 2-4 weeks** if retention/progress claims needed

---

## 4. Option Comparison Matrix

| Factor | Option A: Approval Gate + Gap-Filler | Option B: Full Replacement | Option C: Procore Connector |
|--------|--------------------------------------|---------------------------|---------------------------|
| **Proposal mapping** | **Option 1: Approval Gate + Gap-Filler** | **Option 2: Full Replacement** | Not proposed |
| **Proposal price** | **$12,000** | **$32,000** | N/A |
| **Time to production** | 10 weeks | 18 weeks | 6-8 weeks |
| **New monthly costs** | ~$70-130 | ~$70-130 | ~$70-100 |
| **Total monthly (incl. SmoothX)** | ~$270-630 | ~$70-130 | ~$70-100 |
| **Dev effort (build)** | Low-Medium | Very High | Medium |
| **Dev effort (maintain)** | Low (~1 hr/week) | High (2-4 hrs/week) | Medium |
| **Excel approval dashboard** | Yes (4 tabs) | Yes (4 tabs) | No |
| **PM approvals** | Excel dashboard + email | Excel dashboard + email | Email only |
| **Audit trail** | Excel audit log + n8n logs + QBO | Excel audit log + n8n logs | n8n logs |
| **Retention/retainage** | SmoothX | Must build (complex) | Not supported |
| **Progress claims** | SmoothX | Must build (complex) | Not supported |
| **Existing projects** | Works today | Works today | Requires paid Prof. Services |
| **Canada operations** | Supported | Supported | US-only |
| **API change risk** | Absorbed by SmoothX | You own it | Absorbed by Procore |
| **Financial error risk** | Low | Medium-High (first 6 months) | Medium (feature gaps) |
| **Vendor lock-in** | SmoothX | None | Procore |

### Total Cost of Ownership — Year 1

| Cost Item | Option 1 (Approval Gate + Gap-Filler) | Option 2 (Full Replacement) |
|-----------|---------------------------------------|----------------------------|
| **Development (fixed fee)** | **$12,000** | **$32,000** |
| SmoothX subscription | $2,400-6,000 | $0 (eliminated) |
| n8n Cloud | $600-1,200 | $600-1,200 |
| Claude API (AI matching) | $240-360 | $240-360 |
| **Year 1 Total** | **$15,240-19,560** (incl. existing SmoothX) | **$32,840-33,560** |
| **Monthly run-rate after Year 1** | **$270-630** (incl. SmoothX) | **$70-130** |

**Note:** Development fees are one-time fixed project costs as quoted in the proposal. After go-live, the monthly run-rate drops to subscriptions only (~$70-130/month for n8n + Claude API, plus existing SmoothX for Option 1). Based on $150,000 in confirmed duplicate payment losses last year, even catching just a fraction of duplicates pays for the system within the first few months regardless of option chosen.

### Risk Register

| # | Risk | Likelihood | Impact | Mitigation |
|---|------|-----------|--------|------------|
| 1 | **False positive dedup flags** — system flags legitimate invoices as duplicates, PMs learn to ignore alerts | Medium | High | Tune confidence thresholds during pilot; track false positive rate; target <5% false positive rate before rollout |
| 2 | **SmoothX + n8n dual-write conflict** — both systems write to QBO, creating the very duplicates the system prevents | Low | Critical | **Architectural constraint: n8n NEVER writes vendor bills to QBO.** SmoothX owns bill sync. n8n writes only customer invoices. Enforce via workflow design. |
| 3 | **Subcontractor payment delays** — false-positive blocks delay legitimate payments, risking prompt-payment-act violations and lien deadlines | Medium | High | PM override capability on all blocks; maximum hold time of 24 hours before auto-escalation |
| 4 | **SmoothX vendor dependency** — SmoothX acquired, discontinued, or price increase | Low | High | Option A → B migration path documented; n8n reads from same APIs SmoothX uses, so migration is incremental |
| 5 | **n8n downtime during month-end close** | Low | High | n8n Cloud includes 99.9% SLA; fallback: manual invoice processing for 24-48 hours; no data loss (Excel dashboard retains state) |
| 6 | **Data quality issues during vendor import** — duplicate vendors, inconsistent naming across Procore and QBO | High | Medium | Data cleansing step in Phase 1; AI-assisted vendor name normalization; manual review of vendor mapping before go-live |
| 7 | **PM adoption resistance** — PMs don't trust or use the new system | Low-Medium | Medium | Executive sponsor required; start with one cooperative PM on pilot; demonstrate value with first caught duplicate; Excel is familiar to all PMs — minimal change management; keep PM effort to <1 hour/week |

### Rollback Plan

If the system creates more problems than it solves at any phase:
1. **Phase 1-2:** Disable n8n webhook triggers. Zero impact — no production data has been modified.
2. **Phase 3-4:** Disable customer invoice automation. Revert to manual invoicing. Excel data remains as a reference.
3. **Full rollback:** Disable all n8n workflows. SmoothX continues handling Procore↔QBO sync unaffected (it operates independently). Return to manual invoice processing.

---

## 5. Recommended Path: Option A (Hybrid) — Proposal Option 1

### Why Option 1 (Approval Gate + Gap-Filler) Is Recommended

1. **The duplicate payment problem is the priority.** Every week without the dedup engine and approval gate is another week of exposure to overpayments. Last year, Frontline lost $150,000 to duplicate payments. Option 1 gets the approval gate and dedup engine live in 10 weeks. Option 2 delays full completion to 18 weeks because development time is consumed building plumbing that SmoothX already handles.

2. **The human approval gate is the core value.** Nothing hits QBO without PM sign-off via the Excel dashboard. This single architectural decision — requiring human approval before any payment proceeds — addresses the root cause of duplicate payments. Both options include this.

3. **Retention and progress claims are construction fundamentals.** Frontline operates in commercial construction. Progress billing and retainage are core to the business. Building these from scratch in n8n (Option 2) is a significant engineering effort with high risk of financial errors during the first 6 months.

4. **The savings from dropping SmoothX (~$200-500/month) do not justify the risk.** The annual SmoothX cost (~$2,400-6,000) is a rounding error compared to the $150,000+ annual duplicate payment exposure. Getting the dedup engine live sooner pays for years of SmoothX.

5. **You can always upgrade later.** Start with Option 1. Once the dedup engine is running and proven, revisit whether to bring the Procore↔QBO sync in-house (Option 2). Option 1 is specifically designed as a foundation that Option 2 builds on top of — no rework required.

### When to Choose Each Option

- **Option 1 ($12,000):** Best for most organizations. Solves duplicates with a human approval gate, adds centralized Excel dashboard, PM-friendly approval workflows, and audit trail. No SmoothX replacement risk. Production in 10 weeks.
- **Option 2 ($32,000):** Best if eliminating SmoothX dependency is a strategic priority. Requires ongoing maintenance retainer. Only recommended with Standard or Premium support tier. Production in 18 weeks.

---

## 6. Implementation Roadmap (Option A: Hybrid — Proposal Option 1)

### Phase 1: Foundation — Weeks 1-2 ($3,000)

| Task | Details | Owner |
|------|---------|-------|
| Set up n8n instance | Cloud (~$50-100/mo) or self-hosted on existing infrastructure | Dev/IT |
| Register Procore Developer App | Get Client ID + Client Secret for DMSA (Client Credentials grant) | Admin |
| Register Azure AD application | Azure app registration with Files.ReadWrite and Sites.ReadWrite.All permissions for Microsoft Graph API | Admin/Dev |
| Configure OAuth credentials in n8n | Procore (HTTP Request + OAuth2), QBO (native node), Microsoft Graph (HTTP Request + OAuth2) | Dev |
| Verify API connectivity | Test read operations: list projects, list commitments, list bills, read/write Excel via Graph API | Dev |
| Import vendor list | Pull vendors from both Procore and QBO, normalize names, load into n8n datastore | Dev |

**Milestone:** All systems connected and responding. Azure app registered. Vendor data normalized and loaded.

### Phase 2: Approval Dashboard — Weeks 3-4 ($4,500)

| Task | Details | Owner |
|------|---------|-------|
| Create Excel dashboard template | 4 tabs: Pending Approvals, Recently Processed, Duplicate Alerts, Audit Log | Dev |
| Build Graph API integration | n8n HTTP Request workflows to read/write Excel tables via Microsoft Graph API | Dev |
| Build PM approval workflow | Invoice appears in Pending Approvals tab → PM reviews → marks Approved/Rejected → n8n picks up status change | Dev |
| Build auto-escalation | 4-hour timeout → email PM + finance if no decision made (configurable) | Dev |
| Configure email notifications | PM notification when new invoice needs approval, escalation alerts | Dev |
| Test approval flow end-to-end | Invoice intake → Excel dashboard → PM approval → status update confirmed | Dev + PM |

**Milestone:** Excel approval dashboard operational. PMs can review and approve invoices. Auto-escalation working. Nothing hits QBO without PM sign-off.

### Phase 3: Dedup Engine + Matching — Weeks 5-7 ($3,000)

| Task | Details | Owner |
|------|---------|-------|
| Build Workflow 1: Invoice Intake | Webhook trigger (Procore events) + email trigger + manual upload entry point | Dev |
| Implement fingerprint generator | n8n Code node: SHA256(normalized_vendor + normalized_invoice_num + amount + project) | Dev |
| Implement Layer 1-3 dedup checks | Exact fingerprint match (Excel/n8n datastore), fuzzy match (vendor similarity + amount + date window), amount+vendor pattern match | Dev |
| Implement Layer 4: Procore cross-reference | Query Procore commitments — flag if total payments exceed committed amount | Dev |
| Implement Layer 5: AI analysis | Claude API call via n8n AI Agent node — analyze suspicious patterns | Dev |
| Build Workflow 2: Procore-QBO Matching | Query Procore commitments by vendor + project, compare to incoming invoice | Dev |
| Implement AI matching | Claude API scores match confidence: HIGH (>90%), MEDIUM (60-90%), LOW (<60%) | Dev |
| Log duplicates to Excel | Flagged duplicates appear in Duplicate Alerts tab with details | Dev |
| Configure confidence thresholds | Based on testing results — set auto-approve vs manual review cutoffs | Dev + Finance |
| Test with historical duplicates | Run known past duplicates through the engine, tune thresholds | Dev + Finance |

**Milestone:** Dedup engine catching duplicates. AI matching scoring invoices against Procore commitments. All results logged to Excel dashboard. PMs receiving alerts.

### Phase 4: Customer Invoicing & Rollout — Weeks 8-10 ($1,500)

| Task | Details | Owner |
|------|---------|-------|
| Build Workflow 3: Customer Invoice Generation | On PM approval (Excel status change) → create QBO customer invoice → email to customer | Dev |
| Build Workflow 4: Follow-Up & Escalation | Daily cron: check overdue invoices via Excel → gentle reminder (1-15 days) → firm reminder (16-30) → escalation (31+) | Dev |
| Configure email templates | Customer invoice email, 3-tier reminder sequence, PM escalation alert | Dev + Finance |
| Test end-to-end flow | Invoice received → dedup → match → Excel dashboard → PM approve → customer invoice → follow-up | Dev + PM + Finance |
| Go live on pilot project | Select one active project for production use | Team |
| Roll out to all active projects | Expand from pilot to full project portfolio | Team |
| Train PMs | 30-minute walkthrough: Excel dashboard, what emails to expect, how to approve, where to check status | Dev + PM Lead |
| Document runbooks | How to handle edge cases, manual overrides, system errors | Dev |

**Milestone:** Full end-to-end system running. Customer invoices going out automatically. Follow-ups happening without manual intervention. Team trained. Measurable reduction in duplicate payments.

---

## 7. What Does n8n Actually Build? (For the Technical Team)

### Workflow 1: Invoice Intake & Dedup Engine

```
TRIGGER: Procore Webhook / Email / Manual Upload
    │
    ▼
[Extract Invoice Data] ──► vendor_name, invoice_number, amount, date, project
    │
    ▼
[AI Normalization] ──► "ABC Plumbing LLC" → "abc_plumbing"
                        "INV-001" / "#001" → "001"
    │
    ▼
[Generate Fingerprint] ──► SHA256(normalized_vendor | normalized_invoice_num | amount | normalized_project)
    │
    ▼
[Layer 1: Excel/n8n Datastore Query] ──► Exact fingerprint match? → BLOCK
    │
    ▼
[Layer 2: QBO Query] ──► Same vendor + amount ±1% + date ±30 days? → HOLD
    │
    ▼
[Layer 3: Pattern Check] ──► Same vendor + same amount + within 45-day window? → WARN
    │
    ▼
[Layer 4: Procore Check] ──► Total paid > committed amount? → BLOCK
    │
    ▼
[Layer 5: Claude AI] ──► Suspicious patterns across all data? → HOLD
    │
    ▼
[IF: Duplicate?]
  YES → Log to Excel ("DUPLICATE DETECTED") → Email PM → STOP
  NO  → Forward to Matching Workflow
```

### Workflow 2: Procore-QBO Matching Engine

```
INPUT: Clean (non-duplicate) invoice from Workflow 1
    │
    ▼
[Query Procore Commitments] ──► GET /rest/v1.1/projects/{id}/commitments
    │                           Filter by vendor, get line items + remaining amounts
    ▼
[Claude AI Match] ──► Compare invoice to commitment
                      Score: HIGH (>90%) / MEDIUM (60-90%) / LOW (<60%)
    │
    ▼
[Switch: Confidence]
  HIGH   → Auto-match → Log to Excel → Email PM (FYI)
  MEDIUM → Flag for PM review → Email PM (action required)
  LOW    → Manual match needed → Email PM (urgent)
```

### Workflow 3: Customer Invoice & Follow-Up

```
TRIGGER: Excel status changed to "Approved" (detected by n8n polling via Graph API)
    │
    ▼
[Create QBO Customer Invoice] ──► POST /v3/company/{id}/invoice
    │
    ▼
[Email Invoice to Customer]
    │
    ▼
[Update Excel: "Invoiced to Customer"]
    │
    ▼
[Schedule Follow-Up Timer]

─── Daily Cron ───
    │
    ▼
[Read Excel via Graph API: Overdue Invoices]
    │
    ▼
[Check QBO: Payment Received?]
  YES → Update Excel to "Paid" → Done
  NO  → Escalation ladder:
         1-15 days:  Gentle reminder to customer
         16-30 days: Firm reminder, CC PM
         31+ days:   Escalation to PM + Finance team
```

---

## 8. Excel Approval Dashboard — What Gets Tracked

A single Excel workbook on OneDrive (accessed via Microsoft Graph API) serves as the approval gate and operational dashboard. Procore and QBO remain the systems of record for financial transactions. This replaces the previous multi-database proposal structure.

**4 tabs in the Excel workbook:**

| Tab | Key Columns | Purpose |
|-----|------------|---------|
| **Pending Approvals** | Invoice #, Vendor, Amount, Date, Project, Procore Commitment ID, Match Confidence, PM Assigned, Submitted Date, Days Pending, Approve/Reject (dropdown) | Invoices awaiting PM sign-off — nothing hits QBO until approved here |
| **Recently Processed** | Invoice #, Vendor, Amount, Date, Project, QBO Bill ID, Status (Approved/Rejected), Approved By, Approved Date, QBO Customer Invoice ID, Customer Invoice Sent, Payment Status | Completed invoices with full processing history |
| **Duplicate Alerts** | Invoice #, Vendor, Amount, Date, Project, Fingerprint, Duplicate Of (original invoice ref), Dedup Layer Triggered, Confidence Score, Resolution (dropdown), Resolved By, Resolved Date | Flagged duplicates requiring review — sorted by confidence score |
| **Audit Log** | Timestamp, Action, Invoice #, Vendor, Amount, Performed By, System (n8n/PM/QBO), Details | Complete chronological history of every system action |

**How PMs interact with the dashboard:**
1. PM receives email notification that a new invoice needs approval
2. PM opens the Excel workbook (shared link or OneDrive)
3. PM reviews the invoice details in the **Pending Approvals** tab
4. PM selects "Approved" or "Rejected" in the Approve/Reject column
5. n8n detects the status change via Graph API polling and proceeds accordingly
6. Approved invoices move to **Recently Processed**; rejected invoices are logged with reason

**Auto-escalation:** If an invoice sits in Pending Approvals for >4 hours without a decision, n8n sends an escalation email to the PM and Finance team. This timeout is configurable.

---

## 9. Key Decisions Needed from the Team

| # | Decision | Options | Impact |
|---|----------|---------|--------|
| 1 | **Which engagement option?** | Proposal Option 1 ($12K — Approval Gate + Gap-Filler, recommended) or Option 2 ($32K — Full Replacement) | Timeline, cost, and risk profile |
| 2 | **Executive sponsor** | Who owns this initiative? (VP of Ops, CFO, Controller?) | Accountability and PM adoption |
| 3 | **Project budget approval** | Option 1: ~$15-20K / Option 2: ~$33-34K (see Year 1 TCO table) | Financial authorization |
| 4 | **n8n hosting?** | Cloud (~$50-100/mo, managed) vs. self-hosted (free, you manage server) | IT overhead |
| 5 | **Microsoft Azure app registration** | Who registers the Azure AD app for Microsoft Graph API access? (Admin or Dev) | Required for Excel dashboard integration |
| 6 | **Pilot project selection** | Which active project to test on first? | Ideally medium complexity with cooperative PM and subs |
| 7 | **Designated approver(s)** | Which PM(s) will be responsible for approving invoices in the Excel dashboard? | Approval gate workflow design |
| 8 | **Customer invoice format** | QBO native email, custom PDF template, or both? | Customer-facing presentation |
| 9 | **Follow-up timing** | Net 30/45/60 default? Per-customer? | Collections aggressiveness |
| 10 | **AI model for matching** | Claude (recommended for accuracy — used throughout this spec) vs. GPT-4 (alternative) | Cost, accuracy |
| 11 | **Who maintains n8n long-term?** | Veteran Vectors retainer (Essential/Standard/Premium), in-house developer, or other contractor? | Sustainability |
| 12 | **Parallel run duration** | How long do old + new processes run side-by-side? (Recommended: 2-4 weeks) | Rollback safety |
| 13 | **Go/no-go criteria for pilot** | What metrics must be met before full rollout? (e.g., <5% false positive rate, zero missed invoices for 2 weeks) | Objective success definition |

**Target decision date:** These decisions are needed before development can begin. Every week of delay extends the timeline by one week and continues the duplicate payment exposure.

---

## 10. Expected Outcomes

| Metric | Current State | 90-Day Target | 6-Month Target | How Measured |
|--------|--------------|---------------|----------------|-------------|
| Duplicate payment rate | $150K/year confirmed | Reduce by 80% | Reduce by 95%+ | Count of duplicates caught vs. total invoices processed |
| Invoice processing time | 2-5 days (manual matching) | <8 hours (receipt to QBO entry) | <4 hours | Timestamp difference: Excel "Received" to "Matched" |
| PM time on invoice admin | 5-10 hours/week | 3-5 hours/week (learning curve) | <1 hour/week | PM self-report + Excel dashboard activity |
| Customer invoice turnaround | 3-7 days after approval | Same day for high-confidence matches | Same day for all matches | Timestamp: PM approval to customer email sent |
| Follow-up on overdue invoices | Ad hoc / inconsistent | Automated 3-tier escalation | Automated with <2% missed | Excel follow-up count vs. overdue invoice count |
| Financial visibility | Fragmented across 3+ systems | Excel dashboard with centralized approval gate and audit trail | Fully operational | Dashboard availability + data freshness |
| Audit trail | Partial / manual | Complete automated log for all n8n-processed invoices | 100% coverage | Excel Audit Log tab completeness |

**90-day review checkpoint:** At 90 days post-pilot, the executive sponsor and development lead review these metrics against targets and decide whether to proceed with full rollout, adjust thresholds, or pause.

---

## 11. Next Steps

1. **Review this report and the client proposal** (`PROPOSAL-FRONTLINE-PAYMENT-AUTOMATION.md`) — the proposal presents two engagement options with pricing and timelines
2. **Select an engagement option** (Proposal Option 1 or Option 2) — or discuss with Veteran Vectors
3. **Assign an executive sponsor** — someone who will own this initiative and drive PM adoption
4. **Conduct a limited AP audit** (last 6 months) to validate the duplicate payment rate and establish a measured baseline
5. **Decide on the 13 open questions** in Section 9 — target decision date: within 1 week
6. **Register a Procore Developer App** (Admin task — needed for API access regardless of option chosen)
7. **Register an Azure AD application** (Admin/Dev task — needed for Microsoft Graph API access to Excel on OneDrive)
8. **Select a pilot project** and identify the PM who will be the first user and designated approver
9. **Schedule an audit call** (60-90 min, free) with Veteran Vectors to walk through your actual Procore, QBO, and SmoothX setup before signing
10. **Upon alignment,** Veteran Vectors will deliver a formal Statement of Work (SOW) for signature

---

*This report is based on verified API documentation from Procore Developer Portal, Intuit Developer Portal, Microsoft Graph API docs, n8n documentation, and the Procore App Marketplace. All endpoints and capabilities have been confirmed as available.*
