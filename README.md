# Rogers Bloodstock — Group Winner Analysis

## Project Brief

Client (Rogers Bloodstock) wants to understand which Australian stud farms (vendors) are producing Group race winners. The hunch is that some farms are underperforming relative to their reputation or sales prices.

## Core Question
**Which vendors/stud farms are producing Group race winners, and what did those winners sell for?**

## Scope
- **Races:** All Group 1, Group 2, Group 3 winners in Australia
- **Date range:** ~mid 2023 to present (~3 years)
- **Sales:** All sale types (yearling, weanling, ready-to-run, broodmare) from major Australian sales companies
- **Current sales data:** Inglis (8+ years). Other companies to be added later — same key will work.
- **States:** All Australian states
- **Horse age at win:** No restriction

## Final Deliverable (Detail Table)
| Winner Name | Race Class - Race Name - Year | Sire | Dam | YOB | Sale Name | Sale Price | Sale Year | Vendor |

- Multiple Group wins per horse may be consolidated into one row (to be decided once data is assembled)
- Vendor analysis / pivot by stud farm is the key output for the client
- Visualisations and specific farm focus to be decided once data is in hand

---

## The Join Key
**Dam name + Year of Birth (YOB)**

This is the unique key that connects race winners back to sales records.
- Yearlings and weanlings are almost always unnamed at time of sale — horse name cannot be used
- A dam produces a maximum of one foal per year — Dam + YOB is a reliable unique identifier
- DOB is available in sales data → YOB is calculated from DOB
- YOB must account for Southern Hemisphere racing convention (August 1 cutoff)

---

## Data Pipeline Overview

```
INPUT: group_race_list.csv (DATE, Venue code, Race Name, Class etc.)
         ↓
SCRIPT 1: URL Generator
         → racing.com meeting URLs from venue code + date
         → Output: urls_generated.csv + edge_cases_flagged.csv
         ↓
SCRIPT 2: Race Day Scraper
         → Scrape each meeting page for all races
         → Filter/flag Group 1/2/3 races (keep all placings for future use)
         → Capture: Date | Venue | Race# | Race Name | Class | Distance | Place | Horse Name | Horse Profile href
         → Output: race_results_raw.csv
         ↓
SCRIPT 3: Horse Profile Scraper
         → Visit each unique horse profile URL (harvested hrefs from Script 2)
         → Extract: Sire | Dam | DOB → calculate YOB
         → Output: horse_profiles.csv (deduplicated)
         ↓
SCRIPT 4: Sales Matcher
         → Join race winners to sales data on Dam + YOB
         → Handle multiple sale records per horse
         → Output: sales_matched.csv
         ↓
SCRIPT 5: Consolidation
         → Final joined table
         → Flag winners with no sales match
         → Output: rogers_report_final.csv / .xlsx
```

---

## Repository Structure

```
rogers-bloodstock/
├── SAMPLES/                        # Sample files for reference and testing
│   ├── group_race_list_sample.csv
│   ├── meeting_allresults_urls_racingcom.csv
│   ├── horseprofile_urls_racingcom.csv
│   ├── racingcom_meeting_part_allresultspage.txt
│   └── inglis_saleresults_sample.csv
├── data/
│   ├── raw/                        # Input files — gitignored (client data)
│   ├── interim/                    # Intermediate outputs between scripts
│   └── final/                      # Final deliverable — gitignored
├── scripts/
│   ├── 01_url_generator.py
│   ├── 02_race_scraper.py
│   ├── 03_profile_scraper.py
│   ├── 04_sales_matcher.py
│   └── 05_consolidation.py
├── docs/
│   ├── CONTEXT.md                  # Full project context for Claude Code
│   ├── DATA_DICTIONARY.md          # Column definitions for all files
│   ├── DECISIONS_LOG.md            # Key decisions and reasoning
│   └── EDGE_CASES.md               # Known edge cases and how to handle them
├── .gitignore
└── README.md
```

---

## Important Notes for Claude Code
- **Read `docs/CONTEXT.md` before starting any work** — it has full background
- **Read `docs/EDGE_CASES.md` before writing any scraping script**
- **Read `docs/DECISIONS_LOG.md` before making architectural choices** — decisions already made should not be revisited without flagging
- Sample files in `SAMPLES/` are the source of truth for data structure
- All client data lives in `data/` which is gitignored — never commit raw or final data
