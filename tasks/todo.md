# Frontline Holdings - Payment Automation Implementation

## Research Phase (Complete)
- [x] Research Frontline Holdings company profile
- [x] Research Procore Technologies API capabilities
- [x] Research SmoothX (clarified: integration middleware, NOT ERP)
- [x] Research n8n workflow automation + available nodes
- [x] Research QuickBooks Online API endpoints
- [x] Research Microsoft Graph API for Excel/OneDrive integration
- [x] Research duplicate payment prevention strategies

## Design Phase (Complete)
- [x] System architecture design
- [x] n8n workflow design (3 workflows)
- [x] Excel approval dashboard design (4 tabs)
- [x] Dedup engine design (5-layer strategy)
- [x] Email template design (4 templates)
- [x] API integration specifications
- [x] Implementation roadmap

## Deliverables Created
- [x] `PROPOSAL-FRONTLINE-PAYMENT-AUTOMATION.md` — Client-facing proposal with 2 engagement options and pricing
- [x] `FRONTLINE-PAYMENT-AUTOMATION.md` — Master technical architecture document (updated for 2 options)
- [x] `FRONTLINE-INTEGRATION-ROADMAP-REPORT.md` — Team-facing integration roadmap & options report (updated for 2 options)
- [x] `REVIEW-ASSESSMENT-REPORT.md` — 13-lens assessment and review findings (updated for 2 options)
- [x] `n8n-workflows/workflow-1-invoice-intake-dedup.json` — Invoice intake + dedup
- [x] `n8n-workflows/workflow-2-procore-qbo-matching.json` — Procore-QBO matching
- [x] `n8n-workflows/workflow-3-customer-invoice-followup.json` — Customer invoicing + follow-up
- [x] `notion-2nd-brain/SETUP-GUIDE.md` — Notion database setup guide (reference only — not in current proposal)

## Engagement Options (Client Decision Pending)

### Option 1 — Approval Gate + Gap-Filler ($12,000 / 10 weeks) — RECOMMENDED
- Human approval gate via Excel dashboard in OneDrive
- n8n fills SmoothX gaps (dedup, matching, invoicing, follow-up)
- SmoothX stays. Excel approval dashboard. PM approves via Excel.
- Monthly run-rate: ~$70-130/mo (new costs)
- Total monthly (incl. existing SmoothX): ~$270-630/mo

### Option 2 — Full Replacement ($32,000 / 18 weeks)
- Everything in Option 1 + complete SmoothX replacement
- n8n handles bill sync, payment sync, retention, progress claims
- Monthly run-rate: ~$70-130/mo (SmoothX eliminated)

## Implementation Phase (Pending Client Decision)

### Both Options
- [ ] Set up n8n instance (cloud or self-hosted)
- [ ] Register Procore Developer App + configure DMSA
- [ ] Configure OAuth: Procore, QBO, Microsoft Graph API
- [ ] Register Azure app for Microsoft Graph API access
- [ ] Create Excel approval dashboard template in OneDrive
- [ ] Import + normalize existing vendor list
- [ ] Build Excel dashboard integration (Microsoft Graph API read/write)
- [ ] Build PM approval workflow (poll Excel every 5 min)
- [ ] Build auto-escalation (4-hour timeout)
- [ ] Build Workflow 1: Invoice Intake & 5-Layer Dedup
- [ ] Build Workflow 2: Procore-QBO AI Matching
- [ ] Build Workflow 3: Customer Invoice + Follow-Up
- [ ] Test with historical duplicate invoices
- [ ] Tune matching thresholds
- [ ] PM training (30-min walkthrough)
- [ ] Parallel run (old + new system)
- [ ] Full rollout

### Option 2 Only
- [ ] Build Procore → QBO vendor bill sync workflow
- [ ] Build QBO → Procore payment status sync workflow
- [ ] Build contact/vendor sync workflow
- [ ] Build cost code mapping
- [ ] Build retention/retainage tracking
- [ ] Build progress claim workflows
- [ ] 4-week parallel run (SmoothX + n8n producing identical results)
- [ ] SmoothX cutover and subscription cancellation

## Next Steps (Client Action Required)
- [ ] Schedule audit call (60-90 min, free) with Veteran Vectors
- [ ] Review proposal and select engagement option (1 or 2)
- [ ] Identify executive sponsor
- [ ] Sign Statement of Work (SOW) — 50% payment, building within 48 hours
