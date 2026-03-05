# SWOT Analysis — Company Strategic Intelligence Report

Research a target company and produce a comprehensive SWOT analysis of the company itself — its strategic position, market health, leadership, culture, and trajectory. Use this to understand whether a company is worth your time before you even look at a specific role.

This is NOT a candidate-fit SWOT (that's `/should-i-apply`). This is pure company intelligence: Is this company well-positioned? Is it growing or declining? What are the risks of joining? What opportunities does working there unlock?

## Inputs

**Usage:** `/swot-analysis "company" "focus-area"`

Parse `$ARGUMENTS` as up to two quoted strings:
- **First quoted string** (required) — Company name (e.g., `"Anthropic"`, `"Stripe"`, `"Datadog"`)
- **Second quoted string** (optional) — Focus area or lens for the analysis (e.g., `"engineering"`, `"security"`, `"AI/ML"`, `"leadership"`). Defaults to general company-wide analysis.

If company name is missing, ask for it.

The focus area tailors the depth of research:
- **No focus** — broad strategic SWOT across all dimensions
- **"engineering"** — deeper dive into eng culture, tech stack, technical leadership, developer experience
- **"security"** — security posture, compliance maturity, incident history, security team investment
- **"AI/ML"** — AI strategy, model capabilities, data moat, research team, compute infrastructure
- **"leadership"** — executive team depth, tenure, track record, succession planning, culture-setting
- **"product"** — product-market fit, user growth, competitive differentiation, roadmap signals
- **"financial"** — revenue, burn rate, funding runway, path to profitability, unit economics
- Any other lens the user provides — adapt research accordingly

## Setup

Resolve paths by checking (in order):
1. Read `~/.claude/.claude-job-hunter.conf` for `WORK_DIR`, `REPO_DIR`, and component paths
2. Fall back: `$CLAUDE_JOB_HUNTER_DIR` → `./claude-job-hunter/` → `~/claude-job-hunter/`

If the config exists, use its paths (`EVALUATIONS_DIR`, `RESOURCES_DIR`, etc.). Otherwise, use `[claude-job-hunter]/` as both repo and working directory.

If nothing is found, ask the user to run `/setup` first or provide the path.

## Existing Context

Before starting, check for existing materials:
- `[EVALUATIONS_DIR]/[company]-*/COMPANY_INTEL.md` — prior research from `/should-i-apply`
- `[EVALUATIONS_DIR]/[company]-*/SCORECARD.md` — prior evaluations
- `[claude-job-hunter]/resources/research-checklist.md` — research structure guide

If prior research exists, build on it rather than starting from scratch. Note what's stale (>3 months old) and refresh those areas.

## Output

Create or update:
```
[EVALUATIONS_DIR]/[company]/
└── SWOT_ANALYSIS.md    # Full company SWOT with sources
```

Note: This uses `[company]/` not `[company]-[role]/` since it's company-level, not role-specific.

## Workflow

### Phase 1: Company Foundation Research

Use WebSearch and WebFetch to build a comprehensive company picture. Follow `[claude-job-hunter]/resources/research-checklist.md` as a starting framework, then go deeper.

#### 1a. Company Basics
- [ ] Official website, mission, founding story
- [ ] Wikipedia / Crunchbase profile
- [ ] Founding year, founders, headquarters, employee count
- [ ] Current leadership team (CEO, CTO, CFO, CPO, CISO)
- [ ] Board of directors and notable investors

#### 1b. Business & Financial Health
- [ ] Business model and revenue streams
- [ ] Funding history: rounds, amounts, investors, valuation trajectory
- [ ] If public: stock performance, revenue growth, margins, earnings
- [ ] If private: last known valuation, runway estimates, profitability signals
- [ ] Customer base: who buys, how many, retention signals
- [ ] Pricing model and changes (price increases = confidence; discounting = desperation)
- [ ] Key partnerships and enterprise contracts

#### 1c. Product & Market Position
- [ ] Core products and services
- [ ] Competitive landscape: who are the direct and indirect competitors?
- [ ] Market share estimates or relative positioning
- [ ] Product reviews, G2/Capterra ratings, NPS signals
- [ ] Recent product launches, pivots, or sunsets
- [ ] Developer/user community health (GitHub stars, Discord/Slack activity, Stack Overflow presence)

#### 1d. Technology & Engineering
- [ ] Tech stack (from job postings, engineering blog, GitHub)
- [ ] Engineering blog: frequency, depth, topics (signals investment in eng culture)
- [ ] Open source contributions and projects
- [ ] Infrastructure: cloud providers, scale indicators
- [ ] Technical debt signals (from Glassdoor reviews, job postings mentioning "modernization")
- [ ] AI/ML capabilities and strategy

#### 1e. People & Culture
- [ ] Glassdoor overall rating and trend (improving or declining?)
- [ ] Glassdoor CEO approval rating
- [ ] Blind sentiment and threads
- [ ] Company values (stated vs. demonstrated)
- [ ] DEI initiatives and representation data
- [ ] Remote/hybrid/onsite policy and how it's received
- [ ] Layoff history (dates, sizes, how handled)
- [ ] Hiring velocity: are they growing, flat, or contracting?

#### 1f. Recent News & Events (last 12 months)
- [ ] Major announcements, product launches
- [ ] Leadership changes (departures are as important as hires)
- [ ] Security incidents, outages, or PR crises
- [ ] Regulatory actions or legal issues
- [ ] Awards, recognitions, "best places to work" lists
- [ ] M&A activity (acquiring or being acquired?)

#### 1g. Focus Area Deep Dive
If a focus area was specified, conduct additional targeted research:
- Search for `"[company]" "[focus-area]" blog OR announcement OR strategy`
- Search for `"[company]" "[focus-area]" team OR hiring OR investment`
- Find relevant team leaders on LinkedIn and review their backgrounds
- Check conference talks and publications from team members in this area
- Review job postings in this area for signals about team maturity and priorities

#### Suggested Searches
1. `"[COMPANY]" about mission values founded`
2. `"[COMPANY]" funding series valuation crunchbase`
3. `"[COMPANY]" revenue growth customers`
4. `"[COMPANY]" competitors market share`
5. `"[COMPANY]" engineering blog technical`
6. `"[COMPANY]" glassdoor reviews culture`
7. `"[COMPANY]" layoffs OR restructuring`
8. `"[COMPANY]" news 2026`
9. `"[COMPANY]" security incident OR breach OR outage`
10. `"[COMPANY]" leadership team executive`
11. `"[COMPANY]" "[FOCUS]" team strategy` (if focus area specified)
12. `"[COMPANY]" site:blind`
13. `"[COMPANY]" open source github`
14. `"[COMPANY]" hiring OR careers growth`

**Output:** Present a research summary organized by category. Ask user: "Anything else you know about this company, or areas you want me to dig deeper on?"

---

### Phase 2: SWOT Analysis

Build the SWOT analysis from research findings. Every item must be grounded in evidence with source references — no generic filler.

#### Strengths — "What this company does well"

Identify concrete, evidence-backed strengths:

**Market & Product:**
- Market position and competitive advantages (moat)
- Product quality and differentiation signals
- Customer loyalty and retention indicators
- Network effects, switching costs, or data advantages

**Financial:**
- Revenue growth trajectory
- Funding strength and investor quality
- Path to profitability (or already profitable)
- Cash runway and financial stability

**People & Culture:**
- Leadership team quality and track record
- Engineering talent density and retention
- Culture strengths from reviews and public signals
- Employee satisfaction trends

**Technology:**
- Technical architecture advantages
- Innovation velocity (shipping pace, R&D investment)
- Open source presence and community
- Proprietary technology or IP

For each strength: **Evidence** (what data supports this) and **Durability** (is this sustainable or temporary?)

#### Weaknesses — "Where this company is vulnerable"

Identify honest, evidence-backed weaknesses:

**Market & Product:**
- Product gaps relative to competitors
- Customer complaints and churn signals
- Market positioning risks (too niche, too broad, wrong segment)
- Dependency on single product/customer/market

**Financial:**
- Burn rate concerns or unprofitable growth
- Funding dependency or dilution risk
- Revenue concentration (too few large customers)
- Pricing pressure from competitors

**People & Culture:**
- Leadership gaps or recent departures
- Glassdoor/Blind negative patterns (not outliers — patterns)
- Hiring difficulties in key areas
- Cultural issues (burnout, bureaucracy, politics)
- Layoff aftermath effects

**Technology:**
- Technical debt indicators
- Stack limitations or migration needs
- Security/compliance gaps
- Scaling challenges

For each weakness: **Severity** (minor, moderate, serious) and **Trajectory** (getting better, stable, getting worse?)

#### Opportunities — "Tailwinds and growth vectors"

Identify external and internal opportunities:

**Market Tailwinds:**
- Industry growth trends benefiting this company
- Regulatory changes that help (or hurt competitors)
- Technology shifts they're well-positioned for
- Geographic or segment expansion potential

**Strategic Opportunities:**
- M&A targets or partnership potential
- Adjacent markets they could enter
- Platform plays or ecosystem building
- International expansion

**For job seekers specifically:**
- Growing teams = more roles, faster promotion
- New initiatives = greenfield work, high visibility
- Pre-IPO stage = equity upside potential
- Compliance buildout = job security for specialists
- Leadership gaps = opportunity to step up

For each opportunity: **Likelihood** (high/medium/low) and **Timeline** (near-term, 1-2 years, 3+ years)

#### Threats — "Risks and headwinds"

Identify concrete threats to the company:

**Competitive Threats:**
- Stronger competitors gaining ground
- New entrants disrupting the market
- Open source alternatives eroding value
- Platform risk (dependency on AWS/Google/Apple etc.)

**Market Threats:**
- Market slowdown or contraction
- Customer spending reduction
- Regulatory risk (new laws, enforcement actions)
- Technology shifts that could make their product obsolete

**Internal Threats:**
- Key person dependency (what if the CEO/CTO leaves?)
- Culture deterioration trajectory
- Technical debt reaching critical mass
- Scaling challenges that compound

**For job seekers specifically:**
- Layoff risk indicators
- Equity value risk (down rounds, delayed IPO)
- Role instability (reorgs, scope changes)
- Brand risk (will this company on your resume help or hurt in 2-3 years?)

For each threat: **Probability** (high/medium/low), **Impact** (high/medium/low), and **Mitigation** (what would need to happen to neutralize this?)

---

### Phase 3: Strategic Assessment

Synthesize the SWOT into actionable intelligence.

#### A. Company Health Score

Rate the company across dimensions (1-10 scale):

| Dimension | Score | Trend | Key Signal |
|-----------|-------|-------|-----------|
| Financial Health | X/10 | up/down/stable | [one-line evidence] |
| Product Strength | X/10 | up/down/stable | [one-line evidence] |
| Market Position | X/10 | up/down/stable | [one-line evidence] |
| Leadership Quality | X/10 | up/down/stable | [one-line evidence] |
| Engineering Culture | X/10 | up/down/stable | [one-line evidence] |
| Employee Satisfaction | X/10 | up/down/stable | [one-line evidence] |
| Growth Trajectory | X/10 | up/down/stable | [one-line evidence] |
| Stability/Risk | X/10 | up/down/stable | [one-line evidence] |
| **Overall** | **X/10** | | |

#### B. Company Trajectory

Where is this company headed? Synthesize into one of:

- **Rocketship** — High growth, strong fundamentals, clear market leadership trajectory
- **Steady Climber** — Solid growth, good fundamentals, building toward something
- **Plateau** — Growth slowing, market position stable but not expanding
- **Crossroads** — Could go either way; key decisions or events will determine trajectory
- **Declining** — Losing ground on multiple dimensions, warning signs accumulating
- **Turnaround** — Was declining, showing early signs of recovery under new leadership/strategy

Support the assessment with a 6/12/24 month outlook.

#### C. Job Seeker Verdict

Bottom-line assessment for someone considering joining:

**Best for:** [who thrives here — e.g., "builders who want ownership", "specialists who want scale", "leaders who want to shape culture"]

**Worst for:** [who should avoid — e.g., "people who need stability", "those who want work-life balance", "anyone who needs clear career ladders"]

**Ideal timing:** Is now a good time to join? Why or why not?
- Early enough to get meaningful equity? Or too late?
- Growing enough to have career growth? Or plateaued?
- Stable enough to be safe? Or risky?

**Watch signals:** What should a candidate monitor before/after joining?
- [Signal 1] — if this happens, it's a green flag
- [Signal 2] — if this happens, it's a red flag
- [Signal 3] — watch for this in the interview process

---

### Phase 4: Write Output

Generate `SWOT_ANALYSIS.md`:

```markdown
# Company SWOT Analysis: [Company]
**Date:** [date]
**Focus:** [focus area or "General"]

## Executive Summary
[3-5 sentence summary: what this company is, its trajectory, and the bottom line for job seekers]

## Company Overview
- **Founded:** [year]
- **HQ:** [location]
- **Employees:** [count]
- **Stage:** [startup/growth/mature/public]
- **Valuation/Market Cap:** [if known]
- **Business Model:** [one line]
- **Core Product:** [one line]

## SWOT Analysis

### Strengths
(evidence-backed strengths with durability assessment)

### Weaknesses
(evidence-backed weaknesses with severity and trajectory)

### Opportunities
(tailwinds and growth vectors with likelihood and timeline)

### Threats
(risks and headwinds with probability and impact)

## Company Health Scorecard
(dimension table from Phase 3)

## Trajectory Assessment
(company trajectory with 6/12/24 month outlook)

## Job Seeker Verdict
- **Best for:** [who thrives here]
- **Worst for:** [who should avoid]
- **Ideal timing:** [is now a good time to join?]
- **Watch signals:** [what to monitor]

## Focus Area Deep Dive: [Focus]
(additional analysis for the specified focus area, if provided)

## Sources
(all URLs and data sources used, organized by section)
```

**Output:** Present the executive summary and verdict. Ask: "Want me to go deeper on any section, or research a specific concern?"

---

## Quality Checklist

Before finishing, verify:
- [ ] Every SWOT item has evidence with source URLs — no unsourced claims
- [ ] Strengths include durability assessment (temporary vs. sustainable)
- [ ] Weaknesses include severity AND trajectory (getting better or worse?)
- [ ] Opportunities distinguish between likely and speculative
- [ ] Threats include probability and impact ratings
- [ ] Company health scores are justified by evidence, not vibes
- [ ] Trajectory assessment is supported by multiple data points
- [ ] Job seeker verdict is specific and actionable, not generic
- [ ] Focus area (if specified) has meaningfully deeper analysis than general sections
- [ ] Recent news is truly recent (within last 12 months)
- [ ] Glassdoor/Blind signals look for patterns, not individual reviews
- [ ] Financial analysis uses real data where available, clearly labels estimates
- [ ] Competitive analysis names actual competitors with specific comparisons
- [ ] Watch signals are concrete and observable, not vague
