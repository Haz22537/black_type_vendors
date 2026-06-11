# Edge Cases & Known Issues

Reference this before writing any scraping or matching script. Add new edge cases here as they are discovered.

---

## SCRAPING

### EC-001 — Racing.com JS Rendering (UNRESOLVED — INVESTIGATE FIRST)
**Status:** ⚠️ Unresolved — must be investigated before Script 2 is written
**Issue:** Racing.com appears to be a React/Next.js application. The HTML sample in `SAMPLES/racingcom_meeting_part_allresultspage.txt` shows a horse name `<a>` tag with no `href` attribute. This may mean:
- hrefs are injected by JavaScript after page load (requires Playwright)
- OR the sample was simply cut off before the href appeared (requests may work fine)
**Investigation steps:**
1. Fetch `https://www.racing.com/form/2026-06-06/flemington/` with Python `requests`
2. Check if response contains race data or is a near-empty JS shell
3. Check if horse `<a>` tags have `href` attributes in the static response
4. If JS-rendered, inspect browser network tab for underlying API calls (XHR/fetch)
**Possible outcomes:**
- A: Static → use `requests` + `BeautifulSoup`
- B: JS-rendered → use Playwright
- C: Discoverable API → call API directly (cleanest if available)
**Record outcome here once resolved.**

---

### EC-002 — Horse Profile Numeric Suffix
**Status:** ✅ Known, handled by design
**Issue:** Horse profile URLs contain an unpredictable numeric suffix e.g. `racing.com/horses/uncommon-james-5279249`
**Resolution:** The href must be harvested from the results page. It cannot be constructed. Script 2 must capture the full href for every runner. Script 3 uses these harvested hrefs directly.
**Never attempt to construct horse profile URLs from horse names.**

---

### EC-003 — Venue Name to URL Slug Mapping
**Status:** 🔄 Partially resolved — build iteratively
**Issue:** Race list uses abbreviated venue codes (FLEM, RAND, CAUL etc.) that must map to racing.com URL slugs. Some venues have:
- Sponsor names in the official name (e.g. "Sportsbet Sandown")
- Multiple tracks at one venue (Sandown Hillside vs Sandown Lakeside)
- Name changes over time
- Abbreviated codes that don't obviously map to a slug

**Known confirmed mappings (from sample files):**
| Code | Slug | Full Name |
|------|------|-----------|
| FLEM | `flemington` | Flemington |
| RAND | `royal-randwick` | Royal Randwick |
| E FM | `eagle-farm` | Eagle Farm |
| CAUL | `caulfield` | Caulfield — *to be confirmed* |
| MORP | `morphettville` | Morphettville — *to be confirmed* |
| BLMT | `belmont-park` | Belmont Park — *to be confirmed* |
| HAWK | `hawkesbury` | Hawkesbury — *to be confirmed* |

**Resolution approach:** Script 1 should use a lookup dictionary. Rows with no mapping found get flagged as `needs_review` in the output. Add confirmed mappings to this table as they are verified.

---

### EC-004 — Moved or Abandoned Race Meetings
**Status:** ℹ️ Known risk, handle gracefully
**Issue:** Occasionally a scheduled race meeting is moved to a different date or venue (weather, track conditions etc.). The race list may have the original scheduled date/venue, but racing.com will have results under the actual date/venue.
**Resolution:** When a generated URL returns a 404 or no results, flag it for manual review rather than failing silently. Do not assume a 404 means the meeting didn't happen — it may have moved.
**Script 1 output should include a `url_confidence` column:** `confirmed` / `inferred` / `needs_review`

---

### EC-005 — Race Name Variations
**Status:** ℹ️ Known, low priority
**Issue:** The race list has both `PRA Seasonal Race Name` and `Registered Race Name` columns. These sometimes differ. The scraped race name from racing.com may match either, neither, or a sponsor-prefixed version.
**Resolution:** Use `Registered Race Name` from the race list as the canonical name in the final output. Don't rely on race name matching between the list and scraped results — use date + venue + race number as the match.

---

## SALES DATA MATCHING

### EC-006 — Dam Name Inconsistencies
**Status:** ℹ️ Known risk
**Issue:** Dam names may have minor variations between racing profiles and sales records:
- Spacing differences: `Walk in the Park` vs `Walk In The Park`
- Country suffixes: `Walk in the Park (NZ)` in one source, `Walk in the Park` in another
- Punctuation: apostrophes, hyphens
**Resolution:** Normalise dam names before matching:
1. Strip country suffixes in parentheses: `(NZ)`, `(USA)`, `(IRE)`, `(GB)` etc.
2. Convert to lowercase
3. Strip leading/trailing whitespace
4. Optionally: strip internal extra spaces
**Flag near-matches (e.g. fuzzy match score > 0.9) as `fuzzy_match` for human review.**

---

### EC-007 — Price Formatting in Sales Data
**Status:** ✅ Known, handle in cleaning
**Issue:** The `Price` column in Inglis data contains various formats:
- `"$450,000"` — normal sold price
- `"Passed In - Reserve: $60,000"` — passed in with reserve shown
- Blank / null — withdrawn lots
**Resolution:**
- Keep original `Price` string as-is in all interim files
- In Script 5, add a `sale_status` field derived from Price: `Sold`, `Passed In`, `Withdrawn`
- For numeric analysis, extract numeric value separately into `price_numeric` field
- Never overwrite the original Price string

---

### EC-008 — Multiple Sales Records Per Horse
**Status:** ✅ Known, handled by design
**Issue:** A horse may appear in multiple sales (e.g. sold as weanling, then again as yearling, then as broodmare). The Dam + YOB key will match all of these.
**Resolution:** This is correct behaviour — return all matching sale records. The final output will have one row per sale record per horse. This is valuable information for the client (shows re-selling activity).

---

### EC-009 — YOB Derivation from DOB
**Status:** ℹ️ Needs testing against real data
**Issue:** DOB in sales data is a full date (e.g. `2017-11-10`). YOB is derived as the calendar year of DOB. Australian thoroughbred racing uses 1 August as the universal birthday — but for our purposes (matching sale records to race profiles) we need to ensure both sources use the same YOB convention.
**Resolution:** 
- Derive YOB as simply the calendar year of DOB from sales data: `YOB = DOB.year`
- Check this matches YOB as it appears on racing.com horse profiles during testing
- If a mismatch is found, document the correction rule here

---

## GENERAL

### EC-010 — Rate Limiting / Scraping Etiquette
**Status:** ℹ️ Must be observed
**Issue:** Scraping racing.com at high speed risks IP blocking and is poor practice.
**Resolution:**
- Add random delays between requests (1-3 seconds minimum)
- Add retry logic with exponential backoff for failed requests
- Log all requests with timestamps
- Run scrapes in off-peak hours where possible
- If blocked, pause and resume — do not attempt to circumvent blocks

---

### EC-011 — Incomplete Race Day Scrapes
**Status:** ℹ️ Handle gracefully
**Issue:** A scrape may fail partway through a meeting page, leaving partial results.
**Resolution:**
- Track scrape status per meeting URL
- On re-run, skip meetings already fully scraped (check against log)
- Never append partial results without flagging them
- Keep a `scrape_log.csv` tracking: URL | status | timestamp | rows_captured
