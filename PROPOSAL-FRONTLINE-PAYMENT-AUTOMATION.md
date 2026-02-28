# Professional Services Proposal
## Payment Automation & Duplicate Prevention System

---

**Prepared by:** Veteran Vectors
**Prepared for:** Frontline Holdings LLC
**Date:** February 28, 2026
**Proposal Valid Through:** March 31, 2026
**Version:** 2.0

---

## Dear Frontline Holdings Team,

Thank you for the opportunity to present this proposal. We understand the urgency of your duplicate payment challenge and the financial impact it has on your operations. This proposal outlines three engagement options — each building on the last — so Frontline Holdings can choose the level of automation, visibility, and independence that best fits your goals and budget.

We've already completed extensive research into your technology stack, verified every API endpoint we'll be building against, and designed the full system architecture. We're ready to start at whichever level you choose.

---

## 1. Understanding Your Challenge

Frontline Holdings operates as a veteran-owned, vertically integrated construction management and general contracting firm across multiple divisions — Frontline Builders, Frontline Construction Management, Government & Defense Contracting, and Land Development. You manage multiple active projects simultaneously across Texas, Florida, Oklahoma, Canada, and international markets, working with dozens of subcontractors per project and processing hundreds of invoices monthly.

**The core problem:** Invoices arrive through multiple channels — email, Procore uploads, and physical mail — and are entered into QuickBooks Online by different people at different times with no cross-system deduplication. The result is duplicate and triple payments that erode margins in an industry already operating at 3-8%.

**Estimated financial exposure:** Based on industry benchmarks (1-2% duplicate rate on ~500 invoices/month at ~$15,000 average), the annual exposure is approximately **$720,000+**. We recommend validating this figure with a targeted AP audit early in the engagement.

**What's missing today:**
- No automated deduplication between Procore and QBO
- No real-time matching of invoices to Procore commitments
- No single source of truth for invoice status across systems
- No automated PM notifications or approval workflows
- No systematic follow-up on unpaid customer invoices
- No audit trail connecting invoice receipt to final payment

---

## 2. Three Options

This proposal presents three engagement options. Each is self-contained and delivers value on its own. Options 2 and 3 build on the previous option's foundation.

### At a Glance

| | Option 1 | Option 2 | Option 3 |
|---|----------|----------|----------|
| **Name** | Gap-Filler | Gap-Filler + 2nd Brain | Full Replacement |
| **One-liner** | n8n fills the gaps SmoothX doesn't cover | Option 1 + Notion as a 2nd source of truth | Complete n8n buildout — SmoothX removed |
| **SmoothX** | Keeps SmoothX | Keeps SmoothX | Replaces SmoothX |
| **Notion** | Not included | Central hub for visibility & approvals | Central hub for visibility & approvals |
| **Dedup engine** | 5-layer, n8n-native | 5-layer, Notion-backed | 5-layer, Notion-backed |
| **PM approvals** | Email-based | Notion dashboard + email | Notion dashboard + email |
| **Dashboards** | Procore + QBO native reporting | Real-time Notion dashboards | Real-time Notion dashboards |
| **Audit trail** | n8n execution logs + QBO records | Full Notion audit database | Full Notion audit database |
| **Customer invoicing** | Automated via QBO | Automated via QBO | Automated via QBO |
| **Monthly run-rate** | ~$70-150/mo | ~$320-650/mo | ~$120-250/mo |
| **Fixed project fee** | **$14,000** | **$22,000** | **$42,000** |
| **Timeline** | 8 weeks | 12 weeks | 20 weeks |
| **Risk level** | Low | Low | Medium-High (first 6 months) |

---

## 3. Option 1 — Gap-Filler ($14,000)

### What It Is

n8n workflow automation fills the specific gaps that SmoothX doesn't cover: duplicate detection, AI-powered invoice matching, PM notifications, customer invoice generation, and automated follow-up. SmoothX continues handling what it does well. No new platforms are introduced — everything runs through n8n, Procore, and QBO.

### Architecture

```
SmoothX continues handling:             n8n (new) handles:
├── Procore → QBO bill sync             ├── 5-layer dedup engine
├── QBO → Procore payment sync          ├── AI-powered invoice matching
├── Contact/vendor sync                 ├── PM email notifications
├── Cost code mapping                   ├── Customer invoice generation (QBO)
├── Retention/retainage                 ├── Automated 3-tier follow-up
├── Progress claims                     └── Procore webhook routing
└── API change absorption

Dedup data stored in: n8n internal datastore + QBO custom fields
Audit trail: n8n execution logs + QBO transaction history
Dashboards: Procore native + QBO native reporting
```

**Key rule:** n8n never writes vendor bills to QBO — SmoothX owns that sync. n8n reads from QBO to verify and cross-reference, and writes only customer invoices.

### What Your Team Gets

- **PMs:** Email alerts when invoices arrive with automatic Procore commitment match results. Approve/reject via email reply or link. Automated follow-up on overdue customer invoices.
- **Finance:** Every invoice fingerprinted and checked against 5 dedup layers. Duplicates blocked automatically. Customer invoices generated in QBO on approval.
- **Leadership:** Automated collections process. Dedup protection operational. Reporting via existing Procore and QBO tools.

### Scope of Work

**Phase 1: Foundation (Weeks 1-2)**

| Deliverable | Details |
|-------------|---------|
| n8n Cloud instance setup | Deployment with OAuth credential configuration |
| Procore Developer App | Register app, configure DMSA (Client Credentials), verify API connectivity |
| QBO integration | Configure native n8n QuickBooks node, verify read/write access |
| Vendor data normalization | Pull vendors from Procore and QBO, build normalized lookup table in n8n |
| Dedup datastore setup | Configure n8n internal storage for invoice fingerprints and match history |
| API connectivity verification | End-to-end test of Procore, QBO, and Claude API connections |

**Phase 2: Dedup Engine + Matching (Weeks 3-5)**

| Deliverable | Details |
|-------------|---------|
| Workflow 1: Invoice Intake & Dedup | Procore webhook trigger for real-time invoice capture |
| Fingerprint generator | SHA256 hash of normalized vendor + invoice number + amount + project |
| Layer 1: Exact match | Fingerprint lookup — catches identical invoices |
| Layer 2: Fuzzy match | Vendor name similarity >85% + amount within 1% + date within 30 days |
| Layer 3: Pattern match | Same vendor + same amount within 45-day window |
| Layer 4: Procore cross-reference | Total payments vs. committed amount — catches overpayments |
| Layer 5: AI analysis | Claude API intelligent pattern detection for sophisticated duplicates |
| Workflow 2: Procore-QBO Matching | AI-scored commitment matching (HIGH/MEDIUM/LOW confidence) |
| PM email notifications | Duplicate alerts, match results, approve/reject via email |
| Threshold tuning | Test against historical data, adjust confidence levels |

**Phase 3: Customer Invoicing, Follow-Up & Rollout (Weeks 6-8)**

| Deliverable | Details |
|-------------|---------|
| Workflow 3: Customer Invoicing | On PM approval, create QBO customer invoice and email to customer |
| Automated follow-up | Daily check for overdue invoices with 3-tier escalation (gentle → firm → escalation) |
| QBO payment verification | Check payment status before sending reminders |
| Pilot deployment | Live on one selected project with parallel run |
| PM training | 30-minute walkthrough per PM |
| Finance training | QBO workflow review, edge case handling |
| Full rollout | Expand from pilot to all active projects |
| Operations runbook | Edge case handling, manual overrides, error procedures |

### Deliverables

| # | Deliverable | Format |
|---|------------|--------|
| 1 | n8n Workflow 1: Invoice Intake & 5-Layer Dedup Engine | Deployed n8n workflow |
| 2 | n8n Workflow 2: Procore-QBO AI Matching Engine | Deployed n8n workflow |
| 3 | n8n Workflow 3: Customer Invoice & Automated Follow-Up | Deployed n8n workflow |
| 4 | Procore webhook integration | Configured in Procore + n8n |
| 5 | QBO integration (customer invoices, payment verification) | Configured in n8n |
| 6 | PM notification email templates (4 templates) | HTML email templates in n8n |
| 7 | Customer email templates (invoice + 3 follow-up tiers) | HTML email templates in n8n |
| 8 | PM + Finance team training (recorded) | Live sessions + recording |
| 9 | Operations runbook | Markdown document |

### Investment

| Phase | Fee |
|-------|-----|
| Phase 1: Foundation | $3,500 |
| Phase 2: Dedup Engine + Matching | $6,500 |
| Phase 3: Customer Invoicing, Follow-Up & Rollout | $4,000 |
| **Total** | **$14,000** |

**Payment schedule:** 3 milestone payments of $4,667 (kickoff, Phase 2 complete, full rollout)

**Monthly run-rate after go-live:** ~$70-150/month (n8n Cloud ~$50-100 + Claude API ~$20-50). SmoothX subscription continues as-is (existing cost).

### What Option 1 Does NOT Include

- Notion workspace or dashboards (use Procore + QBO native reporting)
- Centralized audit database (audit trail lives in n8n execution logs + QBO)
- Real-time financial dashboards (rely on Procore and QBO)
- PM approval via dashboard (approval is email-based only)

---

## 4. Option 2 — Gap-Filler + 2nd Brain ($22,000)

### What It Is

Everything in Option 1, plus a full **Notion "2nd Brain"** that acts as a second source of truth alongside Procore and QBO. Notion becomes the centralized hub where every invoice is tracked, PMs approve from a real-time dashboard, duplicates are surfaced visually, and leadership gets financial dashboards that pull data from both Procore and QBO into one place.

### Architecture

```
SmoothX continues handling:             n8n + Notion (new) handles:
├── Procore → QBO bill sync             ├── 5-layer dedup engine (Notion-backed)
├── QBO → Procore payment sync          ├── AI-powered invoice matching
├── Contact/vendor sync                 ├── Notion "2nd Brain" (6 databases)
├── Cost code mapping                   ├── PM dashboard & approval workflows
├── Retention/retainage                 ├── Customer invoice generation (QBO)
├── Progress claims                     ├── Automated 3-tier follow-up
└── API change absorption              ├── Real-time financial dashboards
                                        ├── Complete audit trail database
                                        └── Procore webhook routing

Dedup data stored in: Notion Invoice Tracker database (queryable, searchable)
Audit trail: Dedicated Notion Audit Log database (every action logged)
Dashboards: 6 Notion views (PM, Duplicates, Cash Flow, Project, Vendor, Audit)
```

### What Option 2 Adds Over Option 1

| Capability | Option 1 | Option 2 |
|-----------|----------|----------|
| Invoice tracking | n8n internal datastore | Notion database — searchable, filterable, sortable |
| PM approvals | Email only | Notion dashboard + email (one-click from dashboard) |
| Duplicate visibility | Email alerts | Email alerts + visual Duplicate Alert Board in Notion |
| Financial dashboards | Procore + QBO native | 6 real-time Notion dashboards pulling from all systems |
| Audit trail | n8n execution logs (developer-facing) | Dedicated Notion database (business-user-facing, 7-year retention capable) |
| Vendor history | QBO vendor records | Notion Vendor database with cross-system payment history |
| Project financials | Procore budget views | Notion Project Financial Summary combining Procore + QBO data |
| Status visibility | Check n8n or ask Finance | Anyone on the team can see real-time status in Notion |

### Additional Scope (Beyond Option 1)

**Phase 1 additions:**

| Deliverable | Details |
|-------------|---------|
| Notion workspace setup | 6 interconnected databases: Invoice Tracker, Projects, Vendors, Customers, Audit Log, Payments |
| Notion dashboard views | PM Dashboard, Duplicate Alert Board, Cash Flow Board, Project Financial Summary, Vendor Payment History, Audit Trail |
| Notion integration | Configure native n8n Notion node, verify database operations |
| Vendor data import | Populate Notion Vendors database with normalized Procore + QBO vendor data |

**Phase 2-3 enhancements:**

| Enhancement | Details |
|-------------|---------|
| Notion-backed dedup | Fingerprints and match history stored in Notion (queryable by business users) |
| Notion audit logging | Every invoice action creates a full audit record with match details, confidence score, timestamps |
| PM approval via Notion | One-click approve/reject from Notion dashboard (in addition to email) |
| Status automation | Automatic Notion status progression: Received → Matched → Approved → Invoiced → Follow-Up → Paid |
| Dashboard data sync | n8n workflows feed real-time data to all 6 Notion dashboard views |

**Phase 4 (new): Notion Buildout & Extended Rollout (Weeks 9-12)**

| Deliverable | Details |
|-------------|---------|
| Dashboard refinement | Tune all 6 views based on PM and Finance feedback during pilot |
| Historical data backfill | Import recent invoice history into Notion for immediate dashboard value |
| Extended parallel run | 2-4 weeks of old + new processes side-by-side |
| Advanced training | Notion dashboard walkthrough for PMs, Finance, and Leadership |
| 90-day review criteria | Defined success metrics for post-rollout evaluation |

### Deliverables (in addition to Option 1)

| # | Deliverable | Format |
|---|------------|--------|
| 10 | Notion workspace with 6 databases + 6 dashboard views | Live Notion workspace |
| 11 | Notion-backed dedup and audit trail | Integrated with n8n workflows |
| 12 | System architecture documentation | Markdown document (already drafted) |
| 13 | 90-day review report | Assessment of performance vs. targets |

### Investment

| Phase | Fee |
|-------|-----|
| Phase 1: Foundation + Notion Setup | $5,000 |
| Phase 2: Dedup Engine + Matching (Notion-backed) | $7,000 |
| Phase 3: Customer Invoicing & Follow-Up | $5,000 |
| Phase 4: Notion Buildout, Training & Rollout | $5,000 |
| **Total** | **$22,000** |

**Payment schedule:** 4 milestone payments of $5,500 (kickoff, Phase 2 complete, Phase 3 complete, full rollout)

**Monthly run-rate after go-live:** ~$320-650/month (n8n Cloud ~$50-100 + Notion Team ~$50/mo + Claude API ~$20-50 + SmoothX subscription continues as-is)

---

## 5. Option 3 — Full Replacement ($42,000)

### What It Is

Everything in Option 2, plus n8n **replaces SmoothX entirely**. Veteran Vectors builds the complete Procore-to-QBO financial sync — vendor bill creation, payment status sync, contact/vendor sync, cost code mapping, retention/retainage handling, and progress claim workflows — all in n8n. SmoothX subscription is cancelled. Frontline owns the entire integration stack.

### Architecture

```
n8n handles everything:
├── Procore → QBO vendor bill creation (replaces SmoothX)
├── QBO → Procore payment status sync (replaces SmoothX)
├── Contact/vendor sync (replaces SmoothX)
├── Cost code mapping (replaces SmoothX)
├── Retention/retainage calculations (replaces SmoothX)
├── Progress claim workflows (replaces SmoothX)
├── 5-layer dedup engine (Notion-backed)
├── AI-powered invoice matching
├── Notion "2nd Brain" (6+ databases)
├── PM dashboard & approval workflows
├── Customer invoice generation (QBO)
├── Automated 3-tier follow-up
├── Real-time financial dashboards
├── Complete audit trail database
└── Procore webhook routing

No SmoothX dependency. Full control. Full responsibility.
```

### What Option 3 Adds Over Option 2

| Capability | Option 2 | Option 3 |
|-----------|----------|----------|
| Procore → QBO bill sync | SmoothX | n8n (you own it) |
| QBO → Procore payment sync | SmoothX | n8n (you own it) |
| Contact/vendor sync | SmoothX | n8n (you own it) |
| Cost code mapping | SmoothX | n8n (you own it) |
| Retention/retainage | SmoothX (battle-tested) | n8n (custom-built — requires careful validation) |
| Progress claims | SmoothX (battle-tested) | n8n (custom-built — requires careful validation) |
| API change management | SmoothX absorbs changes | Frontline/Veteran Vectors owns updates |
| SmoothX subscription | Continues (~$200-500/mo) | Cancelled (saves ~$2,400-6,000/year) |
| Vendor lock-in | Dependent on SmoothX | No third-party dependency |
| Maintenance burden | Low (SmoothX manages sync) | Medium-High (~2-4 hours/week ongoing) |

### Additional Scope (Beyond Option 2)

**Phase 5 (new): SmoothX Replacement — Bill Sync (Weeks 13-16)**

| Deliverable | Details |
|-------------|---------|
| Procore commitment → QBO bill workflow | Create vendor bills in QBO from Procore commitments and pay applications |
| Line item mapping | Map Procore line items to QBO bill lines with cost code translation |
| Vendor matching | Cross-reference Procore vendors to QBO vendors, handle mismatches |
| Cost code sync | Build and maintain cost code mapping between Procore and QBO |
| Duplicate bill prevention | Ensure n8n bill creation doesn't conflict with historical SmoothX bills |
| Parallel run with SmoothX | Run both systems side-by-side to verify n8n produces identical QBO results |

**Phase 6 (new): SmoothX Replacement — Payments, Retention & Progress (Weeks 17-20)**

| Deliverable | Details |
|-------------|---------|
| QBO → Procore payment sync | When bills are paid in QBO, update payment status in Procore |
| Retention/retainage handling | Calculate and track retainage per commitment, release on schedule |
| Progress claim workflows | Support AIA-style progress billing with percentage-of-completion calculations |
| Contact/vendor sync | Bidirectional vendor record sync between Procore and QBO |
| SmoothX cutover | Controlled migration — disable SmoothX sync after verification period |
| Extended validation | 4-week parallel run comparing n8n output vs. SmoothX output transaction-by-transaction |
| Regression testing | Full test suite covering edge cases: partial payments, retainage releases, change order adjustments, multi-project vendors |

### Investment

| Phase | Fee |
|-------|-----|
| Phases 1-4: Everything in Option 2 | $22,000 |
| Phase 5: SmoothX Replacement — Bill Sync | $10,000 |
| Phase 6: SmoothX Replacement — Payments, Retention & Progress | $10,000 |
| **Total** | **$42,000** |

**Payment schedule:** 6 milestone payments of $7,000 (kickoff, Phase 2, Phase 3, Phase 4, Phase 5, Phase 6)

**Monthly run-rate after go-live:** ~$120-250/month (n8n Cloud ~$50-100 + Notion Team ~$50/mo + Claude API ~$20-50). SmoothX subscription eliminated.

### Important Considerations for Option 3

| Factor | Detail |
|--------|--------|
| **Development risk** | Retention and progress claim logic is complex. SmoothX has refined this over years. Custom-building it carries medium-high financial error risk during the first 6 months. |
| **Maintenance burden** | Without SmoothX absorbing Procore/QBO API changes, Frontline (or Veteran Vectors via retainer) must monitor and respond to API updates. Estimated 2-4 hours/week ongoing. |
| **Validation period** | A minimum 4-week parallel run (SmoothX + n8n producing identical results) is required before SmoothX cutover. This is non-negotiable for financial safety. |
| **Rollback plan** | SmoothX subscription should not be cancelled until the parallel run confirms zero discrepancies for a full billing cycle. |
| **Long-term savings** | Eliminating SmoothX saves ~$2,400-6,000/year. Break-even on the additional $20,000 investment is 3-8 years depending on SmoothX pricing. The real value is independence, not cost savings. |
| **Retainer strongly recommended** | Option 3 without a maintenance retainer is not recommended. API changes, edge cases, and retention logic require ongoing attention. |

---

## 6. Option Comparison

### Feature Comparison

| Feature | Option 1 | Option 2 | Option 3 |
|---------|----------|----------|----------|
| 5-layer dedup engine | Yes | Yes | Yes |
| AI-powered invoice matching | Yes | Yes | Yes |
| PM email notifications | Yes | Yes | Yes |
| Customer invoice automation | Yes | Yes | Yes |
| 3-tier follow-up escalation | Yes | Yes | Yes |
| Procore webhook integration | Yes | Yes | Yes |
| Notion 2nd Brain (6 databases) | -- | Yes | Yes |
| PM approval dashboard | -- | Yes | Yes |
| Real-time financial dashboards | -- | Yes | Yes |
| Business-user audit trail | -- | Yes | Yes |
| Procore → QBO bill sync | SmoothX | SmoothX | n8n (custom) |
| Retention/retainage | SmoothX | SmoothX | n8n (custom) |
| Progress claims | SmoothX | SmoothX | n8n (custom) |
| Vendor/contact sync | SmoothX | SmoothX | n8n (custom) |
| SmoothX dependency | Yes | Yes | No |

### Cost Comparison

| Cost Item | Option 1 | Option 2 | Option 3 |
|-----------|----------|----------|----------|
| **Project fee (one-time)** | **$14,000** | **$22,000** | **$42,000** |
| n8n Cloud (monthly) | $50-100 | $50-100 | $50-100 |
| Notion Team (monthly) | -- | ~$50 | ~$50 |
| Claude API (monthly) | $20-50 | $20-50 | $20-50 |
| SmoothX (monthly) | $200-500 | $200-500 | Eliminated |
| **Monthly run-rate** | **$270-650** | **$320-700** | **$120-200** |
| **Year 1 total** (project + 12 months run-rate) | **$17,240 - $21,800** | **$25,840 - $30,400** | **$43,440 - $44,400** |
| **Year 2+ annual** (run-rate only) | **$3,240 - $7,800** | **$3,840 - $8,400** | **$1,440 - $2,400** |

### Risk Comparison

| Risk Factor | Option 1 | Option 2 | Option 3 |
|------------|----------|----------|----------|
| Development risk | Low | Low | Medium-High |
| Financial error risk | Low | Low | Medium-High (first 6 months) |
| API change risk | Low (SmoothX absorbs) | Low (SmoothX absorbs) | Medium (you own it) |
| Maintenance burden | Low (~1 hr/week) | Low (~1 hr/week) | Medium-High (~2-4 hrs/week) |
| Vendor lock-in | SmoothX dependency | SmoothX + Notion | Notion only |
| Rollback complexity | Simple | Simple | Complex (must re-enable SmoothX) |

### Recommendation

**Veteran Vectors recommends Option 2** for most organizations at Frontline's scale and complexity.

- **Option 1** is the right choice if the priority is solving the duplicate payment problem as quickly and cheaply as possible, and your team is comfortable with Procore + QBO native reporting.
- **Option 2** is the right choice if you want the duplicate payment solution plus centralized visibility, PM-friendly dashboards, and a business-user audit trail — without taking on the risk of replacing SmoothX.
- **Option 3** is the right choice if eliminating the SmoothX dependency is a strategic priority and you're prepared to invest in ongoing maintenance. We recommend this only with a Standard or Premium retainer.

---

## 7. Monthly Retainer (All Options)

After project completion, Veteran Vectors provides ongoing support to keep the system running and evolving:

| Retainer Tier | Hours/Month | Monthly Fee | Includes |
|--------------|-------------|-------------|----------|
| **Essential** | Up to 5 hours | $625/month | Bug fixes, threshold adjustments, API change updates, email support (24-hour response) |
| **Standard** | Up to 10 hours | $1,250/month | Everything in Essential + feature enhancements, new workflow development, monthly performance review call, priority support (4-hour response) |
| **Premium** | Up to 20 hours | $2,500/month | Everything in Standard + proactive monitoring, quarterly system audits, on-call support (1-hour response), dedicated Slack channel |

**Retainer notes:**
- Unused hours do not roll over
- Hours beyond the monthly allocation are billed at $125/hour
- Either party may adjust or cancel the retainer with 30 days' written notice
- Retainer begins the month following final phase completion
- 2 weeks of post-launch support included in the project fee at no additional cost
- **Option 3: Standard or Premium retainer strongly recommended** due to ongoing API maintenance and retention logic complexity

---

## 8. Timeline by Option

### Option 1: 8 Weeks

```
Week 1-2    ████████░░░░░░░░  Phase 1: Foundation
Week 3-5    ░░░░░░░░████████  Phase 2: Dedup + Matching
Week 6-8    ████████████████  Phase 3: Invoicing, Follow-Up & Rollout
```

### Option 2: 12 Weeks

```
Week 1-2    ████████░░░░░░░░░░░░░░░░  Phase 1: Foundation + Notion
Week 3-4    ░░░░░░░░████████░░░░░░░░  Phase 2: Dedup + Matching (Notion-backed)
Week 5-6    ░░░░░░░░░░░░░░░░████████  Phase 3: Invoicing & Follow-Up
Week 7-8    ████████░░░░░░░░░░░░░░░░  Phase 3 cont. + Phase 4 start
Week 9-12   ░░░░░░░░████████████████  Phase 4: Notion Buildout & Rollout
```

### Option 3: 20 Weeks

```
Week 1-12   ████████████████████████  Phases 1-4: Full Option 2 buildout
Week 13-16  ████████████████░░░░░░░░  Phase 5: SmoothX Replacement — Bills
Week 17-20  ░░░░░░░░████████████████  Phase 6: Payments, Retention & Cutover
```

**Key dependencies from Frontline Holdings (all options):**
- Executive sponsor identified (before kickoff)
- Procore Developer App registered by Frontline admin (Week 1)
- Pilot project selected + pilot PM identified (by mid-project)
- Historical duplicate invoice data for testing (by Week 3)
- Finance team available for parallel run verification
- PMs available for 30-minute training sessions

---

## 9. Expected Results

| Metric | Current State | 90-Day Target | 6-Month Target | Options |
|--------|--------------|---------------|----------------|---------|
| Duplicate payment rate | ~2% estimated | Reduce by 80% | Reduce by 90%+ | All |
| Invoice processing time | 2-5 days | <8 hours | <4 hours | All |
| PM time on invoice admin | 5-10 hours/week | 3-5 hours/week | <1 hour/week | All |
| Customer invoice turnaround | 3-7 days | Same day (high-confidence) | Same day (all) | All |
| Follow-up on overdue invoices | Ad hoc | Automated 3-tier escalation | Automated, <2% missed | All |
| Financial visibility | Fragmented | Improved (QBO + Procore) | Stable | 1 |
| Financial visibility | Fragmented | Single Notion dashboard | Fully operational | 2, 3 |
| Audit trail | Partial / manual | n8n execution logs | Functional | 1 |
| Audit trail | Partial / manual | Complete Notion audit database | 100% coverage | 2, 3 |
| SmoothX dependency | Full | Full | Full | 1, 2 |
| SmoothX dependency | Full | Parallel run | Eliminated | 3 |

**Conservative ROI estimate (all options):** At a 1% reduction in duplicate payments (half the estimated 2% rate), the system saves approximately **$7,500/month** in prevented overpayments. Even Option 1 at $14,000 pays for itself in **under 2 months**.

---

## 10. Why Veteran Vectors

**We've already done the homework.** Before presenting this proposal, we:

- Verified every Procore API endpoint against official developer documentation
- Confirmed OAuth 2.0 configuration (DMSA Client Credentials grant) for server-to-server automation
- Validated n8n's connection method (HTTP Request node + Generic OAuth2 — no native Procore node exists)
- Mapped SmoothX's 5 marketplace apps and identified the exact boundary between what SmoothX handles and what n8n should handle
- Evaluated all 3 Procore-QBO connector options (Procore-built, SmoothX/0link, Interfy)
- Designed the complete 5-layer dedup engine with fingerprint hashing, fuzzy matching, and AI analysis
- Built importable n8n workflow templates for all 3 core workflows
- Designed the full 6-database Notion schema with dashboard views
- Produced a comprehensive architecture document, integration roadmap, and risk register

**We understand construction.** Invoice matching in construction isn't like e-commerce. You deal with progress billing, retainage, change orders, cost codes, and AIA-style pay applications. Same vendor, same amount, different project — that's not a duplicate. Same invoice number reformatted — that is. Our 5-layer dedup engine is designed for these construction-specific edge cases.

**We build for handoff, not dependency.** Everything we build uses open-source tools (n8n), standard APIs, and clear documentation. If Frontline Holdings ever wants to bring maintenance in-house, the operations runbook and architecture documentation make that transition straightforward.

---

## 11. Engagement Terms (Summary)

The following terms are provided for reference and will be formalized in the Statement of Work (SOW):

- **Scope management:** Changes to scope will be handled through written change orders with agreed pricing before work begins.
- **Payment terms:** Net 15 from invoice date, with milestone-based billing as outlined per option.
- **Intellectual property:** All deliverables become Frontline Holdings' property upon final payment. Veteran Vectors retains the right to reuse general knowledge and methodologies (but not Frontline-specific data or configurations).
- **Confidentiality:** All Frontline data, credentials, and business processes will be treated as confidential. No data will be shared with third parties, used for AI training, or retained after engagement completion.
- **Warranty:** 90-day warranty on all delivered workflows. Defects fixed at no charge. Does not cover third-party API changes or Frontline configuration changes.
- **Third-party services:** Frontline Holdings maintains its own subscriptions (Procore, QBO, Notion, n8n, SmoothX, Claude API). Veteran Vectors is not responsible for third-party outages or pricing changes.
- **Retainer:** Optional (strongly recommended for Option 3), begins after final phase. Either party may adjust or cancel with 30 days' notice. Overage billed at $125/hour.

*Full terms, conditions, and liability provisions will be detailed in the formal SOW.*

---

## 12. Next Steps

This proposal is provided for awareness and understanding. A separate Statement of Work (SOW) will be prepared for formal execution upon alignment.

**To move forward:**

1. Review this proposal and the accompanying Integration Roadmap Report
2. Select an option (1, 2, or 3) — or discuss a hybrid approach
3. Identify an executive sponsor for the initiative
4. Schedule a walkthrough call with Veteran Vectors to discuss questions and preferences
5. Upon alignment, Veteran Vectors will deliver a formal SOW for signature

**Contact:** Please reach out to Veteran Vectors to schedule a discussion or request any clarification on the contents of this proposal.

---

## Appendix A: Technology Stack by Option

| System | Option 1 | Option 2 | Option 3 |
|--------|----------|----------|----------|
| **Procore** | Source of commitments, webhooks | Source of commitments, webhooks | Source of commitments, webhooks |
| **QuickBooks Online** | Accounting + customer invoices | Accounting + customer invoices | Accounting + customer invoices |
| **SmoothX** | Keeps (Procore↔QBO sync) | Keeps (Procore↔QBO sync) | Replaced by n8n |
| **n8n** | 3 workflows (dedup, matching, invoicing) | 3+ workflows (dedup, matching, invoicing, Notion sync) | 6+ workflows (all of Option 2 + bill sync, payment sync, retention, progress) |
| **Notion** | Not used | 2nd Brain — 6 databases, dashboards, audit | 2nd Brain — 6 databases, dashboards, audit |
| **Claude API** | AI matching + dedup | AI matching + dedup | AI matching + dedup |
| **Gmail/SMTP** | PM notifications, customer emails | PM notifications, customer emails | PM notifications, customer emails |

## Appendix B: API Verification Summary

All API endpoints have been verified against official documentation:

| API | Endpoints Verified | Auth Method | Rate Limit |
|-----|--------------------|------------|------------|
| Procore REST API | 10 endpoints (projects, commitments, line items, pay apps, direct costs, prime contracts, change orders, vendors, budgets, webhooks) | OAuth 2.0 Client Credentials (DMSA) | 3,600 req/hour |
| QuickBooks Online API | 7 endpoints (invoices, bills, bill payments, vendors, customers, queries, payments) | OAuth 2.0 | 500 req/minute |
| Notion API | 5 endpoints (databases, database query, pages, page update, search) | Internal integration token | 3 req/second |

## Appendix C: What Happens If You Do Nothing

| Month | Estimated Duplicate Exposure (at 2%) | Cumulative Loss (at 40% unrecovered) |
|-------|-------------------------------------|--------------------------------------|
| Month 1 | $150,000 | $60,000 |
| Month 3 | $450,000 | $180,000 |
| Month 6 | $900,000 | $360,000 |
| Month 12 | $1,800,000 | $720,000 |

Even Option 1's $14,000 project fee is equivalent to preventing **less than one duplicate payment** at the average invoice value of $15,000.

---

*Veteran Vectors — Building systems that work as hard as you do.*
