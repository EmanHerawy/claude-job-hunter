# Apply — Full Application Workflow Orchestrator

Run the complete job application workflow. Accepts a job via URL or pasted JD text. Evaluates resume fit, tailors resume and cover letter if needed, saves all artifacts to a structured application directory, and tracks application status.

## Inputs

**Usage:** `/apply "jd-url-or-text" "profile-name" "resume-path"`

Parse `$ARGUMENTS` as up to three quoted strings:
- **First quoted string** (required) — URL to job posting, OR paste the full JD text directly.
- **Second quoted string** (optional) — Profile name. Defaults to `ACTIVE_PROFILE` or auto-detected profile.
- **Third quoted string** (optional) — Path to base resume file. If omitted, uses most recently tailored resume or profile data.

If the first argument is missing, ask: "Please provide the job — paste the URL or JD text."

## Setup

Resolve paths from `~/.claude/.claude-job-hunter.conf`. Required:
- `WORK_DIR`, `PROFILE_DIR`, `EVALUATIONS_DIR`, `APPLICATIONS_DIR`, `RESOURCES_DIR`

---

## Phase 1: Job Intake

### 1a. Resolve the Job Description

**If argument is a URL:** Fetch via WebFetch. Extract:
- Company name
- Role title
- Full JD text (requirements, responsibilities, qualifications)
- Application instructions (direct apply, ATS portal URL, email address)
- Deadline if listed
- Compensation if listed

**If argument is pasted text:** Use directly. Extract company name and role title from content. If ambiguous, ask user to confirm: "Looks like this is [Company] — [Role]. Is that right?"

### 1b. Create Application Directory

```
[APPLICATIONS_DIR]/[company-slug]-[role-slug]/
```

Use lowercase, hyphenated slugs (e.g., `stripe-backend-engineer`, `anthropic-security-engineer`).

Create the directory and save `JD.md` with the full job description text and source URL.

### 1c. Show Job Summary

```
JOB INTAKE
══════════
Company:    [Company]
Role:       [Role Title]
Source:     [URL or "pasted text"]
Apply via:  [URL / email / unknown]
Deadline:   [date or "not listed"]
Salary:     [range or "not listed"]
Saved to:   [application-dir]

Proceeding with full application workflow...
```

---

## Phase 2: Resume Fit Assessment

### 2a. Quick Keyword Scan

Before running the full ATS check, do a rapid scan:
- Extract top 5 Tier 1 keywords from JD
- Check if they appear in the candidate's base resume / profile
- Compute rough Tier 1 coverage %

### 2b. Decision Gate

```
QUICK FIT SCAN
══════════════
Tier 1 keyword coverage (rough): [N%]
Profile match confidence: [HIGH / MEDIUM / LOW]

Recommendation:
  [If < 50%]: ⚠️  LOW MATCH — consider whether to proceed.
               Your strongest Tier 1 gaps: [list top 3]
               Continue anyway? (yes/no)
  [If 50–74%]: Resume needs tailoring before applying.
               Running full ATS check + tailoring now.
  [If ≥ 75%]: Good baseline fit. Running ATS check to optimize.
```

If user responds "no" to low match: stop here and suggest using `/should-i-apply` first.

---

## Phase 3: ATS Check

Run the full ATS audit inline (same logic as `/ats-check`):

1. Extract all JD keywords (Tier 1 / 2 / 3)
2. Score keyword coverage against base resume/profile
3. Check formatting safety (applicable if `.tex` resume provided)
4. Check section quality
5. Compute ATS score (0–100)

Save results to `[application-dir]/ATS_CHECK.md`.

Report:
```
ATS AUDIT
══════════
Score: [N/100] — [VERDICT]
Tier 1 coverage: [N/N] = [N%]
Critical gaps: [list]

Tailoring resume to fix gaps...
```

---

## Phase 4: Tailor Resume

Run full resume tailoring (same logic as `/tailor-resume`):

1. Map all Tier 1 keywords to candidate's actual experience
2. Generate ATS-optimized summary, skills, bullets
3. Populate `resume-template.tex`
4. Run ATS self-check — verify Tier 1 coverage improved
5. Save to `[application-dir]/resume-tailored.tex`

Show before/after score:
```
Resume tailored.
ATS score: [before] → [after]
Tier 1 coverage: [before%] → [after%]
```

---

## Phase 5: Generate Cover Letter

Run cover letter generation (same logic as `/tailor-cover-letter`):

1. Research company (check for COMPANY_INTEL.md; if not found, do targeted search for one specific detail)
2. Generate four paragraphs (hook → why them → why you → CTA)
3. Populate `cover-letter-template.tex`
4. Save to `[application-dir]/cover-letter.tex`

---

## Phase 6: Save Application Record

Create `[application-dir]/APPLICATION.md`:

```markdown
# Application: [Company] — [Role]

## Status
**Current:** PREPARING
**Applied:** (fill when submitted)
**Last updated:** [today]

## Job Details
- **Company:** [Company]
- **Role:** [Role Title]
- **Source:** [URL or "pasted JD"]
- **Apply via:** [portal URL / email / unknown]
- **Deadline:** [date or N/A]
- **Salary:** [range or N/A]

## Files
| File | Description | Status |
|------|-------------|--------|
| `JD.md` | Original job description | ✅ Saved |
| `ATS_CHECK.md` | ATS audit report | ✅ Saved |
| `resume-tailored.tex` | Tailored LaTeX resume | ✅ Saved |
| `cover-letter.tex` | Tailored LaTeX cover letter | ✅ Saved |
| `resume-tailored.pdf` | Compiled PDF | ⏳ Compile with pdflatex |
| `cover-letter.pdf` | Compiled PDF | ⏳ Compile with pdflatex |

## ATS Score
- **Pre-tailoring:** [N/100]
- **Post-tailoring:** [N/100]
- **Verdict:** [ADEQUATE / STRONG]

## Keywords Added
[From TAILORING_NOTES.md — list Tier 1 keywords added]

## Notes
[Any application-specific notes: referral contact, deadline urgency, etc.]

## Application Log
| Date | Action | Notes |
|------|--------|-------|
| [today] | Application prepared | ATS score [N/100] |
```

---

## Phase 7: Final Output

```
✅ APPLICATION READY
════════════════════
Company:  [Company]
Role:     [Role Title]
Saved to: [application-dir]

ATS Score:    [N/100] — [VERDICT]
Resume:       resume-tailored.tex ✅
Cover Letter: cover-letter.tex ✅
Tracker:      APPLICATION.md ✅

COMPILE STEPS:
  cd [application-dir]
  pdflatex resume-tailored.tex
  pdflatex cover-letter.tex

BEFORE SUBMITTING:
  □ Read cover-letter.tex — verify the company-specific detail in paragraph 2
  □ Confirm all metrics in resume bullets are accurate
  □ Check the application portal URL and any required fields
  □ Note any deadline: [deadline or "none listed"]

APPLY VIA: [portal URL / email / manual]

Track your application:
  Update APPLICATION.md → Status: APPLIED when submitted
  Use /track to see all applications
```

---

## Status Lifecycle

Application statuses (update `APPLICATION.md` manually or via `/track`):

| Status | Meaning |
|--------|---------|
| `PREPARING` | Files being generated |
| `READY` | All files saved, not yet submitted |
| `APPLIED` | Submitted — add date |
| `PHONE_SCREEN` | First contact from company |
| `INTERVIEWING` | Active interview process |
| `OFFER` | Offer received |
| `ACCEPTED` | Offer accepted |
| `REJECTED` | Application rejected |
| `WITHDRAWN` | Candidate withdrew |
| `STALE` | No response > 30 days |
