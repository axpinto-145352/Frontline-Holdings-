# Frontline Holdings - Payment Automation Implementation

## Research Phase (Complete)
- [x] Research Frontline Holdings company profile
- [x] Research Procore Technologies API capabilities
- [x] Research SmoothX (clarified: integration middleware, NOT ERP)
- [x] Research n8n workflow automation + available nodes
- [x] Research QuickBooks Online API endpoints
- [x] Research Notion API + 2nd Brain methodology
- [x] Research duplicate payment prevention strategies

## Design Phase (Complete)
- [x] System architecture design
- [x] n8n workflow design (3 workflows)
- [x] Notion database schema design (6 databases)
- [x] Dedup engine design (5-layer strategy)
- [x] Email template design (4 templates)
- [x] API integration specifications
- [x] Implementation roadmap

## Deliverables Created
- [x] `PROPOSAL-FRONTLINE-PAYMENT-AUTOMATION.md` — Client-facing proposal with 3 engagement options and pricing
- [x] `FRONTLINE-PAYMENT-AUTOMATION.md` — Master technical architecture document (updated for 3 options)
- [x] `FRONTLINE-INTEGRATION-ROADMAP-REPORT.md` — Team-facing integration roadmap & options report (updated for 3 options)
- [x] `REVIEW-ASSESSMENT-REPORT.md` — 13-lens assessment and review findings (updated for 3 options)
- [x] `n8n-workflows/workflow-1-invoice-intake-dedup.json` — Invoice intake + dedup
- [x] `n8n-workflows/workflow-2-procore-qbo-matching.json` — Procore-QBO matching
- [x] `n8n-workflows/workflow-3-customer-invoice-followup.json` — Customer invoicing + follow-up
- [x] `notion-2nd-brain/SETUP-GUIDE.md` — Notion database setup guide (Options 2 & 3)

## Engagement Options (Client Decision Pending)

### Option 1 — Gap-Filler ($14,000 / 8 weeks)
- n8n fills SmoothX gaps (dedup, matching, invoicing, follow-up)
- SmoothX stays. No Notion. Email-based PM approvals.
- Monthly run-rate: ~$70-150/mo

### Option 2 — Gap-Filler + 2nd Brain ($22,000 / 12 weeks) — RECOMMENDED
- Everything in Option 1 + full Notion workspace as 2nd source of truth
- 6 databases, PM dashboards, one-click approvals, business-user audit trail
- Monthly run-rate: ~$320-700/mo

### Option 3 — Full Replacement ($42,000 / 20 weeks)
- Everything in Option 2 + complete SmoothX replacement
- n8n handles bill sync, payment sync, retention, progress claims
- Monthly run-rate: ~$120-200/mo (SmoothX eliminated)

## Implementation Phase (Pending Client Decision)

### All Options
- [ ] Set up n8n instance (cloud or self-hosted)
- [ ] Register Procore Developer App + configure DMSA
- [ ] Configure OAuth: Procore, QBO
- [ ] Import + normalize existing vendor list
- [ ] Build Workflow 1: Invoice Intake & 5-Layer Dedup
- [ ] Build Workflow 2: Procore-QBO AI Matching
- [ ] Build Workflow 3: Customer Invoice + Follow-Up
- [ ] Test with historical duplicate invoices
- [ ] Tune matching thresholds
- [ ] PM training
- [ ] Parallel run (old + new system)
- [ ] Full rollout

### Options 2 & 3 Only
- [ ] Set up Notion workspace with 6 databases
- [ ] Configure Notion views (PM Dashboard, Duplicate Board, etc.)
- [ ] Configure Notion API integration
- [ ] Notion-backed dedup and audit trail
- [ ] Dashboard refinement based on pilot feedback
- [ ] Leadership training on Notion dashboards

### Option 3 Only
- [ ] Build Procore → QBO vendor bill sync workflow
- [ ] Build QBO → Procore payment status sync workflow
- [ ] Build contact/vendor sync workflow
- [ ] Build cost code mapping
- [ ] Build retention/retainage tracking
- [ ] Build progress claim workflows
- [ ] 4-week parallel run (SmoothX + n8n producing identical results)
- [ ] SmoothX cutover and subscription cancellation

## Next Steps (Client Action Required)
- [ ] Review proposal and select engagement option (1, 2, or 3)
- [ ] Identify executive sponsor
- [ ] Schedule walkthrough call with Veteran Vectors
- [ ] Sign Statement of Work (SOW)
