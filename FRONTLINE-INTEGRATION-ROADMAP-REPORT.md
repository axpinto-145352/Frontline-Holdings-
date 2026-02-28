# Frontline Holdings — Integration Roadmap & Options Report
## Payment Automation: Procore, QuickBooks Online, Notion, n8n

**Date:** February 28, 2026
**Prepared for:** Frontline Holdings Leadership Team
**Purpose:** Decision-ready summary of integration options, verified API capabilities, architecture paths, and implementation timeline

---

## 1. Situation Summary

Frontline Holdings is losing an estimated **$720,000+/year** to duplicate and triple payments caused by invoices entering through multiple channels (email, Procore, mail) with no cross-system deduplication. The current stack — Procore (project management), QuickBooks Online (accounting), and SmoothX (Procore-QBO sync middleware) — has no automated layer to catch duplicates, match invoices to commitments, notify PMs, or follow up on unpaid customer invoices.

This report covers:
1. What each system's API can actually do (verified against official documentation)
2. Three architecture options with honest trade-offs
3. A phased implementation roadmap with timelines
4. Cost comparison across options

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

**Authentication:** OAuth 2.0 Client Credentials grant (DMSA) recommended for automated workflows. Tokens expire every 1.5 hours; n8n handles refresh automatically. All calls require `Procore-Company-Id` header.

**Rate limit:** 3,600 requests/hour (can request increase to 14,400).

**n8n connection:** No native Procore node. Use HTTP Request node with Generic OAuth2 credentials. Fully functional — we've confirmed the exact configuration parameters.

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

**Bottom line:** n8n can talk to every system in the stack. Procore requires HTTP Request (no native node), but everything else has native nodes.

---

## 3. Three Architecture Options

### Option A: Hybrid — SmoothX + n8n (Recommended)

```
SmoothX owns:                       n8n owns:
├── Procore → QBO bill sync         ├── 5-layer dedup engine
├── QBO → Procore payment sync      ├── AI-powered invoice matching
├── Contact/vendor sync             ├── Notion "2nd Brain" logging
├── Cost code mapping               ├── PM notification emails
├── Retention/retainage             ├── Customer invoice generation
├── Progress claims                 ├── Follow-up & escalation
└── API change absorption           ├── Custom approval workflows
                                    └── Procore webhook event routing
```

**How it works:** SmoothX continues handling the Procore↔QBO financial sync (what it already does today). n8n sits alongside it, reading from both Procore and QBO to run dedup checks, AI matching, PM notifications, and customer invoicing. **n8n never writes vendor bills to QBO** — SmoothX owns that. n8n only writes customer invoices to QBO (which SmoothX doesn't handle).

| Pros | Cons |
|------|------|
| Fastest to production (4-6 weeks for n8n workflows) | Monthly SmoothX subscription continues |
| No risk of breaking existing Procore↔QBO sync | Two systems to monitor instead of one |
| Retention, progress claims, cost codes handled by battle-tested software | Dependency on SmoothX for core financial sync |
| n8n effort focused entirely on the novel problem (dedup + automation) | |
| API changes absorbed by SmoothX | |

**Cost estimate:**
- SmoothX: ~$200-500/month (current cost, already in budget)
- n8n Cloud: ~$50/month (Starter) or self-hosted (free)
- Claude API (AI matching): ~$20-50/month at expected volumes
- Development time: **4-6 weeks** to production

---

### Option B: n8n Only — Full DIY

```
n8n owns everything:
├── Procore → QBO bill sync          (MUST BUILD)
├── QBO → Procore payment sync       (MUST BUILD)
├── Contact/vendor sync              (MUST BUILD)
├── Cost code mapping                (MUST BUILD)
├── Retention/retainage handling     (MUST BUILD — COMPLEX)
├── Progress claims                  (MUST BUILD — COMPLEX)
├── 5-layer dedup engine
├── AI-powered invoice matching
├── Notion "2nd Brain" logging
├── PM notification emails
├── Customer invoice generation
├── Follow-up & escalation
├── Custom approval workflows
└── Procore webhook event routing
```

**How it works:** Drop SmoothX entirely. n8n handles all Procore↔QBO sync plus the dedup/automation layer. You build and maintain the financial sync workflows yourself using the Procore HTTP Request node and QBO native node.

| Pros | Cons |
|------|------|
| No SmoothX subscription | 10-14 weeks to production |
| Full control over every data flow | You build and maintain retention/retainage logic |
| No vendor dependency | You build and maintain progress claim mapping |
| Single orchestration platform | You absorb Procore + QBO API changes forever |
| | Higher risk of financial posting errors during first 3-6 months |
| | 2-4 hours/week ongoing maintenance minimum |
| | Cost code → Chart of Accounts mapping is non-trivial |

**Cost estimate:**
- SmoothX: $0 (eliminated)
- n8n Cloud: ~$50/month or self-hosted (free)
- Claude API: ~$20-50/month
- Development time: **10-14 weeks** to production
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

### Option C: n8n + Procore's Free Built-In QBO Connector

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
| | No retention/retainage handling |
| | No progress claim sync |
| | No attachment sync |
| | Less comprehensive than SmoothX |

**Cost estimate:**
- Connector: $0
- n8n Cloud: ~$50/month or self-hosted
- Procore Professional Services (for existing projects): Unknown — contact Procore
- Development time: **6-8 weeks** to production
- Gap-filling for missing features: **Additional 2-4 weeks** if retention/progress claims needed

---

## 4. Option Comparison Matrix

| Factor | Option A: Hybrid (SmoothX + n8n) | Option B: n8n Only | Option C: Procore Connector + n8n |
|--------|----------------------------------|-------------------|-----------------------------------|
| **Time to production** | 4-6 weeks | 10-14 weeks | 6-8 weeks |
| **Monthly cost** | ~$300-600 | ~$70-100 | ~$50-100 |
| **Dev effort (build)** | Low (dedup/automation only) | Very High (everything) | Medium (dedup + gap-filling) |
| **Dev effort (maintain)** | Low | High (2-4 hrs/week) | Medium |
| **Retention/retainage** | Handled by SmoothX | Must build (complex) | Not supported |
| **Progress claims** | Handled by SmoothX | Must build (complex) | Not supported |
| **Existing projects** | Works today | Works today | Requires paid Professional Services |
| **Canada operations** | Supported | Supported | US-only payment import |
| **API change risk** | Absorbed by SmoothX | You own it | Absorbed by Procore |
| **Financial error risk** | Low (battle-tested sync) | Medium-High (first 6 months) | Medium (feature gaps) |
| **Vendor lock-in** | SmoothX dependency | None | Procore dependency |

---

## 5. Recommended Path: Option A (Hybrid)

### Why

1. **The duplicate payment problem is the priority.** Every week without the dedup engine is another week of potential $150K+ in overpayments. Option A gets the dedup engine live in 4-6 weeks. Option B delays it by 6-8 weeks because development time is consumed building plumbing that SmoothX already handles.

2. **Retention and progress claims are construction fundamentals.** Frontline operates in commercial construction. Progress billing and retainage are core to the business. Building these from scratch in n8n is a significant engineering effort with high risk of financial errors.

3. **The savings from dropping SmoothX (~$300/month) do not justify the risk.** The annual SmoothX cost (~$3,600-6,000) is a rounding error compared to the $720,000+ annual duplicate payment exposure. Getting the dedup engine live 2 months sooner pays for years of SmoothX.

4. **You can always migrate later.** Start with Option A. Once the dedup engine is running and proven, revisit whether to bring the Procore↔QBO sync in-house. By then you'll have real data on how complex your financial sync actually is.

---

## 6. Implementation Roadmap (Option A: Hybrid)

### Phase 1: Foundation — Weeks 1-2

| Task | Details | Owner |
|------|---------|-------|
| Set up Notion workspace | Create 5 databases: Invoice Tracker, Projects, Vendors, Customers, Audit Log | Dev |
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
| Implement fingerprint generator | n8n Code node: SHA256(normalized_vendor + normalized_invoice_num + amount) | Dev |
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
[Generate Fingerprint] ──► SHA256(normalized_vendor | normalized_invoice_num | amount)
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

Five interconnected databases serve as the single source of truth:

| Database | Key Fields | Purpose |
|----------|-----------|---------|
| **Invoice Tracker** | Invoice #, Fingerprint, Vendor, Amount, Date, Project, Procore Commitment ID, QBO Bill ID, Status, Match Confidence, PM, Follow-Up Count | Master record for every invoice |
| **Projects** | Project Name, Procore ID, Customer, PM, Contract Value, Billed to Date, Remaining | Project-level financial view |
| **Vendors** | Name, Normalized Name, Procore ID, QBO ID, Contact Email, YTD Payments | Cross-system vendor mapping |
| **Customers** | Name, QBO ID, Email, Payment Terms, Outstanding Balance | AR tracking |
| **Audit Log** | Timestamp, Action, Invoice, Performed By, System | Complete history of every system action |

**Pre-built views:**
- PM Dashboard — each PM's pending approvals + overdue items
- Duplicate Alert Board — all flagged duplicates needing review
- Cash Flow Board — outstanding receivables by customer
- Project Financial Summary — per-project billing status
- Vendor Payment History — all payments to each vendor (dedup verification)

---

## 9. Key Decisions Needed from the Team

| # | Decision | Options | Impact |
|---|----------|---------|--------|
| 1 | **Which architecture?** | A (Hybrid — recommended), B (n8n only), or C (Procore connector + n8n) | Timeline, cost, and risk profile |
| 2 | **n8n hosting?** | Cloud (~$50/mo, managed) vs. self-hosted (free, you manage server) | IT overhead |
| 3 | **Pilot project selection** | Which active project to test on first? | Ideally medium complexity with cooperative subs |
| 4 | **PM approval method** | Email links, Notion buttons, or Slack integration? | UX for PMs |
| 5 | **Customer invoice format** | QBO native email, custom PDF template, or both? | Customer-facing presentation |
| 6 | **Follow-up timing** | Net 30/45/60 default? Per-customer? | Collections aggressiveness |
| 7 | **AI model for matching** | Claude (recommended for accuracy) vs. GPT-4 (alternative) | Cost, accuracy |
| 8 | **Who maintains n8n long-term?** | In-house developer, contractor, or managed service? | Sustainability |

---

## 10. Expected Outcomes

| Metric | Current State | After Implementation |
|--------|--------------|---------------------|
| Duplicate payment rate | ~2% (est. $150K/month exposure) | <0.1% (automated blocking) |
| Invoice processing time | 2-5 days (manual matching) | <4 hours (automated) |
| PM time on invoice admin | 5-10 hours/week | <1 hour/week (review alerts only) |
| Customer invoice turnaround | 3-7 days after approval | Same day (automated) |
| Follow-up on overdue invoices | Ad hoc / inconsistent | Automated 3-tier escalation |
| Financial visibility | Fragmented across 3+ systems | Single Notion dashboard |
| Audit trail | Partial / manual | Complete automated log |

---

## 11. Next Steps

1. **Team reviews this report** and selects architecture option (A, B, or C)
2. **Decide on the 8 open questions** in Section 9
3. **Register a Procore Developer App** (needed for API access regardless of option chosen)
4. **Select a pilot project** and identify the PM who will be the first user
5. **Begin Phase 1** — Notion workspace setup + n8n instance + OAuth connections

---

*This report is based on verified API documentation from Procore Developer Portal, Intuit Developer Portal, Notion API docs, n8n documentation, and the Procore App Marketplace. All endpoints and capabilities have been confirmed as available.*
