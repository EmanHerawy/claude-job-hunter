# ATS Check — Resume ATS Compatibility Audit

Audit a resume against a specific job description for ATS (Applicant Tracking System) compatibility. Scores the resume on keyword coverage, formatting safety, and section structure. Produces a prioritized fix list with exact rewrites. Run before `/tailor-resume` to understand what needs fixing, or after to verify the tailored version passes.

## Inputs

**Usage:** `/ats-check "resume-path" "jd-url-or-text" "profile-name"`

Parse `$ARGUMENTS` as up to three quoted strings:
- **First quoted string** (optional) — Path to resume file (PDF, markdown, or `.tex`). If omitted, looks for most recent tailored resume in `[EVALUATIONS_DIR]/[active-eval]/` or `[APPLICATIONS_DIR]/`.
- **Second quoted string** (optional) — URL to job description, or paste JD text directly. If omitted, looks for `JD.md` in the current evaluation directory.
- **Third quoted string** (optional) — Profile name.

## Setup

Resolve paths from `~/.claude/.claude-job-hunter.conf`. Required:
- `WORK_DIR`, `PROFILE_DIR`, `EVALUATIONS_DIR`, `APPLICATIONS_DIR`

If config missing, ask user to run `/setup`.

## What ATS Systems Actually Do

Before running the audit, understand the actual mechanism being tested:

1. **Keyword extraction** — ATS parses the JD and resume, scores keyword overlap. Missing required keywords = filtered out regardless of experience.
2. **Section recognition** — ATS looks for standard section headings (Experience, Education, Skills). Non-standard headings (e.g. "Where I've Worked") may be ignored.
3. **Parsing failures** — Tables, text boxes, headers/footers, graphics, and multi-column layouts cause text to be read in wrong order or dropped entirely.
4. **File format** — DOCX and simple PDFs parse best. Complex LaTeX PDFs with custom fonts or embedded graphics can fail silently.
5. **Date parsing** — ATS extracts employment dates to calculate tenure. Non-standard formats cause misreads.
6. **Contact parsing** — ATS extracts name, email, phone, location. These must be in the main body — not in a header/footer or text box.

---

## Phase 1: Read and Parse

1. Read the resume file (if PDF, extract text; if `.tex`, read source directly)
2. Fetch and read the job description from the URL (or read pasted text)
3. Read the candidate profile from `[PROFILE_DIR]/[resolved-name].md`
4. Read the profile's skills section for cross-reference

---

## Phase 2: JD Keyword Extraction

Extract all keywords from the JD in three tiers:

**Tier 1 — Required (eliminate if missing):**
- Skills explicitly listed under "Required", "Must have", "Qualifications"
- Skills mentioned 3+ times in the JD
- Exact job title words
- Specific technologies, frameworks, certifications named

**Tier 2 — Preferred (score boost if present):**
- Skills under "Nice to have", "Preferred", "Bonus"
- Domain vocabulary used throughout the JD
- Soft skills the role emphasizes (if repeated: e.g. "cross-functional collaboration")

**Tier 3 — Context keywords (show domain fluency):**
- Industry terms used in the company description
- Product/technology names mentioned
- Methodology terms (agile, zero-trust, CI/CD, etc.)

Output the extracted keyword lists before proceeding.

---

## Phase 3: Keyword Coverage Score

For each keyword, check whether it appears in the resume. Mark:
- ✅ **Present** — exact match or clear equivalent (e.g. "Rust" = "Rust programming language")
- 🔄 **Implied** — you clearly have the skill (evidence in experience) but the keyword isn't written
- ❌ **Missing** — keyword not in resume; must be added if you actually have the skill

```
KEYWORD COVERAGE REPORT
════════════════════════

TIER 1 — Required (N keywords)
  ✅ [keyword]
  ❌ [keyword] — appears N times in JD — MUST ADD if you have this
  🔄 [keyword] — your [company X] work covers this but word not used

TIER 2 — Preferred (N keywords)
  ✅ [keyword]
  ❌ [keyword] — worth adding if accurate

TIER 3 — Context (N keywords)
  [list with ✅/❌]

Coverage score: [N/N] Tier 1 = [N%] | [N/N] Tier 2 = [N%]
Estimated ATS pass threshold for this role: [N%] Tier 1 coverage
Current status: [LIKELY PASS / AT RISK / LIKELY FILTERED]
```

---

## Phase 4: Formatting Safety Audit

Check the resume for ATS-unsafe formatting. Score each item:

```
FORMATTING AUDIT
════════════════

Layout:
  ✅/❌ Single column layout (multi-column = parsing disaster)
  ✅/❌ No text boxes or floating frames
  ✅/❌ No tables used for layout (data tables OK; layout tables not)
  ✅/❌ No headers/footers containing important info
  ✅/❌ No graphics, icons, or images (logos, headshots, progress bars)
  ✅/❌ No charts or infographics

Section headings:
  ✅/❌ Standard section names: Summary/Objective, Experience/Work Experience,
         Education, Skills, Certifications, Projects
  ✅/❌ No creative section names ("My Journey", "Arsenal", "What I Bring")
  ✅/❌ Headings are text, not images

Contact info:
  ✅/❌ Name, email, phone in main body (not header/footer)
  ✅/❌ Email is plain text, not hyperlink-only
  ✅/❌ Location included (city + country/state minimum)

Dates:
  ✅/❌ Consistent format (Month YYYY or YYYY — not mixed)
  ✅/❌ Employment dates on same line as role/company

Content:
  ✅/❌ No special characters that break parsing (smart quotes OK; unusual Unicode not)
  ✅/❌ No abbreviations for required keywords (write out AND abbreviate: "Natural Language Processing (NLP)")
  ✅/❌ File format: [PDF / DOCX / TEX] — [ATS safe / risk / unsafe]

If .tex file: check that \hypersetup has pdfencoding=auto and fonts are standard
```

---

## Phase 5: Section Quality Check

Beyond keywords and formatting, ATS scores on content quality signals:

```
SECTION QUALITY CHECK
══════════════════════

Summary/Objective:
  ✅/❌ Present (ATS scores resumes with summaries higher for senior roles)
  ✅/❌ Contains target job title keywords
  ✅/❌ Contains top 3 Tier 1 keywords
  ✅/❌ Under 5 sentences

Experience bullets:
  ✅/❌ Start with strong action verbs (Led, Built, Designed — not "Responsible for")
  ✅/❌ Include quantified outcomes (%, $, users, latency, etc.)
  ✅/❌ Keywords distributed throughout (not only in Skills section)
  ✅/❌ Most recent role has most/longest bullets

Skills section:
  ✅/❌ Present as a dedicated section (not buried in bullets)
  ✅/❌ Organized by category (Languages / Frameworks / Tools / Cloud)
  ✅/❌ Contains all Tier 1 keywords you actually have

Education:
  ✅/❌ Degree, field, institution, year — all present
  ✅/❌ No GPA unless > 3.5 or role requires it
```

---

## Phase 6: ATS Score + Gap Report

Produce the final score and prioritized fix list:

```
ATS AUDIT RESULTS
══════════════════
Resume: [filename]
Job: [Company] — [Role]
Date: [today]

SCORES
──────
Keyword coverage (Tier 1):  [N/N] = [N%]
Keyword coverage (Tier 2):  [N/N] = [N%]
Formatting safety:          [N/N] critical issues
Section quality:            [N/N] checks passed

OVERALL ATS SCORE: [N/100]
  [0–49]   ❌ HIGH RISK — likely filtered before human review
  [50–69]  ⚠️  AT RISK — may pass, will score low
  [70–84]  ✅  ADEQUATE — should pass, room to improve
  [85–100] ✅  STRONG — well-optimized for this role

VERDICT: [HIGH RISK / AT RISK / ADEQUATE / STRONG]

PRIORITY FIXES (in order of impact)
──────────────────────────────────
[P1 — CRITICAL: Tier 1 keyword missing]
  Issue: "[keyword]" appears N times in JD but not in resume
  Action: Add to Skills section AND naturally in one experience bullet
  Example rewrite:
    Before: "Developed authentication system for API"
    After:  "Developed [keyword]-based authentication system for API"

[P2 — HIGH: Formatting issue]
  Issue: [specific issue found]
  Action: [exact fix]

[P3 — MEDIUM: Implied keyword not explicit]
  Issue: "[keyword]" is implied by your [Company X] experience but not written
  Action: Add to relevant bullet: append "using [keyword]" or add to Skills

[Continue for all issues, ordered by impact]

OPTIONAL IMPROVEMENTS (nice-to-have)
──────────────────────────────────
  - [Tier 2 keywords worth adding if accurate]
  - [Context keywords that would strengthen domain fit signal]
```

---

## Phase 7: Save Results

Save to:
- If inside an evaluation: `[EVALUATIONS_DIR]/[company-role]/ATS_CHECK.md`
- If inside an application: `[APPLICATIONS_DIR]/[company-role]/ATS_CHECK.md`
- Otherwise: `[WORK_DIR]/ats-check-[date].md`

```
✅ ATS check complete.
Score: [N/100] — [VERDICT]
Saved: [path]

Next steps:
  [If score < 70]: Run /tailor-resume to fix keyword and formatting issues
  [If score ≥ 70]: Review optional improvements, then run /apply
  [If already tailored]: Compare to pre-tailoring score to confirm improvement
```
