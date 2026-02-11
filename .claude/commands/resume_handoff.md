---
description: Resume work from a handoff document with context validation.
model: opus
---

# Resume Handoff

You are resuming work from a previous session's handoff document.

## Input

$ARGUMENTS should be a path to a handoff file. If not provided, look in `docs/handoffs/` for the most recent one (sort by filename timestamp).

## Process

### Step 1: Read and Absorb

1. Read the handoff document COMPLETELY.
2. Read the plan file referenced in the handoff (if any).
3. Read all files listed in "Critical File References".

### Step 2: Validate Current State

The handoff describes a past state. Verify it's still accurate:

```bash
git branch --show-current
git log --oneline -n 10
git status
git diff --stat
```

Spawn sub-agents to check:
- **codebase_analyzer**: Read the critical files referenced in the handoff. Confirm they match what the handoff describes.
- If the plan has automated success criteria for completed phases, run them to confirm they still pass.

### Step 3: Present Understanding

Tell the user:
```
Resuming from handoff: {handoff title}
Created: {date}
Branch: {branch} (current: {actual current branch})

Completed:
- {list from handoff}

In Progress:
- {current state}

Next Steps:
- {what to do next, from handoff}

State Validation:
- {what matches / what's changed since handoff}
```

Ask: "Does this look right? Should I continue from here?"

### Step 4: Continue

After user confirms:
- Create a todo list from "What's Left"
- If a plan exists, use `/implement_plan` to continue execution
- If no plan, work through the remaining items

## Critical Rules

- ALWAYS validate current state. The repo may have changed since the handoff was written.
- If the branch in the handoff doesn't match the current branch, STOP and ask the user.
- If automated checks from completed phases now fail, STOP and report before continuing.
- Don't assume the handoff is perfectly accurate. It was written by a model summarizing a session.
