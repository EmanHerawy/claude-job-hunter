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

## Job Boards

Custom job sites to include in every search. Add any site here once — it will be searched on every future run automatically.

For each board, the agent uses `site:[domain] [keywords]` in web search queries and fetches matching listing pages directly. If the site has a known API or structured URL pattern, note it and the agent will prefer that over plain web search.

<!-- Built-in boards (always included when Web Search is enabled):
  LinkedIn Jobs     site:linkedin.com/jobs
  Wellfound         site:wellfound.com
  Lever             site:jobs.lever.co
  Ashby             site:jobs.ashbyhq.com
  Greenhouse        site:boards.greenhouse.io / site:job-boards.greenhouse.io
-->

<!-- HOW TO ADD A BOARD
Copy one of the examples below and fill in the fields:

- **Name:**        Human-readable name (e.g., "Remote.com")
- **Domain:**      The domain to search (e.g., remote.com)
- **Search URL:**  Direct search URL pattern with {keywords} placeholder if available
                   e.g., https://remote.com/jobs?search={keywords}
                   Leave blank to use site: web search only
- **Tags:**        Optional — categorise the board (remote-only, web3, security, startup, etc.)
                   Used to auto-enable the board only when relevant role keywords match
- **Enabled:**     yes / no (quick toggle without deleting the entry)
- **Notes:**       Anything useful — quality signal, signup required, region bias, etc.
-->

### Example boards (disabled by default — set Enabled: yes to activate)

<!-- Remote-focused boards
- **Name:** Remote.com
- **Domain:** remote.com
- **Search URL:** https://remote.com/jobs?search={keywords}
- **Tags:** remote-only
- **Enabled:** no
- **Notes:** Strong for fully remote roles, global employers

- **Name:** RemoteOK
- **Domain:** remoteok.com
- **Search URL:** https://remoteok.com/remote-{keywords}-jobs
- **Tags:** remote-only, startup
- **Enabled:** no
- **Notes:** Good for startup and indie company remote roles

- **Name:** We Work Remotely
- **Domain:** weworkremotely.com
- **Search URL:** https://weworkremotely.com/remote-jobs/search?term={keywords}
- **Tags:** remote-only
- **Enabled:** no

Web3 / Crypto boards
- **Name:** CryptoJobsList
- **Domain:** cryptojobslist.com
- **Search URL:** https://cryptojobslist.com/search#searchQuery={keywords}
- **Tags:** web3, crypto, blockchain
- **Enabled:** no

- **Name:** web3.career
- **Domain:** web3.career
- **Search URL:** https://web3.career/{keywords}-jobs
- **Tags:** web3, blockchain
- **Enabled:** no

Security / InfoSec boards
- **Name:** CyberSecJobs
- **Domain:** cybersecjobs.com
- **Search URL:**
- **Tags:** security, infosec
- **Enabled:** no

AI / ML boards
- **Name:** AIJobs.net
- **Domain:** aijobs.net
- **Search URL:** https://aijobs.net/?s={keywords}
- **Tags:** ai, ml
- **Enabled:** no
-->

## Web Search

Web-based job discovery. When enabled, `/job-search` searches the web for matching jobs in addition to (or instead of) searching company ATS boards. This lets you find roles without specifying companies.

**Enabled:** no
**Role queries:** (search phrases, e.g., "staff security engineer AI", "platform security remote")
**Include ATS sites:** yes
**Include general boards:** yes
**Exclude companies:** (companies to skip in web results)
**Max results per query:** 20
