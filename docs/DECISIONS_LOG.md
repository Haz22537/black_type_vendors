# Decisions Log

A record of key decisions made during planning. Do not reverse these without flagging for human review. If a decision needs revisiting, add a note under the relevant entry rather than deleting it.

---

## Decision 001 — Join Key: Dam + YOB
**Date:** 2026-06-11
**Decision:** The primary key linking race winners to sales records is **Dam name + Year of Birth (YOB)**
**Reason:** Yearlings and weanlings are almost always unnamed at time of sale. Horse name cannot be used as a join key. A dam produces a maximum of one foal per year, making Dam + YOB a reliable unique identifier across all sale types and years.
**Implication:** YOB must be derived from DOB in sales data. Dam name matching must be case-insensitive and whitespace-stripped.

---

## Decision 002 — Scrape Placings, Report Winners
**Date:** 2026-06-11
**Decision:** Scrape and store 1st, 2nd, and 3rd place for every Group race. Current reporting scope is winners (1st) only.
**Reason:** Scope may expand to placings later. Re-scraping is expensive. Capturing now costs nothing extra.
**Implication:** `race_results_raw.csv` includes all placings. Filtering to winners happens at Script 5 (consolidation), not at scrape time.

---

## Decision 003 — Vendor Column: Use As-Is
**Date:** 2026-06-11
**Decision:** The Vendor column from Inglis data (format: `"Stud Name, Location"`) is used as-is in the final output. No splitting or cleaning of vendor name vs location.
**Reason:** Client is familiar with the vendors and the format is unambiguous. Cleaning adds complexity with no clear benefit at this stage.
**Implication:** Pivot table / grouping on vendor will need to be done on the full string. If a stud farm appears with inconsistent location suffixes across different years of data, this will need revisiting.

---

## Decision 004 — Race Class Scope: G1/G2/G3 Primary, Listed Parked
**Date:** 2026-06-11
**Decision:** Primary scope is Group 1, Group 2, Group 3 winners. Listed races are out of scope for now but may be added later upon client request.
**Reason:** Client brief specified Group winners. Listed adds volume without changing the core analysis.
**Implication:** Race list includes Listed races (Grp = L / 4). These rows should be carried through the pipeline but flagged, not discarded — so expansion requires no re-scrape.

---

## Decision 005 — Sales Data: Inglis First, Others Later
**Date:** 2026-06-11
**Decision:** Build and test the pipeline using Inglis sales data only. Other sales companies (Magic Millions, William Inglis etc.) to be added later.
**Reason:** Inglis data is already available. Pipeline architecture (Dam + YOB key) is sales-company-agnostic — adding more sources later requires no structural changes.
**Implication:** Final output will flag winners with no Inglis sales record. Some of these will have records at other sale companies — this is expected and noted, not an error.

---

## Decision 006 — Scraping Architecture: Investigate Before Building
**Date:** 2026-06-11
**Decision:** Do not assume whether racing.com is static or JS-rendered. Run a live investigation before choosing between requests/BeautifulSoup, Playwright, or direct API calls.
**Reason:** CSS class naming on racing.com suggests React/Next.js. The HTML sample shows an `<a>` tag with no visible `href` — this may indicate JS rendering. Getting this wrong means rewriting the entire scraper.
**Implication:** Script 2 cannot be written until the investigation is complete and the architecture is confirmed. See `EDGE_CASES.md` for investigation steps.

---

## Decision 007 — Venue Mapping: Build Iteratively
**Date:** 2026-06-11
**Decision:** The venue abbreviation → racing.com URL slug mapping will be built iteratively as edge cases are encountered, not solved upfront.
**Reason:** Most venue mappings are predictable. Sponsor names and renamed tracks are the edge cases. Building a complete lookup table upfront is time-consuming and may contain errors. Better to build from known good examples and add edge cases as they arise.
**Implication:** Script 1 (URL generator) should flag rows where it cannot confidently map a venue, rather than failing silently or guessing. A `needs_review` flag in the output handles this.

---

## Decision 008 — Data Privacy: Raw and Final Data Gitignored
**Date:** 2026-06-11
**Decision:** All client data (raw inputs and final outputs) is stored in `data/` which is gitignored. Only sample files in `SAMPLES/` are committed to the repo.
**Reason:** This is client data. It should not be in a version-controlled repository even if the repo is private.
**Implication:** Anyone cloning the repo needs to source the raw data files separately. The README and this log document what files are expected where.
