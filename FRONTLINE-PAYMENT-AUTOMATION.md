# Frontline Holdings - Payment Automation System
## AI-Powered Invoice Matching & Duplicate Payment Prevention

**Date:** 2026-02-28
**Version:** 3.0
**Prepared for:** Frontline Holdings Team
**Prepared by:** Veteran Vectors
**System:** QBO <> Procore <> Excel/OneDrive <> n8n Orchestration

> **Engagement options:** This technical architecture document supports both engagement options presented in the client proposal (`PROPOSAL-FRONTLINE-PAYMENT-AUTOMATION.md`):
> - **Option 1 — Approval Gate + Gap-Filler ($12,000):** Sections 1-6, 8-9 apply. Section 7 (Excel Approval Dashboard) applies.
> - **Option 2 — Full Replacement ($32,000):** All sections apply, plus SmoothX replacement workflows.

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Company Context: Frontline Holdings](#company-context)
3. [The Problem: Duplicate/Triple Payments](#the-problem)
4. [Technology Stack Research](#technology-stack)
5. [System Architecture](#system-architecture)
6. [n8n Workflow Design](#n8n-workflow-design)
7. [Excel Approval Dashboard Setup](#excel-approval-dashboard)
8. [Duplicate Payment Prevention Engine](#duplicate-payment-prevention)
9. [API Integration Specifications](#api-specifications)
10. [Implementation Roadmap](#implementation-roadmap)

---

## 1. Executive Summary

Frontline Holdings, a construction/general contracting company, is experiencing **critical financial losses from duplicate and triple payments** due to disconnected systems between their project management (Procore), accounting (QuickBooks Online), and integration middleware (SmoothX). This document outlines a complete automation system using **n8n workflow automation** as the orchestration layer, **an Excel approval dashboard in OneDrive** for human sign-off and audit trail, and **AI-powered matching logic** to eliminate duplicate payments and streamline the invoice-to-payment lifecycle.

### Key Outcomes (All Options)
- **Eliminate 100% of duplicate/triple payments** through automated 5-layer deduplication
- **Reduce invoice processing time by 70%** through AI-powered matching
- **Automated PM notifications** with follow-up email sequences
- **Customer invoice automation** with 3-tier escalation for overdue invoices

### Additional Outcomes (Both Options)
- **Human approval gate** — nothing hits QBO without PM sign-off via Excel
- **Centralized approval dashboard** in Excel/OneDrive for all payment status tracking
- **Real-time visibility** for PMs via shared Excel file in OneDrive
- **Full audit trail** in Excel Audit Log tab from invoice receipt to customer payment

### Additional Outcomes (Option 2 Only)
- **SmoothX eliminated** — complete Procore↔QBO sync handled by n8n
- **No vendor dependency** — full control over every data flow

---

## 2. Company Context: Frontline Holdings

### Company Profile
**Frontline Holdings LLC** (buildfrontline.com) is a **veteran-owned, vertically integrated construction management and development firm** founded by Brad Bruggemann and Everett Wineman. The company operates as both a **construction manager** and a **general contractor** through multiple divisions:

- **Frontline Builders** — General contracting delivering full project solutions
- **Frontline Construction Management** — CM services overseeing all development phases
- **Government & Defense Contracting** — Public-sector and defense clients (DCAA/FAR compliance)
- **Land Development** — Raw acreage to build-ready lots

**Sectors:** Healthcare, Government, Higher Education, Multifamily, and Commercial
**Geography:** Texas (primarily), Florida, Oklahoma, Canada, and international

### Industry Profile
Frontline Holdings operates in the construction industry as a **general contractor and construction manager**. Construction companies of this profile typically:

- Manage multiple active projects simultaneously
- Work with dozens of subcontractors and vendors per project
- Process hundreds of invoices monthly across projects
- Use progress billing (AIA-style pay applications)
- Must track retainage, change orders, and committed costs
- Operate on thin margins (typically 3-8%) where payment errors are devastating

### Current Technology Stack
| System | Purpose | Status |
|--------|---------|--------|
| **Procore** | Project management, commitments, change orders, pay applications | Primary PM tool |
| **QuickBooks Online (QBO)** | Accounting, AP/AR, general ledger | Primary accounting |
| **SmoothX** | Integration middleware (Procore↔QBO sync) | Operational sync layer |
| **Email** | Invoice receipt, PM communication, customer invoicing | Manual process |

### Current Pain Points
1. **No single source of truth** — invoice data lives in 3+ systems
2. **Manual invoice matching** — PMs manually cross-reference Procore commitments with QBO entries
3. **Duplicate payments** — same invoice entered in QBO multiple times (different people, different times)
4. **Triple payments** — invoice paid via QBO, then re-entered when found in email, then again when PM submits
5. **No automated reconciliation** between Procore commitments and QBO payments
6. **PM bottleneck** — PMs manually route invoices and lose track of status
7. **No follow-up system** — unpaid customer invoices fall through the cracks

---

## 3. The Problem: Duplicate/Triple Payments

### Root Cause Analysis

```
                    INVOICE ARRIVES
                         |
            +------------+------------+
            |            |            |
        Via Email    Via Procore   Via Mail
            |            |            |
        AP Clerk      PM enters    Office Mgr
        enters QBO    in Procore   enters QBO
            |            |            |
            v            v            v
        PAYMENT 1    PAYMENT 2    PAYMENT 3
        (duplicate)  (duplicate)  (triplicate)
```

### Why This Happens
1. **Multiple entry points** — invoices arrive via email, mail, Procore uploads, and PM submissions
2. **No dedup check** — QBO doesn't natively cross-reference against Procore
3. **Timing gaps** — invoice entered Monday, PM submits same invoice Thursday, both get paid
4. **Vendor number inconsistency** — same vendor has multiple IDs across systems
5. **Invoice number variations** — "INV-001", "001", "#001" treated as different invoices
6. **No approval workflow** — payments go out without matching against commitments

### Financial Impact
Last year, Frontline lost **$150,000 to duplicate payments** — a confirmed, validated number. Based on industry benchmarks for a construction company processing ~500 invoices/month with an average invoice of $15,000:
- **A 2% duplicate rate on 500 invoices/month at $15,000 average = up to $150,000/month in theoretical exposure**
- Recovery rate on duplicate payments in construction: ~60% (40% is lost)
- **$150,000 confirmed loss last year provides the ROI basis for this engagement**

---

## 4. Technology Stack Research

### 4.1 Procore Technologies

**What it is:** The leading cloud-based construction management platform serving GCs, specialty contractors, and owners. Used by 16,000+ companies worldwide.

**Key API Capabilities for This Integration:**

| API Resource | Endpoint | Purpose |
|-------------|----------|---------|
| Projects | `GET /rest/v1.0/projects` | List all active projects |
| Commitments (Subcontracts) | `GET /rest/v1.1/projects/{id}/commitments` | Get all subcontracts/POs |
| Commitment Line Items | `GET /rest/v1.0/projects/{id}/commitments/{cid}/line_items` | Get SOV line items |
| Pay Applications | `GET /rest/v1.0/projects/{id}/commitments/{cid}/requisitions` | Subcontractor pay apps |
| Subcontractor Invoices (Requisitions) | `GET /rest/v1.0/requisitions?project_id={id}` | Sub pay apps (legacy name: requisitions) |
| Direct Costs | `GET /rest/v1.1/projects/{id}/direct_costs` | Expenses, invoices, payroll |
| Prime Contracts | `GET /rest/v1.0/prime_contracts?project_id={id}` | Owner contracts |
| Payment Applications | See Payment Applications Reference | Owner invoices (billing) |
| Change Orders | `GET /rest/v1.0/projects/{id}/change_orders` | Track budget changes |
| Vendors | `GET /rest/v1.0/companies/{cid}/vendors` | Vendor directory |
| Budget | `GET /rest/v1.0/projects/{id}/budget/views` | Budget views with costs |
| Webhooks | `POST /rest/v1.0/companies/{cid}/webhooks` | Real-time event triggers |

**Authentication:** OAuth 2.0 — two supported grant types:

| Grant Type | Use Case | Token Expiry |
|-----------|----------|--------------|
| **Authorization Code** | User-interactive apps, requires redirect URI | Reducing from 2 hours to 15 minutes |
| **Client Credentials (DMSA)** | Server-to-server / automated workflows (recommended for n8n) | 5,400 seconds (1.5 hours) |

**OAuth Endpoints:**

| Environment | Authorization URL | Token URL |
|-------------|-------------------|-----------|
| Production | `https://login.procore.com/oauth/authorize` | `https://login.procore.com/oauth/token` |
| Monthly Sandbox | `https://login-sandbox-monthly.procore.com/oauth/authorize` | `https://login-sandbox-monthly.procore.com/oauth/token` |
| Dev Sandbox | `https://login-sandbox.procore.com/oauth/authorize` | `https://login-sandbox.procore.com/oauth/token` |

**Important:** Procore does NOT use OAuth scopes. Permissions are controlled via Procore's project-level permission system (DMSA). All calls require the `Procore-Company-Id` header.

**Rate Limits:** 3,600 requests/hour per app (can request increase to 7,200 or 14,400 via apisupport@procore.com). Rate limit headers: `X-Rate-Limit-Remaining`, `X-Rate-Limit-Reset`. Throttled = HTTP 429.

**Webhooks:** Create/Update/Delete events on any API resource. Payload fields: `id`, `timestamp`, `resource_name`, `resource_id`, `event_type`, `company_id`, `project_id`. 28-day delivery history. Available resources enumerable via `GET /rest/v1.0/webhooks/resources`.

**API Versioning:** Resource-level versioning (v1.0 vs v1.1 per resource, not per API). v1.1 means a breaking change to that specific resource. v2.0 introduces `"data"` wrapper, string IDs, and pagination.

**n8n Connection Method:** No native Procore node exists in n8n. Use **HTTP Request node** with **Generic OAuth2 credentials**:
1. Grant Type: `Client Credentials`
2. Access Token URL: `https://login.procore.com/oauth/token`
3. Client ID / Secret: From Procore Developer App
4. Add Header: `Procore-Company-Id: <your_company_id>`
5. n8n handles token refresh automatically

**Key Procore Data for Matching:**
- Commitment number (links to subcontract/PO)
- Vendor ID + vendor name
- Approved amount on commitment
- Pay application amounts (what's been billed)
- Remaining committed amount
- Project cost codes

### 4.2 SmoothX (Integration Middleware — NOT an ERP)

**Critical Finding:** SmoothX is **NOT an ERP system**. It is a **construction technology integration middleware provider** (smoothx.com) that specializes in connecting Procore with accounting platforms. SmoothX acts as an intelligent translation layer between systems. Listed on the Procore Marketplace as "0link QuickBooks Online Connector by Smoothx." 1,000+ Procore connections worldwide. Infrastructure runs on AWS and DigitalOcean.

**SmoothX Apps on Procore Marketplace:**

| App | Marketplace URL | Description |
|-----|----------------|-------------|
| **0link QBO Connector** | marketplace.procore.com/apps/quickbooks-online-connector-by-0link | Bi-directional Procore ↔ QBO real-time sync |
| **Smoothx Advanced Payments** | marketplace.procore.com/apps/smoothx-advanced-payments | Extended payment management |
| **ProScan+ by Smoothx** | marketplace.procore.com/apps/proscan-by-smoothlink | OCR-powered invoice scanning into Procore |
| **Extractus by Smoothx** | marketplace.procore.com/apps/extractus-by-smoothx | Data extraction tool |
| **Cost+ by Smoothx** | marketplace.procore.com/apps/cost-by-smoothx | Cost-plus management |

**Alternative Procore ↔ QBO Connectors (Marketplace):**

| Connector | Type | Key Limitation |
|-----------|------|----------------|
| **Procore-built QBO Connector** | First-party (free) | New projects only; no pre-existing projects without paid Pro Services; 1 QBO company per site; no negative invoices; US-only payment import |
| **Interfy OneCore** | Third-party | Alternative to SmoothX |
| **SmoothX (0link)** | Third-party | Most comprehensive: retention, progress claims, attachments, payroll |

**Key SmoothX Capabilities:**
- Real-time bi-directional sync (no user interaction after onboarding)
- Syncs: Contacts, Cost Codes, Cost Types, Commitments, Direct Costs, Progress Claims, Payroll/Timesheets
- Attachments travel with invoices/bills (full audit trail)
- Retention/retainage handling on progress claims
- Construction-specific: FIDIC contract standards, multi-region tax rules
- ERP Locking — synced items display green ERP banner in Procore and cannot be amended
- Setup time: ~30 minutes onboarding call, operational within 48 hours
- SmoothAssist — real-time sync status dashboard with green tick/red cross indicators

**SmoothX vs n8n — What Each Can Do:**

| Capability | SmoothX | n8n Direct Build |
|-----------|---------|-----------------|
| Two-way real-time invoice sync | Pre-built | Must build (weeks) |
| Contact/vendor sync | Pre-built | Must build (days) |
| Cost code mapping | Pre-built (CSV import) | Must build (days) |
| Retention/retainage handling | Pre-built | Must build (complex) |
| Progress claims | Pre-built | Must build (complex) |
| Tax management (multi-region) | Pre-built | Must build |
| API change maintenance | SmoothX handles | You own it forever |
| **Custom dedup logic** | No | **Yes** |
| **AI-powered matching** | No | **Yes** |
| **Custom PM notifications** | No | **Yes** |
| **Follow-up email automation** | No | **Yes** |
| **Excel approval dashboard logging** | No | **Yes** |
| **Custom approval workflows** | No | **Yes** |

**Do You Need SmoothX? Decision Matrix:**

| Scenario | SmoothX? | Why |
|----------|----------|-----|
| Full Procore↔QBO financial sync (invoices, payments, contacts, retention, progress claims) | **YES** | Months of dev to replicate; ongoing maintenance |
| Simple one-way data reads (webhook triggers, queries) | **NO** | n8n HTTP Request handles this in days |
| Custom workflows (approvals, AI matching, notifications) | **NO** | n8n is purpose-built for this |
| You want zero vendor lock-in | **NO** | But budget 4-8 weeks dev + ongoing maintenance |

**RECOMMENDED ARCHITECTURE — Hybrid Approach:**

```
SmoothX handles:                    n8n handles:
├─ Procore → QBO bill sync          ├─ Dedup engine (5-layer)
├─ QBO → Procore payment sync       ├─ AI-powered invoice matching
├─ Contact/vendor sync              ├─ Excel approval dashboard (Graph API)
├─ Cost code mapping                ├─ Human approval gate (Excel)
├─ Retention/retainage              ├─ PM notification emails
├─ Progress claims                  ├─ Customer invoice generation
└─ API change absorption            ├─ Follow-up & escalation
                                    ├─ Custom approval workflows
                                    └─ Procore webhook event routing
```

**CRITICAL RULE: n8n does NOT write vendor bills to QBO.** SmoothX owns that sync. n8n READS from QBO to verify and cross-reference, and WRITES only customer invoices (which SmoothX does not handle). This eliminates the dual-write conflict.

### 4.3 QuickBooks Online (QBO)

**API Capabilities:**

| API Resource | Endpoint | Purpose |
|-------------|----------|---------|
| Invoices | `POST /v3/company/{id}/invoice` | Create customer invoices |
| Bills | `GET /v3/company/{id}/bill` | Vendor bills (AP) |
| Bill Payments | `POST /v3/company/{id}/billpayment` | Record payments |
| Vendors | `GET /v3/company/{id}/vendor` | Vendor list |
| Customers | `GET /v3/company/{id}/customer` | Customer list |
| Query | `GET /v3/company/{id}/query?query=` | SQL-like queries |

**Authentication:** OAuth 2.0
**Duplicate Detection Query:**
```sql
SELECT * FROM Bill WHERE VendorRef = '{vendor_id}'
  AND TotalAmt = '{amount}'
  AND TxnDate >= '{date_range_start}'
```

**n8n QBO Node:** Native node available — supports CRUD on invoices, bills, payments, vendors, customers.

### 4.4 n8n Workflow Automation

**What it is:** Open-source workflow automation tool (self-hosted or cloud). Acts as the orchestration layer connecting all systems.

**Available Nodes for This Integration:**

| Node | Purpose | Status |
|------|---------|--------|
| **QuickBooks** | Full QBO API access | Native node |
| **Microsoft Graph / HTTP Request** | Excel dashboard read/write via HTTP Request | Generic node (OAuth2) |
| **HTTP Request** | Procore API + SmoothX API | Generic node |
| **Gmail / SMTP** | Send invoices + PM notifications | Native node |
| **Webhook** | Receive Procore events | Native trigger |
| **Code** | Custom dedup logic, matching algorithms | JavaScript/Python |
| **IF / Switch** | Routing and decision logic | Built-in |
| **Merge** | Combine data from multiple sources | Built-in |
| **AI Agent** | Claude/GPT for intelligent matching | Native node |

**n8n AI Capabilities:**
- AI Agent node can call Claude API for fuzzy invoice matching
- Can use AI to normalize vendor names, invoice numbers
- Can generate follow-up email content dynamically

### 4.5 Microsoft Graph API / Excel in OneDrive

**What it is:** A shared Excel file hosted in OneDrive, accessed programmatically via the Microsoft Graph API. Serves as the approval dashboard and audit trail — PMs review and approve invoices directly in the Excel file, and n8n reads/writes to it via Graph API.
**Key API Endpoints:**

| API Resource | Endpoint | Purpose |
|-------------|----------|---------|
| Workbook Session | `POST /drives/{id}/items/{id}/workbook/createSession` | Create persistent session for batch operations |
| Table Rows | `POST /drives/{id}/items/{id}/workbook/tables/{name}/rows` | Add rows to Excel table |
| Table Rows (Read) | `GET /drives/{id}/items/{id}/workbook/tables/{name}/rows` | Read all rows from table |
| Range Read | `GET /drives/{id}/items/{id}/workbook/worksheets/{name}/range(address='{range}')` | Read specific cell range |
| Range Write | `PATCH /drives/{id}/items/{id}/workbook/worksheets/{name}/range(address='{range}')` | Write to specific cell range |
| Drive Items | `GET /drives/{id}/items/{id}` | Get file metadata, sharing links |
| Worksheets | `GET /drives/{id}/items/{id}/workbook/worksheets` | List all tabs/sheets |

**Authentication:** Azure AD OAuth 2.0 (app registration in Azure portal)
- Grant Type: Authorization Code or Client Credentials
- Scopes: `Files.ReadWrite`, `Sites.ReadWrite.All`
- Token URL: `https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token`

**Rate Limit:** 10,000 requests per 10 minutes per app per tenant

**n8n Connection Method:** Use **HTTP Request node** with **Generic OAuth2 credentials** pointed at Microsoft Graph API. No native Excel/OneDrive node required — the HTTP Request node handles all CRUD operations on the Excel file via REST.

---

## 5. System Architecture

> **Applies to:** Both options. The architecture below shows the full system. For Option 1, the Excel approval dashboard is included and SmoothX remains for Procore-QBO sync. For Option 2, SmoothX is removed and its sync responsibilities are absorbed by additional n8n workflows.

### High-Level Architecture

```
+------------------+     +------------------+     +------------------+
|                  |     |                  |     |                  |
|    PROCORE       |     |   QUICKBOOKS     |     |    SmoothX      |
|  (Project Mgmt) |     |   ONLINE (QBO)   |     |  (Sync Layer)   |
|                  |     |  (Accounting)    |     |                  |
+--------+---------+     +--------+---------+     +--------+---------+
         |                        |                        |
         | Webhooks + API         | OAuth API              | API/DB
         |                        |                        |
+--------v------------------------v------------------------v---------+
|                                                                     |
|                    n8n WORKFLOW ENGINE                               |
|                    (Orchestration Layer)                             |
|                                                                     |
|  +------------------+  +------------------+  +------------------+  |
|  | Invoice Intake   |  | Dedup Engine     |  | Matching Engine  |  |
|  | Workflow         |  | (AI-Powered)     |  | (Procore<>QBO)   |  |
|  +------------------+  +------------------+  +------------------+  |
|                                                                     |
|  +------------------+  +------------------+  +------------------+  |
|  | Customer Invoice |  | PM Notification  |  | Follow-Up        |  |
|  | Generator        |  | System           |  | Scheduler        |  |
|  +------------------+  +------------------+  +------------------+  |
|                                                                     |
+--------+------------------------+------------------------+----------+
         |                        |                        |
         v                        v                        v
+------------------+     +------------------+     +------------------+
|                  |     |                  |     |                  |
| EXCEL/ONEDRIVE   |     |     EMAIL        |     |   AI ENGINE     |
| (Dashboard)      |     | (Gmail/SMTP)     |     |  (Claude API)   |
|                  |     |                  |     |                  |
+------------------+     +------------------+     +------------------+
```

### Data Flow — Invoice Lifecycle

```
Step 1: INVOICE INTAKE
  Invoice arrives (email/Procore/manual upload)
       |
Step 2: NORMALIZE & FINGERPRINT
  n8n extracts: vendor, amount, invoice #, date, project
  AI normalizes vendor name + invoice number
  Creates unique fingerprint hash
       |
Step 3: DEDUPLICATION CHECK
  Check Excel tracking spreadsheet via Graph API: "Does this fingerprint exist?"
  Query QBO: "Matching bill for this vendor + amount + date range?"
  Query Procore: "Matching pay application?"
       |
  IF DUPLICATE FOUND:
    -> Flag in Excel as "DUPLICATE DETECTED"
    -> Alert PM via email
    -> STOP processing (do not create bill)
       |
  IF UNIQUE:
    -> Continue to Step 4
       |
Step 4: MATCH TO PROCORE COMMITMENT
  Find matching commitment in Procore by:
    - Vendor name match
    - Project match
    - Amount within commitment remaining
    - Cost code alignment
       |
Step 5: VERIFY IN QBO (READ-ONLY)
  Query QBO for existing bill (SmoothX creates vendor bills — n8n does NOT)
  Cross-reference Procore commitment with QBO bill record
  Confirm bill exists and amounts match
       |
Step 6: LOG IN EXCEL APPROVAL DASHBOARD
  Create entry in Excel tracking spreadsheet via Graph API:
    - Invoice #, Vendor, Amount, Project
    - Procore Commitment Ref
    - QBO Bill ID
    - Status: "Pending Approval"
    - Fingerprint hash
    - Timestamp
       |
Step 7: PM NOTIFICATION
  Email PM with:
    - Invoice details
    - Procore commitment match result
    - One-click approve/reject link
    - Dashboard link (Excel in OneDrive)
       |
Step 8: CUSTOMER INVOICE GENERATION
  Once approved:
    - Generate customer invoice in QBO
    - Email invoice to customer
    - Log in Excel with "Invoiced" status
       |
Step 9: FOLLOW-UP AUTOMATION
  If unpaid after X days:
    - Send automated reminder to customer
    - Notify PM of overdue status
    - Escalate if needed
```

---

## 6. n8n Workflow Design

### Workflow 1: Invoice Intake & Dedup Engine

```
TRIGGER: Webhook (Procore) / Email Trigger / Manual Upload
    |
    v
[Extract Invoice Data]
  - Parse email attachment (PDF)
  - Or receive Procore webhook payload
  - Extract: vendor_name, invoice_number, amount, date, project_name
    |
    v
[AI Normalization Node (Claude API)]
  - Normalize vendor name: "ABC Plumbing LLC" -> "abc_plumbing"
  - Normalize invoice #: "INV-001", "#001", "001" -> "001"
  - Extract line items if available
  - Generate fingerprint: SHA256(normalized_vendor + normalized_invoice_num + amount + project)
    |
    v
[Dedup Check - Excel Query via Graph API]
  - Read Excel invoice tracking spreadsheet: filter by fingerprint
  - If match found -> DUPLICATE PATH
    |
    v
[Dedup Check - QBO Query]
  - Query QBO Bills: WHERE vendor = X AND amount = Y AND date BETWEEN (invoice_date - 30, invoice_date + 30)
  - If match found -> DUPLICATE PATH
    |
    v
[Dedup Check - Procore Query]
  - Query Procore pay applications for same vendor + project
  - Check if already submitted/approved
    |
    v
[IF Node: Duplicate Detected?]
  |                          |
  YES                        NO
  |                          |
  v                          v
[Create Excel Entry:        [Continue to
 Status = "DUPLICATE"]       Matching Workflow]
  |
  v
[Email PM: "Duplicate
 Invoice Detected"
 with details]
```

### Workflow 2: Procore-QBO Matching Engine

```
TRIGGER: Receives clean (non-duplicate) invoice from Workflow 1
    |
    v
[Query Procore Commitments]
  - GET /rest/v1.1/projects/{project_id}/commitments
  - Filter by vendor
  - Get commitment line items + remaining amounts
    |
    v
[AI Matching Node (Claude API)]
  - Compare invoice to commitment:
    - Vendor match (fuzzy matching)
    - Amount <= remaining committed amount
    - Cost code alignment
    - Date within commitment period
  - Confidence score: HIGH / MEDIUM / LOW
    |
    v
[Switch Node: Match Confidence]
  |              |              |
  HIGH           MEDIUM         LOW/NONE
  (>90%)         (60-90%)       (<60%)
  |              |              |
  v              v              v
[Auto-Match]   [Flag for      [Flag for
               PM Review]     Manual Match]
    |
    v
[NOTE: n8n does NOT create bills in QBO — SmoothX owns that sync.
 n8n logs the match and notifies the PM.]
    |
    v
[Create Excel Entry via Graph API]
  - Invoice #, Vendor, Amount
  - Procore Commitment Ref
  - Match Confidence
  - Status: "Matched - Pending Approval"
  - Fingerprint hash
    |
    v
[Email PM]
  - Invoice matched to Procore commitment #{X}
  - Match confidence: HIGH
  - Review and approve: [Link to Excel dashboard in OneDrive]
```

### Workflow 3: Customer Invoice & Follow-Up

```
TRIGGER: n8n schedule (5-minute polling) - checks Excel for approved invoices
    |
    v
[Read Excel via Graph API: Decision = "APPROVE"]
    |
    v
[Generate Customer Invoice in QBO]
  - POST /v3/company/{id}/invoice
  - Map from Procore billing data
  - Include project details, line items
  - Apply markup/billing rates
    |
    v
[Send Invoice to Customer]
  - Email with PDF attachment
  - Or QBO native email
    |
    v
[Update Excel Entry via Graph API]
  - Status: "Invoiced to Customer"
  - Invoice sent date
  - QBO Invoice ID
    |
    v
[Email PM Confirmation]
  - "Invoice #{X} sent to {Customer} for ${Amount}"
  - Link to Excel dashboard in OneDrive
    |
    v
[Schedule Follow-Up]
  - Set timer: check in 30 days (or custom Net terms)
  - If unpaid -> trigger Workflow 4
```

### Workflow 4: Follow-Up & Escalation

```
TRIGGER: n8n Cron - runs daily
    |
    v
[Read Excel via Graph API: Status = "Invoiced" AND DaysSinceSent > NetTerms]
    |
    v
[For Each Overdue Invoice]
    |
    v
[Check QBO Payment Status]
  - Query QBO: has payment been received?
  - If paid -> update Excel to "Paid" -> SKIP
    |
    v
[Switch: Days Overdue]
  |              |              |
  1-15 days      16-30 days     31+ days
  |              |              |
  v              v              v
[Gentle         [Firm          [Escalation
 Reminder]       Reminder]      Alert]
  |              |              |
  v              v              v
[Email          [Email          [Email PM +
 Customer]       Customer +     Finance Team +
                 CC PM]         Flag Critical]
    |
    v
[Update Excel via Graph API]
  - Follow-up count + 1
  - Last follow-up date
  - Status: "Follow-Up Sent"
```

---

## 7. Excel Approval Dashboard Setup

> **Applies to:** Both options. The Excel approval dashboard is the core human approval gate for the entire system. Nothing hits QBO without PM sign-off in this spreadsheet.

### Overview

A shared Excel file hosted in OneDrive serves as the **approval dashboard and audit trail**. PMs open the familiar Excel interface to review pending invoices, mark decisions (APPROVE or HOLD), and track status. n8n reads and writes to this file via the Microsoft Graph API on a 5-minute polling interval.

Procore and QBO remain the systems of record for financial transactions; the Excel dashboard provides centralized visibility, human approval gating, and a complete audit trail.

### Excel File Structure

The Excel workbook contains 4 tabs (worksheets):

### Tab 1: Pending Approvals

| Column | Type | Purpose |
|--------|------|---------|
| Invoice # | Text | Unique invoice identifier |
| Vendor | Text | Normalized vendor name |
| Amount | Currency | Invoice amount |
| Project | Text | Project name |
| Procore Commitment | Text | Commitment ID from Procore |
| Match Confidence | Text | HIGH / MEDIUM / LOW / MANUAL |
| Decision | Text | **APPROVE** / **HOLD** (PM fills this in) |
| PM | Text | Responsible PM name |
| Timestamp | DateTime | When row was created by n8n |
| Fingerprint | Text | SHA256 dedup hash |
| QBO Bill ID | Text | Bill ID in QuickBooks |
| Entry Source | Text | Email / Procore / Manual / SmoothX |
| Notes | Text | PM notes, AI matching details |

**How it works:** n8n writes new rows to this tab when invoices are matched. PMs open the Excel file, review the rows, and type `APPROVE` or `HOLD` in the Decision column. n8n polls every 5 minutes, reads decisions, and processes approved invoices.

### Tab 2: Recently Processed

| Column | Type | Purpose |
|--------|------|---------|
| Invoice # | Text | Invoice identifier |
| Vendor | Text | Vendor name |
| Amount | Currency | Invoice amount |
| Project | Text | Project name |
| Status | Text | Approved / Rejected / Invoiced to Customer / Paid |
| Decision By | Text | PM who approved/rejected |
| Decision Date | DateTime | When decision was made |
| QBO Invoice ID | Text | Customer invoice ID (if generated) |
| Customer Invoiced Date | DateTime | When customer was invoiced |
| Payment Due | Date | Net terms deadline |
| Payment Received | Date | When customer paid |
| Follow-Up Count | Number | Number of reminders sent |

### Tab 3: Duplicate Alerts

| Column | Type | Purpose |
|--------|------|---------|
| Invoice # | Text | Flagged invoice identifier |
| Vendor | Text | Vendor name |
| Amount | Currency | Invoice amount |
| Project | Text | Project name |
| Match Type | Text | EXACT / FUZZY / AMOUNT_WINDOW / OVER_COMMITMENT / AI_FLAGGED |
| Confidence | Text | Percentage match confidence |
| Original Invoice # | Text | The invoice this is a duplicate of |
| Original Date | DateTime | When original was entered |
| Action | Text | BLOCKED / HELD_FOR_REVIEW |
| PM | Text | PM notified |
| Timestamp | DateTime | When duplicate was detected |

### Tab 4: Audit Log

| Column | Type | Purpose |
|--------|------|---------|
| Timestamp | DateTime | When action occurred |
| Action | Text | Created / Matched / Approved / Paid / Flagged / Duplicate Blocked |
| Invoice # | Text | Related invoice |
| Performed By | Text | System, PM name, or AI |
| Details | Text | Full action details |
| System | Text | n8n / QBO / Procore / Manual / AI |

### Microsoft Graph API Integration

n8n interacts with the Excel file using the Microsoft Graph API via HTTP Request nodes with OAuth2 credentials.

**Azure App Registration Required:**
1. Register app in Azure Active Directory
2. Grant `Files.ReadWrite` and `Sites.ReadWrite.All` permissions
3. Configure OAuth2 redirect URI for n8n
4. Store Client ID, Client Secret, and Tenant ID in n8n credentials

**Key Operations:**

```
READ pending approvals:
  GET /drives/{driveId}/items/{fileId}/workbook/tables/PendingApprovals/rows

WRITE new invoice row:
  POST /drives/{driveId}/items/{fileId}/workbook/tables/PendingApprovals/rows
  Body: { "values": [["INV-001", "ABC Plumbing", 15000, "Project Alpha", ...]] }

READ PM decisions (polling every 5 minutes):
  GET /drives/{driveId}/items/{fileId}/workbook/worksheets/PendingApprovals/range(address='G:G')
  -> Check for "APPROVE" or "HOLD" values

MOVE approved row to Recently Processed:
  POST /drives/{driveId}/items/{fileId}/workbook/tables/RecentlyProcessed/rows
  DELETE /drives/{driveId}/items/{fileId}/workbook/tables/PendingApprovals/rows/{index}

WRITE audit log entry:
  POST /drives/{driveId}/items/{fileId}/workbook/tables/AuditLog/rows
```

**Polling Interval:** n8n checks the Excel file every 5 minutes for new PM decisions. This balances responsiveness with Graph API rate limits (10,000 requests per 10 minutes).

---

## 8. Duplicate Payment Prevention Engine

### Multi-Layer Deduplication Strategy

```
Layer 1: FINGERPRINT MATCHING (Exact Match)
  SHA256(normalize(vendor) + normalize(invoice_num) + amount + normalize(project))
  -> Catches: exact same invoice entered twice

Layer 2: FUZZY MATCHING (Near Match)
  Compare: vendor similarity > 85%
           AND amount within 1%
           AND date within 30 days
  -> Catches: "ABC Plumbing" vs "ABC Plumbing LLC"
              "INV-001" vs "Invoice #001"

Layer 3: AMOUNT + VENDOR WINDOW (Pattern Match)
  Same vendor + same amount + within 45-day window
  -> Catches: monthly recurring bills that might be entered early

Layer 4: PROCORE CROSS-REFERENCE (Commitment Match)
  Total payments to vendor on commitment > committed amount
  -> Catches: overpayment beyond contract value

Layer 5: AI ANALYSIS (Intelligent Review)
  Claude API analyzes suspicious patterns:
  - Same vendor, same project, similar amounts, short time span
  - Multiple invoices from same vendor on same day
  - Invoice amount exceeds remaining commitment
  -> Catches: sophisticated duplicates, split payments, fraud patterns
```

### Dedup Decision Matrix

| Layer 1 | Layer 2 | Layer 3 | Layer 4 | Layer 5 | Action |
|---------|---------|---------|---------|---------|--------|
| MATCH | - | - | - | - | **BLOCK** — Exact duplicate |
| - | MATCH | - | - | - | **HOLD** — Flag for PM review |
| - | - | MATCH | - | - | **WARN** — Possible duplicate, proceed with caution |
| - | - | - | MATCH | - | **BLOCK** — Over-commitment payment |
| - | - | - | - | MATCH | **HOLD** — AI flagged, needs human review |
| CLEAR | CLEAR | CLEAR | CLEAR | CLEAR | **PASS** — Process normally |

### AI Prompt for Duplicate Detection (Claude API)

```
You are an accounts payable specialist for a construction company.
Analyze this invoice against existing records and determine if it
is a duplicate payment.

NEW INVOICE:
- Vendor: {vendor_name}
- Invoice #: {invoice_number}
- Amount: ${amount}
- Date: {date}
- Project: {project_name}

EXISTING RECORDS (last 90 days, same vendor):
{list_of_existing_invoices}

PROCORE COMMITMENT DATA:
- Commitment #: {commitment_id}
- Original Amount: ${committed_amount}
- Previously Billed: ${billed_to_date}
- Remaining: ${remaining_amount}

Respond with:
1. DUPLICATE_STATUS: EXACT_MATCH / LIKELY_DUPLICATE / POSSIBLE_DUPLICATE / UNIQUE
2. CONFIDENCE: 0-100%
3. REASONING: Brief explanation
4. RISK_LEVEL: CRITICAL / HIGH / MEDIUM / LOW
5. RECOMMENDATION: BLOCK / HOLD_FOR_REVIEW / PROCEED_WITH_CAUTION / PROCESS
```

---

## 9. API Integration Specifications

### 9.1 Procore OAuth Setup

```
Grant Type: Client Credentials (recommended for n8n automation)
Token URL: https://login.procore.com/oauth/token
Scope: N/A (Procore does NOT use OAuth scopes — permissions controlled via DMSA)
Required Header: Procore-Company-Id: {company_id}
```

**Required Procore API Calls:**

```javascript
// 1. Get all projects
GET https://api.procore.com/rest/v1.0/projects
Headers: { Authorization: "Bearer {token}" }

// 2. Get commitments for a project
GET https://api.procore.com/rest/v1.1/projects/{project_id}/commitments
Headers: { Authorization: "Bearer {token}" }

// 3. Get pay applications for a commitment
GET https://api.procore.com/rest/v1.0/projects/{project_id}/commitments/{commitment_id}/requisitions
Headers: { Authorization: "Bearer {token}" }

// 4. Get vendors
GET https://api.procore.com/rest/v1.0/companies/{company_id}/vendors
Headers: { Authorization: "Bearer {token}" }

// 5. Register webhook for new invoices
POST https://api.procore.com/rest/v1.0/companies/{company_id}/webhooks
Body: {
  "webhook": {
    "destination_url": "{n8n_webhook_url}/procore/invoice",
    "namespace": "invoice",
    "api_version": "v2"
  }
}
```

### 9.2 QBO API Integration

```javascript
// 1. Query existing bills for dedup
GET https://quickbooks.api.intuit.com/v3/company/{realmId}/query
  ?query=SELECT * FROM Bill WHERE VendorRef='{vendor_id}'
         AND TotalAmt='{amount}'
         AND TxnDate>='{start_date}'
Headers: { Authorization: "Bearer {token}" }

// 2. Query existing vendor bill (READ-ONLY — SmoothX creates bills, n8n does NOT)
// n8n uses this to cross-reference and verify bills created by SmoothX
GET https://quickbooks.api.intuit.com/v3/company/{realmId}/query
  ?query=SELECT * FROM Bill WHERE DocNumber='{invoice_number}'
         AND VendorRef='{vendor_id}'
Headers: { Authorization: "Bearer {token}" }
// NOTE: For Option 2 (Full Replacement), n8n replaces SmoothX and DOES create
// vendor bills. The POST /bill endpoint is used only in Option 2.

// 3. Create customer invoice
POST https://quickbooks.api.intuit.com/v3/company/{realmId}/invoice
Body: {
  "CustomerRef": { "value": "{customer_id}" },
  "Line": [{
    "Amount": {amount},
    "DetailType": "SalesItemLineDetail",
    "SalesItemLineDetail": {
      "ItemRef": { "value": "{item_id}" }
    },
    "Description": "Project: {project_name} - {description}"
  }],
  "BillEmail": { "Address": "{customer_email}" },
  "DocNumber": "{invoice_number}"
}

// 4. Check payment status
GET https://quickbooks.api.intuit.com/v3/company/{realmId}/query
  ?query=SELECT * FROM Payment WHERE CustomerRef='{customer_id}'
         AND TxnDate>='{invoice_date}'
```

### 9.3 Microsoft Graph API Integration (Excel/OneDrive)

```javascript
// Authentication: Azure AD OAuth 2.0
// Token URL: https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/token
// Scopes: Files.ReadWrite, Sites.ReadWrite.All
// Rate Limit: 10,000 requests per 10 minutes

// 1. Create invoice entry in Pending Approvals tab
POST https://graph.microsoft.com/v1.0/drives/{drive_id}/items/{file_id}/workbook/tables/PendingApprovals/rows
Headers: {
  "Authorization": "Bearer {graph_token}",
  "Content-Type": "application/json"
}
Body: {
  "values": [[
    "{invoice_number}",
    "{vendor_name}",
    {amount},
    "{project_name}",
    "{commitment_id}",
    "{confidence}",
    "",
    "{pm_name}",
    "{timestamp}",
    "{fingerprint_hash}",
    "{qbo_bill_id}",
    "{source}",
    ""
  ]]
}

// 2. Query for duplicate check — read Fingerprint column from all tabs
GET https://graph.microsoft.com/v1.0/drives/{drive_id}/items/{file_id}/workbook/worksheets/PendingApprovals/range(address='J:J')
Headers: {
  "Authorization": "Bearer {graph_token}"
}
// Also check RecentlyProcessed tab for previously processed fingerprints
GET https://graph.microsoft.com/v1.0/drives/{drive_id}/items/{file_id}/workbook/tables/RecentlyProcessed/rows
// n8n Code node filters for matching fingerprint

// 3. Read PM decisions (5-minute polling)
GET https://graph.microsoft.com/v1.0/drives/{drive_id}/items/{file_id}/workbook/tables/PendingApprovals/rows
Headers: {
  "Authorization": "Bearer {graph_token}"
}
// n8n Code node checks Decision column (G) for "APPROVE" or "HOLD"

// 4. Write audit log entry
POST https://graph.microsoft.com/v1.0/drives/{drive_id}/items/{file_id}/workbook/tables/AuditLog/rows
Headers: {
  "Authorization": "Bearer {graph_token}",
  "Content-Type": "application/json"
}
Body: {
  "values": [[
    "{timestamp}",
    "Approved",
    "{invoice_number}",
    "{pm_name}",
    "PM approved invoice via Excel dashboard",
    "n8n"
  ]]
}
```

### 9.4 n8n AI Agent Configuration

```javascript
// n8n Code Node: Invoice Fingerprint Generator
const crypto = require('crypto');

function normalizeVendor(name) {
  return name
    .toLowerCase()
    .replace(/\b(llc|inc|corp|ltd|co|company|enterprises?|services?|group)\b/gi, '')
    .replace(/[^a-z0-9]/g, '')
    .trim();
}

function normalizeInvoiceNum(num) {
  return num
    .replace(/^(inv|invoice|#|no|num|number)[-.\s:#]*/i, '')
    .replace(/^0+/, '')
    .trim();
}

function normalizeProject(project) {
  return project
    .toLowerCase()
    .replace(/[^a-z0-9]/g, '')
    .trim();
}

function generateFingerprint(vendor, invoiceNum, amount, project) {
  const normalized = `${normalizeVendor(vendor)}|${normalizeInvoiceNum(invoiceNum)}|${parseFloat(amount).toFixed(2)}|${normalizeProject(project)}`;
  return crypto.createHash('sha256').update(normalized).digest('hex');
}

// Apply to incoming invoice
const invoice = $input.first().json;
const fingerprint = generateFingerprint(
  invoice.vendor_name,
  invoice.invoice_number,
  invoice.amount,
  invoice.project_name
);

return [{
  json: {
    ...invoice,
    normalized_vendor: normalizeVendor(invoice.vendor_name),
    normalized_invoice_num: normalizeInvoiceNum(invoice.invoice_number),
    fingerprint: fingerprint
  }
}];
```

---

## 10. Implementation Roadmap

> **Note:** This roadmap covers the Option 1 build (10 weeks, $12,000). For Option 2, Phases 5-6 are added for SmoothX replacement (18 weeks total, $32,000). See the proposal (`PROPOSAL-FRONTLINE-PAYMENT-AUTOMATION.md`) for detailed per-option scope and pricing.

### Phase 1: Foundation (Weeks 1-2) — $3,000
- [ ] Set up n8n instance (cloud or self-hosted)
- [ ] Azure app registration for Microsoft Graph API (Excel/OneDrive access)
- [ ] Configure OAuth connections: Procore, QBO, Microsoft Graph
- [ ] Create Excel approval dashboard file in OneDrive with 4 tabs
- [ ] Configure Excel tables (Pending Approvals, Recently Processed, Duplicate Alerts, Audit Log)
- [ ] Test API connectivity for all systems
- [ ] Import existing vendor list and normalize names

### Phase 2: Approval Dashboard (Weeks 3-4) — $4,500
- [ ] Build n8n workflows for Excel read/write via Graph API
- [ ] Implement 5-minute polling for PM decisions
- [ ] Build PM notification emails with Excel dashboard links
- [ ] Configure human approval gate — nothing hits QBO without PM sign-off
- [ ] Build duplicate alert email templates
- [ ] Test approval flow end-to-end: write to Excel -> PM approves -> n8n reads decision

### Phase 3: Dedup Engine + Matching (Weeks 5-7) — $3,000
- [ ] Build Workflow 1: Invoice Intake & Dedup
- [ ] Implement 5-layer dedup logic
- [ ] Configure AI matching node (Claude API)
- [ ] Build Workflow 2: Procore-QBO Matching
- [ ] Implement commitment matching logic
- [ ] Test with historical duplicate invoices
- [ ] Tune matching thresholds based on results
- [ ] Configure confidence scoring thresholds

### Phase 4: Customer Invoicing & Rollout (Weeks 8-10) — $1,500
- [ ] Build Workflow 3: Customer Invoice Generation
- [ ] Build Workflow 4: Follow-Up & Escalation
- [ ] Configure email templates (invoice, reminders, escalation)
- [ ] Test end-to-end flow: receive -> match -> approve (Excel) -> invoice -> follow-up
- [ ] Connect SmoothX sync layer data feed
- [ ] PM training on Excel dashboard and approval workflows
- [ ] Pilot deployment on one project, then expand to all active projects

**Option 1 Total: $12,000 / 10 weeks**

### Phase 5: SmoothX Replacement — Bill Sync (Weeks 11-14) — $10,000 *(Option 2 only)*
- [ ] Build Procore→QBO bill sync workflows in n8n
- [ ] Build QBO→Procore payment sync workflows
- [ ] Implement contact/vendor sync
- [ ] Build cost code mapping
- [ ] Retention/retainage handling

### Phase 6: SmoothX Replacement — Payments, Retention & Progress (Weeks 15-18) — $10,000 *(Option 2 only)*
- [ ] Progress claims workflows
- [ ] Parallel run with SmoothX for validation
- [ ] SmoothX cutover and decommission
- [ ] Monitor and tune all workflows for 30 days

**Option 2 Total: $32,000 / 18 weeks**

### Estimated Monthly Run-Rate (Both Options)

| Service | Cost | Notes |
|---------|------|-------|
| n8n Cloud (Pro) | ~$50-100/mo | Or self-hosted for $0 |
| Claude API | ~$20-30/mo | For AI matching (~500 invoices) |
| Excel/OneDrive | Existing | Already included in Microsoft 365 |
| Procore | Existing | Already licensed |
| QBO | Existing | Already licensed |
| SmoothX | Existing | Already in budget *(Option 1 keeps; Option 2 eliminates)* |
| **New costs total** | **~$70-130/mo** | Does not include existing subscriptions |

### Year 1 Total Cost of Ownership

| | Option 1 | Option 2 |
|---|---------|---------|
| Build cost | $12,000 | $32,000 |
| Monthly run-rate (12 mo) | $840-$1,560 | $840-$1,560 |
| **Year 1 TCO** | **$12,840-$13,560** | **$32,840-$33,560** |
| | + existing SmoothX | SmoothX eliminated |

**ROI: Last year, Frontline lost $150,000 to duplicate payments. Even catching one $15,000 duplicate pays for the system's monthly run-rate for years. See proposal for full ROI analysis.**

---

## Appendix A: Email Templates

### Template 1: Duplicate Invoice Alert to PM

```
Subject: [ACTION REQUIRED] Duplicate Invoice Detected - {vendor_name} #{invoice_number}

Hi {pm_name},

Our automated system has detected a potential DUPLICATE invoice:

INCOMING INVOICE:
- Vendor: {vendor_name}
- Invoice #: {invoice_number}
- Amount: ${amount}
- Project: {project_name}

EXISTING RECORD:
- Original Entry Date: {original_date}
- Status: {original_status}
- QBO Bill ID: {qbo_bill_id}

Match Type: {match_type} (Confidence: {confidence}%)

ACTION NEEDED:
- If this IS a duplicate: No action needed, it has been blocked.
- If this is NOT a duplicate: Click here to review in the approval dashboard: {excel_dashboard_link}

Dashboard: {excel_dashboard_link}

This alert prevents double/triple payments. If you have questions,
contact the finance team.

— Frontline Payment Automation System
```

### Template 2: PM Invoice Approval Request

```
Subject: Invoice Ready for Review - {vendor_name} ${amount} - {project_name}

Hi {pm_name},

A new invoice has been matched to your Procore commitment and is ready for review:

INVOICE DETAILS:
- Vendor: {vendor_name}
- Invoice #: {invoice_number}
- Amount: ${amount}
- Date: {invoice_date}

PROCORE MATCH:
- Commitment: #{commitment_number}
- Original Committed: ${committed_amount}
- Previously Billed: ${billed_to_date}
- This Invoice: ${amount}
- Remaining After: ${remaining}
- Match Confidence: {confidence}

APPROVE or REJECT: Open the Excel approval dashboard and update the Decision column: {excel_dashboard_link}

This invoice will be held until you take action.

— Frontline Payment Automation System
```

### Template 3: Customer Invoice Email

```
Subject: Invoice #{invoice_number} from Frontline Holdings - {project_name}

Dear {customer_name},

Please find attached Invoice #{invoice_number} for ${amount} related to
{project_name}.

Invoice Details:
- Invoice #: {invoice_number}
- Amount: ${amount}
- Date: {invoice_date}
- Payment Terms: {net_terms}
- Due Date: {due_date}

Payment Methods:
{payment_instructions}

If you have questions about this invoice, please contact {pm_name}
at {pm_email}.

Thank you for your business.

Frontline Holdings
```

### Template 4: Payment Follow-Up (Gentle)

```
Subject: Friendly Reminder - Invoice #{invoice_number} Due {due_date}

Dear {customer_name},

This is a friendly reminder that Invoice #{invoice_number} for ${amount}
is due on {due_date}.

If payment has already been sent, please disregard this message.

Questions? Contact {pm_name} at {pm_email}.

Thank you,
Frontline Holdings
```

---

## Appendix B: SmoothX Integration Notes

SmoothX serves as Frontline's integration middleware (Procore↔QBO sync layer). Integration approach:

### If SmoothX Has REST API:
- Use n8n HTTP Request node to query job costing data
- Sync vendor records between SmoothX and Excel tracking spreadsheet
- Pull purchase order data for additional dedup layer

### If SmoothX Has Database Access Only:
- Use n8n database node (MySQL/PostgreSQL/MSSQL)
- Schedule sync every 15 minutes
- Read-only access recommended for safety

### If SmoothX Has File Export Only:
- Configure SmoothX to export CSV to shared folder
- n8n watches folder for new exports
- Parse and sync data on arrival

### Key SmoothX Data to Sync:
1. **Job Cost Records** — validates invoice amounts against budgets
2. **Purchase Orders** — additional matching layer for dedup
3. **Vendor Payment History** — historical dedup reference
4. **Budget vs. Actual** — prevents over-billing

---

## Appendix C: Security & Compliance

### Data Security
- All API tokens stored in n8n encrypted credentials store
- OAuth refresh tokens auto-rotated
- Excel/OneDrive file access restricted to finance + PMs via SharePoint permissions
- Audit log captures all system actions in Excel Audit Log tab

### Compliance
- SOX-compatible audit trail in Excel Audit Log tab
- Separation of duties: system matches, PM approves, finance pays
- All duplicate blocks logged with full reasoning
- AI decisions include confidence scores and are reviewable

### Backup & Recovery
- Excel/OneDrive data backed up via OneDrive version history and SharePoint retention policies
- n8n workflow definitions exported to Git
- QBO maintains its own backup (Intuit cloud)
- Procore maintains its own backup (Procore cloud)
