# Job Search Watchlist

Companies and filters for `/job-search`. Edit this file to customize your automated job search.

## Search Filters

Global filters applied to all companies. Leave blank to match all.

**Role keywords:** security, platform, infrastructure
**Department keywords:**
**Description keywords:**
**Work modality:** Remote, Hybrid
**Location:**
**Employment type:** FullTime
**Posted within:** 14
**Region:**
**Exclude region:**

<!--
FILTER REFERENCE
═══════════════════════════════════════════════════════
Posted within: maximum age of job posting, in days (plain number).
  Examples:
    1   = today only
    7   = last 7 days (last week)
    14  = last 14 days (last 2 weeks)
    30  = last 30 days (last month)
    90  = last 90 days (last 3 months)
  Blank = any age. Jobs with unknown posting dates are included with ⚠️ warning.

Region: geographic scope filter.
  Works WITH Work modality — applies to where the job is physically based
  and where remote work is permitted.

  Region: include filter — only show jobs in these regions/countries
    Blank = match anywhere

  Exclude region: exclusion filter — drop jobs in these regions/countries
    Blank = exclude nothing
    Examples:
      Exclude region: USA                   → drop all US-based or US-only-remote jobs
      Exclude region: USA, Canada           → drop North America
      Exclude region: China, Russia         → drop specific countries
      Exclude region: APAC                  → drop entire APAC region

  Predefined region names (case-insensitive, use in either field):
    USA          United States (all US states, US-only remote)
    Canada       Canada
    UK           United Kingdom
    EMEA         Europe, Middle East, Africa
    Europe       EU27 + UK + Norway + Switzerland + Iceland
    APAC         Australia, NZ, Japan, South Korea, Singapore, India,
                 Hong Kong, Taiwan, China, Indonesia, Malaysia, Thailand
    LATAM        Brazil, Mexico, Argentina, Colombia, Chile, Peru, and others
    Global       Remote-worldwide / "Anywhere" / no geographic restriction

  You can also use individual country names:
    Region: Germany, Netherlands, UAE
    Exclude region: USA, China, Russia

  Combining:
    Region: EMEA, UK          → EMEA-based jobs OR UK-based jobs (OR)
    Region: Global, EMEA      → worldwide remote OR EMEA onsite
    Exclude region: USA, APAC → drop US and APAC jobs, keep everything else

  Matching logic:
    OnSite/Hybrid jobs: matched against location field
    Remote jobs:        matched against remoteRegion inferred from description
                        ("US only", "EU only", "Worldwide", etc.)
    Unknown remote:     flagged ⚠️ in results but NOT silently excluded
-->


## Companies

Add companies below. Set platform to `auto` to discover automatically, or specify `ashby` / `greenhouse` with the board token from their careers URL.

<!-- Copy this block for each company:

### [Company Name]
- **Platform:** auto
- **Board token:**
- **Notes:**
- **Filters:** (uses global filters)

Per-company filter overrides (optional — replaces global for this company only):
- **Role keywords:**
- **Work modality:**
- **Posted within:**
- **Region:**
- **Exclude region:**

-->

## Web Search

Web-based job discovery. When enabled, `/job-search` searches the web for matching jobs in addition to (or instead of) searching company ATS boards. This lets you find roles without specifying companies.

**Enabled:** no
**Role queries:** (search phrases, e.g., "staff security engineer AI", "platform security remote")
**Include ATS sites:** yes
**Include general boards:** yes
**Exclude companies:** (companies to skip in web results)
**Max results per query:** 20
