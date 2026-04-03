# OIG Exclusion Monitoring SaaS — Research & Opportunity Analysis

**Date:** 2026-04-02

---

## The Problem

Healthcare companies are **legally required** to screen every employee and vendor against the OIG List of Excluded Individuals/Entities (LEIE) before hiring or contracting with them. If they employ or contract with an excluded person — even unknowingly — the penalty is **$10,000–$50,000 per day per violation**, plus exclusion from Medicare and Medicaid reimbursement, which is an existential threat to most small practices.

The requirement applies to:
- Employees
- Contractors and vendors
- Medical directors
- Board members (in some interpretations)

The requirement is ongoing — not just at hire, but monthly or at minimum upon each OIG list update.

---

## The Data Source

**OIG LEIE** — published by the Department of Health and Human Services Office of Inspector General.

- Free public download: flat CSV/pipe-delimited file updated monthly
- Contains: full name, DOB, NPI (when available), exclusion type, exclusion date, reinstatement date if applicable, state, specialty
- No scraping required for V1 — it's a direct file download
- URL: `oig.hhs.gov/exclusions/exclusions_dlfile.asp`
- Approximately 75,000+ active exclusions in the database

**SAM.gov** — federal debarment list (System for Award Management)
- Free API available
- Covers federal contractors and grant recipients
- Less relevant for small healthcare practices, but worth adding as a checkbox feature

---

## Who Must Comply (Your Buyers)

| Segment | Size | Pain Level |
|---|---|---|
| Home health agencies | ~12,000 in US | Very high — CMS audits routinely |
| Behavioral health clinics | ~10,000+ | Very high — frequent CMS scrutiny |
| Dental practices (Medicaid-accepting) | ~50,000+ | High |
| DME suppliers | ~6,000 | Very high — high fraud history, heavily audited |
| Nursing homes / SNFs | ~15,000 | High — JCAHO + CMS requirements |
| Hospice providers | ~5,000 | High |
| Small physician groups | Large | Medium — varies by state |

**Best initial target:** Home health agencies and behavioral health clinics. They are the most heavily audited, have the highest awareness of the requirement, and are large enough to have an HR function but too small to afford enterprise tools.

---

## The Market Gap

**Enterprise tools** (too expensive for the target buyer):
- Compliancely — $500+/month
- VerifyAmerica — enterprise pricing
- Checkr (partial coverage) — per-screen pricing adds up
- symplr — large hospital systems only
- IntelliCheck — enterprise

**What small practices do today:**
- Manual monthly lookups on oig.hhs.gov (one by one)
- Spreadsheets tracking who was checked and when
- Often skipping it entirely and hoping they don't get audited
- Paying a billing service to do it as part of a bundle (but no audit trail)

**The gap:** No affordable, purpose-built SaaS for practices with 5–100 employees. This is the classic SMB underserved zone.

---

## V1 Product Spec

### Core Flow

```
Sign up → upload staff CSV → monthly auto-screen → receive PDF report → download for audit files
```

### Pages Required (~4 total)

1. **Landing page** — "Monthly OIG screening for small healthcare practices. $49/mo. Audit-ready PDF every month."
2. **Staff list** — Add/remove employees and vendors. Name, DOB, NPI (optional), role.
3. **Screening history** — List of past reports with download links. "June 2026 — 47 screened, 0 exclusions found."
4. **Report page / PDF** — Dated, signed-off report showing each person screened, the list version used, result. This is the compliance artifact.

### What Makes the PDF Critical

If CMS audits a home health agency and asks "show us your exclusion screening records," an email in their inbox is not acceptable documentation. They need:
- Dated report showing WHO was screened
- WHICH version of the OIG list was used (list is versioned by month)
- THE RESULT for each individual
- A downloadable, printable format they can put in a binder

This PDF is the actual product. The scraping/matching is the mechanism.

### Matching Logic (the "hard inference")

Name matching is non-trivial and is where the value lies vs. manual lookups:
- Fuzzy name matching to catch hyphenated names, maiden names, suffixes
- DOB cross-reference to reduce false positives
- NPI cross-reference when available (near-perfect match)
- Flag "possible match — please verify manually" vs. "confirmed match"
- Track match confidence score

A false positive that causes unnecessary alarm is bad. A false negative that misses an actual exclusion is catastrophic. The matching algorithm is where you invest.

---

## Pricing

| Plan | Price | Limit | Target |
|---|---|---|---|
| Starter | $49/mo | Up to 25 people | Solo practices, small clinics |
| Standard | $99/mo | Up to 100 people | Home health agencies, behavioral health |
| Growth | $199/mo | Up to 500 people | Multi-location practices |
| Custom | Contact | Unlimited | Large groups, MSOs |

Annual discount: 2 months free (reduces churn, improves cash flow).

**Justification is trivial:** One avoided fine day = years of subscription paid for. This is a compliance spend, not a discretionary spend.

---

## Moat — State Medicaid Exclusion Lists

The OIG list is just federal. Every state also maintains its own Medicaid exclusion list. Healthcare providers accepting state Medicaid must check both.

All 50 states have their own list. All have different formats. Many require scraping (no bulk download). Some are PDFs. Some are web tables. A few have search-only interfaces requiring headless browser scraping.

**This is the real moat:**
- Each state list added is a barrier competitors must replicate
- The work doesn't compound quickly — it's 50 separate engineering tasks
- But once done, it's extremely defensible
- Marketing angle: "The only tool that checks all 50 state Medicaid exclusion lists automatically"

**Priority order for state lists (by Medicaid spending / home health agency density):**
1. California
2. New York
3. Texas
4. Florida
5. Pennsylvania
6. Illinois
7. Ohio
8. Michigan

---

## Tech Stack

```
Bun + SQLite                  — backend + database
Bun.serve()                   — web server (no Express)
Bun cron / Bun.sleep loop     — monthly screening job
Resend                        — email delivery of reports
@react-pdf/renderer or pdfkit — PDF generation
Stripe                        — billing
```

No expensive infrastructure needed. A single VPS ($10–20/mo on Fly.io or Railway) handles hundreds of customers at this scale.

---

## Go-to-Market

**Where to find buyers:**
- r/homehealth, r/behavioralhealth subreddits
- Facebook Groups for home health agency owners (large, active)
- LinkedIn outreach to "Director of Compliance" and "Agency Administrator" titles
- HHA billing services (partner channel — they already talk to these agencies)
- State home health associations (HHA associations in each state)

**Pitch:**
> "Are you doing your monthly OIG exclusion screening manually? We automate it, generate the audit-ready PDF report, and cost less than 30 minutes of your time. $49/month."

**Content SEO angle:**
- "How to do OIG exclusion screening" (high intent, low competition)
- "OIG LEIE screening for home health agencies"
- "CMS exclusion screening requirements 2026"

---

## Risks

| Risk | Severity | Mitigation |
|---|---|---|
| OIG changes their file format | Low | Simple to adapt; they've been consistent for years |
| Competitor adds affordable SMB tier | Medium | State list coverage creates switching costs |
| False negative liability | High | Clear ToS: tool is an aid, not a legal guarantee; they must verify matches manually |
| Name matching accuracy | Medium | Conservative matching + manual review for "possible" matches |
| Low awareness of requirement | Low | They know they have to do it; they just don't have a good tool |

---

## Summary

This is the clearest path to first revenue among all the ideas researched. V1 requires:
- No paid APIs
- No proxies
- No complex scraping
- ~2 weeks to build
- $0 in infrastructure to start

The moat builds incrementally over months as you add state Medicaid lists. The PDF audit report is the core deliverable. Price it as a compliance spend (not a tool), target home health agencies and behavioral health clinics first.
