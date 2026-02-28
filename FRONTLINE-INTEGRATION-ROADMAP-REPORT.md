# Frontline Holdings — Integration Roadmap & Options Report
## Payment Automation: Procore, QuickBooks Online, Notion, n8n

**Date:** February 28, 2026
**Prepared for:** Frontline Holdings Leadership Team
**Purpose:** Decision-ready summary of integration options, verified API capabilities, architecture paths, and implementation timeline

**How to read this report:** Sections 1-6, 9-11 are written for the leadership team. Sections 7-8 contain technical implementation details for the development team and can be skipped by non-technical readers.

**Companion files in this repository:**
- `PROPOSAL-FRONTLINE-PAYMENT-AUTOMATION.md` — Client-facing proposal with 3 engagement options and pricing
- `FRONTLINE-PAYMENT-AUTOMATION.md` — Full technical architecture document
- `n8n-workflows/` — Importable n8n workflow JSON templates (3 workflows)
- `notion-2nd-brain/SETUP-GUIDE.md` — Step-by-step Notion database setup guide (Options 2 & 3)
- `REVIEW-ASSESSMENT-REPORT.md` — Prior assessment and review findings

---

## 1. Situation Summary

Frontline Holdings faces an estimated **$720,000+/year exposure** to duplicate and triple payments. This estimate is based on an industry-standard model: ~500 invoices/month at ~$15,000 average value with a conservative 2% duplicate rate yields ~$150,000/month in potential overpayments, with an industry-average recovery rate of ~60% (40% loss = ~$720K/year). **This figure should be validated with an AP audit of the last 6 months before presenting as confirmed loss to the board.**

The root cause: invoices enter through multiple channels (email, Procore, mail) with no cross-system deduplication. The current stack — Procore (project management), QuickBooks Online (accounting), and SmoothX (Procore-QBO sync middleware) — has no automated layer to catch duplicates, match invoices to commitments, notify PMs, or follow up on unpaid customer invoices.

This report covers:
1. What each system's API can actually do (verified against official documentation)
2. Three architecture options with honest trade-offs
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

### 2.3 Notion API

| Capability | API Endpoint | Verified? |
|-----------|-------------|-----------|
| Create tracking databases | `POST /v1/databases` | Yes |
| Query with filters (dedup check) | `POST /v1/databases/{id}/query` | Yes |
| Create invoice records | `POST /v1/pages` | Yes |
| Update status/fields | `PATCH /v1/pages/{id}` | Yes |
| Full-text search | `POST /v1/search` | Yes |

**n8n connection:** Native Notion node. Full support for create, read, update, and query.

**Rate limit:** 3 requests/second (adequate for invoice volumes — not a bottleneck).

### 2.4 n8n Workflow Automation

| Node | Purpose | Type |
|------|---------|------|
| QuickBooks | Full QBO API access | Native |
| Notion | Database CRUD | Native |
| HTTP Request | Procore API (via OAuth2) | Generic |
| Gmail / SMTP | PM notifications, customer invoices | Native |
| Webhook | Receive Procore events in real-time | Native trigger |
| Code | Dedup logic, fingerprint hashing, normalization | JavaScript |
| AI Agent | Claude API for fuzzy matching, vendor normalization | Native |
| IF / Switch / Merge | Routing, decision logic, data combining | Built-in |

**What is n8n?** An open-source workflow automation platform (similar to Zapier but self-hostable and far more powerful). It connects systems via visual workflows without requiring full custom software development.

**Bottom line:** n8n can talk to every system in the stack. Procore requires HTTP Request (no native node), but everything else has native nodes.

---

## 3. Three Architecture Options

> **Note:** These architecture options (A, B, C) map to the engagement options in the client-facing proposal (`PROPOSAL-FRONTLINE-PAYMENT-AUTOMATION.md`) as follows:
>
> | Roadmap Option | Proposal Option | Description |
> |---------------|----------------|-------------|
> | Option A (without Notion) | **Proposal Option 1 — Gap-Filler ($14,000)** | n8n fills SmoothX gaps. No Notion. Email-based PM approvals. |
> | Option A (with Notion) | **Proposal Option 2 — Gap-Filler + 2nd Brain ($22,000)** | n8n fills SmoothX gaps + Notion as 2nd source of truth. |
> | Option B | **Proposal Option 3 — Full Replacement ($42,000)** | Complete n8n buildout replaces SmoothX. Includes Notion. |
> | Option C | Not proposed | Procore's free connector has too many limitations (US-only, no retention, existing projects require paid Professional Services). Documented here for completeness but not recommended. |

### Option A: Hybrid — SmoothX + n8n (Recommended)

This option has two variants in the proposal:

**Variant 1 — Without Notion (Proposal Option 1: Gap-Filler)**

```
SmoothX owns:                       n8n owns:
├── Procore → QBO bill sync         ├── 5-layer dedup engine
├── QBO → Procore payment sync      ├── AI-powered invoice matching
├── Contact/vendor sync             ├── PM email notifications
├── Cost code mapping               ├── Customer invoice generation
├── Retention/retainage             ├── Follow-up & escalation
├── Progress claims                 └── Procore webhook event routing
└── API change absorption
                                    Dedup data: n8n internal datastore + QBO custom fields
                                    Audit trail: n8n execution logs + QBO records
                                    Dashboards: Procore + QBO native reporting
```

**Variant 2 — With Notion (Proposal Option 2: Gap-Filler + 2nd Brain)**

```
SmoothX owns:                       n8n owns:
├── Procore → QBO bill sync         ├── 5-layer dedup engine (Notion-backed)
├── QBO → Procore payment sync      ├── AI-powered invoice matching
├── Contact/vendor sync             ├── Notion "2nd Brain" (6 databases)
├── Cost code mapping               ├── PM dashboard & approval workflows
├── Retention/retainage             ├── Customer invoice generation
├── Progress claims                 ├── Follow-up & escalation
└── API change absorption           ├── Real-time financial dashboards
                                    ├── Complete audit trail database
                                    └── Procore webhook event routing
```

**How it works:** SmoothX continues handling the Procore↔QBO financial sync (what it already does today). n8n sits alongside it, reading from both Procore and QBO to run dedup checks, AI matching, PM notifications, and customer invoicing. **n8n never writes vendor bills to QBO** — SmoothX owns that. n8n only writes customer invoices to QBO (which SmoothX doesn't handle). Notion (Option 2 only) adds centralized visibility, dashboards, and a business-user-facing audit trail.

| Pros | Cons |
|------|------|
| Fastest to production (4-6 weeks for n8n workflows) | Monthly SmoothX subscription continues |
| No risk of breaking existing Procore↔QBO sync | Two systems to monitor instead of one |
| Retention, progress claims, cost codes handled by battle-tested software | Dependency on SmoothX for core financial sync |
| n8n effort focused entirely on the novel problem (dedup + automation) | Without Notion (Option 1): limited visibility and audit trail |
| API changes absorbed by SmoothX | |
| With Notion (Option 2): centralized dashboards and audit trail | With Notion (Option 2): additional subscription + PM adoption needed |

**Cost estimate:**
- SmoothX: ~$200-500/month (current cost, already in budget)
- n8n Cloud: ~$50/month (Starter) or self-hosted (free)
- Claude API (AI matching): ~$20-50/month at expected volumes
- Notion Team (Option 2 only): ~$50/month
- Development time: **8 weeks** (Option 1) or **12 weeks** (Option 2) to production

---

### Option B: n8n Only — Full DIY (Proposal Option 3: Full Replacement)

```
n8n owns everything:
├── Procore → QBO bill sync          (MUST BUILD — replaces SmoothX)
├── QBO → Procore payment sync       (MUST BUILD — replaces SmoothX)
├── Contact/vendor sync              (MUST BUILD — replaces SmoothX)
├── Cost code mapping                (MUST BUILD — replaces SmoothX)
├── Retention/retainage handling     (MUST BUILD — COMPLEX)
├── Progress claims                  (MUST BUILD — COMPLEX)
├── 5-layer dedup engine (Notion-backed)
├── AI-powered invoice matching
├── Notion "2nd Brain" (6+ databases)
├── PM dashboard & approval workflows
├── Customer invoice generation
├── Follow-up & escalation
├── Real-time financial dashboards
├── Complete audit trail database
└── Procore webhook event routing
```

**How it works:** Drop SmoothX entirely. n8n handles all Procore↔QBO sync plus the dedup/automation layer. You build and maintain the financial sync workflows yourself using the Procore HTTP Request node and QBO native node. Notion serves as the 2nd Brain (carried over from Option 2). In the proposal, this is structured as a 20-week engagement that completes all of Option 2 first (Phases 1-4, weeks 1-12), then adds the SmoothX replacement work (Phases 5-6, weeks 13-20).

| Pros | Cons |
|------|------|
| No SmoothX subscription | 20 weeks to production (full engagement) |
| Full control over every data flow | You build and maintain retention/retainage logic |
| No vendor dependency | You build and maintain progress claim mapping |
| Single orchestration platform | You absorb Procore + QBO API changes forever |
| | Higher risk of financial posting errors during first 3-6 months |
| | 2-4 hours/week ongoing maintenance minimum |
| | Cost code → Chart of Accounts mapping is non-trivial |
| | Maintenance retainer strongly recommended |

**Cost estimate:**
- SmoothX: $0 (eliminated)
- n8n Cloud: ~$50/month or self-hosted (free)
- Claude API: ~$20-50/month
- Notion Team: ~$50/month
- Development time: **20 weeks** to production
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
└── One-way data flow                ├── Notion "2nd Brain" logging
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
- Claude API (AI matching): ~$20-50/month
- Procore Professional Services (for existing projects): Unknown — contact Procore for a quote before selecting this option
- Development time: **6-8 weeks** to production
- Gap-filling for missing features: **Additional 2-4 weeks** if retention/progress claims needed

---

## 4. Option Comparison Matrix

| Factor | Option A (no Notion) | Option A (with Notion) | Option B: n8n Only | Option C: Procore Connector |
|--------|---------------------|----------------------|-------------------|---------------------------|
| **Proposal mapping** | **Option 1: Gap-Filler** | **Option 2: Gap-Filler + 2nd Brain** | **Option 3: Full Replacement** | Not proposed |
| **Proposal price** | **$14,000** | **$22,000** | **$42,000** | N/A |
| **Time to production** | 8 weeks | 12 weeks | 20 weeks | 6-8 weeks |
| **New monthly costs** | ~$70-150 | ~$120-200 | ~$120-200 | ~$70-100 |
| **Total monthly (incl. SmoothX)** | ~$270-650 | ~$320-700 | ~$120-200 | ~$70-100 |
| **Dev effort (build)** | Low | Medium | Very High | Medium |
| **Dev effort (maintain)** | Low (~1 hr/week) | Low (~1 hr/week) | High (2-4 hrs/week) | Medium |
| **Notion dashboards** | No | Yes (6 databases, 6 views) | Yes (6+ databases) | No |
| **PM approvals** | Email only | Notion dashboard + email | Notion dashboard + email | Email only |
| **Audit trail** | n8n logs + QBO | Notion database (business-user-facing) | Notion database | n8n logs |
| **Retention/retainage** | SmoothX | SmoothX | Must build (complex) | Not supported |
| **Progress claims** | SmoothX | SmoothX | Must build (complex) | Not supported |
| **Existing projects** | Works today | Works today | Works today | Requires paid Prof. Services |
| **Canada operations** | Supported | Supported | Supported | US-only |
| **API change risk** | Absorbed by SmoothX | Absorbed by SmoothX | You own it | Absorbed by Procore |
| **Financial error risk** | Low | Low | Medium-High (first 6 months) | Medium (feature gaps) |
| **Vendor lock-in** | SmoothX | SmoothX + Notion | Notion only | Procore |

### Total Cost of Ownership — Year 1

| Cost Item | Option 1 (Gap-Filler) | Option 2 (+ 2nd Brain) | Option 3 (Full Replacement) |
|-----------|----------------------|------------------------|----------------------------|
| **Development (fixed fee)** | **$14,000** | **$22,000** | **$42,000** |
| SmoothX subscription | $2,400-6,000 | $2,400-6,000 | $0 (eliminated) |
| n8n Cloud | $600-1,200 | $600-1,200 | $600-1,200 |
| Notion Team plan | $0 | $600 | $600 |
| Claude API (AI matching) | $240-600 | $240-600 | $240-600 |
| **Year 1 Total** | **$17,240-21,800** | **$25,840-30,400** | **$43,440-44,400** |
| **Monthly run-rate after Year 1** | **$270-650** | **$320-700** | **$120-200** |

**Note:** Development fees are one-time fixed project costs as quoted in the proposal. After go-live, the monthly run-rate drops to subscriptions only. Even catching just one duplicate payment per month ($15,000 average) pays for the system within the first 1-3 months regardless of option chosen.

### Risk Register

| # | Risk | Likelihood | Impact | Mitigation |
|---|------|-----------|--------|------------|
| 1 | **False positive dedup flags** — system flags legitimate invoices as duplicates, PMs learn to ignore alerts | Medium | High | Tune confidence thresholds during pilot; track false positive rate; target <5% false positive rate before rollout |
| 2 | **SmoothX + n8n dual-write conflict** — both systems write to QBO, creating the very duplicates the system prevents | Low | Critical | **Architectural constraint: n8n NEVER writes vendor bills to QBO.** SmoothX owns bill sync. n8n writes only customer invoices. Enforce via workflow design. |
| 3 | **Subcontractor payment delays** — false-positive blocks delay legitimate payments, risking prompt-payment-act violations and lien deadlines | Medium | High | PM override capability on all blocks; maximum hold time of 24 hours before auto-escalation |
| 4 | **SmoothX vendor dependency** — SmoothX acquired, discontinued, or price increase | Low | High | Option A → B migration path documented; n8n reads from same APIs SmoothX uses, so migration is incremental |
| 5 | **n8n downtime during month-end close** | Low | High | n8n Cloud includes 99.9% SLA; fallback: manual invoice processing for 24-48 hours; no data loss (Notion retains state) |
| 6 | **Notion as financial system of record** — auditors question whether Notion meets documentation requirements | Medium | Medium | Consult external auditor before go-live; configure automated Notion database exports (CSV) for backup/retention; establish 7-year data retention policy |
| 7 | **Data quality issues during vendor import** — duplicate vendors, inconsistent naming across Procore and QBO | High | Medium | Data cleansing step in Phase 1; AI-assisted vendor name normalization; manual review of vendor mapping before go-live |
| 8 | **PM adoption resistance** — PMs don't trust or use the new system | Medium | High | Executive sponsor required; start with one cooperative PM on pilot; demonstrate value with first caught duplicate; keep PM effort to <1 hour/week |

### Rollback Plan

If the system creates more problems than it solves at any phase:
1. **Phase 1-2:** Disable n8n webhook triggers. Zero impact — no production data has been modified.
2. **Phase 3-4:** Disable customer invoice automation. Revert to manual invoicing. Notion data remains as a reference.
3. **Full rollback:** Disable all n8n workflows. SmoothX continues handling Procore↔QBO sync unaffected (it operates independently). Return to manual invoice processing.

---

## 5. Recommended Path: Option A (Hybrid) — Proposal Option 2

### Why Option 2 (Gap-Filler + 2nd Brain) Is Recommended

1. **The duplicate payment problem is the priority.** Every week without the dedup engine is another week of exposure to overpayments. Option 2 gets the dedup engine live in 12 weeks with full visibility. Option 3 delays full completion to 20 weeks because development time is consumed building plumbing that SmoothX already handles.

2. **Notion adds transformative visibility.** Option 1 (Gap-Filler without Notion) solves the duplicate payment problem, but PMs, Finance, and Leadership still lack centralized dashboards, a business-user audit trail, and one-click approval workflows. The $8,000 difference between Option 1 and Option 2 delivers significant operational value.

3. **Retention and progress claims are construction fundamentals.** Frontline operates in commercial construction. Progress billing and retainage are core to the business. Building these from scratch in n8n (Option 3) is a significant engineering effort with high risk of financial errors during the first 6 months.

4. **The savings from dropping SmoothX (~$200-500/month) do not justify the risk.** The annual SmoothX cost (~$2,400-6,000) is a rounding error compared to the $720,000+ annual duplicate payment exposure. Getting the dedup engine live sooner pays for years of SmoothX.

5. **You can always upgrade later.** Start with Option 1 or 2. Once the dedup engine is running and proven, revisit whether to bring the Procore↔QBO sync in-house (Option 3). Option 2 is specifically designed as a foundation that Option 3 builds on top of — no rework required.

### When to Choose Each Option

- **Option 1 ($14,000):** Best if budget is tight and the priority is solving the duplicate payment problem as quickly and cheaply as possible. Team is comfortable with Procore + QBO native reporting.
- **Option 2 ($22,000):** Best for most organizations. Solves duplicates + adds centralized visibility, PM-friendly dashboards, and audit trail. No SmoothX replacement risk.
- **Option 3 ($42,000):** Best if eliminating SmoothX dependency is a strategic priority. Requires ongoing maintenance retainer. Only recommended with Standard or Premium support tier.

---

## 6. Implementation Roadmap (Option A: Hybrid — Proposal Options 1 & 2)

### Phase 1: Foundation — Weeks 1-2

| Task | Details | Owner |
|------|---------|-------|
| Set up Notion workspace | Create 6 databases: Invoice Tracker, Projects, Vendors, Customers, Audit Log, Payments | Dev |
| Configure Notion views | PM Dashboard, Duplicate Alert Board, Cash Flow Board, Project Summary, Vendor History, Audit Trail | Dev |
| Set up n8n instance | Cloud ($50/mo) or self-hosted on existing infrastructure | Dev/IT |
| Register Procore Developer App | Get Client ID + Client Secret for DMSA (Client Credentials grant) | Admin |
| Configure OAuth credentials in n8n | Procore (HTTP Request + OAuth2), QBO (native node), Notion (native node) | Dev |
| Verify API connectivity | Test read operations: list projects, list commitments, list bills, query Notion | Dev |
| Import vendor list | Pull vendors from both Procore and QBO, normalize names, load into Notion Vendors DB | Dev |

**Milestone:** All 3 systems connected and responding. Notion databases populated with current vendor/project data.

### Phase 2: Dedup Engine — Weeks 3-4

| Task | Details | Owner |
|------|---------|-------|
| Build Workflow 1: Invoice Intake | Webhook trigger (Procore events) + email trigger + manual upload entry point | Dev |
| Implement fingerprint generator | n8n Code node: SHA256(normalized_vendor + normalized_invoice_num + amount + project) | Dev |
| Implement Layer 1-3 dedup checks | Exact fingerprint match (Notion), fuzzy match (vendor similarity + amount + date window), amount+vendor pattern match | Dev |
| Implement Layer 4: Procore cross-reference | Query Procore commitments — flag if total payments exceed committed amount | Dev |
| Implement Layer 5: AI analysis | Claude API call via n8n AI Agent node — analyze suspicious patterns | Dev |
| Build duplicate alert emails | Email templates for PM notification when duplicate detected | Dev |
| Test with historical duplicates | Run known past duplicates through the engine, tune thresholds | Dev + Finance |

**Milestone:** Dedup engine catching duplicates on test data. PMs receiving alert emails.

### Phase 3: Matching Engine — Weeks 5-6

| Task | Details | Owner |
|------|---------|-------|
| Build Workflow 2: Procore-QBO Matching | Query Procore commitments by vendor + project, compare to incoming invoice | Dev |
| Implement AI matching | Claude API scores match confidence: HIGH (>90%), MEDIUM (60-90%), LOW (<60%) | Dev |
| Build PM approval flow | Email with invoice details + Procore commitment match + approve/reject action | Dev |
| Connect Notion logging | Every invoice creates/updates a record in Invoice Tracker with full audit trail | Dev |
| Configure confidence thresholds | Based on Phase 2 testing results — set auto-approve vs manual review cutoffs | Dev + Finance |
| Test with live Procore data | Run against real commitments in sandbox/test project | Dev + PM |

**Milestone:** Invoices automatically matched to Procore commitments. PMs approving via email. Full audit trail in Notion.

### Phase 4: Customer Invoicing & Follow-Up — Weeks 7-8

| Task | Details | Owner |
|------|---------|-------|
| Build Workflow 3: Customer Invoice Generation | On PM approval → create QBO customer invoice → email to customer | Dev |
| Build Workflow 4: Follow-Up & Escalation | Daily cron: check overdue invoices → gentle reminder (1-15 days) → firm reminder (16-30) → escalation (31+) | Dev |
| Configure email templates | Customer invoice email, 3-tier reminder sequence, PM escalation alert | Dev + Finance |
| Test end-to-end flow | Invoice received → dedup → match → approve → customer invoice → follow-up | Dev + PM + Finance |
| Go live on pilot project | Select one active project for production use | Team |

**Milestone:** Full end-to-end system running on a pilot project. Customer invoices going out automatically. Follow-ups happening without manual intervention.

### Phase 5: Rollout & Optimization — Weeks 9-12

| Task | Details | Owner |
|------|---------|-------|
| Roll out to all active projects | Expand from pilot to full project portfolio | Team |
| Monitor and tune AI matching | Review false positives/negatives, adjust prompts and thresholds | Dev + Finance |
| Build dashboards | Notion views for: duplicates caught this month, avg. processing time, overdue receivables, savings report | Dev |
| Document runbooks | How to handle edge cases, manual overrides, system errors | Dev |
| Train PMs | 30-minute walkthrough: what emails to expect, how to approve, where to check status | Dev + PM Lead |

**Milestone:** System running across all projects. Team trained. Measurable reduction in duplicate payments.

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
[Layer 1: Notion Query] ──► Exact fingerprint match? → BLOCK
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
  YES → Log to Notion ("DUPLICATE DETECTED") → Email PM → STOP
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
  HIGH   → Auto-match → Log to Notion → Email PM (FYI)
  MEDIUM → Flag for PM review → Email PM (action required)
  LOW    → Manual match needed → Email PM (urgent)
```

### Workflow 3: Customer Invoice & Follow-Up

```
TRIGGER: Notion status changed to "Approved"
    │
    ▼
[Create QBO Customer Invoice] ──► POST /v3/company/{id}/invoice
    │
    ▼
[Email Invoice to Customer]
    │
    ▼
[Update Notion: "Invoiced to Customer"]
    │
    ▼
[Schedule Follow-Up Timer]

─── Daily Cron ───
    │
    ▼
[Query Notion: Overdue Invoices]
    │
    ▼
[Check QBO: Payment Received?]
  YES → Update Notion to "Paid" → Done
  NO  → Escalation ladder:
         1-15 days:  Gentle reminder to customer
         16-30 days: Firm reminder, CC PM
         31+ days:   Escalation to PM + Finance team
```

---

## 8. Notion "2nd Brain" — What Gets Tracked

Six interconnected databases serve as the 2nd source of truth (Procore and QBO remain the systems of record for financial transactions):

| Database | Key Fields | Purpose |
|----------|-----------|---------|
| **Invoice Tracker** | Invoice #, Fingerprint, Vendor, Amount, Date, Project, Procore Commitment ID, QBO Bill ID, Status, Match Confidence, PM, Follow-Up Count | Master record for every invoice |
| **Projects** | Project Name, Procore ID, Customer, PM, Contract Value, Billed to Date, Remaining | Project-level financial view |
| **Vendors** | Name, Normalized Name, Procore ID, QBO ID, Contact Email, YTD Payments | Cross-system vendor mapping |
| **Customers** | Name, QBO ID, Email, Payment Terms, Outstanding Balance | AR tracking |
| **Audit Log** | Timestamp, Action, Invoice, Performed By, System | Complete history of every system action |
| **Payments** | Payment Reference, Invoice, Vendor, Amount, Payment Date, Method, QBO Payment ID, Cleared, Potential Duplicate | Payment tracking with duplicate flagging |

**Pre-built views (6 total):**
- PM Dashboard — each PM's pending approvals + overdue items
- Duplicate Alert Board — all flagged duplicates needing review
- Cash Flow Board — outstanding receivables by customer
- Project Financial Summary — per-project billing status
- Vendor Payment History — all payments to each vendor (dedup verification)
- Audit Trail — complete history of all system actions (from Audit Log database)

---

## 9. Key Decisions Needed from the Team

| # | Decision | Options | Impact |
|---|----------|---------|--------|
| 1 | **Which engagement option?** | Proposal Option 1 ($14K — Gap-Filler), Option 2 ($22K — + 2nd Brain, recommended), or Option 3 ($42K — Full Replacement) | Timeline, cost, and risk profile |
| 2 | **Executive sponsor** | Who owns this initiative? (VP of Ops, CFO, Controller?) | Accountability and PM adoption |
| 3 | **Project budget approval** | Option 1: ~$17-22K / Option 2: ~$26-30K / Option 3: ~$43-44K (see Year 1 TCO table) | Financial authorization |
| 4 | **n8n hosting?** | Cloud (~$50/mo, managed) vs. self-hosted (free, you manage server) | IT overhead |
| 5 | **Pilot project selection** | Which active project to test on first? | Ideally medium complexity with cooperative PM and subs |
| 6 | **PM approval method** | Email links (Option 1), Notion buttons + email (Options 2/3), or Slack integration? | UX for PMs |
| 7 | **Customer invoice format** | QBO native email, custom PDF template, or both? | Customer-facing presentation |
| 8 | **Follow-up timing** | Net 30/45/60 default? Per-customer? | Collections aggressiveness |
| 9 | **AI model for matching** | Claude (recommended for accuracy — used throughout this spec) vs. GPT-4 (alternative) | Cost, accuracy |
| 10 | **Who maintains n8n long-term?** | Veteran Vectors retainer (Essential/Standard/Premium), in-house developer, or other contractor? | Sustainability |
| 11 | **Parallel run duration** | How long do old + new processes run side-by-side? (Recommended: 2-4 weeks) | Rollback safety |
| 12 | **Go/no-go criteria for pilot** | What metrics must be met before full rollout? (e.g., <5% false positive rate, zero missed invoices for 2 weeks) | Objective success definition |
| 13 | **Notion as financial record** | Options 2/3 only: Consult external auditor — is Notion acceptable for 7-year financial audit trail retention? | Compliance |

**Target decision date:** These decisions are needed before development can begin. Every week of delay extends the timeline by one week and continues the duplicate payment exposure.

---

## 10. Expected Outcomes

| Metric | Current State | 90-Day Target | 6-Month Target | How Measured |
|--------|--------------|---------------|----------------|-------------|
| Duplicate payment rate | ~2% (est. — validate with AP audit) | Reduce by 80% | Reduce by 90%+ | Count of duplicates caught vs. total invoices processed |
| Invoice processing time | 2-5 days (manual matching) | <8 hours (receipt to QBO entry) | <4 hours | Timestamp difference: Notion "Received" to "Matched" |
| PM time on invoice admin | 5-10 hours/week | 3-5 hours/week (learning curve) | <1 hour/week | PM self-report + Notion activity logs |
| Customer invoice turnaround | 3-7 days after approval | Same day for high-confidence matches | Same day for all matches | Timestamp: PM approval to customer email sent |
| Follow-up on overdue invoices | Ad hoc / inconsistent | Automated 3-tier escalation | Automated with <2% missed | Notion follow-up count vs. overdue invoice count |
| Financial visibility | Fragmented across 3+ systems | Improved via Procore + QBO (Option 1) or single Notion dashboard (Options 2/3) | Fully operational | Dashboard availability + data freshness |
| Audit trail | Partial / manual | Complete automated log for all n8n-processed invoices | 100% coverage | Audit Log database completeness |

**90-day review checkpoint:** At 90 days post-pilot, the executive sponsor and development lead review these metrics against targets and decide whether to proceed with full rollout, adjust thresholds, or pause.

---

## 11. Next Steps

1. **Review this report and the client proposal** (`PROPOSAL-FRONTLINE-PAYMENT-AUTOMATION.md`) — the proposal presents three engagement options with pricing and timelines
2. **Select an engagement option** (Proposal Option 1, 2, or 3) — or discuss a hybrid approach with Veteran Vectors
3. **Assign an executive sponsor** — someone who will own this initiative and drive PM adoption
4. **Conduct a limited AP audit** (last 6 months) to validate the duplicate payment rate and establish a measured baseline
5. **Decide on the 13 open questions** in Section 9 — target decision date: within 1 week
6. **Register a Procore Developer App** (Admin task — needed for API access regardless of option chosen)
7. **Select a pilot project** and identify the PM who will be the first user
8. **Schedule a walkthrough call** with Veteran Vectors to discuss questions, preferences, and option selection
9. **Upon alignment,** Veteran Vectors will deliver a formal Statement of Work (SOW) for signature

---

*This report is based on verified API documentation from Procore Developer Portal, Intuit Developer Portal, Notion API docs, n8n documentation, and the Procore App Marketplace. All endpoints and capabilities have been confirmed as available.*
