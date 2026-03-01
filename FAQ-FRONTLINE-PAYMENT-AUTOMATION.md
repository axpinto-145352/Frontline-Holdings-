# Frontline Holdings — Payment Automation FAQ
## Plain-Language Guide for the Frontline Team

**Prepared by:** Veteran Vectors (Anthony Pinto)
**Date:** March 1, 2026
**Companion to:** Proposal v3.0 + Architecture Document v3.0

---

## What Problem Are We Solving?

**Last year, Frontline lost $150,000 to duplicate payments.** Invoices come in through email, Procore, and physical mail. Different people enter the same invoice into QuickBooks at different times — and nobody catches it until the money is already gone. There's no system that checks "did we already pay this?" before the payment goes out.

On top of that, there's no approval step. Invoices go straight into QuickBooks without a PM signing off. No one is verifying that the invoice matches what was actually committed in Procore.

This system fixes both problems: it catches duplicates before they're paid, and it puts a human approval gate in front of every invoice.

---

## The Tools You Already Have — and Where They Fall Short

### Procore — Your Project Management Hub

**What it does well:**
- Tracks all your projects, commitments (subcontracts/POs), and change orders
- Stores vendor info, cost codes, and pay applications
- Gives PMs visibility into what's committed and what's been billed on each project

**Where it falls short:**
- Procore doesn't talk to QuickBooks on its own. It needs middleware (SmoothX) to sync data between the two
- Procore has no built-in duplicate invoice detection
- Procore doesn't generate customer invoices — that happens in QuickBooks
- There's no approval workflow that connects "PM reviewed this" to "OK, now pay it"

**Bottom line:** Procore is great at tracking what's happening on a project, but it can't prevent duplicate payments or automate the billing side of things.

---

### SmoothX — Your Current Sync Layer Between Procore and QuickBooks

**What it does well:**
- Syncs vendor bills, contacts, cost codes, and payment status between Procore and QuickBooks — automatically, in real time
- Handles retention/retainage on progress claims (this is complex and SmoothX has been doing it for years)
- Manages attachments — invoices travel with the bill record
- Absorbs API changes — when Procore or QuickBooks update their systems, SmoothX handles the adjustment so you don't have to

**Where it falls short:**
- **No duplicate detection.** SmoothX syncs whatever is in the system. If the same invoice is entered twice, SmoothX dutifully syncs both. It doesn't check for duplicates
- **No human approval gate.** Nothing in SmoothX requires a PM to review or approve an invoice before it hits QuickBooks
- **No AI matching.** SmoothX doesn't compare an incoming invoice against Procore commitments to see if it makes sense — it just syncs the data
- **No PM notifications.** SmoothX doesn't email your PMs to say "hey, this invoice landed, does it look right?"
- **No customer invoice generation.** SmoothX handles the vendor/AP side. It doesn't create customer invoices in QuickBooks or chase down unpaid ones
- **No follow-up automation.** When a customer hasn't paid, SmoothX doesn't send reminders. That's still manual
- **Vendor lock-in.** If SmoothX goes down, raises prices, or stops supporting a feature you need, you're stuck. You don't own any of the logic — it's their platform

**Bottom line:** SmoothX is excellent at syncing financial data between Procore and QuickBooks. But it does nothing to prevent the duplicate payment problem, doesn't require human approval, and doesn't help with customer-facing invoicing or collections.

---

### QuickBooks Online (QBO) — Your Accounting System

**What it does well:**
- Manages all your accounting: vendor bills, customer invoices, payments, general ledger
- Solid API — other tools can read from and write to it

**Where it falls short:**
- No native duplicate detection that works across systems. QuickBooks doesn't know what's in Procore
- No automated matching of invoices against Procore commitments
- No built-in approval workflow for vendor bills
- Generating customer invoices and chasing payments is still a manual process

**Bottom line:** QuickBooks is your books. It's not designed to be a workflow tool or a deduplication engine.

---

## So What's Missing?

Here's the gap in one picture:

```
WHAT YOU HAVE TODAY:

  Procore ←──SmoothX──→ QuickBooks
    (projects)    (sync)     (accounting)

    Invoice comes in → gets entered → gets paid
                       ↑
                       No check. No approval. No dedup.

WHAT THE NEW SYSTEM ADDS:

  Procore ←──SmoothX──→ QuickBooks
    (projects)    (sync)     (accounting)
         \                    /
          \                  /
           n8n (new layer)
           ├─ Catches duplicates (5 different ways)
           ├─ Matches invoices to Procore commitments
           ├─ Writes to Excel dashboard for PM approval
           ├─ PM types APPROVE or HOLD in Excel
           ├─ Only approved invoices move forward
           ├─ Generates customer invoices in QBO
           └─ Sends automated follow-ups on unpaid invoices
```

---

## What Is n8n?

n8n (pronounced "n-eight-n") is a workflow automation tool. Think of it as the traffic controller that sits between your existing systems and makes them work together in ways they can't on their own.

**What it's NOT:**
- It's not a new accounting system
- It's not replacing Procore or QuickBooks
- It's not another app your PMs need to learn
- It's not something your team interacts with directly — it runs in the background

**What it IS:**
- A behind-the-scenes engine that watches for invoices, checks for duplicates, matches them to commitments, writes results to your Excel dashboard, emails your PMs, creates customer invoices, and follows up on unpaid ones
- Open-source software — you own it, no vendor lock-in
- Already connects to QuickBooks natively and to Procore and Microsoft Graph (Excel/OneDrive) via standard API calls

**In construction terms:** n8n is like hiring a tireless back-office person who checks every single invoice against 5 different criteria before it even reaches a PM's desk — and then manages the entire billing and collections process after the PM approves it.

---

## What Is the Excel Approval Dashboard?

This is a shared Excel file that lives in your existing OneDrive (part of your Microsoft 365 subscription — no new software to buy).

**How it works for PMs:**
1. Open the Excel file (same as opening any spreadsheet)
2. See a list of invoices waiting for your review
3. Each row shows: vendor, amount, project, which Procore commitment it matched to, and how confident the system is in the match
4. Type **APPROVE** or **HOLD** in the Decision column
5. That's it. The system picks up your decision within 5 minutes and handles the rest

**Why Excel instead of a new app?**
- Your team already knows Excel. Zero learning curve
- Already included in your Microsoft 365 subscription. No new cost
- Works on desktop, laptop, phone, tablet — anywhere OneDrive works
- Multiple PMs can have it open at the same time

**What's in the spreadsheet (4 tabs):**
1. **Pending Approvals** — invoices waiting for PM decision
2. **Recently Processed** — invoices that have been approved, invoiced, or paid
3. **Duplicate Alerts** — invoices the system flagged as potential duplicates
4. **Audit Log** — complete history of every action the system took

---

## How Does It Catch Duplicates?

The system checks every incoming invoice 5 different ways before it ever reaches a PM:

| Check | What It Catches | Example |
|-------|----------------|---------|
| **1. Exact fingerprint** | Identical invoice entered twice | Same invoice # from same vendor for same amount on same project |
| **2. Fuzzy match** | Same invoice with slight name/number differences | "ABC Plumbing LLC" vs "ABC Plumbing" or "INV-001" vs "#001" |
| **3. Pattern match** | Same vendor + same dollar amount within 45 days | Recurring monthly bill entered early by a second person |
| **4. Procore cross-check** | Total payments exceeding what was committed | $80K paid against a $75K commitment — something's wrong |
| **5. AI analysis** | Suspicious patterns a human might miss | Same vendor, similar amounts, same project, short time span |

If any check flags an issue, the invoice is **blocked or held** — it doesn't move forward until a human reviews it.

---

## The Two Options — and When Each One Makes Sense

### Option 1 — Approval Gate + Gap-Filler ($12,000 / 10 weeks)

**What it is in one sentence:** n8n adds the missing approval step and duplicate protection while keeping SmoothX for the financial sync it already does well.

**This is the right choice if:**
- You want to solve the $150K duplicate payment problem as fast as possible
- You're generally happy with SmoothX for Procore-to-QuickBooks syncing
- You want lower risk — SmoothX keeps doing what it's been doing, n8n only adds what's missing
- You don't want to take on maintaining the Procore-QuickBooks sync yourself
- You want to be up and running in 10 weeks
- Budget priority is keeping the upfront cost down

**What you get:**
- Human approval gate — nothing hits QuickBooks without PM sign-off in Excel
- 5-layer duplicate detection catching duplicates before they become payments
- AI-powered matching that compares every invoice to Procore commitments
- PM email notifications with match details and confidence scores
- Automated customer invoicing through QuickBooks
- 3-tier follow-up system for overdue customer invoices (gentle reminder → firm reminder → escalation)
- Full audit trail in the Excel log tab

**What you keep paying:**
- SmoothX subscription continues as-is (your existing cost)
- ~$70-130/month for n8n cloud hosting + AI matching

**Year 1 total cost:** $12,840-$13,560 (plus your existing SmoothX subscription)

---

### Option 2 — Full Replacement ($32,000 / 18 weeks)

**What it is in one sentence:** Everything in Option 1, plus n8n completely replaces SmoothX — you own the entire integration stack.

**This is the right choice if:**
- Eliminating the SmoothX dependency is a strategic priority for your business
- You want full control over every data flow between Procore and QuickBooks
- You're comfortable with a higher upfront investment that pays off over time through SmoothX subscription savings
- You have the appetite for a slightly higher maintenance commitment (or plan to use a retainer for that)
- You want zero third-party dependency — if something breaks, your team (or your vendor) can fix it directly without waiting on SmoothX support

**What you get (in addition to everything in Option 1):**
- Procore-to-QuickBooks vendor bill sync (replaces SmoothX)
- QuickBooks-to-Procore payment status sync (replaces SmoothX)
- Contact and vendor sync between systems (replaces SmoothX)
- Cost code mapping (replaces SmoothX)
- Retention/retainage handling (replaces SmoothX)
- Progress claim workflows (replaces SmoothX)
- SmoothX subscription cancelled — that monthly cost goes away

**What you stop paying:**
- SmoothX subscription eliminated (saves ~$2,400-6,000/year depending on your plan)

**Year 1 total cost:** $32,840-$33,560 (no SmoothX subscription)

**Important caveats:**
- Retention and progress claim logic is complex. SmoothX has been refining this for years. Custom-building it means a 4-week parallel run where both systems run side-by-side to make sure nothing is off
- Without SmoothX absorbing API changes from Procore and QuickBooks, someone needs to monitor and respond to those updates. A monthly retainer (Standard or Premium tier) is strongly recommended
- Higher risk in the first 6 months while the custom-built sync is proven out in production

---

## Side-by-Side Comparison

| Question | Option 1 | Option 2 |
|----------|----------|----------|
| Does it stop duplicate payments? | Yes | Yes |
| Does it require PM approval before payment? | Yes | Yes |
| Does it catch duplicates 5 different ways? | Yes | Yes |
| Does it use AI to match invoices? | Yes | Yes |
| Does it email PMs automatically? | Yes | Yes |
| Does it generate customer invoices? | Yes | Yes |
| Does it follow up on unpaid invoices? | Yes | Yes |
| Does it use the Excel dashboard? | Yes | Yes |
| Do you keep SmoothX? | Yes | No — replaced by n8n |
| How much upfront? | $12,000 | $32,000 |
| How long to build? | 10 weeks | 18 weeks |
| Monthly run-rate (new costs)? | ~$70-130/mo | ~$70-130/mo |
| Risk level? | Low | Medium-High (first 6 months) |
| Ongoing maintenance? | ~1 hr/week | ~2-4 hrs/week |

---

## Common Questions

### "Why can't SmoothX just add these features?"

SmoothX is an integration middleware — it syncs data between systems. Duplicate detection, AI matching, human approval gates, customer invoicing, and collections automation are fundamentally different capabilities. It would be like asking your electrician to also do your plumbing. SmoothX does its job well; it's just a different job than what's needed here.

### "Why not just be more careful with data entry?"

That's the approach today, and it resulted in $150,000 in losses last year. The problem isn't carelessness — it's that invoices arrive through multiple channels (email, Procore, mail) and are entered by multiple people who don't know what the other has already entered. No amount of "being careful" solves a systemic multi-channel, multi-person timing problem. You need a system that checks automatically.

### "Do our PMs need to learn new software?"

No. The approval dashboard is an Excel file. PMs open it, see the invoices assigned to them, and type APPROVE or HOLD. If your team can use a spreadsheet, they can use this system. Training is a 30-minute walkthrough.

### "What happens if the system goes down?"

Your existing workflow continues to work — Procore, QuickBooks, and SmoothX (Option 1) are unaffected. The automation layer (n8n) is additive. If it goes down temporarily, invoices just aren't auto-checked or auto-routed until it comes back up. No data is lost. n8n Cloud has built-in monitoring and alerting so outages are caught and addressed quickly.

### "How is this different from what Procore or QuickBooks could build natively?"

Procore's built-in QBO connector is limited — it only works for new projects, doesn't handle pre-existing project data without paid professional services, supports only one QBO company, and has no duplicate detection or approval workflow. QuickBooks has no concept of Procore commitments at all. Neither system was designed to do what this automation layer does.

### "What if we start with Option 1 and decide we want Option 2 later?"

That's a perfectly valid path. Option 1 is self-contained — it delivers full value on its own. If you later decide you want to eliminate SmoothX, Phases 5 and 6 can be added on. The foundation built in Option 1 (n8n instance, API connections, Excel dashboard, dedup engine) carries forward. You wouldn't be rebuilding anything.

### "What's the ROI?"

Based on the confirmed $150,000 in duplicate payments last year:

| | Option 1 | Option 2 |
|---|---------|---------|
| **Year 1 cost** | ~$13,000 | ~$33,000 |
| **Year 1 savings (dedup alone)** | $120,000-$142,500 (80-95% reduction) | $120,000-$142,500 |
| **Additional value** (faster collections, PM time savings) | $15,000-$30,000 | $15,000-$30,000 + SmoothX savings |
| **Approximate ROI** | ~13x return | ~5x return (Year 1) |

Option 1 pays for itself within the first month. Option 2 has a longer payback but eliminates an ongoing vendor dependency.

### "What does our team actually need to do during the build?"

| Who | What | When |
|-----|------|------|
| Executive sponsor | Greenlight the project, be available for key decisions | Kickoff |
| Procore admin | Register the Procore Developer App (takes ~15 minutes) | Week 1 |
| IT / admin | Set up Azure app registration for Microsoft Graph API | Week 1 |
| Designated PM(s) | Identified as approvers for the Excel dashboard | Before Phase 2 |
| Finance team | Available for parallel run verification | Weeks 8-10 |
| All PMs | 30-minute training session | Before rollout |
| Anyone | Provide historical duplicate invoice examples for testing | By Week 5 |

### "What does ~$70-130/month cover?"

| Service | Cost | What It Is |
|---------|------|-----------|
| n8n Cloud (Pro plan) | ~$50-100/mo | The automation engine that runs all the workflows. Like paying for a cloud server |
| Claude API | ~$20-30/mo | The AI that does fuzzy matching and intelligent duplicate detection. Pay-per-use based on ~500 invoices/month |
| Excel/OneDrive | $0 (existing) | Already part of your Microsoft 365 subscription |
| Procore | $0 (existing) | Already licensed |
| QuickBooks | $0 (existing) | Already licensed |

No Notion subscription. No new platforms. No per-user fees.

### "Can we self-host n8n instead of using the cloud version?"

Yes. n8n is open-source. Self-hosting reduces the monthly cost to near-zero for the n8n piece (you'd still pay for Claude API). The trade-off is that your team (or IT provider) would manage the server, updates, and uptime. Most companies start with n8n Cloud and move to self-hosted later if they want to.

### "What happens after the project is done? Do we need ongoing support?"

Two weeks of post-launch support are included in the project fee at no extra charge. After that, an optional monthly retainer is available:

| Tier | Hours/Month | Monthly Cost | Best For |
|------|-------------|-------------|----------|
| Essential | Up to 5 hrs | $625/mo | Bug fixes, threshold tweaks, API updates |
| Standard | Up to 10 hrs | $1,250/mo | Above + new features, monthly review call |
| Premium | Up to 20 hrs | $2,500/mo | Above + proactive monitoring, quarterly audits, on-call |

For Option 1, the Essential tier is usually sufficient. For Option 2, Standard or Premium is strongly recommended because you're taking on the Procore-QuickBooks sync maintenance that SmoothX currently handles.

---

## The Bottom Line

**The core problem:** Invoices arrive through multiple channels, get entered by multiple people, and nobody catches duplicates before payments go out. There's no approval gate and no automated way to match invoices to what's actually committed in Procore.

**The solution:** An automation layer (n8n) that sits between your existing systems, catches duplicates 5 different ways, requires PM approval in a simple Excel spreadsheet before anything is paid, matches invoices to Procore commitments using AI, and automates customer invoicing and collections.

**Option 1** ($12,000 / 10 weeks) adds what's missing while keeping SmoothX for what it does well. Low risk, fast deployment, immediate ROI.

**Option 2** ($32,000 / 18 weeks) does everything in Option 1 plus replaces SmoothX entirely. Higher upfront cost, more control, no vendor dependency.

Both options solve the $150,000 duplicate payment problem. Both give your PMs a simple Excel-based approval workflow. Both automate customer invoicing and follow-up. The difference is whether you keep SmoothX or replace it.

**Next step:** Schedule the free audit call so we can walk through your actual Procore setup and confirm everything before we start.

---

*Veteran Vectors — Building systems that work as hard as you do.*
