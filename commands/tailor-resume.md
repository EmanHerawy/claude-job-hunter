# Tailor Resume — ATS-Optimized LaTeX Resume Generator

Generate a tailored, ATS-optimized resume in LaTeX format for a specific job. Fills `resume-template.tex` with candidate-specific content prioritized for the target role. Accepts job input as either a URL or pasted JD text.

## Inputs

**Usage:** `/tailor-resume "resume-path" "jd-url-or-text" "profile-name" "output-dir"`

Parse `$ARGUMENTS` as up to four quoted strings:
- **First quoted string** (optional) — Path to existing resume file (PDF, markdown, or `.tex`). If omitted, uses profile data only.
- **Second quoted string** (optional) — URL to job description, OR paste the full JD text directly. If omitted, looks for `JD.md` in current evaluation directory, then asks user to provide it.
- **Third quoted string** (optional) — Profile name. If omitted, uses `ACTIVE_PROFILE` from config or auto-detects.
- **Fourth quoted string** (optional) — Output directory. If omitted, creates `[APPLICATIONS_DIR]/[company-role]/`.

## Setup

Resolve paths from `~/.claude/.claude-job-hunter.conf`. Required:
- `WORK_DIR`, `PROFILE_DIR`, `EVALUATIONS_DIR`, `APPLICATIONS_DIR`, `RESOURCES_DIR`

If config missing, ask user to run `/setup`.

---

## Phase 1: Ingest All Inputs

### 1a. Read Source Materials

1. Read `[RESOURCES_DIR]/resume-template.tex` — this is the output base
2. Read candidate profile from `[PROFILE_DIR]/[resolved-name].md`
3. If resume path provided: read it. If PDF, extract all text. If `.tex`, read source.
4. Check if ATS check results exist: look for `ATS_CHECK.md` in the current evaluation or application directory. If found, read it — the gap report informs what to fix.

### 1b. Resolve the Job Description

**If second argument is a URL:** Fetch the page using WebFetch. Extract job title, company name, and full JD text. Strip navigation/footer noise.

**If second argument is pasted text:** Use it directly. Attempt to extract company name and role title from the content.

**If no JD argument:**
- Look for `JD.md` in the current evaluation directory
- If not found, ask: "Please provide the job description — paste the full text here, or provide a URL."

Save the resolved JD to `[output-dir]/JD.md` if it doesn't already exist there.

### 1c. Confirm Inputs

Output a brief summary:
```
Resume source: [filename or "profile only"]
Job: [Company] — [Role Title]
Profile: [profile-name]
ATS check: [found with score N/100 | not found — will run analysis inline]
Output: [output-dir]

Proceed? (yes to continue)
```

Wait for confirmation.

---

## Phase 2: Extract and Prioritize Keywords

### JD Keyword Tiers

**Tier 1 — Required:**
- Skills explicitly under "Required", "Must have", "Qualifications"
- Terms appearing 3+ times
- Exact job title words
- Specific technologies, frameworks, certifications

**Tier 2 — Preferred:**
- Skills under "Nice to have", "Preferred", "Bonus"
- Repeated domain vocabulary
- Methodology terms (agile, CI/CD, zero-trust, etc.)

**Tier 3 — Context:**
- Industry terms, product names, company description vocabulary

### Candidate Skill Inventory

From the profile and resume, build a verified skill inventory:
- Languages, frameworks, tools — with evidence (which jobs/projects used them)
- Years of experience per skill where derivable
- Domain expertise areas

### Keyword Mapping

For each Tier 1/2 keyword, determine:
- **DIRECT** — candidate uses this exact term in their experience
- **RENAME** — candidate has the skill under a different name (e.g. "Kubernetes" vs "K8s")
- **DEMONSTRATE** — candidate has the experience but never named the keyword
- **GAP** — candidate genuinely doesn't have this skill (do NOT fabricate)

Only add keywords the candidate actually has. Never invent skills.

---

## Phase 3: Build Tailored Content

Using the candidate's actual experience, generate each section's content, optimized for the target role:

### Summary (2–4 sentences)
- Lead with expert role title + years of relevant experience
- Include top 3 Tier 1 keywords naturally
- Name the target company's domain (AI/Web3/cloud/security/etc.)
- End with a forward-looking statement matching the JD's mission

### Skills Section
Organize into categories matching the JD's language:
- **Domain expertise:** Pull from profile's most relevant domain skills. Include Tier 1 keywords.
- **Languages:** From verified profile skills, ordered by JD relevance
- **Frameworks & tools:** Tier 1 tools first, then Tier 2
- **Cloud & infra:** Match cloud provider terminology exactly as JD names it (AWS not "Amazon cloud")
- **Security:** Include only if JD has security requirements

Rule: Every Tier 1 keyword the candidate has must appear in Skills. Tier 2 keywords included if candidate has them.

### Experience Bullets

For each role (most recent first), generate bullets that:
1. Start with strong action verbs (Led, Built, Designed, Scaled, Reduced, Shipped — not "Responsible for")
2. Include quantified outcomes where profile provides data (%, $, latency, users, uptime)
3. Naturally embed Tier 1 keywords into relevant bullets (not stuffed — must fit the context)
4. Keep most recent role with the most and longest bullets
5. Distribute keywords throughout roles — not all in one role

**Bullet formula:** `[Action verb] [what you did] [with/using JD_KEYWORD] [achieving MEASURABLE_OUTCOME]`

If profile has metrics: use them exactly.
If profile has no metrics for a bullet: describe scope/impact without fabricating numbers.

### Projects Section
- Only include if the projects directly demonstrate Tier 1 skills
- Lead each project description with the most JD-relevant technical detail
- Include tech stack that matches JD terminology

### Education
- Standard format: Degree, Field, Institution, Year
- Add relevant coursework only if it directly matches JD requirements

---

## Phase 4: Populate the LaTeX Template

Open `resume-template.tex` and replace all placeholders:

| Placeholder | Replacement |
|-------------|-------------|
| `FULL NAME` | Candidate's full name from profile |
| `City, Country (or Remote)` | Location from profile |
| `+1 (555) 000-0000` | Phone from profile |
| `email@example.com` | Email from profile |
| `linkedin.com/in/HANDLE` | LinkedIn URL from profile |
| `github.com/USERNAME` | GitHub URL from profile |
| `TAILORED_SUMMARY_PLACEHOLDER` | Generated summary |
| `DOMAIN_SKILL_1` etc. | Skills from Phase 3 |
| `COMPANY_1`, `TITLE_1`, etc. | Work history entries |
| `IMPACT_BULLET_*` | Generated bullets |
| `PROJECT_NAME_*` etc. | Project entries |
| `DEGREE, FIELD` etc. | Education data |

**LaTeX hygiene rules** (enforce before saving):
- Escape special characters: `&` → `\&`, `%` → `\%`, `$` → `\$`, `#` → `\#`, `_` → `\_` (outside math mode), `{` → `\{`, `}` → `\}`
- Smart quotes → straight quotes or LaTeX `\textquoteleft\textquoteleft` / `\textquoteright\textquoteright`
- Em dashes → `---`, en dashes → `--`
- URLs in `\href{}{}` or `\texttt{}` — no raw unescaped `%` in URLs
- Company names with `&` (e.g. "AT&T"): `AT\&T`

Remove template sections not applicable to this candidate:
- If no publications: remove `\section{Publications \& Talks}` block
- If projects are redundant with experience: remove `\section{Selected Projects}` block
- If certifications section not needed: leave commented out

---

## Phase 5: ATS Self-Check

Before saving, run a quick self-verification:

```
TAILORING SELF-CHECK
════════════════════
Tier 1 keywords in resume:   [N/N] = [N%]
Tier 2 keywords in resume:   [N/N] = [N%]

Keywords in Skills section:  ✅/❌
Keywords in bullets (≥1 ea): ✅/❌
Single column layout:        ✅ (template enforces)
No tables for layout:        ✅ (template enforces)
Standard section headings:   ✅/❌
Dates consistent (Mon YYYY): ✅/❌
Contact in body (not header):✅ (template enforces)

Estimated ATS score:         [N/100]
Delta from previous:         [+N / N/A]
```

If any Tier 1 keyword is missing AND candidate has the skill, revise the relevant bullet or skills line before saving.

---

## Phase 6: Save Output

### Files to Write

1. **`[output-dir]/resume-tailored.tex`** — The filled LaTeX file
2. **`[output-dir]/TAILORING_NOTES.md`** — Explanation of changes made:
   - Which keywords were added and where
   - Which bullets were rewritten and why
   - Any Tier 1 gaps that couldn't be filled (genuine skill gaps)
   - Recommended next step (run `/ats-check` to verify, or `/apply` to proceed)

### TAILORING_NOTES.md format

```markdown
# Resume Tailoring Notes
Resume: resume-tailored.tex
Job: [Company] — [Role]
Date: [today]
Profile: [name]

## Keywords Added
| Keyword | Tier | Where Added | Method |
|---------|------|-------------|--------|
| [keyword] | T1 | Skills + [Company X] bullet | Direct |
| [keyword] | T1 | [Company Y] bullet | Renamed from "[old term]" |
| [keyword] | T2 | Skills section | Added |

## Bullets Rewritten
| Company | Original (paraphrased) | Change Made |
|---------|----------------------|-------------|
| [Co] | "Developed auth system" | Added "[Keyword]" + quantified outcome |

## Genuine Gaps (not added — candidate doesn't have these)
| Keyword | Tier | JD Frequency | Note |
|---------|------|-------------|------|
| [keyword] | T1 | 4× | Candidate has no experience with this |

## ATS Self-Check Score: [N/100]

## Recommended Next Steps
- [ ] Review TAILORING_NOTES.md to confirm all changes are accurate
- [ ] Compile: `pdflatex resume-tailored.tex`
- [ ] Run `/ats-check` to get full formatted audit
- [ ] Run `/tailor-cover-letter` to generate matching cover letter
```

### Compile Instructions

Output to the user:

```
✅ Tailored resume saved.

File: [output-dir]/resume-tailored.tex
ATS self-check score: [N/100]

To compile to PDF:
  cd [output-dir]
  pdflatex resume-tailored.tex

Requirements: TeX Live or MacTeX
  macOS:  brew install --cask mactex-no-gui
  Linux:  sudo apt install texlive-full
  Online: Upload to overleaf.com

Next steps:
  /ats-check — full keyword and formatting audit
  /tailor-cover-letter — generate matching cover letter
  /apply — run full application workflow
```
