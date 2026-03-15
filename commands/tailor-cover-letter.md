# Tailor Cover Letter — LaTeX Cover Letter Generator

Generate a tailored cover letter in LaTeX format for a specific job. Fills `cover-letter-template.tex` with candidate-specific content. Accepts job input as either a URL or pasted JD text. Pairs with `/tailor-resume` — run after resume tailoring for a matched set.

## Inputs

**Usage:** `/tailor-cover-letter "resume-path" "jd-url-or-text" "profile-name" "output-dir"`

Parse `$ARGUMENTS` as up to four quoted strings:
- **First quoted string** (optional) — Path to tailored resume `.tex` or original resume. If omitted, uses profile + application directory.
- **Second quoted string** (optional) — URL to job description, OR paste the full JD text directly. If omitted, looks for `JD.md` in current evaluation/application directory.
- **Third quoted string** (optional) — Profile name.
- **Fourth quoted string** (optional) — Output directory. Defaults to existing application directory for this company/role.

## Setup

Resolve paths from `~/.claude/.claude-job-hunter.conf`. Required:
- `WORK_DIR`, `PROFILE_DIR`, `EVALUATIONS_DIR`, `APPLICATIONS_DIR`, `RESOURCES_DIR`

---

## Phase 1: Ingest All Inputs

### 1a. Read Source Materials

1. Read `[RESOURCES_DIR]/cover-letter-template.tex`
2. Read candidate profile from `[PROFILE_DIR]/[resolved-name].md`
3. If resume path provided: read it for specific bullets and metrics to reference
4. Check for existing research:
   - `[output-dir]/SCORECARD.md` — company intel, culture signals, SWOT
   - `[output-dir]/COMPANY_INTEL.md` — research findings
   - `[output-dir]/TAILORING_NOTES.md` — keywords used in resume (for consistency)
   - `[output-dir]/ATS_CHECK.md` — priority skills to echo in letter

### 1b. Resolve the Job Description

**If second argument is a URL:** Fetch via WebFetch. Extract company name, role title, team description, and any mission/values language.

**If second argument is pasted text:** Use directly. Extract company name, role title, key requirements.

**If no JD provided:** Look for `JD.md` in the application directory. If not found, ask the user to provide it.

### 1c. Company Research

If `COMPANY_INTEL.md` or `SCORECARD.md` exists, read them for:
- What the company actually does (specific, not generic)
- A specific product decision, technical choice, engineering blog post, or public architecture detail — something real to reference in the "why them" paragraph
- Culture signals, stated values
- Who the hiring manager/team lead is (for salutation)

If no research exists, do a targeted search:
- Fetch the company's engineering blog or tech blog if URL can be inferred
- Search for `[company name] engineering culture` or `[company name] [role area] blog post`
- Look for a specific, citable detail — do not write generic praise

### 1d. Confirm Before Writing

```
Cover letter for: [Company] — [Role]
Profile: [name]
Company detail found: "[specific thing to reference in why-them para]"
Hiring manager: [name if found / "Hiring Team"]

Proceed? (yes to continue)
```

---

## Phase 2: Generate Cover Letter Content

Write four paragraphs following the hook → why them → why you → CTA structure. Every sentence must be specific and non-generic.

### Paragraph 1: Hook + Role Identification

Rules:
- Do NOT open with "I am writing to apply for..." — this is the weakest possible opener
- Open with something genuine about the company that connects to the candidate's interests
- Name the role in the second or third sentence
- 2–4 sentences

Draw from: candidate's stated motivations in profile + company's mission/product.

Example approach: Lead with what you find genuinely interesting about the technical problem they're solving or a specific product decision, then connect it to why this role specifically caught your attention.

### Paragraph 2: Why Them (specific, not generic)

Rules:
- Reference something real and specific: a product decision, engineering blog post, public architecture choice, funding milestone with context, or market position
- Explain why that specific thing resonates with what the candidate cares about
- No flattery ("I've always admired your company") — only evidence-based connection
- 3–5 sentences, one concrete specific detail required

Draw from: COMPANY_INTEL.md, engineering blog, JD mission language.

If no specific detail can be confirmed: ask the user — "What drew you to [Company]? Any specific product or decision that resonated with you?"

### Paragraph 3: Why You (evidence-backed)

Rules:
- 2–3 strongest achievements mapped directly to JD requirements
- Each achievement: what you did + measurable outcome + why it's relevant here
- No generic claims ("I am a fast learner", "passionate about technology")
- Every sentence specific and verifiable
- 4–6 sentences
- Echo 1–2 Tier 1 keywords from the resume tailoring (for consistency)

Draw from: resume bullets (especially the strongest quantified ones), TAILORING_NOTES.md.

Map each achievement to a specific JD requirement — make the connection explicit in the paragraph logic.

### Paragraph 4: Call to Action

Rules:
- Express genuine enthusiasm — specific to this company/role, not boilerplate
- Request a conversation
- Optionally: name one specific thing you want to discuss (a technical challenge, a product question, a shared problem space)
- 2–3 sentences, confident tone, no begging

### Postscript (optional)

If there's one striking thing that didn't fit above — a specific project link, a metric, a question for them — uncomment the P.S. block and use it. The P.S. is read almost as often as the opening.

Candidates who have exceptional projects, publications, or a very specific alignment signal should use it.

---

## Phase 3: Populate the LaTeX Template

Open `cover-letter-template.tex` and replace all placeholders:

| Placeholder | Replacement |
|-------------|-------------|
| `FULL NAME` (both occurrences) | Candidate's full name |
| `City, Country (or Remote)` | Location from profile |
| `EMAIL` (both occurrences) | Email from profile |
| `HANDLE` | LinkedIn handle from profile |
| `COMPANY NAME` | Company name |
| `Dear Hiring Team,` | `Dear [Hiring Manager Name],` if found, else keep |
| `HOOK_PARAGRAPH_PLACEHOLDER` | Generated paragraph 1 |
| `WHY_THEM_PARAGRAPH_PLACEHOLDER` | Generated paragraph 2 |
| `WHY_YOU_PARAGRAPH_PLACEHOLDER` | Generated paragraph 3 |
| `CLOSING_PARAGRAPH_PLACEHOLDER` | Generated paragraph 4 |
| `POSTSCRIPT_PLACEHOLDER` | P.S. content if applicable (and uncomment the block) |

**LaTeX hygiene rules:**
- Escape `&` → `\&`, `%` → `\%`, `$` → `\$`, `#` → `\#`
- Smart quotes → LaTeX equivalents or straight
- Em dashes → `---`
- URLs in `\href{url}{display text}`
- Keep `\today` for the date — do not hardcode

**Length check:** After populating, verify the document fits on one page. If paragraphs are too long, tighten:
- Cut the weakest sentence from paragraph 2 or 3
- Combine two short sentences in paragraph 4
- Never cut the specific detail or the strongest achievement metric

---

## Phase 4: Save Output

### Files to Write

1. **`[output-dir]/cover-letter.tex`** — The filled LaTeX file
2. **Update `[output-dir]/APPLICATION.md`** (if it exists) to note cover letter generated

### Compile Instructions

Output to the user:

```
✅ Cover letter saved.

File: [output-dir]/cover-letter.tex
Company: [Company] — [Role]

To compile to PDF:
  cd [output-dir]
  pdflatex cover-letter.tex

Review checklist before sending:
  □ Paragraph 2 specific detail is accurate (verify the fact you cited)
  □ Paragraph 3 achievements match your actual resume
  □ Hiring manager name is correct (if used)
  □ P.S. link is live (if used)

Next steps:
  /apply — complete full application workflow
  /ats-check — run ATS audit on the resume before submitting
```
