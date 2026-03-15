# Claude Job Hunter

AI-powered job search toolkit using [Claude Code](https://docs.anthropic.com/en/docs/claude-code) slash commands. Fourteen skills that cover the full job search lifecycle — from profile building to applying with a tailored LaTeX resume and cover letter:

- **`/build-profile`** — Generate your candidate profile from resume, GitHub (skills verified against real code), LinkedIn, and supporting evidence
- **`/job-search`** — Search ATS boards and the web for matching roles, filter, and auto-evaluate matches
- **`/should-i-apply`** — Evaluate WHETHER you should apply, with a scorecard, sentiment-driven SWOT, and deep interview prep
- **`/ats-check`** — Audit resume vs. JD for ATS compatibility: keyword coverage score, formatting safety, section quality, prioritized fix list
- **`/tailor-resume`** — Generate an ATS-optimized tailored resume in LaTeX format; accepts job via URL or pasted JD text
- **`/tailor-cover-letter`** — Generate a tailored cover letter in LaTeX format; hook → why them → why you → CTA structure
- **`/apply`** — Full application orchestrator: ingest job (URL or paste), ATS check, tailor resume + cover letter, save to `applications/[company-role]/`, track status
- **`/skill-gap`** — Build an ideal candidate profile, benchmark your skills against it, and create a time-based development plan
- **`/mock-interview`** — Practice interview loops with researched questions, written answers, and critical feedback on content and communication
- **`/interview-feedback`** — Debrief after each real interview round with signal analysis, performance assessment, and next-round strategy
- **`/comp-research`** — Build an objective compensation picture: market data, leveling, peer benchmarks, geography adjustments
- **`/offer-negotiation`** — Evaluate an offer, build negotiation strategy with scripts, and decide whether to accept, counter, or walk
- **`/swot-analysis`** — Research a company's strategic position, market health, and trajectory — pure company intelligence before evaluating fit
- **`/player-card`** — Generate a password-protected Cloudflare Worker site that showcases you TO a company (deployed, shareable)

All commands use live web research to produce outputs specific to you and the target company — no generic templates.

## Quick Start

### 1. Clone and configure

```bash
git clone https://github.com/deeso/claude-job-hunter.git ~/claude-job-hunter
```

### 2. Install the slash commands

Bootstrap the setup command, then let it install everything else:

```bash
mkdir -p ~/.claude/commands
ln -sf ~/claude-job-hunter/commands/setup.md ~/.claude/commands/setup.md
```

Then open Claude Code and run:

```
# Use the repo directory for everything (simple)
/setup

# Or specify a custom working directory for your personal data
/setup "~/job-search"
```

This symlinks all skills into `~/.claude/commands/`, creates output directories, and writes a config file (`~/.claude/.claude-job-hunter.conf`) so all skills know where to find your data. You only need to do this once (or again after pulling updates that add new commands).

To remove everything later, run `/uninstall` — it removes symlinks and config but leaves your personal data intact.

Using a custom working directory keeps your personal data (profile, evaluations, cards) separate from the repo — useful if you want to keep the repo clean for `git pull` updates.

<details>
<summary>Manual installation (alternative)</summary>

```bash
mkdir -p ~/.claude/commands
ln -sf ~/claude-job-hunter/commands/build-profile.md ~/.claude/commands/build-profile.md
ln -sf ~/claude-job-hunter/commands/job-search.md ~/.claude/commands/job-search.md
ln -sf ~/claude-job-hunter/commands/should-i-apply.md ~/.claude/commands/should-i-apply.md
ln -sf ~/claude-job-hunter/commands/ats-check.md ~/.claude/commands/ats-check.md
ln -sf ~/claude-job-hunter/commands/tailor-resume.md ~/.claude/commands/tailor-resume.md
ln -sf ~/claude-job-hunter/commands/tailor-cover-letter.md ~/.claude/commands/tailor-cover-letter.md
ln -sf ~/claude-job-hunter/commands/apply.md ~/.claude/commands/apply.md
ln -sf ~/claude-job-hunter/commands/mock-interview.md ~/.claude/commands/mock-interview.md
ln -sf ~/claude-job-hunter/commands/interview-feedback.md ~/.claude/commands/interview-feedback.md
ln -sf ~/claude-job-hunter/commands/comp-research.md ~/.claude/commands/comp-research.md
ln -sf ~/claude-job-hunter/commands/offer-negotiation.md ~/.claude/commands/offer-negotiation.md
ln -sf ~/claude-job-hunter/commands/player-card.md ~/.claude/commands/player-card.md
ln -sf ~/claude-job-hunter/commands/uninstall.md ~/.claude/commands/uninstall.md
```
</details>

### 3. Build your candidate profile

Run `/build-profile` to generate your profile automatically from your resume, GitHub, and evidence:

```
/build-profile "~/resume.pdf" "~/evidence/" "https://linkedin.com/in/your-profile" "your-github-username"
```

Or edit your profile (e.g., `profile/default.md`) manually using `profile/candidate-profile.md` as the template. Your profile is the core data source all commands use. The more honest and specific you are (especially the "Honest Considerations" section), the better the outputs.

### 4. Use them

Open Claude Code and run:

```
# First time? Build your profile from your resume + GitHub
/build-profile "~/resume.pdf" "~/evidence/" "https://linkedin.com/in/you" "your-github"

# Search job boards and auto-evaluate matches
/job-search "Anthropic, Cloudflare" "security, platform"

# Or search all watchlist companies with no args
/job-search

# Or search the web by role keywords — no company names needed
/job-search "" "security engineer, platform security"

# Before applying — should I bother?
/should-i-apply "~/resume.pdf" "Anthropic" "Security Engineer" "https://job-url.com" "~/evidence/"

# ATS audit: keyword coverage, formatting safety, score (0-100)
/ats-check "~/resume.pdf" "https://job-url.com"

# Or paste JD text directly
/ats-check "~/resume.pdf" "Senior Security Engineer at Anthropic, requires Rust, Python..."

# Generate a tailored resume in LaTeX (URL or pasted JD)
/tailor-resume "~/resume.pdf" "https://job-url.com"

# Generate a tailored cover letter in LaTeX
/tailor-cover-letter "" "https://job-url.com"

# Full application in one command (URL or pasted JD text)
/apply "https://job-url.com"
/apply "Senior Security Engineer at Anthropic, requires Rust..."

# Practice before the real thing
/mock-interview "Anthropic" "Security Engineer" "technical"

# After each real interview round — how did it go?
/interview-feedback "Anthropic" "Security Engineer" "Jane Smith" "VP of Engineering"

# Know your market value before the offer comes
/comp-research "Anthropic" "Security Engineer" "Staff"

# Offer in hand — what's the play?
/offer-negotiation "Anthropic" "Security Engineer"

# Research a company before looking at specific roles
/swot-analysis "Anthropic"

# Focus on a specific area
/swot-analysis "Anthropic" "security"

# Not ready yet? Benchmark and build a development plan
/skill-gap "Anthropic" "Staff Security Engineer" "https://job-url.com" "1 year"

# Ready to send something impressive — build the card
/player-card "Anthropic" "Security Engineer" "https://job-url.com"
```

## Commands

### `/build-profile`

**Purpose:** Generate your candidate profile automatically instead of filling in the template by hand. The agent extracts, researches, and synthesizes — you validate and add the human context.

**Usage:** `/build-profile "resume-path" "evidence-dir" "linkedin-url" "github-usernames" "profile-name"`

Arguments 2-5 are optional. Multiple GitHub usernames can be comma-separated (e.g., personal + work accounts). Profile name defaults to `default` (see [Multi-Profile Support](#multi-profile-support)).

**How it works:**
1. **Ingests all materials** — reads resume, evidence directory (cover letters, publications, project docs, LinkedIn export, recommendations, performance reviews), catalogs what it found
2. **GitHub deep analysis** — uses `gh` CLI to analyze repos, languages, activity, stars, contribution patterns. Categorizes repos into showcase (impressive), evidence (backs resume claims), and contributions (open source engagement). Cross-references technical skills claims with actual code.
3. **Deep web research** — searches for your public presence:
   - Google Scholar (papers, citations, h-index)
   - USPTO/Google Patents (full patent details)
   - npm/PyPI/crates.io packages authored
   - Conference talks, blog posts, press mentions
   - LinkedIn, Twitter/X, personal website
4. **Candidate interview** — asks the things only you know:
   - Career narrative and why you made each move
   - Biggest impact stories with quantified evidence
   - Top technical skills (what you're truly deep in vs. what's on the resume)
   - **The honest section** — real weaknesses, what drains you, deal-breakers, what a critical reference would say, tenure risks
   - Career goals and what you're optimizing for
5. **Synthesizes the profile** — builds `candidate-profile.md` with GitHub showcase repos, verified publications, and all evidence
6. **Review** — walks through each section for your approval

Supports **update mode**: if a profile exists, merges new information and re-runs web research to catch recent publications, patents, or mentions.

### `/job-search`

**Purpose:** Search for matching roles across ATS boards (Ashby, Greenhouse) and the open web — with or without specifying companies.

**Usage:** `/job-search "company1, company2" "role-keywords" "profile-name"`

All arguments are optional. Three search modes:
- **ATS mode** — specify companies, search their Ashby/Greenhouse boards
- **Web discovery mode** — pass role keywords with no companies (`/job-search "" "security engineer"`), finds jobs across the web
- **Combined mode** — enable `## Web Search` in your watchlist to always run both

```
# Search specific companies
/job-search "Anthropic, Cloudflare" "security"

# Search the web by keywords — no company names needed
/job-search "" "security engineer, platform security"

# Search everything in your watchlist
/job-search
```

**How it works:**
1. **Web discovery** (if no companies) — searches the web for job postings matching your keywords, extracts company board tokens from Ashby/Greenhouse URLs to unlock full API data
2. **Discovers ATS platforms** — tries Ashby and Greenhouse APIs with token variants, falls back to WebSearch for the careers page
3. **Fetches & filters all jobs** — normalizes API and web-scraped results, applies filters (role keywords, department, location, work modality, employment type) with AND across dimensions, OR within each
4. **Deduplicates** — checks against your search log so you only see new postings
5. **You choose** — review the results table, preview full JDs, select which to evaluate
6. **Auto-evaluates** — runs the full `/should-i-apply` pipeline (SCORECARD, INTERVIEW_PREP, COMPANY_INTEL) for each approved match, reusing company research across roles
7. **Logs everything** — tracks what you've seen, evaluated, and skipped; detects closed listings; offers to add web-discovered companies to your watchlist

**Produces:**
- Same evaluation outputs as `/should-i-apply` for each evaluated match
- `WATCHLIST.md` — your saved companies and filter preferences (created from template on first run)
- `JOB_SEARCH_LOG.md` — running history of searches, matches, and statuses

**Watchlist:** Edit `WATCHLIST.md` to save your target companies, board tokens, and filter preferences. Global filters apply to all companies; per-company overrides let you customize. Set platform to `auto` for automatic ATS discovery. Enable the `## Web Search` section to discover new companies automatically via web search.

### `/should-i-apply`

**Purpose:** Decide if a role is worth pursuing before investing time in applications.

**Usage:** `/should-i-apply "resume-path" "company" "role" "jd-url" "evidence-dir"`

Arguments 4 and 5 are optional. The evidence directory can contain cover letters, portfolio pieces, publications, project docs, LinkedIn exports — anything that strengthens your profile.

**Produces:**
```
evaluations/[company]-[role-slug]/
├── SCORECARD.md       # Verdict (Strong Yes → No), SWOT, requirements mapping
├── INTERVIEW_PREP.md  # Likely panel, Q&A with answer frameworks, red flags
├── COMPANY_INTEL.md   # Sourced research on company, leadership, culture
└── README.md          # Summary and usage guide
```

**Key features:**
- Sentiment-driven SWOT: failure risks and success criteria based on YOUR priorities, not generic advice
- Interview prep with specific answer frameworks drawn from YOUR experience
- "Questions to ask them" with what good and bad answers look like
- Culture & values alignment map between you and the company

### `/ats-check`

**Purpose:** Audit a resume against a specific job description for ATS (Applicant Tracking System) compatibility before applying. Run before tailoring to know what to fix, or after to verify the score improved.

**Usage:** `/ats-check "resume-path" "jd-url-or-text" "profile-name"`

Accepts job as either a **URL** (fetched live) or **pasted JD text** directly.

**How it works:**
1. **Keyword extraction** — pulls all JD keywords into Tier 1 (required, eliminates if missing), Tier 2 (preferred, score boost), Tier 3 (context/domain fluency)
2. **Coverage scoring** — marks each keyword ✅ Present / 🔄 Implied (you have the skill but didn't name it) / ❌ Missing
3. **Formatting safety audit** — checks for ATS killers: multi-column layout, tables, text boxes, headers/footers with important info, non-standard section names, inconsistent dates
4. **Section quality check** — summary keywords, bullet action verbs, quantified outcomes, dedicated skills section
5. **Scores 0–100** with verdict: HIGH RISK / AT RISK / ADEQUATE / STRONG
6. **Prioritized fix list** with exact before/after rewrites for every issue

**Produces:** `ATS_CHECK.md` in the evaluation or application directory.

### `/tailor-resume`

**Purpose:** Generate an ATS-optimized, role-specific resume in LaTeX format — not a generic template fill, but a fully tailored document where every bullet, keyword, and summary sentence is targeted at the specific JD.

**Usage:** `/tailor-resume "resume-path" "jd-url-or-text" "profile-name" "output-dir"`

Accepts job as either a **URL** or **pasted JD text**.

**How it works:**
1. Extracts all JD keywords and maps each to the candidate's actual experience (DIRECT / RENAME / DEMONSTRATE / GAP)
2. Generates tailored summary, skills section, experience bullets, and projects — all seeded with Tier 1 keywords
3. Populates `resources/resume-template.tex` with candidate-specific content (no fake data — only what's in the profile)
4. Runs ATS self-check to verify Tier 1 coverage improved
5. Saves `resume-tailored.tex` + `TAILORING_NOTES.md` (what changed and why)

**Output:** `[application-dir]/resume-tailored.tex` + compile instructions (`pdflatex resume-tailored.tex`).

Requires TeX Live / MacTeX to compile: `brew install --cask mactex-no-gui` or upload to overleaf.com.

### `/tailor-cover-letter`

**Purpose:** Generate a tailored cover letter in LaTeX format. Four-paragraph structure: specific hook → why them (one real detail) → why you (evidence-backed achievements) → confident CTA.

**Usage:** `/tailor-cover-letter "resume-path" "jd-url-or-text" "profile-name" "output-dir"`

Accepts job as either a **URL** or **pasted JD text**.

**Key rules enforced:**
- No "I am writing to apply for..." opener
- Paragraph 2 must reference something real and specific about the company (product decision, engineering blog post, public architecture choice) — never generic praise
- Paragraph 3 uses actual achievements with metrics, not generic claims
- Reads `COMPANY_INTEL.md` and `SCORECARD.md` if they exist for research context

**Output:** `[application-dir]/cover-letter.tex` + compile instructions.

### `/apply`

**Purpose:** Full application workflow in one command. Ingest a job (URL or pasted JD text), run the ATS audit, tailor the resume, generate the cover letter, save all artifacts to a structured application directory, and track status.

**Usage:** `/apply "jd-url-or-text" "profile-name" "resume-path"`

Accepts job as either a **URL** or **pasted JD text** — this is the primary input.

**Produces:**
```
applications/[company-role]/
├── JD.md                    # Saved job description
├── ATS_CHECK.md             # ATS audit (pre-tailoring score)
├── resume-tailored.tex      # ATS-optimized LaTeX resume
├── cover-letter.tex         # Tailored LaTeX cover letter
└── APPLICATION.md           # Status tracker + log
```

**Application status lifecycle:** `PREPARING` → `READY` → `APPLIED` → `PHONE_SCREEN` → `INTERVIEWING` → `OFFER` → `ACCEPTED / REJECTED / WITHDRAWN`

The quick fit scan at the start warns you if Tier 1 keyword coverage is below 50% before investing time in the full application.

### `/skill-gap`

**Purpose:** Build an ideal candidate profile for a target role, benchmark your current skills against it, and create a time-based development plan to close the gaps.

**Usage:** `/skill-gap "company" "role" "jd-url" "timeline" "profile-name"`

Arguments 1 and 3-5 are optional. Company can be a specific name (`"Anthropic"`), a category (`"FAANG"`, `"AI startup"`), or omitted for industry-wide benchmarks. Timeline defaults to 1 year.

**How it works:**
1. **Researches the ideal candidate** — analyzes 5-10 similar JDs, career ladders, interview patterns, and LinkedIn profiles to build a market-grounded ideal profile
2. **Benchmarks your skills** — maps your profile against the ideal, incorporating interview/mock interview performance if available (most honest signal)
3. **Gap analysis** — categorizes gaps as quick wins (< 3 months), development priorities (3-6 months), strategic investments (6-12 months), or unrealistic (honest about what can't be closed)
4. **Development plan** — time-blocked activities matched to your learning style and time budget: projects, courses, certifications, portfolio pieces, speaking, networking
5. **Portfolio strategy** — defines what your GitHub, writing, and speaking profile should look like by the end of the timeline

**Produces:** `SKILL_GAP.md` in the evaluation directory:
- Ideal candidate profile (market-grounded, not generic)
- Benchmark scorecard with gap ratings and evidence
- Prioritized gap analysis with honest timeline feasibility
- Time-blocked development plan with specific resources (course names, book titles, project ideas)
- Portfolio strategy and progress tracking milestones

Supports **re-run mode**: re-run to track progress, compare benchmarks over time, and adjust the plan.

### `/swot-analysis`

**Purpose:** Research a company's strategic position and produce a comprehensive SWOT analysis — before you even look at specific roles. Understand whether a company is worth your time.

**Usage:** `/swot-analysis "company" "focus-area"`

Argument 2 is optional. Focus areas: `"engineering"`, `"security"`, `"AI/ML"`, `"leadership"`, `"product"`, `"financial"`, or any custom lens.

**Researches:**
- **Business & financial health** — funding, revenue, burn rate, valuation trajectory
- **Product & market position** — competitive landscape, market share, product quality signals
- **Technology & engineering** — tech stack, eng blog, open source, technical debt signals
- **People & culture** — Glassdoor/Blind patterns, leadership quality, hiring velocity, layoff history
- **Recent news** — last 12 months of announcements, incidents, leadership changes

**Produces:** `SWOT_ANALYSIS.md` in `evaluations/[company]/`:
- Evidence-backed SWOT with durability, severity, probability, and impact ratings
- Company health scorecard (1-10 across 8 dimensions with trend indicators)
- Trajectory assessment (Rocketship / Steady Climber / Plateau / Crossroads / Declining / Turnaround)
- Job seeker verdict: who thrives here, who should avoid, ideal timing, watch signals

This is NOT a candidate-fit analysis (that's `/should-i-apply`). This is pure company intelligence.

### `/mock-interview`

**Purpose:** Practice interview loops with researched questions before the real thing. Get critical feedback on both what you say and how you say it.

**Usage:** `/mock-interview "company" "role" "round-type"`

Argument 3 is optional. Round types: `"phone-screen"`, `"technical"`, `"behavioral"`, `"hiring-manager"`, `"cross-functional"`, `"executive"`, or `"full-loop"` (default).

**How it works:**
1. Researches the company's interview style (Glassdoor reviews, reported questions, values/leadership principles, tech stack)
2. Builds a panel of 4-6 realistic interviewers with distinct personalities and evaluation focuses
3. Runs each round one question at a time — you answer in writing, the agent asks follow-up probes in character
4. After each round, provides detailed feedback before moving to the next

**Feedback covers:**
- **Content assessment** — what was strong, what was missing, specific evidence from your experience you should have included
- **Communication assessment** — structure, conciseness, specificity, confidence signals, storytelling quality
- **Stronger answer frameworks** — rewritten outlines using YOUR actual experience as a structural model
- **Skills heat map** — across all rounds, where you're strong and where to focus practice
- **Communication style report** — recurring habits to build and break, word/phrase audit
- **Practice plan** — prioritized topics, stories to prepare (with STAR outlines from your profile), rounds to re-run

Supports re-run mode: practice the same round type with fresh questions, with feedback comparing against prior attempts.

### `/interview-feedback`

**Purpose:** Structured post-interview debrief that reads the signals and prepares you for the next round.

**Usage:** `/interview-feedback "company" "role" "interviewer-name" "interviewer-title"`

Arguments 3 and 4 are optional (the agent will ask during debrief).

**How it works:** The agent asks you structured questions about:
- What questions the interviewer asked and how you answered
- Interviewer behavior — engagement, body language, selling signals
- Red flags and green flags you observed
- What you learned about the role, team, and culture
- How the interview ended and what next steps were discussed

**Produces:** Appends to `INTERVIEW_LOG.md` in the evaluation directory:
- **Signal analysis** — reads interviewer behavior to assess likelihood of advancing
- **Performance assessment** — rates each answer with "what a stronger answer looks like" for weak spots
- **Preparation gap analysis** — compares against `INTERVIEW_PREP.md` predictions if available
- **Culture & fit updates** — revises SWOT based on new information from the interview
- **Next round strategy** — concrete topics, stories, and questions to prepare
- **Thank-you note draft** — personalized to the conversation, not generic

Builds a running log across rounds with cross-round pattern analysis.

### `/comp-research`

**Purpose:** Build an objective compensation picture before an offer arrives so you negotiate from data, not feelings.

**Usage:** `/comp-research "company" "role" "level"`

Argument 3 is optional (the agent determines likely level from your profile).

**Researches:**
- **Market data** — levels.fyi, Glassdoor, H1B/LCA filings, job posting salary ranges
- **Company-specific** — comp philosophy, pay bands, equity structure (options vs RSUs), benefits
- **Peer benchmarks** — 5-8 companies competing for the same talent with comp comparison table
- **Geography** — cost of living adjustments, remote pay policies, state tax implications
- **Leveling** — maps your experience to the company's framework, flags down-leveling risk

**Produces:** `COMP_RESEARCH.md` in the evaluation directory:
- Compensation bands: below market / market rate / above market / stretch target
- Component breakdown: base, equity, signing, bonus, benefits with specific ranges
- Current comp vs. market comparison table
- Company negotiation intelligence (how they negotiate, flexibility patterns)

### `/offer-negotiation`

**Purpose:** You have an offer. Decide whether to accept, negotiate, or walk — then build the actual negotiation with scripts and counter-offer language.

**Usage:** `/offer-negotiation "company" "role"`

**How it works:** The agent walks you through:
- Capturing every offer component (base, equity, signing, bonus, benefits, clawback, non-compete)
- **Emotional check-in** — do you actually want this job? What's your alternative? What makes you say yes or walk away?
- Offer analysis against market data, current comp, and other offers
- Leveling check (were you down-leveled?)
- Hidden comp factors (equity tax treatment, exercise windows, clawback terms)

**Produces:** `NEGOTIATION_STRATEGY.md` in the evaluation directory:
- **Verdict:** Accept / Negotiate / Walk Away with reasoning
- **Priority-ordered negotiation targets** with justification and fallback positions
- **Ready-to-send counter-offer email** with specific numbers and framing
- **Phone script** and **pushback responses** for each ask
- **Decision framework** — weighted scorecard reflecting YOUR priorities
- **Scenario analysis** — accept now / negotiate-win / negotiate-hold / walk away

Reads prior skills' outputs (`COMP_RESEARCH.md`, `INTERVIEW_LOG.md`, `SCORECARD.md`) to inform strategy.

### `/player-card`

**Purpose:** Generate a deployable site that showcases you to a specific company.

**Usage:** `/player-card "company" "role" "jd-url"`

**Produces:** A complete Cloudflare Worker project with:
- Password-protected single-page site with company-specific pitch
- Embedded resume PDF (no external storage)
- Email alerts when someone views your card (via Resend)
- Custom domain routing (`company.yourdomain.com/player-card`)

**Requires:** Cloudflare account, custom domain, Resend API key.

## Repository Structure

**Repo** (cloned, shared, git-managed):
```
claude-job-hunter/
├── commands/                        # Claude Code slash commands
│   ├── setup.md                     # /setup — installs everything
│   ├── uninstall.md                 # /uninstall — removes everything
│   ├── build-profile.md             # /build-profile
│   ├── job-search.md                # /job-search
│   ├── should-i-apply.md            # /should-i-apply
│   ├── ats-check.md                 # /ats-check
│   ├── tailor-resume.md             # /tailor-resume
│   ├── tailor-cover-letter.md       # /tailor-cover-letter
│   ├── apply.md                     # /apply
│   ├── mock-interview.md            # /mock-interview
│   ├── interview-feedback.md        # /interview-feedback
│   ├── comp-research.md             # /comp-research
│   ├── offer-negotiation.md         # /offer-negotiation
│   ├── player-card.md               # /player-card
│   ├── skill-gap.md                 # /skill-gap
│   └── swot-analysis.md             # /swot-analysis
├── resources/
│   ├── resume-template.tex          # ATS-safe LaTeX resume base
│   ├── cover-letter-template.tex    # LaTeX cover letter base
│   └── ...                          # Research checklists and references
├── examples/                        # Example outputs
└── profile/                         # Profile template (user profiles are gitignored)
```

**Working directory** (your personal data — defaults to repo dir, or custom):
```
[work-dir]/
├── .claude-job-hunter.conf          # Config file (generated by /setup)
├── WATCHLIST.md                     # /job-search target companies and filters
├── JOB_SEARCH_LOG.md               # /job-search run history
├── profile/
│   ├── candidate-profile.md         # Template (read-only reference)
│   ├── default.md                   # Your default profile
│   ├── security.md                  # Optional: security-focused profile
│   └── leadership.md               # Optional: leadership-focused profile
├── evaluations/                     # /should-i-apply, /job-search, etc.
│   └── [company]-[role]/
│       ├── SCORECARD.md
│       ├── INTERVIEW_PREP.md
│       ├── COMPANY_INTEL.md
│       ├── INTERVIEW_LOG.md
│       ├── COMP_RESEARCH.md
│       ├── NEGOTIATION_STRATEGY.md
│       ├── MOCK_INTERVIEW.md
│       ├── SKILL_GAP.md
│       ├── SWOT_ANALYSIS.md
│       └── ATS_CHECK.md
├── applications/                    # /apply outputs
│   └── [company]-[role]/
│       ├── JD.md
│       ├── ATS_CHECK.md
│       ├── resume-tailored.tex
│       ├── cover-letter.tex
│       └── APPLICATION.md
└── cards/                           # /player-card outputs
    └── [company]-[year]/
```

## How It Works

The commands cover the full job search lifecycle:

1. **`/build-profile`** — Onboarding: build your candidate profile from resume, GitHub (skills verified against real code), and LinkedIn
2. **`/job-search`** — Discovery: search ATS boards and the web, filter, and batch-evaluate matches
3. **`/swot-analysis`** — Company research: strategic SWOT of the company itself before evaluating fit
4. **`/should-i-apply`** — Pre-application: research, SWOT analysis, interview prep (single role)
5. **`/ats-check`** — Application prep: keyword coverage audit, formatting safety check, 0-100 score
6. **`/tailor-resume`** — Application prep: ATS-optimized LaTeX resume tailored to the specific JD
7. **`/tailor-cover-letter`** — Application prep: tailored LaTeX cover letter with company-specific research
8. **`/apply`** — Application: full workflow orchestrator — ingest job, ATS check, tailor resume + cover letter, save and track
9. **`/skill-gap`** — Development: benchmark skills against ideal, build a time-based plan to close gaps
10. **`/mock-interview`** — Pre-interview: practice with researched questions, get feedback
11. **`/interview-feedback`** — During process: debrief each round, read signals, prep for next
12. **`/comp-research`** — Pre-offer: build objective comp picture from market data
13. **`/offer-negotiation`** — Offer stage: evaluate, strategize, negotiate with scripts
14. **`/player-card`** — Anytime: deploy a personalized showcase site

Each command follows a multi-phase workflow with user confirmation between phases. Evaluations accumulate in `evaluations/[company]-[role]/`; applications save to `applications/[company-role]/`.

## Multi-Profile Support

You can maintain multiple profiles tailored to different role types (e.g., security-focused, leadership-focused, IC-focused). All profiles live in the `profile/` directory alongside the `candidate-profile.md` template.

### Creating profiles

```bash
# Build a named profile from your resume
/build-profile "~/resume.pdf" "" "" "" "security"

# Build another profile with a different emphasis
/build-profile "~/resume.pdf" "~/leadership-evidence/" "" "" "leadership"
```

### Using profiles

Every command accepts an optional profile name as its last argument:

```bash
/should-i-apply "~/resume.pdf" "Anthropic" "Security Engineer" "" "" "security"
/player-card "anthropic" "Security Engineer" "https://..." "security"
/mock-interview "Anthropic" "Security Engineer" "technical" "security"
```

### Setting a default

Edit `~/.claude/.claude-job-hunter.conf` and set:

```
ACTIVE_PROFILE=security
```

All commands will use `profile/security.md` without needing the profile argument.

### Resolution order

When a command needs your profile, it checks:

1. **Explicit argument** — profile name passed to the command
2. **Config default** — `ACTIVE_PROFILE` in `~/.claude/.claude-job-hunter.conf`
3. **Auto-detect** — scans `profile/` for `.md` files (excluding the template):
   - One profile found → uses it automatically
   - Multiple found → asks you to pick
   - None found → prompts you to run `/build-profile`

### File layout

```
profile/
├── candidate-profile.md    # Template (tracked in git, read-only reference)
├── default.md              # Your default profile (gitignored)
├── security.md             # Security-focused variant (gitignored)
└── leadership.md           # Leadership-focused variant (gitignored)
```

User profiles (`*.md` except `candidate-profile.md`) are gitignored to keep personal data out of the repo.

### Migration

If you created your profile before multi-profile support, running `/setup` again will automatically migrate `candidate-profile.md` → `default.md` and restore the template.

## Tips

- **Start with `/build-profile`.** Let the agent build your profile from your resume and web presence. The candidate interview phase asks the hard questions that make everything else work. Create multiple profiles for different role types if needed.
- **Use `/job-search` for batch discovery.** Set up your watchlist with target companies and filters, then run periodically to catch new postings. It deduplicates across runs so you only see what's new.
- **Be honest in your candidate profile.** The "Honest Considerations" section makes SWOT analysis genuinely useful. Generic strengths produce generic outputs.
- **Use the evidence directory.** The more context you give (cover letters, project docs, publications), the stronger the evidence mapping in scorecards and interview prep.
- **Review between phases.** Both commands pause for confirmation. Add context the AI might have missed.
- **Practice before real interviews.** Run `/mock-interview` to identify weak spots. Re-run specific round types until your answers are tight. The communication feedback is where the real value is.
- **Debrief every round.** Run `/interview-feedback` after each real interview while details are fresh. The running log across rounds reveals patterns.
- **Run comp research early.** `/comp-research` before the offer means you negotiate from data, not gut feelings. The research feeds directly into `/offer-negotiation`.
- **Not ready yet?** Run `/skill-gap` to benchmark your skills against a target role and build a development plan. Re-run periodically to track progress.
- **Full lifecycle.** `/build-profile` → `/job-search` or `/swot-analysis` → `/should-i-apply` → `/ats-check` → `/apply` (tailors resume + cover letter + tracks) → `/skill-gap` (if gaps found) → `/mock-interview` → `/interview-feedback` (per round) → `/comp-research` → `/offer-negotiation` → `/player-card` if you want to send something impressive.

## Capabilities & Known Limitations

This toolkit is a research and document-generation assistant — not a fully automated job application bot. Understanding what it does and doesn't do well will help you use it effectively.

### What works well

| Capability | Confidence | Notes |
|---|---|---|
| Profile building from resume + GitHub | High | Deep `gh` CLI repo analysis, web research, structured candidate interview |
| ATS keyword audit | High | Follows real ATS mechanics — keyword tiers, formatting rules, 0–100 score with fix list |
| Resume tailoring (LaTeX) | High | Keyword mapping + template population; quality scales with profile detail |
| Cover letter generation | High | Hook → why them → why you → CTA; requires genuine company research detail |
| Should-I-Apply analysis | High | Sentiment-driven SWOT with failure risks tied to your stated priorities |
| Interview prep | High | Company-researched Q&A with answer frameworks from your actual experience |
| Job search (Ashby/Greenhouse) | Good | Public APIs are stable and well-documented; returns all open roles per board |
| Date + region filtering | Good | Post-fetch filtering logic is correct; `remoteRegion` inference is heuristic |
| Comp research | Moderate | Relies on scraped data (levels.fyi, Glassdoor) — quality depends on data availability |

### Known gaps and limitations

**LinkedIn job search is unreliable.** LinkedIn actively blocks automated access. Web search queries for `site:linkedin.com/jobs` find some URLs but not systematically. Treat LinkedIn results as supplementary, not primary.

**Greenhouse posting dates are approximate.** The public Greenhouse API returns `updated_at`, not the original publish date. A job updated today may have been posted months ago. The `Posted within` filter will include false positives for Greenhouse boards.

**PDF resume parsing has limits.** Claude extracts text from PDFs, but complex layouts (multi-column, tables, custom fonts, embedded graphics) may produce garbled or incomplete extraction. Plain-text or `.tex` source files give the best results.

**LaTeX requires local compilation.** Resume and cover letter outputs are `.tex` source files. You need TeX Live / MacTeX installed (`brew install --cask mactex-no-gui`) or an online compiler like [Overleaf](https://overleaf.com) to produce PDFs.

**GitHub skill depth is inferred, not measured.** The profile builder cross-references claimed skills against repo languages and topics, but it can't distinguish "wrote 5 lines once" from "built a production service." Star count and commit frequency are weak proxies for proficiency.

**No application submission.** The `/apply` command prepares a complete application package but cannot submit it. ATS portals (Greenhouse, Workday, Lever) require browser interaction, auth, and form-filling. You still upload the PDF and fill the fields manually.

**Rate limiting on ATS APIs.** Ashby and Greenhouse public APIs are unauthenticated and may throttle aggressive bulk fetching. Large watchlists (20+ companies) may hit limits during a single `/job-search` run.

### Best use of this toolkit

The highest-value workflow is: **`/build-profile` → `/ats-check` → `/tailor-resume` → `/apply`** for a specific target role. The ATS audit + tailored LaTeX resume is the strongest piece — most applicants skip this entirely and wonder why they don't hear back.

Job *discovery* via `/job-search` is useful for watchlist companies with Ashby/Greenhouse boards but treat it as a research assistant, not a fully automated pipeline.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- For `/player-card` deployment: Cloudflare account, Wrangler CLI, custom domain, Resend API key
- For LaTeX resume/cover letter compilation: TeX Live or MacTeX (`brew install --cask mactex-no-gui`), or use [Overleaf](https://overleaf.com)
- For all other commands: no infrastructure needed — outputs are local markdown files
