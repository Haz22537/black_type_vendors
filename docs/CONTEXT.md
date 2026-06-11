# Project Context — Rogers Bloodstock Group Winner Analysis

> Read this before starting any task. It contains decisions already made, constraints to respect, and background that will prevent wasted work.

---

## Client Brief
Rogers Bloodstock wants a report on Australian Group race winners over the last ~3 years, focused on identifying which stud farm vendors are producing champions. The client has a hunch that some farms are underperforming.

---

## The Data Sources

### 1. Group Race List (`data/raw/group_race_list.csv`)
- CSV of all Group 1/2/3 races in Australia from ~mid 2023 to present
- Key columns: `DATE`, `Grp` (numeric), `Grp` (text: G1/G2/G3/L), `State`, `Club`, `Venue` (abbreviated code), `Race Name`, `Dist.`, `Prizemoney`
- DATE format: `3-Aug-24` style
- Venue is an abbreviation code (e.g. FLEM, RAND, CAUL, MORP, BLMT, HAWK)
- See `SAMPLES/group_race_list_sample.csv` for structure

### 2. Racing.com Results Pages
- URL pattern: `https://www.racing.com/form/YYYY-MM-DD/venue-slug/`
- Examples:
  - `https://www.racing.com/form/2026-06-06/flemington/`
  - `https://www.racing.com/form/2026-06-06/royal-randwick/`
  - `https://www.racing.com/form/2026-06-06/eagle-farm/`
- Venue slug is a lowercase hyphenated version of the venue name
- Venue abbreviation → slug mapping needs to be built and maintained in `docs/EDGE_CASES.md`
- See `SAMPLES/meeting_allresults_urls_racingcom.csv` for known mappings

### 3. Racing.com Horse Profile Pages
- URL pattern: `https://www.racing.com/horses/horse-name-XXXXXXX`
- The numeric suffix is UNPREDICTABLE — it cannot be constructed
- The href MUST be harvested from the results page scrape
- See `SAMPLES/horseprofile_urls_racingcom.csv` for example
- Profile pages contain: Sire, Dam, Date of Birth

### 4. Inglis Sales Data (`data/raw/inglis_sales.csv`)
- Scraped sales results from Inglis covering 8+ years
- Includes: yearling, weanling, ready-to-run (R2R), and broodmare sales
- Key columns: `Lot`, `Vendor`, `Sire`, `Dam`, `Dam Sire`, `DOB`, `Price`, `Status`, `Sex`, `Colour`
- Horse `Name` column is almost always blank for yearlings/weanlings (unnamed at sale) — this is expected and normal
- Vendor format: `"Stud Name, Location"` e.g. `"Vinery Stud, Scone"` — use as-is
- Price includes strings like `"Passed In - Reserve: $60,000"` and blanks for withdrawn — handle in cleaning
- See `SAMPLES/inglis_saleresults_sample.csv` for structure

---

## The Join Key — CRITICAL

**Dam Name + Year of Birth (YOB)**

This is how race winners connect to sales records. Understand why:
- Yearlings and weanlings are almost always UNNAMED at time of sale
- A dam produces a MAXIMUM of one foal per year
- Therefore Dam + YOB uniquely identifies a horse across sales and race records
- DOB is in sales data → derive YOB from DOB
- YOB from racing profile → use directly

### Southern Hemisphere YOB Convention
Australian thoroughbreds use **1 August** as the universal birthday. A horse born before 1 August is a year "older" in racing terms. When calculating YOB from DOB, apply this if needed to ensure the key matches correctly across datasets.

---

## Scraping Architecture — INVESTIGATE FIRST

**⚠️ Do not assume the scraping approach. Investigate before building.**

Racing.com appears to be a React/Next.js application based on CSS class naming conventions. This means:
- A standard `requests` fetch may return an empty JS shell with no race data
- Horse profile hrefs may only appear after JavaScript execution
- Playwright may be required instead of requests + BeautifulSoup

### Investigation Required (Script 2 prerequisite)
Before writing the race scraper, test:
1. Fetch a known meeting URL with `requests` — does it return populated data?
2. Is there a discoverable underlying API (check page source for XHR endpoints)?
3. Do horse `<a>` tags carry `href` attributes in the static response?

The HTML sample in `SAMPLES/racingcom_meeting_part_allresultspage.txt` shows a horse name in an `<a>` tag with NO visible `href`. This is the key thing to resolve.

**Outcome determines architecture:**
- Static → `requests` + `BeautifulSoup`
- JS-rendered → Playwright
- API discoverable → direct API calls (cleanest option if available)

---

## What To Scrape From Each Meeting Page

For every race on a Group race day, capture:
- Date
- Venue
- Race number
- Race name
- Race conditions (the full condition string e.g. "BM78", "WFA", "Hcp")
- Race class (G1, G2, G3, L, etc.)
- Distance
- Place (1st, 2nd, 3rd — keep all for future use)
- Horse name
- Horse profile href/URL

Filter to **flag** Group 1/2/3 races but **keep all placings** — the scope may expand to Listed races and placings later and we don't want to re-scrape.

---

## Sales Matching Logic

```
For each Group race winner:
  key = (Dam name, YOB)
  find all rows in sales data where:
    sales_data.Dam == key.Dam  [case-insensitive, strip whitespace]
    sales_data.YOB == key.YOB  [derived from DOB]
  
  if multiple matches:
    return all rows (horse may have sold multiple times across sale types)
  if no match:
    flag as "No sales record found"
```

---

## Output Format

Final deliverable columns:
```
Winner Name | Race Class - Race Name - Year | Sire | Dam | YOB | Sale Name | Sale Price | Sale Year | Vendor
```

- Multiple Group wins per horse: consolidate into one row (decision to be made after data review)
- Multiple sale records per horse: include all (one row per sale)
- Vendor analysis by stud farm is the primary client output

---

## Scope Boundaries

| In scope now | Potentially later |
|---|---|
| G1, G2, G3 winners | Listed race winners |
| All Australian states | — |
| All horse ages | — |
| Inglis sales data | Other sales companies (Magic Millions etc.) |
| Winners only | Placings (2nd, 3rd) |

**Do not expand scope without being asked. But do scrape placings now so we don't have to re-scrape later.**
