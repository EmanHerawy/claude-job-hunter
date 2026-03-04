# Claude Job Hunter

AI-powered job search toolkit using [Claude Code](https://docs.anthropic.com/en/docs/claude-code) slash commands. Two skills that turn your resume into targeted outputs:

- **`/player-card`** — Generate a password-protected Cloudflare Worker site that showcases you TO a company (deployed, shareable)
- **`/should-i-apply`** — Evaluate WHETHER you should apply, with a scorecard, sentiment-driven SWOT, and deep interview prep

Both commands use live web research to produce outputs specific to you and the target company — no generic templates.

## Quick Start

### 1. Clone and configure

```bash
git clone https://github.com/deeso/claude-job-hunter.git ~/claude-job-hunter
```

### 2. Fill in your candidate profile

Edit `profile/candidate-profile.md` with your information. This is the core data source both commands use. The more honest and specific you are (especially the "Honest Considerations" section), the better the outputs.

### 3. Install the slash commands

Symlink or copy the commands into your Claude Code commands directory:

```bash
# Create Claude commands directory if it doesn't exist
mkdir -p ~/.claude/commands

# Symlink the commands (recommended — stays in sync with repo updates)
ln -sf ~/claude-job-hunter/commands/player-card.md ~/.claude/commands/player-card.md
ln -sf ~/claude-job-hunter/commands/should-i-apply.md ~/.claude/commands/should-i-apply.md
```

### 4. Use them

Open Claude Code and run:

```
/should-i-apply "~/resume.pdf" "Anthropic" "Security Engineer" "https://job-url.com" "~/evidence/"

/player-card "Anthropic" "Security Engineer" "https://job-url.com"
```

## Commands

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

```
claude-job-hunter/
├── commands/                        # Claude Code slash commands
│   ├── player-card.md               # /player-card command
│   └── should-i-apply.md            # /should-i-apply command
├── profile/
│   └── candidate-profile.md         # Your profile (fill this in)
├── resources/
│   ├── card-architecture.md         # Player card HTML/CSS/Worker reference
│   ├── research-checklist.md        # Company research prompts
│   └── interview-research-checklist.md  # Leadership & culture research
├── examples/
│   ├── openrouter-research.md       # Example company research output
│   └── openrouter-fit-analysis.md   # Example fit analysis output
├── evaluations/                     # /should-i-apply outputs go here
│   └── [company]-[role]/
└── cards/                           # /player-card outputs go here
    └── [company]-[year]/
```

## How It Works

Both commands follow a multi-phase workflow with user confirmation between phases:

1. **Profile ingestion** — reads your resume, profile, and any supporting evidence
2. **JD analysis** — fetches and parses the job description
3. **Live research** — uses WebSearch/WebFetch to research the company, leadership, culture, Glassdoor reviews, engineering blog, etc.
4. **Analysis** — crosses your profile against the company/role to produce personalized outputs
5. **Output generation** — writes structured markdown (should-i-apply) or a deployable project (player-card)

## Tips

- **Be honest in your candidate profile.** The "Honest Considerations" section makes SWOT analysis genuinely useful. Generic strengths produce generic outputs.
- **Use the evidence directory.** The more context you give (cover letters, project docs, publications), the stronger the evidence mapping in scorecards and interview prep.
- **Review between phases.** Both commands pause for confirmation. Add context the AI might have missed.
- **Iterate.** Run `/should-i-apply` first to decide if a role is worth it, then `/player-card` to create the deliverable.

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- For `/player-card` deployment: Cloudflare account, Wrangler CLI, custom domain, Resend API key
- For `/should-i-apply`: no infrastructure needed — outputs are local markdown files
