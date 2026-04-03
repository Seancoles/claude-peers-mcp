# Contractor License Status Monitoring SaaS — Research & Opportunity Analysis

**Date:** 2026-04-02  
**Starter states:** California, New Jersey

---

## The Problem

General contractors, property managers, banks, and insurance companies are **legally and financially liable** when they hire unlicensed or lapsed subcontractors. The consequences:

- **GCs:** Can be held jointly liable for injuries or property damage caused by an unlicensed sub. In California, hiring an unlicensed contractor can void the GC's own license.
- **Property managers:** Face owner lawsuits and insurance denials if work was done by an unlicensed contractor.
- **Banks (construction loans):** Lien rights can be invalidated. Draw schedules require licensed contractors.
- **Insurance companies:** Policies exclude work done by unlicensed contractors. Adjusters must verify before approving claims.

A GC with 50–200 active subcontractors across 2–5 states **cannot manually check every license every month.** The problem grows with each new project and each new sub relationship.

---

## The Data Sources

### California — CSLB (Contractors State License Board)

**Website:** `cslb.ca.gov`

California has the largest contractor licensing database in the US, with over **300,000 active licensees**.

**What's available:**
- Individual license lookup: `cslb.ca.gov/OnlineServices/CheckLicenseII/CheckLicense.aspx`
- **Bulk data download available** — CSLB publishes a downloadable data file (updated regularly)
- Data includes: license number, business name, DBA, license type, status, expiration date, bond info, workers' comp insurance status, personnel list, disciplinary history

**License types:**
- Class A — General Engineering Contractor
- Class B — General Building Contractor
- Class C — Specialty Contractors (C-10 Electrical, C-20 HVAC, C-36 Plumbing, C-39 Roofing, etc. — 44 specialty classifications)

**Status values to monitor:**
- Active ✓
- Inactive — license holder chose not to renew
- Expired — missed renewal deadline
- Suspended — bond/insurance lapse, citation, or disciplinary action
- Revoked — serious violation
- Cancelled — voluntarily surrendered

**Bond and insurance are tracked separately** — a license can be "Active" but bond-lapsed, meaning the contractor is technically unlicensed for that period. This is a critical nuance most manual checkers miss.

**Scraping approach:**
- Start with bulk data file download (no scraping needed for initial load)
- Delta updates: check individual records for any flagged changes on a schedule
- Alternatively: poll the individual lookup by license number (rate limiting will apply)

---

### New Jersey — Division of Consumer Affairs

**Website:** `njconsumeraffairs.gov`

New Jersey has a broad contractor registration/licensing ecosystem managed by the Division of Consumer Affairs.

**Key license types:**

| License Type | Issuing Body | Who Needs It |
|---|---|---|
| Home Improvement Contractor (HIC) | NJ DCA | **Any** contractor doing home improvement work — mandatory |
| Electrical Contractor | NJ Board of Examiners of Electrical Contractors | Electrical work |
| Plumbing Contractor | NJ State Board of Examiners of Master Plumbers | Plumbing |
| HVACR Contractor | NJ Board of Examiners of Heating, Ventilating, Cooling | HVAC |
| Master Electrician | Same as above | Individual credential |

**The HIC registration is particularly valuable:**
- Every contractor doing residential home improvement work must register, regardless of trade
- It's a flat-fee registration (not a skills-based exam), so coverage is very broad
- Many homeowners, property managers, and GCs don't check HIC status before hiring
- Non-registered contractors doing HIC work face: fines up to $10,000 per violation, inability to sue for payment, and consumers can void contracts

**Lookup:**
- `njconsumeraffairs.gov/licensee-search` — individual lookup by name or license number
- No bulk download publicly available (as of 2026) — requires scraping

**Status values:**
- Active
- Inactive
- Expired
- Revoked
- Suspended
- Pending Renewal

**Scraping approach:**
- Scrape the licensee search results (HTML table, straightforward)
- Requires session management — the site uses form-based searches
- Rate limiting: respectful polling with delays

---

## Who Pays (Buyer Profiles)

### 1. General Contractors *(Primary buyer)*
- Medium-to-large GCs managing 50–300 active subs
- Already pay for risk management tools (COI tracking, Procore, etc.)
- Clear ROI: one lawsuit from an unlicensed sub = $50K–$500K+ exposure
- Price sensitivity: low if framed as risk management
- **WTP: $149–499/month**

### 2. Property Management Companies
- Managing residential or commercial portfolios
- Hire contractors constantly for maintenance, renovation, turnover work
- Owners hold them accountable for contractor vetting
- **WTP: $99–299/month**

### 3. Insurance Companies / Adjusters
- Must verify contractor license before approving claims for contractor-performed work
- Currently done manually per claim — slow and error-prone
- A tool that lets adjusters batch-verify 20 contractors before a site visit is high value
- **WTP: Enterprise / per-lookup pricing**

### 4. Construction Lenders / Banks
- Draw schedules require licensed contractors
- Currently done by title companies and inspectors manually
- Opportunity: integrate into their existing loan management workflow
- **WTP: Enterprise**

### 5. Real Estate Investors / Flippers
- Smaller buyers, lower WTP, but large volume in CA and NJ
- Casual use — not recurring monitoring, more on-demand verification
- Better served by a freemium tier or one-time report

---

## V1 Product Spec (CA + NJ Focus)

### Core Flow

```
Sign up → enter your subs (name + license number) → we monitor status weekly → alert when anything changes
```

### Pages Required (~5 total)

1. **Landing page** — "Stop manually checking contractor licenses. We monitor your subs 24/7 and alert you the moment a license lapses, is suspended, or expires."
2. **Sub list** — Add contractors: name, license number, state, license type. Import via CSV.
3. **Dashboard** — Status overview. Green/yellow/red per contractor. Filter by state, status, expiration date.
4. **Alerts** — Configure: email when license expires, suspends, or bond lapses. Set lead time for expiration warnings (e.g., 30 days before expiry).
5. **Contractor detail** — Full license record, status history, bond status, workers' comp status. "Last verified: 2026-04-01."

### Key UX Insight

The most valuable alert is **proactive expiration warning** — "Sub X's license expires in 28 days." This gives the GC time to require renewal before the next project starts, rather than discovering the issue mid-job.

---

## The "Hard Inference" Layer

Raw data from CSLB or NJ DCA is a status field. What customers actually need is **risk signals**:

| Raw Data | Hard Inference |
|---|---|
| Status: Expired | "License expired 14 days ago. Do not use on active projects." |
| Bond: Lapsed | "License is Active but bond has lapsed — contractor is not legally covered for this period." |
| Expiration: 30 days out | "License expires 2026-05-01. Request renewal documentation now." |
| Disciplinary history: 2 citations in 3 years | "Pattern of citations. High risk flag." |
| Status changed: Active → Suspended | "ALERT: License suspended as of 2026-04-01. Reason: citation. Immediate action required." |

The risk scoring + change detection is the moat — not just showing the current status.

---

## California Deep Dive

**Why California first:**
- Largest contractor market in the US by a wide margin
- CSLB bulk data = faster to build (no scraping needed for initial population)
- GCs in CA are acutely aware of license requirements due to CSLB's active enforcement
- CSLB's public contractor check tool is widely used — familiarity = easier sell

**CSLB Bulk Data:**
- Download URL: `cslb.ca.gov/About_Us/Library/Licensing_Statistics/Contractors_Data_Files.aspx`
- Format: pipe-delimited flat file
- Frequency: updated regularly (check for weekly or monthly cadence)
- Includes: all licensees, all statuses, bond info, personnel, disciplinary records

**Initial build plan:**
1. Download CSLB bulk file on a schedule
2. Load into SQLite: license_number, name, dba, status, expiration_date, bond_status, wc_status, last_updated
3. Diff against previous snapshot to detect changes
4. Alert customers who monitor any changed license

**Key CA nuance:** A license can show "Active" on CSLB's site but have a **personnel disassociation** — the Responsible Managing Employee (RME) or Responsible Managing Officer (RMO) has separated from the company, giving the company 90 days to replace them or the license becomes inactive. This is a critical signal that most tools miss.

---

## New Jersey Deep Dive

**Why NJ second:**
- Dense population, high construction activity
- HIC registration covers a very broad universe of contractors (not just specialty trades)
- NJ has historically high contractor fraud complaints — buyers are motivated
- Proximity to NY metro real estate market = higher-value projects = higher WTP

**NJ DCA Scraping:**
- Licensee search: form-based, returns HTML table
- Fields available: license number, name, license type, status, expiration date, address
- No bulk download → requires building a proper scraper with session management
- Rate limit respectfully: 1 request per 2–3 seconds is safe

**Initial build plan for NJ:**
1. Build scraper for `njconsumeraffairs.gov/licensee-search`
2. Parameterize by license number (most targeted) or name search
3. Schedule weekly polling for each monitored contractor
4. Detect status changes and trigger alerts

**Key NJ nuance:** NJ HIC registration requires a registered business address and a $500,000 general liability insurance certificate. When HIC registration lapses, the insurance certificate becomes irrelevant — meaning the contractor may not have active coverage even if they claim to be insured. This is a risk signal worth surfacing explicitly.

---

## Pricing

| Plan | Price | Subs Monitored | Target |
|---|---|---|---|
| Starter | $99/mo | Up to 25 | Small GC, single property manager |
| Professional | $249/mo | Up to 100 | Mid-size GC, property management company |
| Business | $499/mo | Up to 500 | Large GC, multi-state developer |
| Enterprise | Custom | Unlimited + API | Insurance companies, banks |

**Annual: 2 months free.** Meaningful discount for the buyer, better LTV for you.

---

## Moat — Adding States Over Time

Each state is a separate engineering task. Priority order after CA + NJ:

| State | Why | Difficulty |
|---|---|---|
| Texas (TDLR) | Huge construction market, bulk download available | Low |
| Florida (DBPR) | Large market, active enforcement | Medium |
| New York | NYC market, high WTP | Medium-High |
| Illinois | Chicago market | Medium |
| Arizona | Fast-growing construction market | Low |
| Colorado | Fast-growing, tech-savvy buyers | Low |

After 5–6 states, you're covering the majority of US construction activity. The multi-state monitoring capability (one dashboard for subs operating across state lines) becomes a meaningful differentiator — national GCs and developers will pay well for this.

---

## Tech Stack

```
Bun + SQLite                  — backend + database
Bun.serve()                   — web server
Bun cron                      — scheduled polling jobs
node-fetch / Bun native fetch — HTTP requests for scraping
cheerio                       — HTML parsing for NJ DCA
Resend                        — alert emails
Stripe                        — billing
```

For CA: bulk file processing, no headless browser needed.
For NJ: cheerio for simple HTML parsing is enough — no Playwright needed for the licensee search.

Infrastructure: single VPS on Fly.io (~$10–20/mo) handles hundreds of customers.

---

## Go-to-Market

**Where to find buyers:**
- Associated General Contractors (AGC) — has state chapters, email lists, events
- NARI (National Association of the Remodeling Industry) — property managers and remodelers
- CAI (Community Associations Institute) — HOA property managers
- BiggerPockets forums — real estate investors doing rehabs
- LinkedIn: "Project Manager," "Superintendent," "VP Construction" at mid-size GCs
- NJ and CA contractor associations (CAGC, ABC chapters)

**Cold outreach angle:**
> "Do you manually verify your subs' contractor licenses before each project? We automate that — weekly monitoring across CA and NJ, with instant alerts when a license lapses or is suspended. Takes 5 minutes to set up. $99/month."

**Content SEO angle:**
- "California contractor license lookup bulk"
- "How to verify contractor license NJ"
- "CSLB license status monitoring"
- "What happens if your sub has an expired license"

---

## Your Edge (Given Your Background)

Since you have experience in this space, you know:
- Which license types actually matter in practice vs. which are administrative noise
- What GCs actually look at when vetting a sub (it's not just status — bond, WC, and disciplinary history matter)
- The workflow pain: where in the project lifecycle license verification happens and where it breaks down
- What a salesperson at this kind of tool needs to say to a superintendent vs. a CFO

That domain knowledge = better product decisions on day one, faster credibility with early customers, and better marketing copy. This is a meaningful advantage over a pure-tech builder who has to discover all of this through user interviews.

---

## Risks

| Risk | Severity | Mitigation |
|---|---|---|
| CSLB changes bulk file format | Low | They've been consistent; adapt quickly when it happens |
| NJ DCA blocks scraping | Medium | Respectful rate limiting; fallback to individual lookups via headless browser |
| False negative (missed status change) | High | Conservative polling frequency (weekly minimum); clear ToS on limitation |
| Competitor adds affordable tier | Medium | State coverage + change history database creates switching costs |
| Sales cycle too long at enterprise | Low | Start with SMB GCs who can decide in a day |

---

## Summary

This is the **higher-ceiling opportunity** vs. OIG exclusion monitoring. The price points are higher, the market is larger, and your existing domain knowledge is a real advantage. The trade-off is it takes longer to build enough state coverage to feel complete.

**Recommended sequencing:**
1. Build CA first — bulk file download, fastest to V1
2. Add NJ second — scraper, your second market
3. Charge from day one — $99/month starter tier, sell to 5–10 early customers in your network
4. Use early revenue + feedback to prioritize next states

The multi-state monitoring dashboard — one place to see all your subs' license statuses across CA and NJ — is already a differentiated product. Nothing affordable does this today.
