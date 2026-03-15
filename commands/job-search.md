# Job Search — Search ATS Boards & the Web for Matching Roles

Search for matching roles across ATS job boards (Ashby, Greenhouse) and the open web. Supports three modes: ATS-only (search specific companies), web discovery (find jobs by keywords without knowing companies), and combined. Auto-evaluate approved matches using the same pipeline as `/should-i-apply`.

## Inputs

**Usage:** `/job-search "company1, company2" "role-keywords" "profile-name"`

Parse `$ARGUMENTS` as up to three quoted strings:
- **First quoted string** (optional) — Comma-separated list of company names or board tokens to search
- **Second quoted string** (optional) — Comma-separated role keyword filters (e.g., `"security, platform, infrastructure"`)
- **Third quoted string** (optional) — Profile name (e.g., `"default"`, `"security"`). See Profile Resolution below.

**Search modes:**
- **ATS mode** (companies provided via args or watchlist) — searches each company's Ashby/Greenhouse board
- **Web discovery mode** (no companies, but role keywords provided) — searches the web for matching jobs, discovers companies automatically
- **Combined mode** (watchlist has `## Web Search` section with `Enabled: yes`) — runs both ATS and web discovery

If the first argument is empty/blank AND the second argument has role keywords, activate web discovery mode. If both are empty, use the watchlist (companies + web search config if enabled). If nothing is configured, show a clear message explaining the options.

## Setup

Resolve paths by checking (in order):
1. Read `~/.claude/.claude-job-hunter.conf` for `WORK_DIR`, `REPO_DIR`, and component paths
2. Fall back: `$CLAUDE_JOB_HUNTER_DIR` → `./claude-job-hunter/` → `~/claude-job-hunter/`

Required paths:
- **Profile:** `[PROFILE_DIR]/[resolved-profile-name].md`
- **Watchlist:** `[WORK_DIR]/WATCHLIST.md`
- **Search log:** `[WORK_DIR]/JOB_SEARCH_LOG.md`
- **Evaluations dir:** `[EVALUATIONS_DIR]/`
- **Resources dir:** `[RESOURCES_DIR]/` (research checklists)
- **Watchlist template:** `[REPO_DIR]/resources/watchlist-template.md`

If the config exists, use its paths. Otherwise, use `[claude-job-hunter]/` as both repo and working directory.

If nothing is found, ask the user to run `/setup` first or provide the path.

## Profile Resolution

Determine which profile to use:

1. **Explicit argument:** If a profile name was passed (3rd arg), use `[PROFILE_DIR]/[profile-name].md` (append `.md` if not present)
2. **Config default:** Read `ACTIVE_PROFILE` from `~/.claude/.claude-job-hunter.conf` — if set, use `[PROFILE_DIR]/[ACTIVE_PROFILE].md`
3. **Auto-detect:** Scan `[PROFILE_DIR]/` for `.md` files other than `candidate-profile.md`:
   - **One found** → use it automatically
   - **Multiple found** → list each with its first heading line, ask the user to pick
   - **None found** → tell the user: "No profile found. Run `/build-profile` first."

The resolved profile path replaces all references to `candidate-profile.md` below.

## Workflow

Execute these phases in order. Present findings to the user after each phase for confirmation before proceeding.

---

### Phase 1: Setup & Configuration

1. Resolve all paths using the standard method above
2. Read the candidate profile at `[PROFILE_DIR]/[resolved-profile-name].md`
   - If no profile exists, tell the user to run `/build-profile` first
3. Read the watchlist at `[WORK_DIR]/WATCHLIST.md`
   - If no watchlist exists, copy the template from `[REPO_DIR]/resources/watchlist-template.md` to `[WORK_DIR]/WATCHLIST.md`
4. Read the search log at `[WORK_DIR]/JOB_SEARCH_LOG.md` (if it exists) for deduplication
5. Build the search plan by merging:
   - Companies from arguments (first quoted string, comma-separated) — if provided
   - Companies from watchlist `## Companies` section
   - Deduplicate company list
6. Build filter set by merging:
   - Role keywords from arguments (second quoted string, comma-separated)
   - Global filters from watchlist `## Search Filters` section
   - Per-company filter overrides from watchlist (if a company entry has its own `Filters:` line)
   - Parse `Posted within` — read as a plain integer (number of days). Compute cutoff: `now − N days`. Store as absolute ISO 8601 cutoff date. Example: `14` → cutoff = today − 14 days. Blank = no cutoff.
   - Parse `Region` — split comma-separated values; resolve each to a named region or country. Store as an include list (e.g., `["EMEA", "UK"]`).
   - Parse `Exclude region` — split comma-separated values; resolve each to a named region or country. Store as an exclude list (e.g., `["USA", "China"]`). A job is dropped if it matches ANY entry in the exclude list, regardless of what `Region` says.
7. Read the `## Web Search` section from the watchlist (if it exists) for web discovery config
8. **Determine search mode:**
   - If the merged company list is NOT empty → **ATS mode**. Also check if watchlist `## Web Search` has `Enabled: yes` — if so, use **combined mode** (ATS + web discovery).
   - If the merged company list IS empty AND role keywords were provided (from args or watchlist filters) → **web discovery mode**
   - If the merged company list IS empty AND no role keywords AND watchlist has `## Web Search` section with `Enabled: yes` and `Role queries:` → **web discovery mode** using those queries
   - If the merged company list IS empty AND no role keywords AND no web search config → tell the user: "No companies or role keywords provided. Options: (1) pass role keywords: `/job-search \"\" \"security engineer\"`, (2) add companies to your watchlist, (3) enable `## Web Search` in your watchlist with role queries." Then stop.
9. Present the search plan to the user:

```
Job Search Plan

Mode: [ATS / Web Discovery / Combined]
Companies to search: [list or "none — discovering via web search"]
Web search queries: [list, if web discovery or combined mode]
Role keywords: [list]
Department keywords: [list]
Description keywords: [list]
Work modality: [list or "any"]
Location: [list or "any"]
Employment type: [list or "any"]
Posted within: [e.g., "14 days (cutoff: 2026-03-01)" or "any"]
Region: [e.g., "EMEA, UK" or "any"]
Exclude region: [e.g., "USA, China" or "none"]

Previously searched: [count from log] jobs already in log
```

**Confirm with user before proceeding.**

---

### Phase 2a: Web Search Discovery

**Skip this phase if operating in ATS-only mode (companies were provided and no web search config is active).**

Use WebSearch to find job postings matching role keywords across the web. The goal is to discover companies and job URLs, then pivot to structured ATS APIs wherever possible.

#### 2a-i. Construct Search Queries

Build queries by combining role keywords with active filter dimensions (modality, location). For each role keyword, generate queries in three tiers:

**Tier 1 — ATS site searches (highest value — these can pivot to full API data):**
- `site:jobs.ashbyhq.com "{keyword}" {modality} {location}`
- `site:boards.greenhouse.io "{keyword}" {modality} {location}`
- `site:job-boards.greenhouse.io "{keyword}" {modality} {location}`
- `site:jobs.lever.co "{keyword}" {modality} {location}`

**Tier 2 — General job boards:**
- `"{keyword}" {modality} {location} site:linkedin.com/jobs`
- `"{keyword}" {modality} {location} site:wellfound.com`

**Tier 3 — Open web career pages:**
- `"{keyword}" careers OR jobs {modality} {location} {current_year}`

Where `{modality}` and `{location}` come from active filters (omit if "any").

**Date filter in web queries:** If `Posted within` is set, append a time qualifier to Tier 2 and Tier 3 queries where supported:
- LinkedIn: append `&f_TPR=r[seconds]` to LinkedIn Jobs URLs where seconds = N × 86400 (e.g., 7 days → `r604800`, 14 days → `r1209600`, 30 days → `r2592000`)
- General web queries: append `after:[YYYY-MM-DD]` using the computed cutoff date (e.g., `after:2026-03-01`)
- ATS API boards (Ashby/Greenhouse): these return all jobs; apply date filter in Phase 3c after fetching

**Region filter in web queries:** Append region terms to queries where applicable:
- `Region: EMEA` → add `"Europe" OR "EMEA" OR "UK" OR "Remote EMEA"` to Tier 3 queries
- `Region: USA` → add `"United States" OR "Remote US"` to queries
- `Exclude region: USA` → add `-"US only" -"United States only" -"must be based in US"` to queries
- `Exclude region: [country]` → add `-"[country] only" -"[country]-based"` for each excluded entry
- `Region: Global` → add `"remote worldwide" OR "remote global" OR "anywhere"` to queries

**Query limits:** At most 4 Tier 1 queries per keyword (one per ATS platform), 2 Tier 2 per keyword, 1 Tier 3 per keyword. If 3+ role keywords, prioritize the first two for Tier 2 and 3.

If the watchlist `## Web Search` section has custom `Role queries:`, use those in addition to the auto-generated queries.

#### 2a-ii. Execute Searches and Classify URLs

For each query, run WebSearch and collect result URLs. Deduplicate URLs across queries.

**Classify each URL by pattern:**

| URL Pattern | Classification | Action |
|---|---|---|
| `jobs.ashbyhq.com/{token}` or `jobs.ashbyhq.com/{token}/...` | Ashby | Extract `{token}` → add company to ATS list with platform `ashby` |
| `boards.greenhouse.io/{token}/...` or `job-boards.greenhouse.io/{token}/...` | Greenhouse | Extract `{token}` → add company to ATS list with platform `greenhouse` |
| `jobs.lever.co/{company}/...` | Lever | Add to direct-fetch list |
| `*.myworkdayjobs.com/...` | Workday | Add to direct-fetch list |
| `linkedin.com/jobs/view/...` | LinkedIn | Add to direct-fetch list |
| `wellfound.com/...` | Wellfound | Add to direct-fetch list |
| Other career page URLs | Unknown | Add to direct-fetch list |

**ATS pivot is the key win:** For Ashby and Greenhouse URLs, extract the board token from the URL path. Group by token (multiple URLs from the same board = one company). These companies join the Phase 2 ATS discovery pipeline, which fetches ALL jobs from their board — not just the ones found via web search.

If the watchlist `## Web Search` section has `Exclude companies:`, skip URLs whose company name or domain matches.

#### 2a-iii. Fetch Direct-Fetch URLs

For each URL in the direct-fetch list (non-ATS platforms), use WebFetch to retrieve the page and extract job details:

```
WebFetch(url, "Extract from this job posting: job title, company name, location(s),
remote/hybrid/on-site, employment type, department or team, compensation/salary if shown,
and the full job description text. Return as structured data.")
```

**Rate limiting:** Fetch at most 20 direct-fetch URLs (configurable via watchlist `Max results per query:` or default 20). Prioritize Tier 1 results, then Tier 2, then Tier 3.

**Normalize each fetched job to the common schema:**

| Field | Source |
|---|---|
| `title` | Extracted from page |
| `location` | Extracted; `"Unknown"` if not found |
| `secondaryLocations` | `[]` |
| `department` | Extracted if present; `"Unknown"` if not |
| `workplaceType` | Infer from text: `Remote` / `Hybrid` / `OnSite` / `Unknown` |
| `employmentType` | Infer from text; default `FullTime` |
| `description` | Full description text from page |
| `compensation` | Extracted if visible; null otherwise |
| `jobUrl` | The fetched URL |
| `company` | Extracted from page |
| `platform` | `lever`, `workday`, `wellfound`, `linkedin`, or `web` |
| `source` | `web-search` |

Discard URLs where neither a title nor company name can be extracted — note: `"Could not parse job posting at {url} — skipping."`

#### 2a-iv. Merge Into Search Plan

After web discovery completes:

1. **ATS-discovered companies:** Add to the company list for Phase 2 with their discovered platform and board token. Skip duplicates already in the list from args/watchlist.
2. **Direct-fetch jobs:** These skip Phase 2/3a (ATS discovery and API fetch) and enter the pipeline as pre-normalized jobs at Phase 3c (filtering).

3. **Present discovery results:**

```
Web Search Discovery Results

Queries executed: [N]
Unique URLs found: [N]

ATS boards discovered: [N]
  [Company A] → Ashby (token: {token}) — will fetch full board
  [Company B] → Greenhouse (token: {token}) — will fetch full board

Direct-fetch jobs: [N]
  [Company C] — [Title] (via Lever)
  [Company D] — [Title] (via career page)

Unparseable: [N] skipped

These [N] ATS companies will be fully searched via API next.
[N] direct-fetch jobs proceed to filtering.

Continue?
```

**Confirm with user before proceeding.**

---

### Phase 2: ATS Platform Discovery

For each company in the search plan — from arguments, watchlist, AND web search discovery (Phase 2a) — determine the ATS platform and board token.

**Companies from Phase 2a with a known platform and token skip the discovery steps below** and proceed directly to API fetch in Phase 3a.

**Resolution order for each company:**

1. **Check watchlist first** — if the company entry has `Platform: ashby` or `Platform: greenhouse` AND a `Board token:` value, use those directly. Skip discovery for this company.

2. **Try Ashby API** — attempt to fetch:
   ```
   GET https://api.ashbyhq.com/posting-api/job-board/{token}?includeCompensation=true
   ```
   Try these token variants (in order):
   - Company name as-is, lowercased (e.g., `anthropic`)
   - Company name hyphenated (e.g., `palo-alto-networks`)
   - Company name without spaces (e.g., `paloaltonetworks`)
   - Common abbreviations if obvious (e.g., `pan` for Palo Alto Networks)

   A successful response (HTTP 200 with job data) confirms Ashby.

3. **Try Greenhouse API** — attempt to fetch:
   ```
   GET https://boards-api.greenhouse.io/v1/boards/{token}/jobs?content=true
   ```
   Try the same token variants as above.

   A successful response (HTTP 200 with job data) confirms Greenhouse.

4. **WebSearch fallback** — if neither API responds:
   - Search for `"{company name}" careers site` or `"{company name}" jobs board`
   - Look for URLs containing `jobs.ashbyhq.com/{token}`, `boards.greenhouse.io/{token}`, or `jobs.lever.co/{token}`
   - Extract the board token from the URL
   - Confirm by hitting the appropriate API with the extracted token

5. **Give up gracefully** — if no ATS board is found after all attempts:
   - Report: `"⚠ {company}: No Ashby or Greenhouse board found. Skipping."`
   - Suggest the user check the careers page manually and add the token to their watchlist

**Output:** Present discovery results:

```
ATS Discovery Results

✓ Anthropic — Ashby (token: anthropic)
✓ Cloudflare — Greenhouse (token: cloudflare)
✗ Acme Corp — No board found (skipping)

Proceeding with [N] companies. Continue?
```

**Confirm with user before proceeding.**

---

### Phase 3: Fetch & Filter Jobs

For each confirmed company/platform, fetch all jobs and apply filters.

#### 3a. Fetch Jobs

**Ashby:** `GET https://api.ashbyhq.com/posting-api/job-board/{token}?includeCompensation=true`

Response structure (extract from `jobs` array):
```json
{
  "title": "Security Engineer",
  "location": "San Francisco, CA",
  "department": "Engineering",
  "team": "Security",
  "employmentType": "FullTime",
  "workplaceType": "Remote",  // or "Hybrid", "OnSite"
  "compensation": { ... },
  "descriptionPlain": "...",
  "jobUrl": "https://jobs.ashbyhq.com/...",
  "secondaryLocations": [...]
}
```

**Greenhouse:** `GET https://boards-api.greenhouse.io/v1/boards/{token}/jobs?content=true`

Response structure (extract from `jobs` array):
```json
{
  "title": "Security Engineer",
  "location": { "name": "San Francisco, CA" },
  "departments": [{ "name": "Engineering" }],
  "content": "<p>Job description HTML...</p>",
  "absolute_url": "https://boards.greenhouse.io/...",
  "metadata": [...]
}
```

#### 3b. Normalize to Common Schema

Convert API responses and web-discovered jobs to a unified format for filtering:

| Field | Ashby Source | Greenhouse Source | Web-Discovered Source |
|---|---|---|---|
| `title` | `title` | `title` | Extracted from page |
| `location` | `location` | `location.name` | Extracted from page |
| `secondaryLocations` | `secondaryLocations` | — | `[]` |
| `department` | `department` + `team` | `departments[].name` | Extracted if present |
| `workplaceType` | `workplaceType` | Infer from `location.name` | Infer from description |
| `employmentType` | `employmentType` | Infer from `metadata` or title | Infer from description |
| `description` | `descriptionPlain` | Strip HTML from `content` | Full page text |
| `compensation` | `compensation` | Infer from `content` if present | Extracted if visible |
| `jobUrl` | `jobUrl` | `absolute_url` | Page URL |
| `company` | (from search plan) | (from search plan) | Extracted from page |
| `platform` | `ashby` | `greenhouse` | `lever`, `workday`, `web`, etc. |
| `source` | `ats-api` | `ats-api` | `web-search` or `web-search → ashby` / `web-search → greenhouse` |
| `postedAt` | `publishedDate` (ISO 8601) | `first_published_at` or `updated_at` | Extracted from page if present; `null` otherwise |
| `remoteRegion` | Infer from description text | Infer from description text | Infer from description text |

**`remoteRegion` inference** (applied to all jobs where `workplaceType` = `Remote` or `Hybrid`):
Scan the job title + description for restriction language:
- `"US only"`, `"United States only"`, `"must be based in US"`, `"US citizens"` → `US_ONLY`
- `"EU only"`, `"Europe only"`, `"must reside in Europe"`, `"EEA"` → `EU_ONLY`
- `"EMEA"` mentioned as work location → `EMEA`
- `"worldwide"`, `"globally"`, `"anywhere"`, no restriction mentioned → `GLOBAL`
- `"Canada"` mentioned as restriction → `CANADA_ONLY`
- `"UK only"`, `"United Kingdom only"` → `UK_ONLY`
- Ambiguous or not determinable → `UNKNOWN`

If `workplaceType` is `OnSite` or `Hybrid`, set `remoteRegion` to `null` (not applicable; use `location` for region matching instead).

For web-discovered jobs that were pivoted to ATS API (found via web search, fetched via Ashby/Greenhouse API), set `source` to `web-search → {platform}`.

For Greenhouse `workplaceType` inference:
- If location contains "Remote" → `Remote`
- If location contains "Hybrid" → `Hybrid`
- Otherwise → `OnSite`

#### 3c. Apply Filters

Filters are applied as **AND across dimensions, OR within each dimension**:

- **Role keywords** → case-insensitive substring match on `title`. If multiple keywords, match ANY.
- **Department keywords** → case-insensitive match on `department`. If multiple keywords, match ANY.
- **Description keywords** → case-insensitive match on plain text `description`. If multiple keywords, match ANY.
- **Work modality** → exact match on `workplaceType` (`Remote`, `Hybrid`, `OnSite`). If multiple values, match ANY.
- **Location** → case-insensitive substring match on `location` + `secondaryLocations`. If multiple locations, match ANY.
- **Employment type** → normalized match on `employmentType`. Map variants: `FullTime`/`Full-time`/`Full Time` → `FullTime`, etc. If multiple types, match ANY.
- **Posted within** → compare `postedAt` against cutoff datetime. Rules:
  - `postedAt` is after cutoff (recent enough) → **PASS**
  - `postedAt` is before cutoff (too old) → **FILTERED OUT**
  - `postedAt` is `null` (unknown) → **INCLUDE** but mark result with `⚠️ date unknown`
  - If `Posted within` filter is not set → skip (match all regardless of date)
- **Region (include)** and **Exclude region** → two separate fields, evaluated in order:

  **Step 1 — Exclusion (runs first, takes priority):**
  If `Exclude region` has values, drop the job if it matches ANY excluded entry.
  - Remote jobs: check `remoteRegion` (e.g., `Exclude region: USA` → drop if `remoteRegion = US_ONLY`)
  - OnSite/Hybrid: check `location` + `secondaryLocations` against the excluded country/region list
  - If `remoteRegion` is `UNKNOWN` and excluded region is `USA`: **flag ⚠️, do not silently drop** — show to user
  - Multiple exclusions: `Exclude region: USA, China, Russia` → drop if job matches ANY of them

  **Step 2 — Inclusion (only if `Region` is set):**
  Job must match at least one entry in `Region` (OR logic across entries). If `Region` is blank, skip this step.

  For **Remote jobs** (`workplaceType = Remote`) — match against `remoteRegion`:
  | Include region | Passes if remoteRegion is... |
  |---|---|
  | `USA` | `US_ONLY`, `GLOBAL`, or `UNKNOWN` |
  | `UK` | `UK_ONLY`, `EU_ONLY`, `EMEA`, `GLOBAL`, or `UNKNOWN` |
  | `EMEA` | `EMEA`, `EU_ONLY`, `UK_ONLY`, `GLOBAL`, or `UNKNOWN` |
  | `Europe` | `EU_ONLY`, `EMEA`, `GLOBAL`, or `UNKNOWN` |
  | `APAC` | `GLOBAL` or `UNKNOWN` |
  | `Canada` | `CANADA_ONLY`, `GLOBAL`, or `UNKNOWN` |
  | `Global` | `GLOBAL` or `UNKNOWN` |
  | Country name | `GLOBAL` or `UNKNOWN` (or extractable country-specific tag) |

  For **OnSite/Hybrid jobs** — match `location` + `secondaryLocations` against country list:
  - `USA` → "United States", "US", "USA", any US state name or abbreviation (CA, NY, TX…)
  - `UK` → "United Kingdom", "UK", "England", "Scotland", "Wales", "London", "Manchester"…
  - `EMEA` → any European, Middle Eastern, or African country or major city
  - `Europe` → EU27 + UK + Norway + Switzerland + Iceland
  - `APAC` → Australia, NZ, Japan, South Korea, Singapore, India, HK, Taiwan, China, Indonesia, Malaysia, Thailand, Philippines
  - `LATAM` → Brazil, Mexico, Argentina, Colombia, Chile, Peru, and others
  - `Global` → matches everything
  - Individual country/city name → case-insensitive substring match on location

If a filter dimension is empty/unset, skip it (match all).

**Filter summary example:**
```
Posted within:  14  (cutoff: 2026-03-01)
Work modality:  Remote
Region:         EMEA, Global
Exclude region: USA, China
```
Result: Remote jobs open to EMEA or worldwide, excluding US-only and China-only, posted within the last 14 days.

#### 3d. Deduplicate Against Search Log

- Read previous entries from `JOB_SEARCH_LOG.md`
- Use `jobUrl` as the dedup key
- Mark each match as:
  - **NEW** — not seen in any previous search run
  - **PREVIOUSLY_SEEN** — URL exists in a prior run (include the date and status from that run)

#### 3e. Present Results

Display filtered results as a numbered table:

```
Job Search Results — [date]

[N] matches found across [M] companies ([X] new, [Y] previously seen)
Sources: [N] from ATS API, [N] from web search
Filters: posted within [14d] | region [EMEA, Global] | remote only

 #  | Status | Company     | Title                  | Location             | Posted      | Type | Source
----|--------|-------------|------------------------|----------------------|-------------|------|--------
 1  | NEW    | Anthropic   | Security Engineer      | Remote (Worldwide)   | 3 days ago  | Full | web → ashby
 2  | NEW    | Anthropic   | Platform Engineer      | Remote (EU only)     | 8 days ago  | Full | web → ashby
 3  | SEEN   | Cloudflare  | Security Architect     | Remote (US only)     | 2026-02-10  | Full | ats-api
 4  | NEW    | StartupX    | Staff Security Eng     | London, UK           | ⚠️ unknown  | Full | lever

Filtered out: [N] jobs — [N] too old, [N] wrong region, [N] wrong modality
```

Notes on the results display:
- **Posted** column: show relative time ("3 days ago", "2 weeks ago") if recent; ISO date if older; "⚠️ unknown" if `postedAt` is null
- **Location** column: for Remote jobs, append `remoteRegion` in parentheses if determinable: `Remote (US only)`, `Remote (EU only)`, `Remote (Worldwide)`, `Remote (⚠️ region unknown)`
- **Filtered out** breakdown: list how many were removed by each active filter so the user can tune them

If no matches found, report and offer to adjust filters.

**Confirm with user before proceeding.**

---

### Phase 4: User Confirmation

Let the user select which matches to evaluate:

```
Which matches would you like to evaluate?

Enter numbers (e.g., "1, 2, 4"), "all new", or "none":
```

Options:
- **Specific numbers** — evaluate those matches
- **"all new"** — evaluate all NEW matches (skip PREVIOUSLY_SEEN)
- **"all"** — evaluate everything including previously seen
- **"none"** — skip evaluation, just log results

Before confirming selections, offer: `"Want to see the full JD for any match? Enter a number to preview."`

If the user requests a JD preview:
- Display the full description text for that match
- Then re-ask for evaluation selections

---

### Phase 5: Auto-Evaluate Pipeline

For each approved match, run the `/should-i-apply` evaluation pipeline. This produces the same output structure but adapts to the ATS context (JD already fetched, company research can be shared).

#### 5a. Group by Company

Group selected matches by company. This allows sharing `COMPANY_INTEL.md` across multiple roles at the same company.

#### 5b. For Each Company Group

**Company research (once per company):**
- Check if `[EVALUATIONS_DIR]/[company]-*/COMPANY_INTEL.md` already exists from a previous evaluation
  - If yes and it's less than 30 days old, reuse it
  - If yes but older, do a lighter refresh (check for recent news only)
  - If no, run full company research using both research checklists:
    - `[RESOURCES_DIR]/research-checklist.md`
    - `[RESOURCES_DIR]/interview-research-checklist.md`
- Save to `[EVALUATIONS_DIR]/[company]-[first-role-slug]/COMPANY_INTEL.md`

**For each role at this company:**

1. Create output directory: `[EVALUATIONS_DIR]/[company]-[role-slug]/`
2. Read the candidate profile
3. Use the job description already fetched from the API (Phase 3) — no need to re-fetch
4. Run the evaluation pipeline (same phases as `/should-i-apply` Phase 4 and Phase 5):

   **Generate SCORECARD.md:**
   - Overall verdict: Strong Yes / Yes / Maybe / Probably Not / No
   - Match score with justification
   - Sentiment-driven SWOT (strengths, weaknesses, opportunities, threats)
   - Requirements scorecard table
   - Company card

   **Generate INTERVIEW_PREP.md:**
   - Likely interview panel
   - Questions they'll ask (with answer frameworks from candidate experience)
   - Questions to ask them (with good/bad answer signals)
   - Conversation points to weave in
   - Red flags to watch for
   - Culture & values alignment map

   **Copy or symlink COMPANY_INTEL.md** if shared with another role at same company

   **Generate README.md:**
   - Company, role, evaluation date
   - Verdict summary
   - File descriptions
   - Source: `/job-search` auto-evaluation (with JD URL)

5. Present a brief verdict summary after each evaluation:

```
Evaluation: [Company] — [Role]
Verdict: [Strong Yes / Yes / Maybe / Probably Not / No] ([score]%)
Top strength: [one line]
Top risk: [one line]
Output: [path to evaluation directory]
```

6. **Confirm between evaluations:**

```
Continue to next evaluation? (yes / skip remaining / stop)
```

- **yes** — proceed to next match
- **skip remaining** — stop evaluating but still log all matches
- **stop** — stop entirely, log what's done so far

---

### Phase 6: Update Search Log

After all evaluations (or skips) are complete, update `[WORK_DIR]/JOB_SEARCH_LOG.md`.

#### 6a. Create or Update Log

If the file doesn't exist, create it with a header:

```markdown
# Job Search Log

Automated search history from `/job-search`. Each run is prepended below.

---
```

#### 6b. Prepend New Run Section

Prepend (not append) the new run section so the most recent search is always at the top:

```markdown
## Search Run — [YYYY-MM-DD HH:MM]

**Search mode:** [ATS / Web Discovery / Combined]
**Search parameters:**
- Companies (ATS): [list or "none"]
- Web search queries: [list or "none"]
- Role keywords: [list]
- Filters: [summary of active filters]

**Results:** [N] jobs scanned, [M] matches, [E] evaluated, [S] skipped
**Sources:** [N] ATS API, [N] web search → ATS, [N] web direct

| # | Company | Title | URL | Status | Verdict | Score | Source |
|---|---------|-------|-----|--------|---------|-------|--------|
| 1 | Anthropic | Security Engineer | [url] | EVALUATED | Yes | 78% | web → ashby |
| 2 | Anthropic | Platform Engineer | [url] | SKIPPED | — | — | web → ashby |
| 3 | Cloudflare | Security Architect | [url] | PREVIOUSLY_SEEN | — | — | ats-api |

---
```

Status values:
- **EVALUATED** — full evaluation produced (include verdict and score)
- **SKIPPED** — matched filters but user chose not to evaluate
- **PREVIOUSLY_SEEN** — appeared in a prior search run (include date first seen)
- **FILTERED_OUT** — did not match filters (only include count, not individual entries)

#### 6c. Detect Closed Listings

Compare the current API results against previous log entries for the same company:
- If a job URL from a previous run is no longer in the API response, mark it as **CLOSED** in the log
- Add a note: `"[CLOSED as of YYYY-MM-DD] — no longer on {company} job board"`
- Do NOT delete old entries — just annotate them

#### 6d. Offer Watchlist Updates

If any companies were discovered via web search or ATS discovery (not already in the watchlist), offer to add them:

```
Discovered companies not in your watchlist:
  - Anthropic → Ashby (token: anthropic) [found via web search]
  - StartupX → Greenhouse (token: startupx) [found via web search]
  - SomeCo → Lever (no bulk API — requires web search to rediscover)

Add ATS-compatible companies to your watchlist? (yes / select / no)
```

- **yes** — add all ATS-compatible companies (Ashby/Greenhouse with tokens) to the watchlist
- **select** — let the user pick which to add
- **no** — skip

Only companies with Ashby or Greenhouse boards should be added (those platforms support bulk API fetch). Lever, Workday, and custom career page companies require web search mode to rediscover — note this to the user.

---

### Phase 7: Summary Report

Present a final summary:

```
Job Search Complete — [date]

Search mode:         [ATS / Web Discovery / Combined]

Companies searched:  [N]
  From watchlist:    [N]
  From web search:   [N] (discovered)
  Successful:        [N] ([list])
  No board found:    [N] ([list])

Web search:          [if applicable]
  Queries executed:  [N]
  ATS boards found:  [N]
  Direct jobs found: [N]

Jobs scanned:        [N]
Matches found:       [N] (after filters)
  New:               [N]
  Previously seen:   [N]

Evaluations:         [N]
  Strong Yes:        [N]
  Yes:               [N]
  Maybe:             [N]
  Probably Not:      [N]
  No:                [N]

Skipped:             [N]

Top matches:
  1. [Company] — [Role] — [verdict] ([score]%) → [eval dir path]
  2. ...

Next steps:
  - /mock-interview "[company]" "[role]" — practice for your top match
  - /comp-research "[company]" "[role]" — check compensation before applying
  - /player-card "[company]" "[role]" — build a showcase site to send with your application
  - Edit WATCHLIST.md to add discovered companies or refine web search queries
  - Re-run /job-search to check for new postings

Search log: [path to JOB_SEARCH_LOG.md]
```

---

## Quality Checklist

Before finishing, verify:
- [ ] All API calls use the correct endpoints and handle errors gracefully
- [ ] Filters are applied correctly (AND across dimensions, OR within each)
- [ ] Deduplication against the search log works (jobUrl is the key)
- [ ] Evaluations produce the same quality output as `/should-i-apply`
- [ ] COMPANY_INTEL.md is reused across roles at the same company (not re-researched)
- [ ] Search log is prepended (newest first) with all matches recorded
- [ ] Closed listings are detected and annotated
- [ ] Watchlist updates are offered for newly discovered boards
- [ ] User has confirmation points between phases (no runaway automation)
- [ ] All research has source URLs
- [ ] SWOT items are specific to the candidate x company (not generic)
- [ ] Web search queries combine keywords + filter dimensions (modality, location)
- [ ] ATS board tokens are correctly extracted from web search URLs (Ashby, Greenhouse patterns)
- [ ] Web-discovered Ashby/Greenhouse companies pivot to full API fetch (not just the single posting found)
- [ ] Direct-fetch jobs have all available fields extracted; missing fields use sensible defaults
- [ ] Source tracking (`ats-api`, `web-search`, `web-search → ashby`, etc.) is recorded for every job
- [ ] At most 20 direct-fetch URLs processed per run (unless configured otherwise in watchlist)
- [ ] Web-discovered ATS companies offered for watchlist addition
