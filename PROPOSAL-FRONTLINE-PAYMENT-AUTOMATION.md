# Professional Services Proposal
## Payment Automation & Duplicate Prevention System

---

**Prepared by:** Veteran Vectors (Anthony Pinto, Founder)
**Prepared for:** Frontline Holdings LLC
**Date:** February 28, 2026
**Proposal Valid Through:** March 31, 2026
**Version:** 3.0

---

## Dear Frontline Holdings Team,

I put this proposal together after doing deep research into your tech stack — Procore, QuickBooks Online, SmoothX, and how invoices actually move through your business. I've verified every API endpoint we'd be building against, mapped out how SmoothX works (and where it falls short), and designed the full system architecture.

I'm giving you two options — one that works alongside SmoothX and adds what's missing, and one that replaces SmoothX entirely. Both solve the duplicate payment problem. The difference is how far you want to go.

Before anything is signed, I'd like to schedule a brief **audit call** — I'll walk through your actual Procore setup, QBO configuration, and invoice flow to confirm the build is right before we start.

---

## 1. Understanding Your Challenge

Frontline Holdings operates as a veteran-owned, vertically integrated construction management and general contracting firm across multiple divisions — Frontline Builders, Frontline Construction Management, Government & Defense Contracting, and Land Development. You manage multiple active projects simultaneously across Texas, Florida, Oklahoma, Canada, and international markets, working with dozens of subcontractors per project and processing hundreds of invoices monthly.

**The core problem:** Invoices arrive through multiple channels — email, Procore uploads, and physical mail — and are entered into QuickBooks Online by different people at different times with no cross-system deduplication. The result is duplicate and triple payments that erode margins in an industry already operating at 3-8%.

**Last year, Frontline lost $150,000 to duplicate payments.** Based on that confirmed figure and industry benchmarks (1-2% duplicate rate on ~500 invoices/month at ~$15,000 average), the annual exposure is approximately **$150,000-$300,000+** as project volume grows.

**What's missing today:**
- No human approval gate — invoices go straight to QBO without sign-off
- No automated deduplication between Procore and QBO
- No real-time matching of invoices to Procore commitments
- No automated PM notifications or approval workflows
- No systematic follow-up on unpaid customer invoices
- No audit trail connecting invoice receipt to final payment

---

## 2. Two Options

This proposal presents two engagement options. Option 1 is self-contained and delivers full value on its own. Option 2 builds on Option 1's foundation and adds SmoothX replacement.

### At a Glance

| | Option 1 | Option 2 |
|---|----------|----------|
| **Name** | Approval Gate + Gap-Filler | Full Replacement |
| **One-liner** | n8n adds human approval gate + fills gaps SmoothX misses | Complete n8n buildout — SmoothX removed |
| **SmoothX** | Keeps SmoothX | Replaces SmoothX |
| **Approval dashboard** | Excel in OneDrive (existing Microsoft 365) | Excel in OneDrive |
| **Dedup engine** | 5-layer, n8n-native | 5-layer, n8n-native |
| **PM approvals** | Excel dashboard + email | Excel dashboard + email |
| **Customer invoicing** | Automated via QBO | Automated via QBO |
| **Monthly run-rate (new costs)** | ~$70-130/mo | ~$70-130/mo |
| **Fixed project fee** | **$12,000** | **$32,000** |
| **Timeline** | 10 weeks | 18 weeks |
| **Risk level** | Low | Medium-High (first 6 months) |

---

## 3. Option 1 — Approval Gate + Gap-Filler ($12,000)

### What It Is

n8n workflow automation adds a **human approval gate** to every invoice before it hits QBO, plus fills the specific gaps that SmoothX doesn't cover: duplicate detection, AI-powered invoice matching, PM notifications, customer invoice generation, and automated follow-up. The approval dashboard lives in a shared Excel file in OneDrive — no new platforms, no new logins. SmoothX continues handling what it does well.

### Architecture

```
SmoothX continues handling:             n8n (new) handles:
├── Procore → QBO bill sync             ├── Human approval gate (Excel dashboard)
├── QBO → Procore payment sync          ├── 5-layer dedup engine
├── Contact/vendor sync                 ├── AI-powered invoice matching
├── Cost code mapping                   ├── PM email notifications
├── Retention/retainage                 ├── Customer invoice generation (QBO)
├── Progress claims                     └── Automated 3-tier follow-up
└── API change absorption

Approval dashboard: Excel file in OneDrive (Microsoft Graph API)
Dedup data stored in: n8n internal datastore + QBO custom fields
Audit trail: n8n execution logs + QBO transaction history + Excel log tab
```

**Key rule:** n8n never writes vendor bills to QBO — SmoothX owns that sync. n8n reads from QBO to verify and cross-reference, and writes only customer invoices.

### How the Approval Gate Works

1. Invoice comes in (Procore webhook or email)
2. n8n runs 5-layer dedup check + AI matching
3. Match result written to shared Excel file in OneDrive
4. PM opens Excel → sees invoice details, match info, confidence score
5. PM types APPROVE or HOLD in the decision column
6. n8n polls Excel every 5 minutes — picks up the decision
7. Approved → customer invoice created in QBO + confirmation email
8. Held → flagged for finance team review
9. No action within 4 hours → auto-escalation email to PM + finance

### What Your Team Gets

- **PMs:** Open one Excel file. See pending invoices. Type APPROVE or HOLD. Done.
- **Finance:** Every invoice has a human sign-off before anything touches QBO. Full audit trail in the Excel log tab.
- **Leadership:** Collections process automated. Dedup protection live. No new software to learn.

### Edge Cases We Handle

- **Excel file locked by another user** → n8n retries with exponential backoff (common with OneDrive co-authoring)
- **QBO invoice send fails** → retry 3x, then alert PM with manual link
- **PM never responds** → 4-hour escalation (configurable)
- **Multiple invoices from same vendor same day** → each gets its own row, cross-referenced

### Scope of Work

**Phase 1: Foundation (Weeks 1-2)**

| Deliverable | Details |
|-------------|---------|
| n8n Cloud instance setup | Deployment with OAuth credential configuration |
| Procore Developer App | Register app, configure DMSA (Client Credentials), verify API connectivity |
| QBO integration | Configure native n8n QuickBooks node, verify read/write access |
| Microsoft Graph API setup | Azure app registration, OneDrive OAuth, Excel file template |
| Vendor data normalization | Pull vendors from Procore and QBO, build normalized lookup table in n8n |
| API connectivity verification | End-to-end test of Procore, QBO, Microsoft Graph, and Claude API connections |

**Phase 2: Approval Dashboard (Weeks 3-4)**

| Deliverable | Details |
|-------------|---------|
| Excel dashboard template | Shared Excel file in OneDrive with invoice tracking, approval column, status formulas |
| Microsoft Graph integration | n8n reads/writes to Excel via Graph API — no OneDrive desktop sync needed |
| PM approval workflow | n8n polls Excel every 5 min for APPROVE/HOLD decisions |
| Auto-escalation | 4-hour timeout → email PM + finance if no decision made |
| Dashboard views | Pending Approvals, Recently Processed, Duplicate Alerts, Audit Log tabs |

**Phase 3: Dedup Engine + Matching (Weeks 5-7)**

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
| PM notifications | Match results emailed to PM + written to Excel dashboard |
| Threshold tuning | Test against historical data, adjust confidence levels |

**Phase 4: Customer Invoicing & Rollout (Weeks 8-10)**

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
| 4 | Excel approval dashboard (OneDrive) | Shared Excel file with 4 tabs |
| 5 | Procore webhook integration | Configured in Procore + n8n |
| 6 | QBO integration (customer invoices, payment verification) | Configured in n8n |
| 7 | PM notification email templates (4 templates) | HTML email templates in n8n |
| 8 | Customer email templates (invoice + 3 follow-up tiers) | HTML email templates in n8n |
| 9 | PM + Finance team training (recorded) | Live sessions + recording |
| 10 | Operations runbook | Markdown document |

### Investment

| Phase | Fee |
|-------|-----|
| Phase 1: Foundation | $3,000 |
| Phase 2: Approval Dashboard | $4,500 |
| Phase 3: Dedup Engine + Matching | $3,000 |
| Phase 4: Customer Invoicing & Rollout | $1,500 |
| **Total** | **$12,000** |

**Payment schedule:** 4 milestone payments of $3,000 (kickoff, Phase 2 complete, Phase 3 complete, full rollout)

**Monthly run-rate after go-live:** ~$70-130/month (n8n Cloud ~$50-100 + Claude API ~$20-30). Microsoft 365/OneDrive is an existing cost. SmoothX subscription continues as-is (existing cost).

### What Option 1 Does NOT Include

- SmoothX replacement (uses SmoothX for Procore↔QBO sync)
- Procore → QBO bill creation (SmoothX handles this)
- Retention/retainage handling (SmoothX handles this)
- Progress claim workflows (SmoothX handles this)

---

## 4. Option 2 — Full Replacement ($32,000)

### What It Is

Everything in Option 1, plus n8n **replaces SmoothX entirely**. We build the complete Procore-to-QBO financial sync — vendor bill creation, payment status sync, contact/vendor sync, cost code mapping, retention/retainage handling, and progress claim workflows — all in n8n. SmoothX subscription is cancelled. Frontline owns the entire integration stack.

### Architecture

```
n8n handles everything:
├── Human approval gate (Excel dashboard)
├── Procore → QBO vendor bill creation (replaces SmoothX)
├── QBO → Procore payment status sync (replaces SmoothX)
├── Contact/vendor sync (replaces SmoothX)
├── Cost code mapping (replaces SmoothX)
├── Retention/retainage calculations (replaces SmoothX)
├── Progress claim workflows (replaces SmoothX)
├── 5-layer dedup engine
├── AI-powered invoice matching
├── Customer invoice generation (QBO)
├── Automated 3-tier follow-up
└── Procore webhook routing

No SmoothX dependency. Full control. Full responsibility.
Approval dashboard: Same Excel file in OneDrive
```

### What Option 2 Adds Over Option 1

| Capability | Option 1 | Option 2 |
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

### Additional Scope (Beyond Option 1)

**Phase 5: SmoothX Replacement — Bill Sync (Weeks 11-14)**

| Deliverable | Details |
|-------------|---------|
| Procore commitment → QBO bill workflow | Create vendor bills in QBO from Procore commitments and pay applications |
| Line item mapping | Map Procore line items to QBO bill lines with cost code translation |
| Vendor matching | Cross-reference Procore vendors to QBO vendors, handle mismatches |
| Cost code sync | Build and maintain cost code mapping between Procore and QBO |
| Duplicate bill prevention | Ensure n8n bill creation doesn't conflict with historical SmoothX bills |
| Parallel run with SmoothX | Run both systems side-by-side to verify n8n produces identical QBO results |

**Phase 6: SmoothX Replacement — Payments, Retention & Progress (Weeks 15-18)**

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
| Phases 1-4: Everything in Option 1 | $12,000 |
| Phase 5: SmoothX Replacement — Bill Sync | $10,000 |
| Phase 6: SmoothX Replacement — Payments, Retention & Progress | $10,000 |
| **Total** | **$32,000** |

**Payment schedule:** 6 milestone payments of ~$5,333 (kickoff, Phase 2, Phase 3, Phase 4, Phase 5, Phase 6)

**Monthly run-rate after go-live:** ~$70-130/month (n8n Cloud ~$50-100 + Claude API ~$20-30). SmoothX subscription eliminated.

### Important Considerations for Option 2

| Factor | Detail |
|--------|--------|
| **Development risk** | Retention and progress claim logic is complex. SmoothX has refined this over years. Custom-building it carries medium-high financial error risk during the first 6 months. |
| **Maintenance burden** | Without SmoothX absorbing Procore/QBO API changes, Frontline (or Veteran Vectors via retainer) must monitor and respond to API updates. Estimated 2-4 hours/week ongoing. |
| **Validation period** | A minimum 4-week parallel run (SmoothX + n8n producing identical results) is required before SmoothX cutover. This is non-negotiable for financial safety. |
| **Rollback plan** | SmoothX subscription should not be cancelled until the parallel run confirms zero discrepancies for a full billing cycle. |
| **Long-term savings** | Eliminating SmoothX saves ~$2,400-6,000/year. Break-even on the additional $20,000 investment is 3-8 years depending on SmoothX pricing. The real value is independence, not cost savings. |
| **Retainer strongly recommended** | Option 2 without a maintenance retainer is not recommended. API changes, edge cases, and retention logic require ongoing attention. |

---

## 5. Option Comparison

### Feature Comparison

| Feature | Option 1 | Option 2 |
|---------|----------|----------|
| Human approval gate (Excel) | Yes | Yes |
| 5-layer dedup engine | Yes | Yes |
| AI-powered invoice matching | Yes | Yes |
| PM email notifications | Yes | Yes |
| Customer invoice automation | Yes | Yes |
| 3-tier follow-up escalation | Yes | Yes |
| Procore webhook integration | Yes | Yes |
| Procore → QBO bill sync | SmoothX | n8n (custom) |
| Retention/retainage | SmoothX | n8n (custom) |
| Progress claims | SmoothX | n8n (custom) |
| Vendor/contact sync | SmoothX | n8n (custom) |
| SmoothX dependency | Yes | No |

### Cost Comparison

| Cost Item | Option 1 | Option 2 |
|-----------|----------|----------|
| **Project fee (one-time)** | **$12,000** | **$32,000** |
| n8n Cloud (monthly) | $50-100 | $50-100 |
| Claude API (monthly) | $20-30 | $20-30 |
| Microsoft 365/OneDrive (monthly) | Existing cost | Existing cost |
| SmoothX (monthly) | $200-500 (existing) | Eliminated |
| **New monthly run-rate** | **$70-130** | **$70-130** |
| **Total monthly (incl. SmoothX)** | **$270-630** | **$70-130** |
| **Year 1 total** (project + 12 months new run-rate) | **$12,840 - $13,560** + SmoothX | **$32,840 - $33,560** |
| **Year 2+ annual** (new run-rate only) | **$840 - $1,560** + SmoothX | **$840 - $1,560** |

### Risk Comparison

| Risk Factor | Option 1 | Option 2 |
|------------|----------|----------|
| Development risk | Low | Medium-High |
| Financial error risk | Low | Medium-High (first 6 months) |
| API change risk | Low (SmoothX absorbs) | Medium (you own it) |
| Maintenance burden | Low (~1 hr/week) | Medium-High (~2-4 hrs/week) |
| Vendor lock-in | SmoothX dependency | None |
| Rollback complexity | Simple | Complex (must re-enable SmoothX) |

### Recommendation

**I recommend Option 1** for Frontline Holdings right now.

- **Option 1** solves the immediate problem — the $150,000/year in duplicate payments — and adds the human approval gate that's missing today. It's lower risk, faster to deploy, and doesn't require replacing infrastructure that already works.
- **Option 2** is the right choice if eliminating the SmoothX dependency is a strategic priority and you're prepared to invest in ongoing maintenance. I recommend this only with a Standard or Premium retainer.

---

## 6. Monthly Retainer (Both Options)

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
- **Option 2: Standard or Premium retainer strongly recommended** due to ongoing API maintenance and retention logic complexity

---

## 7. Timeline by Option

### Option 1: 10 Weeks

```
Week 1-2    ████████░░░░░░░░░░░░  Phase 1: Foundation
Week 3-4    ░░░░░░░░████████░░░░  Phase 2: Approval Dashboard
Week 5-7    ░░░░░░░░░░░░████████  Phase 3: Dedup Engine + Matching
Week 8-10   ████████████████████  Phase 4: Customer Invoicing & Rollout
```

### Option 2: 18 Weeks

```
Week 1-10   ████████████████████  Phases 1-4: Full Option 1 buildout
Week 11-14  ████████████████░░░░  Phase 5: SmoothX Replacement — Bills
Week 15-18  ░░░░████████████████  Phase 6: Payments, Retention & Progress
```

**Key dependencies from Frontline Holdings (both options):**
- Executive sponsor identified (before kickoff)
- Procore Developer App registered by Frontline admin (Week 1)
- Microsoft Azure app registration for Graph API (Week 1)
- Designated approver(s) for Excel dashboard (before Phase 2)
- Pilot project selected + pilot PM identified (by mid-project)
- Historical duplicate invoice data for testing (by Week 5)
- Finance team available for parallel run verification
- PMs available for 30-minute training sessions

---

## 8. Expected Results

| Metric | Current State | 90-Day Target | 6-Month Target | Options |
|--------|--------------|---------------|----------------|---------|
| Duplicate payments | $150K/year confirmed | Reduce by 80% | Reduce by 95%+ | Both |
| Human approval gate | None — invoices go straight to QBO | Every invoice requires sign-off | 100% gated | Both |
| Invoice processing time | 2-5 days | <8 hours | <4 hours | Both |
| PM time on invoice admin | 5-10 hours/week | 2-3 hours/week | <1 hour/week | Both |
| Customer invoice turnaround | 3-7 days | Same day (high-confidence) | Same day (all) | Both |
| Follow-up on overdue invoices | Ad hoc | Automated 3-tier escalation | Automated, <2% missed | Both |
| SmoothX dependency | Full | Full | Full | 1 |
| SmoothX dependency | Full | Parallel run | Eliminated | 2 |

### Why This Hits 13x in Year One

| Value Driver | Annual Impact |
|-------------|---------------|
| Duplicate payment prevention | $150,000 (confirmed baseline) |
| Faster collections (reduced DSO) | $10,000-20,000 |
| PM time savings | $5,000-10,000 |
| **Total annual value** | **$165,000-180,000** |
| **Option 1 Year 1 cost** | **$12,840-$13,560** |
| **ROI** | **~13x return** |

### If You Do Nothing

| Month | Confirmed Duplicate Exposure (at $150K/year) | Cumulative Loss |
|-------|----------------------------------------------|-----------------|
| Month 1 | ~$12,500 | $12,500 |
| Month 3 | ~$37,500 | $37,500 |
| Month 6 | ~$75,000 | $75,000 |
| Month 12 | ~$150,000 | $150,000 |

Option 1's $12,000 project fee pays for itself before the end of Month 1.

---

## 9. Why Veteran Vectors

**We've already done the homework.** Before putting this proposal together, I:

- Verified every Procore API endpoint against official developer documentation
- Confirmed OAuth 2.0 configuration (DMSA Client Credentials grant) for server-to-server automation
- Validated n8n's connection method (HTTP Request node + Generic OAuth2 — no native Procore node exists)
- Mapped SmoothX's 5 marketplace apps and identified the exact boundary between what SmoothX handles and what n8n should handle
- Evaluated all 3 Procore-QBO connector options (Procore-built, SmoothX/0link, Interfy)
- Designed the complete 5-layer dedup engine with fingerprint hashing, fuzzy matching, and AI analysis
- Built importable n8n workflow templates for all 3 core workflows
- Designed the Excel approval dashboard with Microsoft Graph API integration
- Produced a comprehensive architecture document, integration roadmap, and risk register

**We understand construction.** Invoice matching in construction isn't like e-commerce. You deal with progress billing, retainage, change orders, cost codes, and AIA-style pay applications. Same vendor, same amount, different project — that's not a duplicate. Same invoice number reformatted — that is. Our 5-layer dedup engine is designed for these construction-specific edge cases.

**We build for handoff, not dependency.** Everything we build uses open-source tools (n8n), standard APIs, and clear documentation. If Frontline Holdings ever wants to bring maintenance in-house, the operations runbook and architecture documentation make that transition straightforward.

### Case Studies

**Defense Consulting Firm**
- **Challenge:** Manual timekeeping, fragmented records across systems, billing delays
- **What we built:** AI-powered system pulling from 4 data sources, auto-generated compliant invoices, live dashboard
- **Result:** 40+ hrs/month saved, billing accuracy improved, compliance-ready

**National Insurance FMO**
- **Challenge:** Agent onboarding took days — paper forms, manual data entry, missed renewals
- **What we built:** Self-service onboarding portal, n8n + AMS integration, automated carrier appointments, compliance tracking
- **Result:** Onboarding from 3 days to 45 min, zero missed renewals, 100+ agents/month capacity

**AI Talent Marketplace Startup**
- **Challenge:** Pre-revenue startup needed MVP fast, Bubble prototype wasn't scaling, needed real infrastructure
- **What we built:** Full-stack web app (React + Node + PostgreSQL), Stripe integration, AI matching, admin dashboards
- **Result:** Live in 8 weeks, 1K+ registered users, Stripe recurring billing, featured on industry podcast

### Who I Am

**Anthony Pinto**
Founder, Veteran Vectors
anthony@veteranvectors.io

Navy veteran (USNA '14, submarine officer, deployed twice). After the Navy, I spent several years in tech — startup engineering, automation consulting, and building integration systems for companies ranging from pre-revenue startups to mid-market enterprises. I founded Veteran Vectors to bring military-grade rigor to business operations.

**By the numbers:**
- 40+ clients served
- 100% retention rate
- 150+ hours/month saved across client base

### How We Handle Your Credentials

All OAuth tokens are stored encrypted at rest — never in plaintext, never in code. Tokens are rotated on schedule (QBO every 100 days, Procore every 90 minutes via automatic refresh). n8n Cloud is SOC 2 compliant. Team members get only the minimum required access to each system.

---

## 10. Engagement Terms (Summary)

The following terms are provided for reference and will be formalized in the Statement of Work (SOW):

- **Scope management:** Changes to scope will be handled through written change orders with agreed pricing before work begins.
- **Payment terms:** 50% due at SOW signing, remainder milestone-based as outlined per option.
- **Intellectual property:** All deliverables become Frontline Holdings' property upon final payment. Veteran Vectors retains the right to reuse general knowledge and methodologies (but not Frontline-specific data or configurations).
- **Confidentiality:** All Frontline data, credentials, and business processes will be treated as confidential. No data will be shared with third parties, used for AI training, or retained after engagement completion.
- **Warranty:** 90-day warranty on all delivered workflows. Defects fixed at no charge. Does not cover third-party API changes or Frontline configuration changes.
- **Third-party services:** Frontline Holdings maintains its own subscriptions (Procore, QBO, Microsoft 365, n8n, SmoothX, Claude API). Veteran Vectors is not responsible for third-party outages or pricing changes.
- **Retainer:** Optional (strongly recommended for Option 2), begins after final phase. Either party may adjust or cancel with 30 days' notice. Overage billed at $125/hour.

*Full terms, conditions, and liability provisions will be detailed in the formal SOW.*

---

## 11. Next Steps

### Step 1 — Audit Call (Free)

60-90 minutes. I walk through your actual Procore projects, QBO setup, SmoothX configuration, and invoice flow. This confirms the build is right before anything is signed. No charge.

### Step 2 — Review & Select

Review this proposal and the accompanying Integration Roadmap Report. Pick Option 1 or 2. Ask questions.

### Step 3 — Sign & Start

SOW signed, 50% payment, building within 48 hours.

**Contact:** Anthony Pinto — anthony@veteranvectors.io

---

## Appendix A: Technology Stack by Option

| System | Option 1 | Option 2 |
|--------|----------|----------|
| **Procore** | Source of commitments, webhooks | Source of commitments, webhooks |
| **QuickBooks Online** | Accounting + customer invoices | Accounting + customer invoices |
| **SmoothX** | Keeps (Procore↔QBO sync) | Replaced by n8n |
| **n8n** | 3 workflows (dedup, matching, invoicing) + approval dashboard | 6+ workflows (all of Option 1 + bill sync, payment sync, retention, progress) |
| **Microsoft 365/OneDrive** | Excel approval dashboard (existing subscription) | Excel approval dashboard (existing subscription) |
| **Claude API** | AI matching + dedup | AI matching + dedup |
| **Gmail/SMTP** | PM notifications, customer emails | PM notifications, customer emails |

## Appendix B: API Verification Summary

All API endpoints have been verified against official documentation:

| API | Endpoints Verified | Auth Method | Rate Limit |
|-----|--------------------|------------|------------|
| Procore REST API | 10 endpoints (projects, commitments, line items, pay apps, direct costs, prime contracts, change orders, vendors, budgets, webhooks) | OAuth 2.0 Client Credentials (DMSA) | 3,600 req/hour |
| QuickBooks Online API | 7 endpoints (invoices, bills, bill payments, vendors, customers, queries, payments) | OAuth 2.0 | 500 req/minute |
| Microsoft Graph API | 4 endpoints (workbook session, table rows, range read/write, drive items) | OAuth 2.0 (Azure AD) | 10,000 req/10 min |

## Appendix C: What Happens If You Do Nothing

| Month | Confirmed Duplicate Exposure (at $150K/year rate) | Cumulative Loss |
|-------|----------------------------------------------------|-----------------|
| Month 1 | ~$12,500 | $12,500 |
| Month 3 | ~$37,500 | $37,500 |
| Month 6 | ~$75,000 | $75,000 |
| Month 12 | ~$150,000 | $150,000 |

Option 1's $12,000 project fee pays for itself before the end of Month 1.

---

*Veteran Vectors — Building systems that work as hard as you do.*
