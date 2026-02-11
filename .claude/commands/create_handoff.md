---
description: Create a handoff document for transferring work context to a new session.
model: opus
---

# Create Handoff

You are creating a handoff document so a future Claude session (or a different developer) can pick up exactly where you left off.

## Input

$ARGUMENTS are optional notes about what to emphasize.

## Process

### Step 1: Gather Context

Collect from the current session:
```bash
git branch --show-current
git log --oneline -n 10
git status
git diff --stat
```

Read the current plan file if one exists in `docs/plans/`.

### Step 2: Write Handoff

Write to `docs/handoffs/YYYY-MM-DD_HH-MM-SS_{description}.md`.

```markdown
---
date: {ISO timestamp}
branch: {current branch}
commit: {current commit hash}
plan: {path to plan if exists}
---

# Handoff: {concise description}

## What Was Done
{completed work — be specific, reference commits or phases}

## What's In Progress
{current state of partially-done work}
{if implementing a plan, which phase and which step}

## What's Left
{remaining work, ordered by priority}

## Key Decisions Made
{decisions and their rationale — these are the most likely things to be lost}

## Known Issues
{bugs, edge cases, things that are broken but deferred}

## Critical File References
{exact file:line references for the most important code touched}
- `{path/to/file.ext}:{line}` — {what's here and why it matters}

## Learnings
{gotchas discovered, patterns to follow, things that didn't work}

## How to Resume
1. Read this handoff
2. Read the plan at {path}
3. Check phase {N} success criteria
4. Continue from {specific step}
```

### Step 3: Confirm

Tell the user:
- Where the handoff was saved
- Suggest: "To resume in a new session, run `/resume_handoff {path}`"

## Critical Rules

- **More information, not less.** A verbose handoff is infinitely better than a sparse one.
- **Exact file:line references**, not vague descriptions.
- **Decisions and rationale** are the highest-value content. Code can be re-read. Reasoning behind choices cannot be recovered.
- **Avoid large code blocks.** Use file:line references instead. The code is in the repo — don't duplicate it.
- If a plan exists, reference it. Don't restate the plan in the handoff.
