# Professional Services Proposal
## Payment Automation & Duplicate Prevention System

---

**Prepared by:** Veteran Vectors
**Prepared for:** Frontline Holdings LLC
**Date:** February 28, 2026
**Proposal Valid Through:** March 31, 2026
**Version:** 1.0

---

## Dear Frontline Holdings Team,

Thank you for the opportunity to present this proposal. We understand the urgency of your duplicate payment challenge and the financial impact it has on your operations. This proposal outlines how Veteran Vectors will design, build, and deploy a complete payment automation system that eliminates duplicate payments, automates invoice matching, and gives your team real-time financial visibility across Procore, QuickBooks Online, and Notion.

We've already completed extensive research into your technology stack, verified every API endpoint we'll be building against, and designed the full system architecture. We're ready to start.

---

## 1. Understanding Your Challenge

Frontline Holdings operates as a veteran-owned, vertically integrated construction management and general contracting firm across multiple divisions — Frontline Builders, Frontline Construction Management, Government & Defense Contracting, and Land Development. You manage multiple active projects simultaneously across Texas, Florida, Oklahoma, Canada, and international markets, working with dozens of subcontractors per project and processing hundreds of invoices monthly.

**The core problem:** Invoices arrive through multiple channels — email, Procore uploads, and physical mail — and are entered into QuickBooks Online by different people at different times with no cross-system deduplication. The result is duplicate and triple payments that erode margins in an industry already operating at 3-8%.

**Estimated financial exposure:** Based on industry benchmarks (1-2% duplicate rate on ~500 invoices/month at ~$15,000 average), the annual exposure is approximately **$720,000+**. We recommend validating this figure with a targeted AP audit as part of Phase 1.

**What's missing today:**
- No automated deduplication between Procore and QBO
- No real-time matching of invoices to Procore commitments
- No single source of truth for invoice status across systems
- No automated PM notifications or approval workflows
- No systematic follow-up on unpaid customer invoices
- No audit trail connecting invoice receipt to final payment

---

## 2. Proposed Solution

Veteran Vectors will build an **AI-powered payment automation system** using n8n (workflow automation) as the orchestration layer, Notion as the centralized "2nd Brain," and Claude AI for intelligent invoice matching — all integrated with your existing Procore and QuickBooks Online environments.

### Architecture: Hybrid Approach (SmoothX + n8n)

Your existing SmoothX integration continues handling what it does well — the Procore-to-QBO financial sync (vendor bills, payments, contacts, cost codes, retention, and progress claims). Veteran Vectors builds the automation layer on top:

```
SmoothX continues handling:             Veteran Vectors builds:
├── Procore → QBO bill sync             ├── 5-layer dedup engine
├── QBO → Procore payment sync          ├── AI-powered invoice matching
├── Contact/vendor sync                 ├── Notion "2nd Brain" (6 databases)
├── Cost code mapping                   ├── PM notification & approval workflows
├── Retention/retainage                 ├── Customer invoice generation
├── Progress claims                     ├── Automated follow-up & escalation
└── API change absorption              ├── Real-time Procore webhook routing
                                        └── Financial dashboards & audit trail
```

**Key architectural rule:** n8n never writes vendor bills to QBO — SmoothX owns that sync. n8n reads from QBO to verify and cross-reference, and writes only customer invoices. This eliminates any dual-write conflict.

### What Your Team Gets

**For Project Managers:**
- Email alerts when invoices arrive, with automatic Procore commitment match results
- One-click approve/reject from Notion dashboard
- No more manual invoice hunting across systems
- Automated follow-up on overdue customer invoices

**For Finance/Accounting:**
- Every invoice fingerprinted and checked against 5 dedup layers before processing
- Duplicate invoices automatically blocked with PM notification
- Complete audit trail in Notion from receipt to payment
- Real-time dashboards: outstanding receivables, overdue invoices, vendor payment history

**For Leadership:**
- Single source of truth across Procore, QBO, and Notion
- Measurable reduction in duplicate payment losses
- Per-project financial visibility
- Automated collections process with 3-tier escalation

---

## 3. Scope of Work

### Phase 1: Foundation (Weeks 1-2)

| Deliverable | Details |
|-------------|---------|
| Notion workspace setup | 6 interconnected databases: Invoice Tracker, Projects, Vendors, Customers, Audit Log, Payments |
| Notion dashboard views | PM Dashboard, Duplicate Alert Board, Cash Flow Board, Project Financial Summary, Vendor Payment History, Audit Trail |
| n8n instance setup | Cloud deployment with OAuth credential configuration |
| Procore Developer App | Register app, configure DMSA (Client Credentials), verify API connectivity |
| QBO integration | Configure native n8n QuickBooks node, verify read/write access |
| Notion integration | Configure native n8n Notion node, verify database operations |
| Vendor data import | Pull vendors from Procore and QBO, normalize names, populate Vendors database |
| API connectivity verification | End-to-end test of all system connections |

**Milestone:** All systems connected. Notion databases populated with current vendor/project data. Ready for workflow development.

### Phase 2: Dedup Engine (Weeks 3-4)

| Deliverable | Details |
|-------------|---------|
| Workflow 1: Invoice Intake | Procore webhook trigger for real-time invoice capture |
| Fingerprint generator | SHA256 hash of normalized vendor + invoice number + amount + project |
| Layer 1: Exact match | Notion fingerprint query — catches identical invoices |
| Layer 2: Fuzzy match | Vendor name similarity >85% + amount within 1% + date within 30 days |
| Layer 3: Pattern match | Same vendor + same amount within 45-day window |
| Layer 4: Procore cross-reference | Total payments vs. committed amount — catches overpayments |
| Layer 5: AI analysis | Claude API intelligent pattern detection for sophisticated duplicates |
| Duplicate alert emails | PM notification with invoice details, match type, and Notion dashboard link |
| Threshold tuning | Test against historical data (provided by Frontline Finance team), adjust confidence levels |

**Milestone:** Dedup engine catching duplicates on test data. PMs receiving alert emails. False positive rate measured and tuned.

### Phase 3: Matching Engine (Weeks 5-6)

| Deliverable | Details |
|-------------|---------|
| Workflow 2: Procore-QBO Matching | Automatic commitment matching for incoming invoices |
| AI matching logic | Claude API scores match confidence: HIGH (>90%), MEDIUM (60-90%), LOW (<60%) |
| Routing logic | HIGH = auto-match + log + FYI email to PM. MEDIUM = flag for PM review. LOW = manual match required. |
| PM approval flow | Email with invoice details, Procore commitment match, approve/reject action via Notion |
| Notion logging | Every invoice creates a full audit record with match details, confidence score, and status |
| Live data testing | Run against real Procore commitments in sandbox/test project |

**Milestone:** Invoices automatically matched to Procore commitments. PMs approving via email/Notion. Full audit trail operational.

### Phase 4: Customer Invoicing & Follow-Up (Weeks 7-8)

| Deliverable | Details |
|-------------|---------|
| Workflow 3: Customer Invoice Generation | On PM approval, create QBO customer invoice and email to customer |
| Automated follow-up | Daily check for overdue invoices with 3-tier escalation |
| Escalation tier 1 (1-15 days) | Gentle reminder email to customer |
| Escalation tier 2 (16-30 days) | Firm reminder to customer, CC PM |
| Escalation tier 3 (31+ days) | Escalation alert to PM + Finance team |
| QBO payment verification | Check QBO payment status before sending reminders (prevents reminders on paid invoices) |
| Notion status updates | Automatic status progression: Approved → Invoiced → Follow-Up Sent → Paid |
| End-to-end testing | Full lifecycle test: receive → dedup → match → approve → invoice → follow-up → paid |

**Milestone:** Full end-to-end system running on test data. Customer invoices generating automatically. Follow-ups operational.

### Phase 5: Pilot, Rollout & Training (Weeks 9-12)

| Deliverable | Details |
|-------------|---------|
| Pilot project deployment | Live production deployment on one selected project |
| Parallel run (2-4 weeks) | Old process + new system running side-by-side for verification |
| PM training | 30-minute hands-on walkthrough per PM: what to expect, how to approve, where to check status |
| Finance team training | Dashboard walkthrough, audit trail review, edge case handling |
| Threshold refinement | Adjust matching confidence and dedup sensitivity based on pilot results |
| Full rollout | Expand from pilot to all active projects |
| Operations runbook | Documentation: edge case handling, manual overrides, system error procedures |
| 90-day review criteria | Defined success metrics for post-rollout evaluation |

**Milestone:** System live across all projects. Team trained. Measurable reduction in duplicate payments. Runbook delivered.

---

## 4. Deliverables Summary

| # | Deliverable | Format |
|---|------------|--------|
| 1 | Notion workspace with 6 databases + 6 dashboard views | Live Notion workspace |
| 2 | n8n Workflow 1: Invoice Intake & 5-Layer Dedup Engine | Deployed n8n workflow |
| 3 | n8n Workflow 2: Procore-QBO AI Matching Engine | Deployed n8n workflow |
| 4 | n8n Workflow 3: Customer Invoice & Automated Follow-Up | Deployed n8n workflow |
| 5 | Procore webhook integration (real-time event capture) | Configured in Procore + n8n |
| 6 | QBO integration (customer invoices, payment verification) | Configured in n8n |
| 7 | PM notification email templates (4 templates) | HTML email templates in n8n |
| 8 | Customer email templates (invoice + 3 follow-up tiers) | HTML email templates in n8n |
| 9 | Operations runbook | Markdown document |
| 10 | PM + Finance team training (recorded) | Live sessions + recording |
| 11 | System architecture documentation | Markdown document (already drafted) |
| 12 | 90-day review report | Assessment of performance vs. targets |

---

## 5. Investment

### Project Fee: Fixed Price

| Phase | Scope | Fee |
|-------|-------|-----|
| Phase 1: Foundation | Notion setup, n8n deployment, OAuth config, vendor import, API verification | $3,750 |
| Phase 2: Dedup Engine | 5-layer dedup workflow, fingerprinting, AI matching, duplicate alerts, threshold tuning | $5,000 |
| Phase 3: Matching Engine | Procore-QBO commitment matching, AI scoring, PM approval flow, Notion logging | $5,000 |
| Phase 4: Customer Invoicing | Invoice generation, automated follow-up, 3-tier escalation, end-to-end testing | $4,500 |
| Phase 5: Pilot & Rollout | Pilot deployment, parallel run, PM/Finance training, full rollout, runbook, 90-day criteria | $3,750 |
| **Total Project Fee** | | **$22,000** |

**Payment schedule:**

| Milestone | Amount | When |
|-----------|--------|------|
| Project kickoff | $5,500 (25%) | Upon signed agreement |
| Phase 2 complete (dedup engine operational) | $5,500 (25%) | ~Week 4 |
| Phase 4 complete (end-to-end system tested) | $5,500 (25%) | ~Week 8 |
| Phase 5 complete (full rollout + training) | $5,500 (25%) | ~Week 12 |

### Monthly Retainer: Ongoing Support & Maintenance

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
- Retainer begins the month following Phase 5 completion

### What's Included in the Project Fee

- All development, testing, and deployment labor
- n8n workflow design, build, and configuration
- Notion workspace architecture and setup
- Procore and QBO API integration and OAuth configuration
- AI matching prompt engineering and threshold tuning
- PM and Finance team training (live + recorded)
- Operations runbook
- Architecture documentation
- 2 weeks of post-launch support (bug fixes, adjustments) included at no additional cost

### What's Not Included

- Procore, QBO, Notion, or SmoothX subscription fees (existing Frontline subscriptions)
- n8n Cloud subscription (~$50-100/month — Frontline's account)
- Claude API usage fees (~$20-50/month — Frontline's account)
- Procore Developer App registration (free, requires Frontline admin action)
- Changes to SmoothX configuration (managed by SmoothX support)
- Historical AP audit (recommended but performed by Frontline Finance team)
- Hardware, servers, or infrastructure (n8n Cloud is fully managed)

---

## 6. Timeline

```
Week 1-2    ████████░░░░░░░░░░░░░░░░  Phase 1: Foundation
Week 3-4    ░░░░░░░░████████░░░░░░░░  Phase 2: Dedup Engine
Week 5-6    ░░░░░░░░░░░░░░░░████████  Phase 3: Matching Engine
Week 7-8    ████████░░░░░░░░░░░░░░░░  Phase 4: Customer Invoicing
Week 9-12   ░░░░░░░░████████████████  Phase 5: Pilot, Rollout & Training
```

**Total duration:** 12 weeks from kickoff to full rollout

**Key dependencies from Frontline Holdings:**
- Executive sponsor identified (before kickoff)
- Procore Developer App registered by Frontline admin (Week 1)
- Pilot project selected + pilot PM identified (by Week 4)
- Historical duplicate invoice data for testing (by Week 3)
- Finance team available for parallel run verification (Weeks 9-10)
- PMs available for 30-minute training sessions (Weeks 10-11)

---

## 7. Expected Results

| Metric | Current State | 90-Day Target | 6-Month Target |
|--------|--------------|---------------|----------------|
| Duplicate payment rate | ~2% estimated | Reduce by 80% | Reduce by 90%+ |
| Invoice processing time | 2-5 days | <8 hours | <4 hours |
| PM time on invoice admin | 5-10 hours/week | 3-5 hours/week | <1 hour/week |
| Customer invoice turnaround | 3-7 days | Same day (high-confidence) | Same day (all) |
| Follow-up on overdue invoices | Ad hoc | Automated 3-tier escalation | Automated, <2% missed |
| Financial visibility | Fragmented | Single Notion dashboard | Fully operational |
| Audit trail | Partial / manual | Complete automated log | 100% coverage |

**Conservative ROI estimate:** At a 1% reduction in duplicate payments (half the estimated 2% rate), the system saves approximately $7,500/month in prevented overpayments. Against the $22,000 project fee, the system pays for itself in **less than 3 months**. Against the monthly retainer ($625-2,500/month), the ongoing return is **3x-12x the investment**.

---

## 8. Why Veteran Vectors

**We've already done the homework.** Before presenting this proposal, we:

- Verified every Procore API endpoint against official developer documentation
- Confirmed OAuth 2.0 configuration (DMSA Client Credentials grant) for server-to-server automation
- Validated n8n's connection method (HTTP Request node + Generic OAuth2 — no native Procore node exists)
- Mapped SmoothX's 5 marketplace apps and identified the exact boundary between what SmoothX handles and what n8n should handle
- Evaluated all 3 Procore-QBO connector options (Procore-built, SmoothX/0link, Interfy)
- Designed the complete 5-layer dedup engine with fingerprint hashing, fuzzy matching, and AI analysis
- Built importable n8n workflow templates for all 3 workflows
- Designed the full 6-database Notion schema with dashboard views
- Produced a comprehensive architecture document, integration roadmap, and risk register

**We understand construction.** Invoice matching in construction isn't like e-commerce. You deal with progress billing, retainage, change orders, cost codes, and AIA-style pay applications. Same vendor, same amount, different project — that's not a duplicate. Same invoice number reformatted — that is. Our 5-layer dedup engine is designed for these construction-specific edge cases.

**We build for handoff, not dependency.** Everything we build uses open-source tools (n8n), standard APIs, and clear documentation. If Frontline Holdings ever wants to bring maintenance in-house, the operations runbook and architecture documentation make that transition straightforward.

---

## 9. Engagement Terms (Summary)

The following terms are provided for reference and will be formalized in the Statement of Work (SOW):

- **Scope management:** Changes to scope will be handled through written change orders with agreed pricing before work begins.
- **Payment terms:** Net 15 from invoice date, with milestone-based billing as outlined in Section 5.
- **Intellectual property:** All deliverables become Frontline Holdings' property upon final payment. Veteran Vectors retains the right to reuse general knowledge and methodologies (but not Frontline-specific data or configurations).
- **Confidentiality:** All Frontline data, credentials, and business processes will be treated as confidential. No data will be shared with third parties, used for AI training, or retained after engagement completion.
- **Warranty:** 90-day warranty on all delivered workflows. Defects fixed at no charge. Does not cover third-party API changes or Frontline configuration changes.
- **Third-party services:** Frontline Holdings maintains its own subscriptions (Procore, QBO, Notion, n8n, SmoothX, Claude API). Veteran Vectors is not responsible for third-party outages or pricing changes.
- **Retainer:** Optional, begins after Phase 5. Either party may adjust or cancel with 30 days' notice. Overage billed at $125/hour.

*Full terms, conditions, and liability provisions will be detailed in the formal SOW.*

---

## 10. Next Steps

This proposal is provided for awareness and understanding. A separate Statement of Work (SOW) will be prepared for formal execution upon alignment.

**To move forward:**

1. Review this proposal and the accompanying Integration Roadmap Report
2. Identify an executive sponsor for the initiative
3. Schedule a walkthrough call with Veteran Vectors to discuss questions and preferences
4. Upon alignment, Veteran Vectors will deliver a formal SOW for signature

**Contact:** Please reach out to Veteran Vectors to schedule a discussion or request any clarification on the contents of this proposal.

---

## Appendix A: Technology Stack Summary

| System | Role in This Project | Subscription Owner |
|--------|---------------------|-------------------|
| **Procore** | Source of commitments, pay apps, vendors, change orders. Webhook triggers. | Frontline Holdings |
| **QuickBooks Online** | Accounting system. Customer invoice generation. Payment verification for follow-ups. | Frontline Holdings |
| **SmoothX** | Existing Procore↔QBO sync (bills, payments, contacts, cost codes, retention). Remains as-is. | Frontline Holdings |
| **n8n** | Workflow orchestration engine. Runs all 3 automated workflows. | Frontline Holdings |
| **Notion** | "2nd Brain" — single source of truth. 6 databases, dashboards, audit trail. | Frontline Holdings |
| **Claude API** | AI-powered invoice matching, vendor normalization, duplicate pattern detection. | Frontline Holdings |
| **Gmail/SMTP** | PM notifications, customer invoices, follow-up reminders. | Frontline Holdings |

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

The system's $22,000 project fee is equivalent to preventing **1.5 duplicate payments** at the average invoice value of $15,000.

---

*Veteran Vectors — Building systems that work as hard as you do.*
