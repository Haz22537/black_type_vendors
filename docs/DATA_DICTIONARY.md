# Data Dictionary

Column definitions for all files in the pipeline. Reference this when writing scripts to ensure consistent naming and handling.

---

## Input: `group_race_list.csv`

| Column | Type | Example | Notes |
|--------|------|---------|-------|
| DATE | String | `3-Aug-24` | Format: D-Mon-YY. Convert to YYYY-MM-DD for URL construction |
| R ID | Integer | `693` | Internal race ID — not used in URLs |
| Grp (numeric) | Integer | `1`, `2`, `3`, `4` | 1=G1, 2=G2, 3=G3, 4=Listed |
| Grp (text) | String | `G1`, `G2`, `G3`, `L` | Human-readable class |
| State | String | `VIC`, `NSW`, `SA`, `WA`, `QLD` | Australian state |
| Club | String | `VRC`, `ATC`, `MRC` | Racing club abbreviation |
| Venue | String | `FLEM`, `RAND`, `CAUL` | Track abbreviation → maps to URL slug |
| PRA Seasonal Race Name | String | `AURIE'S STAR HANDICAP` | May differ from registered name |
| Dist. | Integer | `1200` | Distance in metres |
| Prizemoney | String | `"$201,300"` | Formatted string — not used in pipeline |
| Age | String | `O`, `3YO&Up`, `3-Y-O` | Age restriction |
| Sex | String | `O`, `F`, `C&G`, `F&M` | Sex restriction |
| Wgt | String | `Hcp`, `WFA`, `SWP`, `Qlty` | Weight condition |
| Registered Race Name | String | `AURIE'S STAR HANDICAP` | Official name — use this in final output |

---

## Interim: `urls_generated.csv`

| Column | Type | Example | Notes |
|--------|------|---------|-------|
| DATE | String | `3-Aug-24` | Original from race list |
| date_formatted | String | `2024-08-03` | Converted for URL |
| Venue | String | `FLEM` | Original abbreviation |
| venue_slug | String | `flemington` | Derived for URL |
| URL | String | `https://www.racing.com/form/2024-08-03/flemington/` | Constructed URL |
| url_confidence | String | `confirmed`, `inferred`, `needs_review` | Flag for edge cases |
| Grp | String | `G1` | Carried forward from race list |
| Race Name | String | `AURIE'S STAR HANDICAP` | Carried forward |

---

## Interim: `race_results_raw.csv`

| Column | Type | Example | Notes |
|--------|------|---------|-------|
| date | String | `2024-08-03` | Race date |
| venue | String | `flemington` | Venue slug |
| venue_code | String | `FLEM` | Original abbreviation |
| race_number | Integer | `7` | Race number on the day |
| race_name | String | `AURIE'S STAR HANDICAP` | As scraped from page |
| race_class | String | `G3` | Group/Listed class |
| race_conditions | String | `1200m WFA 3YO+` | Full condition string |
| distance | Integer | `1200` | In metres |
| place | Integer | `1` | Finishing position |
| horse_name | String | `Uncommon James` | As appears on results page |
| horse_href | String | `https://www.racing.com/horses/uncommon-james-5279249` | Full profile URL — harvested from page |
| is_group_race | Boolean | `True` | Flag: True if G1/G2/G3 |
| scraped_at | Datetime | `2026-06-11 14:30:00` | When this row was scraped |

---

## Interim: `horse_profiles.csv`

| Column | Type | Example | Notes |
|--------|------|---------|-------|
| horse_name | String | `Uncommon James` | From results page |
| horse_href | String | `https://www.racing.com/horses/uncommon-james-5279249` | Source URL |
| sire | String | `Written Tycoon` | From profile page |
| dam | String | `Uncommon Belle` | From profile page — JOIN KEY (part 1) |
| dob | String | `2019-09-15` | Date of birth from profile |
| yob | Integer | `2019` | Derived from DOB — JOIN KEY (part 2) |
| profile_scraped_at | Datetime | `2026-06-11 14:30:00` | When scraped |

---

## Interim: `sales_matched.csv`

| Column | Type | Example | Notes |
|--------|------|---------|-------|
| horse_name | String | `Uncommon James` | From race results |
| dam | String | `Uncommon Belle` | Join key |
| yob | Integer | `2019` | Join key |
| sale_name | String | `Inglis Easter Yearling Sale` | Which sale |
| sale_year | Integer | `2020` | Year of sale |
| lot_number | Integer | `142` | Lot number at sale |
| sale_price | String | `"$450,000"` | As recorded — keep as string |
| sale_status | String | `Sold` | Sold / Passed In / Withdrawn |
| vendor | String | `"Vinery Stud, Scone"` | Use as-is |
| sex | String | `Colt` | At time of sale |
| colour | String | `B.` | Bay etc. |
| match_type | String | `exact`, `no_match` | Quality of the join |

---

## Output: `rogers_report_final.csv`

| Column | Type | Example | Notes |
|--------|------|---------|-------|
| winner_name | String | `Uncommon James` | Horse name |
| group_wins | String | `G3 Aurie's Star Hcp 2024` | One or more wins — may be concatenated |
| sire | String | `Written Tycoon` | From racing profile |
| dam | String | `Uncommon Belle` | Join key |
| yob | Integer | `2019` | Year of birth |
| sale_name | String | `Inglis Easter Yearling Sale` | Sale catalogue name |
| sale_price | String | `"$450,000"` | Price at sale |
| sale_year | Integer | `2020` | Year sold |
| vendor | String | `"Vinery Stud, Scone"` | Stud farm |
| no_sales_record | Boolean | `False` | True if no match found in sales data |

---

## Key Constants

```python
# Racing.com base URLs
RACING_COM_FORM_BASE = "https://www.racing.com/form/"
RACING_COM_HORSE_BASE = "https://www.racing.com/horses/"

# Group race classes to flag (keep all, flag these)
GROUP_CLASSES = ["G1", "G2", "G3"]
LISTED_CLASS = ["L"]

# Southern Hemisphere YOB cutoff
SH_BIRTHDAY_MONTH = 8   # August
SH_BIRTHDAY_DAY = 1     # 1st
# If DOB month < 8: racing YOB = calendar year
# If DOB month >= 8: racing YOB = calendar year (foal of that year)
# Standard YOB = year from DOB — verify against sales data in testing

# Join key fields
JOIN_KEY_DAM = "dam"
JOIN_KEY_YOB = "yob"
```
