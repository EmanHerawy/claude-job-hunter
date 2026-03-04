# Setup — Install Claude Job Hunter Skills

Install all Claude Job Hunter slash commands by symlinking them into your `~/.claude/commands/` directory. Run this once after cloning the repo, or again after pulling updates that add new commands.

## Inputs

**Usage:** `/setup`

No arguments required. The command auto-detects the repo location.

## Workflow

### Step 1: Locate the repo

Find the `claude-job-hunter` repo by checking (in order):
1. The directory this command file lives in (resolve symlinks to find the repo root — go up one level from `commands/`)
2. `$CLAUDE_JOB_HUNTER_DIR` environment variable
3. `./claude-job-hunter/` relative to cwd
4. `~/claude-job-hunter/`

If not found, ask the user for the path.

### Step 2: Verify repo structure

Confirm the repo has the expected structure:
```
[repo]/commands/*.md          # Slash command files
[repo]/profile/               # Candidate profile directory
[repo]/resources/             # Research checklists and references
[repo]/examples/              # Example outputs
```

If anything is missing, warn the user but continue with what exists.

### Step 3: Create commands directory

```bash
mkdir -p ~/.claude/commands
```

### Step 4: Symlink all commands

For each `.md` file in `[repo]/commands/` **except `setup.md` itself** (setup is already installed if running this):

```bash
ln -sf [repo]/commands/[command].md ~/.claude/commands/[command].md
```

Commands to install:
- `build-profile.md` — `/build-profile`
- `should-i-apply.md` — `/should-i-apply`
- `mock-interview.md` — `/mock-interview`
- `interview-feedback.md` — `/interview-feedback`
- `comp-research.md` — `/comp-research`
- `offer-negotiation.md` — `/offer-negotiation`
- `player-card.md` — `/player-card`

### Step 5: Create output directories

```bash
mkdir -p [repo]/evaluations
mkdir -p [repo]/cards
```

### Step 6: Verify installation

List all installed symlinks and verify they point to valid files:

```bash
ls -la ~/.claude/commands/
```

For each symlink, check that the target file exists. Report any broken symlinks.

### Step 7: Report results

Present a summary:

```
Claude Job Hunter — Setup Complete

Installed commands:
  /build-profile        → Build your candidate profile from resume + evidence
  /should-i-apply       → Evaluate whether to apply (scorecard, SWOT, interview prep)
  /mock-interview       → Practice interview loops with feedback
  /interview-feedback   → Debrief after real interviews
  /comp-research        → Market compensation analysis
  /offer-negotiation    → Offer evaluation and negotiation strategy
  /player-card          → Deploy a candidate showcase site

Next steps:
  1. Run /build-profile to create your candidate profile
  2. Run /should-i-apply for your first target role

Repo location: [path]
Profile: [path]/profile/candidate-profile.md
```

If a `candidate-profile.md` already exists and has content beyond the template, note that it's ready. If it's still the template, remind them to run `/build-profile` first.
