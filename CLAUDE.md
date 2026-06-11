# CLAUDE.md — Instructions for Claude Code

This file is read automatically by Claude Code at the start of every session. Follow these instructions for all work in this repository.

---

## Read These First

Before doing anything else in a new session, read:
1. `docs/CONTEXT.md` — full project background, data sources, pipeline overview
2. `docs/DECISIONS_LOG.md` — decisions already made (do not reverse without flagging)
3. `docs/EDGE_CASES.md` — known issues and how to handle them
4. `docs/DATA_DICTIONARY.md` — column definitions for all files

Sample files are in `SAMPLES/` — use these to understand data structure before writing any scripts.

---

## Project in One Paragraph

Rogers Bloodstock wants to know which Australian stud farm vendors are producing Group race winners. We are building a pipeline that: generates racing.com URLs from a group race list → scrapes race results → scrapes horse profiles for Sire/Dam/DOB → matches winners back to sales data using Dam + YOB as the join key → produces a final report showing Winner | Race | Sire | Dam | YOB | Sale Price | Vendor.

---

## The Join Key — Never Forget This

**Dam name + YOB** is the key that connects race winners to sales records. Horse names cannot be used — yearlings are unnamed at sale. One dam = one foal per year = unique identifier.

---

## Current Status & What Needs Doing Next

### ✅ Done
- Project structure defined
- Sample files reviewed and understood
- Pipeline design agreed
- All context documents created

### ⚠️ Next Task: Investigate racing.com scraping architecture
Before writing Script 2, investigate whether racing.com is statically or JS-rendered.
See `docs/EDGE_CASES.md` EC-001 for exact steps.
This determines the entire scraping approach — do not skip this.

### 📋 Script Build Order (do not skip steps)
1. `scripts/01_url_generator.py` — venue code + date → racing.com URL
2. `scripts/02_race_scraper.py` — scrape meeting pages (architecture TBD pending EC-001)
3. `scripts/03_profile_scraper.py` — scrape horse profiles for Sire/Dam/DOB
4. `scripts/04_sales_matcher.py` — join winners to sales data on Dam + YOB
5. `scripts/05_consolidation.py` — final report assembly

---

## Rules for This Project

**Always:**
- Check `DECISIONS_LOG.md` before making an architectural choice
- Flag edge cases in `EDGE_CASES.md` as you find them (don't just handle them silently)
- Add random delays (1-3 seconds) between any HTTP requests
- Keep raw and final data out of git (see `.gitignore`)
- Use `data/interim/` for outputs between scripts
- Log scrape progress so runs can be resumed if interrupted

**Never:**
- Try to construct horse profile URLs from horse names — the numeric suffix is unpredictable
- Assume racing.com is statically rendered — investigate first (EC-001)
- Commit anything in `data/raw/` or `data/final/`
- Delete or overwrite raw data files
- Expand scope (e.g. add Listed races, add placings to report) without being asked

---

## Asking for Help

If you hit an edge case not covered in `EDGE_CASES.md`, add it there and flag it rather than making an assumption. This project is being coordinated across Claude Code and claude.ai — decisions made in one place need to be visible in both.
